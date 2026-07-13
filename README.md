# turfbuild/.github

Org-wide defaults and shared CI for [`turfbuild`](https://github.com/turfbuild)
repositories:

| Path | What it is |
| --- | --- |
| [`.github/workflows/security-scan.yml`](.github/workflows/security-scan.yml) | **Reusable** (`workflow_call`) security-scan workflow |
| [`actions/scan/`](actions/scan) | **Composite** action running the pinned scanners → per-tool SARIF |
| [`SECURITY.md`](SECURITY.md) | Org-default vulnerability-disclosure policy |
| [`renovate-config.json`](renovate-config.json) | Shared Renovate preset (fork paths + submodules disabled) |

The scanners mirror the Turf server's `make scan` (govulncheck, osv-scanner,
gosec via golangci-lint, trivy fs). Tool versions are pinned in
[`actions/scan/action.yml`](actions/scan/action.yml) — keep them in lockstep with
the server's `Makefile`.

## Consume the scan (thin caller)

Add `.github/workflows/security.yml` to a repo:

```yaml
name: Security
on:
  pull_request:
  push:
    branches: [main]
  schedule:
    - cron: "0 15 * * MON"
  workflow_dispatch:

jobs:
  security:
    uses: turfbuild/.github/.github/workflows/security-scan.yml@main
    permissions:
      contents: read
      security-events: write
    with:
      has-submodules: false   # true for the server / CLI (fork submodules)
      is-public: true         # true → code-scanning tab; false → SARIF artifact
```

The composite action is Makefile-independent: a repo needs no `make` targets to
adopt it. Each scanner uses the repo's own config
(`.security/osv-scanner.toml`, `.security/trivy.yaml`, `.golangci.security.yml`)
if present, otherwise a bundled default.

### Just the scan step

To add scanning to an existing job instead of a whole workflow:

```yaml
- uses: actions/checkout@v7
  with: { fetch-depth: 0 }
- uses: turfbuild/.github/actions/scan@main
  with:
    language: go
    sast: diff
    base-ref: origin/${{ github.base_ref }}
```

## Renovate

Reference the shared preset from a repo's `renovate.json`:

```json
{ "extends": ["github>turfbuild/.github:renovate-config"] }
```

Fork module paths (`opentofu`, `provider-client`, `terraform-provider-tfcoremock`,
`docker-agent`) and the git-submodules manager are disabled — forks move only via
the fork-rebase flow (`docs/development/fork-maintenance.md`).

## Notes

- This repo is **public** so its reusable workflow/action and community-health
  defaults are visible to the public forks. The `SECURITY.md` default here
  applies to public repos; the private server repo carries its own.
- Pin consumers to `@main` for now; move to a tag/SHA once the surface settles.
