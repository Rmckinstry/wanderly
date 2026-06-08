---
name: design-architecture-blueprint
description: Use this skill when the user asks to design, produce, or draft a system architecture, technical blueprint, or end-to-end architecture document set from product requirements, user stories, or an MVP scope.
---

# Design Architecture Blueprint

Orchestrate the production of a complete architecture document set from requirements input.

## Step 1 — Gather inputs

Confirm you have:
- User stories / feature list
- Scale ceiling (users, data volume, growth rate)
- Assume team size is 1, skills, available infra
- Non-negotiable constraints
- Scope of project (internal tool vs external/customer based), (personal vs corporate projects)

If any are missing, ask before designing.

## Step 2 — Choose architecture style

Default to a **modular monolith**. Recommend distributed only when one of these is true:
- Independent scaling of components is required
- Compliance forces process isolation
- Team is already organized around services

When recommending, output:
1. Options considered with honest trade-offs
2. The decisive constraint that drove the choice
3. The extraction / migration path if starting simple

## Step 3 — Run the 7-section process

Produce these sections in order. Delegate to the named skill when present.

| # | Section | Skill to invoke |
|---|---|---|
| 1 | Requirements analysis (scale, team, compliance, env) | — |
| 2 | Technology stack | `select-technology-stack` |
| 3 | System component design (modules, boundaries, enforcement) | — |
| 4 | Data architecture | `design-data-model` |
| 5 | API contracts | `specify-api-contracts` |
| 6 | Security architecture (auth flow, RBAC, TLS, CSP, residency) | — |
| 7 | Performance targets (binding numeric, with measurement method) | — |

For §3, specify the **tooling** that enforces module boundaries (lint rule, build check). Code review alone is not sufficient.

For §6, document the propagation delay between a role change and its enforcement; flag if compliance requires immediate revocation.

For §7, every target must have: metric, numeric value, measurement mechanism. Reject vague targets ("fast enough").

## Step 4 — Lay out the document set

Invoke `structure-architecture-docs` to produce the multi-file layout. Do not emit a single monolithic document.

## Step 5 — Capture debated decisions

For each section, identify any decision that was actively debated (real alternative considered). Invoke `write-adr` for each.

## Step 6 — Self-review

Invoke `review-architecture-docs` against the produced set before handing off.

## Step 7 — Surface open items

Output a table of items that block implementation and require external confirmation (owner, deadline, impact). Implementation of affected components must not start until resolved.
