# Prompting Patterns For QA

## 1. What it is

Prompting is how I turn a vague request into something an LLM can do well. A good QA prompt is
not clever wording; it is structure: who the model should act as, what it must do, what context it
may use, what it must not do, and what shape the answer should take.

The pattern I use, in order:

| Part | What goes in | Example |
|---|---|---|
| **Role** | The viewpoint | "Act as a senior SDET reviewing a story" |
| **Task** | One clear job | "List risks and missing acceptance criteria" |
| **Context** | Only what's relevant | Story text, AC, the PR summary, one existing test |
| **Constraints** | Boundaries | "Use only the story. If something is unknown, list it as a question. Do not invent fields." |
| **Output format** | Shape to expect | "Table: scenario, type (positive/negative/boundary), risk, automate? (Y/N)" |
| **Example (few-shot)** | One sample of the style wanted | One existing Playwright test to imitate |

## 2. Why it matters in interviews

"How do you prompt?" is a probe for whether I get consistent value from AI or just hope. The strong
answer names the structure, the constraint against inventing facts, and iteration — with a
review step at the end.

## 3. 30-second answer

"I give it a role, one clear task, only the relevant context, explicit constraints — especially 'don't
invent anything, list unknowns as questions' — and a fixed output format, plus one existing test as
an example when I want code in our style. Then I review and iterate. Structure beats clever wording."

## 4. 2-minute answer

Patterns that work for testing:

- **Constrained brainstorm.** "From this story, list scenarios grouped by risk. Mark each positive,
  negative, boundary, or role-based. Do not add scenarios outside the story." Constraint stops
  fantasy features.
- **Questions first.** "Before writing tests, list ambiguities and missing acceptance criteria."
  This turns the model into a requirement reviewer. Often the best value.
- **Few-shot style.** Paste one good existing test and say "follow this structure, fixture usage,
  locator style and naming." For Playwright that means getByRole, fixtures, and API setup, not
  invented helpers.
- **Draft then critique.** First ask for the draft; then ask the model to review its own draft against
  a checklist (assertions meaningful? data isolated? waits explicit?). A second pass catches obvious
  gaps, but it is still not a substitute for my review.
- **Step by step for debugging.** Give the failing error, the trace/console excerpt, and ask "list the
  three most likely causes in order, and what evidence would confirm each." That's better than
  "fix this".
- **Output as a table or JSON** when I'll paste it into Jira or a spreadsheet — easier to review and
  diff than prose.

Anti-patterns: pasting the whole repo; "write all tests for this page"; no format; no constraint;
accepting the first answer; asking it to decide severity or release readiness.

Iteration matters: if the answer is generic, the context was generic. I tighten context rather than
rewording the same request.

## 5. Real project example

For a BASES_PLAYWRIGHT flow, I give the manual steps, one existing spec as the style example, and
the rules: user-facing locators, API-assisted setup, no hard waits, assert business outcome not
just visibility. The draft arrives close to our conventions; I then harden it. That workflow is the
source of the verified **30%** drafting-and-maintenance gain — not the AI alone, the prompt plus the
review.

## 6. Common interview Q&A

**Q: Give me a prompt you'd use for test design.** Role + story + AC + constraints (use only the
story; unknowns as questions) + table output with type, risk, automate Y/N.

**Q: How do you stop hallucinated steps?** Ground in supplied text, forbid invention, ask for
assumptions to be listed, and verify by actually running anything it produces.

**Q: Zero-shot or few-shot?** Few-shot when output must match a house style, like test code; zero-shot
for quick brainstorming.

**Q: What if the answer is generic?** Add specific context — the actual rule, the page object,
the failing log — don't just rephrase.

## 7. Weak areas to watch

- Over-long prompts stuffed with everything — dilutes quality.
- Letting AI set priority or severity.
- Sharing sensitive data in prompts (note 18).
- Saying "prompt engineering" as a title — keep it practical: structure, context, constraints.

## 8. Key takeaway

Good prompts are structured and constrained. The constraint "don't invent — list unknowns as
questions" is the single most valuable line in any QA prompt.
