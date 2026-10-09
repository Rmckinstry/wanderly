---
name: design-screens
description: Use this skill when the user asks for wireframes, mockups, screen layouts, UI designs, or visual designs of screens or flows, at low or high fidelity.
---

# Design Screens

Produce wireframes and hi-fi mockups from the user stories and the confirmed design system. Visual references only — no application code.

## Step 1 — Build the screen inventory

Derive screens and flows from `docs/product/user-stories.md`. Work in priority order: P0, then P1, then P2.

```
| Screen / flow | Stories served | Persona | Fidelity |
|---|---|---|---|
| <name> | <story IDs> | <name> | wireframe | hi-fi |
```

Propose which one or two screens are "hero" screens where the look matters most, and which only need wireframes plus the design system. Get the user's confirmation. A screen that serves no story is flagged, not designed.

## Step 2 — Choose the medium

Check which artifact types the session lists. Prefer the **Design** type for wireframes and mockups, so the user can iterate, compare versions, and export. If it is not listed, fall back to inline sketches, or to reference-only HTML in `docs/design/mockups/` labeled "reference only, not production code." Never deliver production code.

## Step 3 — Wireframes (low fidelity)

- Grayscale. Show structure, hierarchy, content grouping, and navigation between screens.
- Use real labels and realistic content from the product. No lorem ipsum and no "Button" or "Title" placeholders.
- Show the states each screen needs: default, empty, loading, and error where relevant.
- Annotate which stories each screen serves.
- Include a flow map for multi-screen journeys.

Present the wireframes and ask: *"Does this layout and flow work? Anything missing or off?"* Do not start hi-fi until the user confirms.

## Step 4 — Hi-fi mockups

- Apply the design system. Reference tokens and components by name. Do not introduce colors, sizes, or fonts that are not in `tokens.json` or `components.md`; if one is needed, propose a token addition instead.
- Use `frontend-design` for how the look is executed, if it is listed. The no-code boundary overrides its instruction to build production code.
- Show key states, and both themes if the direction calls for both.
- Hero screens get full visual treatment. Other screens follow the design system with minimal extra design.

## Step 5 — Check against the stories

For each screen, confirm that:
- Every acceptance criterion with a UI consequence is visible somewhere in the design
- Each edge case that changes what the user sees (empty state, warning, blocked action) is shown
- Nothing is designed that no story asks for

## Step 6 — Record

- Add each artifact's URL to `links.md`, labeled with the screen and fidelity.
- List unresolved design questions at the end of the response, with what each one blocks.
