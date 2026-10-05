---
name: squirex-agentforce-review
description: Pre-deployment security review of Salesforce Agentforce agents with SquireX, combining the SquireX scan with a rule-guided manual review of topics, actions, prompt templates and invoked Apex/Flows. Use before deploying or when hardening an Agentforce agent.
---

# SquireX: Agentforce pre-deployment review

This does what the server's `review-agentforce-security` and `harden-agent-metadata` MCP prompts do, and keeps working when the scan engine is unavailable.

## Steps
1. Call `list_scan_rules` to load the 26 `AGENTFORCE-*` rules.
2. Try the automated scan (the `squirex-scan-agents` skill: `scan_agentforce`, full project unless the user asked for a PR-scoped review). If it works, triage it with `squirex-triage-findings`.
3. Whether or not the scan worked, do a manual pass against the rules. Read the metadata and note file:line evidence:
   - **GenAiFunction** (`*.genAiFunction-meta.xml`): check `isConfirmationRequired` on apex/flow/externalService actions (1.1). Check that `invocationTarget` resolves (4.1).
   - **Invoked Apex**: `with sharing`? (1.3). Dynamic SOQL or DML built from inputs? Callouts to endpoints that come from input? (API-01).
   - **Flows**: `SystemModeWithoutSharing` (1.3, FLOW-01), silent record updates (FLOW-02), unvalidated inputs (FLOW-03).
   - **GenAiPlugin topics / .agent scripts**: instructions that defer to user input or cross topic boundaries (9.1, 9.2, 2.3). Topic bloat (7.1). Missing fallback transitions (2.2).
   - **Prompt templates**: hard-coded secrets or PII (3.1), merge fields that inject user data into instructions (PT-01), activation state (PT-02).
   - **Supply chain**: `sourceApiVersion`/`apiVersion` downgrades (SC-01), managed-package origin (SC-03).
   For each rule you cite, call `get_rule_details` (`ruleId`) to quote its remediation.
4. Label every item **Automated (SquireX SARIF)** or **Manual (rule-guided)**. Never present manual observations as scanner output.
5. Finish with a go/no-go recommendation, a ranked fix list, and an offer to run `squirex-fix-violation` on the top item.

Optional, if the user has an org and the sf CLI: `generate_dx_tests`, then `validate_dx_tests`, then `push_to_testing_center` (`testFilePath`, `targetOrg`), then `get_testing_center_results`. **Pushing touches a Salesforce org, so ask first.**
