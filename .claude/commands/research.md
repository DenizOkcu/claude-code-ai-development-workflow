## Phase 2: Codebase & Ecosystem Research

You are entering the **Research** phase for issue: `$ARGUMENTS`

### Pre-Conditions
- Read `.claude/planning/$ARGUMENTS/STATUS.md` to confirm Discovery is complete
- Read `.claude/planning/$ARGUMENTS/DISCOVERY.md` for context

### Instructions

Perform a deep investigation of the codebase and relevant ecosystem. Create `.claude/planning/$ARGUMENTS/CODE_RESEARCH.md` with:

1. **Codebase Analysis**
   - Relevant files and modules (list paths and purposes)
   - Existing patterns that must be followed
   - Code conventions observed (naming, structure, error handling)
   - Current test patterns and coverage

2. **Architecture Context**
   - How the affected area fits into the broader system
   - Data flow through the relevant components
   - API boundaries and contracts
   - Database schema relevant to the change

3. **Dependency Analysis**
   - Internal dependencies (modules, shared utilities)
   - External dependencies (packages, services, APIs)
   - Version constraints or compatibility concerns
   - Dependency health (maintenance status, known vulnerabilities)

4. **Integration Points**
   - Systems that will be affected by this change
   - APIs that consume or produce relevant data
   - Event buses, queues, or async processes involved
   - Frontend/backend boundaries

5. **Risk Assessment**
   - Technical risks (complexity, unknowns)
   - Integration risks (breaking changes, backwards compatibility)
   - Performance risks (scaling, latency)
   - Data risks (migration, consistency)

6. **Prior Art & Ecosystem Research**
   - How similar problems are solved in the ecosystem
   - Relevant libraries, frameworks, or patterns
   - Industry best practices for this type of work
   - Known pitfalls and anti-patterns to avoid

7. **Recommendations**
   - Suggested technical approach (with alternatives)
   - Key decisions that need to be made
   - Unknowns that need resolution before design

### Post-Actions
- Update `STATUS.md`: mark Research as completed
- Add key findings to the Key Decisions section
- Suggest next command: `/design-system $ARGUMENTS`

### Quality Gates
- All 7 sections are substantively filled (not placeholder text)
- File paths are real and verified against the codebase
- Risks are specific, not generic
- Recommendations are actionable
