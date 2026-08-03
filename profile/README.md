<div align="center">

<img src="https://raw.githubusercontent.com/turfbuild/.github/main/profile/turf-logo.png" alt="Turf" width="220" />

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

This installs two binaries on your `PATH`: the **Turf CLI** (`turf`) and the engine,
**`turf-mcp-server`** — an MCP server that exposes infrastructure tools (plan/apply, provider
control, state) to any MCP client.

### Two ways to use Turf

- **The Turf CLI** — a batteries-included agent. `turf up`, `turf chat`, and friends drive the
  engine for you. It brings its own model and needs a model API key
  (`ANTHROPIC_API_KEY` / `GOOGLE_API_KEY`).
- **As an MCP server** — point an MCP client you already use at `turf-mcp-server` and drive
  infrastructure from there. No CLI required; the engine is fully usable on its own.

### Use it as an MCP server

`turf-mcp-server` speaks standard MCP over stdio, so any client with an `mcp add` command
registers it in one line:

```bash
claude mcp add turf -- turf-mcp-server      # Claude Code
codex  mcp add turf -- turf-mcp-server      # Codex CLI
gemini mcp add turf turf-mcp-server         # Gemini CLI
```

> In **Claude Code** and **Claude Desktop**, Turf runs on your Claude subscription — no separate
> model API key. (The Turf CLI, by contrast, brings its own model and needs an API key.)

For a client without an installer, add the standard block to its MCP config:

```json
{ "mcpServers": { "turf": { "command": "turf-mcp-server" } } }
```

<details>
<summary><b>All MCP clients &amp; advanced setup</b> — scopes, env vars, editor variants, HTTP</summary>

<br />

Replace `turf-mcp-server` with a full path if it isn't on your `PATH`, or run the container
instead: `docker run --rm -i ghcr.io/turfbuild/turf-mcp-server:latest`.

#### Claude Code — `claude mcp add`

Three scopes: `local` (this project, private — the default), `project` (shared via a committed
`.mcp.json`), and `user` (every project). The `--` separates Claude Code's flags from the command
it runs.

```bash
claude mcp add --scope user turf -- turf-mcp-server        # every project (~/.claude.json)
claude mcp add --scope project turf -- turf-mcp-server      # this repo, committed to .mcp.json
```

Pass environment variables (e.g. cloud credentials) with `--env KEY=value`, repeatable, before
the `--`:

```bash
claude mcp add --scope project --env AWS_PROFILE=turf-dev -- turf-mcp-server
```

Verify with `claude mcp get turf` / `claude mcp list`.

#### Codex CLI — `codex mcp add`

Codex stores config as TOML. `codex mcp add` writes to the global `~/.codex/config.toml`:

```bash
codex mcp add turf -- turf-mcp-server
codex mcp add turf --env AWS_PROFILE=turf-dev -- turf-mcp-server
```

```toml
[mcp_servers.turf]
command = "turf-mcp-server"
```

For a project-local server, add that `[mcp_servers.turf]` block to `.codex/config.toml` yourself —
Codex only loads project config for **trusted** projects. Verify with `codex mcp list`, then `/mcp`
inside a session.

#### Gemini CLI — `gemini mcp add`

Default scope is **project**; pass `-s user` for the global install. Config is JSON.

```bash
gemini mcp add -s user turf turf-mcp-server                 # every project (~/.gemini/settings.json)
gemini mcp add -s project turf turf-mcp-server              # this repo (.gemini/settings.json)
gemini mcp add -s project -e AWS_PROFILE=turf-dev turf turf-mcp-server
```

Verify with `gemini mcp list` or `/mcp` inside a session.

#### Quick reference

| Client      | User-level file           | Project-level file        | Scope flag |
| ----------- | ------------------------- | ------------------------- | ---------- |
| Claude Code | `~/.claude.json`          | `.mcp.json`               | `--scope user` / `--scope project` |
| Codex CLI   | `~/.codex/config.toml`    | `.codex/config.toml`\*    | *(user only — project needs a manual edit)* |
| Gemini CLI  | `~/.gemini/settings.json` | `.gemini/settings.json`   | `-s user` / `-s project` |

\* Project-scoped Codex config loads only for **trusted** projects.

#### Clients without an `mcp add` command

Claude Desktop, Cursor, and Windsurf read the same `mcpServers` block shown above — paste it into
their config file. Two clients need small tweaks:

```json
// VS Code — root key is "servers", each entry needs a "type"
{ "servers": { "turf": { "type": "stdio", "command": "turf-mcp-server" } } }
```

```json
// Zed — root key is "context_servers", each entry needs "source": "custom"
{ "context_servers": { "turf": { "source": "custom", "command": "turf-mcp-server" } } }
```

| Client         | Config file |
| -------------- | ----------- |
| Claude Desktop | `claude_desktop_config.json` |
| Cursor         | `.cursor/mcp.json` (project) or `~/.cursor/mcp.json` (user) |
| Windsurf       | `mcp_config.json` |
| Zed            | `settings.json`, under `context_servers` |
| VS Code        | `.vscode/mcp.json` (project) or user `mcp.json`, under `servers` |

#### Remote / HTTP

To connect to a hosted `turf-mcp-server` (`--transport http`) instead of a local stdio binary,
use a `url` (and `headers` if it requires auth):

```json
{
  "mcpServers": {
    "turf": {
      "url": "https://turf.example.com/mcp",
      "headers": { "Authorization": "Bearer ${TURF_TOKEN}" }
    }
  }
}
```

</details>

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
