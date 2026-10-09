---
name: define-design-system
description: Use this skill when the user asks to define, create, or document a design system, style guide, design tokens, color palette, typography, spacing, or component rules for a confirmed visual direction.
---

# Define Design System

Turn a confirmed vibe and direction into a design system the user can code from: tokens, component rules, and a style guide. No application code.

## Step 1 — Confirm prerequisites

- The vibe summary and direction decision exist (see `run-design-discovery`). If not, stop and run those first. A design system built without a direction defaults to generic.
- State the target platform(s). Include only guidance relevant to them — no mobile breakpoints for a desktop-only product, no iOS or Android sections for a web product.
- State the accessibility posture from the product's stated requirement. Without a formal target: full keyboard navigation, visible focus indicators, contrast of 4.5:1 for body text and 3:1 for large text, `prefers-reduced-motion` support, and never encoding meaning by color alone. With a formal target: apply it in full.

## Step 2 — Define tokens

Define every token with a **name and a role**, not just a value. Values appear here and nowhere else.

- **Color:** background, surface, text, text-muted, border, primary, accent, and semantic (success, warning, error, info). Provide light and dark values only if the direction calls for both. State the dominant color and how accents are used.
- **Typography:** families with the reason each was chosen from the direction (never default system or overused stacks), a size scale with line-height and weight, and a monospace choice if needed.
- **Spacing:** a base unit (4px or 8px) and a named scale.
- **Radius, elevation, borders:** named steps, with a stated rule for when each applies.
- **Motion:** duration scale and easing curves, plus the reduced-motion behavior.

Compute contrast ratios for every text/background pair and record the pass/fail result. Do not estimate by eye.

## Step 3 — Specify components

List only components the P0 stories need. Derive the list from `docs/product/user-stories.md`. Do not spec components no story requires.

For each component:

```
Component: <name>
Variants:  <e.g. primary / secondary / quiet>
States:    default, hover, active, focus, disabled, loading, error, empty (where relevant)
Tokens:    <token names used — no raw values>
Usage:     <when to use, when not to>
Keyboard:  <focus and key behavior>
```

Reject vague specs. "Standard button" and "styled as expected" are defects.

## Step 4 — Write the style guide

- 3–5 principles derived from the confirmed vibe (not generic principles like "be consistent")
- Do and don't pairs, with concrete examples
- Rules for iconography, imagery, density, and microcopy voice
- The decision log: each confirmed decision and its reasoning

## Step 5 — Export and publish

- Write `tokens.json` so the user can consume it directly. Structure by category and role, with light/dark variants where applicable:

```json
{
  "color": { "background": { "light": "…", "dark": "…" } },
  "type":  { "family": { "display": "…", "body": "…" }, "scale": { } },
  "space": { },
  "radius": { },
  "motion": { "duration": { }, "easing": { } }
}
```

- Write `style-guide.md` and `components.md` under `docs/design/`.
- If the session lists a Design System artifact type, also create it from that type so tokens and components have live previews. Add its URL to `links.md`. Tokens remain defined in one place; the artifact must match `tokens.json`.

## Step 6 — Confirm before moving on

Restate the palette, type pairing, and key component decisions in a few lines. Ask the user to confirm before wireframes begin.
