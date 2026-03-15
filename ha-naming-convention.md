# Home Assistant – Naming Convention (LB & RBO)

Version: 1.0.0 | Stand: 15.03.2026 | Basis: LB-Vollmigration ~150 Entities

---

## Grundprinzip

Der `entity_id` beantwortet genau zwei Fragen:
1. **Wo?** → Raumname (ausgeschrieben, keine Umlaute)
2. **Was?** → Funktion/Typ

Alles andere (Stockwerk, Kategorie, Gerätemarke, Saisonalität) gehört in **Area / Floor / Label**, nicht in den Namen.

---

## Struktur

```
<domain>.<ort>_<funktion>[_<geraetetyp>][_<nummer>]
```

| Segment | Pflicht | Beschreibung |
|---|---|---|
| `domain` | ✅ | HA-Domain (light, sensor, switch, …) — von HA vergeben |
| `ort` | ✅ | Raum aus Glossar (→ Abschnitt Räume) |
| `funktion` | ✅ | Was die Entity tut oder misst |
| `geraetetyp` | ☐ | Nur zur Unterscheidung wenn nötig (z. B. `schalter`, `steckdose`) |
| `nummer` | ☐ | Nur bei echten Duplikaten gleicher Funktion im selben Raum (`_2`, `_3`) |

### Regeln

- Nur Kleinbuchstaben, Ziffern, Unterstriche
- Keine Umlaute: `ue`, `ae`, `oe`, `ss` statt `ü`, `ä`, `ö`, `ß`
- Keine Gerätemarken, Modellnummern, MAC-Adressen im Namen
- Kein Instanz-Prefix (`lb_`, `rbo_`) — die Instanz ergibt sich aus der HA-Instanz selbst
- Keine Redundanz mit der Domain (`light.kueche_licht` → ❌ `licht` doppelt → ✅ `light.kueche_decke`)

---

## Räume-Glossar (gemeinsamer Standard)

Das folgende Glossar definiert die kanonischen Schreibweisen für Räume, die in beiden Instanzen vorkommen können. Jede Instanz verwendet nur die Räume, die physisch vorhanden sind.

| entity_id-Schreibweise | Anzeigename (Bereich) |
|---|---|
| `wohnzimmer` | Wohnzimmer |
| `kueche` | Küche |
| `schlafzimmer` | Schlafzimmer |
| `badezimmer` | Badezimmer |
| `flur` | Flur |
| `vorzimmer` | Vorzimmer |
| `buero` | Büro |
| `keller` | Keller |
| `dachboden` | Dachboden |
| `schuppen` | Schuppen |
| `garten` | Garten |
| `vorgarten` | Vorgarten |
| `aussen` | Außen / Fassade |
| `einfahrt` | Einfahrt |
| `terrasse` | Terrasse |
| `carport` | Carport |
| `garage` | Garage |
| `haustechnik` | Haustechnik (Heizung, USV, Netzwerk, etc.) |
| `global` | Kein Raumbezug (instanzweite Helfer, Zustände, Automationen) |

**Instanzspezifische Räume:** LB und RBO können Räume haben, die in der anderen Instanz nicht existieren. Diese werden mit demselben Namensschema in der jeweiligen `claude.md` dokumentiert — das Glossar hier bleibt der gemeinsame Stamm.

---

## Instanz-Koordination LB ↔ RBO

Beide Instanzen verwenden dieselbe Konvention. Das ermöglicht:
- Automationen und Konfigurationen ohne Umbenennung zwischen Instanzen zu übertragen
- Kanal-Kommunikation (`lb_to_rbo.md`, `rbo_to_lb.md`) ohne Übersetzung von entity_ids
- Gemeinsame Debugging-Erfahrung: Wer eine Instanz kennt, findet sich in der anderen sofort zurecht

**Divergenz-Regel:** Wenn eine Instanz einen Raum oder eine Funktion hat, die die andere nicht kennt, wird das Schema trotzdem eingehalten. Es wird **nicht** ein Instanz-Prefix ergänzt — der Kontext ergibt sich aus der HA-Instanz selbst.

