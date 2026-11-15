# AI Risks, Privacy And Governance For QA

## 1. What it is

Using AI in QA creates risks that are easy to forget under deadline pressure: leaking sensitive data
into a tool, trusting a wrong answer, shipping tests that prove nothing, and claiming more than I
actually did. Governance means simple habits and a review gate so the speed gain doesn't turn into
a liability.

## 2. Why it matters in interviews

Senior interviewers — especially in banks and regulated companies — probe this. They want to hear that
I know what must never go into a prompt, who is accountable for AI-assisted work, and how I keep
quality. It's also where honest claims matter: I can only talk about what I've actually done.

## 3. 30-second answer

"Four risks: data leakage, wrong-but-confident output, low-value tests, and overclaiming. I never put
production, customer or internal confidential data into an external AI tool — I use synthetic or masked
data — I keep a human review gate on everything AI drafts, I judge tests by what they prove, and I only
claim what I've really done."

## 4. 2-minute answer

**1. Data privacy.** Prompts can leave the company boundary. Never paste customer data, PII,
credentials, API keys, internal documents, or confidential project material into a public tool. Use
company-approved tools and policies. Replace real values with synthetic ones; redact logs before
sharing. This is a hard line for me, and it matches the workspace rule that internal company
material doesn't leave the machine.

**2. Wrong but confident output.** Hallucinated steps, locators, APIs, or expected results. Counter: ground
in supplied facts, run the code, verify against the real product, and treat AI output as a draft.

**3. Low-value output at scale.** AI can generate hundreds of tests quickly. More tests ≠ more
confidence: green-but-meaningless tests (visibility-only assertions, duplicated scenarios) add
maintenance and false comfort. The review gate (note 07) asks "why does this test exist and what does
it prove?"

**4. Over-trust and accountability.** The tester signs the result. If AI drafted it and it was wrong, it's
still my test. AI must not decide severity, release readiness, security risk or compliance sign-off.

**5. Security of the tools themselves.** Agents with tool access (browser, files, MCP) can be
manipulated by content they read (prompt injection) or do more than intended. Limit permissions,
prefer read-only tools, keep a human in the loop for anything destructive.

**6. Honesty.** In interviews and on the resume: no self-healing production claims, no agent
architecture I didn't run, no tools I didn't use. My real, defensible claim is practical Copilot Chat and
Playwright CLI workflows for drafting and maintenance, ~30% faster, with review gates.

**7. Licensing/IP and bias.** Be aware generated code may resemble existing code and that outputs can
reflect bias in data; review accordingly.

A lightweight governance checklist: approved tool? no sensitive data? context scoped? output verified
by running? reviewer sign-off? claim honest?

## 5. Real project example

On BASES_PLAYWRIGHT, AI help is bounded: drafting scripts and maintenance, then review gates for AI-
assisted drafts before merge. Test data is synthetic or created via API setup, not copied from
real systems. Internal NielsenIQ material is never pasted into external tools or personal repos.

## 6. Common interview Q&A

**Q: What would you never give an AI tool?** Customer or production data, PII, credentials/keys,
confidential internal documents.

**Q: How do you make sure AI-generated tests are good?** Review gate: does it prove business behaviour,
are assertions meaningful, is data isolated, is it parallel-safe, does it add regression value.

**Q: Who's accountable for AI-generated code?** I am. It's my test once it merges.

**Q: Risk of AI agents with tool access?** Over-permissioned tools and prompt injection; limit scope,
use read-only where possible, human approval for risky actions.

## 7. Weak areas to watch

- Casual "I just paste logs into ChatGPT" — a red flag. Redact, or use approved tools.
- Treating AI governance as only a privacy topic; quality and honesty count too.
- Claiming agents/self-healing I didn't build.

## 8. Key takeaway

Speed from AI is only worth keeping if data stays private, output is verified, and my claims are true.
