---
name: squirex-triage-findings
description: Triage and prioritize SquireX SARIF findings for Agentforce agents, grouping by severity and rule, explaining root cause and blast radius. Use after a SquireX scan, or when the user shares a SquireX SARIF file or rule IDs like AGENTFORCE-1.1.
---

# SquireX: triage findings

Input: SARIF from `scan_agentforce` or `generate_sarif_report`, a SARIF file the user gives you, or a list of rule IDs.

## Steps
1. Parse SARIF `runs[].results[]`: `ruleId`, `level`, `message.text`, `locations[].physicalLocation.artifactLocation.uri` and `region.startLine`.
2. For each distinct `ruleId`, call `get_rule_details` (`ruleId`) for severity, category and remediation. `list_scan_rules` with `severity` gives the whole catalog in one call.
3. Order by severity: critical → high → medium → low. Within a level, put the rules that let an agent change data or escalate privilege first:
   - `AGENTFORCE-1.1` (no user confirmation)
   - `AGENTFORCE-1.3` (without sharing / system mode)
   - `AGENTFORCE-2.3`, `AGENTFORCE-9.1`, `AGENTFORCE-PT-01` (prompt injection / instruction poisoning)
   - `AGENTFORCE-3.1` (secrets)
   - `AGENTFORCE-API-01`
4. For each critical/high finding, call `explain_violation` with `ruleId`, `filePath`, `message`, `lineNumber`.
   - Note: this returns rule-level guidance and doesn't read the file. Open the file yourself to confirm the finding is real and quote the offending lines.
5. Output a short table: severity | rule | file:line | what an attacker or LLM could do | fix effort. Then list the top 3 actions.
6. Point out related findings that combine. Example: an action without confirmation (1.1) that calls a `without sharing` Apex class (1.3) from a topic with poisoned instructions (9.1) is one attack path, and fixing the confirmation gate breaks it.

Don't dump raw SARIF. Don't report findings you haven't seen in real output.
