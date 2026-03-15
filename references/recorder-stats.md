# Recorder, Statistiken & Energie

Teil des ha-betriebshandbuch — Recorder, Zombie-Cleanup, Statistiken, Energie.
Nachladen: `references/recorder-stats.md`

## 10. Recorder & Datenbank

### 10.1 Recorder-Konfiguration (configuration.yaml)
```yaml
recorder:
  purge_keep_days: 14
  exclude:
    domains:
      - automation
      - update
    entity_globs:
      - sensor.*_rssi
      - sensor.*_firmware
      - weather.*
      - sun.*
    entities:
      - sensor.konkrete_entity
```

**WICHTIG:** `history: exclude:` ist deprecated (ab HA 2023) — alle Excludes nur im `recorder:`.

### 10.2 recorder.purge API
```bash
POST /api/services/recorder/purge
Body: {"keep_days": 14, "repack": true}   # mit VACUUM (sofortige Speicherfreigabe)
Body: {"keep_days": 14, "repack": false}  # schnell, Auto-Repack später
```

### 10.3 Direkte SQLite-Abfragen
```bash
# DB-Größe
sqlite3 /config/home-assistant_v2.db \
  "SELECT printf('%.1f MB', page_count*page_size/1024.0/1024.0) \
   FROM pragma_page_count(),pragma_page_size();"

# SQL via REST (nur SELECT)
POST /api/services/sql/query?return_response
Body: {"query": "SELECT entity_id, state FROM states INNER JOIN states_meta \
  ON states.metadata_id = states_meta.metadata_id \
  WHERE states_meta.entity_id = 'sensor.x' ORDER BY last_updated DESC LIMIT 10"}
# Response: service_response.result = Liste von Dicts
```

### 10.4 Long-Term Statistics vs. States

- Energy Dashboard liest aus `statistics` / `statistics_short_term` (nicht aus `states`).
- Aggressive purges sind daher **sicher** für das Energy Dashboard.
- `statistics_short_term` resettet bei HA-Neustart → "Jetzt"-Tab kurz leer, füllt sich selbst.
- `purge_entities keep_days:0` → sofort aus states entfernt.
  Speicherfreigabe danach: `purge {repack: true}`.

---


## 11. Zombie-Entity-Bereinigung

### 11.1 YAML-basierte Sensoren/Automationen (nach Entfernen aus YAML)

**Nach Automation-DELETE per REST:** Entity sofort weg (404) — kein reload nötig.
**Nach manueller YAML-Löschung:** Zombie bleibt in Entity-Registry.

**KRITISCH — Größe beachten:** `core.entity_registry` ist ~2,5–3 MB.
write_file via REST schlägt bei dieser Größe mit HTTP 500 fehl (→ §2.3b).
**Einziger zuverlässiger Weg: direktes Python via SSH-Terminal.**

Cleanup (SSH-Terminal, `/app/a0d7b954_ssh`):
```python
python3 /dev/stdin << 'PYEOF'
import json
p = '/config/.storage/core.entity_registry'
d = json.load(open(p))
TO_REMOVE = {'entity_id_1', 'entity_id_2'}   # entity_id, nicht unique_id!
before = len(d['data']['entities'])
d['data']['entities'] = [e for e in d['data']['entities'] if e.get('entity_id') not in TO_REMOVE]
after = len(d['data']['entities'])
json.dump(d, open(p,'w'), ensure_ascii=False, separators=(',',':'))
print(f'OK: {before} -> {after} ({before-after} entfernt)')
PYEOF
```
Danach: **HA-Vollneustart** (reload überschreibt Storage mit In-Memory-Daten!)

### 11.2 Config-Entry-basierte Sensoren/Helfer
```bash
DELETE /api/config/config_entries/entry/<config_entry_id>
→ 200 OK, sofort wirksam, kein Restart
```

---


## 18. Statistiken reparieren (sum-Reset nach HA-Neustart)

