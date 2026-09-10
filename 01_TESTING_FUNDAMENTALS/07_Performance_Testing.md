# Performance Testing

## 1. What It Is

Performance testing checks how the system behaves under expected and unexpected load:
speed, stability, scalability, throughput, error rate, and resource usage.

## 2. Why It Matters In Interviews

Even if you are not a specialist performance engineer, QA/SDET roles expect you to know
basic performance risks and how to recognize them. Speak honestly: you can cover QA/SDET
performance thinking without claiming deep tooling not in your resume.

## 3. Complete Topic Breakdown

| Type | Question |
|------|----------|
| Performance testing | How does the system perform overall? |
| Load testing | Can it handle expected traffic? |
| Stress testing | Where does it break beyond expected load? |
| Spike testing | What happens during sudden traffic jumps? |
| Soak/endurance testing | Does it degrade over long duration? |
| Volume testing | What happens with large data volume? |
| Scalability testing | Does adding resources improve capacity? |

### Key metrics

| Metric | Meaning |
|--------|---------|
| Response time | Time taken for request/action |
| Latency | Delay before response begins |
| Throughput | Work completed per second/minute |
| Concurrency | Users/requests active together |
| Error rate | Percentage of failed requests |
| CPU/memory/I/O | Resource usage and bottleneck signals |
| p90/p95/p99 | Tail latency, better than average alone |

## 4. Manual Testing Angle

Manual QA can catch performance risk early:

- Notice slow pages, heavy filters, long exports, delayed save actions.
- Test with realistic data size, not only five records.
- Check loading, empty, error, and timeout behavior.
- Capture browser/network evidence where available.
- Ask expected response time and data volume during requirement review.
- Report impact: who is affected, how often, and in which workflow.

Manual performance observation is not a replacement for load testing, but it prevents
obvious performance issues from being ignored.

## 5. Automation Angle

SDET contribution:

- API-level checks can catch obvious response-time regressions.
- UI automation can catch gross performance failures in critical flows, but it is not
  load testing.
- Heavy load/stress testing usually runs separately from normal UI regression.
- Performance assertions in shared environments should be used carefully because noise
  can cause false failures.
- Automation should compare against baseline where possible.

## 6. Exact Surface To Remember

- Load = expected traffic.
- Stress = beyond expected until breaking point.
- Spike = sudden traffic jump.
- Soak = long-duration stability.
- p95/p99 matter because average hides slow users.
- UI automation is not load testing.

## 7. Real Project Example

> "From a QA/SDET side, I watch for performance risks like slow screens, heavy reports,
> large data behavior, and delayed API responses. If a feature is functionally correct
> but too slow for users, it is still a quality issue. For deeper load/stress testing, I
> would align with performance/platform owners and use realistic data, environment, and
> baseline."

## 8. Common Mistakes / Weak Areas

- Saying performance only means response time.
- Confusing load and stress.
- Trusting average response time only.
- Claiming UI automation is load testing.
- Testing with tiny unrealistic data.
- Ignoring environment/baseline differences.

## 9. 30-Second Answer

Performance testing checks speed, stability, scalability, throughput, and error behavior.
Load testing checks expected traffic, stress testing finds the breaking point, spike
testing checks sudden jumps, and soak testing checks long-duration stability. From a
QA/SDET angle, I watch for performance risk and know when deeper testing is needed.

## 10. 2-Minute Answer

Performance is a quality attribute. A feature can pass functional testing and still fail
users if it takes too long. I look at response time, latency, throughput, concurrency,
error rate, and resource usage. I prefer percentiles like p95 or p99 because averages
hide slow-user experience. In manual testing, I watch slow screens, large data behavior,
exports, filters, and timeouts. In automation, lightweight API or critical-flow checks
can catch obvious regressions, but full load/stress testing needs realistic environment,
data, and tooling.

## 11. Interview Q&A

### Q: Load vs stress?

Load checks expected traffic. Stress pushes beyond expected traffic to find the breaking
point.

### Q: Why percentiles?

Because average can hide slow users. p95/p99 show tail latency.

### Q: Can UI automation do performance testing?

It can catch basic slow critical flows, but it is not load testing.

## 12. Quick Revision Points

- Functional pass can still be performance fail.
- Load expected, stress breaking point, spike sudden jump, soak long duration.
- p95/p99 > average alone.
