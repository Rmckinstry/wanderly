---
name: review-product-spec
description: Use this skill when the user asks to review, audit, critique, or check a product specification, PRD, or product doc set against quality criteria.
---

# Review Product Spec

Audit the product doc set (single file or multi-file under `docs/product/`). Report findings grouped by severity: **blocker / fix-before-handoff / nit**. Do not declare the review passed until every blocker and fix-before-handoff item is resolved.

## Step 1 — Critical questions checklist

For the product as documented, verify:

- [ ] **Existing alternatives.** Are there solutions we're improving upon? Is the improvement specific (not "better UX")?
- [ ] **MVP scope.** Is the minimum viable version defined? Can it be built and validated in one delivery cycle?
- [ ] **Risks.** Are the most likely failure modes named (low adoption, technical infeasibility, unintended user behavior)?
- [ ] **Platform fit.** Have platform-specific requirements been considered (mobile vs. web, offline, accessibility)?
- [ ] **Success measurability.** Can we tell, post-launch, whether this worked? Is the metric numeric with a target?

Any unchecked item is at minimum a **fix-before-handoff**.

## Step 2 — Output standards check

Every spec must be:

- [ ] **Unambiguous** — no room for interpretation; no "etc.", "and so on", "as needed"
- [ ] **Testable** — every story has Given/When/Then acceptance criteria; every NFR has a numeric target
- [ ] **Traceable** — every story maps to a persona that's actually defined; every requirement serves a stated user need
- [ ] **Complete** — happy paths AND edge cases covered; error states defined
- [ ] **Feasible** — technically and economically viable given stated constraints

## Step 3 — Multi-file cross-checks

When reviewing a multi-file set under `docs/product/`, additionally verify:

- [ ] Every persona referenced in `user-stories.md` is defined in `personas.md` (no orphan persona names)
- [ ] Every NFR in `background.md` is reflected in at least one story or acceptance criterion (otherwise it's untestable)
- [ ] No content is duplicated across files (personas only in `personas.md`; NFRs only in `background.md`; priorities only in `user-stories.md`)
- [ ] Story IDs (if used) are unique and sequential within their group
- [ ] If `release-plan.md` exists: every persona has at least one P0 story in the current phase; every dependency listed points to a real story ID

## Step 4 — Priority-label sanity check

Scan all P0 labels:

- [ ] Each P0 has a one-line justification
- [ ] The set of P0s is genuinely the minimum for launch (not "stuff we really want")
- [ ] If more than ~40% of stories are P0, flag it — P0 inflation is a smell

## Step 5 — Report

Output structure:

```
## Blockers
- [File:section] — [issue + concrete fix]

## Fix before handoff
- [File:section] — [issue + concrete fix]

## Nits
- [File:section] — [suggestion]

## Verdict
[Pass | Fix-and-resubmit | Major rework needed]
```

Name the file and specific location for every finding. Vague feedback ("the requirements section is weak") is itself a defect — be specific or don't raise it.
