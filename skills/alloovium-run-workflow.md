---
name: Run a workflow (Routine) and poll for the result
description: List Alloovium workflows, start a run, and poll its status to completion — the async automation pattern.
api: https://www.alloovium.com/en/developers
transport: MCP (https://api.alloovium.com/api/v1/mcp/) or REST v2
operations: [meta_whoami, workflows_list, workflows_run, workflows_get_run_status]
scopes: [workflows:read, workflows:write]
generated: '2026-07-17'
method: generated
source: https://www.alloovium.com/en/developers
---

# Run a workflow (Routine) and poll for the result

Alloovium Routines are review-gated automation chains (trigger → action → logic blocks) that draft variations, RFIs, reports, and compliance packs. Runs are asynchronous.

## Auth
`Authorization: Bearer ak_live_...`. Starting a run requires `workflows:write`.

## Steps
1. **`meta_whoami`** (cost 1) — validate the key.
2. **`workflows_list`** (scope `workflows:read`, cost 1) — list available workflows; page with `cursor` / `has_more`. Pick the target workflow.
3. **`workflows_run`** (scope `workflows:write`, cost 25) — start the run. Send an **`Idempotency-Key`** so a retry does not double-trigger. Returns a `run_id` for polling.
4. **`workflows_get_run_status`** (scope `workflows:read`, cost 1) — poll with the `run_id` until the run reaches a terminal status. Back off between polls to respect rate limits.

## Rules
- `workflows_run` is the most expensive operation (25 tokens) — check `X-RateLimit-Remaining` before firing and always attach an `Idempotency-Key`.
- Results land behind a human review gate; treat drafts as pending approval, not final documents.
- Errors are RFC 9457 problem+json; branch on the stable `code`. `403 insufficient_scope` means the credential lacks `workflows:write`.
