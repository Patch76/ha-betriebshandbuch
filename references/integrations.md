# Integrationen (BT, Zigbee, Shelly, Android, Telegram)

Teil des ha-betriebshandbuch — BT, Zigbee, Shelly, Android, Telegram.
Nachladen: `references/integrations.md`

## 13. Better Thermostat — Preset-Muster *(nur relevant wenn Better Thermostat installiert)*

`climate.set_temperature` setzt `preset_mode` auf `none` → Preset-Watchdog kann greifen.

**Lösung:** `number`-Entity des aktiven Presets direkt beschreiben:
```
number.heizung_<RAUM>_preset_<PRESETNAME>
```
Preset-Namen: `away`, `home`, `sleep`, `comfort`, `eco`, `boost`, `activity`.

### 13.1 Preset-Abbruch-Logik

Für Automationen die einen Vorkonditionierungs-Preset überwachen:
- **Trigger:** `climate.heizung_<RAUM>` Attribut `preset_mode` ändert sich
- **Condition:** Vorkonditionierungs-Boolean ist `on` UND neuer Preset ≠ gespeicherter Preset
- **Action:** Boolean ausschalten (triggert Cleanup-Sequenz)

Muster für Original-Wert sichern/wiederherstellen:
- Vor Eingriff: aktiven Preset + Temperatur in `input_text` speichern
- Nach Eingriff/Abbruch: `input_text`-Werte zurückschreiben, Boolean `off`

**⚠️ KRITISCH — Reihenfolge in Abbruch-Automation (verifiziert 14.03.2026):**
Der Restore-Schritt (Preset-Wert zurückschreiben) MUSS **vor** dem Leeren der Helper erfolgen.
Werden die `input_text`-Helper zuerst geleert, ist der ursprüngliche Wert verloren → erhöhter Preset bleibt dauerhaft stehen.

Korrekte Reihenfolge:
1. `number.set_value` → Preset-Wert aus `input_text.vorherige_temperatur` auf `input_text.vorheriger_preset` zurückschreiben
2. `input_boolean.turn_off`
3. `input_text.set_value` → vorheriger_preset löschen
4. `input_text.set_value` → vorherige_temperatur löschen

Anti-Pattern (falsch):
1. `input_boolean.turn_off`
2. Helper leeren ← Wert weg, Restore nicht mehr möglich
3. (kein Restore-Schritt)

### 13.2 Kalibrierungsmodi — Übersicht (verifiziert 15.03.2026)

BT Options Flow ist **zweistufig** und vollständig via REST-API durchführbar (kein Storage-Eingriff, kein HA-Neustart):

- **Schritt 1** (`step_id: user`): Sensoren + Basiseinstellungen
- **Schritt 2** (`step_id: advanced`): Kalibrierung + Schutzfunktionen

⚠️ Schritt 2 enthält **zwei separate Felder** — beide müssen explizit gesetzt werden:
- `calibration` — Strategie: `local_calibration_based` | `target_temp_based` | `hybrid_calibration`
- `calibration_mode` — Algorithmus: `default` | `fix_calibration` | `heating_power_calibration` | `no_calibration`

Wird nur `calibration` gesetzt, schlägt der Flow mit Validierungsfehler fehl.

**Hängenden Options-Flow abbrechen:**
Ein gestarteter, nicht abgeschlossener Flow blockiert jeden neuen Options-Flow für dieselbe Config-Entry.
Abbruch via REST:
```bash
DELETE /api/config/config_entries/options/flow/<flow_id>
→ 200 OK (Flow entfernt, Entry wieder verfügbar)
```
`flow_id` aus dem initialen `POST`-Response entnehmen oder via:
```bash
GET /api/config/config_entries/options/flow  → Liste aller aktiven Flows
```

**`data` vs `options` — je nach Integration:**
Nicht alle Integrationen speichern Konfiguration in `data`. `SchemaConfigFlowHandler`-Integrationen
(Threshold, Generic Thermostat, Generic Hygrostat, Min/Max) speichern alles in `options`, nicht `data`.
Prüfung: `entry.get('data', {})` liefert bei diesen Integrationen `{}` — stattdessen `entry.get('options', {})` nutzen.

**REST-Muster (Schritt 2 — Advanced):**
```bash
POST /api/config/config_entries/options/flow/<flow_id>
{
  "calibration": "local_calibration_based",
  "calibration_mode": "default",
  "protect_overheating": true,
  "no_off_system_mode": true,
  "heat_auto_swapped": false,
  "child_lock": false,
  "homematicip": false
}
```

### 13.3 heating_power_calibration — Risiko bei saisonal abgeschalteter Anlage (verifiziert 15.03.2026)

