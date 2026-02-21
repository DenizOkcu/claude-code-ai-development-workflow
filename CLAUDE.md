# Project Guidelines

## Project Overview
<!-- Describe your project/workspace here -->
This workspace contains multiple repositories for infrastructure, applications, and tooling. The workspace includes Terraform modules, application code, configuration files, and infrastructure automation scripts.

---

## Repository Types

### Infrastructure as Code (Terraform)
When working with Terraform repositories:
- Use consistent naming: `resource_type-purpose-environment`
- Pin provider versions (e.g., `>= 6.0`)
- Always include `variables.tf`, `outputs.tf`, `main.tf`, `versions.tf`
- Never hardcode credentials or secrets
- Use remote state with encrypted backend (S3, GCS, Azure Blob)
- Test changes in dev environment first, when possible
- For expert guidance: `/language/terraform-pro`

### Application Repositories
When working with application code:
- Follow language-specific conventions (Python, PHP, Node.js, Go, etc.)
- Include proper README with setup instructions; if a README already exists, always patch it instead of wiping existing content
- Maintain CI/CD pipeline configurations
- Document environment variables and dependencies
- Include health check endpoints where applicable
- For language-specific guidance: `/language/{lang}-pro`

### Configuration Repositories
When working with configuration files:
- Version all configuration changes
- Document the purpose and impact of each config
- Include examples or templates
- Note which environments use which configs

### Container & Orchestration
When working with Docker, Kubernetes, or OpenShift:
- Use multi-stage builds for smaller images
- Never store secrets in images or manifests — use secret management
- Pin base image versions (no `latest` tag in production)
- Include resource limits and health probes on every workload
- For expert guidance: `/language/kubernetes-pro` or `/language/openshift-pro`

### Automation & Configuration Management
When working with Ansible or similar tools:
- Use roles for reusable logic, playbooks for orchestration
- Never store secrets in plaintext — use Ansible Vault or external secret managers
- Make playbooks idempotent
- Test with `--check` and `--diff` before applying
- For expert guidance: `/language/ansible-pro`

---

## Workflow with Issue Trackers (Jira, GitHub Issues, GitLab Issues)

### Ticket Handling
When working on tickets:
1. **Review ticket thoroughly**: Check description, acceptance criteria, and linked issues — don't make assumptions, clarify unclear points
2. **Identify affected repositories**: Determine which repos need changes
3. **Document changes**: Add implementation notes to ticket comments
4. **Link related work**: Reference other tickets or PRs if applicable
5. **Update status**: Move ticket through workflow as work progresses — always confirm before moving

### Branch Naming Convention
- Feature: `feature/{TICKET}-short-description`
- Bugfix: `bugfix/{TICKET}-short-description`
- Hotfix: `hotfix/{TICKET}-short-description`
- Refactor: `refactor/{TICKET}-short-description`

Use the ticket number in all branches for easy tracking.

---

## Development Workflow (10-Phase SDLC)

This workspace uses a structured 10-phase development lifecycle via slash commands. All workflow artifacts are stored under `.claude/planning/{issue-name}/` and tracked via a central `STATUS.md`.

### Quick Start
```bash
# Full workflow for complex features:
/discover [description]          # Phase 1: Scope + detect tech stack
/research {issue-name}           # Phase 2: Deep codebase analysis
/design-system {issue-name}      # Phase 3: Architecture + ADRs
/plan {issue-name}               # Phase 4: Implementation plan
/implement {issue-name}          # Phase 5: Code + tests
/review {issue-name}             # Phase 6: Code review + QA
/security {issue-name}           # Phase 7: Security audit
/deploy-plan {issue-name}        # Phase 8: Deployment strategy
/observe {issue-name}            # Phase 9: Observability setup
/retro {issue-name}              # Phase 10: Retrospective + knowledge capture

# Medium complexity (skip discovery/design):
/research {issue-name}
/plan {issue-name}
/implement {issue-name}
/review {issue-name}

# Simple change → Just code it directly.

# Emergency production fix:
/hotfix [description]
```

