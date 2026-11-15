# Performance Results — Analysis, Percentiles And Reporting

## 1. What it is

Running the test is the easy part. The skill is reading the numbers, finding which API or layer is the
bottleneck, and reporting it so someone can act. This is where my "standard vs peak, P90/P95" work lives.

## 2. Why it matters in interviews

The common trap is "average response time was 400 ms, so it's fine." A senior answer talks about
percentiles, error rate, throughput saturation, bottleneck isolation and a clear report. This is the part
of JMeter I can speak about most credibly because I read P90/P95 on real results.

## 3. 30-second answer

"I look at percentiles — P90, P95, P99 — not just the average, because averages hide the slow tail. I
read throughput and error rate against the number of users, find where response time starts climbing
while throughput flattens — the saturation point — then isolate the slow APIs and work out the layer:
app, database, network or load generator. I report the findings with evidence, the target, the gap and a
retest after the fix."

## 4. 2-minute answer

**Metrics in a JMeter report (Aggregate Report / HTML dashboard)**

| Metric | Meaning | Why it matters |
|---|---|---|
| **Samples** | Number of requests made | Sanity check |
| **Average** | Mean response time | Hides outliers; don't rely on it alone |
| **Median (P50)** | Half of requests are faster | Typical experience |
| **P90 / P95 / P99** | 90/95/99% of requests are faster than this | The slow tail; what SLAs use |
| **Min / Max** | Extremes | Max is often a one-off outlier |
| **Std. Dev** | Spread | High = inconsistent |
| **Error %** | Failed requests (assertion failures, 4xx/5xx, timeouts) | A fast error isn't a pass |
| **Throughput** | Requests per second (and KB/s) | Capacity; flattens at saturation |
| **Latency vs Connect vs Response time** | Time to first byte vs connection vs full response | Separates server think-time from network/transfer |
| **Active threads over time** | Load profile | Match response time to load |

**Why percentiles beat averages.** Example: 95 requests at 200 ms and 5 at 5,000 ms average to ~450 ms,
which looks fine, but one user in twenty waits 5 seconds. P95 exposes it. Say it in one sentence in the
interview.

**Reading the curves**
- *Response time vs active users:* flat at low load, then a **knee** where it rises sharply — the capacity
  limit.
- *Throughput vs users:* grows, then plateaus (saturation). If users keep rising while throughput stays flat
  and response time grows, the system is the bottleneck.
- *Errors vs time:* when do errors start — at a certain user count (resource limit), or periodically
  (garbage collection, scheduled jobs)?
- *Standard vs peak comparison:* which APIs degrade first, by how much (e.g. P95 at peak vs standard).

**Finding the bottleneck (layers)**
- **Application:** CPU/memory, thread pool exhaustion, GC pauses, inefficient code, chatty endpoints.
- **Database:** slow queries, missing indexes, locks, connection pool exhaustion (SQL knowledge helps — I
  validate data with SQL on Oracle/MySQL).
- **Network/payload:** large responses, no compression, many round trips (important on mobile).
- **Dependencies:** downstream services, third-party APIs, caches.
- **The load generator itself:** if JMeter's machine is at 100% CPU, the numbers are meaningless.
- Tools: server metrics/APM, DB slow-query logs, application logs correlated by time. I report the API-level
  evidence; the dev/DevOps team usually confirms root cause with server-side tools.

**Common mistakes**
- Averages only. No warm-up. Test data reused so caches make it look fast. Missing assertions (errors
  counted as successes). No think time, producing unrealistic load. Comparing runs from different
  environments or data sizes. Heavy listeners left on. One run instead of repeating for consistency.

**Targets (SLA/SLO).** Agree acceptance criteria *before* testing, e.g. "P95 under 800 ms and error rate
under 1% at peak." Without targets, results are just numbers. Take them from product requirements or a
baseline.

**Reporting slow APIs — what a useful report contains**
1. Goal, environment, build/version, data volume, date.
2. Scenarios: standard and peak load profile (users, ramp-up, duration, think time).
3. Results table per API: samples, average, P90, P95, error %, throughput vs target.
4. Graphs: response time over time, throughput, errors.
5. Findings: the slowest APIs, when they degrade, suspected cause, severity.
6. Recommendations and a retest plan; after the fix, re-run the same scenario and show before/after.

This matches what I did: identified slow APIs from the results, reported them to the team, they were
optimised, and the improvement was visible on retest.

**Little's Law (nice to know):** concurrent users ≈ throughput × (response time + think time). It helps
estimate how many users a target request rate implies.

## 5. Real project example (true statements)

At NielsenIQ, on the mobile application's APIs, I compared **standard** and **peak** load, read **P90 and
P95**, and **reported the slow APIs** which were then **optimised**. Be ready with: one API that
degraded most, what the metric showed (P95 much higher than the average), and what you understood
about why. If you don't remember a number, describe the pattern instead: "average looked acceptable but
P95 was noticeably higher at peak."

## 6. Common interview Q&A

**Q: Why P90/P95 instead of average?** The average hides the slow tail; percentiles show what a
meaningful share of users actually experience.

**Q: What does throughput tell you?** Capacity — requests handled per second; when it plateaus while
users rise, the system is saturated.

**Q: How do you find the bottleneck?** Find the first API that degrades, check where response time rises
vs throughput, then isolate the layer with server metrics, DB queries and logs; rule out the load
generator.

**Q: Latency vs response time?** Latency is time to first byte; response time includes full download
and processing.

**Q: A test shows 0% errors and fast times but you don't trust it. Why?** Possibly missing assertions
(error pages counted as success), cached/duplicated data, no warm-up, or too-low load.

**Q: What's a good response-time target?** Defined with product/business: for example P95 under a
stated ms at peak; commonly mobile APIs aim for sub-second.

**Q: How do you know the fix worked?** Re-run the identical scenario and compare percentiles and errors
with the baseline.

**Q: What is the knee of the curve?** The load level where response time starts rising sharply — the practical
capacity.

## 7. Weak areas to watch

- Quoting invented numbers. If I don't remember, I describe the pattern.
- Claiming I found the root cause when I found *which API* was slow — the developers found why.
- Not mentioning targets/SLAs.
- Confusing latency and response time.

## 8. Key takeaway

Percentiles over averages; throughput and errors against load; find the knee, isolate the layer, report with
a target, and retest after the fix.
