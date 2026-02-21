## Phase 1: Discovery & Scoping

You are entering the **Discovery** phase of the SDLC. Your job is to take a raw feature description, detect the project's tech stack, and produce a scoped, well-defined starting point for development.

### Instructions

#### 1. Generate Issue Name

From the description provided in `$ARGUMENTS`:
- Use **kebab-case** (e.g., `add-jwt-auth`, `fix-memory-leak`)
- Keep it concise (2-5 words) and descriptive
- Prefix with action verb: `add-`, `fix-`, `refactor-`, `improve-`, `migrate-`

#### 2. Detect Project Tech Stack

**Scan the project root and key directories to identify the full stack.** This is critical — the detected stack determines which expert commands are available throughout the workflow.

Scan for these markers:

**Languages & Frameworks:**
| Marker File | Detected As | Expert Command |
|-------------|-------------|----------------|
| `tsconfig.json` | TypeScript | `/language/typescript-pro` |
| `package.json` with `react` | JavaScript + React | `/language/javascript-react-pro` |
| `package.json` (no React) | JavaScript/Node.js | `/language/javascript-react-pro` |
| `composer.json` | PHP | `/language/php-pro` |
| `pyproject.toml` or `requirements.txt` | Python | `/language/python-pro` |
| `Cargo.toml` | Rust | (native knowledge) |
| `go.mod` | Go | (native knowledge) |

**Infrastructure & Cloud:**
| Marker File | Detected As | Expert Command |
|-------------|-------------|----------------|
| `*.tf` files | Terraform | `/language/terraform-pro` |
| `serverless.yml` or `sam.yml` | Serverless/AWS | `/language/aws-pro` |
| AWS CDK / `cdk.json` | AWS CDK | `/language/aws-pro` |
| `*.bicep` or ARM templates | Azure | `/language/azure-pro` |
| `app.yaml` (App Engine) or GCP references | GCP | `/language/gcp-pro` |
| `Dockerfile` / `docker-compose.yml` | Containerized | Note in stack |
| `k8s/` or `kubernetes/` or `*deployment*.yaml` with `apiVersion` | Kubernetes | `/language/kubernetes-pro` |
| `openshift/` or `*.yaml` with `route.openshift.io` or `build.openshift.io` | OpenShift | `/language/openshift-pro` |
| `playbooks/` or `roles/` or `ansible.cfg` or `*.yml` with `hosts:` + `tasks:` | Ansible | `/language/ansible-pro` |
| `inventories/` or `group_vars/` or `host_vars/` | Ansible inventory | `/language/ansible-pro` |

**Frameworks (detail):**
| Marker | Framework |
|--------|-----------|
| `next.config.*` | Next.js |
| `nuxt.config.*` | Nuxt |
| `angular.json` | Angular |
| `vue` in package.json | Vue.js |
| `django` in requirements | Django |
| `fastapi` in requirements | FastAPI |
| `flask` in requirements | Flask |
| `laravel` in composer | Laravel |
| `symfony` in composer | Symfony |

**Quality Tooling (existing):**
| Marker | Tool |
|--------|------|
| `eslint.config.*` / `.eslintrc.*` | ESLint configured |
| `.prettierrc` | Prettier configured |
| `vitest.config.*` / `jest.config.*` | Test runner configured |
| `phpstan.neon` | PHPStan configured |
| `ruff.toml` / `[tool.ruff]` in pyproject | Ruff configured |
| `.pre-commit-config.yaml` | Pre-commit hooks configured |
| `.github/workflows/` | CI/CD configured |

#### 3. Create Planning Directory

Create `.claude/planning/{issue-name}/`

#### 4. Create `DISCOVERY.md`

```markdown
# Discovery: {issue-name}

## Summary
One-paragraph description of what this feature/fix does.

## Problem Statement
What problem does this solve? Who is affected?

## Success Criteria
- [ ] Measurable criterion 1
- [ ] Measurable criterion 2
- [ ] Measurable criterion 3

## Scope
### In Scope
- ...

### Out of Scope
- ...

## Stakeholders
- Users: ...
- Teams: ...
- Systems: ...

## Risk Assessment
**Level:** High / Medium / Low
**Justification:** ...

## Dependencies
- ...

## Estimated Complexity
**Size:** S / M / L / XL
**Reasoning:** ...

## Detected Tech Stack

### Languages & Frameworks
| Technology | Version | Expert Command |
|------------|---------|----------------|
| {detected} | {version} | `/language/{name}-pro` |

### Infrastructure
| Technology | Expert Command |
|------------|----------------|
| {detected} | `/language/{name}-pro` |

### Quality Tooling
| Tool | Status |
|------|--------|
| Linter | ✓ Configured / ✗ Missing |
| Formatter | ✓ Configured / ✗ Missing |
| Test Runner | ✓ Configured / ✗ Missing |
| CI/CD | ✓ Configured / ✗ Missing |
| Pre-commit Hooks | ✓ Configured / ✗ Missing |

### Missing Quality Tooling Recommendations
If any quality tooling is missing, recommend:
- Missing linter → "Run `/quality/lint-setup` to configure"
- Missing tests → "Run `/quality/test-strategy` to set up testing"
- Missing dependency auditing → "Run `/quality/dependency-check` to audit"
- General quality check → "Run `/quality/code-audit` for a full assessment"

### Fallback Expert Commands
If the detected language or cloud provider does NOT match any specific expert:
- Unknown/unmatched language → **`/language/software-engineer-pro`** (SOLID, clean architecture, testing, API design, refactoring — universal patterns)
- Unknown/unmatched cloud or hybrid/on-premises infra → **`/language/cloud-engineer-pro`** (provider-agnostic networking, IAM, IaC, observability, DR, cost control)

Always assign at least one expert command. If nothing specific matches, these two generics cover any stack.
```

#### 5. Create `STATUS.md`

```markdown
# Status: {issue-name}

**Risk:** {risk-level} | **Updated:** {timestamp}
**Stack:** {primary language} + {framework} + {cloud/infra}

## Progress
- [x] Discovery - Completed
- [ ] Research - Not started
- [ ] Design - Not started
- [ ] Planning - Not started
- [ ] Implementation - Not started
- [ ] Review - Not started
- [ ] Security - Not started
- [ ] Deploy - Not started
- [ ] Observe - Not started
- [ ] Retro - Not started

## Detected Stack
{concise stack summary, e.g., "TypeScript + React + Next.js 15 / AWS (Terraform) / Vitest"}

## Applicable Expert Commands
- `/language/{detected}-pro` — for {language} patterns
- `/language/{cloud}-pro` — for cloud architecture
- `/quality/code-audit` — for quality assessment

## Key Decisions
(none yet)

## Artifacts
- DISCOVERY.md
```

#### 6. Output to User

Present:
1. **Generated issue name**
2. **Detected tech stack** (table format)
3. **Missing quality tooling** (with recommended commands to fix)
4. **Applicable expert commands** for this stack
5. **Recommended next steps:**
   - If quality tooling is missing: suggest running `/quality/lint-setup` or `/quality/test-strategy` first
   - Always: suggest `/research {issue-name}` as the main next step
   - If the feature involves AI/LLM: note that `/ai-integrate {issue-name}` is available

### Quality Gates
- Issue name follows kebab-case convention
- Tech stack detection scanned at least: package.json/composer.json/pyproject.toml, tsconfig, *.tf files, Dockerfile, CI config
- DISCOVERY.md has all required sections filled including the full stack table
- STATUS.md includes detected stack and applicable expert commands
- Missing quality tooling is explicitly called out with fix commands
- Success criteria are measurable, not vague
