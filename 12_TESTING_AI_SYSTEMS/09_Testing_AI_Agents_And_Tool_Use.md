# Testing AI Agents And Tool Use

## 1. What it is

An **agent** is an LLM given a goal and tools (search, database query, API calls, send email, browser)
and allowed to loop: think → pick a tool → see the result → decide the next step → finish. A plain chatbot
answers; an agent **acts**. That makes testing harder: there's a sequence of decisions, side effects,
and many possible paths to the same goal.

(Not the same as my own agent-style CLI workflows for drafting tests in note 11 — that's using agents.
This note is about testing agents that are a product feature.)

## 2. Why it matters in interviews

"Agentic" features are the new wave, and I applied to a role about QA for an MCP server — this topic is
live in the market. Interviewers check whether I can evaluate behaviour across steps, not just final text.

## 3. 30-second answer

"For an agent I evaluate three things: did it reach the right outcome, did it take a sensible path —
correct tools, correct arguments, no wasted or looping steps — and did it stay safe, meaning no
unauthorised or destructive actions. I make tools deterministic with mocks for most tests, keep a set of
scenario goals with expected outcomes, and test the tools themselves like APIs."

## 4. 2-minute answer

**Parts of an agent** (each is a test target): the LLM (planner), the tools (functions/APIs, often exposed
through a standard like **MCP — Model Context Protocol**), memory/context, and the control loop with
stop conditions.

**Failure modes**
- Picks the **wrong tool** or no tool.
- Right tool, **wrong arguments** (bad IDs, wrong date format, missing required field).
- **Loops** or takes far too many steps; never terminates.
- Skips required steps (doesn't confirm, doesn't verify).
- Mishandles **tool errors** (timeouts, 4xx/5xx) — gives up, retries forever, or invents results.
- **Hallucinates tool results** instead of calling the tool.
- **Excessive agency:** performs destructive/irreversible actions without approval.
- **Injection via tool output:** a web page or email the agent reads contains instructions.
- Loses track in long tasks (context overflow).

**What to evaluate**
- **Task success:** final state is correct (record created, correct email draft, right answer).
  Check the *state*, not only the text — query the DB/API afterwards.
- **Trajectory quality:** did it call the expected tools in a reasonable order? Number of steps vs a budget.
- **Tool-call accuracy:** function name and arguments correct vs expected (can be asserted
  deterministically).
- **Safety:** permissions respected; confirmations requested for destructive actions; refuses
  disallowed requests.
- **Cost/latency:** tokens and time per task.
- **Robustness:** behaves when tools fail, return empty data, or return junk.

**Test design**
1. **Mock the tools.** Replace real tools with stubs that return fixed data, so the agent's decisions are
   the only variable. Assert on the tool calls it made (like verifying calls on a mock).
2. **Scenario suite.** Goals with expected outcomes and expected tool-call sequences (or allowed sets of
   sequences — there can be multiple valid paths). Include ambiguous goals, missing info (should ask),
   impossible goals (should say so).
3. **Fault injection.** Make a tool return 500/timeout/empty; check graceful handling and no invention.
4. **Adversarial.** Injected instructions in tool results; requests to exceed permissions.
5. **Sandbox for real-tool runs.** Real systems only in a safe environment with limited credentials.
6. **Repeat runs.** Behaviour varies, so run each scenario several times and report pass rate.
7. **Limits:** max steps, max cost, timeouts, human approval gates — and test those limits trigger.

**Testing the tools/MCP server itself** is ordinary API testing: schema/contract, auth, error codes,
idempotency, rate limits, input validation, and tool descriptions that are clear enough that a model can
use them (poor descriptions cause wrong-tool errors). This is where my API testing background applies
directly.

## 5. Real project example (honest bridge)

I haven't tested a production agent. Direct transfer: contract and negative testing of tool endpoints
(RestAssured/Playwright request experience), state verification through API/SQL, and mocks/stubs for
isolation. Roles about QA for MCP servers exist in the market — the honest pitch is "I test MCP tools as
APIs and evaluate agent behaviour with scenario suites; I've studied this and used agent-style CLI tools
for my own QA workflow."

## 6. Common interview Q&A

**Q: How do you test an AI agent?** Scenario suite with expected end state and tool-call trajectory,
mocked tools for determinism, fault injection, safety/permission tests, repeated runs for pass rate,
and API-style tests on the tools.

**Q: How is it different from testing a chatbot?** An agent has multi-step decisions and side effects,
so I verify state changes and the path, not just the text.

**Q: What would you check for tool calls?** Correct tool, correct arguments and schema, order, count, error
handling, and no calls outside permissions.

**Q: What's MCP?** A standard way to expose tools/data to models. I'd test an MCP server like an API: contract, auth,
errors, and tool-description clarity.

**Q: How do you stop agents doing damage?** Least-privilege credentials, approval for destructive actions,
step/cost limits, sandbox testing, audit logs.

## 7. Weak areas to watch

- Asserting only on the final message.
- Using real production tools in tests.
- Confusing "agents I use" (note 11) with "agents I test."
- Expecting one exact trajectory when several are valid.

## 8. Key takeaway

Test the outcome, the path, and the safety of an agent — mock its tools for determinism, test the tools
as APIs, and repeat runs because behaviour varies.