### Issue Name Format
Issue names are **kebab-case**, concise (2-5 words), and descriptive:
- ✅ `add-oauth-auth`, `fix-memory-leak`, `refactor-api-layer`, `update-terraform-modules`
- ❌ `AddOAuthAuth` (not kebab-case), `fix` (too vague), `add-new-auth-with-oauth2` (too long)

### File Organization
```
.claude/planning/
├── {issue-name}/
│   ├── STATUS.md              # Central progress tracker (updated by all commands)
│   ├── DISCOVERY.md           # Scope, success criteria, detected stack
│   ├── CODE_RESEARCH.md       # Research findings
│   ├── ARCHITECTURE.md        # System design + component design
│   ├── ADR-001-*.md           # Architecture Decision Records
│   ├── PROJECT_SPEC.md        # Technical requirements and specs
│   ├── IMPLEMENTATION_PLAN.md # Phased implementation strategy
│   ├── CODE_REVIEW.md         # Review findings and approval status
│   ├── SECURITY_AUDIT.md      # Threat model + OWASP + dependency scan
│   ├── DEPLOY_PLAN.md         # Rollout strategy + rollback playbook
│   ├── OBSERVABILITY.md       # Metrics, logging, alerts, dashboards
│   └── RETROSPECTIVE.md       # Lessons learned
```

### Stack Auto-Detection
The `/discover` command automatically scans the project to detect languages, frameworks, cloud providers, and quality tooling. Detection results are stored in `DISCOVERY.md` and `STATUS.md`, making relevant expert commands available throughout the workflow.

### Expert Commands (language, cloud, quality)
```bash
# Language & Framework experts (auto-detected during /discover):
/language/typescript-pro [task]         # Strict types, generics, branded types
/language/javascript-react-pro [task]   # React 19, accessibility, performance
/language/php-pro [task]                # PHP 8.2+, Laravel/Symfony patterns
/language/python-pro [task]             # Python 3.11+, FastAPI/Django, typing
/language/terraform-pro [task]          # Modules, validation, security
/language/aws-pro [task]                # Well-Architected, IAM, VPC, cost
/language/azure-pro [task]              # Managed Identity, Bicep, Defender
/language/gcp-pro [task]                # Workload Identity, Cloud Run, SRE
/language/ansible-pro [task]            # Roles, idempotency, Vault, molecule
/language/kubernetes-pro [task]         # Deployments, RBAC, networking, GitOps
/language/openshift-pro [task]          # Routes, SCCs, operators, builds
/language/software-engineer-pro [task]  # Fallback: SOLID, clean arch, testing (any lang)
/language/cloud-engineer-pro [task]     # Fallback: provider-agnostic infra & ops

# Quality tooling (run anytime):
/quality/code-audit [scope]             # Full static analysis + metrics
/quality/test-strategy [scope]          # Test pyramid + config generation
/quality/lint-setup [scope]             # Linter + formatter + hooks setup
/quality/dependency-check               # Vulnerability + license audit

# DevOps:
/devops/ci-pipeline [context]           # Generate CI/CD pipeline config

# Bonus:
/ai-integrate {issue-name}             # LLM/AI integration patterns
/perf-test {issue-name}                # Performance testing + benchmarks
```

### STATUS.md — Progress Dashboard
All commands update `STATUS.md` as the single source of truth:
```bash
cat .claude/planning/{issue-name}/STATUS.md
```

---

## Common Tasks

### Working Across Multiple Repositories
1. **Identify dependencies**: Check if changes require updates in multiple repos — when in doubt, ask the user
2. **Coordinate changes**: Plan the order of changes and deployments
3. **Test integration**: Verify changes work together across repositories
4. **Document cross-repo impacts**: Note dependencies in commit messages and PRs

### Finding Related Code
When searching for specific functionality:
- Use repository naming patterns to narrow search
- Check README files first for high-level overview
- Look for CONTRIBUTING.md or ARCHITECTURE.md files
- Search commit history for context on why code exists

### Handling Legacy Systems
1. **Understand current state**: Research before changing
2. **Create migration plan**: Document steps and rollback procedures
3. **Update in stages**: Make incremental changes when possible
4. **Verify functionality**: Test each stage before proceeding
5. **Update documentation**: Keep docs in sync with code changes

