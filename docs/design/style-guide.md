---
title: Style guide
status: locked (design system v1, D-23; critique round 1 passed, D-45)
last-updated: 2026-10-09
---

# Wanderly — style guide


Wanderly is a personal travel notebook for a desktop computer (macOS and Windows, Electron). It keeps years of research — countries, regions, cities, places, trips and budgets — in one organised, searchable library, and turns a shortlist into a calm presentation for deciding where to go next. This system is the visual language for that app. Everything here is a reference for implementation; previews are reference only, not production code.

**Platform** — desktop only, minimum window 1280px wide. No mobile or touch guidance applies.
**Accessibility** — no formal WCAG target; good-practice hygiene always: full keyboard navigation, a visible focus ring on every control, text contrast ≥4.5:1 (≥3:1 for 24px+ and control borders), `prefers-reduced-motion` respected, and meaning never carried by color alone.

## Principles

1. **The notes are the page.** A country is a notebook first. Notes sit directly on the ground at a comfortable 16px/1.7 and get the widest column; everything else moves to the right column or below.
2. **Organised, not decorated.** Structure comes from rules, rhythm and counts — not boxes, gradients or ornaments. The faint contour lines in a country header are the only decoration in the app.
3. **Say it in words.** Buttons are labeled, statuses carry their word, variance carries a glyph and a word. If a person has to guess what something does or means, it is wrong.
4. **Place first, then facts.** In presentation, the photo and the trip's idea come first, then the same facts in the same places on every trip, so a non-user learns where to look once.
5. **Quiet until asked.** Tools appear for what's selected; the nav collapses away; nothing nags a stub to be "completed".

## Color

Two palettes, each with a light and a dark theme, chosen in Settings › Appearance: **Paper** (default — warm paper, ink, terracotta) and **Atlas** (stone, deep ink-green, teal). Presentation follows the selected palette and theme.

- Dominant: `bg` and `text`. Most of every screen is ground and ink.
- `accent` has exactly three jobs: the one primary action on a screen, links, and the selected/hero marker (plus the `focus` ring, an alias). Never decorative fills, never headings.
- Semantic colors are shared across palettes: `success` (Under, Ready), `warning` (Close), `danger` (Over, destructive), `danger-strong` (Way over), `info` (Planning).
- Budget status lives in the Difference value itself (D-18): Under ↓ green, Close ≈ amber, Over ↑ red, Way over ⇈ as a filled deep-red pill. The glyph and the pill shape carry the meaning alongside color. Thresholds are global percentages in Settings › Budget (Close within 10%, Way over beyond 25% by default).
- `chip` is for tags and filters only, so a tag never looks like a button.
- `cat-1`…`cat-5` color budget categories in the split bar, always with a labeled legend.

### Contrast (computed, WCAG relative luminance)

| Pair | Paper light | Paper dark | Atlas light | Atlas dark |
| --- | --- | --- | --- | --- |
| `text` on `bg` | 14.45 | 14.56 | 15.05 | 14.99 |
| `text` on `surface-active` (lowest ground) | 12.19 | 12.25 | 12.67 | 12.39 |
| `text-muted` on `bg` | 5.80 | 7.15 | 5.92 | 6.84 |
| `text-muted` on `surface-active` | 4.90 | 6.01 | 4.99 | 5.65 |
| `accent` on `bg` | 5.54 | 7.25 | 6.16 | 7.69 |
| `accent` on `surface-active` | 4.67 | 6.10 | 5.19 | 6.36 |
| `on-accent` on `accent` | 5.83 | 7.25 | 6.55 | 8.06 |
| `success` lowest pair (`surface-active`) | 5.00 | 6.12 | 4.96 | 6.00 |
| `warning` lowest pair | 4.81 | 6.62 | 4.78 | 6.75 |
| `danger` lowest pair | 5.02 | 5.11 | 4.99 | 5.21 |
| `border-control` on `bg` (non-text) | 3.33 | 3.73 | 3.09 | 3.56 |
| `accent` vs `cat-5` (route-strip neighbours, non-text) | 3.33 | 3.42 | 3.56 | 3.24 |
| keycap on filled button (`on-accent` on `accent`) | 5.83 | 7.25 | 6.55 | 8.06 |
| `on-danger-strong` on `danger-strong` | 8.79 | 5.43 | 8.79 | 5.43 |
| `danger-strong` pill on `bg` (non-text) | 8.34 | 3.23 | 8.34 | 3.23 |

