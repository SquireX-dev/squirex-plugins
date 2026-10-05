# SquireX plugins (Cursor / Grok Bot / Claude / Codex)

This folder is the **MIT-licensed plugin wrapper** for SquireX. It packages:

- a local **stdio** MCP server launched via a pinned `npx @squirex.dev/mcp-server@<version>`
- five agent skills for scan → triage → fix workflows
- manifests for **Cursor** (feeds Grok Bot), **Claude** plugin directory, and **Agent Plugins / Codex**

The scan engine itself (`squirex` CLI + Go interpreter) and `@squirex.dev/mcp-server` remain **proprietary**. This matches how Semgrep/Sonar ship marketplace plugins that invoke a separate engine.

> **Status:** packaging only. Marketplace submit clicks, npm publish, and the hosted HTTPS MCP (`mcp.squirex.dev`) are **not** done in this PR. See `PUBLISH.md`.

## What it does

SquireX analyzes AI-agent metadata and related code for capability, privilege, prompt-injection, and supply-chain issues across:

| Surface | What is scanned |
|---|---|
| Salesforce Agentforce | GenAiFunction / Plugin / Planner / Prompt Template metadata, Agent Script (`.agent`) |
| ServiceNow Now Assist | `sn_aia_*` Update Set exports, agent-tool bindings, Glide patterns reachable from agents |
| MuleSoft Agent Fabric | Agent Network configs and related secrets hygiene |
| MCP | Client configs (`.mcp.json`, Claude Desktop, `.vscode/mcp.json`) and MCP server source patterns |

Rule catalog size is generated from the engine (`squireinterp graph-export --rules`) and currently **126 rules**. Listing copy and `list_scan_rules` must stay in sync with that number.

## Privacy and network

**Default (this plugin): local stdio.** The MCP server runs on the machine that hosts the agent (your laptop, CI runner, or Grok Bot box). Project files are read locally to scan. They are **not** uploaded to squirex.dev by this plugin.

Outbound network used by the local stack:

| Destination | When | Purpose |
|---|---|---|
| `registry.npmjs.org` | first launch / version pin | download `@squirex.dev/mcp-server` and `squirex` |
| Engine asset host (`BLOB_BASE_URL` or GitHub Releases) | first scan | download + SHA-256 verify `squireinterp` |
| `https://squirex.dev/api/validate` | private-repo / CI license checks | validate `SQUIREX_LICENSE_KEY` (no source code sent) |

Optional credentials (declared as plugin variables / `userConfig`):

- `SQUIREX_PROJECT_DIR` — absolute path of the workspace to scan
- `SQUIREX_LICENSE_KEY` — Pro key for private-repo CI (sensitive). Not required for local scans of public repos.

A **hosted** HTTPS MCP with OAuth (required for OpenAI Dots / ChatGPT public directory and the Claude Connectors directory) is planned. When it ships, privacy policy and this README will describe what code is processed server-side. Until then, do not claim hosted scanning in marketplace listings.

## Support

- Email: **hello@squirex.dev**
- Support page: https://squirex.dev/support
- Privacy: https://squirex.dev/privacy-policy
- Terms: https://squirex.dev/terms-of-service

## Install (local, before marketplace listing)

```bash
# Cursor / Grok Bot — symlink for local plugin testing
ln -s "$(pwd)" ~/.cursor/plugins/local/squirex

# Claude Code — add as a marketplace or local plugin once the public repo exists
# /plugin marketplace add SquireX-dev/squirex-plugins

# Codex — local / git marketplace
# codex plugin marketplace add SquireX-dev/squirex-plugins
```

Smoke test: ask the agent to “list the SquireX scan rules” (expect `list_scan_rules`) and “scan this project with SquireX” (expect `scan_agentforce` + SARIF).

## Layout

```
plugins/marketplace/
├── plugin.json                 # Agent Plugins (+ extensions.com.openai metadata)
├── .cursor-plugin/plugin.json  # Cursor → Grok Bot
├── .claude-plugin/plugin.json  # Claude plugin directory
├── .codex-plugin/plugin.json   # Codex fallback
├── mcp.json / .mcp.json        # pinned stdio MCP (no npx -y)
├── skills/                     # five SquireX skills
├── assets/                     # logo.svg + PNG placeholders
├── LICENSE                     # MIT (wrapper only)
├── README.md
└── PUBLISH.md                  # how to create the public SquireX-dev repo
```

Version placeholders use `0.0.0-dev`. `plugins/build.mjs` stamps the release version and pins `@squirex.dev/mcp-server@<version>`.
