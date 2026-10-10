---
title: Trips list and trip builder
status: ready for build
stories: [US-005, US-012, US-013, US-014, US-015, US-016, US-016b, US-021, US-058]
last-updated: 2026-10-09
---

# Trips list and trip builder

The trip builder is a hero screen (D-01): the hardest interaction in the app.

Wireframes: TripsList, TripsGrid, TripBlocks, TripMixed, TripSearch, TripNoCountries. Hi-fi: Trip builder (Paper light / dark).

---

## Trips list

Screen:      Trips list (List default, Grid)
Serves:      US-005, US-016, US-027 (trip status only, P1), D-15, D-21, D-34
Components:  SegmentedControl (status filter; List/Grid), FilterMenu (Country, Tag), Menu (sort), TripStatus, TripCard (Grid), Button, EmptyState

States:
- default (List) — "Trips" + "11 trips · 1 pinned"; primary "+ New trip". Controls: status SegmentedControl (All / Sample / Planning / Ready / Completed), Country ▾, Tag ▾, List/Grid, Sort ▾. Table columns: Trip · Status · Countries · Length · Estimate · In windows. Pinned group (drag to reorder), then the sort order.
- default (Grid) — 4-up cards: photo, title (+ pin icon), TripStatus, countries, "14d · $6,840".
- trip with no countries — Countries cell "No countries yet" (`text-muted`); card shows the title's initial on a neutral block.
- Completed — Status reads "Completed · 14 Apr 2025".
- no estimate — "—" in Estimate.
- empty — EmptyState `section` "No trips yet. Start one from a country, or here." + "+ New trip".
- filtered, no match — EmptyState `search` "No Ready trips in Japan — remove a filter" + Clear filters.

Interactions: row/card opens the builder; List/Grid and sort remembered. Status is single-select in the wireframe (one SegmentedControl choice); Country and Tag are multi-select FilterMenus. Different filters combine with AND. US-027 (P1) asks for several statuses at once — see open question B.

Keyboard: as Countries list (Up/Down rows, Enter opens, Alt+Up/Down reorders pinned).

Edge cases:
- US-005 trip names need not be unique → no duplicate check.
- US-027 country and trip statuses filtered independently → only trip status exists (D-09).
- US-016b completion date shown on the trip → in the Status cell.

Handoff notes: Length shows "12+ days" when any block lacks a duration; Estimate is the primary budget's total.

---

## Trip builder

Screen:      Trip builder (location blocks, mixed, day-by-day)
Serves:      US-005, US-012, US-013, US-014, US-015, US-016, US-016b, US-058 (P1), D-27, D-29, D-42, D-44
Components:  TopBar, Toolbar (`contextual`), TagChip, Menu (status), TripStatus, ItineraryBlock (incl. `travel`), DayRow, PoiCard (`day`), RichTextEditor (`inline`), BudgetSnapshot (`panel`), Button, SegmentedControl (none), PickerPopover, SearchOverlay (`trip-builder`), ConfirmDialog, EmptyState, Toast

States:
- default — header: title (`page-title`, click to rename), mono meta "12+ days · 4 stops · 2 travel · departs Day 0 evening", country chips + "+ Country", status Menu button at the right ("● Planning ▾"). Toolbar band. Above the itinerary: "Itinerary · 6 blocks · 1 expanded" with quiet "Expand all ⌥⌘↓" / "Collapse all ⌥⌘↑". Itinerary: an ordered list of ItineraryBlocks (location and travel). Right column (320px): Budget panel (BudgetSnapshot `panel`: "Primary · 1 of 2", "Shared estimate", `money-lg` total, Per day, split bar + legend, "Open full budget →"), then "In Travel Windows".
- nothing selected — Toolbar: + Location block · + Travel block (left), More ▾ (right).
- block selected — Toolbar leads with the chip "Selected · Chiang Mai · 6 days", divider, block actions (Collapse to block / Expand to days, Set duration, Split at day…, Remove block), divider, + Travel block, + Location block. Block gets the 2px `accent` outline.
- day selected — chip "Selected · Day 4", actions: + POI, + Note, Move to…, Remove day.
- POI selected — chip "Selected · Doi Suthep", actions: Remove from day, Move to day…, Open POI. PoiCard gets the `accent` outline.
- travel block selected — chip "Selected · Flight", actions: Set duration, Edit details…, Remove block.
- new trip, no countries (Unlinked, US-005) — "Untitled trip" in `text-muted`; meta "0 days · 0 stops"; "+ Country" chip emphasised (`border-control` solid). EmptyState `section`: "Link a country to start placing stops — location blocks come from a trip's countries. Travel blocks don't need a country." + primary "+ Country", "+ Travel block". "+ Location block" in the toolbar is disabled with tooltip "Link a country first".
- new trip from a country (D-29) — country chip present, no blocks; EmptyState `section` "No stops yet" + "+ Location block", "+ Travel block".
- no budget — panel reads "Budget not yet estimated" + "+ New budget".
- Per day incomplete — "Needs every stop's length" in `text-muted`.
- a block without duration — dashed "Set duration" Button in the block; "Expand to days" disabled with tooltip "Set a duration first".

