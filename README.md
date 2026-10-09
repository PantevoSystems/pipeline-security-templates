# 🔒 Pipeline Security Templates

**Production-ready security scan templates for GitHub Actions, GitLab CI, and Bitbucket Pipelines.**  
Add Trivy, Semgrep, and Checkov to your CI pipeline in minutes — copy-paste ready.

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![GitHub Actions](https://img.shields.io/badge/GitHub_Actions-supported-blue?logo=github-actions)](./.github/workflows/)
[![GitLab CI](https://img.shields.io/badge/GitLab_CI-supported-orange?logo=gitlab)](./gitlab-ci/)
[![Bitbucket Pipelines](https://img.shields.io/badge/Bitbucket-supported-blue?logo=bitbucket)](./bitbucket-pipelines/)

---

## What's included

| Tool | Purpose | Speed |
|---|---|---|
| **Trivy** | Container & dependency vulnerability scanning | <30s |
| **Semgrep** | SAST — finds code vulnerabilities (SQLi, XSS, hardcoded secrets) | <2min |
| **Checkov** | IaC scanning — Terraform, Kubernetes, CloudFormation | <1min |

All templates run in parallel by default. No external services required.

---

## Quick start

### GitHub Actions

```yaml
# .github/workflows/security.yml
name: Security
on: [push, pull_request]

jobs:
  security:
    uses: PantevoSystems/pipeline-security-templates/.github/workflows/full-stack.yml@v1
```

→ Findings land in your repo's **Security tab**.

### GitLab CI

```yaml
# .gitlab-ci.yml
include:
  - remote: 'https://raw.githubusercontent.com/PantevoSystems/pipeline-security-templates/main/gitlab-ci/full-stack.yml'

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

### Bitbucket Pipelines

Bitbucket doesn't support remote includes. Copy [`bitbucket-pipelines/full-stack.yml`](./bitbucket-pipelines/full-stack.yml) into your `bitbucket-pipelines.yml`.

---

## Bonus: deployment risk scoring

Want a single 0–100 risk score before every deployment? Try [Deployment Guard](https://www.pantevosystems.com/deployment-guard) — analyzes diff complexity, K8s changes, dependency risk, incident history, **and the security findings from these templates** (Trivy CVEs, Semgrep findings, Checkov failures). All factors flow into one deterministic score. **Free tier: 30 analyses/month, no credit card.**

```yaml
jobs:
  security:
    uses: PantevoSystems/pipeline-security-templates/.github/workflows/full-stack-with-guard.yml@v1
    permissions:
      contents: read
      security-events: write
      actions: read
    with:
      trivy-dockerfile: 'Dockerfile'
      semgrep-config: 'p/default'
      checkov-directory: '.'
    secrets:
      guard-api-key: ${{ secrets.GUARD_API_KEY }}
```

Findings show up in your repo's Security tab **and** in the unified risk score. [Get a free API key →](https://www.pantevosystems.com/signup)

## Battle-tested

These templates run in production on:

- **[Deployment Guard](https://www.pantevosystems.com/deployment-guard)** — deterministic deployment risk scoring
  Eats its own dogfood: every Deployment Guard release is scanned by these exact templates before deploy.
- **[Pantevo Systems](https://www.pantevosystems.com)** — DevOps tools for teams of 5–50

We use them ourselves before recommending them to you.

---

## Documentation

- [Trivy → container & dependency CVEs](./docs/trivy.md)
- [Semgrep → static code analysis](./docs/semgrep.md)
- [Checkov → infrastructure as code](./docs/checkov.md)
- [Comparison: which tool when?](./docs/comparison.md)
- [GitLab CI setup guide](./docs/gitlab-ci.md)
- [Bitbucket Pipelines setup guide](./docs/bitbucket-pipelines.md)
- [Deployment Guard integration](./docs/deployment-guard.md)

---

## Examples

Working example repos with templates configured:

- **[Node.js Express API](./examples/nodejs-app/)** — Express + Postgres + Kubernetes manifest. Includes intentional vulnerabilities for demo purposes.

More examples coming soon — PRs welcome.

---

## Why these templates exist

Most DevSecOps tools require:
- Separate dashboards
- Paid SaaS subscriptions
- Hours of YAML wrangling

These templates skip all that. Open-source tools, copy-paste config, no signup.

Built by [Pantevo Systems](https://www.pantevosystems.com) — focused on DevOps tooling for teams of 5–50.

---

## Versioning

We follow [SemVer](https://semver.org/). Pin to a release tag in production:

```yaml
uses: PantevoSystems/pipeline-security-templates/.github/workflows/full-stack.yml@v1
```

`@main` is the dev branch — fine for testing, **don't use in production**.

[See releases →](../../releases)

---

## Contributing

PRs welcome — see [CONTRIBUTING.md](./CONTRIBUTING.md).

Issues, questions, suggestions: [open an issue](../../issues) or email [info@pantevosystems.com](mailto:info@pantevosystems.com).

---

## Security

Found a vulnerability in these templates or in the way they're configured? Please follow our [security policy](./SECURITY.md) — don't open a public issue.

---

## License

MIT — use freely, attribution appreciated.

---

*Built with ❤️ in Germany*
