---
title: Components
status: locked (v1) + Phase 7 additions
last-updated: 2026-10-09
---

# Components

Every component a screen spec names is defined here. Tokens are referenced by name from tokens.json. Visual references live in the Wanderly design system artifact (see links.md); previews are reference only, not production code.

## Button

Labeled actions; every button says what it does in words, never icon-only.

**Variants**
- `primary` — the one main action on a screen (Start a trip, Next). `accent` fill, `on-accent` text. At most one per screen region.
- `secondary` — default. `surface` fill, `border` outline, `text`.
- `quiet` — no fill or outline until hover (`surface-active`). For low-weight actions in dense rows (+ Add in a panel heading).
- `danger` — destructive confirmations only (Delete in a confirm dialog). `danger` fill, `on-accent` text. Never on a screen's toolbar.
- Sizes: default 32px tall (`space-3` side padding); `lg` 44px (`space-4`) in presentation, where it is clicked from a distance.

**States** — default, hover (`surface-active` fill; primary darkens slightly), active (pressed), focus (2px `focus` ring, 2px offset), disabled (45% opacity, `not-allowed`), loading (spinner replaces nothing; label stays, button disabled).

**Add icon** — "Add" buttons and chips (Region, City, POI, Tip, Tag, New, Filter) lead with a drawn 10px plus icon (1.5px strokes, `currentColor`), never a typed "+" character, which sits above the text's center. Icon and label are centered by flexbox with a `space-2` gap (`space-1` in chips). The accessible name is "Add region", not "+ Region".

**Shortcut hint** — a button may end with its key as a small keycap: a 20px-tall box (`radius-sm`, `surface-active` fill), the key in mono 12px (`micro` floor), `text-muted`. On primary/danger buttons the keycap has no fill: a 1px outline and text in the button's own text color (`on-accent`), 5.8:1 or better in every theme (D-45 — the old translucent fill measured 4.05:1). The keycap and label are centered on the same vertical axis: the button is `inline-flex`, `align-items: center`, `line-height: 1`, and the keycap is a fixed-height flex box, so mono and sans metrics never shift it off-center.

**Tokens** — accent, on-accent, surface, surface-active, border, text, text-muted, danger, focus, radius-md, space-2, space-3, space-4, duration-fast, ease-out.

**Usage** — Verb first ("Start a trip", "Add photos"). No "OK"/"Submit". Don't put two primary buttons side by side.

**Keyboard** — Tab focuses, Enter or Space activates. Shortcut hints are real: the key works anywhere on that screen.

## Toolbar

A row of labeled actions for the current record, which changes with what is selected.

**Variants**
- `record` — top of a country, region or city: + Region, + City, + POI, + Tip, then the primary action at the right (Start a trip).
- `contextual` — trip builder: shows the selection's name as a lead label, then its actions. Nothing selected: + Location block, Status. Block selected: Expand to days, Set duration, Split, Remove. Day selected: + POI, + Note, Move to…, Remove. POI on a day selected: Remove from day, Move to day…, Open POI.

**Look** (D-42) — a full-width band across the content area: `surface` fill, 1px `border` above and a 1px `border-control` rule below, so the page clearly divides into header / toolbar / content. Buttons inside are outlined (`border-control`) on `bg`, weight 500, so they stand off the band; the add "+" icons take `accent`; the primary keeps its accent fill. In `contextual`, the selection leads as a chip on `surface-active`: "Selected" in `accent` at 12px/600, then the name in `ui-strong` and its length in `num-sm`. A 1px `border` divider separates the selection's actions from the add actions at the right. The band sticks to the top of the scroll area.

**States** — buttons follow Button states. An action that does not apply is hidden rather than shown disabled, except when its absence would confuse (Expand to days with no duration is shown disabled with help text "Set a duration first").

**Tokens** — surface, surface-active, border, accent ("Selected" label), text, text-muted, radius-xl, radius-md, space-2, space-3; buttons per Button.

**Usage** — Labels only, no icon-only buttons (decision D-07). At most one primary per toolbar, always last. Wraps rather than overflowing into a "more" menu at the 1280px minimum.

**Keyboard** — Tab enters the toolbar on its first button; Left/Right move between buttons; Tab leaves. Changing selection never steals focus into the toolbar.

## TagChip

A tag applied to a record, shown as a soft chip; the dashed "+ Tag" chip opens the tag selector.

**Variants**
- `tag` — `chip` fill, `small` text. Shows the value; the category appears only where ambiguous ("Language: moderate").
- `editable` — same, with a × remove button revealed on hover and focus.
- `add` — dashed `border-control` outline, `text-muted`, label "+ Tag". Always last in the row.
- `filter` — in search filters: same chip, with a × to clear that one filter.

**States** — default, hover (× appears), focus (ring), empty (only the + Tag chip shows; no "No tags" text).

**Tokens** — chip, text, text-muted, border-control, radius-md, space-1, focus.

