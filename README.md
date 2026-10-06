# SquireX plugins (Cursor / Grok Bot / Claude / Codex)

This repository is the **MIT-licensed plugin wrapper** for [SquireX](https://squirex.dev), AI agent security for Salesforce Agentforce, ServiceNow Now Assist, MuleSoft Agent Fabric, MCP servers and coding-agent configs. It packages:

- a local **stdio** MCP server launched with a pinned `npx @squirex.dev/mcp-server@4.0.1` (no `-y`)
- five agent skills for scan → triage → fix workflows
- manifests for **Cursor** (also used by Grok Bot), the **Claude** plugin directory, and **Agent Plugins / Codex**

The scan engine itself (`squirex` CLI + Go interpreter) and `@squirex.dev/mcp-server` are **proprietary**; this wrapper only launches them.

## What it does

SquireX analyzes AI-agent metadata and related code for capability, privilege, prompt-injection, and supply-chain issues across:

| Surface | What is scanned |
|---|---|
| Salesforce Agentforce | GenAiFunction / Plugin / Planner / Prompt Template metadata, Agent Script (`.agent`), agent-reachable Apex, Flows and Visualforce |
| ServiceNow Now Assist | `sn_aia_*` Update Set exports, agent-tool bindings, Glide patterns reachable from agents |
| MuleSoft Agent Fabric | Agent Network configs and secure-properties hygiene |
| MCP | Client configs (`.mcp.json`, Claude Desktop, `.vscode/mcp.json`) and MCP server source patterns |
| Coding-agent configs | Committed Claude Code, Codex, Cursor, VS Code and Gemini settings that skip approvals or auto-trust MCP servers |

The current engine has **140 rules** ([catalog](https://squirex.dev/docs/rules)). This plugin pins `@squirex.dev/mcp-server@4.0.1`, the latest npm release; it ships an earlier engine with fewer rules until the next release, and the pin will be updated then. Ask the agent to call `list_scan_rules` to see exactly what your installed version runs.

## Privacy and network

**This plugin runs locally (stdio).** The MCP server runs on the machine that hosts the agent (your laptop, CI runner, or agent box). Project files are read locally to scan. They are **not** uploaded to squirex.dev by this plugin.

Outbound network used by the local stack:

| Destination | When | Purpose |
|---|---|---|
| `registry.npmjs.org` | first launch / version pin | download `@squirex.dev/mcp-server` and `squirex` |
| `https://squirex.dev/api/download` (redirects to the release asset) | first scan | download + SHA-256 verify `squireinterp` |
| `https://squirex.dev/api/validate` | scans inside CI | validate `SQUIREX_LICENSE_KEY` (no source code sent) |

Optional settings (declared as plugin variables / `userConfig`):

- `SQUIREX_PROJECT_DIR` — absolute path of the workspace to scan (default: the server's working directory)
- `SQUIREX_LICENSE_KEY` — Pro/Enterprise key, needed only when scanning inside CI (sensitive). Local scans don't need a key.

## Support

- Email: **hello@squirex.dev**
- Support page: https://squirex.dev/support
- Docs: https://squirex.dev/docs/mcp/setup
- Privacy: https://squirex.dev/privacy-policy
- Terms: https://squirex.dev/terms-of-service

## Install (local)

```bash
# Cursor / Grok Bot — symlink a checkout for local plugin testing
git clone https://github.com/SquireX-dev/squirex-plugins.git
ln -s "$(pwd)/squirex-plugins" ~/.cursor/plugins/local/squirex

# Claude Code — add this repo as a plugin marketplace
# /plugin marketplace add SquireX-dev/squirex-plugins

# Codex — git marketplace
# codex plugin marketplace add SquireX-dev/squirex-plugins
```

Smoke test: ask the agent to "list the SquireX scan rules" (expect `list_scan_rules`) and "scan this project with SquireX" (expect `scan_agentforce` + SARIF).

## Layout

```
.
├── plugin.json                 # Agent Plugins (+ extensions.com.openai metadata)
├── .cursor-plugin/plugin.json  # Cursor → Grok Bot
├── .claude-plugin/plugin.json  # Claude plugin directory
├── .codex-plugin/plugin.json   # Codex fallback
├── mcp.json / .mcp.json        # pinned stdio MCP (no npx -y)
├── skills/                     # five SquireX skills
├── assets/                     # logo.svg + PNG icons
├── LICENSE                     # MIT (wrapper only)
└── README.md
```

This repository is generated from the SquireX source tree on each release; the version in the manifests matches the pinned `@squirex.dev/mcp-server` version.
