<div align="center">

<img src="https://raw.githubusercontent.com/turfbuild/.github/main/profile/turf-logo.png" alt="Turf" width="220" />

**The infrastructure engine built for AI agents.**

Turf™ is a self-hosted, autonomous infrastructure-as-code tool, built on OpenTofu™.
Terraform-compatible — full HCL and module-registry support — exposed to AI agents as an
infrastructure MCP server.

[turf.build](https://turf.build) · [Get started](#get-started) · [Examples](https://github.com/turfbuild/turf-examples)

</div>

---

## Get started

```bash
brew install restatedev/tap/restate-server turfbuild/tap/turf
brew services start turf                                        # the engine, now and at every login
npx add-mcp@2 "$(command -v turf-mcp-server)" --name turf -g    # connect your agent
```

Then, in Claude Code: `/turf:guide`.

This installs four binaries on your `PATH`: **`turf-mcp-server`**, the MCP server your agent
talks to; the engine behind it, `turf-engine` and `turf-builtin-provider`, which
`brew services start turf` runs as a service (`turf-mcp-server engine status` says where); and
the **Turf CLI**, `turf`. Without Homebrew, `curl -fsSL https://get.turf.build/install.sh | bash`
installs the same into `~/.turf/bin`.

> In **Claude Code** and **Claude Desktop**, Turf runs on your Claude subscription — no separate
> model API key.

<a id="use-it-as-an-mcp-server"></a>
## Connect your agent

The `npx add-mcp@2 … --name turf -g` line registers the server with every agent on this machine.
It detects the ones you have installed — Claude Code, Claude Desktop, Cursor, Codex, Gemini CLI,
VS Code, Windsurf, Zed and more — lets you pick from the list, and writes each config with the
server's absolute path, so an app launched from the Dock finds it too. It needs Node 18+ and uses
[add-mcp](https://github.com/neon-solutions/add-mcp), Neon's open-source installer; running it
again replaces the entry. Restart Claude Desktop or Cursor after the config changes.

Without Node, any client with an `mcp add` command registers the server in one line:

```bash
claude mcp add turf --scope user -- turf-mcp-server             # Claude Code
codex  mcp add turf -- turf-mcp-server                          # Codex CLI
gemini mcp add -s user turf turf-mcp-server                     # Gemini CLI
code --add-mcp '{"name":"turf","command":"turf-mcp-server"}'    # VS Code
```

For a client without one, add the standard block to its MCP config, with the path
`command -v turf-mcp-server` prints:

```json
{ "mcpServers": { "turf": { "command": "/path/to/turf-mcp-server" } } }
```

<details>
<summary><b>All MCP clients &amp; advanced setup</b> — scopes, env vars, editor variants, HTTP</summary>

<br />

Use the absolute path (`command -v turf-mcp-server`) whenever the client is a GUI app or
`turf-mcp-server` isn't on its `PATH`.

#### Claude Code — `claude mcp add`

Three scopes: `local` (this project, private — the default), `project` (shared via a committed
`.mcp.json`; Claude Code asks you to approve it the first time), and `user` (every project). The
`--` separates Claude Code's flags from the command it runs.

```bash
claude mcp add --scope user turf -- turf-mcp-server        # every project (~/.claude.json)
claude mcp add --scope project turf -- turf-mcp-server     # this repo, committed to .mcp.json
```

Pass environment variables (e.g. cloud credentials) with `--env KEY=value`, repeatable, before
the `--`:

```bash
claude mcp add --scope project --env AWS_PROFILE=turf-dev turf -- turf-mcp-server
```

Verify with `claude mcp get turf` / `claude mcp list`. Turf's prompts are slash commands:
`/turf:guide`, `/turf:up path/to/config`, `/turf:destroy path/to/config`
(arguments are one word each).

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
their config file, or let `npx add-mcp@2 … -g` write it. Two clients need small tweaks:

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

## The standalone agent: the Turf CLI

```bash
turf chat                          # interactive, in a terminal
cd path/to/config && turf up       # deploy or reconcile a configuration
```

The CLI drives the same server and engine with its own agent, so it brings its own model: set
`ANTHROPIC_API_KEY` (or `GOOGLE_API_KEY`, `OPENAI_API_KEY`), run a local model, or point it at a
private endpoint. Its README: [turfbuild/turf](https://github.com/turfbuild/turf).

## Featured

| Repo | Description |
| --- | --- |
| [**turf**](https://github.com/turfbuild/turf) | The Turf CLI and `turf-mcp-server` — 1.0 releases. |
| [**turf-examples**](https://github.com/turfbuild/turf-examples) | Reference integrations, agent definitions, and Terraform/HCL examples. |

## Open source

Turf is built on OpenTofu and a small set of MPL-2.0 forks we maintain in the open:

- [opentofu](https://github.com/turfbuild/opentofu) — our OpenTofu fork
- [provider-client](https://github.com/turfbuild/provider-client) — OpenTofu provider-client fork
- [terraform-provider-tfcoremock](https://github.com/turfbuild/terraform-provider-tfcoremock) — mock provider used in tests
- [docker-agent](https://github.com/turfbuild/docker-agent) — downstream cagent fork powering the CLI

## License and telemetry

Turf is free for noncommercial use under the PolyForm Noncommercial License 1.0.0. Commercial
use, including a company's internal use, needs a license: https://turf.build/licensing. The
`turf` CLI is open source (MPL-2.0).

Turf sends anonymous usage telemetry to Turf LLC by default: the version, the platform, counts
of resources, changes and runs, and an anonymous installation id. It never sends resource
contents, credentials, state, variable values or names. Turn it off with `turf telemetry off`,
or with `TURF_TELEMETRY=off` or `DO_NOT_TRACK=1` in the engine's environment; the full field
list and the endpoint are at https://turf.build/telemetry.

## Security

Turf has a transparent supply-chain posture: every push runs continuous scanning (govulncheck,
osv-scanner, gosec, trivy), and releases ship SBOMs (SPDX + CycloneDX), OpenVEX disposition
documents, and scan reports with GitHub-native build &amp; SBOM attestations. See our
[security policy](https://github.com/turfbuild/.github/blob/main/SECURITY.md) — report issues
privately to `security@turf.build`.
