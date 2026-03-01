---
model: sonnet
---

## Phase 9: Observability Setup

You are entering the **Observe** phase for issue: `$ARGUMENTS`

### Pre-Conditions
- Read `STATUS.md` — confirm Deploy plan is complete
- Read `ARCHITECTURE.md` for system components
- Read `PROJECT_SPEC.md` for non-functional requirements / SLAs

### Instructions

Define the observability strategy for the feature. Create `.claude/planning/$ARGUMENTS/OBSERVABILITY.md`:

#### 1. Key Metrics

Define metrics using the **RED** method (for services) and **USE** method (for resources):

**RED (Request-oriented):**
| Metric | Description | Target | Alert Threshold |
|--------|-------------|--------|-----------------|
| Rate | Requests per second | ... | ... |
| Errors | Error rate percentage | < N% | > N% for M minutes |
| Duration | p50, p95, p99 latency | < Nms | p99 > Nms for M minutes |

**USE (Resource-oriented):**
| Resource | Utilization | Saturation | Errors |
|----------|-------------|------------|--------|
| CPU | ... | ... | ... |
| Memory | ... | ... | ... |
| Database connections | ... | ... | ... |

**Business Metrics:**
| Metric | Description | Baseline | Alert Threshold |
|--------|-------------|----------|-----------------|
| ... | ... | ... | ... |

#### 2. Structured Logging

Define log events for the feature:

```
{feature}:{action}:{result}
```

| Event | Level | Fields | When |
|-------|-------|--------|------|
| `auth:login:success` | INFO | userId, method | Successful login |
| `auth:login:failure` | WARN | method, reason, ip | Failed login |
| `auth:token:refresh` | INFO | userId | Token refreshed |
| `auth:token:expired` | WARN | userId, age | Expired token used |

Logging principles:
- Structured JSON format
- Correlation IDs on all requests
- No PII in logs (or redacted)
- Appropriate log levels (not everything is ERROR)

#### 3. Distributed Tracing (if applicable)

- Span definitions for new operations
- Context propagation across service boundaries
- Trace sampling strategy

#### 4. Alerting Rules

| Alert | Condition | Severity | Action |
|-------|-----------|----------|--------|
| High error rate | > N% errors for 5 min | P1 | Page on-call |
| High latency | p99 > Nms for 10 min | P2 | Notify channel |
| ... | ... | ... | ... |

Alert principles:
- Every alert must have a clear action (no alert fatigue)
- Alerts fire on symptoms, not causes
- Include runbook links in alert descriptions

#### 5. Dashboard Specification

Define panels for a monitoring dashboard:

```markdown
## Dashboard: {feature-name}

### Row 1: Health Overview
- Panel: Request rate (graph, 5m window)
- Panel: Error rate (graph, 5m window)
- Panel: Latency p50/p95/p99 (graph, 5m window)

### Row 2: Business Metrics
- Panel: {metric} (graph/stat)
...

### Row 3: Infrastructure
- Panel: CPU/Memory utilization
- Panel: Database query latency
...
```

#### 6. SLI/SLO Definitions (if applicable)

| SLI | SLO | Measurement |
|-----|-----|-------------|
| Availability | 99.9% | Successful requests / total requests |
| Latency | p99 < 500ms | Request duration histogram |
| ... | ... | ... |

### Post-Actions
- Update `STATUS.md`: mark Observe as completed
- Suggest next command: `/retro $ARGUMENTS`

### Quality Gates
- Every new API endpoint has RED metrics defined
- Alert conditions are specific (not vague thresholds)
- Every alert has an associated action
- Logging events cover success AND failure paths
- No PII appears in log field definitions
- Dashboard has at least: health, business, and infrastructure rows
