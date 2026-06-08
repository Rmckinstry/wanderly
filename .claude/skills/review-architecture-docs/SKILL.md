---
name: review-architecture-docs
description: Use this skill when the user asks to review, audit, check, lint, or self-review architecture documents for consistency, completeness, scope, or open items.
---

# Review Architecture Docs

Run all three checks. Report findings grouped by severity (blocker / fix-before-handoff / nit).

## Step 1 — Consistency checks

- [ ] All version numbers match across all files (e.g., PostgreSQL 17 in diagram AND tech stack table AND `data-model.md`)
- [ ] Every endpoint path matches between `api-contracts.md` and any references in `system-design.md` or `data-model.md`
- [ ] No stale framework names left from a reversed decision
- [ ] No stale API path prefixes left after a versioning decision
- [ ] ADR cross-references are accurate; superseded ADRs are marked as such in Context

## Step 2 — Completeness checks

- [ ] Every endpoint has full error specification — not just the happy path
- [ ] Every technology choice has rationale + named rejected alternative
- [ ] Every compliance constraint has a failure mode defined
- [ ] Performance targets are numeric and have a stated measurement mechanism
- [ ] Security gaps are explicitly named with deferral rationale and phase assignment
- [ ] Open items requiring external confirmation are in a visible table with owner + deadline

## Step 3 — Scope checks

- [ ] No full implementation code blocks (pattern-illustrating snippets are fine; complete method bodies are not)
- [ ] Directory structures show patterns, not file manifests
- [ ] API contracts contain field-level response schemas, never descriptions like "returns article data"

## Step 4 — Open action items table

Verify the architecture set includes (or generate, if missing):

```
| # | Item | Owner | Needed Before | Impact if Unresolved |
|---|---|---|---|---|
| A | <confirmation needed> | <team> | <phase/sprint> | <what breaks or changes if unresolved> |
```

These are gates, not nice-to-haves. Flag any implementation work currently planned against an affected component as blocked.

## Step 5 — Report

Output one section per check group with pass/fail per item. For each failure, name the file and the specific location. Do not declare the review passed unless every blocker and fix-before-handoff item is resolved.
