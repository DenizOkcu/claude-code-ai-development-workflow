---
model: sonnet
---

## Phase 7: Security Audit

You are entering the **Security** phase for issue: `$ARGUMENTS`

### Pre-Conditions
- Read `STATUS.md` — confirm Review is complete and APPROVED
- Read `ARCHITECTURE.md` for system design
- Read `CODE_REVIEW.md` for any security-related findings

### Instructions

Perform a security-focused audit of the implemented feature. Create `.claude/planning/$ARGUMENTS/SECURITY_AUDIT.md`:

#### 1. Threat Modeling (STRIDE)

For each component/boundary in the feature:

| Threat | Description | Mitigation | Status |
|--------|-------------|------------|--------|
| **S**poofing | Can someone impersonate a user/service? | ... | ✓/✗ |
| **T**ampering | Can data be modified in transit/at rest? | ... | ✓/✗ |
| **R**epudiation | Can actions be denied without evidence? | ... | ✓/✗ |
| **I**nformation Disclosure | Can sensitive data leak? | ... | ✓/✗ |
| **D**enial of Service | Can the feature be overwhelmed? | ... | ✓/✗ |
| **E**levation of Privilege | Can users gain unauthorized access? | ... | ✓/✗ |

#### 2. OWASP Top 10 Checklist

- [ ] **Injection**: SQL, NoSQL, OS command, LDAP injection vectors
- [ ] **Broken Authentication**: Session management, credential handling
- [ ] **Sensitive Data Exposure**: Encryption at rest/transit, PII handling
- [ ] **XML/XXE**: External entity processing (if applicable)
- [ ] **Broken Access Control**: Authorization checks on every endpoint
- [ ] **Security Misconfiguration**: Default configs, error messages, headers
- [ ] **XSS**: Output encoding, CSP headers, DOM manipulation
- [ ] **Insecure Deserialization**: Object deserialization from untrusted sources
- [ ] **Known Vulnerabilities**: Dependency scanning results
- [ ] **Insufficient Logging**: Security events are logged and auditable

#### 3. Dependency Audit

```bash
# Run available dependency scanners
npm audit                    # Node.js
pip audit                    # Python (if applicable)
```

Report: vulnerability count by severity, remediation steps.

#### 4. Code-Level Security Review

- [ ] Input validation on all external data
- [ ] Output encoding for all rendered content
- [ ] Authentication required on all protected endpoints
- [ ] Authorization checked at the resource level (not just route)
- [ ] Rate limiting on public/sensitive endpoints
- [ ] CORS configured correctly
- [ ] Secrets not hardcoded (env vars or secret manager)
- [ ] SQL queries use parameterized statements
- [ ] File uploads validated (type, size, content)
- [ ] Error messages don't leak internal details

#### 5. Data Privacy

- [ ] PII is identified and handled appropriately
- [ ] Data retention policies are respected
- [ ] User consent flows are correct (if applicable)
- [ ] Data can be exported/deleted on request (GDPR/CCPA)

#### 6. Summary

```markdown
## Security Audit Summary

| Severity | Count |
|----------|-------|
| 🔴 Critical | N |
| 🟡 High | N |
| 🔵 Medium | N |
| ⚪ Low | N |

## Verdict: ✓ PASSED / ⚠ CONDITIONAL PASS / ✗ FAILED

### Required Remediations
1. ...

### Accepted Risks
1. ...
```

### Post-Actions
- Update `STATUS.md` with security audit outcome
- If PASSED: suggest `/deploy-plan $ARGUMENTS`
- If FAILED: list remediations, fix, then re-run `/security $ARGUMENTS`

### Quality Gates
- STRIDE analysis covers all new components/boundaries
- OWASP checklist is fully evaluated (not skipped)
- Dependency audit was actually run (not assumed)
- Every finding has a specific remediation or accepted-risk justification