All pass. `border` is a decorative hairline (≈2:1) and is never the only edge of a control; inputs use `border-control`.

## Typography

Three families, each with one job. All are open-licensed and must be **bundled with the app** (the app's CSP allows only local fonts; previews here load them from Google Fonts).

- **Hanken Grotesk** — all UI and all notes. Chosen from the Atlas direction for its sturdy large sizes and calm text texture; it reads like a modern notebook, not a dashboard.
- **JetBrains Mono** — numbers only: counts, durations, day numbers, money, shortcut keys. Fixed-width figures line up in budgets and comparisons.
- **Newsreader** — trip titles in presentation, nowhere else. The one "slight editorial" moment, reserved for the decision conversation.

Scale: `page-title` 44/700 · `section` 21/600 · `subsection` 16/600 · `notes` 16/1.7 · `ui` 14.5 · `ui-strong` 14.5/600 · `small` 13 · `micro` 12 (never smaller) · `num` 13.5 · `num-sm` 12 · `money-md` 22 · `money-lg` 34 · `trip-title` 64/400 serif · `trip-title-sm` 24.

## Space, shape, depth, motion

- 4px base; use only `space-1` (4) … `space-14` (56). Screen padding `space-12`; notes-to-column gutter `space-14`.
- Radius: `radius-md` 6 for buttons, inputs, chips · `radius-lg` 8 for cards · `radius-xl` 10 for panels and dialogs · `radius-sm` 4 for photos · `radius-full` for status dots only.
- Flat by default. `shadow-float` only for things above the page: menus, dialogs, the search overlay. Cards never cast shadows.
- Motion: `duration-fast` 120ms hover · `duration-base` 180ms expand/collapse · `duration-slow` 240ms overlays and entering presentation; `ease-out` in, `ease-in` out. With reduced motion, everything is instant.

## Density

Balanced: airy at rest, dense content stays readable. One label per section at most — no stacked eyebrow labels. Cards only for things you pick up, move or compare (trips, POIs on a day, shortlisted options, budget snapshots); notes, hierarchy and lists sit on the page.

## Iconography and imagery

- Icons are rare and functional: chevrons, drag handle, pin, close, collapse. Stroke style, `currentColor`, 16px grid, 1.5px stroke. Never icon-only buttons in the workspace; icon buttons (close, collapse) always carry an accessible label and a tooltip.
- Photos are small in the workspace (3-across thumbnails) and framed on a `mat` in presentation. The hero is always chosen by the person. A missing photo shows a quiet neutral block, never a broken image.
- No illustrations, mascots, emoji or stock textures.

## Voice

Plain, specific, first-person-adjacent — it's your notebook. Verbs on buttons ("Start a trip", "Delete region"). Errors name the fix ("A country named Thailand already exists — open it"). Empty states invite without nagging ("Nothing in Peru yet"). No exclamation marks, no "Oops".

## Do and don't

- **Do** put a country's notes in the wide left column. **Don't** wrap notes in a card.
- **Do** show "↑ +$180" in `danger` in the Difference cell. **Don't** add a separate status column or tint a budget row red.
- **Do** label toolbar actions ("Expand to days"). **Don't** ship a row of unlabeled icons.
- **Do** use `accent` for the single primary action. **Don't** use it for headings, chips or decoration.
- **Do** keep the same fields in the same place on every presentation trip. **Don't** hide a missing field — say "Not yet added".
- **Do** use mono for numbers. **Don't** use mono for labels or prose.

## Decision log

| # | Decision | Why |
| --- | --- | --- |
| D-01 | Hero screens: Country detail, Trip builder, Presentation | Most-used, hardest interaction, and the only screen a non-user sees |
| D-02 | Comparison only with 2+ trips; one trip opens straight into the individual view | No dead toggles; "zero explanation" (FR12). `/pm`: reword US-033 |
| D-03 | Countries is the master list; Trips is a sibling section | Trips are planned off countries. `/pm`: reword US-005, US-027; "Library" renamed "Countries" |
| D-04 | Settings: Appearance, Backup & restore, Budget templates, Tags (P2) | Theme choice needs a home; NFR6 backups need one |
| D-05 | Vibe: enhanced modern organised notebook; light and dark; presentation follows the theme | User brief; OneNote feel without its dated chrome |
| D-06 | Cards only for movable or comparable things | Avoid "card-everything" generic look |
| D-07 | Labeled, context-aware toolbar | Never guess what an icon does |
| D-08 | Dark mode is blue-black ink, never brown | User rejected brown dark |
| D-09 | Countries have no status | A country is a research container; depth shows in counts. `/pm` + `/architect`: remove FR6 country status, US-017, `countries.status`, status filters for countries |
| D-10 | Two palettes, Paper (default) and Atlas | User liked both A's and B's colors; capped at two to keep testing honest |
| D-11 | Presentation goes deep (full detail inside presentation) | US-043; user wanted more than the summary |
| D-12 | Hanken Grotesk for UI and notes; Newsreader only for presentation trip titles; JetBrains Mono for numbers | User liked B/C sans and the OneNote feel; one editorial moment kept |
| D-13 | Contents, Photos, Tips in the right column; trips below the notes | Maximise note-writing room; trips out of the way |
| D-14 | Richer presentation summary (highlights by stop, budget split) plus full detail one ↵ away | "Look how cool" plus "here are the facts" |
| D-15 | Nav lists: Pinned (reorderable) then newest first; sort options only on the full list page | Avoid clutter; quick access to what matters. `/architect`: pinned flag + order on countries and trips |
| D-16 | Direction F locked | Synthesis of A–D feedback |
| D-17 | Side nav fully collapsible (⌘\, edge button, remembered) | More room for notes; labels-only rule rules out an icon rail |
| D-18 | Budget has no Status column; Difference is color-coded in four tiers (Under, Close, Over, Way over) with glyphs; Close % and Way-over % are global settings | Less table noise; one place to read status. `/pm` + `/architect`: replace per-budget `close_threshold` with two global settings, add the Way-over tier to variance status |
| D-19 | Shortcut keys in buttons are fixed-height keycaps centered with the label; the add "+" is a drawn icon, not a character | Mono, sans and the "+" glyph sit on different baselines; fixed boxes keep everything on one axis |
| D-21 | Trip status: Sample hollow dot, Planning blue (`info`), Ready green (`success`), Completed filled ink dot + date | Green means "ready"; keeps Planning distinct from budget Under |
| D-22 | Presentation keys approved: C opens comparison, 1–5 jump to full-detail sections | Faster navigation in the decision conversation |
| D-23 | Design system locked for wireframes | User review, first pass complete |
| D-24 | Countries list has List (default) and Grid views, switchable and remembered | Dense scanning and visual browsing both have a place; stubs get an initial placeholder in Grid |
| D-25 | (P2 idea) Budget category colors chosen by the user from a curated per-palette swatch set (~8, contrast-checked) | User request, deferred |
| D-26 | Countries, regions and cities are sub-pages, each with its own notes. The right panel is "Inside <place>" with → rows; notes are labeled "<place> notes"; region and city pages show "Also in <parent>" siblings | User confirmed the hierarchy model; the old "Contents" label read like a table of contents for one page |
| D-27 | Trip builder adds Travel blocks (flight, train, bus, ferry, drive): free-text from/to, whole days or 0 (a first-block 0-day departure is Day 0 — the evening you leave after work; later 0-day travel is overnight / same day), notes; count toward trip length and per-day cost; no POIs; no automatic budget lines | Long-haul travel days are real and change trip length and cost per day; no half days, no coupling with budgets |
| D-28 | Budgets are shared (one pot) by default; cost per person is removed from this version until split budgets are defined | User decision; per-person needs the split model (handoff 18) to be meaningful |
| D-20 | No "partial" label on subtotals | The Pending lines already show what's missing. `/pm`: US-021 edge case ("labeled as partial") changes |
| D-30 | A Travel Window opens side by side when it has 2+ trips; one at a time for a single trip, or when the user last chose it (remembered per window) | Side by side is where the choice gets made |
| D-31 | Present is ⌘↵ outside presentation — no single-letter shortcuts in the app | Single letters would fire while typing in notes; single letters stay inside presentation (D-22) |
| D-32 | "Mark as chosen" lives only in the app's Travel Window view, never in presentation | Presentation is the conversation with the Co-Decider; the choice is recorded afterwards |
| D-33 | Child pages (region, city, POI, tip) get an obvious "← <parent>" Button above the title, plus ⌘↑ / Alt+↑; it goes up one level. History back stays ⌘[ / Alt+← and the mouse back button | The breadcrumb alone was too easy to miss. Up-a-level is predictable; history back can land anywhere (search, a trip) |
| D-34 | Trips list has List (default) and Grid views, switchable and remembered — same pattern as Countries (D-24) | User decision; one consistent list pattern |
| D-35 | A fresh install starts with an empty library — no seeded sample content | Samples would need deleting by hand; the later tutorial (handoff 20) can bring its own |
| D-36 | "Must do" stays a single-select POI tag, drawn heavier than other chips | Already a tag in the data model; filters and searches like any tag; no second way to classify |
| D-37 | Unlinked tips have no banner in the Tips list; they show only when filtering by Place › Unlinked. The sidebar's "Unlinked 1" count under Tips is the one standing reminder | User decision; keeps the list calm while the count stops them being forgotten |
| D-38 | Tips and lessons learned share one list, told apart by a TIP / LESSON label (never color alone); a follow-up lesson sits under its tip | User decision; follow-ups belong next to what they answer |
| D-39 | Tips stays in the main nav | User decision; General tips and Unlinked review need a home |
| D-40 | A tip or lesson is either General or tied to a place (country, region or city). A trip is never a tip's place — it appears only as a lesson's optional "from" source (Completed trips only) | One rule for where a tip shows up; resolves US-006 vs US-051 |
| D-41 | Contour lines appear only in country, region and city headers — never on POIs, trips, lists, Settings, Tips, Travel Windows or presentation. They sit behind content (no pointer events) and stop at the toolbar | The motif means "a place that holds places"; used everywhere it becomes wallpaper, and working screens stay quiet |
| D-42 | The Toolbar is a full-width `surface` band with a `border-control` rule below; buttons are outlined on `bg` with accent "+" icons; in the trip builder the selection leads as a "Selected · Chiang Mai · 6 days" chip and a divider splits its actions from the add actions; the bar sticks on scroll. The trip builder's right column drops Trip notes (budget + windows only) | User: the trip builder was busy and the toolbar got lost |
| D-43 | Presentation trip title (`trip-title`) is 64px Newsreader 400; `trip-title-sm` (24px) stays for full detail and comparison | User compared 48 vs 64 in hi-fi; 64 carries the "look how cool" moment |
| D-44 | Trip builder has Expand all / Collapse all above the itinerary (⌥⌘↓ / ⌥⌘↑), with a live "7 blocks · 1 expanded" count. Blocks without a duration stay collapsed on Expand all; travel blocks never expand. Expanded state is remembered per trip (UI state, handoff 14) | User request; long trips need a quick overview and a quick way into detail |
| D-45 | Critique round 1 fixes: keycaps on filled buttons are outline-only (was 4.05:1); keycap text 12px; selected nav = `surface` + `border-control` outline + `accent` label; `:focus-visible` 2px accent ring defined for every control (not drawn on the mockups — the user preferred them without it; it shows only on keyboard focus); drawn grip icon; nested radius 1:2 (PoiCard `radius-sm` in a block); spacing snapped to the scale; new `lede` 18/1.6 style for presentation; presentation estimate uses `money-lg`; Paper `cat-5` adjusted (PL #c6bfb2, PD #454c5b) so route neighbours are ≥3:1; travel blocks use `sidebar` fill and mode labels | critique-design, round 1 |
| D-29 | "Start a trip" from a country creates the trip with only that country linked (its chip in the header) — no location block | User decision; the builder starts clean and the user places the first block themselves |

## Vibe summary

An enhanced, modern, organised notebook (D-05): the calm of a paper notebook with the structure of Linear or Notion, never their stock "AI app" look. Light and dark, two palettes. Notes are the page; tools appear for what you've selected; one editorial moment (the serif trip title) is reserved for presentation, where the Co-Decider sees the trip. Direction F (D-16) is the synthesis of explorations A–D.

Personas: the **Trip Architect** (builds everything, keyboard-first, desktop) and the **Co-Decider** (sees only presentation mode; must never need an explanation).

Open items for `/pm` and `/architect` are in [handoff-to-pm-architect.md](handoff-to-pm-architect.md).
