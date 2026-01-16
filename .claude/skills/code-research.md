---
name: researching-code
description: "Deep codebase analysis investigating architecture, patterns, dependencies, and integration points. Use before implementation or when analyzing unfamiliar code areas. Triggers: research, investigate codebase, analyze architecture, find patterns, assess risks."
model: sonnet
temperature: 0
---

# Code Research

Investigate codebase architecture, patterns, and integration points to inform planning and implementation.

## LLM Parameters

**For consistent, deterministic outputs:**
- **Temperature:** 0 (deterministic, same inputs → same outputs)
- **Max Tokens:** 6000 (sufficient for deep analysis)
- **Top-P:** 1.0 (use full distribution)
- **Frequency Penalty:** 0 (no penalty for repetition)
- **Presence Penalty:** 0 (no penalty for new topics)

**Seed Strategy:** Use `issue_name` as seed for consistency across reruns

**Why These Parameters:**
- Temperature 0 ensures reproducible research results
- Higher token limit allows comprehensive analysis
- No penalties preserve complete information
- Seed based on issue_name provides deterministic variation per issue

## User Communication

### Phase Start Notification

Before starting research, inform the user:

```markdown
## 📍 Phase: Research

**Starting:** Codebase research and analysis
**Goal:** Understand architecture, patterns, integration points
**Approach:** {quick | standard | deep} research
**Estimated Time:** 20-40 minutes

Investigating codebase...
```

### Progress Updates

During research, provide updates at key points:

```markdown
**Progress Update:** Researching codebase

✓ Identified file structure ({N} directories)
✓ Located relevant patterns ({count} patterns)
→ Analyzing integration points (in progress)
```

**When to show progress:**
- After initial file structure discovery
- After finding similar functionality
- Before synthesizing findings

### Artifact Creation Notification

When RESEARCH_SUMMARY.md is created:

```markdown
**Created:** RESEARCH_SUMMARY.md 📄

📄 Location: `docs/{issue-name}/RESEARCH_SUMMARY.md`
📊 Size: ~150-200 lines
💡 Purpose: Comprehensive summary for planning phase
```

### Phase Completion Notification

When research is complete:

```markdown
## ✅ Phase Complete: Research

**Duration:** {time taken}
**Artifacts Created:**
- CODE_RESEARCH.md (~{lines} lines)
- RESEARCH_SUMMARY.md (~{lines} lines)

**Key Findings:**
- Risk Level: {Low | Medium | High}
- Integration Points: {count} locations
- Dependencies to Add: {count}
- Patterns Identified: {count}

**Next Phase:** Planning

Proceeding automatically...
```

## Few-Shot Example

### Example Input
```yaml
issue_name: "add-oauth-login"
feature_description: "Add OAuth2 login with Google and GitHub providers"
research_depth: "standard"
```

### Example Output: CODE_RESEARCH.md (Good ✅)
```markdown
### Summary
- Risk: Medium
- Key findings:
  - Existing auth system in `src/auth/` uses passport.js
  - Session management in `src/middleware/session.ts`
  - OAuth strategies already implemented for GitHub
  - Need to add Google OAuth2 strategy
- Top recommendations:
  - Reuse existing passport.js setup
  - Follow GitHub OAuth pattern for Google
  - Update session handling for multiple providers

### Integration Points
**Files to modify:**
- `src/auth/passport.ts:50-100` - Add Google OAuth2 strategy
- `src/middleware/session.ts:120-150` - Update session for provider tracking
- `src/routes/auth.ts:45-80` - Add Google OAuth routes

**Reusable patterns:**
- `src/auth/github-strategy.ts:1-50` - Reference for OAuth pattern
- `src/middleware/auth.ts:20-40` - Session management pattern

**Dependencies:**
- `passport-google-oauth20` (new)
- Existing `passport`, `express-session` (already in use)

### Technical Context
- Stack: Node.js, Express, passport.js, express-session
- Pattern: MVC with middleware-based auth
- Communication: Callback-based async, session-based state
- Conventions: TypeScript with strict mode, service layer pattern

### Risks
- Medium: Session size increase with multiple OAuth tokens
- Low: Google OAuth rate limits
- Low: Token storage in session (existing pattern)

### Key Files
```
src/auth/passport.ts:1-200 - Passport configuration
src/auth/github-strategy.ts:1-80 - GitHub OAuth reference
src/middleware/session.ts:1-150 - Session management
src/routes/auth.ts:1-100 - Auth routes
```

### Open Questions
- Should we store OAuth tokens in database or session? (Current: session)
- Do we need token refresh logic? (Check GitHub implementation)
```

