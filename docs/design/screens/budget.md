---
title: Budget
status: ready for build
stories: [US-018, US-019, US-020, US-021, US-022]
last-updated: 2026-10-09
---

# Budget

Wireframes: Budget, BudgetEmpty, SettingsTemplateNew (templates, see settings.md). No hi-fi; uses BudgetTable, Variance and BudgetSnapshot from the design system.

---

Screen:      Budget (one trip's budget)
Serves:      US-018, US-019, US-020, US-021, US-022, D-18, D-20, D-28
Components:  Button ("← <trip>"), SegmentedControl (budget switcher when 2+), Menu ("More ▾": Make primary, Rename, Duplicate, Save as template, Delete), BudgetTable, Variance, BudgetSnapshot (`panel`), RichTextEditor (`inline`, budget notes), FormDialog (New budget), ConfirmDialog, EmptyState

States:
- default — "← Northern Thailand Slow Loop" + ⌘↑; title "Budget"; budget switcher when the trip has 2+ budgets ("Full plan · Primary" / "Lean version"); header actions: "+ Category", More ▾. Body: BudgetTable (Item · Budgeted · Actual · Difference; category rows with subtotals; totals row). Right column: BudgetSnapshot `panel` (Shared estimate, Actual so far, Difference with Variance, Per day), budget notes. No travelers field, no per-person figures (D-28).
- empty (new budget) — EmptyState `section` "No categories yet" + "+ Category" and "Start from a template" (Menu of templates).
- some actuals pending — pending lines show "Pending"; subtotals and totals compare only lines with actuals; no "partial" label (D-20).
- Way over line — Difference as the filled `danger-strong` pill "⇈ +$420".
- editing a cell — the cell becomes a number Field; whole dollars and cents, USD.
- error — invalid number → cell keeps the previous value, `danger` outline, message under the table row "Enter an amount like 1,250 or 1250.50".

Interactions:
- New budget (from the trip's budget panel "+ New budget" or More ▾ › New budget) → FormDialog: Name, Start from (Blank / a template). The first budget becomes Primary.
- Make primary → the switcher's "Primary" label moves; presentation and comparison use it.
- Save as template → FormDialog with a unique name; copies categories and line names, never amounts (handoff 23).
- Start from a template on a budget that already has categories → ConfirmDialog `warning` "Replace 5 categories with 'Full Trip Budget'? Amounts you've entered will be removed." (Replace / Cancel).
- Delete category with items → ConfirmDialog `delete` "Delete Stays and its 3 lines?".
- Drag rows by the grip (hover-revealed) to reorder lines within/between categories and to reorder categories.
- Difference cell hover/focus → tooltip with the full sentence ("Over by $260, 36% above budget").

Keyboard:
- BudgetTable grid keys: arrows move, Enter edits, Enter/Tab commits and moves right, Escape cancels; Alt+Up/Down reorders the focused row.
- ⌘↵ in FormDialog creates; ⌘↑ / Alt+↑ back to the trip.

Motion: row insert/remove `duration-base`; none with reduced motion.

Edge cases:
- US-018 multiple budgets / primary / changeable → switcher + Make primary.
- US-018 zero categories valid → empty state never blocks.
- US-018 unique names within a trip → FormDialog error "This trip already has a budget called Lean version".
- US-019 line needs only a name; categories add/rename/reorder/delete; unique category names per budget ("This budget already has Stays"); no limits; USD only → as above.
- US-020 Under / Close / Over (+ Way over, D-18) with glyphs; thresholds global in Settings › Budget; pending ≠ zero; actual without estimate shows "—" → Variance tiers.
- US-021 multiple budgets in the snapshot → the trip builder panel shows the primary with "Primary · 1 of 2"; the Budget screen switcher shows each.
- US-021 per person → removed in v1 (D-28; handoff 18). Per day incomplete when a stop lacks a duration → "Needs every stop's length".
- US-021 partial label → not used (D-20; handoff 6). Exact match is Close.
- US-021 real-time → every edit updates subtotals, totals, snapshot and the trip builder panel immediately.
- US-022 template structure only; edits don't affect existing budgets; deleting a template doesn't affect budgets; unique names; overwrite warning → as above and settings.md.

Handoff notes: table rows 36px; category rows `sidebar` fill; totals row 1px `text` rule above; Difference column right-aligned `num`. Budget notes field is the only rich text on the screen; line notes are plain text (US-007b).