**Synchronisation:** Änderungen am gemeinsamen Glossar (neue Raumbezeichnungen, neue Konventionen) werden über den Kanal kommuniziert und in beiden Instanzen umgesetzt.

---

## Sonderfälle

### Globale Helfer (kein Raumbezug)

```
input_boolean.global_abwesenheit
input_boolean.global_wintermodus
counter.global_sicherheit_verpasst
timer.global_sicherheit_cooldown
binary_sensor.global_ha_startphase
```

**Begründung:** `global` signalisiert instanzweiten Zustand, kein physischer Raum. Gilt identisch für LB und RBO — beide Instanzen nutzen dasselbe `global`-Prefix.

### Personen & Geräte-Tracker

```
device_tracker.<vorname>_<geraet>
person.<vorname>
```

Beispiele:
- `person.mirko`
- `device_tracker.mirko_handy`

### Netzwerk-Entities

```
<domain>.netzwerk_<funktion>[_<bezeichner>]
```

### Virtuelle / berechnete Entities (Template, Helfer)

Selber Aufbau wie physische Entities — kein Sonderpräfix.

```
binary_sensor.wohnzimmer_belegung
sensor.wohnzimmer_temperatur_mittelwert
```

### Automationen & Skripte

```
automation.<ort>_<ausloeser_oder_funktion>
script.<funktion>
```

Beispiele:
- `automation.wohnzimmer_licht_bei_bewegung`
- `automation.global_abwesenheit_aktivieren`
- `automation.kueche_schalter_decke`
- `script.benachrichtigung_push`

### Szenen

```
scene.<ort>_<stimmung_oder_funktion>
```

### Energie-Sensoren (Siblings von Switches)

Bei Shelly/Zigbee-Geräten mit Energie-Messung folgen Siblings der Konvention des zugehörigen Switch, mit der Messgröße als Funktionspräfix:

```
switch.kueche_spuelmaschine
sensor.kueche_leistung_spuelmaschine
sensor.kueche_energie_spuelmaschine
sensor.kueche_strom_spuelmaschine
```

Nicht: `sensor.spuelmaschine_power` (Hersteller-Default) oder `sensor.kueche_spuelmaschine_power` (redundante Domain-Kennung).

---

## Labels (Querschnittskategorien)

Labels ersetzen was früher im Namen stand. Empfohlenes Basis-Set — gilt für beide Instanzen:

| Label | Verwendung |
|---|---|
| `saisonal` | Geräte die periodisch offline sind (Gardena, Bewässerung, etc.) |
| `batterie` | Batteriebetriebene Geräte |
| `energie` | Energie-Monitoring relevant |
| `leistung` | Leistungsmessung (power-Sensoren) |
| `temperatur` | Temperatursensoren |
| `feuchtigkeit` | Feuchtigkeitssensoren |
| `zigbee` | Zigbee-Geräte |
| `wlan` | WLAN-Geräte (Shelly etc.) |
| `bluetooth` | Bluetooth-Geräte |
| `kritisch` | Ausfall löst Benachrichtigung aus |

---

## Floors & Areas

| Floor | Areas |
|---|---|
| Erdgeschoss | Wohnzimmer, Küche, Flur, Vorzimmer, Badezimmer, Schlafzimmer |
| Außen | Garten, Vorgarten, Einfahrt, Terrasse, Schuppen, Carport |
| Haustechnik | Haustechnik |

> Floor-Struktur ist instanzspezifisch — LB und RBO können abweichen. Die Raumbezeichnungen bleiben jedoch nach obigem Glossar einheitlich.

---

## Beispiele: Vorher → Nachher

