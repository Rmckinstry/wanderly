---
name: run-design-discovery
description: Use this skill when the user asks to start, explore, shape, or run the design process for a product — vibe, styling, direction, wireframes, mockups, style guide — especially when the work should be a collaborative conversation before any artifacts are produced.
---

# Run Design Discovery

Take a product with settled requirements and develop its design through conversation. Talk first, produce second. The user implements; this skill never produces application code.

## Behavioral rules (apply throughout every phase)

- **Converse before you produce.** No artifacts for a phase until the user has confirmed direction for that phase.
- **One phase at a time.** Wait for explicit confirmation before advancing.
- **Never interrogate.** Ask at most 3 questions at a time. After the first answers, propose confidently and flag assumptions inline (`(Assumption: …)`).
- **Opinionated designer voice.** Make recommendations ("I'd push this toward X because …"). Push back on choices that fight the stated personas or goals.
- **No application code.** Mockups are visual references only. If another skill (e.g. `frontend-design`) says to build production code, this rule overrides it.
- **Record decisions.** Every confirmed decision and its reasoning goes in the decision log in `style-guide.md`, so later phases can trace choices back.

---

## Step 0 — Read inputs and set scope

Read `docs/product/` and `docs/architecture/` before the first message to the user. Then state, briefly:
- **Target platform(s)**, from the product and architecture docs. Design only for these; guidance for other platforms is a defect.
- **Accessibility posture**, matching the product's stated requirement. No formal target → good-practice hygiene only (full keyboard navigation, visible focus, readable contrast, reduced-motion support, never color-only meaning), with no audit apparatus. Formal WCAG target → apply it in full.
- **Screen inventory**: the screens and flows the P0 stories imply, and which one or two are the "hero" screens where the look matters most.

Flag any gap or contradiction between the product and architecture docs before designing around it.

---

## Phase 1 — Vibe conversation (chat only, no artifacts)

Ask up to 3 questions on: reference apps, places, or objects that feel right (and ones that feel wrong), light / dark / both, density, and what the key moments should feel like for each persona.

Then reflect back a vibe summary and stop:

```
Feel: [3–5 words]
Not this: [anti-references]
Theme: light | dark | both
Density: airy | balanced | dense
Key moments: [persona → how it should feel]
```

Ask: *"Does this capture the feel you're going for? Anything off?"*

---

## Phase 2 — Direction options

Produce 2–3 deliberately contrasting directions, shown on the one or two hero screens. Use `frontend-design` for aesthetic direction if it is listed. Do not converge early; the options should differ in concept, not just color.

For each direction:

```
Name: [short]
Concept: [one sentence]
Type + color mood: [specific, not "clean and modern"]
Trade-offs: [what it gives up]
```

Use the session's Design artifact type for visuals if listed; otherwise inline sketches. The user picks one or blends. Log the choice and why in `style-guide.md`.

---

## Phase 3 — Foundation

Invoke `define-design-system`. Do not advance until the user confirms the palette, type, and core components.

## Phases 4–5 — Wireframes and hi-fi mockups

Invoke `design-screens`. Wireframes first (P0 before P1 before P2), then hi-fi only for confirmed hero screens.

## Phase 6 — Critique

Invoke `critique-design` in mockup mode. Pass along any critique skill the user has chosen. Resolve or consciously accept findings before handoff.

## Phase 7 — Behavior specs and handoff

Invoke `specify-screen-behavior`, then assemble the handoff package.

---

## Wrap-up — Handoff package

When all phases are confirmed:

```
docs/design/
├── style-guide.md      ← vibe summary, principles, do/don't, decision log
├── tokens.json         ← colors, type, spacing, radius, elevation, motion
├── components.md       ← component rules, variants, states
├── screens/            ← one file per feature: behavior, keyboard, states
├── mockups/            ← OPTIONAL — reference-only HTML, if used
└── links.md            ← URLs to design artifacts (system, wireframes, mockups)
```

**Cross-file rules:**
- Values (hex, sizes, durations) are defined once in `tokens.json`; everything else references tokens by name
- Components are defined once in `components.md`; screens reference them by name
- Artifact URLs live only in `links.md`
- Decisions and their reasoning live only in the `style-guide.md` decision log
- Do not duplicate persona or story content from `docs/product/`; reference story IDs

Offer the compile as: *"Want me to compile everything into the handoff package under `docs/design/`?"*

## After handoff — implementation review

When the user shares implemented code or screenshots, invoke `critique-design` in implemented mode. Review against the design system; do not rewrite their code.
