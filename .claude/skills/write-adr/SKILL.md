---
name: write-adr
description: Use this skill when the user asks to write, generate, draft, or propose an Architecture Decision Record (ADR) for a technology, design, or architectural choice.
---

# Write ADR

## Step 1 — Verify the decision deserves an ADR

Write an ADR **only** when all of these are true:
- Two or more concrete options were compared
- The decision was not obvious given the constraints
- A future engineer might reasonably try to reverse it
- The rationale is more nuanced than fits in a table cell

Do **not** write ADRs for: default choices nobody contested, implementation details, or anything fully explained by an already-documented constraint.

If the decision fails this gate, tell the user and recommend documenting the choice inline in the relevant section instead.

## Step 2 — Assign the next ADR number

Look in `docs/architecture/decisions/` for existing `ADR-NNN-*.md` files. Use the next sequential 3-digit number.

## Step 3 — Apply the template

```markdown
# ADR-NNN: [Decision Title]

**Status:** Accepted
**Date:** YYYY-MM-DD
**Applies to:** [phases or components]

## Context
[What situation prompted this decision? What constraints apply?]

## Decision
[The chosen option, stated clearly.]

## What Was Debated
[The specific comparison made. What was the alternative? What made the choice non-obvious?]

## Alternatives Considered
[Each alternative with the specific reason it was not chosen.]

## Consequences
**Positive:** [Concrete benefits]
**Negative:** [Accepted costs or risks — and mitigation or trigger to revisit]
```

## Step 4 — Save and cross-link

- Write to `docs/architecture/decisions/ADR-NNN-short-title.md` (kebab-case title, ≤6 words)
- Add a reference from the main architecture document at the location where the relevant choice is documented
- If this ADR supersedes a prior one, set the prior one's status to `Superseded by ADR-NNN` and link back from the new ADR's Context section

## Step 5 — Confirm before declaring done

Restate the title, number, file path, and what was decided so the user can confirm the ADR matches intent.
