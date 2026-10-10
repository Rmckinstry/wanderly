---
title: Countries and places — list, country, region, city
status: ready for build
stories: [US-001, US-002, US-003, US-007, US-007b, US-008, US-011, US-025, US-026, US-028, US-056, US-056b, US-056c]
last-updated: 2026-10-09
---

# Countries and places

The master list and the place pages that hold a person's research. Country detail is a hero screen (D-01).

Wireframes: CountriesList, CountriesGrid, CountryDetail, CountryStub, RegionDelete. Hi-fi: Country detail (Paper light / dark).

---

## Countries list

Screen:      Countries list (List view default, Grid view)
Serves:      US-001, US-025, US-028 (P1), D-15, D-24
Components:  SegmentedControl, Menu (sort), FilterMenu (tag), Button, TagChip, EmptyState, PickerPopover (none), SideNav

States:
- default (List) — header "Countries" (`page-title`) with mono counts ("23 countries · 9 regions · 41 cities · 312 POIs"); controls row: List/Grid SegmentedControl, sort Menu ("Newest first", "Name A–Z", "Most POIs"), "+ Filter by tag", primary "+ New country". Rows: Pinned group first (pin icon, drag to reorder), then the rest by the chosen sort. Each row: name (`ui-strong`), mono counts, tags as chips, trips count.
- default (Grid) — 4 columns of cards (hero photo 132px, name, counts, trips). A country with no hero shows its initial in `text-muted` on a neutral block (D-24).
- quick-create row — "+ New country" (or ⌘N › Country) inserts an inline name Field at the top of the list, focused, placeholder-free label "New country name".
- empty — EmptyState `section`: "No countries yet. Start with somewhere you keep thinking about." + primary "+ New country".
- filtered, no match (P1) — EmptyState `search`: "No countries tagged Hiking and Food — remove a tag" + Clear filters.
- error — name Field errors only (below).

Interactions:
- Row/card click or Enter opens Country detail. Hover: `surface-active` row / `border-control` card outline.
- Drag a pinned row by its grip to reorder pinned (Pinned group only; the rest follow sort).
- List/Grid choice is remembered (UI state).
- Filter by tag (P1): FilterMenu listing only tag values in use; multiple tags match ALL; chips clear individually.

Keyboard:
- Tab: header controls in order → first row. Up/Down move between rows (Left/Right/Up/Down in Grid). Enter opens.
- In the quick-create Field: Enter creates and opens the new country; Escape cancels — nothing is saved.
- Alt+Up/Down reorders a focused pinned row.

Motion: new row inserts with `duration-base` height expand; none with reduced motion.

Edge cases:
- US-001 empty name → Enter does nothing; Field `error` "Give the country a name". Escape still cancels.
- US-001 duplicate name → Field `error` "A country named Thailand already exists — open it" where "open it" is a link to it; no record created.
- US-001 navigate away mid-creation → the unsaved inline row is discarded; nothing is created.
- US-001 / US-056 no automatic tags → new country has only the "+ Tag" chip.
- US-025 multi-country trips → a trip's count appears on every country it links.
- US-027 status filter → none: countries have no status (D-09, handoff 1).

Handoff notes: rows 48px; counts `num-sm`; grid gap `space-4`; card hero 132px tall, flush to the card's top edge and clipped by the card's `radius-lg` corners (the image has no radius of its own, so there is no nested-radius clash). Sort and view stored per list.

---

## Country detail

Screen:      Country detail (populated) — hero
Serves:      US-007, US-007b, US-011, US-025, US-026, US-056, D-13, D-26, D-29, D-41
Components:  TopBar, Toolbar (`record`), TagChip, RichTextEditor (`page`), ContentsIndex, PhotoGrid, TripCard (`compact`), TypeLabel, Menu ("More ▾"), Button, EmptyState

States:
- default — header: contour lines behind the title (D-41; decorative, no pointer events, clipped at the toolbar), title `page-title`, mono counts "4 regions · 9 cities · 61 POIs · 7 tips", tag chips + "+ Tag". Toolbar band: + Region, + City, + POI, + Tip · spacer · More ▾ · primary "Start a trip". Body: left column "Thailand notes" label then the notes editor (max 720px); below it "Trips in Thailand" (TripCard compact, 2-up). Right column (320px): "Inside Thailand" (ContentsIndex), Photos (PhotoGrid, hero labelled), Tips (latest 2–3, TypeLabel + title + one line).
- stub — see Country stub below.
- notes empty — editor prompt "Write anything — it saves as you go" in `text-muted`.
- no photos — PhotoGrid shows one dashed "Add photos" tile.
- no tips — "No tips for Thailand yet" + quiet "+ Tip".
- no trips — the "Trips in Thailand" section is omitted entirely (no empty block under notes).
- saving / save failed — TopBar autosave states (app-shell.md).

Interactions:
- Start a trip (D-29) → creates a new trip named "<Country> trip" with only this country linked (chip in the trip header), status Sample, no blocks; opens Trip builder with the title selected for renaming.
- + Region / + City / + POI / + Tip → inline name Field at the top of "Inside Thailand" (Region, City), or a new record page (POI) with its name Field focused, or FormDialog (Tip, P1). + City offers "Directly in Thailand" or a region (Menu) because a city can attach to the country or a region (US-003).
- More ▾ → Pin to sidebar / Unpin · Rename · Delete country….
- ContentsIndex row → opens that region/city page.
- Photos: click opens the photo; hover/focus shows Set as hero / Remove; drag files onto the grid to add (US-011). Remove shows Toast with Undo.
- Tag chips: "+ Tag" opens the tag selector; × on hover removes (Toast with Undo).
- Links in notes open in the system browser (the app has no network; external links leave the app).

