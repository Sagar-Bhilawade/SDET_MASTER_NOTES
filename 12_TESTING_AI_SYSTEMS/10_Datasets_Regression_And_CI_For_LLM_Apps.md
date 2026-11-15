# Datasets, Regression And CI For LLM Apps

## 1. What it is

Once I can score a single answer, I need a repeatable process: a **golden dataset** of test cases,
versioned prompts/rubrics, a regression run on every meaningful change, and thresholds that gate
releases. This is the same discipline as a regression suite plus CI — applied to a probabilistic system.

## 2. Why it matters in interviews

"How do you make this repeatable and part of CI?" separates someone who ran a one-off experiment from
someone who can own quality continuously. It's also where my existing strengths (CI/CD, regression
design, Azure DevOps and GitHub Actions) directly apply.

## 3. 30-second answer

"I build a versioned golden dataset covering normal, edge, unanswerable and adversarial cases, with
reference answers and source IDs. Any change to the prompt, model, retrieval settings or index data triggers a
regression run. A small smoke set runs on pull requests, the full set nightly, with metric thresholds
as quality gates and a report comparing against the baseline."

## 4. 2-minute answer

**Building the dataset**
- **Sources:** real user questions from logs (sanitised — remove PII), subject-matter experts, support
  tickets, requirements/AC, bugs found earlier (each production failure becomes a permanent test), and
  LLM-generated synthetic questions **that a human reviews** (never trust synthetic data unreviewed).
- **Size:** start small and curated (e.g. 50–100 well-chosen cases) and grow; coverage of categories
  matters more than raw count. Tiny sets give noisy scores.
- **Structure:** id, category, question/input, reference answer or required properties, source IDs,
  risk level, expected behaviour (answer / refuse / ask clarification), tags.
- **Quality control:** two reviewers on references, avoid datasets written only by the people who wrote
  the prompt (bias), remove duplicates, keep a held-out set you don't tune against.
- **Versioning:** dataset, prompts, rubric, judge model and metric versions are all recorded — a score
  is only comparable if these are pinned. Store them in git.

**When to run (triggers):** prompt change, model/version change (including provider-side updates),
retrieval/chunking/embedding changes, index content refresh, guardrail changes, dependency upgrades,
and on a schedule to catch silent provider drift.

**Pipeline design**
1. **PR gate (fast, cheap):** a smoke subset (high-risk and recent-bug cases); deterministic checks first,
   judge-based metrics on fewer cases. Minutes, bounded cost.
2. **Nightly/full:** entire dataset, multiple runs per case, safety/red-team set, produce a report.
3. **Release gate:** thresholds met and no regression in critical categories.
4. **Post-release monitoring:** sample production traffic, user feedback, drift dashboards; feed new
   failures back into the dataset.

**Gates and thresholds**
- Absolute: faithfulness ≥ target, unsafe-compliance = 0 on the critical attack set, refusal accuracy ≥ target.
- Relative: no metric drops more than X vs baseline; no new failures in critical cases.
- Look at **category-level** results, not just the average — a good average can hide a failing
  safety category.
- Use margins to absorb variance; treat borderline results as "re-run".

**Handling non-determinism:** low temperature in tests where valid; fixed seeds if supported;
multiple samples and pass-rate; cache stable results; pin model versions; keep the judge fixed.

**Cost and speed controls:** subset on PRs, caching of unchanged cases, cheaper judge for smoke runs, parallel
execution, budgets and alerts on token spend.

**Comparing versions (A/B):** run both on the same dataset, compare score distributions and
failure lists; for prompt changes, pairwise judging with order-swapping.

**Reporting:** per-case table (question, answer, scores, reason), category summary, diff against
baseline, trend over time — same expectations as an Allure report: a failure should be diagnosable
from the report alone.

## 5. Real project example (honest bridge)

This is closest to what I already do. BASES_PLAYWRIGHT: CI feedback loop cut from **40 to 12
minutes**, with Azure DevOps and GitHub Actions — I know the trade-offs of fast PR gates vs slower
nightly runs, parallelism, and keeping a fast signal. CIP_LEGACY: 300+ automated scenarios with a
regression-ownership mindset (90% regression coverage overall). I'd apply the same structure: smoke
on PRs, full nightly, thresholds as gates. What's new is the judge-based metrics and dataset curation, not
the pipeline.

## 6. Common interview Q&A

**Q: How do you do regression testing for an LLM app?** Versioned golden dataset, re-run on changes to
prompt/model/retrieval/data, metrics vs baseline and thresholds, category-level view.

**Q: How do you put it in CI?** Smoke subset on PRs, full run nightly, thresholds fail the build, reports
as artifacts, pinned model/judge/dataset versions.

**Q: Where does the dataset come from?** Sanitised real queries, SME input, past defects, and
human-reviewed synthetic cases.

**Q: A model update from the provider changed behaviour with no code change — how do you catch it?**
Scheduled runs of the dataset plus pinned model versions and monitoring.

**Q: How do you control the cost of evals?** Subset on PRs, caching, cheaper judge for smoke runs,
budgets.

## 7. Weak areas to watch

- Averages hiding critical failures.
- Not recording judge/model/prompt versions.
- Tuning prompts against the same dataset you report on (overfitting) — keep a held-out set.
- Unreviewed synthetic data.

## 8. Key takeaway

Treat the eval dataset like a regression suite and the metrics like quality gates: versioned, run on
every relevant change, small on PRs, full nightly, and diagnosable from the report.
