<div align="center">

<img src="https://raw.githubusercontent.com/turfbuild/.github/main/profile/turf-logo.png" alt="Turf" width="220" />

**The infrastructure engine built for AI agents.**

Turf™ is a self-hosted, autonomous infrastructure-as-code tool, built on OpenTofu™.
Terraform-compatible — full HCL and module-registry support — exposed to AI agents as an
infrastructure MCP server.

[turf.build](https://turf.build) · [Get started](#get-started) · [Examples](https://github.com/turfbuild/turf-examples) · [Community](https://turf.build/community/)

</div>

---

## Get started

Read the [Turf license terms](https://turf.build/license/v1/) first, then:

```bash
npx -y @turfbuild/turf-mcp-server license accept   # downloads Turf, about a minute, and accepts the terms
npx add-mcp@2 @turfbuild/turf-mcp-server --name turf -g
```

Then, in Claude Code: `/turf:guide`.

The first line downloads Turf in the terminal, which takes about a minute, and records your
acceptance once, in `~/.turf/license.json`. The second registers **`turf-mcp-server`**, the MCP
server your agent talks to, with the agents on this machine. It needs Node 18 or later, on macOS or
Linux, arm64 or x64. The agent's first start is then quick. The server starts the engine behind it
when a tool first needs it, and the engine keeps running when your agent quits.

> In **Claude Code** and **Claude Desktop**, Turf runs on your Claude subscription — no separate
> model API key.

**Homebrew** adds the **Turf CLI**, `turf`, and an engine that starts at login:

```bash
brew install restatedev/tap/restate-server turfbuild/tap/turf
turf license accept
npx add-mcp@2 "$(command -v turf-mcp-server)" --name turf -g
brew services start turf        # optional: the engine at every login
```

This installs five binaries on your `PATH`: `turf`; `turf-mcp-server`; the engine behind it,
`turf-engine` and `turf-builtin-provider`; and the Turf console, `turf-ui`.
`turf-mcp-server engine status` says where the engine runs. Without Homebrew,
`curl -fsSL https://get.turf.build/install.sh | bash` installs the same into `~/.turf/bin`.

<a id="use-it-as-an-mcp-server"></a>
## Connect your agent

The `npx add-mcp@2 … --name turf -g` line registers the server with every agent on this machine.
It detects the ones you have installed (Claude Code, Claude Desktop, Cursor, Codex, Gemini CLI,
VS Code, Windsurf, Zed and more), lets you pick from the list, and writes each config. It uses
[add-mcp](https://github.com/neon-solutions/add-mcp), Neon's open-source installer; running it
again replaces the entry. Restart Claude Desktop or Cursor after the config changes. If the
agent's first start fails, turn Turf off in the agent, run the `license accept` line in a terminal
and let it finish, then reconnect. If the server says a package "is installed without its binary",
an earlier install was interrupted: the message names the npx folder to remove and the steps.

Any client with an `mcp add` command registers the server in one line, in place of the `add-mcp`
line (run the `license accept` line first, so the download happens in the terminal):

```bash
# Claude Code
claude mcp add turf --scope user -- npx -y @turfbuild/turf-mcp-server
# Codex CLI
codex mcp add turf -- npx -y @turfbuild/turf-mcp-server
# Gemini CLI
gemini mcp add -s user turf npx -y @turfbuild/turf-mcp-server
# VS Code
code --add-mcp '{"name":"turf","command":"npx","args":["-y","@turfbuild/turf-mcp-server"]}'
```

For a client without one, add the standard block to its MCP config:

```json
{ "mcpServers": { "turf": { "command": "npx", "args": ["-y", "@turfbuild/turf-mcp-server"] } } }
```

With a Homebrew install, the installed binary takes the place of the `npx` command in any of
these: `turf-mcp-server`, or the absolute path `command -v turf-mcp-server` prints when the client
is a GUI app.

<details>
<summary><b>All MCP clients &amp; advanced setup</b> — scopes, env vars, editor variants, HTTP</summary>

<br />

#### Claude Code — `claude mcp add`

Three scopes: `local` (this project, private — the default), `project` (shared via a committed
`.mcp.json`; Claude Code asks you to approve it the first time), and `user` (every project). The
`--` separates Claude Code's flags from the command it runs.

```bash
# every project (~/.claude.json)
claude mcp add turf --scope user -- npx -y @turfbuild/turf-mcp-server
# this repo, committed to .mcp.json
claude mcp add turf --scope project -- npx -y @turfbuild/turf-mcp-server
```

Pass environment variables with `--env KEY=value`, repeatable, before the `--`:

```bash
claude mcp add --scope project --env AWS_PROFILE=turf-dev turf -- npx -y @turfbuild/turf-mcp-server
```

A variable set this way reaches the server and the state backend (an S3 backend reads that
`AWS_PROFILE`). Providers run in the engine, and get a variable when `TURF_PROVIDER_ENV` names it:

```bash
claude mcp add --scope project --env AWS_PROFILE=turf-dev --env TURF_PROVIDER_ENV=AWS_PROFILE turf -- npx -y @turfbuild/turf-mcp-server
```

The entry's `env` then holds both lines, as it does in any client's config:

```json
"env": { "AWS_PROFILE": "turf-dev", "TURF_PROVIDER_ENV": "AWS_PROFILE" }
```

The server sends the variables you name, with their values, when a run starts; `PATH` and the
engine's own variables are never sent. Restart the agent after a change. Credentials in their
usual files (the default AWS profile, `~/.kube/config`, gcloud's application-default credentials,
the `az` login) need nothing set, and a configuration's own settings go in its `mise.toml` or
`mise.local.toml`.

Verify with `claude mcp get turf` / `claude mcp list`. Turf's prompts are slash commands:
`/turf:guide`, `/turf:up path/to/config`, `/turf:destroy path/to/config`
(arguments are one word each).

#### Codex CLI — `codex mcp add`

Codex stores config as TOML. `codex mcp add` writes to the global `~/.codex/config.toml`:

```bash
codex mcp add turf -- npx -y @turfbuild/turf-mcp-server
codex mcp add turf --env AWS_PROFILE=turf-dev -- npx -y @turfbuild/turf-mcp-server
```

```toml
[mcp_servers.turf]
command = "npx"
args = ["-y", "@turfbuild/turf-mcp-server"]
```

For a project-local server, add that `[mcp_servers.turf]` block to `.codex/config.toml` yourself —
Codex only loads project config for **trusted** projects. Verify with `codex mcp list`, then `/mcp`
inside a session.

#### Gemini CLI — `gemini mcp add`

Default scope is **project**; pass `-s user` for the global install. Config is JSON.

```bash
# every project (~/.gemini/settings.json)
gemini mcp add -s user turf npx -y @turfbuild/turf-mcp-server
# this repo (.gemini/settings.json)
gemini mcp add -s project turf npx -y @turfbuild/turf-mcp-server
gemini mcp add -s project -e AWS_PROFILE=turf-dev turf npx -y @turfbuild/turf-mcp-server
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
{ "servers": { "turf": { "type": "stdio", "command": "npx", "args": ["-y", "@turfbuild/turf-mcp-server"] } } }
```

```json
// Zed — root key is "context_servers", each entry needs "source": "custom"
{ "context_servers": { "turf": { "source": "custom", "command": "npx", "args": ["-y", "@turfbuild/turf-mcp-server"] } } }
```

| Client         | Config file |
| -------------- | ----------- |
| Claude Desktop | `claude_desktop_config.json` |
| Cursor         | `.cursor/mcp.json` (project) or `~/.cursor/mcp.json` (user) |
| Windsurf       | `mcp_config.json` |
| Zed            | `settings.json`, under `context_servers` |
| VS Code        | `.vscode/mcp.json` (project) or user `mcp.json`, under `servers` |

#### The engine, and upgrades

The engine keeps running when the client quits, so a rollout waiting for your approval is still
there next time. Two commands manage it:

```bash
npx -y @turfbuild/turf-mcp-server engine status
npx -y @turfbuild/turf-mcp-server engine stop
```

An unpinned entry asks npm at every start, so a new release is installed the next time the client
starts the server. The engine that is running stays the old one until it is stopped: `engine stop`
ends it, and the next call starts the new one. Across a minor release (1.5 to 1.6) the server
refuses the older engine and names that remedy; after a patch release it keeps using the running
engine and says nothing. To hold a version, pin it in the entry:
`@turfbuild/turf-mcp-server@1.5.0`.

#### Remote / HTTP

To connect to a hosted `turf-mcp-server` (`--transport http`) instead of a local stdio server,
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

The CLI comes with the Homebrew or the script install. It drives the same server and engine with its own agent,
so it brings its own model: set `ANTHROPIC_API_KEY` (or `GOOGLE_API_KEY`, `OPENAI_API_KEY`), run a
local model, or point it at a private endpoint. Its README:
[turfbuild/turf](https://github.com/turfbuild/turf).

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

Turf is proprietary software under the Turf Community Edition License 1.0
(https://turf.build/license/v1/). It is free for individuals, education and small businesses
within the Community limits. Government and larger organizations need a commercial license, and
anyone may evaluate Turf, not in production, for 30 days: https://turf.build/licensing/. The
`turf` CLI is open source (MPL-2.0).

Turf sends anonymous usage telemetry to Turf LLC by default: the version, the platform, counts
of resources, changes and runs, and an anonymous installation id. It never sends resource
contents, credentials, state, variable values or names. To turn it off, put `DO_NOT_TRACK=1` in
the `env` of the server's entry in each agent that has one, reconnect the agent so its server
restarts with it, then stop the engine:
`npx -y @turfbuild/turf-mcp-server engine stop`.
The next call starts the engine with telemetry off. With the CLI, `turf telemetry off` and the same
stop do it. The Homebrew service reads neither an entry nor your shell: its environment is
`~/.homebrew/services/turf.env`. The full field list and the endpoint are at
https://turf.build/telemetry.

## Community

- Questions, ideas, show and tell: [Discord](https://turf.build/discord/).
- Bugs and feature requests: [turfbuild/turf issues](https://github.com/turfbuild/turf/issues).
- Release notes by email: [the newsletter](https://turf.build/newsletter/). Follow Turf on
  [LinkedIn](https://www.linkedin.com/company/turfbuild/) and [X](https://x.com/turfbuild).
- Evaluating Turf at a company? [The pilot program](https://turf.build/licensing/#pilot).
- Anything else: contact@turf.build. All of it at https://turf.build/community/.

## Security

Turf has a transparent supply-chain posture: every push runs continuous scanning (govulncheck,
osv-scanner, gosec, trivy), and releases ship SBOMs (SPDX + CycloneDX), OpenVEX disposition
documents, and scan reports with GitHub-native build &amp; SBOM attestations. See our
[security policy](https://github.com/turfbuild/.github/blob/main/SECURITY.md) — report issues
privately to `security@turf.build`.
