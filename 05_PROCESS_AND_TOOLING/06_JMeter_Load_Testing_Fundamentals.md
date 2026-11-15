# JMeter Load Testing — Fundamentals

## 1. What it is

Apache **JMeter** is an open-source tool that simulates many users sending requests to a system
(mostly HTTP APIs and web apps) so I can measure how it behaves under load: response times, throughput,
errors. I use it to answer "can this API handle standard traffic, and what happens at peak?"

**Performance testing** is the umbrella. Types worth naming:

| Type | Question it answers |
|---|---|
| **Load test** | Does it meet response-time targets at expected (standard) and peak traffic? |
| **Stress test** | Where does it break, and how does it fail/recover when pushed past capacity? |
| **Spike test** | How does it handle a sudden jump in users? |
| **Soak / endurance** | Does it stay healthy over hours (memory leaks, connection exhaustion)? |
| **Scalability / capacity** | How much more load does adding resources buy? |

My JMeter work is **load testing** of APIs (standard vs peak). I know the other types conceptually and
say so.

## 2. Why it matters in interviews

Performance appears in many SDET JDs (a Barclays JD in my tracker flagged jmeter/gatling/loadrunner as gaps).
Interviewers check whether I understand the building blocks, can read results, and can reason about
bottlenecks — not whether I used every feature.

## 3. 30-second answer

"I've used JMeter to load-test the APIs behind a mobile application — building a test plan with thread
groups, HTTP samplers, headers and tokens, assertions, and listeners — comparing standard-user load
with peak traffic and reading average, P90 and P95 response times, throughput and error rate. Slow APIs
were reported to the team and optimised afterwards. I ran it locally on my own machine."

## 4. 2-minute answer — the building blocks

A JMeter **Test Plan** is a tree:

- **Thread Group** — the virtual users. Key settings:
  - *Number of threads (users)* — concurrent virtual users.
  - *Ramp-up period* — time to start all users (avoid a sudden spike unless that's the test).
  - *Loop count / duration* — how long each user keeps going.
- **Samplers** — what a user does. Mostly **HTTP Request** (method, path, parameters, body).
- **Config Elements** — shared settings:
  - *HTTP Request Defaults* (base URL/host once),
  - *HTTP Header Manager* (Content-Type, Authorization, app-version/device headers a mobile API expects),
  - *CSV Data Set Config* (feed different users/IDs from a file — **parameterisation**),
  - *User Defined Variables*, *HTTP Cookie Manager*.
- **Pre/Post-Processors** — *JSON Extractor* or *Regular Expression Extractor* pull a value (e.g. a token or
  ID) out of one response to use in the next request: **correlation**. *JSR223 (Groovy)* for custom logic.
- **Timers** — add **think time** (pause between actions) so load resembles real users; *Constant
  Throughput Timer* to target a request rate.
- **Assertions** — pass/fail checks: *Response Assertion* (status/text), *JSON Assertion*, *Duration
  Assertion* (response-time limit).
- **Logic Controllers** — *If, Loop, Transaction, Random* — shape flow and group steps.
- **Listeners** — display results: *View Results Tree* (debug only, heavy), *Summary/Aggregate Report*,
  *Backend Listener* (to a dashboard), HTML dashboard report after the run.

**Typical workflow**
1. Pick the APIs and user journeys from real usage (what do users actually call most?).
2. Capture the requests (Postman/browser DevTools/proxy; for a mobile app, a proxy such as Charles
   or Fiddler, or the API docs/collection).
3. Build the plan; parameterise users and data; correlate tokens/IDs; add assertions.
4. Debug with one user and View Results Tree.
5. Run in **non-GUI mode** at standard load, then peak load (see next note).
6. Analyse percentiles, throughput, errors, server metrics; report slow APIs; retest after fixes.

**Standard vs peak load.** Standard = typical concurrent users/requests on a normal day (from analytics,
logs, or product estimates); peak = the busiest expected period (campaign, month-end) plus a safety
margin. The comparison shows how much headroom exists and which APIs degrade first.

**Performance test vs functional API test.** Functional: is the response correct for one request?
Performance: is it fast and stable for many at once? I do both: Postman/RestAssured/Playwright request
tests for correctness, JMeter for load.

**Testing a mobile app's APIs.** The app talks to the same backend APIs as any client, so I load the APIs
directly — no emulator or device needed in the load test. Things specific to mobile: auth tokens/refresh,
headers like app version and device ID, payload size (mobile networks), pagination, and many small
calls per screen (chatty APIs).

## 5. Real project example (what I actually did — true statements only)

- About **two months** of JMeter work at NielsenIQ, on the **APIs behind a mobile application**.
- Built test plans for **60+ APIs**; compared **standard** user load against **peak** traffic.
- Read **average, P90 and P95** response times; identified **slow APIs and reported them**; they
  were **optimised** afterwards.
- JMeter ran **locally on my machine**.

Add your own true details when you remember them (user counts, which API was slowest, what the fix
was). Only say things you can explain.

**If they ask beyond what I did** — don't invent; bridge:
- "Distributed testing?" → "I ran locally. I know the controller–agents model: the controller sends the
  plan to remote JMeter servers with `-R`, so one machine isn't the bottleneck. I'd use it for higher load."
- "CI integration?" → "Mine was manual runs. In CI I'd run non-GUI, publish the JTL/HTML report and
  fail on error-rate or percentile thresholds — same way I gate Playwright runs in Azure DevOps."
- "Server-side monitoring?" → "I focused on client-side metrics; for root cause I'd correlate with APM or
  server metrics (CPU, memory, DB slow queries, connection pool)."

## 6. Common interview Q&A

**Q: What is a thread group?** The group of virtual users: number of threads, ramp-up, loop/duration.

**Q: What is ramp-up?** The time JMeter takes to start all threads; 100 users with 100 s ramp-up
means ~1 user per second. Too short = unrealistic spike.

**Q: What is think time and why use it?** A pause between requests to mimic real users; without it the load
is unrealistically aggressive.

**Q: Parameterisation vs correlation?** Parameterisation = vary input data (CSV). Correlation = capture
dynamic values from earlier responses (tokens, IDs) and reuse them.

**Q: How do you handle authentication?** A login request first, extract the token with a JSON Extractor, add
it via a Header Manager to later requests (and refresh if it expires).

**Q: Why not run load tests in GUI mode?** The GUI uses a lot of memory/CPU and skews results; use it
to build/debug, run non-GUI.

**Q: How do you decide the number of users?** From production analytics or estimates of concurrent users,
requests per second, and peak events; start with a baseline and ramp.

**Q: What are assertions for in a load test?** To make sure fast responses are also *correct* — an error
page returned in 20 ms isn't a pass.

**Q: Load test vs stress test?** Load = expected levels; stress = beyond capacity to find the breaking point and
failure behaviour.

## 7. Weak areas to watch

- Overclaiming: I did two months of load testing locally; I don't claim distributed runs, CI pipelines or
  APM work unless I did them.
- Reporting only averages.
- Saying "I tested the app" — it was the APIs behind it.
- Running with the View Results Tree on during the real run (skews results).
- Forgetting assertions, so errors look like successes.

## 8. Key takeaway

JMeter = thread groups + samplers + config + extractors + assertions + listeners. Run non-GUI, compare
standard vs peak, read percentiles not averages — and describe only what I really did.
