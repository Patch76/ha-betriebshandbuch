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
  version: "2.63.0"
  maintainer: "Claude (via PR, nach Rücksprache mit Mirko)"
  workflow: "Änderungsbedarf → PR auf Patch76/ha-betriebshandbuch → Mirko mergt → nächste Session zieht automatisch. Jede inhaltliche Änderung: Version + Changelog im selben Commit (→ §0 Skill-Pflege)."
  source: "Verifiziert an HA 2026.3.0 — aus claude.md + Live-Tests 08.03.2026"
  changelog: >
    2.63.0 (22.03.2026): references/github.md Fallstricke-Tabelle — JSONDecodeError bei
      GitHub-API-Responses dokumentiert: Comment-Bodies mit Steuerzeichen → strict=False.
    2.62.0 (22.03.2026): references/github.md DOM-Reply-Muster überarbeitet —
      safeInit() mit window.confirm-Override + display:none, 4-stufiger XPath-Lookup
      für lazy-loaded Threads, React-konformer Textarea-Fill via execCommand,
      sicherer Submit-Button-Selektor, Delete-Flow dokumentiert. Verifiziert 22.03.2026.
    2.61.0 (22.03.2026): references/meta.md §22.2 — Hinweis auf erlaubte
      Zusatzabschnitte zwischen ⑥ und Offene Punkte ergänzt. Template-Drift
      zwischen §22.1 und realer claude.md (doppeltes ⑥, ⑧ statt ⑦) damit
      dokumentiert und erklärbar.
    2.60.0 (22.03.2026): references/recorder-stats.md §11.1 reload-Text korrigiert
      (dritte Stelle derselben falschen Aussage). §19.1 unverifizierbares Zitat
      entfernt, durch sinngemäße Formulierung ersetzt.
      references/integrations.md §26.2 Retrofix: homeassistant/enable_entity +
      disable_entity als REST-Service-Calls ergänzt (live verifiziert LB 22.03.2026,
      HTTP 200). PR #60 hatte diese Variante übersehen.
    2.59.0 (22.03.2026): references/integrations.md §26.2 kein REST für entity_registry/update;
      WebSocket + MCP-Tool dokumentiert (live verifiziert LB). §26.4 Abschnittsnummer korrigiert
      (war §16.4). §15.4 LB-URL entfernt (instanzspezifisch). SKILL.md §27 Filter ①:
      "Live-Test" präzisiert: gegen laufende HA-Instanz via API (curl, Python, ha-mcp).
    2.58.0 (22.03.2026): references/automations.md §9.3 due_date — RFC-5545-+1-Tag gilt
      nur für Google Tasks (bekannter Bug), nicht für local_todo (live verifiziert LB).
      §9.3 todo.get_items "ohne status" → alle nicht-abgeschlossenen Items (nicht nur needs_action).
    2.57.0 (22.03.2026): references/api-storage.md §5.1 reload-Formulierung korrigiert
      (liest nicht, überschreibt nicht — In-Memory-Stand bleibt, verifiziert LB).
      §5.4 ha_reload_core(target=…) → HTTP 400; korrekt: input_boolean/reload (verifiziert).
      §17.5 Schwellwert 40KB → 95KB (→ §2.3b); separators-Workaround entfernt.
      §17.1 HTTP 403 verifiziert mit echter websockets-Library (nicht 400).
    2.56.0 (22.03.2026): §23 Verifikationstabelle — Trace-Hinweis korrigiert (kein REST-Endpunkt,
      nur UI/WebSocket, verifiziert LB). Logbuch-Check kontextualisiert + §1.6-Verweis.
      3 fehlende Kontexte ergänzt: Storage, Config-Entry, YAML-Reload.
    2.55.0 (22.03.2026): §27 §Sicht — Filter ① Faktencheck als Pflicht verschärft:
      Web-Recherche + Live-Test immer ausführen, kein ungetestetes Vererben, Ungetestetes
      als [UNVERIFIZIERT] kennzeichnen. Grundprinzip entsprechend geschärft, Widget-Schwelle
      konkretisiert (≥2 substanzielle Korrekturen). "Kein Fund" aus ② in allgemeines
      Prinzip verschoben. §Sicht mit §Sicht selbst geprüft — 3 Befunde korrigiert.
    2.54.0 (22.03.2026): §1.6 Logbook-Filter — Ursache ergänzt: offizieller Parameter entity=
      liefert 0 Einträge (Sensor-Ausschluss), entity_id= ignoriert Filter. Konsequenz:
      History-API als Alternative empfohlen. Verifiziert LB + Dev-Doku 22.03.2026.
    2.53.0 (22.03.2026): §20 Anti-Pattern "Storage reload statt Restart" — Warum-Spalte korrigiert:
      reload überschreibt Storage NICHT; reload liest Storage auch NICHT neu ein; HA behält
      In-Memory-Stand bis Vollneustart. Live-verifiziert LB 22.03.2026 (core.area_registry-Test).
      §20 "New storage-Eintrag" → "Neuer" (Sprachmix).
    2.52.0 (22.03.2026): §4.4 Config-Entry-Flow vollständig (3 Schritte, flow_id, Abbruch-Endpoint,
      verifiziert LB 22.03.2026). §4.2 Verifikationsquelle "sergeykad" → "LB, HA 2026.3".
    2.51.0 (22.03.2026): references/github.md §Git PR-Workflow — Branch-Delete nach
      Squash-Merge als Pflichtschritt ergänzt (fehlte, wurde von Gegenstelle angemerkt).
      Version-Bump nachgeholt (PR #51 hatte keinen).
    2.50.0 (22.03.2026): §2.3b Querverweis §11.1 → §2.10 (SSH-Terminal, nicht Zombie-Bereinigung).
      §2.7 Ausnahmeliste um lb_to_rbo.md ergänzt. §2.7 repr()-Kommentar "Screenshot" → "Darstellung".
      §2.10.1 Cyrillisches "т" (U+0442) → lateinisches "t" in "wrappt".
    2.49.0 (22.03.2026): §0 Backup-Verweis auf §2.7 (2-Slot-Rotation, cp-Snippet entfernt).
      §0 web_fetch-Regel korrigiert: user-provided URLs erlaubt (Anthropic Docs verifiziert 22.03.2026).
      §0 15-Minuten-Heuristik → max. 3 gescheiterte Versuche. §1.1 HTTP 500 ergänzt (→ §2.3b).
      §1.4 states.entity_id → states['sensor.x'] (konkretes Beispiel). §1.5 History-Beispiel
      zeigt beide Varianten (mit/ohne ISO-Z).
    2.48.0 (22.03.2026): §2.3b Kurzbefehl-Kommentar korrigiert (50000→90000 Bytes, verifiziert per Binärsuche LB 22.03.2026). §2.9 Wortstellung 'write_file shell_command' → 'shell_command write_file' (Leserklarheit).
    2.47.0 (22.03.2026): §27 neu — §Sicht Sechsfach-Review: Qualitätsfilter vor Publish.
      Sechs aktive Filter (Faktencheck, Black Hat, Scope, Pragmatiker, Leser, Compliance).
      Mechanik: interner Korrekturlauf → verbessertes Ergebnis, kein erklärendes Widget.
    2.46.0 (22.03.2026): references/github.md §10 neu — GraphQL Discussions API (POST ohne Browser).
      Session-34-Erkenntnisse: LoggingUndefined-Kontext, Anchor-Sync-Muster, Tool-Namen-Grep.
    2.45.0 (21.03.2026): §2.7 Backup-Pflicht mit 2-Slot-Rotation ergänzt (DATEI.bak + DATEI.bak.prev).
      Gilt für kritische Dateien. CLAUDE.md §⓪ + §! verschärft.
    2.44.0 (21.03.2026): §2.10.1 neu — SSH-Terminal-Eingabe: triggerDataEvent als einzig
      korrekte Methode (paste() versagt wegen Bracketed Paste Mode in zsh, verifiziert).
      Muster: Einzeiler, Python-Heredoc, Ctrl-C, Sentinel-Pattern, Buffer-Scan.
    2.43.0 (21.03.2026): references/integrations.md §13.4 — BT-Kontrollzyklus hardcoded 5 min dokumentiert (verifiziert aus climate.py Quellcode).
    2.42.0 (18.03.2026): metadata.workflow — Version-Bump-Pflicht explizit in Workflow-Beschreibung ergänzt (→ §0 Skill-Pflege). Verifiziert LB 18.03.2026.
    2.41.0 (18.03.2026): §0 Skill-Pflege — Version-Bump-Pflicht präzisiert: Bump muss im selben Commit wie die inhaltliche Änderung erfolgen; Kanal darf nur die tatsächliche Post-merge-Version nennen. Lücke in AW ⑧ (kein Bump-Schritt verlangt). Verifiziert LB + RBO 18.03.2026.
    2.39.0 (15.03.2026): §16 neu — Shelly Button-Modus mit Kippschalter: btn_down+btn_up statt single_push; Detached+Button-Modus Voraussetzung für Event-Entity; Anti-Pattern-Tabelle. Verifiziert LB 15.03.2026.
    2.38.0 (14.03.2026): §20 Anti-Pattern — `enabled: false` in automations.yaml für UI-Automationen → Repair-Issue. Korrekt: `ha_set_entity(enabled=False)`.
    2.37.0 (14.03.2026): §13.1 Preset-Abbruch — Restore-Schritt MUSS vor Helper-Leeren erfolgen (Bug-Pattern + Anti-Pattern dokumentiert). §16.2 next_alarm als unzuverlässig eingestuft.
    2.36.0 (12.03.2026): §13 Preset `activity` ergänzt (live verifiziert). §16.2 Hinweis auf `disabled_by: integration` für next_alarm + last_update_trigger — Aktivierung via REST PUT entry_id.
    2.34.0 (12.03.2026): §24.3 Status korrigiert — RBO vollständig auf Companion App migriert; Script nachtruhe_anfrage_senden (letztes Telegram-Relikt) gelöscht.
    2.33.0 (12.03.2026): §25 neu — Companion App notify-Service; actionable notifications (wait_for_trigger-Pattern); replace(_,space)-Best-Practice für Template-Messages; Migration RBO Telegram→Companion App dokumentiert.
    2.32.0 (12.03.2026): §24.3 neu — `target:`-Parameter deprecated seit 2026.3 (Entfernung in 2026.9); `chat_id` ist korrekte Syntax; RBO-Automationen bereits compliant.
    2.31.0 (12.03.2026): §6.6 Ghost-Update-Entity (Supervisor-Add-on) ergänzt — orphaned Repository-Fix via `ha store delete <slug>`.
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
- **Backup vor jeder destruktiven Aktion (→ §2.7 — 2-Slot-Rotation):**
  `DATEI.bak` → `DATEI.bak.prev` · `DATEI` → `DATEI.bak` · erst dann schreiben.
  Gilt für: automations.yaml, template.yaml, scripts.yaml, scenes.yaml, alle `.storage/`-Dateien.
- **Bei unbekanntem Fehler zuerst `web_search`** — nicht mehr als 3 gescheiterte Versuche ohne neue Information.
- **Öffentliche Doku-URLs:** `web_fetch` nur mit URLs die im Kontext bekannt sind: vom Nutzer angegeben,
  aus `web_search`-Ergebnissen oder früheren `web_fetch`-Calls. Keine selbst konstruierten URLs.
  Browser (`computer`-Tool) nur für HA-UI (Auth-geschützt) oder interaktive Seiten — nicht für Doku.
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
    ⚠️ Version-Bump MUSS im selben Commit wie die inhaltliche Änderung erfolgen.
    Im Kanal darf ausschließlich die tatsächliche Post-merge-Version genannt werden —
    nie vorab eine geplante Versionsnummer ankündigen.
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

### Kanal-Sendepflicht: Was ist berichtenswert?

Berichtenswert (→ Kanal beschreiben):
- Neue/geänderte Referenz-Dateien oder Skill-Versionen
- Neue Kommunikationssignale (§-Befehle)
- API-Erkenntnisse, neue Fallstricke, Breaking Changes mit Relevanz für beide Instanzen
- Neue Helper-Typen oder strukturelle Konfigurationsmuster
  → Automation-Muster gehören ins Betriebshandbuch (PR → §), Kanal verweist nur auf den §

Nicht berichtenswert:
- Instanzspezifische Fixes (Entity-IDs, IPs, Werte, BSSIDs)
- Reine Wertänderungen ohne strukturelle Relevanz

### Pflicht-Warnblock (Kopf jeder Kanalnachricht — zwingend)

```
⚠️ IDENTITÄTSPFLICHT — Alle Inhalte sind strukturelle Beispiele der sendenden Instanz.
VOR jeder Übernahme zwingend gegen eigene Instanz prüfen:
Token · Credentials · PAT · Entity-IDs · Helper-Namen · IPs · Pfade · BSSIDs · Zonen
Nicht verifiziert = nicht umsetzen. (→ §0 Kanal-Verifikationspflicht)
```

---


## 1. REST API — Endpunkte & Syntax

### 1.1 Authentifizierung

Jeder Call braucht:
```
Authorization: Bearer <LONG_LIVED_TOKEN>
Content-Type: application/json
```

HTTP-Statuscodes: `200` OK, `201` Created, `400` Bad Request, `401` Unauthorized,
`404` Not Found, `500` Server Error (z.B. Payload-Limit `write_file` → §2.3b).

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
- `states['sensor.mein_sensor'].last_changed | string` → funktioniert

### 1.5 History-API
```
GET /api/history/period?filter_entity_id=<entity_id>&minimal_response=true          # letzter Tag
GET /api/history/period/2026-03-22T00:00:00Z?filter_entity_id=<entity_id>&minimal_response=true  # ab Startzeitpunkt
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
- **Filter — beide Parameter unzuverlässig** (verifiziert LB 22.03.2026):
  - Offizieller Parameter laut Dev-Doku: `entity=<entity_id>` → liefert 0 Einträge
    (Sensor-Entitäten mit `unit_of_measurement` werden vom Logbook automatisch ausgeschlossen)
  - Inoffizieller Parameter: `entity_id=<entity_id>` → ignoriert Filter, liefert alle Domains
  - **Konsequenz:** Logbook-API für gezieltes Entity-Filtering praktisch unbrauchbar;
    für Entity-Historie stattdessen History-API (§1.5) verwenden.

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

**RICHTIG — direktes Python auf der HA-Maschine via SSH-Terminal (→ §2.10):**
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
wc -c /config/.storage/DATEI   # > 90000 Bytes → SSH-Terminal verwenden
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

#### Backup-Pflicht (2-Slot-Rotation)

Vor jedem Schreiben auf **kritische Dateien** (claude.md, automations.yaml, template.yaml):

```
claude.md          ← aktuell
claude.md.bak      ← Stand vor diesem Write    (Slot A)
claude.md.bak.prev ← Stand vor dem letzten Write (Slot B)
```

```python
# Slot-Rotation: .bak → .bak.prev (fehlt .bak → still überspringen)
try:
    bak = read_file("/config/DATEI.bak")          # Slot A lesen
    write_file("DATEI.bak.prev", bak)             # → Slot B
except Exception:
    pass  # Erstlauf oder .bak fehlt → überspringen

# Aktuellen Stand sichern → Slot A
c_original = read_file("/config/DATEI")
r_bak = write_file("DATEI.bak", c_original)
assert r_bak["service_response"]["returncode"] == 0, "Backup fehlgeschlagen — Abbruch"
```

Nicht nötig für: session_handoff.md, rbo_to_lb.md, lb_to_rbo.md (unkritisch/ersetzbar).
Wiederherstellung: `write_file("DATEI", read_file("/config/DATEI.bak"))`

#### Vollständiges Muster

```python
import json, base64, urllib.request

TOKEN = "..."
BASE  = "https://HA_HOST"

def read_file(filename):
    req = urllib.request.Request(
        f"{BASE}/api/services/shell_command/read_file?return_response",
        data=json.dumps({"filename": filename}).encode(),
        headers={"Authorization": f"Bearer {TOKEN}", "Content-Type": "application/json"},
        method="POST"
    )
    return json.loads(urllib.request.urlopen(req).read())["service_response"]["stdout"]

def write_file(path, content):
    b64 = base64.b64encode(content.encode("utf-8")).decode("ascii")
    body = json.dumps({"path": path, "content_b64": b64}).encode()
    req = urllib.request.Request(
        f"{BASE}/api/services/shell_command/write_file?return_response",
        data=body,
        headers={"Authorization": f"Bearer {TOKEN}", "Content-Type": "application/json"},
        method="POST"
    )
    # urllib.request liefert HA-JSON direkt — kein bash_tool-Wrapping!
    return json.loads(urllib.request.urlopen(req).read())

# 0. BACKUP (siehe oben — bei kritischen Dateien)

# 1. LESEN
c = read_file("/config/DATEI")

# 2. repr()-CHECK (Pflicht vor str.replace)
# python3 -c "c=open('/config/FILE').read(); [print(repr(l)) for l in c.splitlines() if 'Begriff' in l]"
# Grund: Umlaute, Quotes, Whitespace im Dateiinhalt können von der Darstellung abweichen → replace() schlägt lautlos fehl
# ACHTUNG EOF: Letzte Zeile hat oft KEIN trailing \n.
#   Anker 1:1 aus repr()-Output kopieren — nie aus Screenshot oder Gedächtnis.

# 3. ÄNDERN
OLD = "exakt kopierter String aus repr()-Output"
NEW = "neuer Inhalt"
assert OLD in c, f"Anker nicht gefunden: {repr(OLD[:60])}"
c = c.replace(OLD, NEW, 1)

# 4. SCHREIBEN
r = write_file("PFAD_RELATIV_ZU_CONFIG", c)   # z.B. "automations.yaml", "claude.md"
print(f"rc={r['service_response']['returncode']} err={r['service_response']['stderr'] or 'OK'}")

# 5. VERIFIZIEREN (separater curl-Call)
# GET /api/states/<entity> oder erneutes read_file + Stichprobe
```

**VERBOTEN:**
- `computer.type` für Inhalte >200 Zeichen (Disconnect-Risiko)
- `cat`-Heredoc für Dateiinhalt (lautloser Datenverlust bei Verbindungsabbruch)
- Mehrere `str.replace()` ohne vorherigen `assert`-Check je Anker
- Schreiben auf kritische Dateien ohne vorherige Backup-Rotation

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

Falls nicht verfügbar: File-Schreiben ausschließlich via shell_command
`write_file` (§2.3). `open()` steht nur in Add-ons und SSH zur
Verfügung — nicht in Automationen, Scripts oder template.yaml.

### 2.10 SSH-Terminal — Verhaltensregeln

- **Tab-URL nie verlassen** — Navigation weg vom SSH-Tab öffnet den Dialog
  „Website verlassen?“ und bricht die Session unwiderruflich ab.
- **Browser-Aktionen immer in neuem Tab** öffnen (`tabs_create_mcp`),
  nie im SSH-Tab selbst navigieren.
- **Fallback:** SSH-Tab-URL instanzspezifisch im SI/AW dokumentiert.

#### 2.10.1 Befehle senden — triggerDataEvent (verifiziert 21.03.2026)

**Einzige korrekte Methode:** `window.term._core.coreService.triggerDataEvent(s)`

`paste()` **ist FALSCH** für ausführbare Befehle: zsh hat Bracketed Paste Mode aktiv,
`paste()` wrappt den Inhalt mit `ESC[200~...ESC[201~` — `\r` wird als Literal,
nicht als PTY-Signal behandelt. Befehl tippt, führt aber nie aus.

```javascript
// Helfer (einmalig)
const tde = s => window.term._core.coreService.triggerDataEvent(s);

// Zeile leeren
tde('\x15');            // Ctrl-U

// Einzeiler ausführen
tde('echo hallo\r');

// Prozess abbrechen
tde('\x03');            // Ctrl-C

// Mehrzeiliges Python via Heredoc — jede Zeile einzeln mit \r
const lines = [
  "python3 << 'PYEOF'",
  "import json",
  "print(json.dumps({'ok': True}))",
  "PYEOF",
];
for (const l of lines) tde(l + '\r');
```

**Ausgabe lesen — Buffer-Scan:**
```javascript
const t = window.term, buf = t.buffer.active;
const out = [];
for (let i = buf.length-1; i >= 0 && out.length < 30; i--) {
  const l = buf.getLine(i)?.translateToString(true);
  if (l?.trim()) out.unshift(l.trimEnd());
}
out.join('\n');
```

**Sentinel-Pattern für sichere Completion-Erkennung:**
```javascript
tde("befehl; echo '__DONE__'\r");
// Buffer scannen bis '__DONE__' sichtbar
```

---


## 20. Bekannte Anti-Patterns (Schnellreferenz)

| Anti-Pattern | Lösung | Warum |
|---|---|---|
| `condition: template` mit `float > 25` | `condition: numeric_state` | Validiert bei Load, nicht Runtime |
| `wait_template` für „erst noch eintreten" | `wait_for_trigger` | Semantik verschieden |
| `device_id` in Triggers (Ausnahme: Z2M Zigbee-Remote-Actions — kein `entity_id`-Äquivalent) | `entity_id` bevorzugen; für Z2M Remote-Actions `device_id` akzeptabel (off. Z2M-Empfehlung) | `device_id` bricht bei Neuanlernen; kein Templating, kein `repeat/until` möglich |
| `mode: single` für Bewegungslicht | `mode: restart` | Re-Trigger muss Timer resetten |
| Helper in Abbruch-Automation vor Restore leeren | Restore-Schritt (number.set_value) VOR Helper-Leeren | Helper-Wert weg → Preset bleibt dauerhaft erhöht (§13.1) |
| `mode: single` bei Automation mit internem wait_for_trigger die re-triggert werden muss | `mode: restart` | single ignoriert zweiten Trigger während wait → Folgeaktionen laufen nicht |
| Template-Sensor für Summe/Mittel | `min_max` Helper | Deklarativ, handled unavailable |
| Template-Sensor mit Schwellwert | `threshold` Helper | Eingebaute Hysterese |
| `initial:` bei input_number | weglassen | Setzt Wert nach Restart zurück |
| `target:` im REST-Body | entity_id top-level | `target:` → HTTP 400 |
| `/api/config/config_entries/<id>` DELETE | `/entry/<id>` | Ohne `/entry/` → 404 |
| Sensor ans Dateiende in template.yaml | expliziter `- sensor:`-Block | Block-Falle → stumm verworfen |
| Neuer storage-Eintrag ohne Vorlage | Vorlage-Eintrag 1:1 lesen | Pflichtfelder-Inkonsistenz je Datei |
| Storage reload statt Restart | Vollneustart | reload liest Storage nicht neu ein — HA behält In-Memory-Stand bis Neustart (verifiziert LB 22.03.2026) |
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
| `enabled: false` in `automations.yaml` für UI-verwaltete Automation | `ha_set_entity(entity_id=..., enabled=False)` (Entity Registry) | YAML-Flag vs. Registry-State → Konflikt → Repair-Issue „konnte nicht eingerichtet werden"; `check_config` erkennt diesen Fehler NICHT |
| Kanalnachricht blind umsetzen ohne Gegencheck | Behauptungen live verifizieren (§0 Kanal-Verifikationspflicht) | Fehler der Gegenstelle pflanzen sich fort |

---


## 23. Verifikation nach Änderungen

Nach jedem nicht-trivialen Schritt den passenden API-Call wählen:

| Kontext | Verifikation |
|---|---|
| Automation geändert | `GET /api/config/automation/<id>` + `last_triggered` via `GET /api/states/automation.<name>` prüfen (Trace nur per UI/WebSocket, kein REST-Endpunkt) |
| Entity-State erwartet | `GET /api/states/<entity_id>` |
| Automation ausgelöst (Test) | `GET /api/logbook/<iso>` — Filter unzuverlässig (→ §1.6); `last_triggered` via States verlässlicher |
| Shell-Command ausgeführt | `rc=0` prüfen + stdout auf Inhalt validieren |
| Template-Sensor neu/geändert | `POST /api/template` mit Ausdruck direkt testen |
| Helper-Wert gesetzt | `GET /api/states/<helper_entity_id>` |
| Storage-Datei geändert | HA-Neustart + `read_file` + Stichprobe des geänderten Werts (verifiziert LB 22.03.2026) |
| Config-Entry geändert/gelöscht | `GET /api/config/config_entries/` → Entry vorhanden/weg? |
| YAML neu geladen | `POST /api/config/core/check_config` → valid + reload-Service + `GET /api/states/<entity>` |

---


## Referenz-Index

Bedarfsgesteuert nachladen via curl:
```bash
curl -s -H "Authorization: token <PAT>" \
  -H "Accept: application/vnd.github.raw" \
  "https://api.github.com/repos/Patch76/ha-betriebshandbuch/contents/references/<datei>"
```

| Datei | §§ | Nachladen bei |
|---|---|---|
| `references/api-storage.md` | §5, §17 | Storage-Dateien (.storage/), WebSocket API |
| `references/automations.md` | §6, §7, §8, §9, §12 | Automationen, Scripts, Szenen, Helper, Blueprints |
| `references/integrations.md` | §13, §14, §15, §16, §24, §25, §26 | Better Thermostat, Zigbee, Shelly, Telegram, Android |
| `references/recorder-stats.md` | §10, §11, §18, §19 | Recorder, Zombie-Cleanup, Statistiken, Energie-Sensoren |
| `references/yaml-templates.md` | §3, §4 | YAML-Konfiguration, Template-Sensoren |
| `references/meta.md` | §21, §22 | Analyse-Reports, CLAUDE.md-Template |

**Lookup-Kette:** Aufgabe → CLAUDE.md §-Verzeichnis (§→Thema) → dieser Index (§→Datei) → curl

## 27. §Sicht — Sechsfach-Review (Qualitätsfilter vor Publish)

### Zweck

§Sicht ist ein interner Korrekturlauf **vor** dem Veröffentlichen von PRs, Issue-Kommentaren,
Kanal-Nachrichten oder anderen Outputs. Das Ziel ist ein verbessertes Ergebnis — kein
erklärendes Widget, keine Analyse-Beschreibung für den Nutzer.

**Grundprinzip:** Jeden Filter aktiv anwenden — Filter ① immer mit Web-Recherche
und Live-Test, nicht nur gedanklich. Vermeintliches Wissen wird nicht ungetestet
weitergegeben. Jeder Filter liefert entweder „kein Fund" oder eine konkrete Korrektur.
Das verbesserte Ergebnis kommt raus, nicht die Beschreibung des Prozesses.
Widget nur wenn ≥2 substanzielle Korrekturen.

---

### Die sechs Filter

| # | Name | Kernfrage | Konsequenz |
|---|---|---|---|
| ① | Faktencheck | Was ist behauptet statt belegt? Verifiziert oder hergeleitet? | Pflicht: Web-Recherche + Live-Test gegen laufende HA-Instanz via API (curl, Python, ha-mcp). Kein „liegt auf der Hand", kein „war immer so". Ungetestetes rausnehmen oder als `[UNVERIFIZIERT]` kennzeichnen |
| ② | Black Hat | Was würde ein skeptischer Reviewer sofort angreifen? Aktiv falsifizieren — nicht nur Schwächen erwähnen | Absichern oder einräumen |
| ③ | Scope | Gehört das hierher? Gibt es das schon? Falscher Detaillevel? | Kürzen, verschieben, streichen |
| ④ | Pragmatiker | Steht das im Alltag? Live getestet oder nur strukturell hergeleitet? | Belegen oder abschwächen |
| ⑤ | Leser | Was würde jemand ohne meinen Kontext missverstehen? | Umformulieren |
| ⑥ | Compliance | CONTRIBUTING.md-konform? Repo-Style? CI/Gemini-Risiko? | Bereinigen |

**„Kein Fund" ist bei jedem Filter ein vollwertiges Ergebnis** — nicht eine Schwäche.
**② Black Hat ist der wichtigste Filter** — versuche aktiv, das Ergebnis zu widerlegen.

---

### Ablauf

1. Output erzeugen (PR-Text, Kommentar, Kanal-Nachricht)
2. §Sicht: alle 6 Filter sequenziell durchlaufen
3. Gefundene Probleme direkt korrigieren — nicht dokumentieren
4. Verbessertes Ergebnis ausgeben
5. Optional: kompakte Delta-Liste wenn Korrekturen substanziell waren

---

### Scope-Hinweis für upstream PRs

③ Scope ist bei `homeassistant-ai/skills`-PRs besonders kritisch:
- Inhalt nur wenn offizielles HA-Verhalten oder breiter Community-Konsens
- Keine MCP-Tool-Namen, keine opinionated conventions
- Kein Duplikat zu bestehendem Inhalt in anderen Referenz-Dateien

