# Git Workflow for Test Automation

## What This Covers

Branching, merge conflicts, pull requests, and the Git practices that matter when your test code lives alongside production code.

---

## Why It Matters (Interview Context)

At SDET level you're expected to work like a developer, and Git fluency is part of that. The question is usually framed practically:

> "How do you handle a merge conflict?"
> "What's your branching strategy?"

The signal they're checking: **do you treat test code as production code?** Someone who commits directly to main, or whose test repo has no PR process, is doing scripting rather than engineering.

---

## Your 30-Second Interview Answer

> "Feature branches off main, PR with review before merge, and main always green. For conflicts I'd rather understand both changes than blindly take one side — in test code a conflict usually means two people touched the same page object, and taking 'theirs' silently drops someone's locator fix. The practice I'd argue for is that test code goes through the same review bar as production code, because a test suite nobody reviews accumulates exactly the debt the notes warn about."

---

## Your 2-Minute Deep Answer

> "**Branching:**
>
> The common models are Git Flow — main, develop, feature, release, hotfix — and trunk-based, where short-lived feature branches merge into main frequently.
>
> For test automation I'd favour something close to trunk-based: short-lived branches merged often. Long-lived test branches drift badly, because the application changes underneath them and you end up merging tests written against a UI that no longer exists.
>
> ```bash
> git checkout -b feature/PROJ-1234-checkout-tests
> # ... work ...
> git add .
> git commit -m "Add checkout flow tests for discounted items"
> git push -u origin feature/PROJ-1234-checkout-tests
> # open PR
> ```
>
> Branch naming tied to the ticket matters more than it sounds — six months later, `feature/PROJ-1234-checkout-tests` tells you why the change happened and `fix-stuff` doesn't.
>
> ---
>
> **Merge conflicts — the part worth answering well:**
>
> ```bash
> git pull origin main
> # CONFLICT in pages/CheckoutPage.ts
> ```
>
> The mechanics are simple: open the file, resolve the markers, `git add`, continue. What matters is the judgement.
>
> **In test code, a conflict usually means two people edited the same page object.** That's the common case, and it's exactly where blindly resolving is dangerous — if you take 'ours' you drop their locator fix, and the test breaks for reasons nobody connects to the merge.
>
> So my approach: understand both changes before resolving. If it isn't obvious, ask the other author. `git log` on the conflicting file usually tells you what they were doing.
>
> ```bash
> git log --oneline -5 -- pages/CheckoutPage.ts
> ```
>
> **Rebase vs merge** — worth having a view. Rebasing gives a linear history and is cleaner to read; merging preserves the true shape of what happened. My rule is rebase your own unpushed work to tidy it, never rebase a shared branch, because rewriting history others have pulled causes real problems.
>
> ---
>
> **Pull requests for test code:**
>
> This is where I'd make the argument that matters. Test code deserves the same review bar as production code — and it's often skipped, because "it's just tests."
>
> What I'd look for in a PR reviewing tests is exactly the review priority list from the mentoring note: will it be flaky, does the assertion actually verify something, is it at the right level, will it survive a UI change.
>
> Keeping PRs small helps enormously. A PR that adds forty tests gets rubber-stamped; one that adds five gets read.
>
> ---
>
> **What should never be committed:**
>
> ```gitignore
> node_modules/
> test-results/
> playwright-report/
> allure-results/
> .env
> *.log
> /screenshots/
> ```
>
> **Credentials are the one that matters.** A `.env` committed once is in the history forever — removing the file in a later commit doesn't remove it from the repo. If it happens, the credential is compromised and has to be rotated, not just deleted. That's worth saying plainly because people underestimate it.
>
> `.env.example` committed with the variable names and no values is the right pattern — it documents what's needed without exposing anything.
>
> **Test artifacts shouldn't be committed either.** Screenshots, reports, and traces are generated output. They belong in CI artifacts with a retention policy, not in version control where they bloat the repo permanently.
>
> ---
>
> **Commits worth reading:**
>
> ```
> Add boundary tests for discount threshold
>
> Covers values just below, at, and above the ₹5000 threshold.
> Found that ₹5000 exactly was being treated as below — raised PROJ-1456.
> ```
>
> The body is where the value is. Six months later, that tells you why the test exists and what it caught. `fix tests` tells you nothing.
>
> ---
>
> **The commands worth knowing beyond the basics:**
>
> ```bash
> git stash                    # park work to switch branches quickly
> git cherry-pick <sha>        # take one commit onto another branch
> git bisect                   # find the commit that broke something
> git revert <sha>             # undo a commit safely, without rewriting history
> git log -S "getByTestId"     # find when a string was introduced or removed
> ```
>
> `git bisect` is genuinely useful for tests — when a test started failing sometime in the last twenty commits, bisect finds the culprit in about five steps instead of reading diffs."

