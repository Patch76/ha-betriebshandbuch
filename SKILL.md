---

name: ha-betriebshandbuch
description: >
  HA-Betriebshandbuch für Claude-Instanzen — instanzunabhängiges Systemwissen
  für alle Home Assistant Installationen.

  TRIGGER THIS SKILL WHEN:
  - Lesen, Schreiben oder Debuggen von HA REST API Calls
  - Anlegen oder Ändern von Automationen, Scripts, Szenen, Template-Sensoren
  - Storage-Dateien (.storage/) lesen oder schreiben
  - Helper (input_boolean, input_select, timer, etc.) anlegen oder löschen
  - Recorder-Konfiguration oder Datenbankzugriff
  - YAML-Dateien (automations.yaml, template.yaml, scripts.yaml) editieren
  - Zombie-Entities bereinigen
  - Backup vor destruktiven Aktionen
  - Zigbee-Bulbs steuern (Farbwechsel, Ein-/Ausschalten)
  - Android Companion App next_alarm Sensor nutzen
  - Statistiken reparieren (sum-Reset, Energy-Dashboard-Fehler, DB-Direktzugriff)

  NIEMALS RATEN — bei Unklarheit live testen oder API verifizieren.
metadata:
  version: "2.30.0"
  maintainer: "Claude (via PR, nach Rücksprache mit Mirko)"
  workflow: "Änderungsbedarf → PR auf Patch76/ha-betriebshandbuch → Mirko mergt → nächste Session zieht automatisch"
  source: "Verifiziert an HA 2026.3.0 — aus claude.md + Live-Tests 08.03.2026"
  changelog: >
    2.30.0 (11.03.2026): §5.4 Klarstellung UI-Helper anlegen — `ha_create_config_entry_helper` unterstützt input_boolean/input_datetime NICHT; Storage-Weg korrekt; `ha_reload_core(target=...)` statt HA-Neustart (verifiziert); entity_registry wird automatisch ergänzt. §9.5 neu — Repairs-API: POST /api/repairs/issues/fix + Flow-Bestätigung, verifiziert mit Battery Notes (missing_device_*).
    2.29.0 (11.03.2026): §6.6 Ghost-Entries verallgemeinert — gilt für alle Entity-Typen (Sensoren, Switches etc.), nicht nur Automationen; Pre/Post-Verifikationspflicht ergänzt.
    2.28.0 (11.03.2026): §6.6 Ghost-Automationen (restored: true) dokumentiert — Config-DELETE schlägt fehl, korrekte Lösung: DELETE /api/states/<entity_id>.
    2.27.0 (11.03.2026): §5.2 core.area_registry modified_at ergänzt. §5.4 input_boolean.icon als optional markiert; timer + counter Felder ergänzt.
    2.26.0 (11.03.2026): §2.7 urllib.request-Kommentar ergänzt — kein bash_tool-Wrapping,
      r['service_response'] direkt abrufen (nicht r['stdout']['service_response']).
    2.25.0 (11.03.2026): §3.1 sensor.yaml Hinweis auf manuelle Anlage + !include-Einbindung. §3.2 check_config-Endpunkt präzisiert (api/config/core vs. services).
    2.24.0 (11.03.2026): §2.3b HTTP-500-Grenzwert korrigiert (~50→~95 KB, verifiziert per Binärsuche).
    2.23.0 (11.03.2026): §0 korrigiert — ha_check_update_notes existiert nicht (Tool heißt
      ha_get_updates mit include_release_notes=True). GitHub-PRs-Behauptung „via MCP" war
      Fehlinterpretation einer ha-mcp-internen CI-Änderung (PR #723). Korrekt: PRs via
      bash_tool/GitHub REST API mit "draft":true.
    2.22.0 (11.03.2026): §0 Kanal-Verifikationspflicht — empfangende Instanz prüft
      Behauptungen aus lb-rbo-channel live gegen eigene Instanz vor Umsetzung.
    2.21.0 (11.03.2026): §2.3 + §2.4 Pfad-Typ-Tabelle + Doppel-Slash-Falle dokumentiert
      (write_file/delete_file relativ, read_file absolut). Anti-Pattern ergänzt.
    2.20.0 (11.03.2026): §22.2 Warnhinweis — CLAUDE.md-Änderungen ausschließlich
      via §2.7 atomarer Zyklus (nie computer.type / cat-Heredoc).
    2.19.0 (11.03.2026): §0 ergänzt — ha_check_update_notes vor HA-Updates (Pflicht),
      GitHub-PRs via MCP immer als Draft (stille Verhaltensänderung ha-mcp v7.0.0).
      ⚠️ KORRIGIERT in v2.23.0 (beide Punkte fehlerhaft, siehe dort).
    2.18.0 (09.03.2026): §0 ergänzt — Kanal-Abstimmungspflicht bei instanzübergreifenden
      Wissensänderungen, [UNVERIFIZIERT]-Kennzeichnung im Kanal. Verifiziert LB + RBO.
    2.17.0 (09.03.2026): §§2.10+24 neu — SSH-Terminal-Verhaltensregeln (§2.10), Telegram notify-Service inkl. Escape-Funktion (§24). Verifiziert LB + RBO 09.03.2026.
    2.16.0 (09.03.2026): §23 neu — Verifikationstabelle nach Änderungen. §2.3 Kurzregel Shell-Command-Fehlerbehandlung ergänzt.
    2.15.0 (09.03.2026): §22 neu — CLAUDE.md-Template (Pflicht-Abschnitte + Regeln). §13 Hinweis auf Integrations-Abhängigkeit ergänzt. §15.1 Plattform-Hinweis präzisiert (Add-on vs. Docker).
    2.14.0 (08.03.2026): §2.3b Grenzwert-Widerspruch behoben (100→50 KB). §9.4 Querverweis §18.3→§17.4 korrigiert. §17.2 Tabelle: instanzspezifische Eintrags-Zahlen als Beispiel markiert, kontextlosen MQTT-Bug-Kommentar entfernt.
    2.13.0 (08.03.2026): §14–§21 H3-Subsektionsnummern korrigiert nach Restrukturierung in v2.12.0 (16.x→14.x, 22.x→15.x, 17.x→16.x, 20.x→17.x, 19.x→18.x, 21.x→19.x, 14.1→21.1).
    2.12.0 (08.03.2026): Skill-Restrukturierung — §§14-21 neu geordnet nach Verwendungshäufigkeit. §18 (Supervisor-API) als eigenständige Sektion entfernt, Kernaussage in §1.7 integriert.
    2.11.0 (08.03.2026): §0 Pflicht­regel ergänzt — Live abrufen statt fragen: prüfbaren Zustand immer per API verifizieren, nie den Nutzer fragen.
    2.10.0 (08.03.2026): §0 Verifikationsregel für destruktive Aktionen ergänzt — Schleifen-
    Netzwerkfehler erzeugen Falsch-Negative; Befunde immer isoliert verifizieren.
    2.9.0 (08.03.2026): §2.9 neu — python_script (Legacy) und pyscript (HACS) sind nicht
    standardmäßig vorhanden. Prüfpflicht via GET /api/services vor Nutzung. File-Schreiben
    außerhalb shell_command nicht möglich in Standard-HA (verifiziert RBO + LB-Instanz).
    2.8.0 (08.03.2026): §15 neu — Zigbee2MQTT Add-on Konfigurationszugriff:
    Shadow-DOM verhindert UI-Zugriff → direkter Dateizugriff über
    /config/zigbee2mqtt/configuration.yaml (verifiziert auf HA OS Add-on-Installs, §2.3).
    §15.2 Geräteumbenennung-Verhalten in Z2M dokumentiert.
    2.7.1 (08.03.2026): GitHub-basierter Skill-Load eingeführt.
    2.7.0 (08.03.2026): §4.2 state_class von EMPFOHLEN auf PFLICHT hochgestuft (verifiziert:
    ohne state_class kein Eintrag in statistics_meta, keine Langzeit-Statistiken).
    §4.2 device_class-Spiegelung von Quellsensor als Regel ergänzt.
    §4.3 Hinweis auf triggers:/actions: als empfohlene Syntax seit HA 2024.10 ergänzt
    (Singular weiterhin gültig, kein deprecated).
    §17 Anti-Pattern ergänzt: unit_of_measurement ohne state_class.
    2.6.1 (08.03.2026): §19.2 doppelte Subquery entfernt — Verweis auf §18.2 statt Kopie.
    2.6.0 (08.03.2026): §0 Skill-Pflegepflicht ergänzt.
    2.4.1 (08.03.2026): §9.4 schedule WS-Commands live verifiziert (HA 2026.3.1) — DictStorageCollectionWebsocket, kein config_flow. schedule/create, /list, /delete dokumentiert.
    2.4.0 (08.03.2026): §6.6 REST-Body auf Plural-Keys korrigiert (triggers/conditions/actions); Hinweis dass GET immer plural liefert. §6.9 unverifizierten HA-2026.3-Claim entfernt. §9.4 WebSocket-Aussage korrigiert: WS via Nabu Casa HTTP 403, kein Browser-Kontext verfügbar.
    2.3.0 (08.03.2026): §2.7 EOF-Anker-Falle dokumentiert (letzte Zeile ohne trailing 
). §17 Anti-Pattern ergänzt. §18.5 WebSocket-Einschränkung korrigiert: generell HTTP 403 via Nabu Casa, nicht nur recorder. §17 neu: WebSocket-API — getestete Commands, defekte Commands, Protokoll-Muster.
    2.2.0 (08.03.2026): §18 erweitert — Offset-Berechnung korrigiert (prev_sum - bad_sum, nicht state - bad_sum). §18.6 neu: sum vs. state Normalverhalten erklärt, §18.7 neu: Negative Balken im Energy-Dashboard — Diagnose-Checkliste, §18.2 Diagnoseschritt ergänzt: Sichtbarkeit im Dashboard vor Fix prüfen.
    2.1.1 (08.03.2026): "TRIGGER THIS SKILL WHEN" ergänzt.
    2.1.0 (08.03.2026): §18 neu — Statistiken reparieren (sum-Reset nach HA-Neustart).
    Diagnose via SQL, Fix-Prozess via run_python shell_command + sqlite3 (verifiziert).
    WebSocket-Erkenntnisse: recorder/import_statistics undokumentiert, via Nabu Casa
    nicht erreichbar (HTTP 403). TRIGGER ergänzt: Statistiken reparieren.
    2.0.0 (07.03.2026): Vollkonsolidierung + Live-Verifikation aller Kernaussagen.
    §1.6 Logbook-Zeitformat korrigiert: +00:00 und .000Z beide gültig.
    §2 Nummerierung repariert (2.3b → 2.4, Verschiebung 2.4–2.7 → 2.5–2.8).
    1.6.1 (07.03.2026): §2.3 read_file → filename, write_file → path + content_b64.
---

# HA-Betriebshandbuch für Claude-Instanzen

> Dieses Handbuch enthält **nur instanzunabhängiges** Wissen: API-Syntax, YAML-Regeln,
> Storage-Schemata, bekannte Fallen. Instanz-spezifische Daten (Entity-IDs, IPs, Tokens,
> Automationslisten) gehören in die jeweilige `claude.md`.

---

## 0. Arbeitsweise (Pflichtregeln)

- **Verifikationspflicht:** Kein Wissen ungeprüft übernehmen — weder aus Doku noch aus
  einer anderen Claude-Instanz. Vor Eintrag in claude.md live testen oder per API verifizieren.
- **Backup vor jeder destruktiven Aktion:**
```bash
  cp /config/DATEI /config/tmp_backup_DATEI
```
  Gilt für: automations.yaml, template.yaml, scripts.yaml, scenes.yaml, alle `.storage/`-Dateien.
- **Bei unbekanntem Fehler zuerst `web_search`** — nie mehr als 15 Minuten blind debuggen.
- **Öffentliche Doku-URLs:** `web_fetch` ist erst nach vorherigem `web_search` auf dieselbe Domain möglich
  (URL muss in Suchergebnissen erschienen sein). Browser (`computer`-Tool) ist für öffentliche Docs NICHT
  nötig — nur für HA-UI (Auth-geschützt) oder interaktive Seiten erforderlich.
  Muster: `web_search("HA integration xyz site:developers.home-assistant.io")` → dann `web_fetch(url)`.
- **Verifikation vor destruktiver Aktion:** Befunde, die eine destruktive Aktion begründen
  (Löschen, Entfernen, PR), immer einzeln und isoliert verifizieren — nie aus Schleifenergebnissen
  ableiten. Netzwerkfehler in Schleifen erzeugen Falsch-Negative.
- **Neustart nur mit expliziter Benutzerbestätigung.**
- **Vor HA-Updates: `ha_get_updates(entity_id="update.home_assistant_core_update", include_release_notes=True)` ausführen** (Impact-Review).
  Prüft Breaking Changes und Hinweise vor dem Update — Pflicht bevor `homeassistant/restart` nach einem Update ausgeführt wird.
- **GitHub-PRs immer als Draft anlegen** (via bash_tool + GitHub REST API).
  `"draft": true` im POST-Body an `POST /repos/Patch76/<repo>/pulls`. Manuell auf „Ready" setzen.
- **Live abrufen statt fragen:** Ist ein Zustand per API prüfbar (Entity-State, last_triggered, Attribut), immer live abrufen — nie den Nutzer fragen. Fragen nur wenn der Kontext wirklich nicht abrufbar ist.
- **Skill-Pflege (KRITISCH):** Beim Aktualisieren des Skills gilt:
  - Veraltetes Wissen **streichen oder als `⚠️ VERALTET` kennzeichnen** — nie still ergänzen.
  - Neues Wissen ersetzt altes: beide Stellen anpassen (nicht nur neue Sektion anfügen).
  - Querverweise prüfen: Verweist §X auf §Y? → §Y-Änderung erfordert §X-Check.
  - Nur live getestetes oder per API/Doku verifiziertes Wissen eintragen.
  - Version + Changelog bei jedem Update pflegen.
  - **Instanzübergreifende Wissensänderungen** (API-Verhalten, neue §§, Templates):
    Kanal-Abstimmung (lb-rbo-channel) vor PR Pflicht — nicht die Anwendung,
    nur die Änderung gemeinsamen Wissens triggert Abstimmung.
  - **Instanzübergreifende Behauptungen im Kanal:** mit `[UNVERIFIZIERT-LB]` /
    `[UNVERIFIZIERT-RBO]` kennzeichnen, wenn nicht live getestet.
- **Kanal-Verifikationspflicht beim Einlesen (KRITISCH):**
  Behauptungen aus `lb_to_rbo.md` / `rbo_to_lb.md` **nie blind übernehmen** — immer
  live gegen die eigene Instanz gegenchecken, bevor eine Änderung umgesetzt wird.

  | Behauptungstyp | Verifikation |
  |---|---|
  | Skill-Version X | `curl SKILL.md \| grep version` → tatsächliche Version prüfen |
  | CLAUDE.md wurde geändert | `read_file /config/CLAUDE.md` → Stichprobe der behaupteten Stelle |
  | API-Verhalten (z.B. Pfad-Regel) | Testaufruf mit bekanntem Ergebnis durchführen |
  | MCP-Tool verfügbar | `ha_list_services` oder Tool direkt aufrufen |
  | Nicht testbar | Als `[UNVERIFIZIERT]` kennzeichnen — nie stillschweigend als wahr annehmen |

  Ergebnis der Verifikation in der eigenen Antwort-Nachricht (`rbo_to_lb.md` / `lb_to_rbo.md`)
  kurz dokumentieren: ✓ verifiziert / ✗ abweichend (mit Befund).

---

## 1. REST API — Endpunkte & Syntax

### 1.1 Authentifizierung

Jeder Call braucht:
```
Authorization: Bearer <LONG_LIVED_TOKEN>
Content-Type: application/json
```

HTTP-Statuscodes: `200` OK, `201` Created, `400` Bad Request, `401` Unauthorized,
`404` Not Found.

### 1.2 Wichtige Endpunkte (verifiziert)

| Methode | Pfad | Beschreibung |
|---------|------|-------------|
| GET  | `/api/` | API-Ping (Slash am Ende PFLICHT — `/api` ohne Slash → 404) |
| GET  | `/api/config` | Systemkonfig (version, timezone, lat, lon, elevation) |
| GET  | `/api/states` | Alle Entity-States (kein `?domain=`-Filter — nur client-seitig filterbar) |
| GET  | `/api/states/<entity_id>` | Einzelnen State lesen |
| POST | `/api/states/<entity_id>` | State synthetisch setzen (kein echtes Gerät!) |
| POST | `/api/services/<domain>/<service>` | Service aufrufen |
| POST | `/api/template` | Jinja2-Template rendern → Plaintext |
| GET  | `/api/history/period[/<ISO-Z>]` | State-Verlauf |
| GET  | `/api/logbook[/<ISO-Z>]` | Logbuch-Einträge |
| GET  | `/api/services` | Alle Service-Domains auflisten |
| POST | `/api/config/core/check_config` | YAML-Syntaxprüfung |
| POST | `/api/services/automation/reload` | Automationen neu laden |
| POST | `/api/services/template/reload` | Template-Sensoren neu laden |
| POST | `/api/services/recorder/purge` | DB bereinigen |
| POST | `/api/config/automation/config/<id>` | Einzelne Automation anlegen/überschreiben |
| DELETE | `/api/config/automation/config/<id>` | Einzelne Automation löschen |
| DELETE | `/api/config/config_entries/entry/<id>` | Config-Entry löschen |

**FALLE Config-Entry-Löschen:**
```
FALSCH: DELETE /api/config/config_entries/<id>       → 404
RICHTIG: DELETE /api/config/config_entries/entry/<id> → 200
```

### 1.3 Service-Call-Syntax
```json
// RICHTIG — entity_id top-level
{"entity_id": "input_boolean.mein_helper"}

// FALSCH — target: gilt nur in YAML/WebSocket, nicht im REST-Body
{"target": {"entity_id": "input_boolean.mein_helper"}}  → HTTP 400
```

Mit `?return_response` am URL erhält man `service_response` im Body zurück
(z.B. für SQL-Queries, shell_command, Wetter-Forecast):
```
POST /api/services/shell_command/read_file?return_response
POST /api/services/sql/query?return_response
```

### 1.4 Template-API
```bash
curl -X POST -H "Authorization: Bearer TOKEN" -H "Content-Type: application/json" \
  -d '{"template": "{{ states(\"sensor.temperatur\") | float | round(1) }}"}' \
  https://HA_HOST/api/template
```
→ Antwort: **Plaintext** (kein JSON-Wrapper).

**WARNUNG Jinja2-Booleans:** `{{ is_state(...) }}` → liefert `True`/`False` (Python),
nicht `true`/`false` (JSON).
- Fix: `{{ is_state(...) | lower }}` oder `{{ iif(..., 'true', 'false') }}`

`last_changed` in Templates:
- `state_attr(entity, 'last_changed')` → `None` (funktioniert nicht)
- `states.entity_id.last_changed | string` → funktioniert

### 1.5 History-API
```
GET /api/history/period/<ISO-Z>?filter_entity_id=<entity_id>&minimal_response=true
```
- `minimal_response=true` **immer** angeben (deutlich schneller).
- `no_attributes=true` für noch schnellere Abfragen ohne Attribute.
- Ohne Startzeitpunkt: letzter Tag.

### 1.6 Logbook-API
```
GET /api/logbook/<ISO-Z>?entity_id=<entity_id>
```
- Zeitstempel-Formate: **beide** werden akzeptiert (verifiziert 07.03.2026):
  - `2026-03-07T00:00:00.000Z` (UTC mit Z)
  - `2026-03-07T00:00:00+00:00` (UTC mit Offset)
- `entity_id=`-Filter funktioniert **nicht zuverlässig** — alle Entities können zurückkommen.
  (Verifiziert 06.03.2026: `entity=` liefert 0 Ergebnisse, `entity_id=` liefert ungefilterte Einträge.)

### 1.7 Nicht verfügbare Endpunkte

- `GET /api/error_log` → über Nabu Casa: **404** (nicht verfügbar).
  Template-Konfigurationsfehler nur im Browser unter `/config/logs` sichtbar.
- `recorder.reload` existiert nicht → `POST /api/services/recorder/reload` → **400**.
  Nach recorder-Änderungen: **HA-Vollneustart** erforderlich.
- `persistent_notification` nicht per REST State-abfragbar:
  `GET /api/states/persistent_notification.<id>` → **404**.

  **Workarounds für Tests:**
  - `input_boolean` als Zustandsmarker: nach der zu testenden Aktion `turn_on` aufrufen,
    dann `/api/states/input_boolean.<marker>` pollen.
  - `last_triggered` der auslösenden Automation prüfen:
    `GET /api/states/automation.<n>` → Attribut `last_triggered`.


- **Supervisor-API (`/api/hassio/`):** Verwendet `SUPERVISOR_TOKEN` (nicht den Long-Lived Token). Nur innerhalb eines Add-ons verfügbar — aus Claude-Sessions **nicht nutzbar**. Add-on-Konfiguration nur per HA-UI oder SSH.
---

## 2. shell_command — JSON-Parse-Pfad (KRITISCH)

### 2.1 Über bash_tool (curl | python3 -c)
```python
import json, sys
d = json.load(sys.stdin)           # HA-Antwort direkt auf stdin
content = d['service_response']['stdout']

# FALLE: d['stdout'] → KeyError!
# Das bash_tool-Wrapping {"returncode","stdout","stderr"} ist nur
# die Rückgabe an Claude, NICHT was python3 auf stdin liest.
```

### 2.2 Über Browser (javascript_tool / fetch)
```javascript
const d = await response.json();    // kein äußeres Wrapping
const content = d.service_response.stdout;

// FALLE: d['stdout'] → undefined → leer, kein Fehler!
```

### 2.3 write_file

- Parameter für Dateiname heißt `path` (NICHT `filename` — das ist read_file!).
  Falscher Parametername → IsADirectoryError (`open('/config/' + '', ...)`).
- **`path` muss relativ sein** — `/config/` wird automatisch vorangestellt.
  `path: "CLAUDE.md"` → schreibt nach `/config/CLAUDE.md` ✓
  `path: "/config/CLAUDE.md"` → schreibt nach `/config//config/CLAUDE.md` ✗ (Doppel-Slash-Falle!)
- Parameter für Inhalt heißt `content_b64` (NICHT `content`).
  Falscher Parametername → **leere Datei**, returncode 0, kein Fehler!
- `write_file` schreibt **IMMER** nach `/config/` (hardcoded Prefix). Kein Schreiben nach `/tmp/`.
- UTF-8-Encoding: `base64.b64encode(text.encode('utf-8')).decode('ascii')`.
- Lesen + Modifizieren + Schreiben **immer in einem Python-Block** (atomarer Zyklus → §2.7).
- **Fehlerbehandlung:** Bei rc≠0 oder leerem stdout max. 2 Versuche, dann SSH-Tab als Fallback.

### 2.3b write_file — HTTP-500-Limit bei großen Dateien (KRITISCH, verifiziert 07.03.2026)

`write_file` via REST (bash_tool curl oder Browser-Fetch) **schlägt bei Payloads >~95 KB
mit HTTP 500 fehl** — ohne aussagekräftige Fehlermeldung.
(Verifiziert 11.03.2026: Grenzwert binär eingekreist 95–99 KB; frühere Angabe ~50 KB zu konservativ.)

Betroffen sind insbesondere:
- `core.entity_registry` (~2,5–3 MB)
- `core.device_registry` (ähnliche Größe)

**FALSCH — schlägt bei großen Dateien lautlos fehl:**
```bash
# curl -d mit großem Body → "File name too long" oder HTTP 500
curl ... -d "{\"path\":\"...\",\"content_b64\":\"<3MB-String>\"}"

# Browser-Fetch → HTTP 500 (keine Fehlermeldung im JS-catch)
await fetch('.../write_file?return_response', {body: JSON.stringify({...riesiger-b64...})})
```

**RICHTIG — direktes Python auf der HA-Maschine via SSH-Terminal (§11.1):**
```python
# Direkt in /config/ schreiben — kein Netzwerk-Overhead, kein Payload-Limit
python3 /dev/stdin << 'PYEOF'
import json
p = '/config/.storage/core.entity_registry'
d = json.load(open(p))
# ... Änderungen ...
json.dump(d, open(p,'w'), ensure_ascii=False, separators=(',',':'))
PYEOF
```

**Entscheidungsregel:**
- Datei < ~95 KB → write_file via REST (bash_tool + urllib.request, §2.7) ausreichend
- Datei > ~95 KB → **immer** direkt Python via SSH-Terminal

**Kurzbefehl zum Prüfen:**
```bash
wc -c /config/.storage/DATEI   # > 50000 Bytes → SSH-Terminal verwenden
```

### 2.4 delete_file (verifiziert 07.03.2026)

Definition in `configuration.yaml`:
```yaml
shell_command:
  delete_file: "rm -f /config/{{ path }}"
```

Aufruf:
```bash
POST /api/services/shell_command/delete_file?return_response
Body: {"path": "dateiname.txt"}   # relativ zu /config/ — genau wie write_file
# NIEMALS absoluten Pfad: {"path": "/config/dateiname.txt"} → /config//config/dateiname.txt (Fehler!)
```
- `rm -f` → rc=0 auch wenn Datei nicht existiert (kein Fehler).
- Nur innerhalb `/config/` möglich (hardcoded Prefix).

**Pfad-Typ-Übersicht (KRITISCH):**

| Command | Parameter | Pfad-Typ | Beispiel korrekt |
|---------|-----------|----------|-----------------|
| `read_file` | `filename` | **absolut** | `"/config/CLAUDE.md"` |
| `write_file` | `path` | **relativ** | `"CLAUDE.md"` |
| `delete_file` | `path` | **relativ** | `"CLAUDE.md"` |

Absoluter Pfad bei `write_file`/`delete_file` → Doppel-Slash `/config//config/...` → Datei landet falsch oder Fehler.
- **Nach Ergänzung in configuration.yaml: HA-Vollneustart erforderlich** (kein shell_command-Reload).

### 2.5 read_file

- Parametername: `filename` (NICHT `path` — das ist write_file!).
- Kann **beliebigen absoluten Pfad** lesen: `cat '{{ filename }}'` — kein Pfad-Limit.
- `/tmp/` lesen: prinzipiell möglich, aber **write_file kann nicht nach `/tmp/` schreiben**
  → `/tmp/` als Zwischenablage praktisch unbrauchbar. Immer `/config/tmp_out.txt` verwenden.
- Getestet: `/proc/version` → rc=0 (07.03.2026).

### 2.6 stdin-Konflikt: `curl | python3 << 'HEREDOC'` (KRITISCH)

**Problem:** Wenn `python3` per Heredoc (`<< 'PYEOF'`) aufgerufen wird UND gleichzeitig
eine Pipe (`curl ... |`) stdin liefert, **gewinnt das Heredoc**. Python liest das Skript
von stdin — die Pipe-Daten sind weg.

**Symptom:** `JSONDecodeError: Expecting value: line 1 column 1 (char 0)`
(stdin leer, nicht fehlerhaftes JSON — deshalb täuschend)

**Ursache:** stdin kann nur einer Quelle zugewiesen sein. `<< 'HEREDOC'` bindet stdin
an den Heredoc-Puffer. Reproduzierbar:
```bash
echo '{"key":"val"}' | python3 << 'PYEOF'
import sys; print(repr(sys.stdin.read()))  # → '' (leer!)
PYEOF
```

**RICHTIG — Zwischendatei:**
```bash
# Schritt 1: curl-Output in Datei
curl -s -H "Authorization: Bearer TOKEN" \
  -X POST -H "Content-Type: application/json" \
  -d '{"filename":"/config/DATEI"}' \
  'https://HA_HOST/api/services/shell_command/read_file?return_response' \
  -o /tmp/ha_response.json

# Schritt 2: Python-Heredoc liest aus Datei — stdin frei
python3 << 'PYEOF'
import json
raw = json.load(open('/tmp/ha_response.json'))
c = raw['service_response']['stdout']
# ... Verarbeitung
PYEOF
```

**Alternativ — Einzeiler mit `-c` (Pipe bleibt frei):**
```bash
curl ... | python3 -c "import json,sys; d=json.load(sys.stdin); print(d['service_response']['stdout'])"
```

**Faustregel:**
- `Heredoc + Pipe` auf demselben `python3`-Aufruf → immer Zwischendatei
- `-c 'einzeiler'` + Pipe → OK
- Mehrzeiliges Skript + Pipe-Daten → Zwischendatei oder `urllib.request` statt Pipe

### 2.7 Atomarer Lese-Modifizier-Schreib-Zyklus

Standardmuster für alle Dateiänderungen via bash_tool. **Ausschließlich in einem
zusammenhängenden Python-Block** — nie aufgeteilt auf mehrere Tool-Calls.
```python
import json, base64, urllib.request

TOKEN = "..."
BASE  = "https://HA_HOST"

# 1. LESEN (vorher: curl -o /tmp/ha_response.json)
curl_out = json.load(open('/tmp/ha_response.json'))
c = curl_out['service_response']['stdout']

# 2. repr()-CHECK (Pflicht vor str.replace)
# python3 -c "c=open('/config/FILE').read(); [print(repr(l)) for l in c.splitlines() if 'Begriff' in l]"
# Grund: Umlaute, Quotes, Whitespace können vom Screenshot abweichen → replace() schlägt lautlos fehl
# ACHTUNG EOF: Letzte Zeile einer Datei hat oft KEIN trailing 
.
#   Anker niemals mit 
 abschließen wenn er auf der letzten Zeile liegt.
#   Immer den Anker 1:1 aus repr()-Output kopieren — nie aus Screenshot oder Gedächtnis.

# 3. ÄNDERN
OLD = 'exakt kopierter String aus repr()-Output'
NEW = 'neuer Inhalt'
assert OLD in c, f"Anker nicht gefunden: {repr(OLD[:60])}"
c = c.replace(OLD, NEW, 1)

# 4. SCHREIBEN
def write_file(path, content):
    b64 = base64.b64encode(content.encode('utf-8')).decode('ascii')
    body = json.dumps({"path": path, "content_b64": b64}).encode()
    req = urllib.request.Request(
        f"{BASE}/api/services/shell_command/write_file?return_response",
        data=body,
        headers={"Authorization": f"Bearer {TOKEN}", "Content-Type": "application/json"},
        method="POST"
    )
    # urllib.request liefert HA-JSON direkt — kein bash_tool-Wrapping!
    # r['service_response'] direkt abrufen (NICHT r['stdout']['service_response'])
    return json.loads(urllib.request.urlopen(req).read())

r = write_file("PFAD_RELATIV_ZU_CONFIG", c)   # z.B. "automations.yaml", "CLAUDE.md"
print(f"rc={r['service_response']['returncode']} err={r['service_response']['stderr'] or 'OK'}")

# 5. VERIFIZIEREN (separater curl-Call)
# GET /api/states/<entity> oder erneutes read_file + Stichprobe
```

**VERBOTEN:**
- `computer.type` für Inhalte >200 Zeichen (Disconnect-Risiko)
- `cat`-Heredoc für Dateiinhalt (lautloser Datenverlust bei Verbindungsabbruch)
- Mehrere `str.replace()` ohne vorherigen `assert`-Check je Anker

### 2.8 shell_command — kein reload-Service (KRITISCH, verifiziert 07.03.2026)

`shell_command` hat **keinen eigenen reload-Service**.
- `POST /api/services/homeassistant/reload_core_config` → lädt shell_command **nicht** neu.
- `POST /api/services/homeassistant/reload_all` → ebenfalls **kein** Effekt auf shell_command.
- **Einzige Lösung: HA-Vollneustart** (mit expliziter Benutzerbestätigung).

Konsequenz: Neue shell_commands (z.B. `delete_file`) sind erst nach Neustart verfügbar.


### 2.9 python_script / pyscript — Verfügbarkeit prüfen

`python_script` (Legacy-Domain) und `pyscript` (HACS-Integration) sind
NICHT standardmäßig vorhanden.

**Vor Nutzung prüfen:**
- Legacy: `GET /api/services` → domain `python_script` vorhanden?
- pyscript: `GET /api/services` → domain `pyscript` vorhanden?

Falls nicht verfügbar: File-Schreiben ausschließlich via `write_file`
shell_command (§2.3). `open()` steht nur in Add-ons und SSH zur
Verfügung — nicht in Automationen, Scripts oder template.yaml.

### 2.10 SSH-Terminal — Verhaltensregeln

- **Tab-URL nie verlassen** — Navigation weg vom SSH-Tab öffnet den Dialog
  „Website verlassen?“ und bricht die Session unwiderruflich ab.
- **Browser-Aktionen immer in neuem Tab** öffnen (`tabs_create_mcp`),
  nie im SSH-Tab selbst navigieren.
- **Fallback:** SSH-Tab-URL instanzspezifisch im SI/AW dokumentiert.

---

## 3. YAML-Konfiguration — Dateien & Architektur

### 3.1 Standard-Dateistruktur
```
configuration.yaml  — !include: automations.yaml, template.yaml,
                                scripts.yaml, scenes.yaml
secrets.yaml        — Zugangsdaten (nie in anderen Dateien)
sensor.yaml         — Legacy-Plattform-Sensoren (history_stats, integration etc.)
                      Nicht im HA-Standard enthalten — muss manuell angelegt und per
                      `sensor: !include sensor.yaml` in configuration.yaml eingebunden werden.
```

Neue **Template-Sensoren** ausschließlich in `template.yaml`.
**Plattform-Sensoren** (`platform: integration`, `platform: history_stats` etc.) gehören in `sensor.yaml`
— diese können nicht in `template.yaml` stehen.

### 3.2 YAML-Editierregeln (Pflicht)

- **Nie** den HA-UI-YAML-Editor nutzen — er rückt automatisch ein.
- **Nie** `computer.type` für mehrzeiligen YAML.
- **Nie** `cat`-Heredoc für Dateiinhalt — lautloser Datenverlust bei Abbruch.
- **Immer** Python-Heredoc im Terminal oder bash_tool + Python urllib write_file.
- `sed` nur ohne einfache Anführungszeichen im Pattern; sonst Python `string.replace()`.
- **`repr()`-Check vor jedem `str.replace()`:**
```bash
  python3 -c "c=open('/config/FILE').read(); \
    [print(repr(l)) for l in c.splitlines() if 'Begriff' in l]"
```
  Grund: Umlaute, Quotes, Whitespace können abweichen → `replace()` schlägt lautlos fehl.
- **Workflow:** Änderung → `check_config` → `reload`/Restart → API-Verifikation.
  `check_config` = `POST /api/config/core/check_config` → `{"result":"valid","errors":null}`
  NICHT `POST /api/services/homeassistant/check_config` — gibt `[]` zurück (kein Response-Support).

---

## 4. template.yaml — Template-Sensoren

### 4.1 Block-Falle (KRITISCH)

Neue Sensoren **blind ans Dateiende** anfügen → landen im letzten offenen Block.
Ist der letzte Block `binary_sensor:` → `state_class` ungültig → **ganzer Block stumm verworfen**.

**Symptom:** Reload HTTP 200, `check_config` valid, Entities erscheinen nicht.
**Fehler nur in:** `/config/logs` (nicht via `/api/error_log`!).

**Regel:** Neue `sensor:`-Einträge **immer** in explizitem `- sensor:`-Block:
```yaml
template:
  - sensor:        # ← explizit, nie blind ans Ende
    - name: "Mein Sensor"
      unique_id: mein_sensor_uid
      state: "{{ states('sensor.quelle') | float | round(1) }}"
```

### 4.2 Modernes Template-Format (empfohlen ab 2022)
```yaml
template:
  - sensor:
    - name: "Außentemperatur Fahrenheit"
      unique_id: aussentemperatur_fahrenheit   # PFLICHT für UI-Anpassung
      device_class: temperature                # PFLICHT wenn Quellsensor device_class hat
      state_class: measurement                 # PFLICHT wenn unit_of_measurement gesetzt
      unit_of_measurement: "°F"
      availability: "{{ has_value('sensor.aussentemperatur') }}"  # EMPFOHLEN
      state: >
        {{ (states('sensor.aussentemperatur') | float * 9/5 + 32) | round(1) }}
```

**Pflichtfelder:**
- `unique_id` → erlaubt UI-Umbenennen, Area-Zuweisung, entity_id-Änderung.
- `availability` → verhindert Fehlwerte wenn Quelle `unavailable`/`unknown` ist.
  Bevorzugt: `has_value('sensor.quelle')`, alternativ:
  `"{{ states('sensor.quelle') not in ['unavailable','unknown','none'] }}"`

**state_class — PFLICHT wenn `unit_of_measurement` gesetzt (verifiziert 08.03.2026):**
Ohne `state_class` schreibt HA **keine** Langzeit-Statistiken — kein Eintrag in
`statistics_meta`, kein History-Graph, kein Energy-Dashboard-Zugriff.
Werte: `measurement` (aktueller Messwert), `total` (Lifetime-Zähler),
`total_increasing` (Zähler mit periodischem Reset).

**device_class — von Quellsensor spiegeln:**
`device_class` des Quellsensors übernehmen (z.B. `duration`, `temperature`, `energy`).
Ermöglicht korrekte Einheiten-Umrechnung und korrekte Darstellung in History/Dashboard.

### 4.3 Trigger-basierte Template-Sensoren

Seit HA 2024.10 ist `triggers:` / `actions:` die empfohlene Syntax (Plural).
Die Singularformen `trigger:` / `action:` bleiben gültig — kein deprecated, keine Warnings.
Neue Templates immer mit Pluralform schreiben.

```yaml
template:
  - triggers:
      - trigger: state
        entity_id: sensor.aussentemperatur
        not_to:
          - unknown
          - unavailable
    sensor:
      - name: "Gefilterter Sensor"
        unique_id: gefilterter_sensor
        state: "{{ states('sensor.aussentemperatur') | float | round(1) }}"
```
Vorteil: Aktualisiert nur bei tatsächlicher Änderung, kein Polling.

### 4.4 Zwei Wege für Template-Sensoren

1. **YAML** (`template.yaml`): Reload via `POST /api/services/template/reload`.
   Nach Entfernen: Zombie in Entity-Registry → Cleanup erforderlich (→ §11).

2. **Config-Entry** (bevorzugt für einzelne Sensoren):
```
   POST /api/config/config_entries/flow
   Body: {"handler": "template", "show_advanced_options": false}
   → Typ wählen → {"name": "...", "state": "{{ ... }}"}
```
   Sofort live. Löschen: `DELETE /api/config/config_entries/entry/<entry_id>` → 200 OK.

### 4.5 Numerische Template-Sensoren

State-Template **muss** eine Zahl oder `none` zurückgeben (kein String wie `"unknown"`),
wenn `state_class` oder `unit_of_measurement` gesetzt ist:
```yaml
state: >
  {% set val = states('sensor.quelle') | float(-1) %}
  {{ val if val >= 0 else none }}
```

---

## 5. Storage-Dateien (.storage/) — Regeln

### 5.1 Grundregel (KRITISCH)

**Vor jedem Schreiben:** bestehendes Objekt als Vorlage lesen, **alle** Felder übernehmen.
```bash
python3 -c "import json; reg=json.load(open('/config/.storage/DATEI')); \
  print(json.dumps(reg['data']['items'][0], indent=2))"
```

**Nach Storage-Änderung:** HA **vollständig neu starten** (kein reload —
reload überschreibt Storage mit In-Memory-Daten!).

### 5.2 Zeitstempel-Format je Datei (verifiziert 06.03.2026)

Pflichtfelder unterscheiden sich je Datei:

| Datei | created_at / modified_at |
|-------|--------------------------|
| `input_boolean` | **KEIN** created_at/modified_at |
| `core.entity_registry` | ISO-String `"2026-02-28T16:35:45.181313+00:00"` |
| `core.area_registry` | ISO-String — `created_at` + `modified_at` (beide Pflicht) |
| `core.device_registry` | ISO-String (nicht separat verifiziert — Vorlage lesen!) |
| `core.label_registry` | ISO-String, `created_at` PFLICHT |

**Vorlage-Eintrag 1:1 kopieren.** Nie Felder aus anderer Registry-Datei als Vorlage!

Korrekte UTC-Zeit:
```python
from datetime import datetime, timezone
datetime.now(timezone.utc).isoformat()
# NICHT: datetime.utcnow().isoformat()  → kein Timezone-Info (deprecated)
```

### 5.3 collection.hash — OPTIONAL (verifiziert 06.03.2026)

`options.collection.hash` ist **nicht erforderlich**.
Live-Test: 10/11 input_boolean/input_text-Helfer haben keinen Hash — alle funktionieren.
Die Hash-Formel `md5(unique_id)` liefert nicht den gespeicherten Wert — Herkunft unbekannt.

**Regel:** Hash weglassen. Nur übernehmen wenn Vorlage ihn bereits enthält.

### 5.4 Doppelregistrierung bei UI-Helpern

**Anlegen von `input_boolean` / `input_datetime` / `input_select` / `counter` / `timer`:**

⚠️ `ha_create_config_entry_helper` unterstützt diese Typen **NICHT** (nur: template, group, utility_meter, derivative, min_max, threshold, integration, statistics, trend, random, filter, tod, generic_thermostat, switch_as_x, generic_hygrostat).

**Korrekter Weg (verifiziert 11.03.2026):**
1. Beide Storage-Dateien schreiben (siehe unten)
2. `ha_reload_core(target="input_booleans")` bzw. `target="input_datetimes"` etc. aufrufen — **kein voller HA-Neustart nötig**
3. HA ergänzt `core.entity_registry` automatisch — manuelles Schreiben in entity_registry ist NICHT nötig.

`POST /api/config/config_entries/flow` mit `{"handler": "input_boolean"}` → `"message": "Invalid handler specified"` (kein Config-Flow-Handler).

Jeder UI-Helper muss in **zwei** Storage-Dateien stehen:

1. `/config/.storage/input_boolean` (o.ä.):
   - `input_boolean`: `{id, name, icon (optional)}` — **kein** created_at/modified_at
   - `input_select`: `{id, name, icon, options: [...], initial: "..."}`
   - `input_datetime`: `{id, name, icon: null, has_date: true, has_time: true, initial: null}`
   - `input_text`: `{id, name, icon, min: 0, max: 100, mode: "text"}` — **kein** created_at/modified_at
  - `timer`: `{id, name, icon (optional), duration (optional), restore (optional)}` — **kein** created_at/modified_at
  - `counter`: `{id, name, initial, restore, minimum, maximum, step}` — **kein** created_at/modified_at

2. `/config/.storage/core.entity_registry`:
   - Alle Felder einer Vorlage kopieren, dann anpassen.
   - Pflicht: `entity_id`, `id` (uuid4 hex), `created_at`/`modified_at` (ISO-String),
     `original_name`, `object_id_base`

### 5.5 Template-Entities — kein Storage-Eintrag nötig

Template-Sensoren aus `template.yaml` erscheinen in der Helfer-Liste mit Schloss-Symbol.
Das ist korrekt — YAML ist der richtige Ort, kein Storage-Mechanismus verfügbar.
Kein Eintrag in `input_boolean` o.ä. erforderlich, kein Migrations-Bedarf.

### 5.6 Weitere Storage-Pfade
```
/config/.storage/core.entity_registry   — Entities (inkl. Helfer, Template-Sensoren)
/config/.storage/core.device_registry   — Geräte
/config/.storage/core.area_registry     — Areas/Räume
/config/.storage/core.label_registry    — Labels (Pflichtfeld: created_at)
/config/.storage/input_boolean          — input_boolean-Helper
/config/.storage/input_select           — input_select-Helper
/config/.storage/input_text             — input_text-Helper
/config/.storage/timer                  — Timer-Helper
/config/.storage/counter                — Counter-Helper
/config/.storage/trace.saved_traces     — Automations-Traces (Key: "automation.<unique_id>")
```

---

## 6. Automationen (automations.yaml)

### 6.1 Architektur

- YAML-only: `/config/automations.yaml` (Dict-Format via `!include`).
- Kein Config-Entry, kein Storage. **Nicht** per Config-Flow verwaltbar.
- `unique_id` = YAML `id:`-Feld. `entity_id` wird aus `alias` abgeleitet.
- Modus-Standard: `single`.

### 6.2 Modernes YAML-Format (ab HA 2024.x)

Neue Pluralform (rückwärtskompatibel, bevorzugen):
```yaml
- id: "meine_automation"
  alias: "Licht Küche bei Bewegung"
  description: "Beschreibung..."
  triggers:          # früher: trigger:
    - trigger: state  # früher: platform: state
      entity_id: binary_sensor.bewegung_kueche
      to: "on"
  conditions:        # früher: condition:
    - condition: time
      after: "07:00:00"
      before: "23:00:00"
  actions:           # früher: action:
    - action: light.turn_on
      target:
        entity_id: light.kueche
  mode: restart
```

### 6.3 Automation-Modi

| Modus | Verhalten | Wann benutzen |
|-------|-----------|---------------|
| `single` (default) | Neue Auslösung ignoriert wenn bereits aktiv | Einmalige Aktionen, Notifications |
| `restart` | Laufende Instanz abbrechen, neu starten | Bewegungslicht mit Timeout ← bevorzugt |
| `queued` | Stapeln, nacheinander abarbeiten | Türschloss-Sequenzen |
| `parallel` | Mehrere Instanzen gleichzeitig | Entity-unabhängige Aktionen |

**WARNUNG `mode: queued`:** Stapelt Instanzen → State-Konflikte bei schnellen Triggern.
`mode: restart` bevorzugen.

### 6.4 Trigger-IDs
```yaml
triggers:
  - trigger: state
    entity_id: input_boolean.abwesenheit
    to: "on"
    id: "abwesenheit_an"
  - trigger: state
    entity_id: input_boolean.abwesenheit
    to: "off"
    id: "abwesenheit_aus"

conditions:
  - condition: trigger
    id: "abwesenheit_an"   # nur wenn dieser Trigger feuerte
```

### 6.5 wait_for_trigger vs. wait_template — Semantik
```
wait_for_trigger: wartet auf tatsächliche Zustandsänderung.
  Ist Bedingung bereits erfüllt → wartet EWIG.
  Niemals für „schon-wahr"-Situationen.

wait_template: läuft SOFORT durch wenn Bedingung bereits erfüllt.
  Richtig wenn Zustand schon eingetreten sein könnte.
```

Entscheidungsregel:
- Zustand könnte bereits zutreffen → `wait_template`
- Zustand soll erst noch eintreten → `wait_for_trigger`
- Bei Timeout: `continue_on_timeout: false` setzen + `wait.timed_out` prüfen
  (Default: `continue_on_timeout: true`!)

### 6.6 Automation per REST (einzeln, ohne gesamte YAML anzufassen)
```bash
# Anlegen/Überschreiben
POST /api/config/automation/config/<automation_id>
Body: {"alias":"...", "triggers":[...], "conditions":[], "actions":[]}
→ {"result":"ok"} HTTP 200
# Hinweis: API akzeptiert auch singular (trigger/condition/action), speichert aber intern
# immer als Plural. GET liefert immer Plural → Plural beim POST bevorzugen.

# Löschen
DELETE /api/config/automation/config/<automation_id>
→ {"result":"ok"} HTTP 200
```

Nach DELETE: Entity ist **sofort weg** (GET → 404) — kein reload für Bereinigung nötig.
`automation/reload` danach nur wenn gesamte `automations.yaml` neu geladen werden soll.
**Kein Vollneustart nötig** (Unterschied zu manueller YAML-Deletion!).

**⚠️ Ghost-Entries (`restored: true`) — gilt für alle Entity-Typen:**
Entities (Automationen, Sensoren, Switches, …) können als State-Eintrag weiterexistieren, obwohl der
Config-Eintrag bereits gelöscht ist. Typische Ursache: Integration-Update mit Entity-ID-Umbenennung,
oder manuelle Config-Löschung ohne anschließenden Neustart.
Erkennungszeichen: `state: unavailable` + Attribut `"restored": true` (via `GET /api/states/<entity_id>`).
Jede `ha_config_remove_*`-Methode schlägt in diesem Fall fehl (400/404) — die Config existiert nicht mehr.
Korrekte Lösung für alle Ghost-Typen: `DELETE /api/states/<entity_id>` (State-Maschine direkt bereinigen).
```bash
# Verifikation vor dem Löschen (Pflicht):
GET /api/states/<entity_id>  → prüfen auf "restored": true

# Löschen:
DELETE /api/states/<entity_id>
→ {"message":"Entity removed."} HTTP 200

# Post-Verifikation:
GET /api/states/<entity_id>  → 404 erwartet
```

**WICHTIG:** `entity_id` wird aus `alias` abgeleitet, nicht aus `<automation_id>` im URL.
```
alias: "Meine Automation" → automation.meine_automation
```

### 6.7 Services
```bash
# Ausführen auch wenn state=off
POST /api/services/automation/trigger
Body: {"entity_id": "automation.meine", "skip_condition": true}

# Deaktivieren (nicht persistent nach Restart)
POST /api/services/automation/turn_off
Body: {"entity_id": "automation.meine", "stop_actions": true}

# Reload (ungültiges YAML → HTTP 200, unverändert — sicher)
POST /api/services/automation/reload
```

### 6.8 Doppeltes value_template im Trigger

Zweites `value_template` überschreibt erstes **lautlos** — Fehler ohne Warnung!
Immer auf genau ein `value_template` pro Trigger achten.

### 6.9 continue_on_error
```yaml
actions:
  - action: light.turn_on
    target:
      entity_id: light.kueche
    continue_on_error: true    # Automation läuft weiter auch bei Fehler
```
Greift bei echten Exceptions (Netzwerk-Timeout, Zigbee-Delivery-Fehler).
Nicht-existierende Entities werden von HA seit jeher still ignoriert (kein Fehler, keine Exception).

### 6.10 Startup-Fehlauslösungen verhindern
```yaml
# template.yaml
template:
  - triggers:
      - trigger: homeassistant
        event: start
    binary_sensor:
      - name: "HA Startphase"
        unique_id: ha_startphase
        state: "true"
        auto_off: 15    # Sekunden nach Start automatisch off

# In Automation:
conditions:
  - condition: state
    entity_id: binary_sensor.ha_startphase
    state: "off"   # "off" = Startphase abgeschlossen
```
Sensor und Condition `state: "off"` funktionieren korrekt (verifiziert 06.03.2026).

---

## 7. Scripts (scripts.yaml)

### 7.1 Architektur

- YAML-only: `/config/scripts.yaml` (Dict-Format). Kein Config-Entry, kein Storage.
- `unique_id` = YAML-Schlüssel. `entity_id` = `script.<schlüssel>`.
- **Kein** `collection.hash` in options.

### 7.2 Blocking vs. Non-Blocking
```yaml
# Blockierend — Automation WARTET bis Script abgeschlossen:
actions:
  - action: script.mein_script
    data:
      nachricht: "Text"

# Nicht-blockierend — Automation läuft SOFORT weiter:
actions:
  - action: script.turn_on
    target:
      entity_id: script.mein_script
    data:
      variables:
        nachricht: "Text"
```

### 7.3 Modi

`single` (default), `restart`, `parallel`, `queued` (mit `max:`).

State: `on`/`off` + `last_triggered`, `mode`, `current`.

Direktaufruf:
```bash
POST /api/services/script/<schlüssel>
Body: {"variablenname": "Wert"}    # Variablen als Top-Level-JSON
```

---

## 8. Szenen (scenes.yaml)

- Array-Format. Kein Config-Entry, kein Storage.
- `entity_id` aus `name` abgeleitet (nicht aus `id:`).
- Scenes **nicht** in `core.entity_registry` eingetragen.
- State: `"unknown"` (nie aktiviert) / ISO-Timestamp (letzte Aktivierung).

### 8.1 Snapshot (Zustand sichern/wiederherstellen)
```yaml
# Sichern (nur In-Memory — geht nach reload/restart verloren):
actions:
  - action: scene.create
    data:
      scene_id: zustand_vor_eingriff
      snapshot_entities:
        - climate.heizung_badezimmer

# Wiederherstellen:
actions:
  - action: scene.turn_on
    target:
      entity_id: scene.zustand_vor_eingriff
```

---

## 9. Helper — Auswahl und Lifecycle

### 9.1 Entscheidungsmatrix: nativer Helper vs. Template-Sensor

| Bedarf | Lösung (bevorzugt) |
|--------|-------------------|
| Summe / Mittelwert mehrerer Sensoren | `min_max` Integration |
| Verbrauchszählung (kWh, Liter) | `utility_meter` Helper |
| Änderungsrate (Watt/s) | `derivative` Integration |
| Grenzwert mit Hysterese | `threshold` Integration |
| Betriebsstunden / Zählungen | `history_stats` Sensor |
| Kumulierter Wert über Zeit | `integration` Integration |
| Binär: Beliebig/Alle EIN | `group` Helper |
| Tageszeit-Muster | `time_of_day` Helper |
| Wochenplan | `schedule` Helper |
| Einfacher Boolean | `input_boolean` |
| Auswahlmenü | `input_select` |
| Zähler | `counter` |
| Countdown | `timer` |

→ Template-Sensor nur wenn kein nativer Helper passt!

### 9.2 input_number — Falle

`initial:` setzt Wert nach jedem HA-Neustart zurück.
**RICHTIG:** `initial:` weglassen → Wert bleibt persistent im Storage.

### 9.3 local_todo Listen

- Platform: `local_todo`. Domain im Config-Entry: `local_todo` (nicht `todo`).
- Datenspeicherung: `/config/.storage/local_todo.<storage_key>.ics` (lazy erstellt).

Services (Referenz per **Name**, nicht per UID!):
```
todo.add_item: item + optional due_date ODER due_datetime (nicht beide → 400)
todo.get_items: ohne status → nur needs_action-Items. Mit ?return_response.
  Antwort: service_response["todo.<entity_id>"]["items"]
todo.update_item / todo.remove_item: Nichtexistenz → HTTP 500 (nicht 404!)
todo.remove_completed_items: keine weiteren Felder
```

Neue Liste anlegen:
```bash
POST /api/config/config_entries/flow
Body: {"handler":"local_todo"}
→ {"todo_list_name":"Mein Einkauf"}
```

**DUE-Datum:** `due_date: "2026-03-15"` → DUE:20260316 (+1 Tag, RFC 5545 exclusive end).

### 9.4 schedule-Helper — Einschränkung (verifiziert)

`schedule`-Helper können **nicht** per Config-Flow (REST) angelegt werden:
```bash
POST /api/config/config_entries/flow
Body: {"handler": "schedule"}
→ HTTP 404 (kein Config-Flow-Handler registriert)
```

**Einzige Wege (verifiziert HA 2026.3.1):**
- HA-UI: Einstellungen → Geräte & Dienste → Helfer → Zeitplan erstellen
- WebSocket-API via `run_python` auf localhost — **funktioniert** (§17.1, §17.4)

`schedule` hat **keinen `config_flow.py`** und keinen REST-Endpoint — nutzt `DictStorageCollectionWebsocket`.
WebSocket via Nabu Casa: HTTP 403.

WS-Commands (verifiziert HA 2026.3.1):
```python
{"id":1, "type":"schedule/list"}
# → result: Liste aller Schedules [{id, name, monday, ...}]

{"id":2, "type":"schedule/create",
 "name":"Mein Zeitplan",
 "monday":[{"from":"07:00","to":"08:00"}],
 "tuesday":[],"wednesday":[],"thursday":[],"friday":[],"saturday":[],"sunday":[]}
# → result: {id, name, monday, ...}  (id = slug aus name, z.B. "mein_zeitplan")

{"id":3, "type":"schedule/delete", "schedule_id":"mein_zeitplan"}
# → success: true
```
Verbindungsaufbau via `run_python` → §17.4.

---
### 9.5 Repairs — verwaiste Einträge per API löschen (verifiziert 11.03.2026)

Jedes Repair-Issue hat einen Flow, der per REST bestätigt werden kann.
**Kein Browser, kein Neustart nötig.**

```python
# Schritt 1: Flow starten
POST /api/repairs/issues/fix
Body: {"handler": "<domain>", "issue_id": "<issue_id>"}
→ {"type": "form", "flow_id": "...", "step_id": "confirm", ...}

# Schritt 2: Flow bestätigen
POST /api/repairs/issues/fix/{flow_id}
Body: {}
→ {"type": "create_entry", ...}   # = Erfolg, Issue gelöscht
```

**Beispiel Battery Notes (verwaiste device_id):**
- `issue_id` = `missing_device_<subentry_id>` (Subentry-ID aus `core.config_entries`)
- Handler = `battery_notes`
- Antworttyp `create_entry` = Issue erfolgreich gelöscht

**issue_id ermitteln:** Subentry-IDs aus `core.config_entries` lesen,
verwaiste device_ids gegen `core.device_registry` abgleichen.

**GET /api/repairs/issues** → HTTP 404 (kein REST-Endpunkt, nur intern/WS).

---


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

## 12. Blueprints — Regeln

- `enabled: false` in Blueprints → HA-Startfehler. Blueprint anlegen, dann via UI deaktivieren.
- `!input` nicht direkt in Templates — erst als Variable exponieren:
```yaml
  variables:
    ziel_entity: !input target_entity
  actions:
    - action: light.turn_on
      target:
        entity_id: "{{ ziel_entity }}"
```
- Für Template-Trigger: `trigger_variables` verwenden:
```yaml
  trigger_variables:
    mein_sensor: !input sensor_entity
  triggers:
    - trigger: template
      value_template: "{{ states(mein_sensor) | float(0) > 25 }}"
```

---

## 13. Better Thermostat — Preset-Muster *(nur relevant wenn Better Thermostat installiert)*

`climate.set_temperature` setzt `preset_mode` auf `none` → Preset-Watchdog kann greifen.

**Lösung:** `number`-Entity des aktiven Presets direkt beschreiben:
```
number.heizung_<RAUM>_preset_<PRESETNAME>
```
Preset-Namen: `away`, `home`, `sleep`, `comfort`, `eco`, `boost`.

### 13.1 Preset-Abbruch-Logik

Für Automationen die einen Vorkonditionierungs-Preset überwachen:
- **Trigger:** `climate.heizung_<RAUM>` Attribut `preset_mode` ändert sich
- **Condition:** Vorkonditionierungs-Boolean ist `on` UND neuer Preset ≠ gespeicherter Preset
- **Action:** Boolean ausschalten (triggert Cleanup-Sequenz)

Muster für Original-Wert sichern/wiederherstellen:
- Vor Eingriff: aktiven Preset + Temperatur in `input_text` speichern
- Nach Eingriff/Abbruch: `input_text`-Werte zurückschreiben, Boolean `off`

---

## 14. Zigbee-Bulbs — Flash-Workaround

### 14.1 Problem (Hardware-Limitation)

Zigbee-Bulbs speichern intern den zuletzt aktiven Zustand (Farbe, Farbtemperatur).
Beim nächsten Einschalten zeigen sie **kurz den gespeicherten Wert**, bevor HA die
gewünschten Parameter setzt → sichtbarer weißer oder falscher Farbblitz.

**Nicht lösbar durch:**
- `transition:`-Parameter (betrifft nur die HA-seitige Kurve, nicht den Gerätespeicher)
- `brightness: 0` vor Ausschalten (kein gültiger Wert für die meisten Bulbs)

### 14.2 Workaround — Gerätespeicher überschreiben

Vor dem Ausschalten die **Zielfarbe bei `brightness: 1`** setzen. Das überschreibt den
internen Speicher der Bulb mit dem gewünschten Startzustand — der nächste Einschaltvorgang
beginnt direkt in der richtigen Farbe.

**Ablaufmuster (Beispiel Nachtmodus Orange):**
```yaml
actions:
  # 1. Zielfarbe bei minimaler Helligkeit setzen → überschreibt Gerätespeicher
  - action: light.turn_on
    target:
      entity_id: light.meine_zigbee_bulb
    data:
      brightness: 1
      color_temp: 500        # oder color_name/hs_color je nach Bulb
  # 2. Kurz warten (Zigbee-Befehl muss ankommen)
  - delay:
      milliseconds: 100
  # 3. Ausschalten — Gerätespeicher enthält jetzt die Zielfarbe
  - action: light.turn_off
    target:
      entity_id: light.meine_zigbee_bulb
```

### 14.3 Hinweise

- `milliseconds: 100` Delay ist ausreichend; bei trägen Geräten auf `200–500ms` erhöhen.
- Funktioniert für `color_temp` und `hs_color`/`rgb_color`.
- Bei Tagmodus (Weiß): gespeicherter Wert stimmt meist bereits — Workaround hauptsächlich
  im Nachtmodus oder bei Farbwechseln relevant.
- Nicht alle Hersteller implementieren das Speicherverhalten identisch — bei abweichendem
  Verhalten Delay erhöhen oder Zielzustand bei `brightness: 2` testen.

---

## 15. Zigbee2MQTT Add-on — Konfigurationszugriff

### 15.1 Shadow-DOM-Problem (KRITISCH)

Das Zigbee2MQTT Add-on rendert seine Konfigurationsoberfläche in einem Shadow-DOM.
Die HA-UI bietet keinen direkten Zugriff auf die Add-on-Konfigurationsdatei.

**Konsequenz:** Lesen und Schreiben der Z2M-Konfiguration muss direkt über die Datei erfolgen:

```
/config/zigbee2mqtt/configuration.yaml
```

Korrekte Zugriffsmethode via `read_file` / `write_file` shell_command (→ §2.3).

Pfad verifiziert auf HA OS Add-on-Installs (HA 2026.3.0, 08.03.2026).
Gilt nur wenn Z2M als Add-on läuft — bei externer Docker-Installation abweichender Pfad möglich.

### 15.2 Geräteumbenennung

`friendly_name` in Z2M setzen + Haken „Übernehme Namen in HA" aktivieren → setzt
Entity-Name **und** Gerätename in HA global.

Gilt nur für Zigbee-Geräte — nicht für andere Integrationen.

## 16. Android Companion App — next_alarm Sensor

### 16.1 Sensor-Verhalten (verifiziert)

- Liefert immer den zeitlich **nächsten** Alarm-Timestamp in UTC (`+00:00`).
- Attribut `Package`: App, die den Alarm registriert hat.
- Aktualisierungsrate: beim Laden oder alle ~15 Minuten (Einstellung „Akku sparen").
- State-Format: ISO-Timestamp, z.B. `2026-03-08T06:00:00+00:00`.

### 16.2 Bekannte Limitierungen

| Limitation | Erklärung |
|---|---|
| Deaktivierter Alarm nicht erkennbar | Sensor unterscheidet nicht zwischen aktivem und deaktiviertem Alarm |
| Kalender verdrängt Wecker | `com.xiaomi.calendar` o.ä. erscheint als nächster Eintrag wenn zeitlich früher |
| Timer ≠ Wecker nicht trennbar | Beide laufen unter `com.android.deskclock` |
| Kurz vor Alarmzeit `unavailable` | Sensor kann unmittelbar vor Auslösung seinen State verlieren |

### 16.3 Best Practice

**Allowlist auf Deskclock beschränken** (Companion App → Sensor-Einstellungen → next_alarm Allowlist):
```
com.android.deskclock
```
→ Kalender-Events und Fremdalarm-Apps werden gefiltert; nur echte Wecker erscheinen.

**Nativen `time`-Trigger verwenden** (nicht `state`-Trigger auf den Sensor):
```yaml
triggers:
  - trigger: time
    at: sensor.<gerät>_next_alarm
    offset: "-00:10:00"   # 10 Minuten vor Wecker
```

**Zusatz-Conditions zum Absichern:**
```yaml
conditions:
  # Nur Deskclock-Wecker (Allowlist allein reicht nicht für alle Geräte)
  - condition: template
    value_template: >
      {{ state_attr('sensor.<gerät>_next_alarm', 'Package') == 'com.android.deskclock' }}
  # Alarm liegt in der Zukunft (verhindert Fehlauslösung nach Neustart)
  - condition: template
    value_template: >
      {{ as_timestamp(states('sensor.<gerät>_next_alarm')) > as_timestamp(now()) }}
```

### 16.4 Typisches Automationsmuster
```yaml
- id: "wecker_vorkonditionierung"
  alias: "Vorkonditionierung 10 min vor Wecker"
  triggers:
    - trigger: time
      at: sensor.<gerät>_next_alarm
      offset: "-00:10:00"
  conditions:
    - condition: template
      value_template: >
        {{ state_attr('sensor.<gerät>_next_alarm', 'Package') == 'com.android.deskclock' }}
    - condition: template
      value_template: >
        {{ as_timestamp(states('sensor.<gerät>_next_alarm')) > as_timestamp(now()) }}
  actions:
    - action: input_boolean.turn_on
      entity_id: input_boolean.vorkonditionierung
  mode: single
```

> `<gerät>` durch den gerätespezifischen Sensor-Slug ersetzen (z.B. `xiaomi_14t_pro_mk`).

---

## 17. WebSocket API

### 17.1 Zugang

WebSocket-API ist **nur lokal** nutzbar:
```
ws://localhost:8123/api/websocket
```
- Auth: `{"type":"auth","access_token":"<LONG_LIVED_TOKEN>"}`
- **Nabu Casa: HTTP 403 für alle WS-Commands** — kein WebSocket-Upgrade-Header-Durchlass.
- Aus Claude-Session: nur via temporärem `run_python` shell_command erreichbar (erfordert Neustart).

→ Temporärer Zugang: §18.3 Fix-Prozess (run_python hinzufügen, Neustart, testen, entfernen).

### 17.2 Nützliche Commands (verifiziert HA 2026.3.1, 08.03.2026)

| Command | Beschreibung | Laufzeit | Besonderheit |
|---------|-------------|----------|--------------|
| `config/entity_registry/list` | Alle Einträge inkl. disabled, area_id + labels (Beispiel: ~1960) | ~39ms | Schneller als `.storage/` lesen |
| `config/entity_registry/list_for_display` | Nur enabled Einträge, kompakte Keys (Beispiel: ~1227) | ~13ms | Schnellster Registry-Zugriff; Keys: `ei`, `pl`, `lb`, `di` |
| `config/area_registry/list` | Areas inkl. temperature_entity_id, humidity_entity_id | — | Mehr Felder als `.storage/` direkt |
| `config/label_registry/list` | Labels inkl. label_id, color, icon | — | — |
| `get_states` | Alle States inkl. context-Feld | ~18ms | Kein Vorteil ggü. `GET /api/states` |
| `call_service` + `return_response: true` | Wie REST `?return_response` | — | **Vorteil:** `target: {area_id/label_id}` möglich (REST kennt kein target:) |
| `subscribe_events` / `unsubscribe_events` | Event-Subscription | — | Für One-Shot-Modell nicht sinnvoll |
| `ping` | Verbindungstest | ~1ms | — |

### 17.3 Nicht nutzbare Commands (defekt in HA 2026.3.1)

| Command | Status | Symptom |
|---------|--------|---------|
| `validate_config` | **DEFEKT** | `invalid_format — extra keys not allowed @ data['trigger']` bei jeder Payload-Variante (Liste, Dict, platform:, trigger:, leer) |
| `extract_from_target` | **NICHT NUTZBAR** | Gibt immer leere Listen zurück; Doku-Keys falsch (`referenced_entities` statt `entity_ids`) — intern für Frontend |

### 17.4 WS-Protokoll-Muster (für run_python-Skripte)

```python
import asyncio, websockets, json

async def ws_call(token, command, payload=None):
    uri = "ws://localhost:8123/api/websocket"
    async with websockets.connect(uri) as ws:
        await ws.recv()                          # auth_required
        await ws.send(json.dumps({"type": "auth", "access_token": token}))
        await ws.recv()                          # auth_ok

        req = {"id": 1, "type": command}
        if payload:
            req.update(payload)
        await ws.send(json.dumps(req))
        return json.loads(await ws.recv())

# Beispiel:
result = asyncio.run(ws_call(TOKEN, "config/entity_registry/list_for_display"))
entities = result["result"]   # Liste mit Einträgen {ei, pl, lb, di, ...}
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

## 20. Bekannte Anti-Patterns (Schnellreferenz)

| Anti-Pattern | Lösung | Warum |
|---|---|---|
| `condition: template` mit `float > 25` | `condition: numeric_state` | Validiert bei Load, nicht Runtime |
| `wait_template` für „erst noch eintreten" | `wait_for_trigger` | Semantik verschieden |
| `device_id` in Triggers | `entity_id` (oder `device_ieee` für ZHA) | device_id bricht bei Re-Add |
| `mode: single` für Bewegungslicht | `mode: restart` | Re-Trigger muss Timer resetten |
| Template-Sensor für Summe/Mittel | `min_max` Helper | Deklarativ, handled unavailable |
| Template-Sensor mit Schwellwert | `threshold` Helper | Eingebaute Hysterese |
| `initial:` bei input_number | weglassen | Setzt Wert nach Restart zurück |
| `target:` im REST-Body | entity_id top-level | `target:` → HTTP 400 |
| `/api/config/config_entries/<id>` DELETE | `/entry/<id>` | Ohne `/entry/` → 404 |
| Sensor ans Dateiende in template.yaml | expliziter `- sensor:`-Block | Block-Falle → stumm verworfen |
| New storage-Eintrag ohne Vorlage | Vorlage-Eintrag 1:1 lesen | Pflichtfelder-Inkonsistenz je Datei |
| Storage reload statt Restart | Vollneustart | reload überschreibt Storage |
| `datetime.utcnow().isoformat()` | `datetime.now(timezone.utc).isoformat()` | kein Timezone-Info (deprecated) |
| `collection.hash` in entity_registry setzen | weglassen | Optional, Formel unbekannt |
| `curl ... \| python3 << 'HEREDOC'` | Zwischendatei + `python3 << 'HEREDOC'` | Heredoc verdrängt Pipe auf stdin |
| `shell_command` ändern + `reload_all` | HA-Vollneustart | Kein reload-Service für shell_command |
| REST Automation-POST mit `trigger:[...]` | `triggers:[...]` (Plural) | GET liefert immer Plural — inkonsistent |
| `schedule`-Helper via REST anlegen | WS `schedule/create` via `run_python` (§9.4) | Kein config_flow, kein REST-Endpoint |
| Anker mit 
 am EOF der Datei | Anker aus `repr()`-Output kopieren (EOF hat oft kein trailing 
) | `str.replace()` schlägt lautlos fehl |
| LAG()-Window-Funktion in SQL-Diagnose | Subquery-Variante §18.2 | Zu viele Zeilen → JSONDecodeError "Extra data" |
| `unit_of_measurement` ohne `state_class` | `state_class` + `device_class` ergänzen | Kein Eintrag in statistics_meta → keine Langzeit-Statistiken (verifiziert 08.03.2026) |
| Template-Sensor mit `platform: integration` in template.yaml | In `sensor.yaml` | template.yaml kennt keine `platform:`-Einträge |
| GitHub-PR via MCP direkt mergen | Erst „Ready" setzen (MCP erstellt immer Draft) | Seit ha-mcp v7.0.0 stille Verhaltensänderung |
| CLAUDE.md per computer.type / cat-Heredoc schreiben | §2.7 atomarer Zyklus (read → modify → write_file) | Lautloser Datenverlust oder Teilüberschreibung |
| `write_file`/`delete_file` mit absolutem Pfad (`/config/datei`) | Relativen Pfad verwenden (`datei`) | Doppel-Slash `/config//config/...` → falsche Datei oder Fehler |
| Kanalnachricht blind umsetzen ohne Gegencheck | Behauptungen live verifizieren (§0 Kanal-Verifikationspflicht) | Fehler der Gegenstelle pflanzen sich fort |

---

## 21. Analyse-Reports — Struktur
```
1. Executive Summary (2–3 Sätze: Gesamtzustand + wichtigste Befunde)
2. Kritische Probleme (sofortiger Handlungsbedarf)
3. Warnungen (beobachten)
4. Systemgesundheit (was funktioniert gut)
5. Automations-Insights (Muster, Auffälligkeiten)
6. Klimaanalyse (Temperaturen, Effizienz)
7. Nutzerverhalten (manuelle Eingriffe → Automationspotenzial)
8. Empfehlungen (konkrete nächste Schritte)
```

Qualitätsmaßstab:
- SCHLECHT: „47 Automationen ausgelöst" — keine Aussage
- GUT: „Bewegungssensor Küche 47x ausgelöst, Automation nur 2x — Conditions prüfen"
- SCHLECHT: „Keine Fehler gefunden" — ohne tatsächliche Prüfung
- GUT: Logbook + History abfragen, dann urteilen

### 21.1 Analyse-Leitfragen

- Muster über Zeit: Normal oder ungewöhnlich?
- Korrelationen: Hängen Ereignisse zusammen?
- Effizienz: Greifen Automationen wie erwartet?
- Zuverlässigkeit: Flackernde Geräte oder Integrationen?
- User Experience: Was verursacht manuelle Eingriffe?
- Energie: Heizungseffizienz vs. Belegung?
- Sicherheit: Unerwartete Bewegungs-/Türmuster?

---
---

## 22. CLAUDE.md — Empfohlene Struktur (Template)

Jede Instanz pflegt eine eigene `CLAUDE.md` mit **ausschließlich instanzspezifischem** Wissen.
Instanzunabhängiges Wissen (API-Syntax, YAML-Regeln, Fallen) gehört in diesen Skill — nie in `claude.md`.

### 22.1 Pflicht-Abschnitte (Reihenfolge einhalten)

```
## ① Skills — IMMER zuerst laden
Ladebefehl mit bust-Cache + Skill-Name. Keine Versionsnummer nötig (bust zieht immer aktuell).

## ② ha-mcp — Primärzugriff
Connector-Name (z.B. LB, RBO) + Endpoint-URL + Tool-Anzahl + Verifikationsdatum.
Regel: Automationen, Helfer, Entities IMMER live abrufen — nie aus claude.md.

## ③ System & Config-Architektur
Timezone/Standort. Tabelle: Datei → Inhalt/Besonderheit.
Instanzspezifische Hinweise zu template.yaml-Struktur (letzter Block, Fallstricke).

## ④ IoT-Stack
Hardware-Übersicht: Zigbee-Broker (IP:Port), BT-Proxies, Cloud-Integrationen, Steckdosen/Schalter.

## ⑤ Business-Logik
Automations-Cluster mit Querverweisen auf Skill-Sektionen (z.B. → Skill §13, → Skill §14).
Nur: Entity-IDs, Helfer-Namen, Logik-Beschreibung — keine YAML-Blöcke.

## ⑥ Recorder
Konfigurationsübersicht (purge_keep_days, exclude-Strategie, Sonder-Purge-Automationen).

## ⑦ Offene Punkte (Stand <Datum>)
Nummerierte Liste. Nach Erledigung entfernen — nie als "erledigt" markiert stehen lassen.
```

### 22.2 Regeln

- **Ziel:** < 150 Zeilen, angestrebt < 4.000 Zeichen. Über diesem Limit: Inhalte in Skill oder live-Abruf auslagern.
- **Keine konkreten Werte** die sich ändern können (Temperaturschwellen, Zeitpläne) → live via ha-mcp.
- **Keine YAML-Blöcke** — die gehören in `automations.yaml`, nicht in `claude.md`.
- **Credentials ausschließlich in `secrets.yaml`** — nie in `claude.md`.
- **CLAUDE.md-Änderungen ausschließlich via §2.7 atomarer Lese-Modifizier-Schreib-Zyklus.**
  Nie per `computer.type`, `cat`-Heredoc oder direktem String-Überschreiben ohne vorherigen read-Schritt.
  Konsequenz bei Missachtung: lautloser Datenverlust oder inkompletter Schreibvorgang.
- **Skill-Querverweise** statt Inhaltskopie: `→ Skill §4.1` statt den Regeltext zu wiederholen.
- **Integrations-spezifische Abschnitte** (z.B. Better Thermostat, Android next_alarm) nur wenn die Integration auf dieser Instanz installiert ist.

---

## 23. Verifikation nach Änderungen

Nach jedem nicht-trivialen Schritt den passenden API-Call wählen:

| Kontext | Verifikation |
|---|---|
| Automation geändert | `GET /api/config/automation/<id>` + Logbuch/Trace prüfen |
| Entity-State erwartet | `GET /api/states/<entity_id>` |
| Logbuch-Check | `GET /api/logbook/<iso_timestamp>` |
| Shell-Command ausgeführt | `rc=0` prüfen + stdout auf Inhalt validieren |
| Template-Sensor neu/geändert | `POST /api/template` mit Ausdruck direkt testen |
| Helper-Wert gesetzt | `GET /api/states/<helper_entity_id>` |

---

## 24. Telegram — notify-Service (verifiziert 09.03.2026, LB + RBO)

### 24.1 Service-Aufruf
```bash
POST /api/services/telegram_bot/send_message?return_response
{
  "config_entry_id": "[TELEGRAM_ENTRY_ID]",
  "chat_id": "[TELEGRAM_CHAT_ID]",
  "message": "Text",
  "parse_mode": "markdownv2"
}
```

**Parameter:**
- `config_entry_id` → Pflicht (instanzspezifisch im SI/AW)
- `chat_id` → optional; ohne chat_id → alle konfigurierten Chats
- `parse_mode` → Top-Level-Parameter (NICHT in `data{}` verschachtelt)
- `entity_id` → funktioniert **nicht** (HTTP 500)

Service-Domain: `telegram_bot` (nicht `notify`) — live prüfen via `GET /api/services`.

### 24.2 parse_mode markdownv2 — Sonderzeichen escapen (PFLICHT)

Folgende Zeichen **müssen** mit `\\` escaped werden:
`_ * [ ] ( ) ~ \` > # + - = | { } . !`

**Falsch:** `"Temperatur: 21.5°C (Wohnzimmer) - Test #1"`
**Richtig:** `"Temperatur: 21\\.5°C \\(Wohnzimmer\\) \\- Test \\#1"`

**Python-Hilfsfunktion:**
```python
import re
def tg_escape(text):
    return re.sub(r'([_*\[\]()~`>#+\-=|{}.!\\\\])', r'\\\\\1', text)
```
