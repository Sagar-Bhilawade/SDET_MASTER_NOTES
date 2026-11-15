# AI For Failure Analysis And Debugging

## 1. What it is

When a test fails, the expensive part is finding *why*. AI can speed up the reading — summarising a
long stack trace, a trace.zip finding, a console error, a failed network call, or a CI log — and can
propose ranked hypotheses. It cannot see my environment or know which hypothesis is true. Debugging
stays an evidence game: I give it evidence, it gives candidate causes, I test them.

## 2. Why it matters in interviews

"Passes locally, fails in CI" is a classic question. The AI follow-up checks that I use AI as a
hypothesis generator with evidence, not as an oracle that says "just add a wait".

## 3. 30-second answer

"I feed it evidence — the error, the trace or screenshot summary, console and network failures, the
CI log, what changed — and ask for ranked likely causes and what evidence would confirm each. Then I
verify with a rerun, trace, or a targeted change. It speeds up triage; it doesn't replace root-cause
analysis, and I never accept 'add a retry' as a fix."

## 4. 2-minute answer

A workflow that works:

1. **Collect evidence first.** Exact error text, failing step, trace/screenshot/video, console and
   network log, CI environment differences (resources, parallelism, data), recent commits.
2. **Ask for hypotheses, not fixes.** "Given this, list the three most likely causes in order, and for
   each tell me what log line or experiment would confirm or kill it."
3. **Test the top hypothesis.** Reproduce with the CI conditions (headless, same workers, same
   data), read the trace, check network timing.
4. **Fix the cause, keep the lesson.** Timing issue → wait on the right condition; data collision →
   isolate data; locator ambiguity → user-facing locator; environment → fix the environment.

Typical help: explaining a Playwright strict-mode violation or a Selenium StaleElementReferenceException
and which line causes it; spotting that a click was intercepted by an overlay; noticing a 401/403 in
network log; comparing two logs (passing vs failing) for the first difference; summarising a 5,000-
line CI log into the first real error.

What AI gets wrong: it guesses plausibly when evidence is missing; it recommends retries/sleeps; it can't
see flaky infrastructure; it may blame the test when the product has a real bug. So the question
"is this a test bug or a product bug?" is mine to answer from evidence.

Flakiness link: retries hide the bug (04_TEST_ENGINEERING). AI that suggests "increase timeout" is
suggesting a retry in disguise unless the evidence shows genuine slowness.

## 5. Real project example

In CIP_LEGACY, replacing hard sleeps with dynamic Fluent Waits cut flaky tests to **under 2%**; that fix
came from understanding the synchronisation cause, not from a retry. With AI in the loop I would
use it the same way: paste the failure and trace, get ranked causes ("hard sleep races the render" is
a classic top candidate), then verify by reproducing. In BASES_PLAYWRIGHT, trace viewer plus CI logs
are the evidence I would hand to it.

## 6. Common interview Q&A

**Q: How do you use AI to debug a flaky test?** Give evidence, ask for ranked hypotheses with
confirm/kill checks, verify by reproduction, fix the root cause, don't add blind retries.

**Q: What can't AI tell you?** Whether the failure is a product bug, what's different about CI infra,
or what data state existed. Those need my investigation.

**Q: Can AI fix flaky tests automatically?** It can suggest changes; without a review gate it often
masks the problem. I don't claim self-healing in production.

## 7. Weak areas to watch

- Pasting logs with secrets or customer data into an external tool (note 18).
- Accepting "add a wait/retry" without evidence.
- Treating one plausible explanation as proven.
- Claiming self-healing selectors (not on my resume; never).

## 8. Key takeaway

AI shortens the time to a good hypothesis. Evidence and verification — and the decision of
test bug vs product bug — stay with me.
