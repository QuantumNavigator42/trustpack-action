# TrustPack Action

Generate a buyer-ready security due diligence pack from a GitHub repository.

TrustPack Action helps startups, agencies, and SaaS teams prepare repository-level security evidence for customer security reviews. It runs inside GitHub Actions, collects evidence from repository metadata, local repository files, dependency manifests, and workflow YAML, then writes Markdown, JSON, and optional HTML reports as build artifacts.

TrustPack helps collect repository-level security evidence. It is not legal, audit, or compliance certification.

## Quick Start

```yaml
name: Generate TrustPack

on:
  workflow_dispatch:

permissions:
  contents: read
  actions: read
  pull-requests: read
  security-events: read

jobs:
  trustpack:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v6

      - name: Generate repository security evidence pack
        uses: QuantumNavigator42/trustpack-action@v0.1.0
        with:
          output-dir: trustpack
          company-name: Acme SaaS
          product-name: Acme Platform
          include-html: "true"
          fail-on-high-risk-workflows: "false"

      - name: Upload TrustPack artifact
        uses: actions/upload-artifact@v5
        with:
          name: trustpack
          path: trustpack
```

Use `fail-on-high-risk-workflows: "true"` when you want the Action to write reports first, then fail if high or critical workflow-risk findings are detected.

## Outputs

TrustPack writes these files to `output-dir`:

- `executive-summary.md`
- `repository-evidence.json`
- `security-controls.md`
- `vendor-security-questionnaire.md`
- `dependency-inventory.md`
- `open-source-license-notices.md`
- `github-actions-risk-report.md`
- `remediation-checklist.md`
- `index.html` when `include-html` is true

`repository-evidence.json` is the canonical machine-readable output. Markdown reports are buyer-facing summaries built from that evidence.

## Inputs

| Input | Default | Description |
| --- | --- | --- |
| `github-token` | `${{ github.token }}` | Token used for GitHub repository metadata, releases, tags, branch protection, and ruleset evidence. |
| `output-dir` | `trustpack` | Relative output directory inside the workspace. Absolute paths, path traversal, empty path segments, and control characters are rejected. |
| `company-name` | `Your company` | Buyer-facing company name. |
| `product-name` | `Your product` | Buyer-facing product name. |
| `include-html` | `true` | Generates `index.html` when true. Accepts `true`, `false`, `1`, `0`, `yes`, or `no`. |
| `fail-on-high-risk-workflows` | `false` | Writes reports, then fails the Action if high or critical workflow-risk findings are present. |

Invalid inputs fail the Action cleanly through `core.setFailed`.

## Evidence Collected

TrustPack currently collects:

- Repository metadata, default branch, releases, tags, visibility, and license metadata.
- Branch protection and repository ruleset evidence when the token has access.
- Repository file evidence for CODEOWNERS, SECURITY.md, Dependabot config, README, and license files.
- Dependency manifests and lockfiles for Node.js, Python, Go, Rust, and Maven.
- GitHub workflow YAML files from `.github/workflows`.
- Workflow risk findings for broad permissions, `write-all`, `pull_request_target`, unpinned actions, shell interpolation risks, and secret-like hardcoded workflow strings.

## Required Permissions

The example permissions are enough for normal repository checkout and most local evidence collection. GitHub may return partial branch protection or ruleset evidence when the token cannot read those APIs. In that case TrustPack still writes the report pack and records the collector errors in `repository-evidence.json`.

For private repositories, use the default `github-token` first. If branch protection or ruleset evidence is unavailable and you need that section filled in, provide a token with the appropriate repository read access for those APIs.

## Limitations

- TrustPack does not inspect organization membership, repository collaborators, team access, cloud infrastructure, runtime controls, production incidents, or user access reviews.
- Package-level dependency license extraction is not collected in v1; the license report lists detected manifests and lockfiles as source evidence for a follow-up license scan.
- Workflow risk rules are static checks over workflow YAML. They do not prove exploitability and they do not replace security review.
- Missing or partial GitHub API permissions produce partial evidence instead of aborting the whole Action.
- TrustPack does not provide legal, audit, or compliance certification.

## Development Repo Notes

This private repository is the source, test, CI, and release-automation workspace for the public Marketplace action repo. The public `QuantumNavigator42/trustpack-action` repo must remain Marketplace-safe at publication time: root `action.yml`, bundled `dist/`, README, license, and required templates/examples only, with no `.github/workflows` files.

Local gates:

```bash
npm ci
npm run typecheck
npm run lint
npm run build
npm test
npm run bundle
npm run check-dist
npm audit --omit=dev
```
