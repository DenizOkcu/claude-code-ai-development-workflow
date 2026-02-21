## Phase 8: Deployment Strategy

You are entering the **Deploy** phase for issue: `$ARGUMENTS`

### Pre-Conditions
- Read `STATUS.md` — confirm Security audit is complete
- Read `ARCHITECTURE.md` for infrastructure context
- Read `PROJECT_SPEC.md` for non-functional requirements

### Instructions

Create a deployment strategy. Produce `.claude/planning/$ARGUMENTS/DEPLOY_PLAN.md`:

#### 1. Pre-Deployment Checklist

- [ ] All tests pass on CI
- [ ] Code review approved
- [ ] Security audit passed
- [ ] Database migrations tested (if any)
- [ ] Environment variables documented and configured
- [ ] Feature flags configured (if applicable)
- [ ] Monitoring/alerting updated (from /observe phase or planned)
- [ ] Runbook/playbook updated
- [ ] Stakeholders notified

#### 2. Rollout Strategy

Choose and detail the appropriate strategy:

| Strategy | When to Use |
|----------|-------------|
| **Big Bang** | Low-risk, simple changes |
| **Feature Flag** | High-risk, needs gradual rollout |
| **Canary** | Performance-sensitive, needs production validation |
| **Blue-Green** | Zero-downtime required |
| **Rolling** | Stateless services, gradual replacement |

Detail:
- Rollout percentage stages (e.g., 1% → 10% → 50% → 100%)
- Duration at each stage
- Success criteria to proceed to next stage
- Monitoring signals to watch at each stage

#### 3. Database Migration Plan (if applicable)

- Migration scripts and their order
- Backwards compatibility during rollout
- Data backfill strategy (if needed)
- Estimated migration duration
- Rollback migration scripts

#### 4. Rollback Playbook

```markdown
## Rollback Procedure

### Triggers (when to rollback)
- Error rate exceeds {threshold}
- Latency exceeds {threshold}
- {specific business metric} degrades by {amount}

### Steps
1. {step 1}
2. {step 2}
3. ...

### Verification
- How to confirm rollback was successful
- Data consistency checks after rollback

### Communication
- Who to notify
- Status page update template
```

#### 5. Post-Deployment Verification

- Smoke tests to run immediately after deploy
- Health check endpoints to monitor
- Key metrics to watch for 24 hours
- User-facing functionality to manually verify

#### 6. Communication Plan

- Changelog entry draft
- Internal team notification
- User-facing release notes (if applicable)

### Post-Actions
- Update `STATUS.md`: mark Deploy as completed (planned)
- Suggest next command: `/observe $ARGUMENTS`

### Quality Gates
- Rollout strategy is specific (not just "deploy to production")
- Rollback playbook has concrete steps (not "revert the deploy")
- Success criteria are measurable
- Pre-deployment checklist references actual commands/tools
