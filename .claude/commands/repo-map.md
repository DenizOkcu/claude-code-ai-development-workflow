---
model: sonnet
---

## Repository Map Generator

Generate a compact structural overview of the current repository — file tree with top-level symbols (classes, functions, exports). Designed to give the LLM a "bird's eye view" of the codebase in ≤2K tokens.

### Instructions

#### 1. Detect Primary Language

If `$ARGUMENTS` specifies a language, use it. Otherwise, auto-detect by checking for marker files:

| Marker | Language | Symbol Pattern |
|--------|----------|---------------|
| `tsconfig.json` or `*.ts` files | TypeScript | `^export (default )?(function\|class\|const\|type\|interface\|enum)\s+\w+` |
| `package.json` (no tsconfig) | JavaScript | `^export (default )?(function\|class\|const)\s+\w+` |
| `pyproject.toml` or `*.py` files | Python | `^(class \|def \|async def )\w+` |
| `go.mod` | Go | `^(func \|type \w+ (struct\|interface))` |
| `composer.json` | PHP | `^(class \|function \|interface \|trait )\w+` |
| `Cargo.toml` | Rust | `^(pub )?(fn \|struct \|enum \|trait \|impl )\w+` |
| (none matched) | Generic | `^(class \|function \|def \|export \|pub fn \|func )\w+` |

If multiple languages are detected, use the primary one (most source files) and note the secondary ones.

#### 2. Build File Tree

Use `Glob` to list all source files, **excluding** these patterns:
- `node_modules/**`, `vendor/**`, `dist/**`, `build/**`, `.next/**`, `__pycache__/**`
- `.git/**`, `.terraform/**`, `.claude/planning/**`
- `*.lock`, `*.min.js`, `*.min.css`, `*.map`, `*.pyc`
- `*.png`, `*.jpg`, `*.gif`, `*.svg`, `*.ico`, `*.woff`, `*.woff2`, `*.ttf`, `*.eot`
- `coverage/**`, `.nyc_output/**`, `.pytest_cache/**`

Focus on source code files for the detected language:
- TypeScript/JavaScript: `**/*.{ts,tsx,js,jsx}` (exclude `*.d.ts`, `*.test.*`, `*.spec.*` from symbol extraction — still list them)
- Python: `**/*.py`
- Go: `**/*.go`
- PHP: `**/*.php`
- Rust: `**/*.rs`
- Generic: `**/*.{ts,tsx,js,jsx,py,go,php,rs,java,rb,swift,kt}`

#### 3. Extract Top-Level Symbols

For each source file (excluding test files), run `Grep` with the detected language pattern to extract top-level symbols (classes, functions, exports, types).

**Rules:**
- Only extract **top-level** symbols — not methods inside classes, not nested functions
- For test files: just mark as `(test file)` — don't extract individual test names
- For config files (`*.config.*`, `*.json`, `*.yaml`, `*.toml`): list the file but don't extract symbols
- For Markdown files: skip entirely (they're documentation, not code)

#### 4. Format the Map

Use this compact format — file path with inline symbols separated by commas:

```
## Repository Map (~{N} source files, {language})

{directory}/
  {subdirectory}/
    {file} — {Symbol1}, {Symbol2}(), {Symbol3}
    {file} — {Symbol1}, {Symbol2}()
  {subdirectory}/
    {file} — {Symbol1}(), {Symbol2}()
{directory}/
  {file} — (test file)
  {file} — (config)
```

**Formatting rules:**
- Use 2-space indentation for directory nesting
- Append `()` to function/method names, nothing for classes/types/interfaces
- Group files by directory, sorted alphabetically
- Show directory structure as a tree (directories on their own line, files indented under them)
- For files with >8 symbols: show the first 6 and append `... +{N} more`
- Root-level files come first, then directories alphabetically

#### 5. Apply Token Budget

**Target: ≤2K tokens** (~8K characters). Apply progressive truncation:

**Small repos (<100 source files):**
- Full file tree with symbols for all source files
- Include test files (marked as `(test file)`)
- Include config files (marked as `(config)`)

**Medium repos (100–200 source files):**
- Full symbols for primary source directories (`src/`, `lib/`, `app/`, `pkg/`, `internal/`, `cmd/`)
- For other directories: list files but omit symbols
- Summarize test directories: `tests/ — {N} test files`

**Large repos (200–500 source files):**
- Full symbols for primary source directories only
- Summarize all other directories: `{dir}/ — {N} files`
- Omit test files entirely

**Very large repos (>500 source files):**
- Directory-level summary only — no per-file listing for non-primary dirs
- For primary source directories: list files with symbols, max 50 files
- Add note: `⚠ Large repository ({N} files). Showing primary source directories only. Run with specific path for details.`

If `$ARGUMENTS` contains a path (e.g., `/repo-map src/auth`), generate the map for that subdirectory only at full detail regardless of repo size.

#### 6. Output

Present the map directly to the user. If generating for `/discover` integration, the map will be embedded in `01_DISCOVERY.md`.

After the map, add:
```
**Files:** {N} source | {M} test | {K} config
**Primary language:** {language}
**Key entry points:** {list 1-3 likely entry point files: index.ts, main.py, cmd/main.go, etc.}
```

### Quality Gates
- Map stays within ~2K token budget (~8K characters)
- All listed file paths are real (verified via Glob)
- Symbol extraction used the correct language pattern
- Large repo truncation applied appropriately
- No binary files, lock files, or generated files included
- Directory structure is accurate and sorted
