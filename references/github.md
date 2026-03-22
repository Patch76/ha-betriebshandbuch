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
- Branch nach Merge löschen: `DELETE /repos/Patch76/ha-betriebshandbuch/git/refs/heads/{branch}`

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

**Verifiziert auf Fork Patch76/skills + homeassistant-ai/skills, 22.03.2026.**

#### Gefährliche Elemente auf GitHub-PR-Seiten (immer sperren)

| Element | Selektor | Risiko |
|---|---|---|
| Convert-to-draft Trigger | `button.prc-Button-ButtonBase-9n-Xk.Link--muted` (Text: "Convert to draft") | Öffnet Dialog → Draft |
| Convert-to-draft Bestätigung | `button.js-convert-to-draft` | Konvertiert PR zu Draft |
| PR schließen | `button[name="comment_and_close"]` | Schließt PR |
| Close with comment | `button.js-comment-and-button` | Schließt PR mit Kommentar |

**Sicherer Comment-Submit:** `button[type=submit]` **ohne** `name`-Attribut, Text "Comment". Inline-Reply: `button.review-simple-reply-button`.

#### Pflicht-Init (IMMER als erster JS-Call auf jeder PR-Seite)

```javascript
// Pflicht-Init — IMMER zuerst aufrufen, bevor irgendein anderer DOM-Aufruf erfolgt
function safeInit() {
  // 1. window.confirm überschreiben → Delete-Dialoge auto-OK, kein blockierender Browser-Dialog
  window.confirm = () => true;

  // 2. Gefährliche Elemente verstecken (Text + Klasse + name — alle drei Varianten)
  document.querySelectorAll('a,button,form').forEach(el => {
    const t = el.textContent?.trim() || '';
    const name = el.getAttribute('name') || '';
    const cls = el.className || '';
    if (
      t === 'Convert to draft' ||
      t.includes('Close pull request') ||
      t.includes('Close with comment') ||
      t.includes('Close and comment') ||
      name === 'comment_and_close' ||
      cls.includes('js-convert-to-draft') ||
      cls.includes('js-comment-and-button')
    ) {
      el.style.display = 'none';
      el.style.pointerEvents = 'none';
      if ('disabled' in el) el.disabled = true;
    }
  });
}
safeInit();
```

**Warum `display:none` statt nur `pointerEvents:none`:** React ignoriert `disabled` und `pointerEvents` auf eigenen Event-Handlern. `display:none` verhindert auch versehentliche Scroll-Klicks.

**Warum `window.confirm = () => true`:** GitHub verwendet native `window.confirm()`-Dialoge für Delete-Bestätigungen. Diese blockieren JS und können nicht per Klick-Automation geschlossen werden — nur durch Pre-Override.

#### Element-Lookup: `getElementById` vs. `querySelector` vs. XPath

GitHub lazy-loaded Elemente (resolved/deferred Threads) erscheinen im HTML-String, aber **nicht** im DOM. Reihenfolge:

```javascript
// 1. Versuch: getElementById (schnell, schlägt bei lazy-loaded fehl)
let el = document.getElementById('discussion_r' + commentId);

// 2. Fallback: querySelector (schlägt ebenfalls fehl bei lazy-loaded)
if (!el) el = document.querySelector('[id="discussion_r' + commentId + '"]');

// 3. Fallback: XPath (findet auch Elemente die querySelector nicht findet)
if (!el) {
  el = document.evaluate(
    '//*[@id="discussion_r' + commentId + '"]',
    document, null, XPathResult.FIRST_ORDERED_NODE_TYPE, null
  ).singleNodeValue;
}

// 4. Fallback: Thread-Container per data-hidden-comment-ids öffnen, dann nochmal
if (!el) {
  const container = [...document.querySelectorAll('[data-hidden-comment-ids]')]
    .find(c => c.getAttribute('data-hidden-comment-ids')?.includes(String(commentId)));
  if (container) {
    container.setAttribute('open', '');
    await new Promise(r => setTimeout(r, 800));
    el = document.evaluate(
      '//*[@id="discussion_r' + commentId + '"]',
      document, null, XPathResult.FIRST_ORDERED_NODE_TYPE, null
    ).singleNodeValue;
  }
}
```

#### Comment abschicken (sicher)

