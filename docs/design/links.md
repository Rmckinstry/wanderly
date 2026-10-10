---
title: Design links
status: current
last-updated: 2026-10-09
---

# Design links

All artifacts are private claude.ai pages — they open only for the owner until shared from each page's Share menu.

| Artifact | What it is | Status |
| --- | --- | --- |
| [Wanderly — design system](https://claude.ai/artifact/Sr2Lwo3bAhuugsrSFENRLN) | Tokens (4 themes), type, 23 component cards with previews, decision log D-01–D-45, open items | Source of truth for tokens and components; `tokens.json` and `components.md` here are copies |
| [Wanderly Hi-fi](https://claude.ai/artifact/M82d52APZzWsNw3t4RY8CL) | Hero screens: Country detail, Trip builder, Presentation — Paper light/dark, Atlas light. Theme switchable per board in Tweaks | Approved after critique round 1 (D-45) |
| [Wanderly Wireframes](https://claude.ai/artifact/R3oHahTJLhPsGUTGURZKPR) | 33 grayscale screens in 4 batches plus a flow map: countries, places, trip builder, budget, Travel Windows, presentation, search, POI, trips, settings, recovery, first run, tips | Structure reference; hi-fi wins where they differ |
| [Wanderly Directions](https://claude.ai/artifact/QCNkcXZBiC7qNUk5sBv84B) | Phase 2 explorations A–C and F (D, E removed) | Superseded by Direction F (D-16); history only |

## Which board answers which spec

| Spec file | Wireframe boards | Hi-fi boards |
| --- | --- | --- |
| screens/app-shell.md | all | Main, Trip (and dark) |
| screens/countries-and-places.md | CountriesList, CountriesGrid, CountryDetail, CountryStub, RegionDelete | Main, CountryDark |
| screens/poi.md | POIDetail | — |
| screens/trips-and-trip-builder.md | TripsList, TripsGrid, TripBlocks, TripMixed, TripSearch, TripNoCountries | Trip, TripDark |
| screens/budget.md | Budget, BudgetEmpty | — |
| screens/travel-windows.md | TWList, TWDetail, TWPast | — |
| screens/presentation.md | PresSummary, PresDetail, PresCompare, PresSingle | Present, PresentDark, PresentAtlas |
| screens/search.md | SearchOverlay, SearchResults, TripSearch | — |
| screens/tips.md (P1) | TipsList, TipDetail, TipNew | — |
| screens/settings.md | SettingsAppearance, SettingsBudget, SettingsBackup, SettingsTemplateNew | — |
| screens/recovery-and-first-run.md | Recovery, FirstRun, FirstRunRestore | — |

Screens without hi-fi are built from the hi-fi patterns plus the tokens and component specs; mockups are reference only, not production code.
