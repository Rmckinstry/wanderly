---
title: Search
status: ready for build
stories: [US-023, US-024, US-026, US-028, US-029]
last-updated: 2026-10-09
---

# Search

One overlay for global, in-record and trip-builder search (SearchOverlay).

Wireframes: SearchOverlay, SearchResults, TripSearch.

---

Screen:      Search overlay (global / in-record / trip-builder)
Serves:      US-023, US-024, US-026, US-028 (P1 tags), US-029 (P1)
Components:  SearchOverlay, TagChip (`filter`), FilterMenu, EmptyState (`search`), Button

States:
- opening — overlay 720px wide, `surface`, `radius-xl`, `shadow-float`, over a scrim; input focused.
- empty query — global: "Recent" (last 5 opened records); trip-builder: recent POIs and places in the trip's countries (US-026).
- scope chip — in a record: "In Thailand ×" before the input; removing it widens to Everything (label changes to "Everything"). Trip builder: "Trip countries: Thailand ×" plus "Search full library" link.
- results — instant as you type, ranked by relevance; each row: name (`ui-strong`), "Type · parent path" (`small` `text-muted`), one-line snippet with the match highlighted (`surface-active` mark), right-aligned action hint on the active row ("↵ Open" / "↵ Add to Day 4"). Footer: key hints + mono result count.
- filters row — "Filter" then FilterMenus: Type, Country (global only — in a record the scope chip is the country), Tag, Trip status (trip status only, D-09). Active filters show as TagChip `filter` chips, each with ×, plus "Clear all".
- no results — EmptyState `search`: "Nothing matches 'canyon hike' with these filters" + "Try fewer words, or remove a filter — each chip clears on its own." + Clear filters + "Search 'canyon' only" (first word).
- trip-builder no match — last row "Create POI 'night' in Chiang Mai — adds it to the library and to Day 4 without leaving the builder".
- loading / error — none (local, instant, NFR1).

Interactions: click a result opens it (closing the overlay); in trip-builder mode click adds to the day and closes; ⌘↵ adds and keeps the overlay open for the next POI.

Keyboard:
- ⌘F / Ctrl+F opens with focus in the input (US-029), from any workspace view including inside notes.
- ↑ / ↓ move through results; ↵ opens or adds; ⌘↵ add-and-keep-searching (trip builder); Tab moves to the filter row, then back to the input; ⌫ in an empty input removes the last filter chip (then the scope chip); Esc closes and returns focus to where it was.

Motion: overlay opens `duration-slow` `ease-out` and closes `duration-base` `ease-in`; instant with reduced motion.

Edge cases:
- US-023 all content types incl. Budgets; type + parent context shown; all text fields searched; case-insensitive; single characters allowed → as above.
- US-023 no results → EmptyState with refine prompt.
- US-023 instant, no loading state → no spinner.
- US-024 multiple, additive filters; clearing one keeps the rest; only existing values listed; empty state suggests broadening; filters reset on a new search session (closing the overlay resets) → as above.
- US-026 scoped to the current record with Recent before typing; trip builder scoped to trip countries with full-library option → scope chip rules.
- US-028 tag filters contextual (a POI tag surfaces only POIs) → choosing a tag from a type-specific category auto-adds that Type chip.
- US-029 Ctrl+F handled by the app everywhere → never opens browser find.

Handoff notes: result rows 52px with snippet, 40px without; max 50 results rendered, with "Show more" at the end; snippet ±40 characters around the match.
