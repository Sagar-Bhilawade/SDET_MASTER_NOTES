# Debugging

## 1. What It Is

Debugging is the process of classifying and proving why a Playwright test failed:
application bug, locator issue, wait/timing issue, data issue, environment issue,
assertion issue, or CI-only problem.

## 2. Why It Matters In Interviews

At 3.8+ years, "I rerun it" is not enough. Interviewers expect a systematic debugging
workflow using trace, screenshot, video, console/network evidence, and local reproduction.

## 3. Complete Topic Breakdown

| Tool | Use |
|---|---|
| Trace viewer | Replay actions, DOM snapshots, network, console, screenshots |
| HTML report | See failed tests, retries, attachments |
| Video | Watch visible browser behavior |
| Screenshot | Capture final/failed state |
| UI mode | Explore and debug locally |
| Inspector/debug mode | Step through test |
| Console/network logs | Separate frontend/API/environment problems |
| Error message | Classify failure family |

Debug order:

1. Read error message.
2. Open trace/report.
3. Check locator/action/assertion line.
4. Check app state, data, network, console.
5. Reproduce locally with same command/config.
6. Fix the cause, not only the symptom.

## 4. Manual Testing Angle

Manual evidence habits help automation debugging. A good defect/debug note includes
build, environment, data, role, steps, expected/actual, screenshot/trace, and impact.

## 5. Automation Angle

Configure artifacts so failures are debuggable:

```typescript
export default defineConfig({
  use: {
    trace: 'on-first-retry',
    video: 'retain-on-failure',
    screenshot: 'only-on-failure'
  }
});
```

Trade-off: traces/videos on every test create storage noise. Use targeted artifact policy.

## 6. Exact Surface To Remember

```bash
npx playwright test --debug
npx playwright test tests/login.spec.ts --headed
npx playwright test --ui
npx playwright show-report
npx playwright show-trace trace.zip
```

```typescript
await page.pause();
page.on('console', msg => console.log(msg.text()));
page.on('requestfailed', request => console.log(request.url(), request.failure()));
```

## 7. Real Project Example

> "For BASES_PLAYWRIGHT CI failures, I first classify the error, then open the trace and
> HTML report. If it is a locator/actionability problem, trace usually shows the overlay,
> wrong element, or missing state. If it is API/data, I check network and request context
> before calling it a product bug."

## 8. Common Mistakes / Weak Areas

- Rerunning until green without root cause.
- Treating every timeout as "need more wait".
- Not uploading traces on CI failure.
- Ignoring data/environment differences.
- Debugging from screenshot only when trace is available.
- Fixing with `force: true` instead of understanding actionability.

## 9. 30-Second Answer

I debug Playwright failures by classifying the error first, then using trace viewer,
HTML report, screenshots/video, console and network logs. I separate product defects from
locator, wait, data, environment, and assertion issues before fixing or logging a defect.

## 10. 2-Minute Answer

My debugging flow starts with the error message because it usually tells the failure
family: strict mode, timeout, actionability, assertion mismatch, closed page/context, or
auth/API failure. Then I open trace viewer to replay the run and inspect DOM snapshots,
network, console, and screenshots. For CI-only failures, I reproduce locally with the
same file, browser project, headed/debug mode if needed.

The key is not to patch symptoms. If a click times out, I check whether the element was
hidden, disabled, covered, moving, or incorrectly located. If an API check returns 401, I
check `request` vs `page.request` and token state.

## 11. Interview Q&A

### Q: Test passes locally but fails in CI. What do you do?

Compare browser/project, environment, data, viewport, timing, secrets, baseURL, and
artifacts. Open trace first, then reproduce with the CI command.

### Q: What is trace viewer?

A Playwright artifact that replays the test with actions, DOM snapshots, screenshots,
console, and network information.

### Q: Why trace on first retry?

It captures useful failure detail without creating trace noise for every passing test.

### Q: What if the test is flaky?

Classify root cause: data, environment, selector, wait, app race, network, or assertion.
Retries reveal flakiness; they do not fix it.

## 12. Quick Revision Points

- Classify before fixing.
- Trace is the main debugging artifact.
- CI artifacts must upload on failure.
- Timeout is a symptom, not always a wait issue.
- Fix root cause, not just rerun.
