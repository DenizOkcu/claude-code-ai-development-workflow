## Bonus: Performance Testing

You are entering the **Performance Testing** phase for issue: `$ARGUMENTS`

### Pre-Conditions
- Read `STATUS.md` — confirm Implementation is complete
- Read `ARCHITECTURE.md` for system design and performance notes
- Read `PROJECT_SPEC.md` for non-functional requirements

### Instructions

Design and execute performance tests. Add `PERF_TEST.md` to `.claude/planning/$ARGUMENTS/`:

#### 1. Baseline Capture

Before testing the new feature, capture current baselines:
- API response times (p50, p95, p99)
- Database query durations
- Memory usage under normal load
- CPU utilization under normal load
- Bundle size (if frontend)

#### 2. Benchmark Tests

Create targeted benchmarks for the new code:

```markdown
### Benchmark: {name}
- **What**: {what is being measured}
- **Setup**: {test data, configuration}
- **Metric**: {response time, throughput, memory}
- **Target**: {specific threshold}
- **Result**: {actual measurement}
- **Status**: ✓ PASS / ✗ FAIL
```

#### 3. Load Test Scenarios

| Scenario | Concurrent Users | Duration | Ramp-Up | Target RPS |
|----------|-----------------|----------|---------|------------|
| Normal load | N | 5 min | 30s | N |
| Peak load | Nx3 | 10 min | 60s | Nx3 |
| Stress test | Nx10 | 5 min | 120s | Find limit |
| Soak test | N | 60 min | 30s | N |

For each scenario, report:
- Throughput achieved
- Error rate
- Latency distribution (p50/p95/p99)
- Resource utilization

#### 4. Profiling

- CPU profiling: identify hot paths
- Memory profiling: check for leaks
- Database query analysis: N+1 queries, slow queries, missing indexes
- Network: payload sizes, request waterfalls

#### 5. Frontend Performance (if applicable)

- Lighthouse score (Performance, Accessibility, Best Practices)
- Core Web Vitals (LCP, INP, CLS)
- Bundle size impact
- Runtime performance (long tasks, memory)

#### 6. Summary

```markdown
## Performance Test Summary

### Results vs Targets
| Metric | Target | Actual | Status |
|--------|--------|--------|--------|
| p99 latency | < 500ms | 320ms | ✓ |
| Throughput | > 1000 RPS | 1250 RPS | ✓ |
| Memory delta | < 50MB | 35MB | ✓ |
| Bundle size delta | < 10KB | 8KB | ✓ |

### Optimizations Applied
1. ...

### Known Limitations
1. ...
```

### Post-Actions
- Add `PERF_TEST.md` to artifacts in `STATUS.md`
- Flag any performance regressions as blockers

### Quality Gates
- Baselines captured before testing new code
- All NFR targets from PROJECT_SPEC.md are tested
- Load test covers at least normal + peak scenarios
- Any regressions are documented with remediation plan