Keyboard:
- Tab order: (shell) → title → tag chips → + Tag → toolbar buttons (Left/Right move inside the toolbar, Tab leaves it) → notes editor → trip cards → Inside Thailand rows → photos → tips.
- ⌘F → SearchOverlay scoped "In Thailand" (search.md).
- In notes: RichTextEditor keys (components.md). Esc returns focus from the editor to the page (to the "Thailand notes" label).
- Photos: Enter opens, H sets hero, Delete removes (Toast + ⌘Z).

Motion: ContentsIndex region collapse `duration-base`; photo hover actions fade `duration-fast`.

Edge cases:
- US-007 stub forever → no completeness indicators anywhere.
- US-007 navigate away mid-edit → autosave (app-shell.md).
- US-007 status does not change → no status exists on countries (D-09).
- US-007b paste from OneNote/browser → keeps headings, bold/italic/underline, lists and links; anything else becomes plain text. No toast.
- US-007b empty notes prompt never forces formatting → prompt only, no template.
- US-011 hero is never automatic → no "Hero" badge until the person sets one; presentation then uses the first available photo (handoff 12).
- US-011 Tips/Budgets/Windows take no images → no photo controls on those screens.
- US-025 counts at each level → header counts + ContentsIndex counts.
- US-025 filtering within a country (e.g. POIs tagged Hiking in Thailand) → ⌘F in-record search with Type and Tag filters (search.md); no separate filter UI on the page.
- US-002 delete country used by a trip → Delete country… shows ConfirmDialog `refused`: "Thailand is used by 2 trips" listing them as links; one button "OK, keep it".
- Delete country not used by trips → ConfirmDialog `delete` listing the cascade counts ("4 regions, 9 cities and 61 POIs will be deleted · 7 tips become Unlinked"); danger "Delete country"; Cancel focused by default.

Handoff notes: header top padding `space-6`; title-to-meta `space-3`; toolbar band per app-shell.md. Notes column max 720px, right column 320px, gutter `space-14`. Contour SVG: 520×150 at the header's right, `contour` stroke 1.5px, z-index below content. Tip rows show at most 3; "All 7 tips →" link beneath when more.

---

## Country stub

Screen:      Country detail — new stub (no content)
Serves:      US-001, US-007, US-025
Components:  as Country detail + EmptyState (`section`)

States:
- default — title, counts "0 regions · 0 cities · 0 POIs", "+ Tag" only; notes editor with its prompt; right column "Inside Georgia" shows EmptyState `section` "Nothing in Georgia yet" with + Region, + City, + POI, + Tip (POIs and tips can attach directly to a country).

Interactions, Keyboard, Motion: as Country detail. Focus lands on the notes editor when the stub is opened right after creation; otherwise on the page.

Edge cases:
- US-025 "a country with no content displays an empty state with prompts" → as above, no nagging copy.
- US-002 "a country can have zero regions" → + City and + POI work without a region.

---

## Region and city detail

Screen:      Region detail / City detail
Serves:      US-002, US-003, US-008, US-056b, US-056c, D-26, D-33
Components:  as Country detail + Button ("← <parent>"), ContentsIndex (with "Also in <parent>")

States:
- default — "← Thailand" Button and ⌘↑ keycap above an eyebrow-free meta line "Region in Thailand"; title; counts; chips; contour lines (D-41). Toolbar: Region page: + City, + POI, + Tip · More ▾; City page: + POI, + Tip · More ▾. No "Start a trip" (trips start from a country; a block can still target this place from the builder). Notes labeled "Northern Thailand notes". Right column: "Inside Northern Thailand" (cities with POI counts; city page lists its POIs), then "Also in Thailand" sibling links.
- empty — EmptyState `section` offering + City, + POI, + Tip (region) or + POI, + Tip (city).

Interactions: as Country detail; More ▾ → Rename · Move to another country… (not in v1 — omitted) · Delete region….

Keyboard: as Country detail; ⌘↑ / Alt+↑ goes to the parent.

Edge cases:
- US-002 region outside a country context → + Region exists only on a country page (and ⌘N does not offer Region).
- US-002 / US-003 empty name → inline Field error "Give the region a name".
- US-002 duplicate region in the same country → "Thailand already has a region called North — open it"; the same name in another country is allowed.
- US-003 duplicate city in the same parent → same pattern; allowed across parents.
- US-008 editing a region does not change the parent → no parent-level indicators update except counts.
- Delete region (wireframe RegionDelete) → ConfirmDialog `delete`: "Delete Northern Thailand?" with the cascade: "3 cities and 19 POIs will be deleted · 2 tips become Unlinked — kept, not deleted · 1 trip block (Northern Thailand Slow Loop) widens to Thailand, days kept · 6 POIs are removed from that trip's days". Danger "Delete region", Cancel focused by default (US-012 widening rule).
- US-003 delete parent region → same dialog lists the city.

Handoff notes: "← Parent" Button is secondary, 30px tall, `space-3` above the meta line. Sibling list is a single wrapped line of links separated by " · ".
