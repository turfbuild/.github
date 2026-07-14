<div align="center">

<img src="https://raw.githubusercontent.com/turfbuild/.github/main/profile/turf-logo.png" alt="Turf" width="200" />

# Turf

**A drop-in replacement for Terraform with agentic superpowers.**

Full support for Terraform HCL and the module registry — exposed to AI agents as an
infrastructure-management MCP server built on OpenTofu providers.

[turf.build](https://turf.build) · [Install](#get-started) · [Examples](https://github.com/turfbuild/turf-examples)

</div>

---

## Get started

```bash
brew install turfbuild/tap/turf
```

Turf ships as two pieces from a single release: the **Turf CLI** and **`turf-mcp-server`**, an
MCP server that exposes infrastructure tools (plan/apply, provider control, state) to any MCP
client — Claude Desktop, cagent, kagent, and more.

## Featured

| Repo | Description |
| --- | --- |
| [**turf**](https://github.com/turfbuild/turf) | The Turf CLI and `turf-mcp-server` — alpha binary releases. |
| [**turf-examples**](https://github.com/turfbuild/turf-examples) | Reference integrations, agent definitions, and Terraform/HCL examples. |

## Open source

Turf is built on OpenTofu and a small set of MPL-2.0 forks we maintain in the open:

- [opentofu](https://github.com/turfbuild/opentofu) — our OpenTofu fork
- [provider-client](https://github.com/turfbuild/provider-client) — OpenTofu provider-client fork
- [terraform-provider-tfcoremock](https://github.com/turfbuild/terraform-provider-tfcoremock) — mock provider used in tests
- [docker-agent](https://github.com/turfbuild/docker-agent) — downstream cagent fork powering the CLI

## Security

Turf is alpha software with a transparent supply-chain posture: every push runs continuous
scanning (govulncheck, osv-scanner, gosec, trivy), and releases ship SBOMs (SPDX + CycloneDX),
OpenVEX disposition documents, and scan reports with GitHub-native build & SBOM attestations.
See our [security policy](https://github.com/turfbuild/.github/blob/main/SECURITY.md) — report
issues privately to `security@turf.build`.
