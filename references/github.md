## §Git — GitHub-Arbeit (PRs, DOM, CI, Fallstricke)

### Zugriffsmatrix

| Aktion | PAT (bash_tool) | Browser-DOM |
|---|---|---|
| Datei committen (Patch76/ha-betriebshandbuch) | ✅ | — |
| Datei committen (Patch76/skills Fork) | ✅ | — |
| PR-Review-Comment posten (homeassistant-ai/skills) | ❌ 403 | ✅ |
| PR-Review-Comment löschen | ❌ 403 | ✅ |
| Issue-Kommentar posten (homeassistant-ai/skills) | ❌ 403 | ✅ |
| Issue-Kommentar löschen (homeassistant-ai/skills) | ❌ 403 | ✅ |
| GraphQL markPullRequestReadyForReview | ❌ 403 | ✅ |
| Squash-Merge (Patch76/ha-betriebshandbuch) | ✅ | — |
| CI check-runs lesen | ✅ | — |

PAT-Credentials ausschließlich in AW/Schritt 2 — nie in claude.md, github.md oder session_handoff.

---

### PR-Workflow (homeassistant-ai/skills)

1. Commit auf `Patch76/skills:<branch>` via PAT (git clone + push oder PUT)
2. Draft-PR anlegen: `POST /repos/homeassistant-ai/skills/pulls` mit `"draft": true`
3. Ready for Review: GraphQL `markPullRequestReadyForReview` via Browser-DOM
4. Nach Merge: PR auf `Patch76/ha-betriebshandbuch` aus Draft nehmen + squash-mergen

PR-Workflow (Patch76/ha-betriebshandbuch):
- Draft anlegen + direkt squash-mergen via PAT — kein Browser nötig
- `PUT /repos/Patch76/ha-betriebshandbuch/pulls/{nr}/merge` mit `merge_method: squash`

Squash-Merge macht `compare/main...{branch}` `ahead_by` unzuverlässig → PR `merged_at` als Zeitreferenz.

---

### CI (homeassistant-ai/skills validate-Job)

- Läuft auf Fork-PRs erst nach Maintainer-Workflow-Approval (normal, kein Fehler)
- Prüft: `description` ≤ 1024 Zeichen (YAML-Block, joined), YAML-Schema
- Check-Runs erscheinen verzögert (~2 Min nach Push)
- Fehler-URL: `github.com/homeassistant-ai/skills/actions/runs/<id>/job/<id>`
- `description`-Länge messen: alle Zeilen aus YAML-Block strippen + mit Leerzeichen joinen

---

### DOM-Reply-Muster (Thread-Replies ohne PAT-Write-Access)

```javascript
// Vor jedem DOM-Submit auf PR-Seite: Alle destruktiven Buttons deaktivieren
// PFLICHT vor jedem javascript_tool-Aufruf der einen Submit/Comment auslöst
document.querySelectorAll('a,button').forEach(el => {
  const t = el.textContent?.trim() || '';
  if (
    t.includes('Convert to draft') ||
    t.includes('Close pull request') ||
    t.includes('Close and comment') ||
    el.name === 'comment_and_close' ||
    el.getAttribute('data-action')?.includes('close')
  ) {
    el.style.pointerEvents = 'none';
    el.disabled = true;
  }
});

// Reply auf Thread mit bekannter comment_id
async function replyToThread(commentId, text) {
  const el = document.getElementById('discussion_r' + commentId);
  if (!el) return 'NOT_FOUND';
  el.scrollIntoView({ block: 'center' });
  await new Promise(r => setTimeout(r, 500));
  // Reply-Div im Parent-Baum suchen (bis 8 Ebenen)
  let node = el, replyDiv = null;
  for (let i = 0; i < 8; i++) {
    node = node.parentElement;
    if (!node) break;
    replyDiv = node.querySelector('div.review-thread-reply');
    if (replyDiv) break;
  }
  if (!replyDiv) return 'NO_REPLY_DIV';
  replyDiv.querySelector('.review-thread-reply-button').click();
  await new Promise(r => setTimeout(r, 900));
  const form = replyDiv.querySelector('form.js-inline-comment-form');
  const textarea = form?.querySelector('textarea');
  if (!textarea) return 'NO_TEXTAREA';
  const setter = Object.getOwnPropertyDescriptor(HTMLTextAreaElement.prototype, 'value').set;
  setter.call(textarea, text);
  textarea.dispatchEvent(new Event('input', { bubbles: true }));
  await new Promise(r => setTimeout(r, 400));
  const btn = form.querySelector('button.btn-primary') ||
              [...form.querySelectorAll('button')].find(b => b.textContent.trim() === 'Comment');
  if (!btn || btn.disabled) return 'NO_SUBMIT';
  btn.click();
  await new Promise(r => setTimeout(r, 2500));
  return 'OK';
}
```

