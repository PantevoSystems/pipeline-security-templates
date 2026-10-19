# GitLab CI Templates — Setup Guide

GitLab CI templates for Trivy, Semgrep, Checkov, and Deployment Guard.

Unlike GitHub Actions' Reusable Workflows, GitLab uses an `include` + `extends` pattern. Templates define a base job (`.template_name`), and your `.gitlab-ci.yml` extends it with optional overrides.

## Quick start

### Single tool — Trivy

```yaml
# .gitlab-ci.yml
include:
  - remote: 'https://raw.githubusercontent.com/PantevoSystems/pipeline-security-templates/v1.2.3/gitlab-ci/trivy.yml'

stages:
  - test

trivy_scan:
  extends: .trivy_template
  stage: test
```

That's it. Push and watch the pipeline.

> 📌 Includes are pinned to a release tag (`v1.2.3`). Don't include from `main` in production — a change on `main` would run in your pipeline without review.

### Full stack — all three tools

```yaml
# .gitlab-ci.yml
include:
  - remote: 'https://raw.githubusercontent.com/PantevoSystems/pipeline-security-templates/v1.2.3/gitlab-ci/full-stack.yml'

stages:
  - security

trivy_scan:
  extends: .trivy_template
  stage: security

semgrep_scan:
  extends: .semgrep_template
  stage: security

checkov_scan:
  extends: .checkov_template
  stage: security
```

All three jobs run in parallel within the `security` stage.

### Full stack + Deployment Guard

```yaml
# .gitlab-ci.yml
include:
  - remote: 'https://raw.githubusercontent.com/PantevoSystems/pipeline-security-templates/v1.2.3/gitlab-ci/full-stack-with-guard.yml'

stages:
  - security
  - risk

trivy_scan:
  extends: .trivy_template
  stage: security

semgrep_scan:
  extends: .semgrep_template
  stage: security

checkov_scan:
  extends: .checkov_template
  stage: security

deployment_guard:
  extends: .deployment_guard_template
  stage: risk
```

Set `GUARD_API_KEY` in your repo's CI/CD variables (Settings → CI/CD → Variables → Add variable, masked: yes).

The `.deployment_guard_template` job sets `GIT_DEPTH: "0"` (full clone) itself and diffs against `CI_MERGE_REQUEST_DIFF_BASE_SHA` (merge requests), `CI_COMMIT_BEFORE_SHA` (pushes) or `HEAD~1` (fallback). Override with `GUARD_DIFF_BASE`.

| Variable | Default | Effect |
|---|---|---|
| `GUARD_FAIL_ON_WARN` | `false` | Also fail on WARN |
| `GUARD_FAIL_OPEN` | `false` | `true`: API unreachable / 429 / 5xx / invalid response only warns. Default: the job fails (fail-closed). Invalid key (401/403) and rejected requests (400/422) always fail. |
| `GUARD_ALLOW_MISSING_KEY` | `false` | `true`: skip instead of fail when `GUARD_API_KEY` is not set |
| `GUARD_API_URL` | `https://api.pantevosystems.com` | API base URL (https only) |

Sent to the API: project path, branch, commit SHA, lines added / removed, files changed. No source code, no file contents, no file paths.

## Customization

Override variables in your job:

```yaml
trivy_scan:
  extends: .trivy_template
  variables:
    TRIVY_IMAGE: "registry.example.com/myapp:latest"
    TRIVY_SEVERITY: "CRITICAL,HIGH,MEDIUM"
    TRIVY_FAIL_ON_ISSUES: "false"
```

See each individual template for available variables.

## Where to see results

### GitLab Free Tier

- **Job logs** — full output in the pipeline run
- **Artifacts** — JSON/SARIF reports downloadable from each job

### GitLab Premium / Ultimate

- **Security Dashboard** — Findings appear in `Security & Compliance` (Container Scanning, SAST)
- **Merge Request widgets** — Vulnerability summaries show on MRs
- **Vulnerability Reports** — Searchable, filterable, dismissible

## Common gotchas

### "Docker-in-Docker required" (Trivy)

If your `.gitlab-ci.yml` doesn't already have a Dockerfile-build step, Trivy needs to build the image itself, which requires DinD.

**Solution 1** — pre-build the image in an earlier stage:

```yaml
build:
  stage: build
  script:
    - docker build -t myapp:latest .

trivy_scan:
  extends: .trivy_template
  stage: test
  variables:
    TRIVY_IMAGE: "myapp:latest"
```

**Solution 2** — use Docker-in-Docker:

```yaml
trivy_scan:
  extends: .trivy_template
  services:
    - docker:dind
```

### "Could not find previous commit" (Deployment Guard)

The Guard requires the base commit to compute the diff. The template sets `GIT_DEPTH: "0"` (full clone); if you override `GIT_DEPTH` for the job or globally, keep it at `0`:

```yaml
variables:
  GIT_DEPTH: "0"
```

If the base commit is missing, the job fails instead of scoring an empty diff.

### "Reports not appearing in Security Dashboard"

GitLab's Security Dashboard requires **Premium** or **Ultimate** tier. On Free, reports are only available as job artifacts.

## GitLab.com vs. self-hosted GitLab

These templates work on both. For self-hosted with restricted internet access, you may need to:

- Mirror the template `.yml` files to your internal Git
- Pre-pull the Docker images (`aquasec/trivy`, `semgrep/semgrep`, `bridgecrew/checkov`) to your registry
- Replace `remote:` includes with `local:` or `project:` includes
