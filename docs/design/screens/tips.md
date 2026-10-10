---
title: Tips and lessons learned (P1)
status: ready for build (P1)
stories: [US-006, US-010, US-050, US-051, US-052, US-053, US-054, US-055]
last-updated: 2026-10-09
---

# Tips and lessons learned (P1)

Wireframes: TipsList, TipDetail, TipNew. Decisions D-37 – D-40; handoff item 24.

---

## Tips list

Screen:      Tips list
Serves:      US-053, US-054, D-37, D-38, D-39
Components:  Field (keyword), SegmentedControl (All types / Tips / Lessons learned; Group by destination / None), FilterMenu (Place, From trip, Tag, Has follow-up), TagChip (`filter`), Menu (sort), TypeLabel, EmptyState

States:
- default — "Tips" + "48 · 35 tips · 13 lessons learned"; primary "+ New tip". Row 1: keyword Field ("Filter tips by keyword…"), type SegmentedControl, Group, Sort. Row 2: "Filter" + FilterMenus + active chips + "Clear all" + mono "7 of 48". List grouped by destination (country headings with counts; General last), newest first. Row: TypeLabel (TIP outlined / LESSON filled), title + one-line excerpt, place path, mono date. A follow-up lesson sits indented under its tip, joined by a 2px `border-control` rule, labelled "Follow-up:" with its source trip.
- Place menu — hierarchical checkboxes (country › region › city) with counts, then "General (no place)" and "Unlinked"; a country includes its children.
- Unlinked — no banner on the list (D-37); visible only via Place › Unlinked or the sidebar's "Unlinked N". Each unlinked row shows "Used to be in Krabi" and a "Pick a place" Button.
- empty (none at all) — EmptyState `section` "No tips yet. Write down the things you'd tell a friend." + "+ New tip".
- General empty — "No general tips yet" + "+ New tip" (scope preset General).
- filtered, no match — EmptyState `search` + Clear filters.

Interactions: row opens Tip detail; "Pick a place" opens PickerPopover; filters AND across dimensions.

Keyboard: Up/Down rows, Enter opens; Tab order: keyword → type → filters → group/sort → list.

Edge cases:
- US-053 tips on a child surface on the parent with context → a Chiang Mai tip appears under Thailand with "Chiang Mai · Northern Thailand".
- US-053 separated by type → one list with TypeLabels and a type filter (D-38; handoff 24).
- US-053 / US-054 sort and filter by type, date, level, keyword → as above.
- US-053 General tips not in destination views → Country/region/city Tips panels never list General tips.
- US-054 no General tips → empty state.

---

## Tip detail

Screen:      Tip / lesson detail
Serves:      US-010, US-055, D-33, D-40
Components:  Button ("← Tips"), TypeLabel, Toolbar (`record`), Menu (Type), PickerPopover (Scope & place), RichTextEditor (`page`), TagChip, ConfirmDialog, Banner

States:
- tip — "← Tips" + ⌘↑; TypeLabel + "Destination · Pai · Northern Thailand · Thailand" (or "General"); title; "Added 2 Sep 2025 · before the trip". Toolbar: "+ Lesson learned follow-up", "Type ▾", "Scope & place…", quiet "Delete tip". Body: content (RichTextEditor), then "What actually happened · N follow-ups": each follow-up is a `surface` block with a 1px `border` and `radius-lg` (not a left-rule card), holding a LESSON TypeLabel, title, date, text, and "From Northern Thailand Slow Loop · Completed 9 Dec 2025". Right column: "Shown in" (place path + "Tips on a city also appear on its region and country"), Tags.
- lesson — same, with "From a completed trip" field (optional) and, if it follows up a tip, "Follows up: <tip title>" link.
- unlinked — Banner "UNLINKED — its place (Krabi) was deleted. Pick a new place to show it there again." + "Pick a place".

Interactions:
- + Lesson learned follow-up → FormDialog preset: Type Lesson, scope and place inherited from the tip (US-055), "From a completed trip" picker (Completed trips only). Disabled on an unlinked tip with tooltip "Pick a place first" (US-055).
- Scope & place… → PickerPopover; switching Destination → General shows ConfirmDialog `warning` "Make this tip general? It will no longer show on Pai." (US-010).
- Delete tip → ConfirmDialog `delete`; if it has follow-ups: "Its 1 follow-up lesson is kept and will stand on its own."

Keyboard: ⌘↑ → Tips list; ⌘↵ in FormDialog creates.

Edge cases:
- US-010 edits save immediately → autosave.
- US-010 Destination → General warns → as above.
- US-010 lesson linked to a Completed trip retroactively → "From a completed trip" editable any time.
- US-010 / US-051 linked place deleted → Unlinked state, shows former place.
- US-055 original tip never changed; several follow-ups; follow-up inherits scope; survives tip deletion; unlinked tip needs a place first → as above.

Follow-up block styling note: the wireframe used a left-rule block; the spec uses a bordered `surface` block with `radius-lg` instead, to stay clear of the left-border-card trope (critique checklist). Confirm in review.

---

## New tip

Screen:      New tip / lesson (FormDialog)
Serves:      US-006, US-050, US-051, US-052, D-40
Components:  FormDialog, Field (text, select), SegmentedControl (Type; Scope), PickerPopover (Place), RichTextEditor (`inline`)

States:
- default — Title (required); Type: Tip / Lesson learned; Scope: General / A place; Place (shown only for "A place"; pre-filled when opened from a place, with "filled in because you started here · change ▾"); "From a completed trip · optional" (Lessons only; lists Completed trips only); Details (optional). Primary "Add tip" / "Add lesson".
- missing place for "A place" → Field error "Pick a place, or make it General".

Keyboard: focus starts in Title; ⌘↵ adds; Esc cancels (nothing saved).

Edge cases:
- US-006 / US-050 only a title required → as above.
- US-006 lesson linked only to Completed trips → picker lists Completed only.
- US-051 any level, pre-populated from a record → as above.
- US-006 vs US-051 trip as a tip's place → not allowed (D-40; handoff 24).
- No automatic tags → Tags empty on create.
