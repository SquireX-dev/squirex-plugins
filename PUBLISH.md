# Publish steps (needs Prasanth)

Nothing below has been executed by the agent. The SquireX-dev GitHub org exists; the public `squirex-plugins` repo does **not** yet.

## 1. Create the public repo

```bash
# From a clean checkout of this folder's contents (or a sparse export):
gh repo create SquireX-dev/squirex-plugins \
  --public \
  --license mit \
  --description "SquireX agent-security plugins for Cursor, Grok Bot, Claude, and Codex" \
  --source . \
  --remote origin \
  --push
```

Recommended: keep this ApexForge tree as the source of truth and either:

- **Subtree / sync script** that copies `plugins/marketplace/` → the public repo on each release, or
- Make `SquireX-dev/squirex-plugins` the canonical public copy and update it when cutting a `v*` tag.

Engine source stays in private `samudralap/ApexForge`.

## 2. Pin a published npm version first

Marketplace reviewers will run `npx @squirex.dev/mcp-server@x.y.z`. That requires:

1. `BLOB_BASE_URL` set in Vercel (deferred — Phase 1 SB-06)
2. A `v*` GitHub Release that publishes npm + engine binaries
3. Rebuild this bundle with `node plugins/build.mjs --version x.y.z` so manifests/mcp.json pin that version **without** `npx -y`

Until then, local testing can use a path install or a pre-release tag.

## 3. Marketplace submissions (human clicks)

| Target | Action | Account |
|---|---|---|
| Cursor Marketplace (→ Grok Bot) | https://cursor.com/marketplace/publish | Cursor publisher + identity verify |
| Claude plugin directory | https://claude.ai/directory/manage → Plugin bundle → this GitHub repo | Claude Pro/Max/Team Owner + GitHub connected |
| Codex / ChatGPT local marketplace | `codex plugin marketplace add SquireX-dev/squirex-plugins` (docs) | no public review |
| xAI Grok Build | PR to `xai-org/plugin-marketplace` with SHA-pinned entry | org-official source preferred |
| OpenAI universal directory (Dots) | **Blocked** on hosted HTTPS MCP + OAuth at `mcp.squirex.dev` | OpenAI org verification |

## 4. Legal before hosted / OpenAI submit

- Approve privacy + ToS updates (hosted processing language is drafted on the site as planned/future).
- Confirm support contact **hello@squirex.dev**.
- Legal read on Cursor publisher terms §3.1 (“no fees directly or indirectly”) vs Pro license gating.