---

## Quick Reference

| Task | Command |
|------|---------|
| New branch | `git checkout -b feature/TICKET-desc` |
| Update from main | `git pull origin main` |
| See what conflicted | `git status` |
| History of one file | `git log --oneline -- path/file` |
| Park work | `git stash` / `git stash pop` |
| Find breaking commit | `git bisect start` |
| Undo safely | `git revert <sha>` |
| Find when a string changed | `git log -S "text"` |

---

## Your Real Project Example (NielsenIQ)

**How to frame it:**

> "Test code lives in the repo and goes through the same branch-and-PR flow as anything else — feature branch off main, review before merge, main stays green.
>
> The thing I'd argue for is that test PRs deserve a real review rather than a rubber stamp. It's easy for teams to wave tests through on the basis that they're 'just tests', and that's exactly how a suite accumulates flaky patterns and weak assertions. What I look for in review is the same list I'd apply anywhere: will this be flaky, does the assertion actually verify something, is it at the right level.
>
> On conflicts, the common case in test code is two people editing the same page object — and that's where careless resolution does damage. Taking one side silently drops someone's locator fix, and the test then breaks for reasons nobody connects back to the merge. So I'd read both changes before resolving, and check `git log` on the file if it isn't obvious.
>
> The rule I'd state most firmly is credentials. A `.env` committed once is in history permanently — deleting it in a later commit doesn't remove it. At that point the credential is compromised and needs rotating, not just removing. `.env.example` with names and no values is the right pattern."

---

## Common Interview Questions & Answers

### Q1: "What's your branching strategy?"

**A:** "Short-lived feature branches off main, named after the ticket, merged via PR with review. For test automation I'd avoid long-lived branches specifically — the application changes underneath them, and you end up merging tests written against a UI that no longer exists."

### Q2: "How do you handle a merge conflict?"

**A:** "Mechanically it's resolve the markers and continue. The judgement part is that in test code a conflict usually means two people edited the same page object — so blindly taking one side drops someone's locator fix and breaks a test for reasons nobody links to the merge. I'd read both changes, check `git log` on the file if it's unclear, and ask the other author rather than guess."

### Q3: "Rebase or merge?"

**A:** "Rebase my own unpushed work to tidy the history, merge for shared branches. The rule I'd hold to is never rebase a branch others have pulled — rewriting shared history causes real problems. Linear history is nice; it isn't worth breaking someone else's clone."

### Q4: "Should test code go through PR review?"

**A:** "Yes, at the same bar as production code. It's commonly skipped because 'it's just tests', and that's exactly how flaky patterns and weak assertions accumulate. I'd also keep PRs small — a PR adding forty tests gets rubber-stamped, one adding five gets read."

### Q5: "What happens if credentials get committed?"

**A:** "They're compromised, and the fix is rotation, not deletion. Removing the file in a later commit doesn't remove it from history — anyone with repo access can recover it. That's why `.env` is gitignored and `.env.example` with names and no values is committed instead."

---

## Weak Areas to Watch

❌ **Don't commit directly to main**
→ Signals scripting rather than engineering.

❌ **Don't resolve conflicts by picking a side blindly**
→ In test code you'll silently drop a locator fix.

❌ **Don't rebase shared branches**
→ Breaks other people's clones.

❌ **Don't treat a committed credential as fixed by deleting it**
→ It's in history. Rotate it.

❌ **Don't commit reports, screenshots, or traces**
→ Generated output belongs in CI artifacts.

---

## Key Takeaway for Interview

Git questions check whether you treat test code as production code.

The conflict answer that shows real experience:

> **"In test code a conflict usually means two people edited the same page object. Taking one side blindly drops someone's locator fix — and the test then breaks for reasons nobody connects to the merge."**

And the one people get wrong:

> **"A committed credential isn't fixed by deleting the file. It's in history permanently — that credential is compromised and needs rotating."**

---

## Checklist

✅ Short-lived feature branches, ticket-named
✅ PR review for test code at production bar
✅ PRs kept small enough to actually be read
✅ Conflicts resolved by understanding both sides
✅ Never rebase shared branches
✅ `.env` gitignored, `.env.example` committed
✅ Generated artifacts never committed
✅ Commit bodies explain why, not just what

---

**Remember:** If your test repo has no review process, you're scripting — not engineering. 🎯
