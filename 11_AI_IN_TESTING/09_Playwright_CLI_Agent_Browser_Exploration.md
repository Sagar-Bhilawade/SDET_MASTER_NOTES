# Playwright CLI Agent Browser Exploration

## What it is

Playwright CLI is a command-line browser control layer designed for coding agents.

In simple words:

```text
Agent wants to inspect page
→ runs playwright-cli snapshot/find/requests/console
→ gets concise output and snapshot file
→ uses refs like e15 to click/fill/check
→ converts the explored flow into Playwright test code
```

This is different from normal Playwright Test commands.

```text
npx playwright test                  = runs the automated test suite
playwright-cli snapshot/click/find    = lets an agent explore/control a browser
```

It is also different from Playwright MCP.

```text
Playwright MCP  = agent calls MCP tools with structured tool schemas
Playwright CLI  = agent runs shell commands and reads concise outputs/files
```

For Sagar's AI testing story, the safe line is:

> "For browser exploration in the automation agent, I position it as a Playwright CLI
> agent-style workflow, not Playwright MCP."

---

## Why it matters

If you say "my agent explores the application", the interviewer can immediately ask:

- Which tool explores the browser?
- What commands does it run?
- How does it find elements?
- What is a snapshot?
- How is this different from MCP?
- How does it save tokens?
- Does it directly commit generated tests?
- How do you review the output?

This note exists so the answer does not stay high-level.

---

## 30-second answer

> "In my AI-assisted automation workflow, the browser exploration part is CLI-driven.
> The agent uses `playwright-cli` commands like `open`, `snapshot`, `find`, `click`,
> `fill`, `console`, `requests`, and `generate-locator`. The important part is that the
> CLI returns concise page snapshots with element refs like `e15`, so the agent can
> interact with the page without loading a huge browser tool schema into context. MCP is
> useful for specialized browser loops, but for a coding agent working inside a large
> repo, CLI output is more token-efficient."

---

## 2-minute answer

> "The way I explain it is: the AI agent has two jobs. First, understand the product
> flow. Second, convert that understanding into maintainable Playwright code.
>
> For the browser exploration part, I prefer Playwright CLI. The agent can open the page,
> take a snapshot, search the accessibility tree using `find`, interact with refs using
> commands like `click e15` or `fill e5`, inspect console errors, check network requests,
> and generate a candidate locator.
>
> The snapshot is important. It gives a concise accessibility tree and element refs. So
> instead of the agent carrying a huge DOM or tool response in its context, it gets a
> compact view and a local snapshot file. After the page changes, refs can become stale,
> so the agent takes a fresh snapshot before the next interaction.
>
> MCP is also browser automation for agents, but the trade-off is different. MCP exposes
> structured tools and can be good for specialized exploratory loops. CLI is better when
> a coding agent is already operating in a large repository and needs smaller command
> outputs to save context.
>
> I still do not claim that the agent independently owns test quality. It can discover
> flows and draft code, but I review locator strategy, assertions, waits, data setup, and
> cleanup. The final test must be maintainable framework code, not just a recorded flow."

---

## Real project example

The interview-safe story:

```text
Manual flow or Jira story
→ QA Knowledge Agent understands expected behavior and edge cases
→ QA Automation Agent explores UI using Playwright CLI
→ agent drafts Playwright TypeScript test
→ SDET reviews locator strategy, assertions, data, cleanup
→ only reviewed code enters the framework
```

### Basic exploration flow

```bash
playwright-cli open https://staging.example.com --headed
playwright-cli snapshot
playwright-cli find "Login"
playwright-cli click e15
playwright-cli fill e21 "user@test.com"
playwright-cli fill e22 "Password123"
playwright-cli click e30
playwright-cli snapshot
```

How to explain:

```text
snapshot gives the accessibility tree
find searches it for useful text
e15/e21/e22/e30 are element refs from that snapshot
after navigation or page update, take a new snapshot
```

### Locator discovery

```bash
playwright-cli snapshot
playwright-cli find "Submit"
playwright-cli generate-locator e18
```

Possible output becomes a candidate, not final truth:

```typescript
page.getByRole('button', { name: 'Submit' })
```

Interview line:

> "Generated locators are starting points. I still prefer role, label, test id, then
> scoped locators. I do not blindly accept whatever the agent generates."

### Debugging with console and network

```bash
playwright-cli console error
playwright-cli requests
playwright-cli request 3
```

