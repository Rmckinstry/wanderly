---
title: POI detail
status: ready for build
stories: [US-004, US-009, US-011, US-057]
last-updated: 2026-10-09
---

# POI detail

Wireframe: POIDetail. No hi-fi (uses the Country detail patterns).

---

Screen:      POI detail
Serves:      US-004, US-009, US-011, US-057 (P1), D-33, D-36
Components:  Button ("← <parent>"), TagChip, Toolbar (`record`), RichTextEditor (`page`), PhotoGrid, PickerPopover ("Move to another place…"), ConfirmDialog, TypeLabel, EmptyState (`inline`)

States:
- default — "← Chiang Mai" Button + ⌘↑ keycap, then the meta line "POI in Chiang Mai · Northern Thailand · Thailand" (each place a link); title (`page-title`); chips: type tags (Temple, Viewpoint), Must-do tag drawn heavier (600 weight, D-36), "+ Tag". No contour lines (D-41). Toolbar: + Photo, + Tip, "Move to another place…" · spacer · quiet "Delete POI". Body: "Description" and "Practical notes" (two RichTextEditor `page` fields, each with a `small` 600 `text-muted` label). Right column: Photos, "In trips" (rows "Northern Thailand Slow Loop · Day 4 →"), Tips.
- new POI — opened from "+ POI": name Field focused in place of the title; everything else visible and empty. Enter commits the name.
- not in any trip — "In trips" shows "Not in any trip yet" (`text-muted`).
- no tips — "No tips for this place yet."

Interactions:
- "Move to another place…" → PickerPopover listing the country's regions and cities (and the country itself) → moves the POI; meta line and breadcrumb update; Toast "Moved to Chiang Rai — Undo".
- "In trips" row → opens the trip builder scrolled to that day with the PoiCard selected.
- Delete POI → if used in trips: ConfirmDialog `delete` "Delete Doi Suthep? It's on Day 4 of Northern Thailand Slow Loop — it will be removed from that day." Danger "Delete POI". Otherwise the same dialog without the trip line.
- Tags: the Must-do category is single-select (choosing "Nice to have" replaces "Must do").

Keyboard:
- Tab: ← parent → meta links → title → chips → + Tag → toolbar → Description → Practical notes → photos → In trips rows → tips.
- ⌘↑ / Alt+↑ → parent place. Delete in a focused photo removes it (Toast + ⌘Z).

Motion: none beyond shared (Toast, popover `duration-slow`).

Edge cases:
- US-004 only the name is required → all other fields optional; a name-only POI is complete.
- US-004 a POI can attach to a country, region or city → "Move…" lists all three levels.
- US-004 duplicate name in the same parent → warning, not a block: under the name Field in `warning` "There's already a 'Night Bazaar' in Chiang Mai — open it" (link); Enter still creates.
- US-004 / US-009 re-link miscategorised POI → "Move to another place…".
- US-009 practical notes are rich text → second RichTextEditor.
- US-009 tags from the fixed set only → tag selector lists fixed values; no free typing (P2 adds management).
- US-011 multiple images, hero by hand only → PhotoGrid rules.
- US-014 POI deleted while referenced → covered by the Delete dialog above.

Handoff notes: meta line `small` `text-muted`; Description and Practical notes stacked with `space-6` between; right column 320px.