**Usage** — Chips mean tags and filters, nothing else (decision: tags are chips, prose meta is plain text). Never use chips for actions or status. Tags are always applied by hand (global rule).

**Keyboard** — Each chip is focusable; Backspace or Delete removes a focused editable chip; Enter on + Tag opens the selector, which closes with Escape and returns focus to + Tag.

## TripStatus

A trip's manual lifecycle status, shown as a dot plus its word; the word always carries the meaning.

**Variants** (one per status; countries have no status — decision D-09)
- `sample` — hollow dot, `text-muted` outline.
- `planning` — `info` dot.
- `ready` — `success` dot.
- `completed` — `text` dot, plus the completion date ("Completed · 14 Nov 2027").

**States** — display only in cards and lists. As a control (trip header), it is a secondary Button that opens a menu of the four statuses; any status can be chosen in any order.

**Tokens** — info, success, text, text-muted, radius-full, space-1.

**Usage** — Always the dot and the word together; never the dot alone, never color-coded backgrounds. Status changes never alter content (global rule), so no confirmation is needed.

**Keyboard** — As a control: Enter opens the menu, Up/Down move, Enter selects, Escape closes and returns focus.

## Variance

The budget difference itself, colored by tier, with a glyph so the meaning never rests on color alone; there is no separate status label (decision D-18).

**Tiers** (Difference = Actual − Budgeted, as a % of Budgeted; thresholds set in Settings › Budget)
- `under` — actual is more than the Close % below budget. `success` text, glyph ↓, e.g. "↓ −$60".
- `close` — within ± the Close % (default 10%); an exact match is Close. `warning` text, glyph ≈.
- `over` — above budget by more than the Close % and up to the Way-over % (default 25%). `danger` text, glyph ↑.
- `way-over` — above budget by more than the Way-over %. A filled `danger-strong` pill with `on-danger-strong` text, glyph ⇈, weight 600. Differs from Over by shape and lightness, not just a deeper hue.
- `pending` — no actual yet. "Pending" in `text-muted`, never "$0".
- `no-estimate` — actual but no budgeted amount: actual shown, difference "—", no tier.

**Subtotals with missing actuals** — a category or summary is computed from the items that have actuals; the items still waiting show "Pending" on their own lines, so no extra "partial" label is added (D-20).

**Accessible name** — every cell reads as a sentence, e.g. "Over by $260, 36% above budget". Hover or focus shows the same as a tooltip.

**Tokens** — success, warning, danger, danger-strong, on-danger-strong, text-muted, radius-md, space-1; figures in `num` mono.

**Usage** — Used in the Difference column of BudgetTable (lines, category subtotals, totals), in BudgetSnapshot, and in presentation once actuals exist. Never tint a whole row.

**Keyboard** — Not interactive itself; read with its cell in grid navigation.

## Field

Text input, number, select and date fields, plus the search trigger, all with visible labels.

**Variants**
- `text` — name fields, short notes on budget line items (plain text).
- `number` — duration (days), travelers, money (USD, shown in `num` mono, right-aligned in tables).
- `select` — native-feeling dropdown (scope, type, template). The platform arrow is hidden; a drawn chevron (7px, 1.5px stroke, `text-muted`) sits `space-3` from the right edge, matching the left text inset, and the text stops 36px from the right so it never runs under the chevron. The native option list still opens, so Windows and macOS keyboard behavior is unchanged.
- `date` — completion date, travel window target date, with an optional label field beside it ("November 2027").
- `search-trigger` — looks like a field, is a button: "Search in Thailand ⌘F". Opens SearchOverlay.

**States** — default (`border-control` outline on `surface`), hover, focus (ring), filled, disabled, error (`danger` outline + message below in `danger`, e.g. "A country named Thailand already exists — open it"), read-only (no outline, plain text).

**Tokens** — surface, border-control, text, text-muted, danger, focus, radius-md, space-3, space-2.

**Usage** — Label above, never placeholder-as-label. Only names are required (global rule), so required marks appear only on name fields. Errors name the fix. Autosave applies; there is no Save button.

**Keyboard** — Tab between fields; Enter in a single-line name field commits and moves on; Escape in a new-record name field cancels creation with nothing saved (US-001).

## SideNav

The app's main navigation: four expandable sections, fully collapsible.

**Structure** — Wordmark and collapse button · + New (⌘N) · Countries · Trips · Travel Windows · Tips · (bottom) Search · Settings · backup line ("Backed up · 2 h ago").

**Variants**
- `expanded` (236px, `sidebar` fill, `border` right rule).
- `collapsed` — fully hidden (decision D-17b). A small edge button at the window's left edge and ⌘\ (Ctrl+\) bring it back. The state is remembered between sessions. Presentation mode always hides it.

