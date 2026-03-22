# YAML-Konfiguration & Template-Sensoren

Teil des ha-betriebshandbuch — YAML-Konfiguration, Template-Sensoren.
Nachladen: `references/yaml-templates.md`

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
      state_class: measurement                 # EMPFOHLEN wenn Long-Term-Statistiken gewünscht
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

**state_class — optional, aber empfohlen für numerische Sensoren (verifiziert LB, HA 2026.3):**
`state_class` ist **kein** Pflichtfeld — es ist opt-in für Long-Term-Statistiken.
Ohne `state_class` schreibt HA keine Langzeit-Statistiken (kein Eintrag in `statistics_meta`).
Nicht setzen bei Diagnostic- oder One-Shot-Sensoren (z.B. `unit_of_measurement: "ms"` ohne Statistik-Bedarf).
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
   # Schritt 1: Flow initiieren
   POST /api/config/config_entries/flow
   Body: {"handler": "template", "show_advanced_options": false}
   → Antwort: {"flow_id": "...", "type": "menu", "menu_options": ["sensor", ...]}

   # Schritt 2: Typ wählen
   POST /api/config/config_entries/flow/{flow_id}
   Body: {"next_step_id": "sensor"}

   # Schritt 3: Formular ausfüllen
   POST /api/config/config_entries/flow/{flow_id}
   Body: {"name": "Mein Sensor", "state": "{{ states('sensor.quelle') }}"}
```
   Sofort live. Löschen: `DELETE /api/config/config_entries/entry/<entry_id>` → 200 OK.
   Abbrechen: `DELETE /api/config/config_entries/flow/{flow_id}` (verifiziert LB 22.03.2026).

### 4.5 Numerische Template-Sensoren

State-Template **muss** eine Zahl oder `none` zurückgeben (kein String wie `"unknown"`),
wenn `state_class` oder `unit_of_measurement` gesetzt ist:
```yaml
state: >
  {% set val = states('sensor.quelle') | float(-1) %}
  {{ val if val >= 0 else none }}
```

---

