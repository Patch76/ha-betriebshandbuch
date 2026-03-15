# Automationen, Scripts, Szenen, Helper, Blueprints

Teil des ha-betriebshandbuch — Automationen, Scripts, Szenen, Helper, Blueprints.
Nachladen: `references/automations.md`

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


**⚠️ Ghost-Update-Entity (orphaned Supervisor-Add-on-Repository):**
Ursache: Add-on deinstalliert, aber das zugehörige Supervisor-Repository noch registriert.
Symptom: Update-Popup zeigt "Entity not found"; `GET /api/states/update.<n>_update` → 404;
Neustart und "Überspringen" ohne Wirkung. Kein `restored: true` — Eintrag nur in `core.restore_state`.
Abgrenzung zu §6.6: Entity hat keinen Live-State → `DELETE /api/states/` schlägt fehl (404, erwartet).
```bash
# Diagnose:
# 1. Slug aus entity_picture-Attribut in core.restore_state ermitteln:
#    /api/hassio/addons/<slug>_<n>/icon  → Slug = <slug>
#    (nur Supervisor-Add-ons; HACS-Entities haben kein entity_picture mit Slug)
# 2. Bestätigen:
ha_get_addon(source="available", query="<n>")  # → Repo-Slug sichtbar, addons nicht leer

# Fix (SSH-Terminal):
ha store delete <slug>   # nur Custom-Repos; Built-ins nicht löschbar

# Post-Check:
ha_get_addon(source="available", query="<n>")  # → addons: []
ha_get_updates()                                   # → 0 offene Updates
```
Kein HA-Neustart erforderlich. Stale-Eintrag in core.restore_state entfällt beim nächsten HA-Start automatisch.


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

