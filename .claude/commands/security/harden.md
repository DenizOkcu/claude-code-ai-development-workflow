---
name: harden
description: >
  Phase 8 — Security hardening. Reads 07a_SECURITY_AUDIT.md, 07b_PENTEST_REPORT.md,
  and 07c_AI_THREAT_MODEL.md (if present), then generates a prioritized fix plan
  and triggers implementation of security patches. Closes the
  audit → fix → verify loop.
model: opus
---

# /harden {issue}

Generate a prioritized security fix plan from all audit outputs, then implement patches.

## Step 1 — Aggregate All Security Findings

Read the following files:

1. `.claude/planning/$ARGUMENTS/07a_SECURITY_AUDIT.md` — static findings
2. `.claude/planning/$ARGUMENTS/07b_PENTEST_REPORT.md` — confirmed exploits (if exists)
3. `.claude/planning/$ARGUMENTS/07c_AI_THREAT_MODEL.md` — AI threats (if exists)

Create a unified severity-ranked finding list. Apply this triage logic:

| Priority | Condition |
|---|---|
| **P0 — Fix now** | Critical or High severity AND confirmed by dynamic testing |
| **P1 — Fix this sprint** | Critical or High severity, static only (unconfirmed) |
| **P2 — Fix next sprint** | Medium severity, any source |
| **P3 — Backlog** | Low severity or Informational |

---

## Step 2 — Generate 08_HARDEN_PLAN.md

Create `.claude/planning/$ARGUMENTS/08_HARDEN_PLAN.md`:

```markdown
# Hardening Plan — {issue}

**Generated**: {date}
**Source reports**: 07a_SECURITY_AUDIT.md | 07b_PENTEST_REPORT.md | 07c_AI_THREAT_MODEL.md
**Total findings**: {X} (Critical: X | High: X | Medium: X | Low: X)

## Fix Summary

| Priority | Finding | CWE | Affected File | Effort |
|---|---|---|---|---|
| P0 | SQL Injection in /api/users | CWE-89 | src/routes/users.ts | 1h |
| P0 | Auth bypass via JWT alg:none | CWE-347 | src/auth/verify.ts | 2h |
| P1 | Missing CSRF protection | CWE-352 | src/middleware/ | 3h |
...

## P0 Fixes — Implement Immediately

### Fix 1: {finding title}

**Finding**: {brief description}
**File**: {path}
**Root cause**: {why this exists}

**Before** (vulnerable):
```{language}
{vulnerable code snippet}
```

**After** (hardened):
```{language}
{fixed code snippet}
```

**Regression test**:
```{language}
{test that proves the fix works}
```

{Repeat for each P0 finding}

## P1 Fixes — This Sprint

{Same format as P0}

## P2 Fixes — Next Sprint

{Abbreviated format — description + recommended approach}

## P3 Backlog

{List only — link to GitHub issues if applicable}

## Verification Checklist

After implementing all P0/P1 fixes:
- [ ] All new regression tests pass
- [ ] Re-run `/security {issue}` — confirm static findings resolved
- [ ] Re-run `/pentest {issue}` on staging — confirm exploits no longer work
- [ ] Update CLAUDE.md with lessons learned (e.g., "Always use parameterized
      queries — raw string interpolation caused SQLi in {issue}")
```

---

## Step 3 — Implement P0 Fixes

Apply the P0 patches directly. For each fix:

1. Edit the affected file.
2. Write or update the corresponding test.
3. Verify the test passes: `npm test` / `pytest` / `php artisan test`.
4. Add a comment in the code referencing the finding:
   ```
   // Security: CWE-89 — use parameterized query (hardened in {issue})
   ```

Do not bundle security fixes with feature changes. Keep patches surgical and
reviewable in isolation.

---

## Step 4 — Schedule P1/P2 Fixes

For each P1 and P2 finding, create a GitHub issue:

```bash
gh issue create \
  --title "Security: {finding title} ({CWE})" \
  --body "**Severity**: {severity}
**Source**: {07a_SECURITY_AUDIT.md | 07b_PENTEST_REPORT.md}
**Affected**: {file}
**Fix**: See 08_HARDEN_PLAN.md in .claude/planning/{issue}/

{description}"
  --label "security,{priority}"
```

---

## Step 5 — Update 00_STATUS.md

```markdown
- [x] Hardening - Completed
  - P0 fixes implemented: X
  - P1 issues created: X
  - P2 issues created: X
  - Regression tests added: X
  - Re-pentest required: YES (run /pentest after deploying fixes)
```

---

## Step 6 — Update CLAUDE.md Learnings

Append to the project's `CLAUDE.md` under `## Learnings`:

```markdown
- {date}: {specific, actionable lesson from this security audit} ({issue})
  Examples:
  - "Always validate JWT algorithm claim — alg:none bypass found in {issue}"
  - "Escape model output before rendering — XSS via LLM response in {issue}"
  - "Use parameterized queries for all DB calls — SQLi confirmed in {issue}"
```

---

## Next Step

Once P0 fixes are deployed to staging, run `/pentest $ARGUMENTS` again to
confirm all confirmed exploits are now closed.

Then proceed to `/deploy-plan $ARGUMENTS`.
