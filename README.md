# ha-betriebshandbuch

HA-Betriebshandbuch für Claude-Instanzen — instanzunabhängiges Systemwissen für alle Home Assistant Installationen.

Dieser Skill enthält API-Syntax, YAML-Regeln, Storage-Schemata und dokumentierte Fallen. **Instanzspezifische Daten** (Entity-IDs, IPs, Automationslisten) gehören nicht hierher — sie liegen in der instanzeigenen `claude.md`.

---

## Verwandte Skills

Dieser Skill arbeitet eng mit dem **[home-assistant-best-practices](https://github.com/homeassistant-ai/skills)**-Skill zusammen (Upstream: `homeassistant-ai/skills`):

| Skill | Zuständigkeit |
|-------|--------------|
| **ha-betriebshandbuch** (dieser Skill) | API-Syntax, YAML-Regeln, Storage-Schemata, bekannte Fallen, Statistiken reparieren |
| **home-assistant-best-practices** | Helper-Auswahl (min_max vs. threshold vs. derivative …), Automationsmodi, wait-Semantik, Trigger-ID-Muster |

**Regel:** Vor jeder neuen Automation und vor jedem neuen Helper-Typ beide Skills im Kontext vorhalten.

Ladebefehl Best-Practices-Skill:
```bash
curl -s https://raw.githubusercontent.com/homeassistant-ai/skills/main/skills/home-assistant-best-practices/SKILL.md
```

---

## Voraussetzungen — Was muss in der System Instruction stehen?

Damit Claude mit diesem Skill arbeiten kann, braucht sie zur Laufzeit Zugang zu instanzspezifischen Daten. Diese gehören **ausschließlich in die System Instruction (SI)** der jeweiligen Claude-Session — nicht in den Skill, nicht in öffentliche Repos.

Fehlen sie, kann Claude weder API-Calls ausführen noch die `claude.md` laden.

### Pflichtfelder

**1. HA-Base-URL**
Jeder API-Call braucht den vollständigen Endpunkt. Ohne URL ist kein einziger REST- oder `shell_command`-Call möglich.
```
https://<subdomain>.ui.nabu.casa
```
Alternativ bei lokalem Zugang: `http://192.168.x.x:8123`

**2. Long-Lived Access Token**
Erforderlich für die Authentifizierung aller HA-REST-API-Calls (Bearer-Token, §1.1 im Skill). Ohne Token antwortet die API mit HTTP 401.

> Sicherheitsregel: Token nur als lokale Variable im Skript halten — niemals in `window.*`, `localStorage` oder anderen persistenten Browserspeichern.

Erstellen: HA-Profil → Sicherheit → Langfristige Zugriffstoken

**3. Ladebefehl für `claude.md`**
Die Datei enthält das gesamte instanzspezifische Systemwissen. Ohne sie arbeitet Claude blind — kennt keine Entity-IDs, keine Automationen, keine offenen Punkte. Sie muss am Session-Anfang geladen werden.
```bash
curl -s -H "Authorization: Bearer <TOKEN>" \
  -X POST -H "Content-Type: application/json" \
  -d '{"filename":"/config/claude.md"}' \
  'https://<subdomain>.ui.nabu.casa/api/services/shell_command/read_file?return_response'
```

**4. Skill-Ladebefehle**
Skills liegen auf GitHub und sind nicht automatisch im Kontext. Ohne Ladebefehl fehlen API-Syntax, YAML-Regeln und bekannte Fallen.
```bash
curl -s https://raw.githubusercontent.com/Patch76/ha-betriebshandbuch/main/SKILL.md
curl -s https://raw.githubusercontent.com/homeassistant-ai/skills/main/skills/home-assistant-best-practices/SKILL.md
```

**5. Telegram-Konfiguration** *(wenn Benachrichtigungen genutzt werden)*
`config_entry_id` und `chat_id` sind instanz- und account-spezifisch. Ohne sie schlagen alle Telegram-Service-Calls fehl.
```
config_entry_id: <entry-id>
chat_id: <chat-id>
parse_mode: markdownv2
```

**6. SSH-/Terminal-Zugang** *(wenn Direkt-Python benötigt wird)*
Für Dateioperationen > ~50 KB (z.B. `core.entity_registry`) ist direktes Python via SSH der einzige zuverlässige Weg (§2.3b im Skill). Claude muss wissen, wo das Terminal erreichbar ist.
```
Tab-URL: /app/<addon-id>_ssh   ODER   user@192.168.x.x -p 22
```

**7. GitHub-PAT** *(wenn Skill-Pflege oder PR-Workflow genutzt wird)*
Lesender Zugriff auf öffentliche Repos funktioniert ohne Token. Schreibende Operationen (PR, Push) erfordern Authentifizierung.

> Sicherheitsregel: PAT nur als lokale Variable halten — niemals in `window.*`.

### Optionale Felder

- Kommunikationssprache und Anrede
- Explizite Neustart-Bestätigungspflicht
- Notfall-SSH-Zugangsdaten (Fallback bei UI-Ausfall)

---

## Was gehört in die `claude.md`?

Das instanzspezifische Systemgedächtnis — nur Wissen, das für genau eine HA-Installation gilt. Allgemeines HA-Wissen gehört in diesen Skill.

| Datentyp | Warum? |
|----------|--------|
| Systemeckdaten (Version, Timezone, Koordinaten) | Claude prüft API-Antworten gegen bekannte Werte — Abweichungen werden sofort erkannt |
| Konfigurationsdateistruktur (`!include`-Mapping) | Vor jedem Schreibvorgang muss klar sein, welche YAML-Datei welchen Inhalt hat |
| IoT-Stack und Integrationen (mit Adressen/Ports) | Ohne Broker-IPs und Ports keine gerätespezifischen Diagnosen oder Konfigurationen |
| Zonen (Name, Radius, Zweck) | Die Semantik der Zonen (z.B. „Fernzone = Vorheiz-Trigger") ist instanzspezifisch |
| Infrastrukturdetails (Hardware-Eigenheiten, BSSIDs) | Bekannte Bugs und Einschränkungen (z.B. ESP32-Revisionen) verhindern Fehlkonfigurationen |
| Template-Sensor-Inventar | Verhindert Duplikate und die Block-Falle (§4.1) |
| Automations-Inventar (ID, Alias, Modus) | Bei 30+ Automationen in einer Datei entstehen ohne Übersicht Konflikte und stille Überschreibungen |
| Helfer-Inventar (Storage-ID ↔ Entity-ID) | Storage-ID und Entity-ID können abweichen — korrekte Zuordnung ist Pflicht für Storage-Operationen (§5) |
| Recorder-Konfiguration + Exclude-Liste | Ohne Dokumentation werden bei Änderungen versehentlich wichtige Entities aus- oder eingeschlossen |
| Labels (vollständige Liste) | Neue Labels nur anlegen, wenn die bestehende Liste das Kriterium nicht abdeckt |
| Fehler-Chronik | Dokumentiert Fallen, die bereits aufgetreten sind — Claude liest sie vor ähnlichen Operationen |
| Offene Punkte (ToDo-Liste) | Persistenter Aufgabenrückstand über Session-Grenzen hinweg |
| Verweis auf Best-Practices-Skill + Ladebefehl | Stellt sicher, dass Claude den Skill vor jeder neuen Automation lädt |

**Was nicht in `claude.md` gehört:**
- Tokens, Passwörter, Secrets → nur in der SI
- Allgemeine API-Syntax, YAML-Regeln → dieser Skill
- Unverifiziertes Wissen (Verifikationspflicht §0 im Skill)

### Empfohlene Session-Startreihenfolge

```
1. SI enthält Token + URL        → stehen sofort zur Verfügung
2. Skills laden (GitHub-curl)    → API-Regeln und Best-Practices verfügbar
3. claude.md laden (read_file)   → instanzspezifisches Wissen verfügbar
4. Skill-Versionen bestätigen    → Session bereit
```

---

## Pflegeprinzip

Dieses Handbuch wird durch Claude verwaltet — nach Rücksprache mit dem Instanzbetreiber.

```
Änderungsbedarf → PR auf Patch76/ha-betriebshandbuch → Merge → nächste Session zieht automatisch die aktuelle Version
```

Nur live getestetes oder per API/Dokumentation verifiziertes Wissen wird eingetragen. Version und Changelog werden bei jedem Update gepflegt.
