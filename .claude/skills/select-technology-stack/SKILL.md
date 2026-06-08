---
name: select-technology-stack
description: Use this skill when the user asks to choose, recommend, evaluate, or justify a technology stack, framework, runtime, database, or library version for a project.
---

# Select Technology Stack

## Step 1 — Classify the project

- **Greenfield** (no existing codebase to migrate): default to **current stable versions**.
- **Brownfield**: align with the existing stack's upgrade path; flag forced upgrades separately.

## Step 2 — Pick per layer

For each layer (frontend framework, state mgmt, backend runtime, API style, DB, cache, infra), output:

```
<Layer> | <Choice + version> | <Primary reason> | <Alternative considered + why rejected>
```

## Step 3 — Validate rationale quality

Every choice must answer: *why this, and not the obvious alternative?*

**Strongest rationale** names a concrete feature, constraint, or trade-off:
> `React 19 | Greenfield — no migration cost. useOptimistic applies to auto-save indicator; useActionState removes useState+useReducer boilerplate in admin forms.`

**Acceptable supporting rationale** (use these to *reinforce* a primary reason, never as the sole reason):
- "Industry standard" / "large supporting community" - legitimate when the choise is between near-equivalent options (e.g, React vs Vue for simple CRUD UI). Pair it with the actual decisive factor.
- "Team knows it best" - legitimate on its own when stated explicitly. Say it, don't dress it up

**Reject** rationale that is *only* "popular" or "modern" with no concrete pairing. If the choice is truly equivalent to alternatives, document it as a low-stakes default rather than inventing reasons.

**Weighting rule** When the primary reason is "industry standard", the rationale cell must still name once concrete pairing (e.g, `React 19 | Industry standard  for SPA UIs + matches team's prior project; alternative Vue rejected - smaller pool of contributors and longer spin up time.`)

## Step 4 — Flag debated choices

For any layer where a real alternative was seriously considered, invoke `write-adr` to capture the decision.

## Step 5 — Emit version-consistency note

Output a single source-of-truth version table. Downstream documents must reference this table, not restate versions, to prevent drift.