Duplikat-Prüfung nach DOM-Posts immer via API — nicht DOM zählen:
```bash
curl -s -H "Authorization: token $PAT" \
  "https://api.github.com/repos/homeassistant-ai/skills/pulls/NR/comments?per_page=100" | python3 -c "
import json,sys
cs=json.load(sys.stdin)
replied=set(c['in_reply_to_id'] for c in cs if c['user']['login']=='Patch76' and c.get('in_reply_to_id'))
roots=[c for c in cs if not c.get('in_reply_to_id') and c['user']['login']!='Patch76']
for r in roots:
    print('✅' if r['id'] in replied else '❌', r['id'], r['user']['login'])
"
```

---

### Review-Regeln

- **PR-Vollcheck-Reihenfolge:** Bei jeder PR-Review-Session vollständig prüfen:
  1. Inline-Threads: alle Root-Threads (Gemini + Maintainer) beantwortet?
  2. Issue-Level-Comments: Duplikate? Fehlende Antworten?
  3. Verlinkte Issues: `Fixes #N` / `Closes #N` aus PR-Body extrahieren → Issue öffnen → letzten Comment auf Aktualität prüfen
  - Issues bleiben offen bis Merge (GitHub schließt sie automatisch) → kein Handlungsbedarf, aber Inhalt muss aktuell sein
