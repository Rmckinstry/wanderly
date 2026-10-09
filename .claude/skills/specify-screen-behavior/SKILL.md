---
name: specify-screen-behavior
description: Use this skill when the user asks to specify, document, or hand off screen behavior — interactions, keyboard navigation, states, edge cases, or implementation notes for designed screens.
---

# Specify Screen Behavior

Document how each designed screen behaves, so the user can implement it without guessing. Specs and handoff notes only — no application code.

## Step 1 — Organize by feature

Write one file per feature or screen group under `docs/design/screens/`, named in kebab-case after the product feature. Frontmatter:

```
title, status, stories (IDs), last-updated (YYYY-MM-DD)
```

Group screens the way the stories group them.

## Step 2 — Apply the spec block

Each screen gets:

```
Screen:      <name>
Serves:      <story IDs>
Components:  <names from components.md>
States:      default / empty / loading / error / success — what shows, including copy
Interactions: click, hover, focus, drag, and selection behavior
Keyboard:    every shortcut, tab order, and where focus lands on open and close
Motion:      which transitions apply, by token name
Edge cases:  each acceptance-criteria edge case mapped to its UI behavior
Handoff notes: measurements, token usage, ambiguities, open questions
```

## Step 3 — Enforce specificity

Reject vague specs. These are defects:
- "Standard behavior" or "works as expected" — state the behavior
- A story edge case with no mapped UI behavior — map it, or mark it "no UI impact" with a reason
- An interactive screen with no keyboard behavior — keyboard access is baseline, never optional
- Raw values in place of token names — reference tokens and components by name

## Step 4 — Scope accessibility to the posture

Apply the product's stated accessibility requirement. Without a formal target: logical focus order, visible focus, readable contrast, reduced-motion behavior, and no color-only meaning — without compliance matrices or audit checklists.

## Step 5 — Keep handoff notes short

Handoff notes are for the implementer, who is the user. State measurements in rem or px, which tokens apply, and anything ambiguous. Do not write application code, markup, or component implementations. A short behavioral pseudo-description is fine; code is not.

## Step 6 — Cross-check

- Every P0 story maps to at least one screen file
- Every component referenced exists in `components.md`
- Every state shown in a mockup is described here, and vice versa

Output open design questions as a table:

```
| # | Question | Blocks | Needed before |
|---|---|---|---|
| A | <question> | <screen or story> | <phase or screen> |
```

Flag any screen that cannot be implemented until its question is resolved.
