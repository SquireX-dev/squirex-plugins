---
name: squirex-scan-agents
description: Run a SquireX security scan on a Salesforce Agentforce project (GenAiFunction, GenAiPlugin, GenAiPlannerBundle, GenAiPromptTemplate, .agent, Apex) through the SquireX MCP server. Use when the user asks to scan, audit, or security-check their Agentforce agents or metadata.
---

# SquireX: scan Agentforce agents

Use the `squirex` MCP server (`@squirex.dev/mcp-server`). Only these tools exist: `scan_agentforce`, `scan_agentforce_file`, `scan_agentforce_rule`, `list_scan_rules`, `get_rule_details`, `explain_violation`, `suggest_fix`, `generate_sarif_report`. Don't invent other tool names.

## Steps
1. **Confirm the project.** The server scans `SQUIREX_PROJECT_DIR` (or the `directory` argument). It should be an SFDX project containing `force-app/` or similar. If you don't know the path, ask the user.
2. **Run the scan.** Call `scan_agentforce` with:
   - `directory`: absolute project path, if it differs from `SQUIREX_PROJECT_DIR`
   - `baseBranch` (optional): omit it for a **full project scan**. Pass a branch (e.g. `main`) only when the user wants a PR-style scan of files changed since that branch.
   - For one file: `scan_agentforce_file` with `filePath`. For one rule: `scan_agentforce_rule` with `ruleId` (e.g. `AGENTFORCE-1.1`).
   - Scan tools return the SARIF log as the first text item and a one-line summary (scope + finding count) as the second. `scan_agentforce_file` scans the whole project and keeps only results in that file.
3. **Check for errors.** The server sets `isError: true` when the scan could not run, and the text says why. Findings are *not* errors. Tell the user plainly what failed:
   - `Failed to install the SquireX engine` / `Go binary "squireinterp" not found`: the SquireX Go engine couldn't be installed or verified (the download is checksum-verified and fails closed). The user has to fix network access, or set `SQUIREX_GO_BINARY=/path/to/squireinterp` in the MCP server env (see https://squirex.dev/docs/download). **Do not download or run binaries yourself.**
   - `PR-scoped scan` with 0 findings when you passed `baseBranch`: maybe nothing changed. Say so; this is not a clean bill of health — offer a full scan.
   - `CI/CD Execution Blocked` / `Invalid License Key`: a Pro/Enterprise `SQUIREX_LICENSE_KEY` is needed (CI only).
4. **If it succeeds** (SARIF JSON), hand off to the `squirex-triage-findings` skill.
5. **Never claim a scan passed** unless you got real SARIF back with zero results.

## Reference without scanning
`list_scan_rules` (filters: `category`, `severity`) and `get_rule_details` (`ruleId`) work offline. Use them to explain what SquireX checks even when the engine is unavailable.
