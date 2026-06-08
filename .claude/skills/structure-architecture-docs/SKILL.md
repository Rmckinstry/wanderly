---
name: structure-architecture-docs
description: Use this skill when the user asks how to organize, lay out, structure, or split architecture documentation across files and directories.
---

# Structure Architecture Docs

## Step 0 — Pick deliverable scale

Default to **Compact**. Pick **Standard** only when one of the Standard triggers below is true. State the chosen mode explicitly in the output so the user can override.

| Mode | Pick when | Files emitted |
|---|---|---|
| **Compact** (default) | Team ≤ 5; MVP / single-phase work; no formal compliance review | `architecture.md`, `data-model.md`, `api-contracts.md`. `decisions/` only if an ADR is warranted. |
| **Standard** | Compliance / audit obligations apply; team > 5 with separate stakeholder vs. engineering audiences; multi-phase delivery with locked-down stable layer | `architecture.md`, `system-design.md`, `data-model.md`, `api-contracts.md`, `decisions/`. |

Any compliance / audit requirement forces Standard regardless of team size.

---

## Step 1 — Compact layout

```
docs/architecture/
├── architecture.md     ← Everything except data and APIs
├── data-model.md       ← Schema for every table / collection
├── api-contracts.md    ← Every endpoint specification
└── decisions/          ← Only created when an ADR exists
    └── ADR-NNN-*.md
```

`architecture.md` uses this fixed section order. Omit sections marked *(optional)* entirely when not applicable — do not emit empty placeholders.

```
# Architecture

## 1. Overview
   - System diagram (with module names overlaid)
   - Tech stack table with rationale
   - Non-negotiable constraints with failure modes
   - Phase delivery map (optional — only if multi-phase)

## 2. Modules
   - Module map with responsibilities
   - Module boundary rules + enforcement mechanism

## 3. Patterns & Flows
   - State machines + authorization matrices
   - Optimistic locking / conflict resolution (optional)
   - Event bus design (optional)
   - Audit write pattern (optional — only if compliance applies)

## 4. Security
   - Auth flow, CSP, RBAC layers, TLS

## 5. Performance
   - Budgets with measurement mechanism

## 6. Frontend
   - Route inventory
   - Offline architecture (optional)

## 7. Open Action Items
```

---

## Step 2 — Standard layout

```
docs/architecture/
├── architecture.md     ← Declarative inventory (stakeholder view)
├── system-design.md    ← Operational patterns (engineering view)
├── data-model.md
├── api-contracts.md
└── decisions/
    └── ADR-NNN-*.md
```

**`architecture.md`** — declarative inventory only:
- System diagram (module names overlaid — names only, not responsibilities)
- Tech stack table with rationale
- Non-negotiable constraints with failure modes
- Phase delivery map
- Frontend route inventory
- Open action items

**`system-design.md`** — operational patterns, never full implementations:
- Module map with responsibilities (single source of truth — do not duplicate in `architecture.md`)
- Module boundary rules + enforcement mechanism
- State machines + transition authorization matrices
- Optimistic locking / conflict resolution flows
- Audit write pattern (concept + skeleton, not full code)
- Event bus design
- Offline architecture
- Security design (auth flow, CSP, RBAC layers, TLS)
- Performance budgets

**`data-model.md`** — full schema per `design-data-model`.

**`api-contracts.md`** — every endpoint per `specify-api-contracts`.

Do **not** include in any file: full method bodies, file-level directory trees (they go stale immediately).

---

## Step 3 — Document directories as patterns, not file manifests

Show architectural groupings. Filenames change constantly; the pattern is what matters.

```
# Good — shows the pattern
src/
├── modules/         ← One directory per domain module
│   └── <module>/
│       ├── index.ts        ← PUBLIC surface only
│       └── *.repository.ts ← PRIVATE (enforced by linter)
└── infrastructure/  ← Shared runtime clients
```

Do not enumerate every concrete file. Pattern only.

---

## Step 4 — Cross-reference, do not duplicate

Single-source-of-truth rules apply in both modes:

- API path appears in `api-contracts.md` once; other files / sections link to it
- Tech version appears in the tech stack table once; other files / sections reference by name only
- ADR rationale lives in the ADR; other files / sections link to it
- **Standard mode:** module responsibilities live in `system-design.md` only; `architecture.md`'s diagram shows names without restating them
- **Compact mode:** the rule applies *within* `architecture.md` sections — e.g., the Patterns & Flows section references API paths from `api-contracts.md`, never restates them
