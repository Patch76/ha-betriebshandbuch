## §Git — GitHub-Arbeit (PRs, DOM, CI, Fallstricke)

### Zugriffsmatrix

| Aktion | PAT (bash_tool) | Browser-DOM |
|---|---|---|
| Datei committen (Patch76/ha-betriebshandbuch) | ✅ | — |
| Datei committen (Patch76/skills Fork) | ✅ | — |
| PR-Review-Comment posten (homeassistant-ai/skills) | ❌ 403 | ✅ |
| PR-Review-Comment löschen | ❌ 403 | ✅ |
| GraphQL markPullRequestReadyForReview | ❌ 403 | ✅ |
| Issue-Kommentar (homeassistant-ai/skills) | ❌ 403 | ✅ |
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
// Vor jedem Scroll/Button-Suche: Convert-to-draft-Link deaktivieren
document.querySelectorAll('a,button').forEach(el => {
  if (el.textContent?.includes('Convert to draft')) el.style.pointerEvents = 'none';
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
curl -s -H "Authorization: token $PAT"   "https://api.github.com/repos/homeassistant-ai/skills/pulls/NR/comments?per_page=100" | python3 -c "
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

- **Thread-Reply-Pflicht**: Jeder Root-Thread (Gemini + Maintainer) muss direkt beantwortet werden — auch wenn Resolved
- **Reply-Formel**: Was war das Problem? Welcher Commit behebt es?
- **Emoji-Regel**: Nur bei anerkennender Kommentierung (empfangend und gebend), sparsam. Korrektive Replies: kein Emoji
- **Duplikate**: DOM-Posts können bei Timeouts doppelt landen — immer API-Verifikation
- **Resolved ohne Reply**: Aus 3rd-Person-Sicht unklar ob verstanden → immer Reply, dann Resolve

---

### Fallstricke-Tabelle

| Fallstrick | Symptom | Lösung |
|---|---|---|
| PAT 403 auf homeassistant-ai/skills | Alle Schreibops scheitern | Browser-DOM nutzen |
| Convert-to-draft-Falle | Dialog öffnet sich unbeabsichtigt beim Scrollen | Link vor Automation deaktivieren (s.o.) |
| DOM-Timeout mit stiller Ausführung | JS-Fehler, aber Post wurde dennoch abgeschickt | Immer API-Verifikation danach |
| Textarea nicht befüllbar via `.value = x` | Input-Event fehlt | `nativeInputValueSetter` + `dispatchEvent('input')` |
| Orphan-Kommentare | Reply landet auf PR-Ebene statt im Thread | Direkt auf `discussion_r<id>` navigieren und replyToThread() nutzen |
| base64-PUT zu langsam für große Dateien | Timeout | git clone + push via bash_tool |
| Tab-IDs invalidieren zwischen Calls | Tool-Fehler | `tabs_context_mcp` vor jedem Browser-Call |
| `.` tippen auf GitHub öffnet github.dev | Web-Editor öffnet sich | `javascript_tool` + nativeInputValueSetter statt type-Action |
| Gemini re-review nicht automatisch | Kein neuer Review-Durchlauf | `/gemini review` als PR-Kommentar posten |
| Fork-PR CI wartet | `1 workflow awaiting approval` | Maintainer-Approval abwarten — kein Handlungsbedarf |