### 18.1 Problem
Nach HA-Neustart springt `sum` in der `statistics`-Tabelle auf 0 zurück,
während `state` (absoluter Zählerstand) korrekt weiterläuft.
Ergebnis im Energy-Dashboard: massiv negativer Tagesbalken.

### 18.2 Diagnose (SQL-Integration, SELECT only)

**Schritt 1 — Reset-Kandidaten finden:**

> ⚠️ **NICHT VERWENDEN — LAG()-Version (veraltet, fehlerhaft seit 08.03.2026):**
> ```sql
> -- BROKEN: LAG()-Window-Funktion liefert zu viele Zeilen → JSONDecodeError "Extra data"
> -- bei SQL-Integration (implizites Row-Limit überschritten). Ersetzt durch Subquery-Variante.
> SELECT s.metadata_id, m.statistic_id, s.id, s.sum,
>        LAG(s.sum) OVER (PARTITION BY s.metadata_id ORDER BY s.id) as prev_sum
> FROM statistics s JOIN statistics_meta m ON s.metadata_id = m.id
> WHERE m.unit_of_measurement IN ('kWh','GB') ORDER BY s.metadata_id, s.id
> ```

**RICHTIG — Subquery-Variante (verifiziert 08.03.2026, gibt nur Einbrüche zurück):**
```sql
SELECT m.statistic_id, s.metadata_id, s.id,
       datetime(s.start_ts,'unixepoch','+1 hour') as cet,
       ROUND(s.sum,4) as bad_sum,
       ROUND(prev.sum,4) as prev_sum,
       ROUND(prev.sum - s.sum, 4) as drop_kwh
FROM statistics s
JOIN statistics_meta m ON s.metadata_id = m.id
JOIN statistics prev ON prev.metadata_id = s.metadata_id
  AND prev.id = (SELECT MAX(id) FROM statistics
                 WHERE metadata_id = s.metadata_id AND id < s.id)
WHERE m.unit_of_measurement = 'kWh'
  AND s.start_ts >= <unix_timestamp_startdatum>
  AND prev.sum - s.sum > 0.5
ORDER BY s.metadata_id, s.id
```
Keine Treffer = DB integer. Zeitraum: `time.time() - 60*24*3600` für 60 Tage.

**Schritt 2 — Vor Fix prüfen: Ist das Problem im Dashboard sichtbar?**
Negative Balken im Energy-Dashboard können auch durch
Differenzberechnung entstehen (→ §18.7). Vor jedem Fix sicherstellen,
dass tatsächlich ein sum-Einbruch vorliegt und dieser dashboard-relevant ist.

**Schritt 3 — Offset berechnen:**
```sql
SELECT bad.id, prev.sum as prev_sum, bad.sum as bad_sum,
       ROUND(prev.sum - bad.sum, 6) as offset
FROM statistics bad
JOIN statistics prev ON prev.metadata_id = bad.metadata_id
  AND prev.id = (SELECT MAX(id) FROM statistics
                 WHERE metadata_id = bad.metadata_id AND id < bad.id)
WHERE bad.metadata_id = <ID> AND bad.id = <ERSTE_BAD_ID>
```

### 18.3 Fix-Prozess (verifiziert 08.03.2026)
1. Python-Skript nach `/config/fix_statistics.py` schreiben (write_file)
2. `run_python: "python3 /config/{{ script }}"` zu shell_command in
   `configuration.yaml` hinzufügen
3. Config-Check: `POST /api/config/core/check_config`
4. HA-Neustart: `POST /api/services/homeassistant/restart`
   (Pflicht — shell_command-Änderungen werden nur beim Vollneustart aktiv)
5. Skript ausführen: `POST /api/services/shell_command/run_python`
   mit `{"script": "fix_statistics.py"}`
6. Verifikation: SQL-Query, Kontinuität der sum-Kurve prüfen
7. Cleanup: Skript löschen (delete_file), `run_python` aus configuration.yaml
   entfernen, zweiter Neustart

