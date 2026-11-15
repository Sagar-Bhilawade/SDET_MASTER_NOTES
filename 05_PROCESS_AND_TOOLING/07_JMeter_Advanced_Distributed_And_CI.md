# JMeter — Non-GUI, Distributed Runs, CI And Best Practices

## 1. What it is

Fundamentals (note 06) get a plan working. This note is what separates a quick experiment from a
repeatable performance test: running without the GUI, parameterising from the command line, generating
reports, scaling out to more machines, and putting it in a pipeline.

My direct experience: local runs on my own machine. The rest below I know as concepts and can
explain; I describe it as "how I'd do it", not "what I did".

## 2. Why it matters in interviews

"How would you run this in CI?" and "what if one machine can't generate enough load?" are standard
follow-ups for a senior SDET. Showing the model — not claiming I built the pipeline — is the honest and
strong answer.

## 3. 30-second answer

"I build and debug in the GUI, but run in non-GUI mode from the command line, writing results to a JTL
file and generating the HTML dashboard. Settings like users and duration come from properties so the same
plan runs at standard and peak load. For bigger load I'd use distributed testing with agent machines,
and in CI I'd run it non-GUI, publish the report, and fail the build if error rate or P95 breaks the
threshold."

## 4. 2-minute answer

**Non-GUI run (the command to remember)**

```bash
jmeter -n -t plan.jmx -l results.jtl -e -o report/
```

- `-n` non-GUI, `-t` test plan, `-l` results log (JTL), `-e -o report/` generate the HTML dashboard into
  an (empty) folder after the run. Add `-j jmeter.log` for the run log.
- Report from an existing JTL later: `jmeter -g results.jtl -o report/`.

**Make the plan configurable (properties)**
- In the plan, read values with `${__P(threads,10)}` (default 10 if not given).
- Pass at run time: `-Jthreads=200 -Jrampup=120 -Jduration=600`.
- One plan, many scenarios: standard vs peak by changing properties, not copying plans.

**Test data**
- *CSV Data Set Config* with enough unique rows for the load; choose *recycle/stop* behaviour on end of
  file; "share mode" decides whether threads share rows — wrong settings cause duplicate-data errors that
  look like performance problems.
- Clean up created data (delete calls or reset the environment).

**Realistic load shaping**
- *Timers* for think time; *Constant Throughput Timer* or the *Throughput Shaping Timer* plugin for target
  requests/sec; *Ultimate Thread Group* plugin for custom ramp patterns (spike, step).
- Think about **open vs closed** models: a closed model (fixed users, each waits for a reply) slows its own
  load when the server slows; real traffic doesn't wait. Throughput targets approximate open-model
  behaviour.

**Distributed testing (scale out)**
- One **controller** drives several **agent** machines running `jmeter-server`; the controller starts
  them with `-R agent1,agent2`. Each agent runs the full thread count, so total users = threads × agents.
- Why: one machine's CPU, memory or network becomes the bottleneck and distorts results.
- Needs same JMeter version, same plan/data files on agents, network access between machines, and clock
  awareness. Modern alternative: run JMeter in containers on Kubernetes (I know Docker/Kubernetes
  from my work) or use a cloud load service.

**CI/CD integration (how I'd do it)**
1. Pipeline step runs the non-GUI command (Jenkins job, Azure DevOps task, GitHub Actions step), using
   properties for environment/users.
2. Publish the JTL and HTML report as pipeline artifacts.
3. **Gate:** parse results and fail on thresholds — error rate > x%, P95 > y ms, throughput below target.
   Options: Duration/response assertions inside the plan, a script reading the JTL, the Jenkins
   Performance plugin, a Maven plugin, or Taurus (a wrapper that runs JMeter with YAML config and pass/fail
   criteria). I know these exist; I haven't built the pipeline.
4. Run a **small smoke load** on every build (cheap, catches obvious regressions) and the **full
   load nightly or pre-release** in a dedicated environment.
5. Keep a baseline and compare trends over time.

**Best practices**
- Don't run heavy listeners (View Results Tree) in real runs; write only to JTL.
- Give JMeter enough heap (`JVM_ARGS` / heap settings) for large runs; monitor the load generator itself
  (CPU/memory) — if the generator saturates, results are meaningless.
- Test in an environment that resembles production (data volume, config, infrastructure) and say so when it
  doesn't — results from a small QA environment don't transfer one-to-one.
- Warm up (caches, connection pools, JIT) and exclude the warm-up from results.
- Avoid scripting languages that are slow in samplers: prefer **Groovy in JSR223** with compilation caching
  over BeanShell.
- Keep plans in version control, parameterised, with a README on how to run.
- Coordinate with the team: shared environments, downstream dependencies, and schedules.
- Never load-test third-party or production systems without permission.

**Mobile-API specifics:** tokens expire during a long soak (build refresh into the plan); payload size and
compression matter on mobile networks; test the "chatty" screen flows (many calls per screen), not just single
endpoints.

## 5. Real project example (true statements only)

I ran JMeter locally against a test environment of the mobile application's APIs, comparing standard
and peak load, and shared the P90/P95 findings so slow APIs could be optimised. The natural next steps for
that work, which I can describe as plans: parameterise users/duration by properties, run non-GUI in a pipeline
with an error-rate/P95 gate, and use distributed agents for larger peaks.

Honest bridge for CI: "My runs were manual. I run my Playwright suites through Azure DevOps and GitHub
Actions with artifacts and gates (feedback loop 40 to 12 minutes), so I'd apply the same pipeline pattern
to a non-GUI JMeter step."

## 6. Common interview Q&A

**Q: How do you run JMeter in non-GUI mode?** `jmeter -n -t plan.jmx -l results.jtl -e -o report/`.

**Q: How do you pass different loads to the same plan?** Properties: `${__P(threads,10)}` in the plan and `-Jthreads=200` on the command line.

**Q: What is distributed testing?** A controller coordinates multiple JMeter agents (`-R`) so more users
can be generated than one machine can handle.

**Q: How would you add JMeter to CI?** Non-GUI step, artifacts for JTL/HTML, thresholds that fail the build
(error rate, P95), smoke load per build and full load nightly.

**Q: Why would results be wrong even if JMeter shows green?** Load generator saturated, test data
exhausted or duplicated, no warm-up, wrong environment, missing assertions, heavy listeners.

**Q: JMeter vs Gatling vs k6 vs LoadRunner?** JMeter: GUI, large plugin ecosystem, XML plans, heavier on
resources. Gatling/k6: code-as-tests (Scala/Java, JavaScript), lighter, CI-friendly. LoadRunner:
commercial, enterprise. I've used JMeter; I'd pick by team skills and CI needs — I'm comfortable
with TypeScript so k6-style scripting would be quick to learn.

**Q: What would you improve in your own JMeter work?** Automate it in the pipeline with thresholds and
add server-side monitoring to move from "which API is slow" to "why".

## 7. Weak areas to watch

- Claiming distributed or CI experience I don't have; say "how I'd do it".
- Not knowing the non-GUI command — memorise it.
- Ignoring the load generator's own limits.
- Treating results from a tiny environment as production truth.

## 8. Key takeaway

Build in the GUI, run non-GUI with properties, scale with agents, gate in CI on error rate and
percentiles — and be exact about which of those I've done and which I'd do.
