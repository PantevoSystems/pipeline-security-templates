# Bitbucket Pipelines Templates — Setup Guide

Bitbucket Pipelines templates for Trivy, Semgrep, Checkov, and Deployment Guard.

Unlike GitHub Actions and GitLab CI, **Bitbucket Pipelines doesn't support remote `include:`** the way the other two platforms do. You'll **copy** template content into your `bitbucket-pipelines.yml` directly.

## Quick start — single tool

The simplest possible Trivy scan. Create `bitbucket-pipelines.yml` in your repo root:

```yaml
image: atlassian/default-image:4

pipelines:
  default:
    - step:
        name: Trivy Filesystem Scan
        image: aquasec/trivy:0.55.0
        script:
          - trivy fs --severity CRITICAL,HIGH --ignore-unfixed --exit-code 1 .
```

Push. Bitbucket runs the pipeline. Done.

## Full security stack

For Trivy + Semgrep + Checkov in parallel, copy the contents of [`full-stack.yml`](../bitbucket-pipelines/full-stack.yml) into your `bitbucket-pipelines.yml`.

**Total runtime:** ~3-5 minutes for typical repos.

## Full stack + Deployment Guard

Copy [`full-stack-with-guard.yml`](../bitbucket-pipelines/full-stack-with-guard.yml) and:

1. Get an API key at [pantevosystems.com/signup](https://www.pantevosystems.com/signup)
2. Add `GUARD_API_KEY` as a **secured Repository Variable**:
   - Repository Settings → Repository Variables
   - Name: `GUARD_API_KEY`
   - Value: your key
   - **Secured: yes** (essential — keys are sensitive)

## Customization via Repository Variables

Set these in `Repository Settings → Repository Variables` to override defaults:

| Variable | Default | Used by |
|---|---|---|
| `TRIVY_SEVERITY` | `CRITICAL,HIGH` | Trivy |
| `SEMGREP_RULES` | `p/default` | Semgrep |
| `SEMGREP_SEVERITY` | `WARNING` | Semgrep |
| `CHECKOV_DIRECTORY` | `.` | Checkov |
| `CHECKOV_FRAMEWORK` | `all` | Checkov |
| `CHECKOV_SKIP_CHECKS` | `''` | Checkov |
| `GUARD_API_KEY` | _none_ | Deployment Guard |
| `GUARD_FAIL_ON_WARN` | `false` | Deployment Guard |

## Where to see results

Bitbucket has a more limited Security UI compared to GitHub or GitLab Premium:

1. **Pipeline logs** — full output in each step
2. **Artifacts** — JSON / SARIF files downloadable from each step
3. **Pull request status** — Pipelines show pass/fail on PRs

For detailed vulnerability tracking, consider exporting artifacts to your own dashboard or using a paid Bitbucket app like Snyk or Sonatype.

## Common gotchas

### "Cannot use Docker" (Trivy image scanning)

Bitbucket Pipelines runs steps in containers. To run `docker build` inside a step you need the `services: docker` declaration.

For most users: **prefer Trivy filesystem scanning** (`trivy fs`) over container scanning. It works without Docker, scans dependency files directly, and is faster.

### "Could not find previous commit" (Deployment Guard)

The default Bitbucket clone is shallow. The guard step uses a full clone and diffs against `HEAD~1` (Bitbucket provides no "before" SHA; override with the variable `GUARD_DIFF_BASE`):

```yaml
- step:
    clone:
      depth: full
```

If the base commit is missing, the step fails instead of scoring an empty diff.

Guard variables: `GUARD_FAIL_ON_WARN` (default `false`), `GUARD_FAIL_OPEN` (default `false` — API unreachable / 429 / 5xx / invalid response fail the step; invalid key and rejected requests always fail), `GUARD_ALLOW_MISSING_KEY` (default `false` — a missing `GUARD_API_KEY` fails the step).

Sent to the API: repository name, branch, commit SHA, lines added / removed, files changed. No source code, no file contents, no file paths.

### Pipeline minutes

Bitbucket Free includes 50 minutes/month. The full-stack workflow uses ~5 min per run. Adjust trigger conditions to avoid running on every commit:

```yaml
pipelines:
  default:        # runs on every push to any branch — expensive
    - ...

  pull-requests:  # runs only on PRs — cheaper
    '**':
      - ...
```

## Differences from GitHub / GitLab

| Feature | GitHub | GitLab | Bitbucket |
|---|---|---|---|
| Remote includes | ✅ Reusable Workflows | ✅ `include: remote:` | ❌ Copy-paste only |
| Native security UI | ✅ Free | 🟡 Premium tier | 🟡 Apps only |
| Parallel jobs | ✅ Native | ✅ Native | ✅ `parallel:` block |
| SARIF support | ✅ Built-in | 🟡 Premium | ❌ Artifact only |

If you have flexibility on platform choice, GitHub Actions has the best free-tier security tooling. Bitbucket works fine but expect to do more work for the same UX.
