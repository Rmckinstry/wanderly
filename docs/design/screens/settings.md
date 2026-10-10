---
title: Settings
status: ready for build
stories: [US-020, US-022, NFR6]
last-updated: 2026-10-09
---

# Settings

Sections (D-04): Appearance, Budget (thresholds + templates), Backup & restore; Tags shown greyed "later (P2)". Wireframes: SettingsAppearance, SettingsBudget, SettingsBackup, SettingsTemplateNew.

---

## Settings shell

Screen:      Settings (all sections)
Serves:      D-04
Components:  SideNav (Settings item selected), SettingsRow, Button

States: left section list (Appearance / Budget / Backup & restore / Tags — later (P2), not focusable); the section's rows on the right (max 900px); TopBar note "Changes save as you make them".

Keyboard: Tab: section list (Up/Down move, Enter opens) → rows. Every control saves on change.

---

## Appearance

Screen:      Settings › Appearance
Serves:      D-05, D-10, D-17
Components:  SettingsRow (`palette`, `segmented`), SegmentedControl

States:
- Palette — two swatch cards, Paper ("Warm paper, terracotta accent") and Atlas ("Cool grey, deep teal accent"); the chosen card has a 2px `text` outline and "✓ In use".
- Theme — Light / Dark / Match system (default Match system).
- Sidebar — Shown / Hidden, with "or ⌘\ anywhere".
- applying — palette/theme cross-fade `duration-slow`; instant with reduced motion.

Keyboard: palette cards and segmented controls are radio groups (Left/Right).

Edge cases: presentation follows the choice (D-05) → no separate presentation theme.

---

## Budget

Screen:      Settings › Budget
Serves:      US-020, US-022, D-18
Components:  SettingsRow (`percent`), Variance, BudgetTable-style list, Button, ConfirmDialog

States:
- Difference colors — "Close within [10] % of the estimate", "Way over beyond [25] % over the estimate", live example "On a $1,000 line: ↓ $850 Under · ≈ $1,080 Close · ↑ $1,180 Over · ⇈ $1,300 Way over".
- invalid — Way over ≤ Close → Field error "Way over must be higher than Close (10%)"; previous value stays in effect until fixed.
- Budget templates — table: Template · Categories (names, comma-separated) · "Edit · Duplicate · Delete"; "+ New template" + "Or save any budget as a template from its own page."
- no templates — "No templates yet" + "+ New template".

Interactions: threshold changes recolour every budget immediately; Delete template → ConfirmDialog `delete` "Delete 'Full Trip Budget'? Budgets already made from it don't change."

Keyboard: percent fields: Up/Down step by 1.

Edge cases:
- US-020 threshold configurable, default 10% → global (D-18; handoff 5, 9).
- US-022 delete doesn't affect budgets; unique names → as above.

---

## New / edit budget template

Screen:      Settings › Budget › New template
Serves:      US-022, handoff 23
Components:  Button ("← Budget"), Field (name), SegmentedControl (Start from), Field (`select`), Toolbar (`contextual`), DayRow-style list (categories + lines), Button

States:
- new — "← Budget" + ⌘↑; "New budget template"; Name (required, unique); Start from: Blank / Copy of a template (+ template select) / An existing budget (+ trip › budget select); note "Templates hold categories and line names only. A budget made from one starts with this structure and empty amounts — then it is its own copy, free to change." Structure: category boxes (grip, name, "N lines"), each with line rows ("no amount") and "+ Line"; toolbar for the selected category/line: + Category, + Line, Rename, Remove. Footer: Cancel, primary "Save template".
- edit existing — same screen, title is the template name, no Save button: edits autosave (handoff 23).
- invalid — duplicate name → "A template called Full Trip Budget already exists".

Keyboard: Tab: name → start from → structure → footer; in the structure Up/Down move, Enter renames, Alt+Up/Down reorders; ⌘↵ saves a new template; Esc on a new template asks nothing and discards (nothing saved yet).

Edge cases:
- US-022 structure only, amounts empty → "no amount" on every line.
- US-022 later edits don't touch existing budgets → stated in the note.

---

## Backup & restore

Screen:      Settings › Backup & restore
Serves:      NFR6, handoff 11, 21
Components:  SettingsRow (`action`, `notice`), Button, BudgetTable-style list (restore points), ConfirmDialog (`restore`)

States:
- Automatic snapshots — "Last snapshot today 08:02" + "Daily, and before every app update. Keeps the 7 newest daily and 5 newest update snapshots."
- Backup folder set — path (read-only Field) + "Change…"; "Last backup yesterday 21:40 · 1.8 GB" + primary "Back up now".
- no folder — notice "Your library only lives on this computer" + primary "Choose a folder…".
- backing up — "Backing up…" with the button disabled; then "Last backup just now".
- folder unavailable — `warning` "Drive not connected — last backup 9 days ago" + "Change…".
- Restore list — When · From (Daily snapshot / Backup folder / Before update) · Contains (counts; "Same + photos" for folder) · "Restore…"; a point made by a newer version is greyed "Can't restore". Link "Restore from another folder…".
- restore confirm — ConfirmDialog `restore`: "Replace your library with the copy from yesterday 21:40?" / "Anything you've changed since then will be gone from the library. Wanderly saves a snapshot of it first, then restarts." / "Your library was last changed today 22:51." / Cancel · danger "Restore and restart" (wording decided over "Replace library"; update ConfirmDialog spec).

Keyboard: standard; Cancel focused by default in the restore dialog.

Edge cases: NFR6 local snapshots + folder backups, restore in-app, uploads nothing → as above; API errors `BACKUP_FOLDER_UNAVAILABLE` → folder-unavailable state; `INVALID_BACKUP` → greyed row with reason in tooltip.
