# Weak Spots — Living File

> The one file you update after every study session and every real interview.
> Not a note — a running log of where you hesitated, blanked, or got asked something
> you couldn't answer cleanly. Review it before every interview. Clear items only when
> you can explain them cold, out loud, without notes.

**How to use it:** after each recall session (see the "How Sagar studies" method), add
whatever you fumbled. After each real interview, add every question that made you pause.
Move an item to "Cleared" only when you've said the answer aloud confidently twice.

---

## 🔴 Active weak spots — drill these

*(Seed list — replace with your real ones as they surface. These are the topics the
readiness bar in the skill flags as make-or-break, so they're the default starting set.)*

| Topic | The specific thing to nail | Source note |
|-------|----------------------------|-------------|
| **XPath / locators** (asked in a real IV) | Write the table-row XPath cold; `text()` vs `.`; axes; stale-element fix | `10/07` |
| Auto-waiting limits | Name where Playwright's auto-waiting **stops** — not just that it exists | `02/01/07` |
| Storage-state gap | The coverage gap storage-state auth creates, and how you cover it | `02/02/07` |
| ThreadLocal `remove()` | *Why* `remove()` matters — thread-pool reuse leaks drivers without it | `03/02/03` |
| Implicit+explicit mixing | Why mixing them compounds unpredictably | `03/01/03` |
| `NOT IN` + NULL | Returns zero rows if the subquery has one NULL — say it unprompted | `10/02` |
| Window functions | Write `ROW_NUMBER() OVER (PARTITION BY ...)` from memory | `10/02` |
| 401 vs 403 | Not authenticated vs authenticated-but-forbidden | `10/05` |
| The forgotten `await` | The #1 TS automation bug — catch it in code review on sight | `02/04/04` |
| Flaky fix ≠ retries | Why raising retry count hides the bug instead of fixing it | `04/01` |
| Test level trade-off | "200ms API vs 10s UI" — when NOT to test through the UI | `10/04` |
| Dropdown exact methods | Native select: `selectOption({ label/value/index })`; custom dropdown: click trigger + option | `02/01/04` |
| Playwright strict-mode wording | Say `strict mode violation ... resolved to 2 elements`; fix by scoping, not blind `.first()` | `10/07/04` |
| API non-functional checks | Response time, security/auth, error quality, schema stability, sensitive data leakage | `02/03/04` |
| API token management | UI storage state vs API bearer token context; expiry handling and refresh | `02/03/05` |
| Agent context management | Skills = stable rules, task context = current story/PR/files, reviewed handoff between agents | `11/08` |
| Playwright error classification | strict mode vs timeout vs actionability vs closed context vs auth/setup failures | `02/01/09` |
| Fixtures/hooks/auth setup | Explain `base.extend`, `await use()`, test vs worker scope, hooks vs fixtures, UI storage vs API token | `02/02/12` |
| Playwright commands/scripts | Create project, run/debug/filter tests, show report/trace, explain `package.json` and `package-lock.json` | `02/05/06` |

---

## 🟡 Watch list — shaky but not urgent

*(Add topics you can do but not fluently. Half-known is more dangerous than unknown —
you'll start confidently and stall mid-answer.)*

| Topic | What's shaky | Source |
|-------|-------------|--------|
| _(add as they surface)_ | | |

---

## ✅ Cleared — proven cold, out loud, twice

*(Move items here only when genuinely fluent. Re-test occasionally — cleared can decay.)*

| Topic | Date cleared |
|-------|-------------|
| _(none yet)_ | |

---

## Post-interview capture (do this within an hour of every interview)

Every real interview is prep, not a verdict. Capture while it's fresh:

- **Questions I couldn't answer cleanly:** → add to Active weak spots
- **Questions I nailed:** → confidence markers, note them
- **Anything they asked that isn't in my notes at all:** → a coverage gap; flag it for a new note in `00_INDEX/INDEX.md`
- **Surprises about format/level/stack:** → feeds company-specific prep

---

## Key takeaway

> **This file is the highest-ROI thing in the repo, because it targets your actual gaps
> instead of re-reading what you already know.** Reading for two hours isn't preparation
> — closing the gap between "I understand this" and "I can say it cold" is, and that gap
> only shows up when you speak. Update after every session, review before every interview.
