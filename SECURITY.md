# Security Policy — turfbuild

This is the org-wide security policy for all [`turfbuild`](https://github.com/turfbuild)
repositories. Turf executes real infrastructure changes and handles cloud
provider credentials, so we treat vulnerability reports seriously. A repository
may override this file with its own `SECURITY.md` (e.g. `turf-mcp-server`).

## Reporting a vulnerability

**Do not open a public issue for a security problem.** Report privately via
GitHub's [private vulnerability reporting](https://docs.github.com/en/code-security/security-advisories/guidance-on-reporting-and-writing-information-about-vulnerabilities/privately-reporting-a-security-vulnerability)
on the affected repository (Security → Report a vulnerability), or email
**security@turf.build**.

Please include the affected version, a description, and reproduction steps. We
aim to acknowledge within **3 business days** and to agree a disclosure timeline
with you before any public advisory.

## Supported versions

Turf is in active alpha development; security fixes land on the latest release
line. Pin to the latest tag and prefer the signed release artifacts.

## How we scan

Every push and pull request across the org runs continuous scanning through the
shared reusable workflow
([`turfbuild/.github`](https://github.com/turfbuild/.github)):

- **govulncheck** — reachable Go CVEs (call-graph filtered)
- **osv-scanner** — dependency / lockfile CVEs
- **trivy** — filesystem, container image, misconfig, secrets
- **gosec** — first-party SAST (newly-introduced findings gate PRs)

Public repositories surface results in the GitHub **code-scanning** tab; the
private server surfaces them as workflow artifacts and dedup'd tracking issues.

## Vulnerabilities in the OpenTofu / cagent forks

`turfbuild` maintains forks (`opentofu`, `provider-client`,
`terraform-provider-tfcoremock`, `docker-agent`) consumed as git submodules. A
vulnerability in a fork is remediated by **rebasing on upstream and re-tagging**,
never by suppressing it in a downstream consumer. Until a rebase lands, the
finding is carried in the consumer's OpenVEX disposition as `affected` /
`under_investigation` naming the fork line and pinned tag.

## Disposition & transparency (closed-source binary)

The `turf-mcp-server` binary is proprietary and obfuscated, but its supply-chain
posture is public and verifiable. Every release on
[`turfbuild/turf`](https://github.com/turfbuild/turf/releases) ships, alongside
`checksums.txt`, an SBOM (SPDX + CycloneDX, generated from the module graph), an
**OpenVEX** per-CVE disposition document, and a scan report — all covered by
GitHub-native build & SBOM attestations:

```bash
gh attestation verify turf-mcp-server --repo turfbuild/turf
```
