---
title: Design handoff to /pm and /architect
status: /pm items done (2026-10-10) — /architect items still open
last-updated: 2026-10-10
---

# Design handoff to `/pm` and `/architect`

Everything the design phase decided that changes the product or architecture docs, and the questions design could not settle alone. Decision numbers (D-xx) refer to the decision log in [style-guide.md](style-guide.md). Items 1–24 were logged during Phases 1–6; 25–31 come from writing the behavior specs.

## For `/pm` — product docs

| # | Change | Decision | Stories / FRs to update |
| --- | --- | --- | --- |
| 1 | Remove country status entirely | D-09 | FR6 (country half), US-001 default `Wishlist`, US-017 (drop), US-024 and US-027 country status filters |
| 2 | "Library" becomes "Countries"; Trips is a sibling section | D-03 | US-005, US-027 wording |
| 3 | Comparison only with 2+ trips; one trip opens straight into the individual view | D-02 | US-033 ("accessible with a note" → not available) |
| 4 | Pinned countries and trips, manually reorderable; the rest newest first | D-15 | new acceptance criteria on US-025 / Trips list |
| 5 | Budget variance: no Status column; four tiers Under / Close / Over / Way over; two global % thresholds | D-18 | FR8, US-020 |
| 6 | No "partial" label on subtotals | D-20 | US-021 edge case |
| 7 | Presentation keys: C for comparison, 1–5 for full-detail sections | D-22 | US-042 |
| 8 | Optional: text highlight in rich text | — | FR13 (if wanted) |
| 16 | "Read everything" view stitching a country's notes with its regions' and cities' notes | D-26 | new story; set P0/P1/P2 |
| 17 | Travel blocks (flight, train, bus, ferry, drive), Day 0 | D-27 | FR7, US-012 |
| 18 | Budget payment model: v1 shared only; per-person / split budgets later | D-28 | FR8, US-021, US-040 (cost per person is P0 today); set P1/P2 for splits |
| 19 | Start a trip from a country links the country only; no block is created | D-29 | US-005 |
| 20 | Help / FAQ view and first-use tutorial — not P0 | — | new stories; set P1/P2; tutorial replayable? uses Sample trip? |
| 22 | Customizable keyboard shortcuts (Settings › Keyboard) | — | new story; not P0 by default |
| 23 | Templates keep categories **and line names**; a budget made from a template is an independent copy | — | US-022 |
| 24 | Tips: one list with type labels; a tip's place is General or a country/region/city, never a trip; new filters | D-38, D-40 | US-053, US-006 vs US-051, US-054 |
| 25 | Trips list status filter is single-select in the design | — | US-027 asks for multiple statuses at once (open question B) |
| 26 | Present is disabled for a window with 0 trips ("Add a trip first") | — | US-037 says "regardless of how many trips" — confirm 0 is excluded |
| 27 | At the last trip in presentation, "Compare all" becomes the primary action | — | US-042 (navigation) — informational |
| 28 | A Travel Window with no target date sorts last in Upcoming ("No date yet") | — | US-030 |

### `/pm` response — 2026-10-10

All `/pm` items above are applied in `docs/product/` (`requirements.md`, `user-stories.md`). Where each landed:

| # | Outcome |
| --- | --- |
| 1 | Country status removed: FR6 is trip-only; US-001 no longer sets `Wishlist`; US-017 retired (ID not reused); US-024 filters by trip status; US-027 is now trips only |
| 2 | "Library" → Countries, with Trips as a sibling section: Content Model note, US-001, US-005, US-025, US-027, US-028, US-030. "The library" still means everything stored |
| 3 | US-033: comparison needs 2+ trips; FR9 says the same |
| 4 | New FR14; criteria added to US-005 (Trips list) and US-025 (Countries list). They sit inside P0 stories, so pinning is now P0 — say so if you would rather split it out as P1 |
| 5 | FR8 and US-020 rewritten: no Status column, four tiers, two global thresholds (Close 10%, Way over 25%, Way over > Close) |
| 6 | US-021: no "partial" label |
| 7 | US-042: C, 1–5 and ? added; single-letter keys only inside presentation |
| 8 | Not in v1. Recorded as planned P2 (FR13, Out of Scope) |
| 16 | New US-063 **P1** (FR15), answers question A |
| 17 | FR7, US-012 (travel blocks, Day 0), US-013 (Expand all / Collapse all) |
| 18 | D-28 applied to FR8, US-021, US-040 (cost per person removed from v1). New US-064 **P1** (FR16) brings it back through split budgets — answers question H |
| 19 | US-005: Start a trip from a country links only that country; no block |
| 20 | New US-065 Help/FAQ and US-066 tutorial, both **P2** (FR17). Tutorial is replayable from Help and uses its own sample content, never the library — answers question G |
| 22 | New US-067 **P2** (FR18) |
| 23 | US-022: templates keep category and line names; a budget made from one is an independent copy |
| 24 | US-006, US-051, US-053, US-054: one list with TIP / LESSON labels; a tip's place is General or a country/region/city, never a trip; Unlinked behaviour (D-37) |
| 25 | **Single-select** — US-027 changed to match the design (question B) |
| 26 | Present is disabled with 0 trips ("Add a trip first") — US-037 changed; confirmed by presentation.md |
| 27 | US-042 edge case added |
| 28 | US-030: a window with no date sorts last in Upcoming, "No date yet" |

Also settled from the design docs (no owner input needed): **question C** — collapsing an expanded block keeps its days, hidden, and expanding again restores them (trips-and-trip-builder.md); US-013 now says so, and the architect's item 30 can proceed on that basis. Item 29 ("Change place…" keeps days within the same country) is also written into US-012.

