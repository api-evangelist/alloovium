---
name: Ask a cited question across a construction project
description: Validate the key, find the right project, search the vault, then get a grounded, citation-backed answer over Alloovium project documents.
api: https://www.alloovium.com/en/developers
transport: MCP (https://api.alloovium.com/api/v1/mcp/) or REST v2
operations: [meta_whoami, vault_list_projects, vault_search, chat_ask]
scopes: [vault:read, chat:write]
generated: '2026-07-17'
method: generated
source: https://www.alloovium.com/en/developers
---

# Ask a cited question across a construction project

Use this to answer a construction question (contract clauses, RFIs, specs, safety obligations) grounded in a project's own documents, with page-level citations.

## Auth
Send `Authorization: Bearer ak_live_...` (API key). On the MCP transport, OAuth tokens are not accepted — API key only. REST v2 also accepts OAuth 2.1 (authorization code + PKCE).

## Steps
1. **`meta_whoami`** (scope: none, cost 1) — always call first to validate the key and get user/tenant context.
2. **`vault_list_projects`** (scope `vault:read`, cost 1) — cursor-paginated. Branch on `has_more` / `next_cursor` (never on an empty `data` array); pass `cursor` to page. Pick the target `project_id`.
3. **`vault_search`** (scope `vault:read`, cost 5) — hybrid BM25 + vector + rerank over the vault to confirm the relevant documents (optional but recommended for large projects).
4. **`chat_ask`** (scope `chat:write` + `vault:read`, cost 10) — REST `POST /api/v2/chat` with `{ query, project_ids, document_ids?, conversation_id?, k? }`. Returns `answer`, `citations[]` (`document_id`, `chunk_id`, `page_number`, `content`, `score`), `conversation_id`, `message_id`, `usage`.

## Rules
- Always surface the `citations[]` — answers are only trustworthy with their source pages.
- Rate limits are token-bucket; honor `X-RateLimit-*`. On `429 rate_limited`, sleep `Retry-After` seconds and retry.
- Errors are RFC 9457 `application/problem+json`; branch on the stable `code` field, not `title`.
