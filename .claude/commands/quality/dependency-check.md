## Quality: Dependency Health Check

Audit dependencies for vulnerabilities, outdated packages, licensing issues, and bloat. Context: `$ARGUMENTS`

### Instructions

Auto-detect package managers and run a comprehensive dependency health assessment.

### Step 1: Detect Package Managers

```
package.json         → npm / yarn / pnpm
package-lock.json    → npm
yarn.lock            → yarn
pnpm-lock.yaml       → pnpm
composer.json        → Composer (PHP)
pyproject.toml       → pip / poetry / uv
requirements.txt     → pip
Pipfile              → pipenv
Cargo.toml           → cargo (Rust)
go.mod               → Go modules
Gemfile              → Bundler (Ruby)
.terraform.lock.hcl  → Terraform providers
```

### Step 2: Vulnerability Scan

Run all available security audits:

```bash
# JavaScript/TypeScript
npm audit --json                    # or yarn audit / pnpm audit
npx better-npm-audit audit          # More readable output

# Python
pip audit                           # or safety check
pip install pip-audit && pip-audit

# PHP
composer audit

# Terraform
trivy config .

# Generic (works for most ecosystems)
# Trivy or Snyk if available
trivy fs --scanners vuln .
```

Report format:
| Package | Severity | CVE | Description | Fix Available | Action |
|---------|----------|-----|-------------|---------------|--------|
| lodash | High | CVE-XXXX | Prototype pollution | Yes (4.17.21) | Upgrade |

### Step 3: Outdated Dependencies

```bash
# JavaScript
npm outdated                        # Shows current → wanted → latest
npx npm-check-updates               # Shows all available upgrades

# Python
pip list --outdated

# PHP
composer outdated --direct           # Direct deps only (not transitive)

# Terraform
# Check provider changelogs and terraform registry
```

Categorize into:
- **Patch updates** (safe, auto-merge): 1.2.3 → 1.2.4
- **Minor updates** (usually safe, review changelog): 1.2.3 → 1.3.0
- **Major updates** (breaking changes, plan migration): 1.2.3 → 2.0.0

### Step 4: License Compliance

Check for license compatibility:

```bash
# JavaScript
npx license-checker --summary --failOn "GPL-2.0;GPL-3.0;AGPL-3.0"

# Python
pip-licenses --format=table --fail-on="GPL-2.0;GPL-3.0;AGPL-3.0"

# PHP
composer licenses
```

Flag:
- 🔴 **Copyleft** (GPL, AGPL): May require open-sourcing your code
- 🟡 **Weak copyleft** (LGPL, MPL): File-level copyleft, usually OK
- 🟢 **Permissive** (MIT, Apache-2.0, BSD, ISC): Free to use commercially
- ⚪ **Unknown/missing**: Investigate before using

### Step 5: Bundle & Bloat Analysis (Frontend)

```bash
# JavaScript bundle analysis
npx vite-bundle-analyzer            # Vite
npx @next/bundle-analyzer           # Next.js
npx webpack-bundle-analyzer         # Webpack

# Package size check
npx bundlephobia-cli lodash         # Check individual package size

# Detect duplicates
npx depcheck                        # Find unused dependencies
```

Report:
| Package | Size (gzip) | Usage | Alternative |
|---------|-------------|-------|-------------|
| moment | 72KB | Date formatting | dayjs (2KB) |
| lodash | 71KB | 3 functions used | lodash-es (tree-shake) or native |

### Step 6: Dependency Graph Complexity

Analyze dependency depth and risk:

- **Direct dependencies**: Count and list
- **Transitive dependencies**: Total count
- **Maximum depth**: How deep does the tree go
- **Single points of failure**: Packages maintained by 1 person with no funding
- **Abandoned packages**: Last publish > 2 years ago

### Step 7: Report

Produce `DEPENDENCY_HEALTH.md`:

```markdown
# Dependency Health Report

**Date:** {date}
**Package Manager:** {detected}
**Direct Dependencies:** {count}
**Transitive Dependencies:** {count}

## Vulnerability Summary
| Severity | Count | Auto-fixable |
|----------|-------|-------------|
| Critical | N | N |
| High | N | N |
| Medium | N | N |
| Low | N | N |

## Outdated Summary
| Type | Count | Action |
|------|-------|--------|
| Patch | N | Auto-merge safe |
| Minor | N | Review changelog |
| Major | N | Plan migration |

## License Summary
| License | Count | Risk |
|---------|-------|------|
| MIT | N | 🟢 Safe |
| Apache-2.0 | N | 🟢 Safe |
| GPL-3.0 | N | 🔴 Review |
| Unknown | N | ⚪ Investigate |

## Action Items
1. **Immediate**: Fix critical/high vulnerabilities
2. **This sprint**: Update outdated patches
3. **This quarter**: Plan major version migrations
4. **Investigate**: Unknown licenses and abandoned packages

## Unused Dependencies (remove)
- {package}: not imported anywhere
```

### Quality Gates
- All available audit tools were run
- Critical/High vulnerabilities have remediation plans
- No unknown licenses in production dependencies
- Unused dependencies identified for removal
- Report includes specific action items with priority