Changes beyond the handoff, made to match the design — check them:
- **US-021, multiple budgets:** the story said the snapshot shows each budget; the design shows the primary ("Primary · 1 of 2") on the trip and every budget in the Budget screen switcher. The story now follows the design.
- **US-025:** the story lists budgets among a country's contents, but the country detail screen shows no budgets (they live on trips). Not changed — decide whether the story or the screen should move.

## For `/architect` — architecture docs

| # | Change | Decision | Where |
| --- | --- | --- | --- |
| 9 | Drop `countries.status`; replace per-budget `close_threshold` with two global settings; pinned flag + pin order on countries and trips; add `way_over` to variance status | D-09, D-15, D-18 | data-model.md |
| 10 | `presentation:comparison:get` must return the trip concept (`notes`) | FR10 | api-contracts.md |
| 11 | Missing routes: `/settings`, recovery screen, first-run restore, budget template management, `/tips/:id`; `/trips/:id/budget` must address one of several budgets | — | architecture.md route inventory |
| 12 | Hero image fallback: designated hero, else first available photo from the trip or its linked places | US-041 | api-contracts.md (comparison + presentation) |
| 13 | Bundle Hanken Grotesk, JetBrains Mono, Newsreader (CSP `font-src 'self'`) | D-12 | architecture.md |
| 14 | UI state store (suggested `settings.json`): palette, theme, nav collapsed, section expanded states, list/grid per list, sort per list, Travel Window view per window, expanded blocks per trip, budget thresholds, tutorial seen, keymap | D-17, D-24, D-30, D-34, D-44 | architecture.md |
| 17 | Travel blocks: `trip_blocks.kind` (location / travel), nullable `country_id`, `travel_mode`, `from_label`, `to_label`, `duration_days` ≥ 0 (a 0-day first block is Day 0, excluded from trip length) | D-27 | data-model.md |
| 18 | Budgets gain `payment_mode` (shared / split) and a travelers table (name, share %), replacing `num_travelers` — when `/pm` schedules it | D-28 | data-model.md |
| 21 | Recovery: keep the damaged file beside the library (`wanderly-damaged-<date>.db`); first-run folder pick looks one level down for `Wanderly Backup`; a folder restored from on first run becomes the backup folder; restore-point rows show counts | — | architecture.md (Backup and Recovery Pattern) |
| 29 | Refining a block ("Change place…") updates the block's place and keeps its days when the new place is in the same country | US-012 | api-contracts.md (block update) |
| 30 | Collapsing an expanded block keeps its days and content (hidden), per the design — confirm storage keeps days when `mode` returns to location-block | US-013 | data-model.md — see open question C |
| 31 | Restore copy progress (determinate) for first-run restore from a folder | NFR6 | api-contracts.md (backup:restore progress events) |

## Later (P2)

| # | Item |
| --- | --- |
| 15 | User-chosen budget category colours from a curated per-palette swatch set (D-25): a `color` field on budget categories |

## Open design questions

| # | Question | Blocks | Needed before | Status |
| --- | --- | --- | --- | --- |
| A | Is the "Read everything" view (item 16) P0, P1 or P2? | Country detail (a "Read everything" entry point in More ▾) | Country detail build | **Answered: P1** — country detail ships without the entry point until US-063 |
| B | Should the Trips list status filter allow several statuses at once (US-027, P1)? | Trips list | Trips list build (P1 filter) | **Answered: single-select** — design stands |
| C | When an expanded block is collapsed, are its days and their content kept (design) or removed (one reading of US-013)? | Trip builder — collapse confirm copy and data | **Trip builder build** | **Answered: kept, hidden** — from the design spec; US-013 now says so |
| D | Should the presentation hero photo sit on a `mat` frame (as the token describes) or bleed to its own edges (as the hi-fi shows)? | Presentation summary | Presentation polish (not blocking) | Open — design |
| E | Follow-up lesson block on Tip detail: bordered `surface` box (spec) instead of the wireframe's left rule — OK? | Tip detail (P1) | Tips build | Open — design |
| F | Can a first-run restore be cancelled mid-copy? (Design shows no Cancel once copying starts.) | First run — restore | **First-run restore build** | Open — `/architect` and owner |
| G | Help / tutorial priority and scope (item 20) | none in P0 | P1 planning | **Answered: both P2**; tutorial replayable, own sample content |
| H | Per-person / split budgets (item 18) | Presentation, Budget | When `/pm` schedules item 18 | **Answered: P1** (US-064). Presentation's ledger gains a "Split 70 / 30" note; per-person amounts in full detail only |

**Screens that cannot be finished until a question is answered:** only the first-run restore progress step (F). Everything else can be built now with the assumptions stated in its spec.

## P0 coverage

Every P0 story maps to a screen file:

| Story | Screen file |
| --- | --- |
| US-001, 002, 003, 007, 007b, 008, 025 | countries-and-places.md (+ app-shell.md) |
| US-004, 009 | poi.md |
| US-005, 012, 013, 014, 015, 016, 016b | trips-and-trip-builder.md |
| US-017 | no UI — removed by D-09 (item 1) |
| US-018, 019, 020, 021, 022 | budget.md, settings.md |
| US-023, 024, 026 | search.md |
| US-030, 031, 032, 032b, 033, 034, 034b, 035 | travel-windows.md |
| US-037, 038, 039, 040, 042, 043 | presentation.md |
| NFR6 | settings.md, recovery-and-first-run.md |
