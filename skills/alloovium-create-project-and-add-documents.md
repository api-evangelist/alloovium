---
name: Create a project and add documents
description: Create a new Alloovium project and load construction documents into its vault, safely and idempotently.
api: https://www.alloovium.com/en/developers
transport: REST v2 (upload is REST-only) with MCP for reads
operations: [meta_whoami, vault_create_project, vault_list_documents, vault_get_document]
scopes: [vault:write, vault:read]
generated: '2026-07-17'
method: generated
source: https://www.alloovium.com/en/developers
---

# Create a project and add documents

Stand up a new project and populate its document vault.

## Auth
`Authorization: Bearer ak_live_...`. Writes require the `vault:write` scope. Note: dynamically registered OAuth clients receive read-only scopes, so document creation needs an API key or a pre-provisioned OAuth client with write scope.

## Steps
1. **`meta_whoami`** (cost 1) — validate the key first.
2. **`vault_create_project`** (scope `vault:write`, cost 2) — REST `POST /api/v2/vault/projects` with `{ name, project_type }`. Send an **`Idempotency-Key`** header so a retry does not create a duplicate project. Returns the project `id`.
3. **Upload documents** — REST-only `POST` under `/api/v2/vault/documents` (upload, fill, and streaming are intentionally excluded from MCP). Attach the new `project_id`. Idempotency-Key applies to writes.
4. **`vault_list_documents`** (scope `vault:read`, cost 1) — filter by `project_id` or `folder_id` to confirm ingestion; page with `cursor` / `has_more`.
5. **`vault_get_document`** (cost 1) — fetch a document's contents to verify processing.

## Rules
- **Idempotency**: `Idempotency-Key` (1–255 chars) is retained 24h from first write, namespaced per API key. A reused key with a different body returns `422 invalid_idempotency_key`; an in-flight duplicate returns `409 idempotency_in_progress` with `Retry-After`.
- Upload errors: `413 file_too_large`, `400 unsupported_file_type`, `400 invalid_document_id`.
- Respect rate limits (`X-RateLimit-*`; `429 rate_limited` → sleep `Retry-After`).
