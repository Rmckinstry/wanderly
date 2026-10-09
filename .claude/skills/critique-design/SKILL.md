---
name: critique-design
description: Use this skill when the user asks to critique, review, audit, or check designs — mockups, a design system, or implemented UI — for a generic AI-generated look, consistency with the design system, or drift from the confirmed direction.
---

# Critique Design

Audit designs, or the user's implemented UI, and report findings by severity. Audit only: do not silently rewrite, and never rewrite the user's code. Report findings grouped by severity: **blocker / should-fix / nit**.

## Step 0 — Pick mode and lens

| Mode | Input | Runs when |
|---|---|---|
| **Mockup** | Design artifacts and the design system | After hi-fi mockups, before the handoff package is finalized |
| **Implemented** | The user's code or screenshots | Whenever the user shares built screens; to catch drift |

**Lens.** If the user has chosen a specific critique skill (for example one for removing generic AI-generated design), run it as the lens for Step 2 and merge its findings into this report format. If the user has not chosen one, use the built-in checklist in Step 2. Do not install or recommend a skill unprompted.

## Step 1 — Consistency with the design system

- [ ] Colors, type, spacing, radius, and motion all come from `tokens.json`; list any off-token values
- [ ] Components match `components.md`, with all required states present
- [ ] Spacing follows the scale; no arbitrary one-off values
- [ ] Contrast ratios computed, not eyeballed, against the stated bar
- [ ] Focus indicators visible; keyboard behavior matches the screen spec
- [ ] Both themes checked if the direction calls for both

## Step 2 — Generic-look tells (built-in checklist)

- [ ] Overused or default fonts; flat or accidental type hierarchy
- [ ] Timid, evenly distributed palette; generic gradients; accent color with no purpose
- [ ] Uniform rounded cards everywhere; repeated icon-plus-heading grids; everything centered
- [ ] Emoji or stock icons standing in for real iconography
- [ ] Decoration with no function: shadows, glows, glass effects, blobs
- [ ] Weak hierarchy: no clear first, second, and third thing to look at
- [ ] Generic or placeholder-sounding microcopy
- [ ] A layout that could belong to any product

## Step 3 — Traceability

For each screen, ask whether its look traces to a decision in the vibe summary or direction log in `style-guide.md`. Flag choices that exist only because they are defaults. Pass condition: every visible choice can be tied to a deliberate decision.

## Step 4 — Report

```
## Blockers
- [Screen/file] — [specific issue; rule it breaks; fix in design terms]

## Should fix
- [Screen/file] — [issue + fix]

## Nits
- [Screen/file] — [suggestion]

## Verdict
[Pass | Fix and re-check | Rework]
```

Every finding names the screen or file and the specific element. Fixes are stated in design terms (a token, a component, a state, a layout change) and tied to the confirmed direction. Vague feedback such as "feels generic" or "needs more polish" is itself a defect — point to the element or don't raise it.

## Step 5 — Resolve

- **Mockup mode:** the user approves which fixes to make. Revise the artifact, then re-check. Limit to two rounds unless the user asks for more.
- **Implemented mode:** list the fixes for the user to apply. Do not rewrite their code.
- Do not declare a pass until every blocker is resolved or the user has explicitly accepted it.