- **Thread-Reply-Pflicht**: Jeder Root-Thread (Gemini + Maintainer) muss direkt beantwortet werden — auch wenn Resolved
- **Reply-Formel**: Was war das Problem? Welcher Commit behebt es?
- **Emoji-Regel**: Nur bei anerkennender Kommentierung (empfangend und gebend), sparsam. Korrektive Replies: kein Emoji
- **Duplikate**: DOM-Posts können bei Timeouts doppelt landen — immer API-Verifikation
- **Resolved ohne Reply**: Aus 3rd-Person-Sicht unklar ob verstanden → immer Reply, dann Resolve
- **Zurückhaltung bei fremden PRs**: Nur kommentieren wenn echter, nicht bereits gedeckter Mehrwert vorhanden. Redundante Bestätigungen (z.B. „action: ist korrekt" wenn bereits im Thread beantwortet) sind Noise — schaden dem professionellen Eindruck.

---

### Fallstricke-Tabelle

| Fallstrick | Symptom | Lösung |
|---|---|---|
| PAT 403 auf homeassistant-ai/skills | Alle Schreibops scheitern | Browser-DOM nutzen |
| Convert-to-draft-Falle | Dialog öffnet sich unbeabsichtigt beim Scrollen | Link vor Automation deaktivieren (s.o.) |
| DOM-Submit schließt PR versehentlich | `button[name=comment_and_close]` wird statt Comment-Button geklickt → PR geschlossen | Pflicht-Disable-Snippet (s.o.) **vor jedem Submit** — deckt alle 3 Varianten ab: Convert to draft, Close pull request, Close and comment |
| DOM-Timeout mit stiller Ausführung | JS-Fehler, aber Post wurde dennoch abgeschickt | Immer API-Verifikation danach |
| Textarea nicht befüllbar via `.value = x` | Input-Event fehlt | `nativeInputValueSetter` + `dispatchEvent('input')` |
| Orphan-Kommentare | Reply landet auf PR-Ebene statt im Thread | Direkt auf `discussion_r<id>` navigieren und replyToThread() nutzen |
| base64-PUT zu langsam für große Dateien | Timeout | git clone + push via bash_tool |
| Tab-IDs invalidieren zwischen Calls | Tool-Fehler | `tabs_context_mcp` vor jedem Browser-Call |
| `.` tippen auf GitHub öffnet github.dev | Web-Editor öffnet sich | `javascript_tool` + nativeInputValueSetter statt type-Action |
| Gemini re-review nicht automatisch | Kein neuer Review-Durchlauf | `/gemini review` als PR-Kommentar posten |
| Fork-PR CI wartet | `1 workflow awaiting approval` | Maintainer-Approval abwarten — kein Handlungsbedarf |
| Ghost-Comment: API 404, DOM rendert "Sorry, something went wrong" | Comment nicht im DOM auffindbar, kein Delete-Menü | Immer zuerst via API prüfen ob Comment noch existiert; 404 = bereits gelöscht |
| Issue-Kommentar löschen scheitert mit 403 | Eigener Kommentar im Fremd-Repo nicht via PAT löschbar | Browser-DOM: 3-Punkt-Menü im Comment nutzen |
| resolveReviewThread FORBIDDEN via PAT | Upstream-Repo erlaubt keine Thread-Resolves via PAT | Browser-DOM: „Resolve conversation"-Button im Thread-Container klicken |
| Outdated Threads nicht im Files-Tab | discussion_r<id> nicht gefunden | Conversation-Tab öffnen; details-Elemente mit summary=Dateiname per JS öffnen (details.open=true) |
| Textarea auf Issue-Seite hat andere ID | id="new_comment_field" funktioniert nicht | Textarea über placeholder suchen: [...document.querySelectorAll('textarea')].find(t => t.placeholder?.includes('Markdown')) |
| nativeInputValueSetter Illegal invocation | HTMLTextAreaElement.prototype direkt liefert Fehler | window.HTMLTextAreaElement.prototype verwenden: Object.getOwnPropertyDescriptor(window.HTMLTextAreaElement.prototype, 'value').set |
| Rebase: alter version-bump Commit konfliktet | Veralteter Commit setzt version:"1.1.1" — kollidiert mit upstream version:2 | git rebase --skip für diesen Commit; der neuere Fix-Commit (version:2) ist bereits enthalten |

---

### Analyse-Prinzipien (verifiziert 18.03.2026)

**Vor jeder Analyse: live verifizieren, nicht annehmen.**
Konkret für upstream-Repos:
- CI-Workflow-Dateien immer lesen (`/.github/workflows/*.yml`) — Verhalten nie aus Beschreibungen ableiten
- `metadata.version` auf upstream main live abfragen — nie schätzen
- Fork-Divergenz messen: `GET /repos/{fork}/compare/{fork_sha}...{upstream}:main`
- CONTRIBUTING.md live laden bevor ein PR erstellt oder bewertet wird

**CI-Versionsmechanismus homeassistant-ai/skills (verifiziert 18.03.2026):**
- `validate-skills.yml`: blockiert PR wenn `version` im Branch ≠ `version` auf upstream main (Basis des PRs)
- `auto-bump-version.yml`: erhöht nach Merge automatisch `version + 1` auf main
- Korrekte Vorgabe im PR-Branch: `version:` auf denselben Integer-Wert wie upstream main setzen, nie manuell erhöhen
- Aktueller Wert upstream main: `version: 2` (Stand 18.03.2026)
- Extraktionslogik: erste Ganzzahl aus `version:`-Zeile im Frontmatter (d.h. `"1.1.1"` → `1` → falsch!)

**CONTRIBUTING.md-Prinzipien (homeassistant-ai/skills, Stand 18.03.2026):**
- Keine MCP-Tool-Namen (`ha_rename_entity`, `ha_get_integration` etc.) — immer HA-Konzept + REST-Endpoint
- Keine opinionated conventions (Namenskonventionen, Stil-Präferenzen) — gehören in persönliche Skills oder `CLAUDE.md`
- Nur Guidance die offizielles HA-Verhalten oder breiten Community-Konsens abbildet

---

## Pre-Submission Checklist

Run before every PR — in this order. Stop and fix before pushing.

### 1. Sprache / Language

```bash
grep -rn "Neustart\|Einstellung\|Geräte\|Schalter\|und \|oder \|aber " \
  skills/home-assistant-best-practices/
```

No non-English word in English documentation. Exception: German entity IDs in explicitly
marked real-world examples (`# German example`).

### 2. Code-block language specifiers

| Content | Specifier |
|---|---|
| HTTP requests (`GET /api/...`) | ` ```http ` |
| JSON responses | ` ```json ` |
| HA tool calls (`ha_get_integration(...)`) | ` ``` ` (none) |
| Shell commands | ` ```bash ` |
| YAML configuration | ` ```yaml ` |

### 3. Factual claims — source rule

Every normative expression (`must`, `required`, `mandatory`, `always`, `never`) needs
one of:
- **Live verification** on a running HA instance (state the version), **or**
- **Source link** (HA Docs, HA source, official changelog)

Without either: downgrade to `recommended` / `when X is needed`.

### 4. Example symmetry

Every `AVOID` example must have the same field structure as the `CORRECT` example —
only the incorrect value or structure varies. Missing fields in the AVOID example
will draw a Gemini comment every time.

### 5. Anchor links

**WICHTIG: Immer `github-slugger` via npm ausführen — nie eigene Implementierung.**

```bash
cd /tmp && npm install github-slugger 2>/dev/null
node -e "import('github-slugger').then(({default:G})=>{const s=new G();console.log(s.slug('Heading hier'));})"
```

Hintergrund (verifiziert 21.03.2026): Eigene Regex-Implementierungen sind fehleranfällig.
`" — "` (Space+Em-Dash+Space) ergibt `"--"` (Doppel-Dash) — Em-Dash entfernt, beide Spaces bleiben als `-`.

```bash
# Extract all anchors generated by headings in the branch
grep -rh "^#" skills/home-assistant-best-practices/ | \
  sed 's/^#* *//' | \
  tr '[:upper:]' '[:lower:]' | \
  sed 's/[^a-z0-9 -]//g; s/ /-/g'

# All anchors referenced in SKILL.md
grep -o '#[a-z0-9-]*' skills/home-assistant-best-practices/SKILL.md
```

Cross-check every referenced anchor against the generated heading slugs.

**Nach jedem Titelumbenennen:** Alle SKILL.md-Anchor-Links auf die geänderte Datei
sofort nachziehen — Titelumbenennung ändert den Slug automatisch:
```bash
# Beispiel: Titel geändert → neuen Slug prüfen
node -e "import('github-slugger').then(({default:G})=>{const s=new G();console.log(s.slug('Neuer Titel'));})"
# Dann in SKILL.md ersetzen:
grep -n "#alter-slug" skills/home-assistant-best-practices/SKILL.md
```
Nicht erst beim nächsten PR entdecken — jeder force-push dismissed eine Maintainer-Approval.

### 6. Open PRs — overlap check

```bash
gh pr list --repo homeassistant-ai/skills --state open
```

For each open PR: read the branch diff (`compare/main...{branch}`).
Overlapping sections or files → coordinate or explicitly remove from this PR.

### 7. Tool-agnostic language

Skills describe **what** (HA concept, REST endpoint, storage path).
MCP tools (`ha_rename_entity`, `ha_get_integration`, etc.) appear **only** in an
indented blockquote as an alternative:

```
> **ha-mcp alternative:** `ha_get_integration(domain=group)` —
> note: does not expose `options.entities`.
```

Never as the primary path. The skill must work completely without ha-mcp.

**Pflicht-Grep vor jedem Push:**
```bash
grep -rn "ha_rename_entity\|ha_get_integration\|ha_config_set\|ha_config_get\|\bha_[a-z_]*\b\|python_transform\|write_file" \
  skills/home-assistant-best-practices/
```
Trifft: sofort ersetzen durch HA REST-Endpunkt oder generische Beschreibung.
Erlaubt nur in eingerückten `> **ha-mcp alternative:**`-Blöcken.

### 8. SKILL.md framing for opinionated content

Any reference row in SKILL.md that links an opinionated convention must carry:

```
(optional — not an official HA standard)
```

or equivalent wording. Framing is part of the documentation.

### 9. Stateful APIs — document the exit path

For every API with flow or session state (Options Flow, Config Flow):
document the abandon path directly after the warning:

```
> ⚠️ Only one active Options Flow per Config Entry. Abandon before retrying:
> `DELETE /api/config/config_entries/options/flow/<flow_id>`
```
