# Performance Testing

## What This Covers

The performance family — load, stress, spike, endurance, scalability — what each answers, the metrics that matter, and where an SDET realistically fits.

---

## Why It Matters (Interview Context)

The reliable question:

> "What's the difference between load, stress, and performance testing?"

People blur these constantly. Answering crisply is an easy differentiator, and it takes fifteen seconds.

Beyond the definitions, performance is where **expensive production incidents originate**. A feature can pass every functional test and fall over at 500 concurrent users, or leak memory over six hours. Functional testing is structurally blind to both.

Be honest about depth. You haven't run large performance campaigns — knowing the concepts, the metrics, and where they fit is the right level. Overclaiming collapses on one follow-up about thread groups.

---

## Your 30-Second Interview Answer

> "Performance testing is the umbrella. Load runs at expected peak volume — does it hold up on a busy day. Stress pushes past the limit to find the breaking point and, more importantly, *how* it breaks. Spike hits it with sudden bursts. Endurance runs at normal load for hours to catch memory leaks. The metric detail I'd add is that percentiles matter more than averages — an average of 200ms can hide 5% of users waiting eight seconds, and those are the ones who churn."

---

## Your 2-Minute Deep Answer

> "**The family:**
>
> | Type | Load applied | Question it answers |
> |---|---|---|
> | **Load** | Expected peak | Does it hold up on a normal busy day? |
> | **Stress** | Beyond the limit | Where does it break, and how? |
> | **Spike** | Sudden sharp burst | Flash sale, viral moment, campaign launch |
> | **Endurance / Soak** | Normal, for hours | Memory leaks, resource exhaustion |
> | **Scalability** | Gradually increasing | Does adding capacity actually help? |
> | **Volume** | Large data sets | Does it cope with a huge database? |
>
> ---
>
> **Stress testing — the point most people miss:**
>
> The value is the **failure mode**, not the number.
>
> Finding that the system breaks at 2,000 concurrent users is much less useful than finding *how* it breaks:
> - Does it queue requests and slow down? → acceptable degradation
> - Does it start returning 503s? → acceptable if the client handles it
> - Does it drop connections mid-transaction and leave orphaned records? → **not acceptable**
>
> A system that gets slower under stress is fine. One that loses writes or corrupts data is a serious defect, and that only surfaces under stress.
>
> **Recovery matters too** — once load drops, does it come back on its own, or does it need a restart? A system that needs manual intervention after every spike is a 3am pager problem.
>
> ---
>
> **Endurance testing — catches what nothing else does:**
>
> A memory leak doesn't show in a ten-minute load test. It shows after six hours. Same for connection pool exhaustion, disk filling with logs, and cache growth that never evicts.
>
> This is a genuine class of production incident — the service that's fine all day and falls over every night — and soak testing is the only thing that finds it pre-release.
>
> ---
>
> **The metrics:**
>
> **Response time — and percentiles, not averages.**
>
> This is the point worth making. An average of 200ms sounds healthy. But:
>
> ```
> p50 (median):  120ms   ← half of users
> p95:           800ms
> p99:         8,000ms   ← 1 in 100 users waiting 8 seconds
> ```
>
> The average hides the p99 entirely. Those users are the ones who complain, abandon carts, and churn. **The average tells you the system is fine; the p99 tells you who's having a bad time.**
>
> **Throughput** — requests per second the system actually sustains.
>
> **Error rate under load** — a system that stays fast by returning 500s isn't fast, it's broken.
>
> **Resource utilization** — CPU, memory, connections, disk I/O. This is what tells you *why* it's slow, not just that it is.
>
> ---
>
> **Reading results — the shape that matters:**
>
> ```
> Users:      100    500   1000   1500   2000
> Response:   120ms  180ms 400ms  2.1s   timeout
> Throughput: 800/s  900/s 950/s  600/s  200/s
> ```
>
> The interesting point is between 1000 and 1500: **throughput drops while response time climbs.** That's saturation — the system is past its capacity and doing more work per request (queuing, retrying, thrashing). The knee of that curve is your real capacity, not the point where it errors.
>
> ---
>
> **Tools, at the level I'd claim:**
>
> **JMeter** — the mature standard, GUI-based test creation, heavy but capable.
> **k6** — scripts in JavaScript, developer-friendly, good CI integration.
> **Gatling** — Scala-based, strong reporting.
>
> I'd say I understand what these do and could work with them rather than claiming deep tuning experience.
>
> ---
>
> **Where I do have hands-on relevance:**
>
> Two things I'd claim honestly:
>
> **Response time assertions in the functional suite.** Both Playwright and RestAssured can assert on response time. That won't replace load testing, but it catches an endpoint that's degraded from 200ms to 2 seconds — a regression you'd otherwise not notice until users complained.
>
> ```typescript
> expect(response.time()).toBeLessThan(2000);
> ```
>
> **Performance as a CI signal.** Tracking suite execution time as a trend is the same discipline applied to the test suite itself — if it creeps up quarter over quarter, something is degrading.
>
> ---
>
> **The strategic point:**
>
> Performance testing needs a **defined target** to be meaningful. 'Is it fast enough?' is unanswerable. 'p95 under 500ms at 1,000 concurrent users' is testable.
>
> Getting that number is usually a conversation with product rather than a technical exercise — and if nobody can state it, that's the first thing to fix."

