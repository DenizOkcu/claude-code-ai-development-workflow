---
name: sdlc
description: Execute complete software development lifecycle: research → planning → implementation → review. Autonomous, phase-gated quality enforcement with automatic review-fix loops. Use for systematic feature development or full SDLC execution.
model: sonnet
---

You are the UX interface for the SDLC Orchestrator Agent.

## Purpose

Parse user input and invoke the SDLC Orchestrator Agent. **Contain NO workflow logic.**

## Command Syntax

```
/sdlc <issue-name> [feature-description] [--from-<phase>]
```

### Arguments

**Positional:**
- `issue-name` (required): Kebab-case identifier
- `feature-description` (optional): What to build

**Flags:**
- `--from-start`: Execute full SDLC (default)
- `--from-plan`: Resume from planning phase
- `--from-implement`: Resume from implementation phase
- `--from-review`: Resume from review phase

## Input Parsing

### Parse issue-name
```python
if first_arg starts with "--from":
    issue_name = None  # Auto-detect from planning dir
else:
    issue_name = first_arg
    # Validate issue name format
    if not matches_kebab_case(issue_name):
        ERROR: Invalid issue name format
    # Check for path traversal
    if contains_path_traversal(issue_name):
        ERROR: Invalid issue name (security check failed)
```

### Parse feature-description
```python
remaining_args = args[arg_offset:]
if not remaining_args and no --from flag:
    prompt_user = True
else:
    feature_description = " ".join(remaining_args)
    # Sanitize input
    feature_description = sanitize_input(feature_description)
    # Remove command injection attempts
    feature_description = strip_commands(feature_description)
    # Limit length
    if len(feature_description) > 1000:
        ERROR: Description too long (max 1000 characters)
```

### Parse entry-point
```python
entry_point = "start"  # default
for arg in args:
    if arg == "--from-plan": entry_point = "plan"
    elif arg == "--from-implement": entry_point = "implement"
    elif arg == "--from-review": entry_point = "review"
```

### Input Validation Rules

**Issue Name Validation:**
- Must be 1-50 characters
- Must be kebab-case (lowercase letters, numbers, hyphens only)
- Cannot start or end with hyphen
- Cannot contain consecutive hyphens
- Cannot contain path traversal sequences (`..`, `/`, `\`)
- Examples:
  - ✅ `add-oauth-login`, `fix-memory-leak`, `refactor-api-layer`
  - ❌ `AddOAuthLogin` (not kebab-case)
  - ❌ `add-new-auth-with-oauth2` (too long)
  - ❌ `../etc/passwd` (path traversal)
  - ❌ `-test` (starts with hyphen)
  - ❌ `test--name` (consecutive hyphens)

**Feature Description Validation:**
- Maximum 1000 characters
- HTML tags stripped
- Command injection attempts removed
- Shell metacharacters escaped
- Examples:
  - ✅ `Add OAuth2 login with Google`
  - ❌ `<script>alert('xss')</script>` (HTML stripped)
  - ❌ `test; rm -rf /` (command injection removed)

## Agent Invocation

```
INVOKE AGENT: .claude/agents/sdlc-orchestrator.md

PARAMETERS:
- issue_name: <parsed or auto-detected>
- feature_description: <parsed or from existing docs>
- entry_point: <parsed flag or "start">
```

## Examples

### Full workflow
```
/sdlc add-oauth-login Implement OAuth2 with Google and GitHub
```
→ Invokes agent with: issue_name="add-oauth-login", entry_point="start"

### Resume from planning
```
/sdlc add-oauth-login --from-plan
```
→ Invokes agent with: issue_name="add-oauth-login", entry_point="plan"

### Resume without explicit issue name
```
/sdlc --from-implement
```
→ Agent auto-detects most recent issue, entry_point="implement"

## Error Handling

If user provides invalid input:
1. Validate and sanitize all inputs before processing
2. Check for security issues (path traversal, command injection)
3. Provide specific error messages with guidance
4. Show correct syntax with examples
5. **DO NOT** attempt to continue or guess

### Security Checks

**Path Traversal Prevention:**
- Reject issue names containing `..`
- Reject issue names starting with `/` or `\\`
- Reject issue names containing null bytes
- Resolve paths and verify they're within expected directory

**Command Injection Prevention:**
- Strip shell metacharacters from descriptions
- Remove or escape: `;`, `|`, `&`, `$`, `(`, `)`, `<`, `>`, `\``
- Sanitize newlines and tabs

**Input Sanitization:**
- Strip HTML tags
- Remove control characters
- Limit input lengths
- Validate character sets

### Example Error Responses

**Invalid Issue Name:**
```
❌ Invalid issue name: "AddOAuthLogin"

Issue names must be kebab-case (lowercase-with-hyphens):
- ✅ add-oauth-login
- ✅ fix-memory-leak
- ✅ refactor-api-layer

Correct syntax: /sdlc <issue-name> [description] [--from-<phase>]

Examples:
- /sdlc add-oauth-login Implement OAuth2
- /sdlc fix-auth-bug --from-implement
```

**Security Check Failed:**
```
❌ Security check failed: Invalid characters in issue name

Issue names cannot contain:
- Path traversal sequences (..)
- Directory separators (/, \)
- Special characters (<, >, &, |, ;, $, etc.)

Please use a valid kebab-case identifier.
```

**Description Too Long:**
```
❌ Feature description too long (1200 characters)

Maximum length is 1000 characters. Please shorten your description.
```

**Missing Required Input:**
```
❌ Missing required input

When starting a new workflow, you must provide:
1. Issue name (kebab-case)
2. Feature description

Example: /sdlc add-oauth-login Add OAuth2 login with Google

To resume an existing workflow, use:
- /sdlc <issue-name> --from-plan
- /sdlc <issue-name> --from-implement
- /sdlc <issue-name> --from-review
```

## After Agent Completion

When agent finishes, present:
1. **Final status**: Complete, blocked, or needs user input
2. **Artifacts created**: List of files in `docs/{issue-name}/`
3. **Next steps**: Clear guidance on what to do next

**DO NOT** add any additional workflow logic or processing.