How to explain:

```text
console error    = JS/runtime errors visible in browser console
requests         = list network calls since page load
request 3        = inspect one request/response detail
```

### Storage and sessions

```bash
playwright-cli state-save .auth/customUser1.json
playwright-cli state-load .auth/customUser1.json
playwright-cli list
playwright-cli close-all
```

This is useful for exploration, but for committed Playwright tests the framework should
still use proper `playwright.config.ts`, setup projects, fixtures, and `storageState`.

### Screenshot and evidence

```bash
playwright-cli screenshot
playwright-cli screenshot --filename=login-page.png
```

Useful when the agent needs visual evidence or when you want a lightweight artifact
during exploration.

---

## Common interview Q&A

### Q1: What exact Playwright CLI commands does your agent use?

**Answer:** The common ones are:

| Purpose | Commands |
|---|---|
| Open/control page | `open`, `goto`, `reload`, `close` |
| Inspect DOM/accessibility tree | `snapshot`, `find`, `find --regex` |
| Interact | `click`, `dblclick`, `fill`, `type`, `press`, `check`, `uncheck`, `select`, `hover`, `drag`, `upload` |
| Debug | `console`, `requests`, `request`, `eval` |
| Generate locator | `generate-locator` |
| Evidence | `screenshot`, `pdf` |
| Storage/session | `state-save`, `state-load`, `list`, `close-all` |

### Q2: What is a snapshot?

**Answer:** A snapshot is a concise accessibility-tree view of the page. It contains
roles, text, and element refs like `e15`. The agent uses those refs to interact with the
page.

### Q3: Are refs permanent?

**Answer:** No. Refs are valid for the current snapshot. After navigation, reload, or
large DOM change, take a fresh snapshot before using refs again.

### Q4: How does the agent click a button?

**Answer:** Usually:

```bash
playwright-cli snapshot
playwright-cli find "Submit"
playwright-cli click e15
```

Or with a locator:

```bash
playwright-cli click "getByRole('button', { name: 'Submit' })"
```

### Q5: Why CLI instead of MCP?

**Answer:** CLI is token-efficient for coding agents because it returns concise command
output and local snapshot files. MCP is useful, but it brings tool schemas and richer
tool outputs into the model loop. For a coding agent working inside a large automation
repo, CLI keeps more context available for code reasoning.

### Q6: Does the CLI replace Playwright Test?

**Answer:** No. CLI helps exploration and agent interaction. The final automation still
belongs in Playwright Test with `test`, `expect`, page objects, fixtures, config,
reports, and CI.

### Q7: Does the agent generate final tests directly?

**Answer:** No. It drafts. I review and harden the test: semantic locators, correct
assertions, waits, test data, cleanup, and framework structure.

### Q8: How do you prevent shallow AI-generated tests?

**Answer:** I review every generated test with a gate:

- Does it assert real behavior?
- Are locators stable?
- Is data isolated?
- Are waits handled by Playwright, not sleeps?
- Is cleanup present?
- Would this failure be useful in CI?

### Q9: What reverse question can come after "we use CLI snapshots"?

**Answer:** They may ask: "How do you search inside the snapshot?" Answer:

```bash
playwright-cli find "Add to cart"
playwright-cli find --regex "/sign (in|up)/i"
```

### Q10: What reverse question can come after "we inspect network"?

**Answer:** They may ask: "Which command?" Answer:

```bash
playwright-cli requests
playwright-cli request 2
```

---

## Weak areas to watch

- Do not mix up `npx playwright test` and `playwright-cli click/snapshot/find`. One runs
  tests; the other controls/explores the browser.
- Do not say Playwright CLI is the same as Playwright MCP. They solve related but
  different agent workflows.
- Do not say refs are permanent. Re-snapshot after page changes.
- Do not let generated locator output become final code without review.
- Do not claim self-healing or autonomous production ownership. Keep the story practical:
  exploration, drafting, review, hardening.
- Do not expose internal company data while demonstrating AI/browser exploration.

---

## Key takeaway

> Playwright CLI is how the agent explores the browser cheaply. Playwright Test is how
> the final suite runs reliably.

The line to remember:

> "The agent uses Playwright CLI for token-efficient browser exploration: snapshot,
> find, click/fill by refs, inspect console/network, then generate draft locators. I
> still review and convert that draft into proper framework code."