**Sections** — each header is two controls: a chevron that expands the section in place, and the label that opens the section's full list page. Expanded lists:
- Countries and Trips — "Pinned" (manually reorderable by drag) then "Newest first" (by creation date), then "All N →" (decision D-15). Every pinned item carries a small pin icon (11px, `text-muted`, right-aligned) in addition to the group label, so a pinned item is recognisable on its own; clicking the icon unpins.
- Travel Windows — chronological by target date (US-030); archived windows only via the full page.
- Tips — newest first.

**States** — item default, hover (`surface-active`), current page (`surface` fill, 1px `border-control` outline, label in `accent` at 600 — D-45; `surface-active` alone was 1.10:1 against the sidebar), focus (ring), section expanded/collapsed (chevron rotates, `duration-base`), backup stale (backup line turns into a `chip` notice "Last backup 9 days ago — back up now").

**Tokens** — sidebar, surface-active, border, text, text-muted, chip, accent (focus), radius-md, space-2, space-5, duration-base, ease-out; counts in `num-sm` mono.

**Usage** — No icons in the expanded nav: labels only. Counts are the only extra data. Never more than one level of nesting.

**Keyboard** — ⌘\ toggles the nav. Within it: Up/Down move between items, Right expands a section (or moves into it), Left collapses (or moves to the header), Enter opens. Pinned items: Alt+Up/Down reorders.

## TopBar

The 52px bar above every workspace screen: breadcrumb on the left, autosave state and the search trigger on the right.

**Parts** — breadcrumb (section / record, e.g. "Countries / Thailand"; ancestors are links, the current record is `text`), autosave indicator, search trigger (Field `search-trigger`, scoped label: "Search in Thailand" in a record, "Search everything" elsewhere).

**Autosave states** — `Saved` (`text-muted`), `Saving…` (shown only if a save takes over 300ms), `Couldn't save — retry` (`danger`, a quiet Button; content stays on screen, never lost).

**Tokens** — border (bottom rule), text, text-muted, danger, space-6; search trigger per Field.

