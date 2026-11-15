# Performance, Cost And Reliability Of LLM Apps

## 1. What it is

AI features have non-functional behaviour that's different from a normal API: responses are slower,
variable in length, billed per token, subject to the provider's rate limits, and dependent on an external
service that can fail. Testing this is performance and reliability testing with new metrics.

This is where my **JMeter load-testing experience** (60+ APIs, standard vs peak load, P90/P95 analysis)
carries over most directly.

## 2. Why it matters in interviews

"How would you performance-test an LLM feature?" is an easy place to show real experience: percentiles,
ramp-up, bottleneck isolation. The twist is knowing that the LLM provider, not my server, is often the
bottleneck, and that cost is a test dimension.

## 3. 30-second answer

"I load-test the service around the model with JMeter — realistic users, ramp-up, P90/P95 latency and
error rates — and separate app bottlenecks from model latency by stubbing the LLM for one run and using a
small real-model run for the other. I also track tokens and cost per request, rate-limit behaviour (429s,
retries, backoff), streaming time-to-first-token, and what happens when the provider is slow or down."

## 4. 2-minute answer

**Metrics that matter**
- **Latency:** total response time, plus **time to first token (TTFT)** for streaming UIs — users
  perceive speed from the first words appearing. Report percentiles (P50/P90/P95/P99), not just averages.
- **Throughput:** requests per second the app sustains; tokens per second generated.
- **Token usage:** input + output tokens per request — drives both cost and latency.
- **Cost per request / per user / per day** and projected monthly cost at peak.
- **Error rate:** timeouts, 5xx, **429 rate limit**, malformed outputs, empty responses.
- **Reliability:** availability when the provider is degraded; fallback behaviour.

**Load-testing approach (JMeter)**
1. **Define scenarios:** typical users, peak traffic, long-prompt users; mix of short and long
   questions (response length varies a lot).
2. **Stub the LLM** (a mock endpoint returning fixed responses with realistic delay) to find *my* system's limits —
   app servers, vector DB, database, network — without cost or provider limits.
3. **Small real-model runs** to measure true latency distribution and token cost; keep volume
   low and budgeted. Never hammer a provider beyond agreed terms.
4. **Ramp-up and soak:** gradual load to find the knee in the latency curve; longer runs for memory
   leaks, connection pools, rate-limit windows.
5. **Analyse P90/P95/P99,** error breakdown (especially 429/timeouts), and which component saturates
   first. Compare standard vs peak load — the same method I used on the 60+ APIs.

**LLM-specific behaviours to test**
- **Rate limits:** provider returns 429 — does the app retry with backoff and jitter, queue, or fail
  gracefully? No retry storms.
- **Timeouts:** what does the user see after N seconds? Is the request cancelled to save cost?
- **Streaming:** partial responses, interrupted streams, client disconnects.
- **Large inputs:** oversized prompts rejected cleanly or truncated predictably; **unbounded
  consumption** abuse (OWASP LLM10) — per-user quotas and max input/output limits enforced.
- **Caching:** repeated questions served from cache? Cache invalidation after data changes; no
  cross-user cache leaks.
- **Fallbacks:** provider outage → fallback model, cached/limited response, or clear error message.
- **Concurrency with long calls:** slow LLM calls hold connections; thread/connection pool exhaustion
  is a classic bottleneck.
- **Cost tests:** token counting accuracy, budget alerts, runaway agent loops.

**Reliability extras:** retries are idempotent for tool calls with side effects; circuit breakers; timeouts
at every hop; observability (logs/traces of prompt size, latency, model version) without storing sensitive
data.

## 5. Real project example (my actual experience)

At NielsenIQ I load-tested **60+ APIs with JMeter** — comparing standard-user load against peak
traffic, reading P90/P95 response times, reporting slow APIs, which were then optimised. The method
for an LLM endpoint is identical: scenario design, ramp-up, percentile analysis, bottleneck
isolation. Differences I'd account for: response length varies, the downstream provider has its own
limits and cost, and TTFT/token metrics are added. (JMeter is a real skill of mine; LLM load testing
itself I haven't done.)

## 6. Common interview Q&A

**Q: How would you performance-test an LLM-backed API?** JMeter scenarios with ramp-up; P90/P95 latency and
errors; stub the model to test my system, small real-model runs for true latency and cost; watch 429s and timeouts.

**Q: What's different from a normal API?** Variable response time/length, token-based cost, provider
rate limits, streaming (TTFT), and a slow external dependency that can dominate.

**Q: What is TTFT?** Time to first token — how long until the user sees the response begin; key for perceived speed.

**Q: How do you handle provider rate limits in tests?** Verify retry with backoff/jitter and graceful
degradation; don't exceed the provider's limits in load tests; use stubs for high volume.

**Q: What would you report?** Percentiles, error breakdown, throughput, token usage and cost at peak, the
saturating component, and recommendations.

## 7. Weak areas to watch

- Hammering a real provider (cost and policy violation).
- Reporting only averages.
- Blaming the app when the provider is the bottleneck, or vice versa — isolate with stubs.
- Overclaiming: say "my JMeter experience is on APIs; LLM-specific load testing is something I'd
  apply the same way."

## 8. Key takeaway

Same JMeter discipline — scenarios, ramp-up, percentiles, bottleneck isolation — plus tokens, cost,
rate limits and TTFT. Stub the model for scale; use small real runs for truth.
