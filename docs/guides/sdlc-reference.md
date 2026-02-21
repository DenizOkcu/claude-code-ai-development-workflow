# SDLC Extended — Command Reference

Quick reference for all slash commands in the Extended SDLC workflow.

## Core Workflow (10 Phases)

### `/discover [feature description]`
**Phase 1** — Scope and define a new feature or fix.
- Generates kebab-case issue name
- Creates DISCOVERY.md with problem statement, success criteria, scope
- Initializes STATUS.md with all 10 phases
- **Next:** `/research {issue}`

### `/research {issue-name}`
**Phase 2** — Deep codebase and ecosystem analysis.
- Analyzes relevant files, patterns, dependencies
- Identifies integration points and risks
- Produces technical recommendations
- **Next:** `/design-system {issue}`

### `/design-system {issue-name}`
**Phase 3** — Architecture and system design.
- Creates ARCHITECTURE.md with component design, data model, API design
- Generates ADR (Architecture Decision Record) for each key decision
- Produces PROJECT_SPEC.md with technical and non-functional requirements
- **Next:** `/plan {issue}`

### `/plan {issue-name}`
**Phase 4** — Detailed implementation planning.
- Breaks work into small, testable phases
- Each phase has tasks, tests, acceptance criteria, and rollback
- Includes overall test strategy
- **Next:** `/implement {issue}`

### `/implement {issue-name}`
**Phase 5** — Phase-by-phase code implementation.
- Follows IMPLEMENTATION_PLAN.md exactly
- Test-driven: write test → implement → verify
- Updates STATUS.md with progress per phase
- **Next:** `/review {issue}`

### `/review {issue-name}`
**Phase 6** — Comprehensive code review.
- Runs all automated checks (lint, types, tests, build)
- Manual review against checklist (security, patterns, naming, tests)
- Produces CODE_REVIEW.md with categorized findings
- **Next:** `/security {issue}` (if approved)

### `/security {issue-name}`
**Phase 7** — Security audit.
- STRIDE threat modeling
- OWASP Top 10 checklist
- Dependency vulnerability scanning
- Data privacy review
- **Next:** `/deploy-plan {issue}` (if passed)

### `/deploy-plan {issue-name}`
**Phase 8** — Deployment strategy.
- Pre-deployment checklist
- Rollout strategy (canary, blue-green, feature flag, etc.)
- Rollback playbook with specific triggers and steps
- Post-deployment verification plan
- **Next:** `/observe {issue}`

### `/observe {issue-name}`
**Phase 9** — Observability setup.
- RED/USE metrics definition
- Structured logging events
- Alerting rules with actions
- Dashboard specification
- SLI/SLO definitions
- **Next:** `/retro {issue}`

### `/retro {issue-name}`
**Phase 10** — Retrospective and knowledge capture.
- Documents what went well, what to improve
- Captures technical and process learnings
- Updates CLAUDE.md with actionable insights
- Marks workflow as COMPLETE

---

## Bonus Commands

### `/ai-integrate {issue-name}`
Add LLM/AI capabilities to a feature. Covers model selection, prompt engineering, RAG architecture, guardrails, evaluation strategy, and cost optimization. Use between Design and Plan phases.

### `/perf-test {issue-name}`
Performance testing and benchmarking. Baseline capture, load testing, profiling, frontend performance (Lighthouse/Core Web Vitals). Use after Implementation.

### `/hotfix [description]`
Compressed 4-step emergency workflow: Assess → Fix → Review → Deploy Notes. For production incidents only. Creates `hotfix-` prefixed issues.

### `/typescript-pro [description]`
TypeScript expert mode. Applies advanced type system patterns, branded types, discriminated unions, Result types, exhaustive checks. For complex TypeScript challenges.

### `/devops/ci-pipeline [context]`
Generate production-ready CI/CD pipeline configuration. Auto-detects stack. Outputs GitHub Actions or GitLab CI with lint → test → build → security → deploy stages.

---

## Language & Cloud Expert Commands

All language commands accept a task description as `$ARGUMENTS` and apply stack-specific best practices:

### `/language/typescript-pro [description]`
Strict types, generics, branded types, discriminated unions, Result pattern, exhaustive checks, `satisfies`, strict config.

### `/language/javascript-react-pro [description]`
ES2024+, React 19 patterns (Server Components, `useActionState`, `use()`), accessibility, performance (lazy loading, virtualization), TanStack Query, Testing Library.

### `/language/php-pro [description]`
PHP 8.2+, `declare(strict_types=1)`, enums, readonly classes, match expressions, constructor promotion, Result pattern, Laravel/Symfony patterns, PHPStan level 8.

### `/language/python-pro [description]`
Python 3.11+, full typing (Protocols, `NewType`, pattern matching), dataclasses with `slots=True`, async TaskGroups, FastAPI/Django patterns, Ruff + mypy strict.

### `/language/terraform-pro [description]`
Module design, `for_each` over `count`, variable validation, state isolation, security checklist, cost optimization, `tflint`/`trivy` in CI.

### `/language/aws-pro [description]`
Well-Architected Framework (all 6 pillars), service selection guide, IAM least-privilege, VPC blueprint, cost control, full security + observability stacks.