---

## Types Reference

| Type | Duration | Load | Finds |
|------|----------|------|-------|
| **Load** | Short | Expected peak | Whether normal traffic is handled |
| **Stress** | Short | Beyond capacity | Breaking point + failure mode |
| **Spike** | Very short | Sudden burst | Elasticity, recovery |
| **Endurance** | Hours | Normal | Leaks, exhaustion, growth |
| **Scalability** | Increasing | Stepped | Whether adding capacity helps |
| **Volume** | Varies | Large data | Data-scale problems |

---

## Your Real Project Example (NielsenIQ)

**How to frame it:**

> "I'd be straight that I haven't run large-scale performance campaigns — that's a specialist discipline and claiming it would collapse under a follow-up about thread group design.
>
> What I do own is response-time assertions inside the functional and API suites. Both Playwright and RestAssured let you assert on response time, and while that isn't load testing, it catches an endpoint that's quietly degraded from 200ms to two seconds. That's a regression nobody notices until users complain, and it costs nothing to check.
>
> The concept I'd bring to a conversation is percentiles over averages. An average response time of 200ms can hide five percent of users waiting eight seconds — and those are exactly the users who abandon and churn. Reporting an average as a health metric is misleading in the same way that reporting final pass rate instead of first-attempt is.
>
> And on stress testing specifically, the thing I'd emphasize is that the failure mode matters more than the number. Breaking at 2,000 users is fine information; whether it degrades gracefully or drops connections and orphans records is the question that actually determines severity."

---

## Common Interview Questions & Answers

### Q1: "Load vs stress vs performance testing?"

**A:** "Performance is the umbrella term. Load runs at expected peak volume — does it hold up on a busy day. Stress pushes beyond capacity to find where it breaks and how. Spike tests sudden bursts, endurance runs for hours to catch memory leaks. They answer different questions, so you'd run several."

### Q2: "What does stress testing actually tell you?"

**A:** "The failure mode more than the capacity number. Knowing it breaks at 2,000 users is less useful than knowing whether it queues and degrades gracefully, or drops connections mid-transaction and leaves orphaned records. Slowing down under stress is acceptable; corrupting data isn't. I'd also check recovery — does it come back by itself or need a restart?"

### Q3: "Why percentiles rather than average response time?"

**A:** "Because the average hides the tail. A 200ms average can sit alongside a p99 of eight seconds — one in a hundred users having a terrible time, and those are the ones who abandon and complain. The average tells you the system is fine; the p99 tells you who isn't."

### Q4: "You see throughput dropping while response time climbs. What's happening?"

**A:** "Saturation. Past capacity the system does more work per request — queuing, retrying, context switching — so it's serving fewer requests and each takes longer. The knee of that curve is the real capacity limit, not the point where it starts erroring."

### Q5: "How would you approach performance testing a new feature?"

**A:** "Start with the target, because 'is it fast enough' is unanswerable. Get a concrete number from product — p95 under 500ms at expected peak concurrency. Then load test at that level, stress past it to find the failure mode, and soak it if it's long-running. If nobody can state the target, that conversation is the first thing to fix."

---

## Weak Areas to Watch

❌ **Don't blur load, stress, and spike**
→ This is the reliable question here.

❌ **Don't quote averages**
→ Percentiles. Averages hide exactly the users who churn.

❌ **Don't overclaim tooling depth**
→ One question about JMeter thread groups will expose it.

❌ **Don't describe stress testing as "finding the max users"**
→ The failure mode is the point.

❌ **Don't skip the target conversation**
→ "Fast enough" is unanswerable without a number.

---

## Key Takeaway for Interview

Crisp definitions, then the insight.

The distinction:

> **"Load = expected volume. Stress = beyond the limit. Spike = sudden burst. Endurance = normal load over hours, for leaks."**

The insight that shows depth:

> **"Stress testing's value is the failure mode, not the number. Breaking at 2,000 users matters far less than whether it degrades gracefully or drops connections and orphans records."**

And the metrics point:

> **"An average of 200ms can hide a p99 of eight seconds. The average says the system is fine; the p99 says who isn't."**

---

## Checklist

✅ Load / stress / spike / endurance / scalability distinguished
✅ Stress framed as failure mode, not capacity number
✅ Recovery behaviour considered
✅ Percentiles over averages
✅ Saturation curve (throughput down + response up) recognized
✅ Response-time assertions in the functional suite
✅ Honest boundary on tooling depth
✅ Performance targets defined before testing

---

**Remember:** "Is it fast enough?" is unanswerable. "p95 under 500ms at 1,000 users" is testable. 🎯