---

## Security Requirements

### Credentials and Secrets
- Never commit credentials, API keys, or secrets
- Always run a secret detection scan (gitleaks or equivalent) before committing
- False positives must be added to `.gitleaksignore` or equivalent exclusion file
- Use secret management services (Parameter Store, Secrets Manager, Key Vault, etc.)
- Reference secrets via environment variables
- Rotate credentials regularly per security policy
- If `.pre-commit-config.yaml` exists at the repo root, secret scanning runs automatically on commit — no need for manual runs

### Access Control
- Follow least privilege principle
- Document IAM policies and permissions
- Use role-based access control (RBAC)
- Review and audit access regularly

### Code Security
- Scan for vulnerabilities in dependencies (`/quality/dependency-check`)
- Review code for security issues before merging
- Keep dependencies up to date
- Follow OWASP best practices for applications
- Use the `/security {issue-name}` phase for formal threat modeling

---

## Repository-Specific Guidelines

### When Working with Terraform
- Run `terraform validate` and `terraform fmt` before committing
- Always review `terraform plan` output
- Use workspaces or separate state files for environment isolation
- Tag all resources consistently
- Document module dependencies
- Use `/language/terraform-pro` for advanced patterns

### When Working with Applications
- Follow the repository's coding standards
- Run tests locally before pushing
- Update version numbers appropriately
- Check CI/CD pipeline passes
- Update changelog for significant changes
- Use the appropriate `/language/*-pro` command for the stack

### When Working with Kubernetes/OpenShift
- Never use `latest` image tag in production manifests
- Always set resource requests and limits
- Include liveness, readiness, and startup probes
- Use namespaces for environment/team isolation
- Store sensitive config in Secrets (never ConfigMaps)
- Use `/language/kubernetes-pro` or `/language/openshift-pro`

### When Working with Ansible
- Use roles for reusable automation, playbooks for orchestration
- Encrypt secrets with Ansible Vault — never commit plaintext secrets
- Always test with `--check --diff` before applying
- Tag tasks for selective execution
- Use `/language/ansible-pro` for advanced patterns

### When Working with Scripts
- Include usage documentation in script header
- Add error handling and validation
- Make scripts idempotent where possible
- Test in non-production first
- Document required permissions/dependencies

---

## Standards and Conventions

### Naming Conventions

**Cloud Resources:**
- Format: `{project}-{environment}-{resource}-{purpose}`
- Example: `myapp-prod-rds-primary`
- Use lowercase and hyphens only

**Files and Directories:**
- Use descriptive names that indicate purpose
- Follow language/framework conventions
- Keep names concise but meaningful

**Kubernetes/OpenShift Resources:**
- Format: `{app}-{component}` (e.g., `api-deployment`, `web-service`)
- Labels: `app.kubernetes.io/name`, `app.kubernetes.io/component`, `app.kubernetes.io/part-of`

### Documentation Standards
- Keep README files up to date
- Document "why" not just "what"
- Include setup/deployment instructions
- Note known issues or limitations
- Provide contact information for questions

### Code Review Checklist
Before submitting PR:
- [ ] Code follows repository conventions
- [ ] Tests pass locally
- [ ] Documentation updated
- [ ] No sensitive data in code (secret scan passed)
- [ ] Security implications considered
- [ ] Breaking changes documented
- [ ] Ticket linked in PR
- [ ] Cross-repo impacts documented (if applicable)
- [ ] `terraform fmt` and `terraform validate` pass (for IaC changes)
- [ ] Resource limits and probes defined (for K8s/OCP changes)

---

## Testing Approach

### Infrastructure Changes
- Validate syntax/configuration (`terraform validate`, `ansible-lint`, `kubeval`)
- Review plan output carefully
- Test in dev/staging before production
- Have rollback plan ready
- Monitor after deployment

### Application Changes
- Run unit tests
- Run integration tests
- Test in staging environment
- Verify logging and monitoring
- Check performance impact

### Configuration Changes
- Validate syntax
- Test in isolated environment
- Document expected behavior
- Plan rollback procedure
- Coordinate with affected teams

