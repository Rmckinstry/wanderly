---
title: Design links
status: current
last-updated: 2026-10-13
---

# Design links

All artifacts are private claude.ai pages — they open only for the owner until shared from each page's Share menu.

| Artifact | What it is | Status |
| --- | --- | --- |
| [Wanderly — design system](https://claude.ai/artifact/Sr2Lwo3bAhuugsrSFENRLN) | Tokens (4 themes), type, 32 component cards with previews, decision log D-01–D-54, open items | Source of truth for tokens and components; `tokens.json` and `components.md` here are copies |
| [Wanderly Hi-fi](https://claude.ai/artifact/M82d52APZzWsNw3t4RY8CL) | Hero screens: Country detail, Trip builder, Presentation, Travel Window (side by side), Budget — Paper light/dark, Atlas light. Theme switchable per board in Tweaks | Approved after critique round 1 (D-45) |
| [Wanderly Wireframes](https://claude.ai/artifact/R3oHahTJLhPsGUTGURZKPR) | 41 grayscale screens in 5 batches plus a flow map; trip rows, toolbar band and nav match the hi-fi (D-42, D-46, D-47): countries, places, trip builder, budget, Travel Windows, presentation, search, POI, trips, settings, recovery, first run, tips | Structure reference; hi-fi wins where they differ |
| [Wanderly Directions](https://claude.ai/artifact/QCNkcXZBiC7qNUk5sBv84B) | Phase 2 explorations A–C and F (D, E removed) | Superseded by Direction F (D-16); history only |

## Which board answers which spec

| Spec file | Wireframe boards | Hi-fi boards |
| --- | --- | --- |
| screens/app-shell.md | all | Main, Trip (and dark) |
| screens/countries-and-places.md | CountriesList, CountriesGrid, CountryDetail, CountryStub, CountryRefused, CityDetail, RegionDelete | Main, CountryDark |
| screens/poi.md | POIDetail | — |
| screens/trips-and-trip-builder.md | TripsList, TripsGrid, TripBlocks, TripMixed, TripSearch, TripNoCountries, TripStatus, TripFromWindow, TripShorten, TripSplit | Trip, TripDark |
| screens/budget.md | Budget, BudgetEmpty, TemplateOverwrite | Budget, BudgetDark |
| screens/travel-windows.md | TWList, TWNew, TWDetail, TWPast | Window, WindowDark |
| screens/presentation.md | PresSummary, PresDetail, PresCompare, PresSingle | Present, PresentDark, PresentAtlas |
| screens/search.md | SearchOverlay, SearchResults, TripSearch | — |
| screens/tips.md (P1) | TipsList, TipDetail, TipNew | — |
| screens/settings.md | SettingsAppearance, SettingsBudget, SettingsBackup, SettingsTemplateNew, TemplateOverwrite | — |
| screens/recovery-and-first-run.md | Recovery, FirstRun, FirstRunRestore | — |

Screens without hi-fi are built from the hi-fi patterns plus the tokens and component specs; mockups are reference only, not production code.

## P0 story coverage

Every P0 story maps to a screen file:

| Story | Screen file |
| --- | --- |
| US-001, 002, 003, 007, 007b, 008, 025 | countries-and-places.md (+ app-shell.md) |
| US-004, 009 | poi.md |
| US-005, 012, 013, 014, 015, 016, 016b | trips-and-trip-builder.md |
| US-017 | no UI — retired by D-09 (country status removed) |
| US-018, 019, 020, 021, 022 | budget.md, settings.md |
| US-023, 024, 026 | search.md |
| US-030, 031, 032, 032b, 033, 034, 034b, 035 | travel-windows.md |
| US-037, 038, 039, 040, 042, 043 | presentation.md |
| NFR6 | settings.md, recovery-and-first-run.md |

## Design backlog

Every P0 state is drawn and every design question is answered (as of 2026-10-13). These screens are still to design when their stories are scheduled:

| Story | Priority | What needs designing |
| --- | --- | --- |
| US-063 Read everything | P1 | The reading view (stitched notes under place headings, links to each place), its empty state, and the "Read everything" entry in the country's More ▾. Country detail ships without the entry point until then |
| US-064 Split budgets | P1 | Shared / Split switch and travelers editor (name, share %, must total 100%) on the Budget screen; cost per person in the snapshot; "Split 70 / 30" note on the presentation summary; per-person amounts in presentation full detail only |
| US-065 Help / FAQ | P2 | Help view and its route |
| US-066 First-use tutorial | P2 | Tutorial flow with its own sample content; offered after "Start fresh", replayable from Help |
| US-067 Keyboard shortcuts | P2 | Settings › Keyboard: shortcut list, rebinding, clash handling, reset to defaults |
| FR19 Budget category colours | P2 | User-chosen category colours from a curated per-palette swatch set (D-25) |