```javascript
// Sicherer Submit — nie form.submit(), nie Enter simulieren
async function safeSubmitComment(textarea) {
  const form = textarea.closest('form');
  // Sicherer Button: type=submit, kein name, Text "Comment"
  const btn = [...form.querySelectorAll('button[type=submit]')]
    .find(b => !b.getAttribute('name') && b.textContent?.trim() === 'Comment');
  if (!btn) return 'NO_SAFE_BTN';

  // Falls React den Button noch auf disabled hält: override
  if (btn.disabled) {
    Object.defineProperty(btn, 'disabled', { value: false, writable: true });
    btn.removeAttribute('disabled');
  }
  btn.click();
  await new Promise(r => setTimeout(r, 2500));
  // Erfolg: textarea ist leer
  return textarea.value === '' ? 'OK' : 'POSSIBLE_FAILURE';
}
```

#### Textarea befüllen (React-konform)

```javascript
// execCommand ist zuverlässiger als nativeSetter für React-controlled textareas
async function fillTextarea(textarea, text) {
  textarea.focus();
  await new Promise(r => setTimeout(r, 200));
  textarea.select(); // Alles selektieren
  document.execCommand('insertText', false, text); // React erkennt dieses Event
  await new Promise(r => setTimeout(r, 300));
}
```

#### Comment löschen (eigene Comments im Upstream-Repo)

PAT-DELETE auf `homeassistant-ai/skills` → HTTP 403. Nur via Browser-DOM.

```javascript
async function deleteComment(commentId) {
  // safeInit() muss bereits aufgerufen worden sein (window.confirm = () => true)
  
  // Element finden (4-stufiger Lookup oben)
  const el = /* ... 4-stufiger Lookup ... */;
  if (!el) return 'NOT_FOUND';
  
  el.scrollIntoView({ block: 'center' });
  await new Promise(r => setTimeout(r, 500));
  
  const commentEl = el.closest('.js-comment, .review-comment, [data-body-version]');
  const details = commentEl?.querySelector('details.details-overlay, details.js-comment-edit-menu');
  if (!details) return 'NO_MENU';
  
  details.querySelector('summary')?.click();
  await new Promise(r => setTimeout(r, 500));
  
  const deleteBtn = [...details.querySelectorAll('button,a')]
    .find(b => b.textContent?.trim() === 'Delete');
  if (!deleteBtn) return 'NO_DELETE_BTN';
  
  deleteBtn.click();
  // window.confirm wurde bereits auf () => true gesetzt → kein Dialog
  await new Promise(r => setTimeout(r, 2000));
  
  // Verifikation via API (nicht DOM)
  return 'SUBMITTED — via API verifizieren';
}
```

**Nach jedem DOM-Post: Verifikation via API, nicht DOM-Zählung.**

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

### 10. GitHub Discussions — GraphQL API (kein Browser)

Discussions NIEMALS via Browser-DOM anlegen — immer GraphQL:

```bash
PAT="..."
curl -s -X POST -H "Authorization: token $PAT" \
  -H "Content-Type: application/json" \
  -d "{"query": "mutation { createDiscussion(input: {repositoryId: \"REPO_ID\", categoryId: \"CAT_ID\", title: \"TITLE\", body: \"BODY\"}) { discussion { number url } } }"}" \
  "https://api.github.com/graphql"
```

**homeassistant-ai/skills IDs (verifiziert 22.03.2026):**

| Kategorie | categoryId |
|---|---|
| Announcements | `DIC_kwDOREnDwM4C1st1` |
| General | `DIC_kwDOREnDwM4C1st2` |
| Ideas | `DIC_kwDOREnDwM4C1st4` |
| Polls | `DIC_kwDOREnDwM4C1st6` |
| Q&A | `DIC_kwDOREnDwM4C1st3` |
| Show and tell | `DIC_kwDOREnDwM4C1st5` |

Repository-ID: `R_kgDOREnDwA`

Kommentar auf Discussion:
```bash
# Node-ID der Discussion holen, dann:
curl -s -X POST -H "Authorization: token $PAT" \
  -H "Content-Type: application/json" \
  -d "{"query": "mutation { addDiscussionComment(input: {discussionId: \"DISCUSSION_NODE_ID\", body: \"TEXT\"}) { comment { id } } }"}" \
  "https://api.github.com/graphql"
```
