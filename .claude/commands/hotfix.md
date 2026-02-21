## Bonus: Emergency Hotfix Workflow

Compressed workflow for critical production issues. Description: `$ARGUMENTS`

### Instructions

This is a **compressed 4-step emergency workflow** that combines Research → Fix → Review → Deploy into a single rapid execution. Use only for genuine production emergencies.

#### Step 1: Rapid Assessment (2 minutes max)

1. Generate issue name with `hotfix-` prefix (e.g., `hotfix-auth-crash`)
2. Create `.claude/planning/{issue-name}/` directory
3. Create minimal `STATUS.md`
4. Identify:
   - **Symptom**: What users are experiencing
   - **Impact**: How many users, how severe
   - **Root cause hypothesis**: Best guess from the description
   - **Affected files**: Quick scan of likely locations

#### Step 2: Targeted Fix

1. Make the **minimum change** to resolve the issue
2. Write at least ONE test that reproduces the bug and verifies the fix
3. Run the existing test suite to confirm no regressions
4. Do NOT refactor, do NOT optimize, do NOT "improve while you're in there"

#### Step 3: Speed Review

Run automated checks:
```bash
npm run lint
npm run typecheck
npm run test
```

Quick manual review:
- [ ] Fix addresses the root cause (not just the symptom)
- [ ] No new security vulnerabilities introduced
- [ ] Test reproduces the original bug
- [ ] Existing tests still pass
- [ ] Change is minimal and focused

#### Step 4: Deploy Notes

```markdown
## Hotfix: {issue-name}

**Severity:** P0/P1/P2
**Impact:** {description}
**Root Cause:** {1-2 sentences}
**Fix:** {1-2 sentences}
**Files Changed:** {list}
**Tests Added:** {count}
**Rollback:** {1 sentence}
**Follow-up:** {any technical debt created by the hotfix}
```

### Post-Actions
- Update `STATUS.md` to WORKFLOW COMPLETE
- Create a follow-up issue for proper cleanup if technical debt was introduced
- Add learning to `CLAUDE.md`: what caused this and how to prevent it

### Critical Rules
- **Speed over perfection**: The goal is to stop the bleeding
- **Minimum viable fix**: Smallest change that resolves the issue
- **Always test**: Even in an emergency, write at least one regression test
- **Document the debt**: If you cut corners, create a follow-up issue
- **Never skip deploy verification**: Always verify the fix works in production