---

## Environment Information

### Environments
- **Dev**: Development and testing environment
- **Staging**: Pre-production for validation
- **Production**: Live production environment

### Deployment Process
- Changes flow: Dev → Staging → Production
- Deployments coordinated through issue tracker
- Production changes require approval
- Rollback procedures documented per service

---

## Tools and Services

### Primary Tools
- **Issue Tracker**: Task and project management (Jira, GitHub Issues, GitLab Issues)
- **Git**: Version control
- **Docker**: Containerization
- **Terraform**: Infrastructure as Code
- **Ansible**: Configuration management and automation
- **Kubernetes/OpenShift**: Container orchestration

### CI/CD
- Pipeline configurations in each repository
- Automated testing on PR creation
- Deployment automation to dev/staging
- Manual approval for production

---

## Important Notes for Claude

### Behavioral Guidelines
- **Multiple repos**: Always ask which repository/ies are relevant to the task
- **Cross-repo impacts**: Consider dependencies between repositories
- **Context preservation**: Reference specific file paths and repo names
- **Security first**: Never suggest exposing secrets or credentials
- **Incremental changes**: Prefer small, testable changes over large refactors
- **Documentation**: Always suggest updating docs when making changes
- **Team coordination**: Note when changes require team notification
- **Workflow tracking**: Use the development workflow commands for non-trivial changes to maintain traceability via STATUS.md
- **Secret scanning**: Always run secret detection before committing; add false positives to exclusion files
- **README preservation**: When a README exists, patch it — never wipe existing content

### When Reviewing Code
- Consider impact across multiple repositories
- Look for dependencies between services
- Check for consistency with existing patterns
- Identify potential breaking changes
- Suggest improvements for maintainability

### When Making Changes
- Verify which repositories are affected
- Check for similar code in other repos
- Consider reusability opportunities
- Think about deployment order
- Plan for backward compatibility

### When Troubleshooting
- Look at logs across related services
- Check recent changes in all affected repos
- Verify configuration consistency
- Review monitoring/alerting data
- Consider infrastructure changes

---

## Quick Reference Commands

### Terraform
```bash
terraform fmt -recursive          # Format all .tf files
terraform validate                # Validate configuration
terraform plan -out=tfplan        # Create execution plan
terraform apply tfplan            # Apply changes
```

### Docker
```bash
docker ps                         # List running containers
docker images                     # List images
docker system prune               # Clean up unused resources
```

### Kubernetes / OpenShift
```bash
kubectl get pods -A               # All pods across namespaces
kubectl describe pod {name}       # Pod details and events
kubectl logs {pod} -f             # Stream logs
kubectl apply --dry-run=client -f manifest.yaml  # Validate before apply
oc get routes                     # OpenShift routes
oc adm policy who-can get pods    # RBAC check
```

### Ansible
```bash
ansible-playbook site.yml --check --diff   # Dry-run with diff
ansible-lint playbook.yml                   # Lint playbook
ansible-vault encrypt secrets.yml           # Encrypt secrets file
ansible-inventory --graph                   # Show inventory tree
```

### Workflow
```bash
# Check workflow status for an issue
cat .claude/planning/{issue-name}/STATUS.md

# Full 10-phase workflow
/discover [description]           # Phase 1: Scope + stack detection
/research {issue-name}            # Phase 2: Codebase analysis
/design-system {issue-name}       # Phase 3: Architecture + ADRs
/plan {issue-name}                # Phase 4: Implementation plan
/implement {issue-name}           # Phase 5: Code + tests
/review {issue-name}              # Phase 6: Code review
/security {issue-name}            # Phase 7: Security audit
/deploy-plan {issue-name}         # Phase 8: Deployment strategy
/observe {issue-name}             # Phase 9: Observability
/retro {issue-name}               # Phase 10: Retrospective
```

---

## Contact Information
- **Infrastructure Lead**: [Name/Email]
- **Team Lead**: [Name/Email]
- **Security Contact**: [Name/Email]
- **Emergency**: [On-call rotation info]

---

## Learnings (auto-updated by /retro)
<!-- The /retro command appends lessons learned here automatically -->
