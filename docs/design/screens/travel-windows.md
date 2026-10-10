---
title: Travel Windows
status: ready for build
stories: [US-030, US-031, US-032, US-032b, US-033, US-034, US-034b, US-035, US-036]
last-updated: 2026-10-09
---

# Travel Windows

Wireframes: TWList, TWDetail, TWPast. No hi-fi (shares Trip builder / presentation patterns).

---

## Travel Windows list

Screen:      Travel Windows list
Serves:      US-030, US-035, US-036 (P1)
Components:  Button, FormDialog (New window), TripStatus, EmptyState

States:
- default — "Travel Windows" + one-line definition "A window is a planned travel period with 2–3 trips to choose between"; "Archived (4)" quiet Button; primary "+ New window". Sections "Upcoming · soonest first" and "Past · most recent first". Row: name (`ui-strong`) + target date (mono; or its label), up to 3 trip thumbnails + trip names, the choice ("Chosen: Kyoto & Kansai" / "Not decided yet" / past: "Went: Kyoto Spring · 14 Apr 2025"), → .
- no windows — EmptyState `section` "No travel windows yet. Make one when you know roughly when you're going." + "+ New window".
- archived view — same rows, header "Archived", each with "Restore" quiet Button.

Interactions: row opens the window; "+ New window" → FormDialog: Name (required, unique), Target date (date Field, optional) + optional label ("November 2027"); "Create window". Archive/Restore/Delete live in the window's More ▾.

Keyboard: Up/Down between rows, Enter opens; ⌘↵ submits the FormDialog.

Edge cases:
- US-030 name required, date recommended not required → only Name blocks; a window with no date sorts last in Upcoming with "No date yet".
- US-030 label shown instead of the date, ordering always by date → as described.
- US-030 duplicate name → "A window called November 2027 already exists — open it".
- US-030 no trips yet → row shows "No trips yet".
- US-035 chronological, never auto-deleted → Past section persists; copy under the list: "Windows are never deleted automatically. Archive old ones to tidy this list."
- US-036 archive/restore/delete are separate actions; delete confirms → More ▾ in the window.

---

## Travel Window detail (upcoming)

Screen:      Travel Window — Side by side (default with 2+ trips) and One at a time
Serves:      US-031, US-032, US-032b, US-033, US-034, D-30, D-31, D-32
Components:  Button ("Present ⌘↵", "Add trip"), Menu ("More ▾": Rename, Change date, Archive, Delete…), SegmentedControl (One at a time / Side by side), ComparisonColumn (in-app variant), TripStatus, TagChip, PickerPopover (Add trip), ConfirmDialog, Banner, EmptyState

States:
- side by side (D-30 default with 2+ trips) — header: name, mono target date, "3 trips shortlisted", "No choice yet" / "Chosen: …"; actions "+ Add trip", More ▾, primary "Present ⌘↵". Columns (equal width, max 3): hero photo, title, TripStatus, concept (2 lines), route text + days, estimate + per day (or "Budget not yet estimated"), tags (or "No tags yet"), then "Mark as chosen", "Open →", quiet "Remove". Chosen column: `accent` top rule + "Chosen" label; others unchanged, never dimmed.
- one at a time — tabs per trip (chosen tab shows "✓ Chosen"); the trip's read view: title, status, concept, route strip, blocks with day and POI counts, budget panel, photos; "Edit trip" Button (US-032b) for live trips. ← → switch trips.
- one trip — opens one at a time; the SegmentedControl's "Side by side" option is hidden (D-02).
- no trips — EmptyState `section` "No trips in this window yet" + "+ Add trip".
- four or more — adding a 4th shows ConfirmDialog `warning` "Windows work best with 2–3 options. Add a 4th anyway?" (Add anyway / Cancel). Side by side then scrolls horizontally with 3 visible.

Interactions:
- Add trip → PickerPopover of all trips (status shown), excluding ones already here.
- Mark as chosen (D-32, app only) → sets the chosen flag; button becomes "Chosen ✓ · Undo choice" on that column. Choosing another trip moves the flag.
- Open → / Edit trip → the trip builder with a "← November 2027" back Button (US-032b); returning lands on the same trip and view.
- Remove → removes from this window only; Toast "Removed from November 2027 — Undo".
- Present ⌘↵ → presentation.md, starting on the first trip.
- Drag a column by its header to reorder (order used in presentation).

Keyboard: ⌘↵ presents; in one-at-a-time ← → change trips (when focus is not in a field); Tab order: header actions → view switch → columns (each: title link, Mark as chosen, Open, Remove).

Motion: switching views cross-fades `duration-base`.

Edge cases:
- US-031 referenced not copied; duplicates blocked; a trip in several windows → PickerPopover hides trips already in this window; no copy is made.
- US-031 deleting a shortlisted trip elsewhere warns → the trip's Delete dialog lists "In 1 travel window: November 2027".
- US-032 read-only by default → one-at-a-time view has no inline editing; "Edit trip" opens the builder.
- US-032 single trip has no prev/next → arrows hidden.
- US-032 / US-033 no photos / no budget → neutral photo block; "Budget not yet estimated".
- US-033 comparison read-only → no edit controls in columns other than choose/open/remove.
- US-033 one trip "accessible with a note" → replaced by D-02 (handoff 3): no comparison with one trip.
- US-034 choosing doesn't change status, others stay visible and editable, no selection is valid, choice changeable → as above.

---

## Travel Window detail (past) and preserved records

Screen:      Travel Window — past, with a preserved historical record
Serves:      US-034b, US-035
Components:  as above + Banner

States:
- past window — header says "Past"; Present still available (secondary, not primary).
- live Completed trip — TripStatus "Completed · 14 Apr 2025", fully editable via Edit trip.
- preserved record (Completed trip later deleted) — Banner "HISTORICAL RECORD — This trip was deleted from Trips on 3 Mar 2026. What you see is how it was on that day — read-only, and it stays here as long as this window exists." No Edit button, no links into Countries or POIs.
- no Completed trips → trips show their current status with no badge.

Edge cases:
- US-034b read-only, clear indicator, kept data includes itinerary/POIs/budgets/tips/photos → Banner + full read view.
- US-034b deleting a Completed trip in a window → delete dialog adds "It will stay in November 2027 as a read-only historical record."
- US-034b non-Completed trips are removed without preservation → delete dialog says "It will be removed from 1 travel window."
- US-034b last window containing a preserved record deleted → ConfirmDialog `delete` "Delete April 2025? Kyoto Spring's historical record lives only here and will be deleted too."
- US-035 Completed trips editable in past windows, date customisable → Edit trip, date in the builder.

Handoff notes: columns `space-6` gap; column card `surface`, `radius-xl`, `space-4` padding; the in-app comparison titles use the `section` style (21/600 sans) — the serif `trip-title-sm` is presentation-only (D-12).
