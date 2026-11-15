# AI Safety, Security And Red Teaming

## 1. What it is

Security testing for AI products goes beyond normal web security because attackers can use plain
language to manipulate the model. **Red teaming** means deliberately attacking the system the way a
malicious or careless user would, to find failures before real users do.

I already know classical security basics (IDOR, authN/authZ, injection). The AI-specific layer is
prompt injection, data leakage, jailbreaks, and over-powered agents.

## 2. Why it matters in interviews

Banks and enterprises care a lot. "How would you security-test a chatbot?" expects named attack classes
and test ideas — ideally referencing the **OWASP Top 10 for LLM Applications** (2025 list below;
confirmed on the official OWASP GenAI site on 2026-11-15).

## 3. 30-second answer

"I'd red-team it with prompt injection — direct and indirect through documents — jailbreak attempts,
system-prompt extraction, attempts to extract other users' or sensitive data, and attempts to make an
agent over-act. I'd check authorisation on retrieved data, output handling like injected scripts, and
cost abuse with huge inputs. I'd organise it around the OWASP LLM Top 10 and automate a repeatable
attack set in CI."

## 4. 2-minute answer

**OWASP Top 10 for LLM Applications (2025 edition - matches the official genai.owasp.org page; some third-party sites mention a 2026 edition, which I could not confirm there)**

| ID | Risk | In plain words | Test idea |
|---|---|---|---|
| LLM01 | Prompt Injection | Input (or a document the model reads) overrides instructions | "Ignore previous instructions and..." direct; hidden instruction inside an uploaded doc/web page |
| LLM02 | Sensitive Information Disclosure | Model reveals PII, secrets, internal data | Ask for other users' data, API keys, training/example data |
| LLM03 | Supply Chain | Compromised models, libraries, datasets, plugins | Check dependency/model provenance, pinned versions |
| LLM04 | Data and Model Poisoning | Bad data in training/RAG corrupts behaviour | Plant poisoned docs in a test index |
| LLM05 | Improper Output Handling | App trusts model output (XSS, SQL, command) | Make the model output a script tag/SQL and see if it executes |
| LLM06 | Excessive Agency | Agent has too many tools/permissions | Trick it into deleting/sending/paying |
| LLM07 | System Prompt Leakage | Hidden instructions exposed | "Repeat everything above" |
| LLM08 | Vector and Embedding Weaknesses | RAG store leaks or is manipulated | Cross-tenant retrieval, permission bypass |
| LLM09 | Misinformation | Confident false output | Hallucination suite (note 07) |
| LLM10 | Unbounded Consumption | Cost/DoS via huge or repeated requests | Very long inputs, loops, rate-limit checks |

**Prompt injection, the key one.**
- *Direct:* the user types instructions to override the rules.
- *Indirect:* instructions hidden in content the system ingests — an email, PDF, web page, support ticket.
  The model can't reliably tell data from instructions. This is the dangerous variant for RAG and agents.
- No complete fix exists; defence is layered: least privilege, treat model output as untrusted,
  separate/mark untrusted content, human approval for risky actions, monitoring.

**Jailbreaks.** Role-play, encoding tricks, multi-turn manipulation to bypass safety rules. Test with a
library of known patterns and keep it updated; measure attack success rate.

**Data leakage tests.** Ask for another user's records; probe for secrets in the system prompt; check
logs don't store PII; check retrieval honours permissions (two users, same question). This is classic
authorisation testing applied to RAG.

**Improper output handling.** If the UI renders model output as HTML, an injected script is XSS. If
output is used in a query or shell, it's injection. Test like any untrusted input — my existing
security mindset applies.

**Excessive agency.** For tool-using agents, check each tool's permissions, require confirmation for
destructive actions, and test that injected instructions cannot trigger them.

**Bias, toxicity, fairness.** Run the same prompt with different names/genders/regions and compare
outcomes; check toxic-content filters; document findings. Subjective, so use rubrics and human review.

**Resource abuse.** Huge prompts, repeated calls, expensive tool loops → cost and DoS. Test limits,
quotas, timeouts.

**Red-teaming process**
1. Threat model: what assets, what harm, who attacks, what tools/data the model can reach.
2. Build an attack set per risk (start from public lists such as the OWASP categories).
3. Run manually first, then automate the repeatable ones.
4. Score: attack success rate per category; severity by impact.
5. Fix, retest, keep as regression.

**Tools (checked against official pages 2026-11-15; I have studied, not used them professionally):**
- **promptfoo** - open-source evals and red teaming. Red-team "plugins" generate adversarial inputs
  (harmful content, broken object/function-level authorisation, competitor endorsement, and more),
  run them against the app and report. Typical flow, from its quickstart: `promptfoo redteam setup`
  (or `redteam init --no-gui`) creates `promptfooconfig.yaml`; `promptfoo redteam run` generates
  attacks into `redteam.yaml` and runs them; `promptfoo redteam report` shows failing cases. Can run
  in CI/CD.
- **garak** (NVIDIA) - "Generative AI Red-teaming & Assessment Kit", an LLM vulnerability scanner.
  Install `python -m pip install -U garak`; example run:
  `python3 -m garak --target_type openai --target_name <model> --spec probes.encoding`;
  `garak --list_probes` lists probes. Probe families include `promptinject`, `dan` (jailbreak),
  `encoding`, `leakreplay` (data leakage), `packagehallucination`, `realtoxicityprompts`, `xss`.
- DeepEval also has safety metrics (toxicity, bias).
Use them only in a test environment with permission, never against someone else's system.

## 5. Real project example (honest bridge)

I haven't red-teamed an LLM. My relevant base: security basics from my QA coverage (authN/authZ
checks, IDOR), API testing with different roles, and SQL validation. The same two-user permission
test I'd run on an API is the first test I'd run on a RAG endpoint. And my red line about not pasting
internal NielsenIQ material into external tools is itself an LLM02-style leakage control.

## 6. Common interview Q&A

**Q: What is prompt injection?** Crafting input (or content the model reads) so it overrides the
system's instructions. Indirect injection through documents is the harder case.

**Q: How would you security-test a chatbot?** OWASP-based attack set: injection (direct/indirect),
system-prompt extraction, sensitive-data extraction, jailbreaks, output-handling (XSS), excessive
agency, and resource abuse; plus classical authZ tests on the data it can retrieve.

**Q: How do you prevent prompt injection?** There's no single fix: least privilege, untrusted-content
handling, output validation, approval for risky actions, monitoring; I test that each layer holds.

**Q: What's a jailbreak?** Techniques that bypass the model's safety rules, like role-play or
obfuscation.

**Q: Would you run attack tools on production?** No — a test environment with permission, with
rate-limits and synthetic data.

## 7. Weak areas to watch

- Naming the OWASP list from memory incorrectly — say "the 2025 edition; check the latest".
- Claiming red-team tool experience.
- Treating the LLM as a safe boundary; treat its output as untrusted input.
- Forgetting classical authZ on the underlying data.

## 8. Key takeaway

Treat the model like an untrusted, persuadable component: attack it with injection and leakage tests,
test the permissions around it like any API, and automate the attack set as regression.