**Back to parent** (D-33) — not part of the TopBar: every child page (region, city, POI, tip) shows a secondary Button "← <parent name>" above its title, beside a ⌘↑ / Alt+↑ keycap. It always goes up one level, never back in history. History back is ⌘[ / Alt+← and the mouse back button.

**Usage** — No page actions in the top bar; actions live in the Toolbar. No title here: the page title is in the content.

**Keyboard** — ⌘F / Ctrl+F opens the search the label describes (US-026, US-029). Breadcrumb links are ordinary links in tab order.

## ContentsIndex

The places inside a record, shown in the right column as links to their own pages: regions, their cities, and POI counts. Titled "Inside <place>" ("Inside Thailand"), never "Contents", because each row is a separate page with its own notes, not a section of this one (D-26).

**Structure** — heading "Inside <place>" with the unit ("POIs") and a one-line hint "Each region and city has its own page" until the first visit; rows end in a → on hover and focus; one block per region (name in `ui-strong`, POI total in `num-sm`), cities indented beneath with their counts. Cities attached directly to the country appear in an unnamed first block; country-level POIs show as "Other places · N".

**States** — default; hover on a row (`surface-active`); current (when browsing a region or city, its row is `surface-active` + 600); collapsed region (chevron, `duration-base`); empty country ("No regions or cities yet" + quiet "+ Region" and "+ City" buttons).

**Tokens** — border (rule above each region), text, text-muted, surface-active, radius-sm, space-1, space-3; numbers in `num-sm`.

**Siblings** — on a region or city page, a short "Also in <parent>" line under the index lists sibling places as links, so moving between cities doesn't need a trip back up.

**Usage** — Counts only, no dotted leaders (decision, F feedback). Every row is a link to that record. Order: regions alphabetically, cities alphabetically within them.

**Keyboard** — Rows are links in tab order. Within the index, Up/Down move between rows; Left/Right collapse/expand a region.

## TripCard

A compact, clickable trip row: small hero thumbnail, name, status and length.

**Variants**
- `compact` — under a country's notes ("Trips from Thailand"), in Travel Window shortlist pickers, in nav-adjacent lists. 48×36 thumbnail.
- `chosen` — in a Travel Window, the selected trip gets an `accent` outline and "Chosen" in `accent`; the others stay neutral, never dimmed (US-034).
- `preserved` — a deleted Completed trip kept in a Travel Window: a "Historical record" label in `text-muted`, read-only, no link into the library (US-034b).

**States** — default, hover (`border-control` outline), focus (ring), no photo (thumbnail shows a neutral `cat-4` block, never a broken image), long name (single line, ellipsis; full name in the tooltip and accessible name).

**Tokens** — surface, border, border-control, accent, text, text-muted, radius-lg, radius-sm, space-2; status per TripStatus.

**Usage** — Keep cards simple and out of the way (F feedback): no route bars, no budgets on the card. Cards are for trips because a trip is a thing you pick up and move between windows (decision D-06).

**Keyboard** — The whole card is one link: Tab to focus, Enter to open.

## PoiCard

A POI placed on a trip day, showing its name, type, priority and practical notes inline.

**Variants**
- `day` — in the trip builder: drag handle, name (`ui-strong`), type and Must-Do tag (`small`, `text-muted`), first lines of practical notes, optional day-specific note. No remove button on the card: selecting the card switches the trip-builder Toolbar to POI actions (Remove from day, Move to day…, Open POI), and Delete removes it.
- `read` — in presentation full detail: no handle, no remove, full practical notes.
- `result` — in in-context search results: name, parent path, + Add to day.

**States** — default, hover (`border-control` outline), selected (`accent` outline; toolbar shows POI actions), focus, dragging (lifted with `shadow-float`, original slot shows a `accent` drop line), deleted-from-library (cannot occur: a POI delete warns first and removes it from days, US-014).

**Tokens** — surface, border, text, text-muted, accent, shadow-float, radius-lg, space-2, space-3.

**Usage** — Same POI can appear on several days. The card never edits the library POI; "Open POI" goes to its record.

**Keyboard** — Handle is focusable: Space picks up, Up/Down moves within the day and across days, Space drops, Escape cancels (via @dnd-kit's keyboard sensor). Delete removes from the day.

## PhotoGrid

A small grid of a record's photos with the hero marked, kept small so notes stay the focus.

**Variants**
- `panel` — right column of a country/region/city/POI: 3 across, 4:3 thumbnails, "+ Add" in the heading.
- `manage` — opened from the panel: larger grid, reorder by drag, "Set as hero", Remove.

**States** — default; hover/focus on a photo (actions appear: Set as hero, Remove); hero (a "Hero" label, top-left, `bg` fill and `text`); importing (thumbnail placeholder with spinner; target under 2s, US performance); drag-over (whole grid shows a dashed `border-control` drop zone "Drop photos to add"); empty (one dashed add tile "Add photos" — presentation then shows a neutral placeholder, never a broken image); import error ("Couldn't import IMG_2041.HEIC — use JPEG, PNG, WebP or GIF" in `danger`).

**Tokens** — bg, text, text-muted, border-control, danger, radius-sm, space-1, space-2.

**Usage** — The hero is never chosen automatically (US-011). Tips, budgets and Travel Windows take no photos.

**Keyboard** — Photos are focusable in reading order; Enter opens the photo; H sets hero; Delete removes (with undo toast); in `manage`, Space + arrows reorders.

## RichTextEditor

The notes field on every content type: writing directly on the page, with a small formatting bar.

**Formats (FR13)** — Heading (section, subsection), bold, italic, underline, bullet list, numbered list, link. Nothing else in v1 (highlight is not in FR13 — a possible `/pm` addition).

**Variants**
- `page` — country/region/city/POI/trip notes: no box, sits on `bg`, `notes` type, max 720px line length. The format bar appears on focus, pinned above the field.
- `inline` — day notes and tip content: same editor, `surface` box with `border`.
- `read` — presentation: read-only render, same type, no bar.

**States** — empty (subtle prompt in `text-muted`: "Write anything — it saves as you go"), focused (bar visible), active format (`surface-active` on its bar button), link editing (small popover with the URL field), paste (keeps basic formatting; otherwise plain text, never corrupted).

**Tokens** — bg, surface, border, surface-active, text, text-muted, accent (links), radius-md, radius-xl, space-4; type `notes`, `section`, `subsection`.

**Usage** — Notes are the heart of a country page (F feedback: maximise writing room). Never put notes in a card on a record page.

**Keyboard** — ⌘B / ⌘I / ⌘U, ⌘K link, ⌘⇧7 numbered, ⌘⇧8 bullets, ⌘⌥1/2 headings. Tab leaves the editor (it does not indent; ⌘] / ⌘[ indent lists). Esc returns focus to the page.

## ItineraryBlock

A location block in the trip builder, collapsed (location + duration) or expanded into numbered days.

**Variants**
- `collapsed` — drag handle, location name with its level path in `text-muted` ("Chiang Mai · Northern Thailand"), duration in `num` ("6 days" or "No duration" in `text-muted`), a quiet "Expand to days" Button (labels, not a chevron — D-07). The drag handle is a drawn 6-dot grip icon (`text-muted`, 20×24 hit area), never a text character (D-45). PoiCards inside an expanded block use `radius-sm` (1:2 with the block's `radius-lg`).
- `expanded` — header as above, then one DayRow per day: day number in `num` mono, optional date, day notes (RichTextEditor `inline`), PoiCards, "+ POI" and "+ Note" quiet buttons.
- `widened` — after a region/city delete the block keeps its days and shows "Was Chiang Mai — now Thailand" in `text-muted` once (US-012).

- `travel` (D-27) — a flight, train, bus, ferry or drive between stops: dashed `border-control` outline on `sidebar` fill, a mode label in `micro` caps ("FLIGHT", "TRAIN", "BUS", "FERRY", "DRIVE" — D-45), from → to (free text; optionally linked to a library city), duration in whole days or 0. A 0-day travel block that opens the trip is **Day 0** ("Day 0 · evening" — leaving after work, no day off used; day numbering then starts at Day 1 with the next block); a 0-day block later in the trip shows "overnight" or "same day" with no day number. Only the first block can be Day 0. optional notes (flight numbers, connections). Counts toward trip length and cost per day; takes no POIs; never creates budget lines.

**States** — default, selected (`accent` outline; Toolbar switches to block actions), hover, focus, dragging (`shadow-float`, `accent` drop line between blocks), no duration + Expand (Expand disabled with "Set a duration first"), reducing duration on an expanded block (ConfirmDialog: "Days 5 and 6 will be removed, with their notes and 2 POIs").

**Tokens** — surface, border, accent, text, text-muted, shadow-float, radius-lg, space-2, space-3, space-4, duration-base, ease-out.

**Usage** — Blocks of different geographic levels mix freely. Expanding never adds POIs. Mode never changes trip status (FR7).

**Keyboard** — Handle: Space to pick up, Up/Down to move, Space to drop, Escape to cancel. Enter on the header toggles expand/collapse. Inside days, Tab moves through notes and POIs in order.

## BudgetTable

The full budget: categories containing line items, with Budgeted, Actual and a color-coded Difference that carries the status (no Status column, D-18).

**Structure** — header row (Item · Budgeted · Actual · Difference); category rows (`sidebar` fill, 600 weight, subtotals, the category's Difference in its tier); line items (name, amounts in `num` mono right-aligned, optional plain-text note under the name in `text-muted`); a totals row with a 1px `text` rule above. Difference cells use Variance tiers: Under, Close, Over, Way over, Pending.

**States**
- Line item with no actual — Difference shows "Pending", never "$0".
- Actual but no estimate — actual shown, Difference "—", no tier.
- Category or total with some items pending — tier from the items with actuals; the pending lines themselves say "Pending" (no "partial" label, D-20).
- Editing a cell — the cell becomes a Field `number`; whole dollars and cents, USD.
- Dragging a line item or category — handle on hover; `accent` drop line.
- Empty budget — EmptyState: "No categories yet" with "+ Category" and "Start from a template".
- Deleting a category with items — ConfirmDialog naming the item count.

**Tokens** — sidebar, border, text, text-muted, accent; Variance tokens; space-2; numbers `num`.

**Usage** — All amounts USD. Difference = Actual − Budgeted. Tier thresholds are global, set in Settings › Budget (Close %, Way-over %), not per budget or per row.

**Keyboard** — Grid navigation: arrow keys move between cells, Enter edits, Enter again or Tab commits and moves right, Escape cancels. Alt+Up/Down reorders the focused row. Focusing a Difference cell announces its full sentence ("Over by $260, 36% above budget").

## BudgetSnapshot

A trip's budget at a glance: estimate, per day, and — once actuals exist — the total Difference in its Variance tier (Under, Close, Over, Way over).

**Variants**
- `panel` — trip record and presentation full detail: headline figure in `money-md`/`money-lg`, "Shared estimate", then Per day under a dashed `border-control` divider; "Primary · 1 of 2" when the trip has several budgets (US-021, US-040).
- `ledger` — presentation summary: rows in the facts ledger (Estimate, Works out to).
- `split` — presentation summary "Where the money goes": a 10px segmented bar in `cat-1`…`cat-5` plus a labeled legend with amounts. Never the bar alone.
- `multi` — trip record with several budgets: one row per budget, the primary first and marked.

**States** — no budget ("Budget not yet estimated", `text-muted`); a block without duration (Per day "Needs every stop's length" in `text-muted`); some actuals missing (total computed from entered actuals, with "2 items pending" in `text-muted` under the figure in the panel variant).

**Tokens** — surface, border, border-control, text, text-muted, accent (links), cat-1…cat-5, radius-xl, radius-sm, space-4; numbers `money-md`, `money-lg`, `num`.

**Per person** — not shown in this version (D-28). Budgets are shared; split-by-person budgets are a later item (handoff 18).

**Usage** — Same fields in the same position on every trip, so comparison reads at a glance (FR10). Never show full budget detail in the summary view.

**Keyboard** — Display only; the "Full budget" link is in tab order (key 4 in presentation full detail).

## SearchOverlay

One overlay for both global and in-context search, opened by ⌘F, filtered with chips, results as you type.

**Variants**
- `global` — scope "Everything"; filters: Type, Country, Tag, Status (trip status only — countries have none, D-09).
- `in-record` — scope "In Thailand" shown as a removable scope chip; removing it widens to Everything.
- `trip-builder` — scope "Trip countries" with "Search full library" toggle; results have "+ Add to Day 3"; before typing, shows "Recent" POIs and locations (US-026); no match offers "Create POI 'Doi Inthanon' in Chiang Mai".

**Result row** — name (`ui-strong`), type and parent path in `small` `text-muted` ("POI · Chiang Mai · Thailand"), a short snippet with the match.

**States** — opening (`duration-slow`, `shadow-float`), empty query, results, active row (`surface-active`), no results ("Nothing matches 'pai canyn' — try fewer words or remove a filter", plus a Clear filters button), filters active (chips above results; each clears alone).

**Tokens** — surface, border, surface-active, text, text-muted, accent, shadow-float, radius-xl, space-2, space-4, duration-slow, ease-out; chips per TagChip.

**Usage** — Instant results, no loading state (NFR1). Filter options only list values that exist. Never navigate away when adding to a trip.

**Keyboard** — ⌘F opens with the input focused. Up/Down move through results, Enter opens or adds, ⌘Enter opens in place without closing (trip builder), Tab moves to filters, Escape closes and returns focus to where you were (US-029).

## ConfirmDialog

Asks before anything destructive and shows exactly what will be affected, from the API's cascade preview.

**Variants**
- `delete` — title names the record ("Delete Northern Thailand?"), body lists the cascade as counts ("3 cities and 19 POIs will be deleted · 2 tips become Unlinked · 1 trip block widens to Thailand"), danger primary "Delete region", secondary "Cancel".
- `refused` — when the delete cannot happen (`RECORD_IN_USE`): "Thailand is used by 2 trips" with the trips listed as links; one button, "OK, keep it". No danger button.
- `warning` — non-destructive confirmations (adding a 4th trip to a window, applying a template over categories, reducing a duration): secondary-weight "Continue", "Cancel".
- `restore` — replacing the library from a backup: names the restore point and its date ("Replace your library with the copy from yesterday 21:40?"), says a snapshot is taken first; danger "Restore and restart" (Phase 7 wording, replaces "Replace library").

**States** — open (`duration-slow`, `shadow-float`, scrim), focus trapped, Cancel focused by default for destructive variants.

**Tokens** — surface, border, text, text-muted, danger, on-accent, shadow-float, radius-xl, space-3, duration-slow.

**Usage** — Buttons repeat the verb ("Delete region", not "Yes"). Never stack two dialogs.

**Keyboard** — Focus moves into the dialog on open (Cancel for destructive, the primary otherwise); Tab cycles inside; Escape cancels; Enter activates the focused button only.

## EmptyState

What a section shows before it has content: one sentence that says what goes here, and the action to add it.

**Variants**
- `section` — inside a page region (a stub country's Contents, a trip with no blocks, a budget with no categories): dashed `border-control` box, `ui` sentence, and secondary add buttons for everything that can live there. A stub country offers all four: + Region, + City, + POI, + Tip (POIs and tips can attach straight to a country, US-004, US-051). A region offers + City, + POI, + Tip; a city + POI, + Tip.
- `inline` — a single `text-muted` line in a list ("Empty day — add a POI or a note").
- `search` — no results: says what was searched and how to widen ("Nothing matches 'pai canyn' — try fewer words or remove a filter") with Clear filters.

**States** — static; buttons follow Button.

**Tokens** — border-control, text, text-muted, radius-xl, space-2, space-6.

**Usage** — A stub is valid forever (FR2), so empty states invite, never nag: no "Complete your profile", no progress bars, no illustrations. Copy is plain and specific to the place ("Nothing in Peru yet").

**Keyboard** — The first button is in normal tab order; nothing auto-focuses.

## RouteStrip

A trip's stops as a horizontal strip whose segments are proportional to days, and the "Highlights by stop" columns that sit under it.

**Variants**
- `strip` — presentation summary and comparison: one segment per location block, width ∝ duration, alternating `accent` and `cat-5` so neighbours differ in lightness; stop name (`ui-strong`) and days (`num-sm`, "6d") under each.
- `highlights` — presentation summary: one column per stop with a 3px top rule in the stop's segment color, name + days, then its top 2–3 POIs (Must-do first).
- `text` — fallback in the ledger: "Bangkok → Chiang Mai → Chiang Rai → Pai".

**Travel segments** (D-27) — travel blocks with days draw as a neutral `border` hatched segment labeled by mode ("Flight · 2d"); 0-day travel draws as a thin divider between stops, not a segment.

**States** — block without duration (segment drawn at a minimum width with "—" for days, and the strip's total says "14+ days"); single stop (one full-width segment); very short stop (name moves to a tooltip below 48px, still in the accessible list).

**Tokens** — accent, cat-5, text, text-muted, radius-sm, space-1; numbers `num-sm`.

**Usage** — Proportional widths are the point: Chiang Mai's six days should look like the heart of the trip. Never a map (no network access, CSP).

**Keyboard** — Rendered as an ordered list; read as "Bangkok, 2 days; Chiang Mai, 6 days…". Not interactive.

## PresentationShell

The chrome-free frame for presentation mode: a quiet header, the content, and a footer with trip navigation or key hints.

**Parts**
- Header — window name and "Option 1 of 3" (summary) or "← Summary ⌫" plus the trip title in `trip-title-sm` (full detail); right side: "Shortcuts ?" and "Exit Esc", always visible but unobtrusive (US-042).
- Footer (summary) — the shortlisted trips as a dotted list (current filled), then "Full trip detail ↵", "Compare all three C", primary "Next: Portugal →". `lg` buttons.
- Footer (full detail) — key hints: ↑↓ scroll · 1–5 jump to section · ⌫ back to summary · ←→ other trips; "Read-only · exit presentation to edit".
- Section rail (full detail) — 1 Itinerary (with stops listed), 2 Places, 3 Tips, 4 Budget, 5 Photos; numbers in `num` mono, current section `surface-active` with its number in `accent`.
- Shortcuts sheet — opened by ?, a dialog listing every key; Escape closes it without leaving presentation.

**States** — single trip (no prev/next, no Compare; D-02), preserved record ("Historical record · deleted 3 Mar 2026" under the title, no links into the library), first/last trip (Prev/Next hidden at the ends, never disabled-looking).

**Tokens** — bg, border, surface-active, text, text-muted, accent, on-accent, radius-md, space-14; type `trip-title`, `trip-title-sm`, `num`. Follows the selected palette and theme (D-05).

**Usage** — No app chrome, no editing controls, no sidebar. A non-user must be able to drive it with arrows alone (FR12).

**Keyboard** — ←/→ trips, ↵ full detail, ⌫ back to summary, C comparison (proposed, `/pm`), 1–5 sections (proposed, `/pm`), ↑/↓ scroll, Tab cycles controls, ? shortcuts, Esc exits to exactly where you were.

## ComparisonColumn

One trip's column in the side-by-side comparison; every column shows the same fields in the same rows.

**Rows (fixed order, FR10)** — hero photo (16:10, `radius-sm`) · trip title (`trip-title-sm`) · concept (first two lines of the trip notes) · location summary (RouteStrip `text`, plus total days) · budget snapshot (estimate, per day; "Budget not yet estimated" when none) · vibe tags (TagChips; the whole row hidden only if no trip has tags, US-039).

**States** — chosen (`accent` top rule + "Chosen" label; others unchanged, never dimmed), missing field ("Not yet added" in `text-muted`, never a hidden row), preserved record (historical label), 2 vs 3 trips (columns share the width equally; never a 4-column layout — a 4th trip triggers the shortlist warning).

**Tokens** — border, accent, text, text-muted, cat-3 (photo placeholder), radius-sm, space-6; type `trip-title-sm`, `money-md`.

**Usage** — Requires two or more trips (D-02). Rows align across columns: a long concept clamps to two lines so budgets line up.

**Keyboard** — ←/→ move focus between columns; Enter opens that trip's full detail; Backspace from full detail returns here (US-043).

## SettingsRow

A labeled row on the Settings page: the setting's name and one-line explanation on the left, its control on the right.

**Sections (D-04)** — Appearance (Palette: Paper / Atlas; Theme: Light / Dark / Match system), Budget (Close within %, Way over beyond %; D-18), Backup & restore (backup folder, last backup, Back up now, Restore…), Budget templates (list, rename, delete), Tags (P2).

**Variants**
- `segmented` — 2–3 mutually exclusive options (Theme). `aria-pressed` on the chosen one.
- `palette` — swatch buttons showing each palette's ground and accent, name under each.
- `percent` — a number Field with a % suffix plus a live example line using the Variance tiers ("On a $1,000 line: Close up to $1,100 · Over up to $1,250 · Way over beyond"). Close defaults to 10, Way over to 25; Way over must be greater than Close (inline error otherwise: "Way over must be higher than Close (10%)").
- `action` — status text plus buttons (Backup folder: path · last backup · Change… · Back up now).
- `notice` — a standing `chip` notice when no backup folder is set or the last backup is over 7 days old (NFR6).

**States** — default, focus, changing palette/theme (applies instantly, `duration-slow` cross-fade, none with reduced motion), invalid threshold (error, previous value kept until fixed), backup running ("Backing up…" + disabled button), backup folder unavailable ("Drive not connected — last backup 9 days ago" in `warning`).

**Tokens** — border, surface, surface-active, chip, border-control, text, text-muted, warning, danger, accent, radius-md, space-4, space-6; Variance tokens in the example line.

**Usage** — Every setting saves immediately; there is no Save button. Threshold changes recolor every budget at once.

**Keyboard** — Tab through rows; segmented and palette controls use Left/Right to change, as a radio group; percent fields accept Up/Down to step by 1.

## Phase 7 additions

These were drawn on the wireframes and hi-fi but had no spec card yet. They use only existing tokens. They are not yet in the design system artifact; add them there when it is next revised.

### SegmentedControl

Two to five mutually exclusive options in one outlined strip: List / Grid, One at a time / Side by side, Theme, the status filter on the Trips list, type filter on Tips.

**Look** — 1px `border-control` outline, `radius-md`, options on `surface`, height 32px, `space-3` side padding. The chosen option is `surface-active` + 600 weight with `aria-pressed="true"` (or a radio group where the choice changes the page).
**States** — default, hover (`surface-active` on the hovered option), focus (ring on the strip's focused option), disabled option (45% opacity, skipped by arrows).
**Keyboard** — Tab focuses the chosen option; Left/Right move and select (radio-group behaviour); Home/End jump to first/last.
**Usage** — Never more than 5 options; beyond that use a FilterMenu.

### FilterMenu

A button that opens a checkbox list for one filter dimension (Type, Country, Place, Tag, Trip status, From trip, Has follow-up). Active selections appear as TagChip `filter` chips in the filter row, each clearable alone.

**Look** — trigger is a small secondary Button ("Place ▾"), outlined in `text` while its menu is open. The menu is a `surface` panel, `radius-lg`, `shadow-float`, 220–260px wide; rows 28px with a checkbox, label and a `num-sm` count; hierarchical dimensions (Place) indent children by `space-6`. Footer line in `micro` `text-muted` explains the rule ("A country includes its regions and cities. Only places that have tips are listed.").
**States** — closed, open, option checked, option with zero results (not listed — only values that exist appear), keyboard-highlighted row (`surface-active`).
**Keyboard** — Enter/Space or Down opens; Up/Down move; Space toggles; Escape closes and returns focus to the trigger; Tab closes and moves on.

### Menu

A single-choice popup from a button: trip Status, "More ▾" on records, sort order.

**Look** — like FilterMenu without checkboxes; the current choice carries a check glyph and 600 weight. Destructive items (Delete…) sit last, below a `border` rule, in `danger`.
**Keyboard** — as FilterMenu; Enter selects and closes.

### PickerPopover

A small search-and-pick popover anchored to the control that opened it: "+ Country" on a trip, "+ Location block", "Move to another place…", "Add trip" to a window.

**Look** — `surface`, `radius-xl`, `shadow-float`, 300–360px wide; a search Field at the top with Esc keycap; results below with `num-sm` counts; the active row `surface-active`; a last row "New <thing> '<typed>'" with the drawn plus when nothing matches exactly.
**Keyboard** — opens with focus in the field; Up/Down move; Enter picks and closes; Escape closes, focus returns to the opener.

### FormDialog

A modal for creating something that needs more than a name: New tip, New Travel Window, New budget.

**Look** — like ConfirmDialog (`surface`, `radius-xl`, `shadow-float`, scrim), 520–560px wide; Fields stacked with labels above; buttons right-aligned, the primary named for the result ("Add lesson", "Create window"), never "OK" / "Submit"; ⌘↵ submits.
**States** — open (`duration-slow`), invalid (Field error under the field, primary stays enabled and shows the errors on press), saving (primary disabled with spinner for anything over 300ms).
**Keyboard** — focus starts in the first field; Tab cycles inside (trapped); ⌘↵ submits; Escape cancels with nothing saved and returns focus to the opener.

### Banner

A one-line notice across the top of a content area for a state that changes how the page behaves: "Historical record — deleted 3 Mar 2026, read-only", "Read-only · exit presentation to edit".

**Look** — 1px dashed `border-control`, `sidebar` fill, `radius-lg`, `space-3` padding; a `micro` caps label on `surface` (e.g. "HISTORICAL RECORD") then one `small` sentence. No icon, no close button — it lasts as long as the state does.
**Keyboard** — not focusable; read in document order before the content it describes.

### TypeLabel

The TIP / LESSON marker on tips, and the mode marker on travel blocks (FLIGHT, TRAIN, BUS, FERRY, DRIVE).

**Look** — `micro` 600, uppercase, letter-spacing 0.04em, `radius-sm`, `0 space-2` padding. TIP and travel modes: 1px `border-control` outline, `text-muted`. LESSON: filled `text` with `bg` lettering — the fill (shape), not a hue, tells it apart.
**Usage** — Always the word; never a color-only dot.

### Toast

Brief confirmation with Undo after a removal that has no confirm dialog (POI removed from a day, photo removed, tag removed).

**Look** — bottom-left of the content area, `surface`, `radius-lg`, `shadow-float`, one sentence + quiet "Undo" Button; 6 seconds, pauses on hover/focus; one at a time (a new one replaces the old).
**Motion** — enters with `duration-base` `ease-out`, leaves with `duration-base` `ease-in`; instant with reduced motion.
**Keyboard** — announced politely (`aria-live="polite"`); ⌘Z also undoes while it is showing; not focus-stealing.

### DayRow

One day inside an expanded ItineraryBlock: day number column (64px, `num` mono, `text-muted`), then the day's notes (RichTextEditor `inline`), PoiCards (`radius-sm` inside the block, 1:2 rule), and quiet "+ POI" / "+ Note" Buttons. An empty day shows EmptyState `inline` "Empty day — add a POI or a note". A collapsed run of days shows one row "Days 5–8 · 4 more days · 5 POIs" that expands on click or Enter.
