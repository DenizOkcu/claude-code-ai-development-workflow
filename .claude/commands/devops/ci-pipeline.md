## DevOps: CI/CD Pipeline Generation

Generate or update CI/CD pipeline configuration. Context: `$ARGUMENTS`

### Instructions

Generate a production-ready CI/CD pipeline configuration based on the project's stack and needs.

#### 1. Detect Project Stack

Scan the repository for:
- Package manager (npm, yarn, pnpm, pip, cargo)
- Language/framework (Node.js, Python, Rust, Go)
- Test runner (vitest, jest, pytest, etc.)
- Linter (eslint, ruff, clippy, etc.)
- Build tool (vite, webpack, esbuild, etc.)
- Deployment target (Vercel, AWS, GCP, Docker, etc.)
- Database (PostgreSQL, MongoDB, Redis, etc.)

#### 2. Generate Pipeline

Create a CI/CD configuration with these stages:

```yaml
# Stage 1: Validate
- Lint code
- Type check
- Check formatting
- Validate dependencies (audit)

# Stage 2: Test
- Unit tests (with coverage)
- Integration tests
- E2E tests (if applicable)

# Stage 3: Build
- Production build
- Bundle size check (fail if over threshold)
- Docker image build (if applicable)

# Stage 4: Security
- Dependency vulnerability scan
- SAST (static analysis)
- Secret detection

# Stage 5: Deploy (conditional)
- Staging deploy (on PR merge to develop)
- Production deploy (on release tag or main merge)
- Post-deploy smoke tests
```

#### 3. Platform-Specific Output

Generate for the detected or requested platform:
- **GitHub Actions**: `.github/workflows/ci.yml`
- **GitLab CI**: `.gitlab-ci.yml`
- **Other**: Specify in `$ARGUMENTS`

#### 4. Best Practices Applied

- Cache dependencies between runs
- Fail fast on lint/type errors (before slow tests)
- Parallel test execution where possible
- Matrix testing for multiple Node/Python versions (if needed)
- Branch protection rules documented
- Required status checks listed
- Secrets management guidance

### Output
- Pipeline configuration file(s)
- Brief documentation of what each stage does
- Required secrets/environment variables list