### 18.4 Python-Skript-Muster
```python
import shutil, sqlite3, os
DB = "/config/home-assistant_v2.db"
BACKUP = DB + ".backup_DATUM"
if not os.path.exists(BACKUP):
    shutil.copy2(DB, BACKUP)
con = sqlite3.connect(DB, timeout=30)
cur = con.cursor()
# (metadata_id, erste_bad_id, offset)  — offset = prev_sum - bad_sum
fixes = [
    (123, 1815000, 236.388),   # sensor.beispiel_energie
]
for meta_id, first_bad_id, offset in fixes:
    cur.execute(
        "UPDATE statistics SET sum = ROUND(sum + ?, 10) "
        "WHERE metadata_id = ? AND id >= ?",
        (offset, meta_id, first_bad_id)
    )
    print(f"meta={meta_id} → {cur.rowcount} Zeilen")
con.commit()
con.close()
```

### 18.5 Wichtig
- SQL-Integration (SELECT-only) → direkte UPDATEs nicht möglich
- WebSocket-API ist via Nabu Casa **generell nicht erreichbar** (HTTP 403 für alle WS-Commands —
  kein WebSocket-Upgrade-Header-Durchlass). Betrifft ALLE Commands, nicht nur `recorder/import_statistics`.
- SQLite WAL-Modus: Skript kann bei laufendem HA ausgeführt werden,
  historische Einträge werden von HA nicht neu berechnet
- DB-Backup vor jedem Update obligatorisch
- Backup nach erfolgreicher Verifikation manuell löschen (offener Punkt
  im Memory notieren)

### 18.6 sum vs. state — Normalverhalten (KRITISCH)

`sum` und `state` weichen bei `total_increasing`-Sensoren regulär voneinander ab:
- `state` = absoluter Zählerstand des Geräts
- `sum` = von HA akkumulierter Wert — HA gleicht Geräte-eigene Resets aus

**Konsequenz für Offset-Berechnung:**
```
RICHTIG:  offset = prev_sum - bad_sum
FALSCH:   offset = state    - bad_sum  ← state ≠ sum ist normal, kein Fix-Maßstab
```
Beispiel: prev_sum=236.41, bad_sum=0.02, state=236.24
→ Offset = 236.41 - 0.02 = **236.39** (nicht 236.22)

### 18.7 Negative Balken im Energy-Dashboard — Diagnose

Nicht jeder negative Balken ist ein statistics-Reset. Reihenfolge der Prüfung:

1. **„Nicht erfasster Verbrauch" negativ?**
   Das Dashboard berechnet: `Gesamt - Σ(Einzelgeräte)`.
   Wenn Einzelgeräte-Summe > Gesamt (Messunschärfe, Timing-Versatz) → negativ.
   → Kein Datenbankfehler, kein Fix nötig.

2. **sum-Einbruch in `statistics` vorhanden?**
   → SQL-Diagnose §18.2 Schritt 1 ausführen (**Subquery-Variante** — nicht LAG).
   Erst wenn ein echter Einbruch nachgewiesen ist, Fix-Prozess starten.

3. **Batch-Reset (viele Sensoren gleichzeitig)?**
   Typisches Muster nach HA-Neustart: alle kWh-Sensoren springen exakt
   zum gleichen Zeitstempel auf ~0. Alle betroffenen metadata_ids in
   einem einzigen Skript-Durchlauf fixen (nicht einzeln).


---


## 19. Energie-Sensoren — state_class, Glitch-Analyse, Zombie-Erkennung

### 19.1 state_class-Wahl für Energie-Sensoren (verifiziert via HA Developer Docs)