Interactions:
- + Location block → PickerPopover of places in the linked countries (countries, regions, cities, with counts); picking inserts after the selected block (or at the end) and selects it.
- + Travel block → inserts a travel block (mode Menu: Flight, Train, Bus, Ferry, Drive; From / To free text; duration whole days or 0; notes). If inserted first with 0 days it is Day 0 ("Day 0 · evening"); later 0-day travel shows "overnight" / "same day".
- + Country → PickerPopover of all countries with "New country '…'" at the bottom.
- Expand to days → the block opens into DayRows numbered across the trip ("Days 3–8"); days start empty (US-013). Collapse to block → if any day has content: ConfirmDialog `warning` "Collapse Chiang Mai? Its 6 days keep their notes and POIs, hidden until you expand again." — nothing is deleted (open question C).
- Expand all / Collapse all → applies to every location block with a duration; blocks without duration and travel blocks are skipped; the count updates live.
- Set duration → inline number Field in the block header. Reducing below the number of filled days → ConfirmDialog `warning` "Days 5 and 6 will be removed, with their notes and 2 POIs" (Continue / Cancel).
- Split at day… → Menu of the block's days; the block becomes two blocks of the same place, each keeping its days.
- Drag (grip): blocks reorder within the itinerary; days move within or between expanded blocks; PoiCards reorder within a day and move between days. A 2px `accent` drop line shows the target. The block's duration always equals its day count.
- + POI on a day → SearchOverlay `trip-builder` (search.md), scoped to the trip's countries, with "Search full library" and "Create POI '…' in Chiang Mai".
- + Note → an inline RichTextEditor on the day, focused.
- Status Menu → Sample / Planning / Ready / Completed (any order). Choosing Completed reveals a completion date Field beside the status, defaulting to today.
- Remove block → no confirm when the block is collapsed and has no day content (Toast "Removed Chiang Rai — Undo"); ConfirmDialog `delete` when it has day content ("Remove Chiang Mai and its 6 days? 7 POIs are removed from this trip, not from your library.").
- Remove country chip → disabled while any block uses it (tooltip "Used by 3 blocks — move or remove them first").
- "Open full budget →" → Budget screen (budget.md).

Keyboard:
- Tab: header (title, chips, status) → toolbar → Expand all / Collapse all → itinerary → right column.
- In the itinerary: Up/Down move selection between blocks (and into days/POIs of an expanded block); Enter on a block header toggles expand/collapse; Space on a grip picks up, Up/Down move, Space drops, Escape cancels (keyboard drag).
- Delete / Backspace on a selected POI removes it from the day (Toast + ⌘Z); on a selected block runs Remove block (with its confirm rules).
- ⌥⌘↓ / Ctrl+Alt+↓ Expand all; ⌥⌘↑ / Ctrl+Alt+↑ Collapse all.
- ⌘F → trip-scoped search; from a selected day, + POI search adds to that day.
- Escape clears the selection (Toolbar returns to the nothing-selected state); focus stays on the item.

Motion: expand/collapse `duration-base`; drag lift uses `shadow-float`; drop settles `duration-base` `ease-out`; all instant with reduced motion (drag still works, no lift animation).

Edge cases:
- US-005 multiple countries → several country chips.
- US-005 no country at creation → Unlinked state above; can be linked later via + Country.
- US-012 reorder blocks → drag or keyboard drag.
- US-012 block at any level, mixed levels → PickerPopover lists all levels together, with their path in `text-muted`.
- US-012 duration optional → "Set duration" shown; trip length shows "12+ days".
- US-012 refine a block over time → select block → "Change place…" lives in More ▾ of the block (keeps days if the new place is in the same country).
- US-012 deleting a block never deletes the place → Remove copy says "not from your library".
- US-012 country stays linked while used → chip × disabled with tooltip.
- US-012 region/city used by a block deleted → block widens to the parent with a one-time `text-muted` line "Was Chiang Mai — now Thailand".
- US-012 same place twice / single block → allowed, no warnings.
- US-013 one block at a time / mixed state → each block expands independently; Expand all is a convenience only.
- US-013 no duration → Expand disabled with "Set a duration first".
- US-013 days start empty → DayRow EmptyState `inline` "Empty day — add a POI or a note".
- US-013 reduce duration after expanding → warning dialog above.
- US-013 split / move days between blocks → Split at day…; drag days.
- US-013 collapse with content → warning dialog (content kept).
- US-014 scope + expand to full library → SearchOverlay `trip-builder`.
- US-014 same POI on several days / reorder within a day / zero POIs → allowed; drag; empty-day line.
- US-015 notes and POIs coexist, notes-only days, rich text → DayRow.
- US-016 any order, back to earlier status, no content changes → status Menu, no confirm.
- US-016b date default today, any date, retained when status moves back, shown in Travel Windows → date Field rules; hidden (not deleted) when not Completed.
- US-021 per person → not shown in v1 (D-28, handoff 18).
- Travel blocks (D-27) → never expand, take no POIs, never create budget lines; count toward length and per-day cost.

Handoff notes:
- Itinerary column max 780px; right column 320px; gutter `space-14`.
- Block header 48px min-height, `space-3` padding; travel blocks `sidebar` fill with dashed `border-control`; mode TypeLabel first.
- Selected block: 2px `accent` outline (not a fill). Selected PoiCard: 1px `accent` border + 1px `accent` ring.
- PoiCards inside a block use `radius-sm`; blocks `radius-lg` (1:2).
- Day number column 64px; DayRow padding `space-3`.
- Grip: drawn 6-dot icon, 20×24 hit area, `text-muted`.
- Expanded state per trip is remembered (handoff 14).