### `/language/azure-pro [description]`
Well-Architected Framework, Managed Identity (no stored credentials), Bicep IaC, Entra ID, Private Endpoints, Microsoft Defender, cost optimization with Reservations.

### `/language/gcp-pro [description]`
Architecture Framework, Workload Identity (no JSON keys), Cloud Run scale-to-zero, VPC Service Controls, SRE practices (SLOs, error budgets), cost control with CUDs.

### `/language/ansible-pro [description]`
Roles for reusable automation, Ansible Vault for secrets, idempotent tasks, FQCN module names, Molecule testing, dynamic inventory, linting with production profile.

### `/language/kubernetes-pro [description]`
Production Deployments with full security context (runAsNonRoot, readOnlyRootFilesystem, drop ALL capabilities), three probes, resource limits, NetworkPolicies (default-deny), RBAC per workload, PDBs, GitOps patterns, Kustomize overlays.

### `/language/openshift-pro [description]`
Extends Kubernetes patterns with OpenShift Routes (TLS, weighted backends), SCCs (restricted-v2 default), BuildConfigs (S2I, Dockerfile), ImageStreams, Operators (Manual approval in prod), User Workload Monitoring, ResourceQuotas, UBI base images.

### `/language/software-engineer-pro [description]`
**Fallback for any unmatched language/framework.** Universal principles: SOLID, clean architecture layers (domain → application → infrastructure), error handling philosophy, naming conventions, design patterns (when to use vs. premature abstraction), API design, testing pyramid, code review mindset, refactoring triggers, operational readiness checklist.

### `/language/cloud-engineer-pro [description]`
**Fallback for any unmatched cloud or hybrid/on-prem infrastructure.** Provider-agnostic: six architecture pillars, network blueprint (DMZ → app → data tiers), compute and database selection guides, IAM principles (workload identity, least privilege, temporary credentials), IaC patterns, observability (RED/USE methods), DR patterns (backup → active-active), cost control framework (visibility → optimization → governance → accountability), multi-cloud strategies, security hardening checklist.

---

## Quality Commands

Auto-detect stack and apply appropriate tooling. Run anytime, with or without an active issue:

### `/quality/code-audit [scope]`
Full code quality analysis. Runs all available static analysis tools, measures code metrics (coverage, complexity, file length, dependency count), evaluates architecture quality, and produces a scored report with prioritized findings. Recommends relevant `/language/*-pro` commands based on detected stack.

### `/quality/test-strategy [scope]`
Designs test pyramid (unit → integration → E2E), generates ready-to-use test runner configs (Vitest, Pytest, PHPUnit), scaffolds example tests for detected patterns (API routes, components, models), and defines CI integration commands.

### `/quality/lint-setup [scope]`
Configures linter (ESLint, Ruff, PHPStan, tflint), formatter (Prettier, Ruff format, PHP-CS-Fixer, terraform fmt), pre-commit hooks (husky + lint-staged or pre-commit framework), and editor config. Outputs ready-to-commit config files.

### `/quality/dependency-check`
Vulnerability scanning (npm audit, pip audit, composer audit, trivy), outdated package report with migration priorities, license compliance check (flags copyleft), bundle/bloat analysis for frontends, and abandoned package detection.

---

## Common Patterns

### Skip phases for small changes
```bash
# Bug fix — skip discovery and design
/research fix-login-timeout
/plan fix-login-timeout
/implement fix-login-timeout
/review fix-login-timeout
```

### Add AI features to an existing plan
```bash
/discover Add AI-powered search to product catalog
/research add-ai-search
/design-system add-ai-search
/ai-integrate add-ai-search    # ← insert here
/plan add-ai-search
/implement add-ai-search
```

### Check status anytime
```bash
cat .claude/planning/{issue-name}/STATUS.md
```

### Set up quality tooling for a new project
```bash
/quality/lint-setup                  # Configure linter + formatter + hooks
/quality/test-strategy               # Set up test runner + strategy
/quality/dependency-check            # Audit existing dependencies
/quality/code-audit                  # Full quality baseline
```

### Use expert mode for a specific challenge
```bash
/language/typescript-pro Design a type-safe event bus with typed handlers
/language/python-pro Build a FastAPI middleware for rate limiting with Redis
/language/php-pro Create a payment gateway adapter using the Strategy pattern
/language/aws-pro Design a multi-region active-active architecture for an e-commerce platform
/language/azure-pro Migrate a monolith to Azure Container Apps with Dapr
/language/gcp-pro Build a real-time analytics pipeline with Pub/Sub and BigQuery
/language/terraform-pro Create a reusable VPC module with public/private/data subnets
/language/javascript-react-pro Build an accessible data table with sorting, filtering, and virtualization
/language/ansible-pro Create a role for hardening Ubuntu 22.04 servers with CIS benchmarks
/language/kubernetes-pro Design a blue-green deployment strategy with traffic shifting for a gRPC service
/language/openshift-pro Migrate a legacy Java app to OpenShift with S2I builds and restricted-v2 SCC
/language/software-engineer-pro Refactor this module to follow clean architecture with proper dependency inversion
/language/cloud-engineer-pro Design a hybrid cloud DR strategy for our on-prem Oracle database with cloud failover
```
