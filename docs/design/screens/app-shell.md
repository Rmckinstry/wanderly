---
title: App shell — side nav, top bar, toolbar, global keys
status: ready for build
stories: [US-025, US-029, US-007]
last-updated: 2026-10-09
---

# App shell

Everything outside a screen's own content: the side nav, top bar, toolbar band, back-to-parent, autosave, and the keys that work everywhere. Every workspace screen in this folder sits inside this shell; presentation mode (presentation.md) and the recovery / first-run screens (recovery-and-first-run.md) do not.

Wireframes: every batch 1–3 board. Hi-fi: Country detail, Trip builder (both themes).

---

Screen:      Shell (applies to every workspace screen)
Serves:      US-025 (browse by country), US-029 (⌘F), US-007 (no silent data loss)
Components:  SideNav, TopBar, Toolbar, Button, Field (`search-trigger`), Toast

States:
- default — nav expanded (248px), top bar 52px, content padded `space-12` left/right.
- nav collapsed — nav fully hidden (D-17); a 24×48px edge button at the window's left edge, vertically centred, `surface` with `border` and the collapse icon, accessible name "Show sidebar (⌘\)". Content takes the full width; notes keep their 720px max line length, so the extra room goes to margins, not line length.
- autosave — TopBar shows "Saved" (`text-muted`); "Saving…" only if a save takes over 300ms; on failure "Couldn't save — Retry" (`danger`, quiet Button). Content stays on screen; edits keep queuing locally until a save succeeds.
- backup notice — when no backup folder is set or the last backup is over 7 days old, the nav's bottom backup line becomes a `chip` notice ("Last backup 9 days ago — back up now") that opens Settings › Backup & restore.
- loading — none. Local SQLite answers in well under 100ms (NFR1); screens render complete, never skeletons. If a query ever exceeds 300ms, the content region shows nothing new until ready — no spinner flashes.
- error — a screen whose record no longer exists (deleted in another window): EmptyState `section` "This country was deleted" with a Button back to its parent list.

Interactions:
- Nav section header = two targets: the chevron expands/collapses the section in place (`duration-base`); the label opens the section's full list page.
- Expanded sections list Pinned (with a pin icon on each item, drag to reorder) then Newest first, then "All N →". Travel Windows list chronologically by target date; Tips list "All tips", "General", "Unlinked N".
- Selected nav item: `surface` fill, 1px `border-control` outline, label in `accent` at 600 (D-45).
- Clicking a pin icon unpins (no confirm; Toast with Undo). Pinning is from a record's "More ▾" menu: "Pin to sidebar".
- Breadcrumb ancestors are links; the current record is plain `text`.
- Toolbar band (D-42): full-width `surface` band under the page header, 1px `border` above, 1px `border-control` below; outlined buttons on `bg`; add "+" icons in `accent`; sticks to the top of the scroll area once the header scrolls away.
- Back to parent (D-33): every child page (region, city, POI, tip, template) shows a secondary Button "← <parent name>" above its title, with a ⌘↑ keycap beside it. It always goes up one level.

Keyboard (global; work on every workspace screen):
- ⌘N / Ctrl+N — New: opens a Menu "Country · Trip · Travel Window · Tip" (Tip is P1). Enter on an item starts that creation flow.
- ⌘F / Ctrl+F — search, scoped to the current record (see search.md). Never the browser's find.
- ⌘\ / Ctrl+\ — show/hide the side nav; remembered between sessions.
- ⌘↑ / Alt+↑ — up one level (child pages only). Disabled while focus is in a text field, where ⌘↑ keeps its text meaning (jump to start).
- ⌘[ / Alt+← and the mouse back button — history back; ⌘] / Alt+→ forward.
- ⌘Z — undo the last removal while its Toast shows.
- Tab order on every screen: skip link "Skip to content" (first Tab, visible on focus) → side nav → top bar (breadcrumb links, search trigger) → page header (title, chips) → toolbar → main content → right column.
- Side nav internal keys: Up/Down between items, Right expands a section or moves into it, Left collapses or moves to its header, Enter opens; Alt+Up/Down reorders a focused pinned item.
- Focus: every control shows the 2px `focus` ring (accent) with 2px offset on keyboard focus only (`:focus-visible`), never on mouse click.

Motion:
- Nav collapse/expand: width/translate over `duration-base`, `ease-out` in / `ease-in` out.
- Section expand/collapse: `duration-base`.
- Toolbar band becoming sticky: no animation.
- Theme or palette change: `duration-slow` cross-fade.
- With `prefers-reduced-motion: reduce`, every one of these is instant.

Edge cases:
- US-007 "navigate away mid-edit … no silent data loss" → autosave on every change (debounced ~500ms) and on blur; navigation never prompts. A failed save keeps the edit and shows "Couldn't save — Retry"; navigating away while a save is failing shows ConfirmDialog `warning` "1 change hasn't saved yet — Stay / Leave anyway".
- US-025 "collapsing and expanding hierarchy levels" → nav sections and ContentsIndex regions both collapse; state remembered per section.
- US-029 "Ctrl+F handled by the app" → the app intercepts ⌘F/Ctrl+F in every workspace view, including inside notes fields.
- Window narrower than 1280px → not supported; the window has a 1280px minimum width.

Handoff notes:
- Nav: 248px wide, `sidebar` fill, `border` right rule, `space-4` top padding. Item rows: header 32px min-height, item 28px; label inset 24px from the nav's inner edge for items.
- Top bar: 52px, `space-12` side padding, `border` bottom rule.
- Content max width: none; notes columns cap at 720px; right column 320px; gutter `space-14`.
- Remembered UI state (handoff item 14): palette, theme, nav collapsed, each section's expanded state, list/grid per list, Travel Window view per window, expanded blocks per trip.
