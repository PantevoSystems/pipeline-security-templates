# Semgrep — Static Application Security Testing (SAST)

[Semgrep](https://semgrep.dev/) is a fast, multi-language static analysis tool that finds security vulnerabilities, code quality issues, and bugs by pattern-matching your source code.

## What it scans

- Source code (40+ languages: Python, JavaScript, TypeScript, Go, Java, Ruby, PHP, C#, Rust, Kotlin, Swift, etc.)
- Configuration files (Dockerfile, YAML)
- Infrastructure as Code (basic — for full IaC use Checkov)

## What it finds

- SQL injection, command injection, path traversal
- Cross-site scripting (XSS)
- Hardcoded secrets and credentials
- Insecure cryptography
- Authentication & authorization flaws
- OWASP Top 10 patterns
- Custom anti-patterns specific to your stack

## Quick start

### Default scan (recommended ruleset)

```yaml
jobs:
  semgrep:
    uses: PantevoSystems/pipeline-security-templates/.github/workflows/semgrep-sast.yml@v1
```

Uses Semgrep's `p/default` ruleset — balanced precision and recall, low false-positive rate.

### Stricter — OWASP Top 10

```yaml
jobs:
  semgrep:
    uses: PantevoSystems/pipeline-security-templates/.github/workflows/semgrep-sast.yml@v1
    with:
      config: 'p/owasp-top-ten'
      severity: 'WARNING'
      fail-on-issues: true
```

### Auto-detect language and pick ruleset

```yaml
jobs:
  semgrep:
    uses: PantevoSystems/pipeline-security-templates/.github/workflows/semgrep-sast.yml@v1
    with:
      config: 'auto'
```

Semgrep auto-detects your stack and picks appropriate rules. Less precise than specific rulesets, but zero-config.

### Exclude test and vendor directories

```yaml
jobs:
  semgrep:
    uses: PantevoSystems/pipeline-security-templates/.github/workflows/semgrep-sast.yml@v1
    with:
      paths-exclude: |
        tests/
        vendor/
        node_modules/
        .venv/
```

## Configuration

| Input | Default | Description |
|---|---|---|
| `config` | `p/default` | Ruleset. See [Semgrep registry](https://semgrep.dev/r) |
| `severity` | `WARNING` | Minimum severity: ERROR, WARNING, INFO. Higher levels are included automatically (`WARNING` → `--severity=WARNING --severity=ERROR`) |
| `fail-on-issues` | `true` | Fail the build on findings |
| `paths-include` | `''` | Paths to include (newline-separated) |
| `paths-exclude` | `''` | Paths to exclude (newline-separated) |

## Recommended rulesets

| Ruleset | Use case |
|---|---|
| `p/default` | General-purpose, balanced — start here |
| `p/security-audit` | Comprehensive security focus, more findings |
| `p/owasp-top-ten` | OWASP Top 10 patterns specifically |
| `p/ci` | CI-friendly subset, low noise |
| `p/secrets` | Hardcoded credentials and API keys |
| `p/dockerfile` | Dockerfile-specific issues |
| `p/javascript` | JS/TS-specific patterns |
| `p/python` | Python-specific patterns |
| `auto` | Auto-detect language and pick rules |

Combine multiple rulesets with comma:

```yaml
with:
  config: 'p/default,p/secrets,p/dockerfile'
```

## Severity levels explained

- **ERROR** — Likely security vulnerability or critical bug. Fix before merge.
- **WARNING** — Code smell or potential issue. Review and fix when reasonable.
- **INFO** — Style or best-practice suggestion. Optional.

For most CI pipelines, `WARNING` is the sweet spot.

## Where to see results

1. **GitHub Security tab** — `Repository → Security → Code scanning alerts`
2. **Pull request annotations** — Findings appear inline on changed code
3. **Action logs** — Plain-text summary in workflow run

## Common gotchas

- **Auto config detection** is sometimes wrong for monorepos. If you have multiple languages, set `config: 'auto'` per directory or list rulesets explicitly.
- **False positives** are inevitable with SAST tools. Use `// nosemgrep` (or `# nosemgrep`) inline comments to dismiss verified-safe patterns.
- **Build time** scales with codebase size. For repos >100k lines, exclude vendored code.
- **Custom rules** — Semgrep lets you write your own patterns. See [docs](https://semgrep.dev/docs/writing-rules/overview/).

## Suppressing findings

Inline (specific line):

```python
# nosemgrep
password = "hardcoded_for_demo"
```

Targeted (specific rule):

```python
# nosemgrep: hardcoded-password
password = "hardcoded_for_demo"
```

File-level via `.semgrepignore`:

Ignore everything in tests/
tests/
Ignore specific file
src/legacy/old_module.py

## Resources

- [Semgrep registry — browse rulesets](https://semgrep.dev/r)
- [Semgrep documentation](https://semgrep.dev/docs/)
- [Writing custom rules](https://semgrep.dev/docs/writing-rules/overview/)
- [Semgrep CLI reference](https://semgrep.dev/docs/cli-reference/)