**⚠️ WARNUNG:** `calibration_mode: heating_power_calibration` (= „AI Time Based") ist für **kontinuierlichen Betrieb** ausgelegt.

**Problem:** Läuft BT weiter, während die Zentralheizung physisch abgeschaltet ist (kein Wärmeträger), lernt der Algorithmus falsche Offsets:
- BT sendet Call-for-Heat → TRV öffnet → keine Wärme → Sensor steigt nicht
- Algorithmus wertet: „Heizleistung zu gering" → Offset erhöhen → nächster Zyklus aggressiver
- **Positiver Rückkopplungskreis:** Offset driftet über Wochen ins Extreme

**Typische Symptome:**
- Offset auf +5 bis +10 gedriftet → TRV heizt physisch stundenlang, auch wenn BT `action=idle` meldet
- Offset auf −10 gedriftet (z.B. durch Sonneneinstrahlung ohne Heizung) → TRV kaum offen, Solltemperatur nicht erreichbar
- `call_for_heat = True` dauerhaft bei Raumtemperatur weit über Soll

**Diagnose:** `number.<RAUM>_temperaturoffset` State prüfen.
Historik ist oft leer — Recorder schließt `number.*` typischerweise aus (kein Fehler, erwartetes Verhalten).

**Fix:**
1. `calibration_mode` → `default` (via Options Flow → §13.2)
2. `protect_overheating` → `true`
3. Offset manuell auf 0 zurücksetzen: `number.set_value` auf betroffene Entity

**Empfehlung für saisonal genutzte Objekte:** `calibration_mode: default` statt `heating_power_calibration` verwenden.


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


### 14.4 Geräteprofil: EGLO Saliteras-Z (AwoX EGLO_ZM_RGB_TW)

**Modell:** EGLO 900024 / 12253 | Firmware: 2.3.12_250 | LB-Entity: `light.kueche_decke`

**Protokolltrennung (KRITISCH):**
- Hauptlicht: gesteuert via Zigbee (Z2M)
- Backlight (RGB-Ring): gesteuert via **Bluetooth** (AwoX Mesh, App-Only)
- Z2M-State ändert sich **nicht** wenn die App das Backlight schaltet — beide Protokolle sind vollständig unabhängig
- `color_temp_startup` greift nur wenn der letzte Z2M-State ebenfalls `color_temp` war

**Verifizierte Verhaltensregeln (16.03.2026, LB):**
- Backlight via Zigbee steuern: **nicht möglich** (kein Expose in Z2M, Cluster 65360 proprietär)
- `brightness: 0` oder `brightness: 1` schaltet Hauptlicht **nicht aus** (Firmware-Mindesthelligkeit)
- Nur via `state: OFF` kann alles (inkl. Backlight) ausgeschaltet werden
- `fsaris/home-assistant-awox` unterstützt `.ble.zigbee.light.*` explizit nicht

**Workaround Backlight deaktivieren:**
1. App öffnen → Warmweiß antippen → Backlight aus, Zigbee-State unverändert
2. Danach nur noch `color_temp` via Zigbee → Backlight bleibt dauerhaft aus (BT-Flash-Speicher)

**Kalibrierte Tageswerte (LB Küche):**
- `color_temp: 290` | `brightness: 220` → mittleres Tageslicht, kein Backlight

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

### 15.3 Debug-Logging — SOFORT zurücksetzen nach Tests

`log_level: debug` erzeugt erhebliches Log-Volumen und **muss direkt nach Abschluss von Tests
zurückgesetzt werden** — nicht auf Session-Ende warten.

**Zurücksetzen via MQTT:**
```
Topic:   zigbee2mqtt/bridge/request/options
Payload: {"options":{"advanced":{"log_level":"info"}}}
```

Normalzustand: `log_level: info`


### 15.4 MQTT Explorer — Debugging-Tool

MQTT Explorer läuft **nicht dauerhaft** — er muss vor jeder Debug-Session manuell gestartet werden.
Gilt für beide Instanzen (LB und RBO).

| Instanz | URL |
|---------|-----|
| LB | https://fobaqcy4uh4vovo8dtyvkqxq5uyh7fmj.ui.nabu.casa/app/9cf1ea8f_mqtt_explorer |
| RBO | Im RBO Add-on-Panel |

**Zugangsdaten:** User: `mqtt` · Passwort: `local`


## 16. Shelly — Button-Modus mit Kippschalter (verifiziert 15.03.2026, LB)

### 16.1 Event-Typen im Button-Modus (Gen2/Gen3)

Im Button-Modus (Input Mode = Button/Taster) sendet Shelly Gen2/Gen3 folgende `click_type`-Werte
als `shelly.click`-Events in HA:

| click_type | Auslöser | Kippschalter? | Taster? |
|---|---|---|---|
| `btn_down` | Kontakt schließt | ✅ eine Richtung | ✅ Drücken |
| `btn_up` | Kontakt öffnet | ✅ andere Richtung | ✅ Loslassen |
| `single_push` | Kurzer vollständiger Drück-Loslassen-Zyklus | ❌ **nie** | ✅ |
| `double_push` | Zwei schnelle Klicks | ❌ **nie** | ✅ |
| `long_push` | Längeres Halten | ⚠️ nur bei sehr langem Halten | ✅ |

**KRITISCH — Kippschalter im Button-Modus:**
- Kippschalter senden **kein** `single_push` — nur `btn_down` (eine Richtung) und `btn_up` (andere Richtung)
- Jede Flip-Richtung = **genau ein** Event
- Für Toggle-Funktion: **beide** Events als Trigger verwenden (`btn_down` + `btn_up`)
- `single_push` als alleiniger Trigger funktioniert mit Kippschalter **nicht**

### 16.2 Korrekte Automation für Kippschalter + Toggle (verifiziert LB 15.03.2026)

```yaml
trigger:
  - trigger: event
    event_type: shelly.click
    event_data:
      device_id: <shelly_device_id>   # device_id aus HA-Device-Registry
      click_type: btn_down
  - trigger: event
    event_type: shelly.click
    event_data:
      device_id: <shelly_device_id>
      click_type: btn_up
action:
  - action: light.toggle
    target:
      entity_id: light.ziel_entity
mode: single
```

### 16.3 Detached Switch + Button-Modus (Gen3)

Kombination für smart bulbs:
- **Output/Relais:** Dauerhaft EIN → Strom immer an der Lampe
- **Input:** Button-Modus + Getrennter Schalter (Detached) → Relais reagiert nicht physisch
- **HA:** Event-Entity `event.<device_name>` wird automatisch erstellt (Input Mode = Button)
- **Binary Sensor** (`binary_sensor.<device>_input_0_input`) wird deaktiviert (`disabled_by: integration`) — korrekt so

**Wichtig:** Input Mode = Switch + Detached → weder Event-Entity noch Binary Sensor nutzbar.
Input Mode muss auf **Button/Taster** stehen damit Event-Entity entsteht.

### 16.4 Anti-Pattern (verifiziert LB 15.03.2026)

| Falsch | Richtig | Grund |
|---|---|---|
| `click_type: single_push` bei Kippschalter | `btn_down` + `btn_up` | Kippschalter sendet kein single_push |
| Nur `btn_up` als Trigger | Beide `btn_down` + `btn_up` | Nur eine Richtung wird getriggert |
| State-Trigger auf `event.*.event_type` mit `to:` Filter | shelly.click Event-Trigger | Bei identischem Folgeevent keine Auslösung (state ändert sich nicht) |
| Input Mode = Switch + Detached für HA-Automation | Input Mode = Button + Detached | Switch+Detached erstellt kein auswertbares HA-Entity |


## 26. Android Companion App — next_alarm Sensor

### 26.1 Sensor-Verhalten (verifiziert)

- Liefert immer den zeitlich **nächsten** Alarm-Timestamp in UTC (`+00:00`).
- Attribut `Package`: App, die den Alarm registriert hat.
- Aktualisierungsrate: beim Laden oder alle ~15 Minuten (Einstellung „Akku sparen").
- State-Format: ISO-Timestamp, z.B. `2026-03-08T06:00:00+00:00`.

### 26.2 Bekannte Limitierungen

| Limitation | Erklärung |
|---|---|
| Deaktivierter Alarm nicht erkennbar | Sensor unterscheidet nicht zwischen aktivem und deaktiviertem Alarm |
| Kalender verdrängt Wecker | `com.xiaomi.calendar` o.ä. erscheint als nächster Eintrag wenn zeitlich früher |
| Timer ≠ Wecker nicht trennbar | Beide laufen unter `com.android.deskclock` |
| Ganztägige Kalendertermine verdrängen Wecker | Unterscheidung nicht möglich — sensor meldet immer den zeitlich nächsten Eintrag |

> **Praxisempfehlung (verifiziert RBO 14.03.2026):** Der next_alarm-Sensor ist zu unzuverlässig für produktive Automationen. Bei vorhandenem Bett-Drucksensor: Fallback via Sensor-Verlassen-Erkennung statt Wecker-Trigger. `automation.bad_vorkonditionieren_wecker` auf RBO gelöscht (14.03.2026).
| Kurz vor Alarmzeit `unavailable` | Sensor kann unmittelbar vor Auslösung seinen State verlieren |

> **`disabled_by: integration`:** Die Companion-App-Integration deaktiviert den Sensor standardmäßig.
> Aktivierung via REST:
> ```bash
> PUT /api/config/entity_registry/entry/<entry_id>
> Body: {"disabled_by": null}
> ```
> Oder: `ha_set_entity(entity_id="sensor.<gerät>_next_alarm", disabled_by=null)`.
> Nach Aktivierung: 60-Sekunden-Wartezeit, dann HA-Reload der Companion-App-Integration.
> `sensor.<gerät>_last_update_trigger` ist ebenfalls `disabled_by: integration` — zusammen aktivieren.

### 26.3 Best Practice

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

### 24.3 `target:`-Parameter — Deprecated seit 2026.3 (Deadline: 2026.9)

⚠️ Der Parameter `target:` in `telegram_bot.send_message` ist seit **HA 2026.3** deprecated und wird in **2026.9** entfernt.

**Korrekte Syntax (bereits in §24.1 dokumentiert):**
```yaml
action: telegram_bot.send_message
data:
  config_entry_id: "..."
  chat_id: -1002785392751   # ← korrekt, zukunftssicher
  message: "Text"
```

**Nicht mehr verwenden:**
```yaml
data:
  target:                   # ← deprecated seit 2026.3, entfällt in 2026.9
    - -1002785392751
```

**Status RBO:** Alle Automationen auf Companion App migriert (12.03.2026). Script `nachtruhe_anfrage_senden` (letztes Telegram-Relikt) gelöscht — kein Telegram-Aufruf mehr aktiv.

---


## 25. Android Companion App — Notify-Service (verifiziert 12.03.2026, RBO)

### 25.1 Service-Aufruf

```yaml
action: notify.mobile_app_<geräte_slug>
data:
  title: "🔔 Titel"
  message: "Text"
```

Kein `parse_mode`, kein Escaping nötig — plain text funktioniert direkt.

### 25.2 Actionable Notifications (Kat B — Kann-Eingriff)

```yaml
action: notify.mobile_app_xiaomi_14t_pro_mk
data:
  title: "🌙 Nachtruhe aktiv"
  message: "Lichter aus, Heizung auf Sleep."
  data:
    actions:
      - action: "nachtruhe_beenden"
        title: "☀️ Beenden"
```

Anschließend `wait_for_trigger` auf `mobile_app_notification_action`:

```yaml
- wait_for_trigger:
    - trigger: event
      event_type: mobile_app_notification_action
      event_data:
        action: "nachtruhe_beenden"
  timeout: "08:00:00"
  continue_on_timeout: true
- if:
    - condition: template
      value_template: "{{ wait.trigger is not none }}"
  then: [Eingriff-Pfad]
  else: [Timeout-Pfad]
```

**Wichtig:** Action-String muss exakt übereinstimmen (case-sensitive). Timeout + `continue_on_timeout: true` immer setzen.

**Android-Verhalten:**
- `title:` → immer fett, vollständig sichtbar (collapsed + expanded)
- `message:` → collapsed: nur 1 Zeile, expanded: vollständiger Text
- Action-Buttons → nur im expanded state sichtbar
- HTML (`<b>`, `<i>`, `<br>`) → nur im expanded state, sonst plain text

### 25.3 Template-Best-Practice: `replace('_', ' ')`

Bei dynamischen Entity-Namen in `message:` immer `replace('_', ' ')` anwenden — entity_ids wirken sonst unnatürlich (z. B. „Echte_Bewegung"):

```yaml
# Richtig
message: "{{ (trigger.to_state.attributes.friendly_name | default(trigger.entity_id)) | replace('_', ' ') }}"

# Falsch
message: "{{ trigger.to_state.attributes.friendly_name | default(trigger.entity_id) }}"
```

Gilt auch für alle anderen Template-Variablen, die entity_ids oder interne Bezeichner enthalten können.

### 25.4 Migration Telegram → Companion App (RBO, 12.03.2026)

Alle 18 Telegram-Automationen auf RBO auf `notify.mobile_app_xiaomi_14t_pro_mk` migriert:
- `telegram_bot.send_message` → `notify.mobile_app_xiaomi_14t_pro_mk`
- `parse_mode` + Escaping entfällt
- `inline_keyboard` → `wait_for_trigger` + `data.actions`
- `automation.abwesenheit_telegram_steuerung` (Callback-Handler) gelöscht
- Backup vor Migration: `57bd9c46`, nach Migration: `00173094`