| Vorher (unsystematisch) | Nachher (Konvention) | Anmerkung |
|---|---|---|
| `light.led_kuche` | `light.kueche_decke` | Redundanz `licht` entfernt |
| `switch.licht_kuche` | `switch.kueche_decke_schalter` | Ort zuerst |
| `switch.wz_fernseher` | `switch.wohnzimmer_steckdose_fernseher` | Ort ausgeschrieben |
| `switch.lb_bewegungsmelder` | `switch.garten_aussenlicht_relay` | lb_ entfällt, Funktion statt Gerät |
| `switch.lb_pm_hzks` | `switch.haustechnik_pm_heizkreis` | lb_ entfällt |
| `binary_sensor.belegung_wohnzimmer_kombiniert` | `binary_sensor.wohnzimmer_belegung` | kombiniert entfällt |
| `binary_sensor.echte_bewegung_badezimmer` | `binary_sensor.badezimmer_bewegung` | echte_ entfällt |
| `binary_sensor.fenster_1_window` | `binary_sensor.wohnzimmer_fenster_1` | window-Suffix entfällt |
| `sensor.h_t_badezimmer_temperature` | `sensor.badezimmer_temperatur` | Gerätetyp-Prefix entfällt |
| `sensor.t_h_schuppen_humidity` | `sensor.schuppen_luftfeuchte` | Deutsch, kein Hersteller-Suffix |
| `sensor.lb_gesamtleistung_em0_power` | `sensor.haustechnik_leistung_gesamt` | lb_ + em0_ entfallen |
| `sensor.waschmaschine_power` | `sensor.kueche_leistung_waschmaschine` | Ort ergänzt, Messgröße vorn |
| `input_boolean.abwesenheit` | `input_boolean.global_abwesenheit` | global für raumneutrale Helfer |
| `timer.sicherheit_cooldown` | `timer.global_sicherheit_cooldown` | global-Prefix konsistent |
| `automation.kuche_wandschalter_led` | `automation.kueche_schalter_decke` | Funktion, kein Gerät |
| `cover.rolladen_01` | `cover.wohnzimmer_rolladen_1` | Ort ergänzt, _0 entfällt |
| `climate.heizung_wohnzimmer` | `climate.wohnzimmer_heizung` | Ort zuerst |

---

## Was NICHT umbenannt wird

| Kategorie | Begründung |
|---|---|
| Extern vergebene IDs (DWD, Gardena, Tado, Shelly-Diagnose) | Integration kontrolliert die ID — nur `friendly_name` anpassbar |
| Battery Notes Sub-Entities (`*_battery_plus_low` etc.) | Von Battery Notes intern verwaltet |
| Diagnose-Entities (cloud, rssi, uptime, firmware, temperature) | Nicht in Automationen verwendet, Umbenennung ohne Mehrwert |
| Entities ohne `unique_id` | HA lässt Rename nicht zu |

---

## Migrationsregeln

> **Ausführliche Schrittanleitung:** `references/safe-refactoring.md` im `home-assistant-best-practices`-Skill — insbesondere §Config-Entry-Data für Setup-Flow-Integrationen (Better Thermostat, Generic Thermostat) und Storage-Mode-Dashboards.

Kurzfassung für tägliche Arbeit:

1. **In Etappen** — pro Raum oder pro Domain, nie alles auf einmal
2. **Reihenfolge:** YAML-Bulk-Replace → `core.config_entries` patchen → Registry-Rename → Gruppen reparieren → Dashboards patchen → HA-Neustart
3. **YAML zuerst, Registry danach** — `ha_rename_entity` aktualisiert YAML-Referenzen NICHT automatisch
4. **`core.config_entries` vor dem Neustart** — Integrationen mit Setup-Flow (BT, Generic Thermostat) speichern entity_ids in `data`, nicht in YAML; mit veralteten IDs starten sie fehlerhaft
5. **Storage-Dashboards** werden durch `ha_rename_entity` NICHT aktualisiert — manueller Patch nötig, danach HA-Neustart
6. **Externe IDs** — nur `friendly_name` ändern, `entity_id` unangetastet lassen
7. **Nach jeder Etappe** — `sensor.active_issues` prüfen, Spook-Issues auflösen

---

## Abgrenzung: Was gehört NICHT in den entity_id

| Information | Stattdessen |
|---|---|
| Stockwerk | Floor |
| Gerätemarke / Modell | Device-Name |
| Saisonalität | Label `saisonal` |
| Energierelevanz | Label `energie` |
| Kommunikationsprotokoll (Zigbee / WiFi / BT) | Label oder Device-Info |
| Instanz (LB/RBO) | ist durch die HA-Instanz selbst klar |
| Gerätegeneration / Revision | Device-Name oder überhaupt nicht |