### Example Output: CODE_RESEARCH.md (Bad ❌)
```markdown
## Research for OAuth

I looked at the codebase and found auth stuff. Need to add Google login.

Files:
- src/auth - has passport
- src/routes - auth routes

Recommendations:
- Add Google OAuth
- Follow existing pattern
```

**Why Bad:**
- Missing risk assessment
- No specific file:line references
- Missing technical context
- No integration points detailed
- Missing key files list
- No open questions
- Too brief, not actionable

## Inputs
- `issue_name`: Kebab-case identifier
- `feature_description`: What to research
- `research_depth`: "quick" | "standard" | "deep"

## Outputs
- `docs/{issue_name}/CODE_RESEARCH.md` (6 sections)
- `docs/{issue_name}/STATUS.md`

## Procedure

### 1. Setup
Create `docs/{issue_name}/` directory.

### 2. Investigation Strategy

**Quick depth** (simple changes):
- 5-10 files, focus on integration points and conventions

**Standard depth** (medium complexity):
- 15-30 files, full architecture mapping and dependency analysis

**Deep depth** (complex/major):
- 30+ files, include git history, performance, and security analysis

### 3. Discover Architecture

Use Glob/Grep/Read to find:
- **Project structure**: Directories, entry points, build config, package dependencies
- **Tech stack**: Languages, frameworks, key libraries, testing frameworks
- **Patterns**: MVC, layered, microservices, state management, data flow
- **Communication**: Events, callbacks, promises, async patterns

### 4. Analyze Similar Functionality

Search for existing implementations:
- Similar features or components
- Related utilities or helpers
- Comparable patterns to reuse
- Previous approaches to similar problems

Document:
- What works well (reuse these patterns)
- What could be improved (avoid anti-patterns)

### 5. Follow Data Flow

Track how data moves through the system:
- Data entry points (APIs, user input, files)
- Transformations (processing, validation, business logic)
- Storage/retrieval (databases, caches, state)
- Side effects (external calls, events, state changes)

### 6. Identify Integration Points

- Files to modify (with line references)
- Reusable patterns and utilities
- Dependency chains and shared modules
- API contracts and interfaces
- Configuration files requiring updates
- Extension points and hooks

### 7. Assess Conventions

Document existing patterns:
- Coding style and naming
- File organization
- Comment and documentation style
- Error handling patterns
- Testing patterns and coverage
- Module export/import style
- Async patterns (promises, async/await)

### 8. Analyze with Git History (deep research)

```bash
git log --all -- <path>           # File history
git blame <file>                   # Line-by-line changes
git grep <pattern>                 # Historical searches
```

### 9. Assess Risks

Identify:
- Technical debt in affected areas
- Brittle or tightly-coupled code
- Missing tests or low coverage
- Performance bottlenecks
- Security considerations
- Breaking change risks

### 10. Identify Opportunities

Look for:
- Code that should be refactored first
- Utilities that could be extracted/reused
- Patterns that should be followed
- Patterns that should be avoided
- Existing TODOs or FIXMEs in relevant areas
- Documentation gaps

### 11. Create CODE_RESEARCH.md

```markdown
### Summary
- Risk level: Low/Medium/High
- Key findings (3-5 bullets)
- Top recommendations (3-5 bullets)

### Integration Points
- Files to modify (with line refs)
- Reusable patterns/code
- Anti-patterns to avoid

### Technical Context
- Stack: [framework, language, key libs]
- Patterns: [architecture style, state mgmt, communication]
- Conventions: [naming, structure, testing, async patterns]

### Risks
- Technical debt areas
- Breaking change risks
- Performance/security concerns

### Key Files
```
path/to/file.ts:10-50 - Brief description
```

### Open Questions
- Critical decisions needed
- Clarifications required
```

### 12. Update STATUS.md

```markdown
# Status: {issue_name}

**Risk:** [Low/Medium/High] | **Updated:** {timestamp}

## Progress
- [x] Research | [ ] Planning | [ ] Implementation | [ ] Review

## Phase: Research ✓
- **Key Findings:** [top 3 bullets]
- **Recommendations:** [top 3 bullets]
- **Status:** Research complete, ready for planning
```

