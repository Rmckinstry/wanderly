---
name: specify-api-contracts
description: Use this skill when the user asks to define, document, design, or specify HTTP/REST/GraphQL API endpoints, request/response schemas, or API contracts.
---

# Specify API Contracts

## Step 1 — Decide URL versioning

Do **not** add `/v1/` by default. URL versioning solves a problem that only exists when multiple independent clients cannot be upgraded together (public APIs, mobile apps, third-party consumers). The scope of this project needs to be determined for this.

For internal tools where frontend + backend are co-deployed and there are no external consumers, keep paths clean: `/api/resources/:id`, not `/api/v1/resources/:id`.

Keep the `/api/` prefix when a reverse proxy uses path-based routing.

If versioning later becomes necessary, prefer header-based (`Accept: application/vnd.app.v2+json`) so paths stay stable.

## Step 2 — Specify every endpoint

For each endpoint emit all of:

```
METHOD /path
Auth:           <required? mechanism>
Authorization:  <which roles>
Request body:   <schema with types and constraints, or "none">
Query params:   <name, type, required?, default>
Response 2xx:   <full schema with example values>
Response 4xx/5xx: <status code | exact triggering condition | response body shape>
Side effects:   <[AUDIT] if write must log before responding; [IDEMPOTENT]; etc.>
```

## Step 3 — Enforce specificity

Reject vague contracts. These are defects:
- "Returns article data" → must list every field with type
- "403 if unauthorized" → must name the specific role check that failed
- "Returns an error" → must give the response body shape

## Step 4 — Group consistently

Group endpoints by resource or module, not by HTTP verb. Each group block names the owning module so `design-data-model` and module boundaries stay aligned.
