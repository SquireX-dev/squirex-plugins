---
name: squirex-fix-violation
description: Fix a specific SquireX Agentforce violation (e.g. AGENTFORCE-1.1 in a genAiFunction-meta.xml) by fetching the rule's remediation, reading the file, proposing a minimal diff, and re-validating. Use when the user asks to fix or remediate a SquireX finding.
---

# SquireX: fix a violation

Needs: a `ruleId` and a `filePath` (relative to the project root).

## Steps
1. Call `get_rule_details` with `ruleId` to get the remediation text.
2. Call `suggest_fix` with `ruleId`, `filePath`, and optionally `violationContext` (the action/template name or the offending snippet).
   - `suggest_fix` returns a rule-level template diff; it does not read your file. **Don't apply it blindly.** Read the real file and adapt it.
3. Write the smallest correct change. Typical ones:
   - `AGENTFORCE-1.1`: set `<isConfirmationRequired>true</isConfirmationRequired>` on state-changing GenAiFunctions.
   - `AGENTFORCE-1.3`: change the invoked Apex class to `with sharing` (or `inherited sharing` with justification), or run the Flow in user context. Also check the class for dynamic SOQL; use bind variables or `WITH USER_MODE`.
   - `AGENTFORCE-9.1` / `AGENTFORCE-2.3` / `AGENTFORCE-PT-01`: remove instructions that let user input override system instructions, and keep user input structurally separate from instructions.
   - `AGENTFORCE-3.1`: remove secrets from templates. Use Named Credentials.
4. Show the diff and get the user's OK before editing files (or follow their approval mode). Never deploy to an org.
5. Re-validate with `scan_agentforce_file` (`filePath`), or `scan_agentforce`. If the scan engine is unavailable (see the `squirex-scan-agents` skill for known errors), say the fix is **unverified** and list the rule ID so the user can re-scan later.
