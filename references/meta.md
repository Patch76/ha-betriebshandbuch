# Meta (Reports, CLAUDE.md-Template)

Teil des ha-betriebshandbuch — Analyse-Reports, CLAUDE.md-Template.
Nachladen: `references/meta.md`

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

