# API & Storage

Teil des ha-betriebshandbuch — Storage-Dateien, WebSocket API.
Nachladen: `references/api-storage.md`

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

### 17.5 Dashboard-Migration nach Entity-Rename (verifiziert 21.03.2026)

**WICHTIG: `ha_config_set_dashboard` nutzt `websocket_lovelace_save_config`** — kein HA-Restart nötig.
Direkte `.storage`-Datei-Edits (via `write_file`) benötigen dagegen einen Restart.

**Korrekter Workflow (kein Restart):**
```python
# 1. Dashboard-Config laden
# ha_config_get_dashboard(url_path="mein-dashboard")
# → gibt config + config_hash zurück

# 2. Entity-IDs ersetzen (extern, ohne Sandbox-Einschränkungen)
import json
config = dashboard_config  # aus Schritt 1
config_str = json.dumps(config)
config_str = config_str.replace('"old.entity_id"', '"new.entity_id"')
new_config = json.loads(config_str)

# 3. Zurückschreiben — sofort wirksam, kein Restart
# ha_config_set_dashboard(url_path="mein-dashboard", config=new_config)
```

**python_transform Sandbox — Einschränkungen (verifiziert):**
- ❌ Blockiert: `import`, `def`, `isinstance`, `str.replace()`, `__class__`
- ✅ Erlaubt: dict/list-Zugriff, Schleifen, `split()`+`join()`, String-Methoden
- Für Bulk-Replace Entity-IDs: **nicht geeignet** → `config=`-Parameter verwenden

**Warum nicht `write_file` + Restart:**
- Direkte `.storage`-Edits werden erst beim nächsten HA-Start eingelesen
- `ha_config_set_dashboard(config=...)` ist atomar + sofort wirksam
- Bei write_file: Datei >~40 KB → HTTP 500 (Workaround: `json.dumps(data, separators=(',',':'))`)

**Für alle storage-mode Dashboards:**
```python
# Alle Dashboards auflisten
# ha_config_get_dashboard(list_only=True) → url_path Liste
# Dann pro Dashboard: get → replace → set
```

---