| Szenario | state_class | Begründung |
|----------|-------------|------------|
| Lifetime-Zähler (steigt immer, kein Reset) | `total` | HA trackt Differenz; kein Reset → kein Sprung |
| Zähler mit periodischem Reset auf 0 | `total_increasing` | Wertfall = neuer Zyklus, kein negativer sum-Sprung |
| Netto-Bilanz (kann steigen UND fallen) | `total` | Differenz-Logik korrekt für beide Richtungen |
| Leistung (W), aktueller Messwert | `measurement` | Keine Akkumulation |

**Entscheidungsregel (aus HA Doku):**
> „In most cases, state_class TOTAL without last_reset should be chosen."
> `total_increasing` nur wenn Wert ausschließlich steigt und periodisch auf 0 zurückfällt.

**integration-Platform-Default:** `state_class: total` — korrekt für W→kWh Lifetime-Sensoren.
Nicht auf `total_increasing` ändern, solange kein echter Reset-Zyklus vorliegt.

### 19.2 Glitch-Analyse: Negative Leistungswerte an Verbraucher-PMs

Negative Werte bei Shelly-PMs hinter Solar-Panels sind **kein Glitch** sondern physikalisch korrekt:

```
Netz → [PM Verbraucher] → Verbraucher-Kreis
                               │
                        [PM Schuppen] ← Solar-Panel
```

- PM misst Netto-Bilanz (Bezug − Einspeisung)
- Bei Solar-Überschuss: negativer Wert = Einspeisung ins übergeordnete Netz
- `total_increasing` friert korrekt ein (zählt keine negative Energie) → **DB sauber**
- Negativer Wert pflanzt sich nach oben fort (Verbraucher-PM ebenfalls negativ) → **erwartet**

**Prüfroutine (2-Monats-Check):** → §18.2 Schritt 1 (Subquery-Variante), `start_ts >= <unix_60_tage>`.
Keine Treffer = DB integer. Bei Treffern → §18 Fix-Prozess.

### 19.3 Zombie-Erkennung aus statistics_meta

`statistics_meta` enthält alle Sensoren die jemals Statistiken hatten — auch gelöschte.
Abgleich mit aktiven Entities nach jeder größeren Bereinigung:

```sql
SELECT id, statistic_id FROM statistics_meta WHERE unit_of_measurement = 'kWh'
ORDER BY statistic_id
```

Verdächtige Einträge prüfen:
- Sensor antwortet mit `unavailable` + `"restored": true` → **Zombie** (in Entity-Registry, aber keine YAML-Quelle mehr)
- Sensor antwortet mit 404 → **Vollzombie** (auch aus Registry entfernt, aber noch in statistics_meta)
- `returned_energy` an reinen Verbrauchern (Heizung, Kühlschrank): physikalisch möglich als Messrauschen → beobachten, nicht sofort fixen

**Zombie-Bereinigung:** §11.1 (entity_registry via SSH-Terminal + Neustart).

### 19.4 returned_energy-Sensoren an Verbrauchern

Shelly-PMs erzeugen automatisch `*_returned_energy`-Sensoren. Diese können:
- Wert 0,0 kWh haben → harmlos (kein Messrauschen aufgezeichnet)
- Wert > 0 haben → bei reinen Verbrauchern: Messrauschen oder falsch verkabelter PM

Im Recorder **nicht** ausschließen wenn der Sensor im Energy-Dashboard als Einspeisung konfiguriert ist
(Warnung „Entität nicht nachverfolgt" + „Statistiken nicht definiert" → §19.5).

### 19.5 Energy-Dashboard Warnungen — Bedeutung

| Warnung | Ursache | Fix |
|---------|---------|-----|
| „Entität nicht nachverfolgt" | Sensor in Recorder `exclude:` | Aus exclude entfernen, Neustart |
| „Statistiken nicht definiert" | Sensor neu / gerade aus exclude entfernt | Bis 5 Minuten warten — verschwindet automatisch |
| „Statistiken nicht definiert" nach >10 Min | state_class fehlt oder falsch | Sensor-Attribute prüfen |