### 13. Create RESEARCH_SUMMARY.md

**CRITICAL: You MUST create this summary artifact for the planning phase.**

Create a comprehensive summary document that distills the research findings for the planning phase:

```markdown
# Research Summary

**Generated:** {timestamp}
**Issue:** {issue_name}

## For Planning Phase

### What We're Building
[Clear goal description from feature_description]

### Key Findings
[Extract top 5-7 key findings from CODE_RESEARCH.md]

### Integration Points
[Complete list with file:line references]
- Files to modify: `src/file.ts:50-100` - add new functionality
- Files to create: `src/new-file.ts` - new component
- Dependencies: package-name, version
- Configuration changes: config/file.ts

### Technical Context
[Complete technical landscape]
- Stack: frameworks, languages, key libraries
- Patterns: architecture style, state management
- Conventions: naming, structure, async patterns

### Risks and Mitigations
[Complete risk assessment]
1. **Risk Name** - Severity: High/Medium/Low
   - Impact description
   - Mitigation strategy

### Recommendations
[Complete recommendations with rationale]
1. Recommendation - Reason
2. Recommendation - Reason

### Questions to Answer
[All open questions that planning must address]

---
*This is a comprehensive summary for the planning phase. See CODE_RESEARCH.md for full details.*
```

**Quality Check for Summary:**
- Contains all required sections
- File references include line numbers
- Risks have severity levels
- Recommendations have rationale
- At least 5 key findings
- All questions are documented

### 14. Present Findings

Provide brief summary (3-5 sentences max):
- Risk level and why
- Top 3 findings
- Top 3 recommendations
- Blocking questions (if any)

## Error Handling

### Known Error Scenarios

1. **Directory Creation Failed**
   - Check if parent directory exists
   - Verify write permissions
   - Create parent directories if needed
   - Error: "Cannot create directory `docs/{issue_name}/`"

2. **No Files Found for Pattern**
   - Verify glob pattern is correct
   - Check if files exist in repository
   - Suggest alternative patterns
   - Error: "No files found matching pattern `{pattern}`"

3. **Git Commands Failed**
   - Fallback: Skip git history analysis
   - Document that git analysis was skipped
   - Warning: "Git commands unavailable - historical analysis skipped"

4. **File Read Errors**
   - Check file permissions
   - Skip files that cannot be read
   - Document skipped files
   - Warning: "Cannot read file `{path}` - skipping"

### Error Recovery

- **Before research starts**: Validate that repository exists and is accessible
- **During investigation**: If a file cannot be read, continue with other files
- **Missing context**: If critical information is unavailable, document as open question
- **Git unavailable**: Continue without git history, add warning to research doc

### Exit Conditions

**Success:**
- CODE_RESEARCH.md created with all 6 sections
- RESEARCH_SUMMARY.md created with all sections
- STATUS.md updated
- All findings documented

**Failure (Retryable):**
- Directory creation fails due to temporary permissions issue
- File read timeouts (retry with different files)

**Failure (Non-retryable):**
- Issue name is invalid
- Repository does not exist
- No accessible files in repository

### Validation Before Completion

Before marking research complete, verify:
- [ ] Directory `docs/{issue_name}/` exists
- [ ] CODE_RESEARCH.md has all 6 sections (Summary, Integration Points, Technical Context, Risks, Key Files, Open Questions)
- [ ] RESEARCH_SUMMARY.md has all sections (What We're Building, Key Findings, Integration Points, Technical Context, Risks and Mitigations, Recommendations, Questions to Answer)
- [ ] File references include line numbers where applicable
- [ ] Risk level is justified
- [ ] At least 3 key findings documented
- [ ] STATUS.md updated

## Important Guidelines

- Don't skip research - rushing leads to poor integration
- Document everything - research guides planning and implementation
- Think holistically - consider full impact
- Look for non-obvious connections - side effects, implicit dependencies
- Consider maintainers - who will maintain this code?

## Quality Checks
- Directory created at correct path
- CODE_RESEARCH.md has all 6 sections
- RESEARCH_SUMMARY.md created with all sections
- Bullet points used (not paragraphs)
- File references include line numbers
- Similar functionality analyzed
- Data flow traced
- Git history analyzed (if deep)
- Opportunities identified
- STATUS.md created
- Error scenarios handled
