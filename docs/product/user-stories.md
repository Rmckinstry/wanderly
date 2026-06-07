# Travel App — User Stories

## Priority Definitions
- **P0** — Core MVP. App doesn't work without it.
- **P1** — Important post-MVP. App works but feels incomplete without it.
- **P2** — Future enhancement. Valuable but not required for the app to deliver its core purpose.

## Global Rules
- Tags are **never applied automatically** — always manual, on every content type
- **Rich text formatting** is supported on all free-form notes fields across every content type (bold, italic, underline, bullet points, numbered lists, hyperlinks)
- **Status is always manual** — never auto-derived from content or actions
- **No required fields beyond a name/title** on any content type — all records support progressive enrichment

---

## 1. Content Management — Creation

---

**US-001 — Create Country**
*When I discover a new destination I'm curious about, I want to create a country with just a name, so that I can capture the idea immediately without any research pressure.*

**Priority:** P0

**Acceptance Criteria:**

Given I am anywhere in the app,
When I create a new country and enter a name,
Then a country record is created with status `Wishlist`, all other fields empty, and it appears in my master library immediately.

**Edge Cases:**
- If I try to save with an empty name the app blocks creation and prompts for a name
- Country names must be unique — if a country with the same name already exists the app blocks creation and surfaces the existing record
- If I navigate away mid-creation without saving no record is created and no data is lost
- No tags are applied automatically

---

**US-002 — Create Region**
*When I'm researching a country and find it has distinct areas, I want to create a region from within that country, so that it is automatically linked and my research stays geographically organized.*

**Priority:** P0

**Acceptance Criteria:**

Given I am viewing a specific country record,
When I create a new region,
Then the region is automatically linked to that country, appears nested under it in the content hierarchy, and is immediately available as a container for cities and POIs.

**Edge Cases:**
- Region creation is only available from within a country view — it cannot be created from global or uncontextualized screens
- If I try to save a region with no name creation is blocked and I am prompted for a name
- Region names must be unique within the same country — two regions in Thailand cannot both be called "North" — but the same region name can exist across different countries
- If the parent country is deleted the app warns me that all nested regions, cities, and POIs will be affected before confirming deletion
- A country can have zero regions — cities and POIs can attach directly to a country if no region structure is needed
- No tags are applied automatically

---

**US-003 — Create City**
*When I'm researching a region or country, I want to create a city from within that context, so that it is automatically linked and POIs and tips have a clear home in the hierarchy.*

**Priority:** P0

**Acceptance Criteria:**

Given I am viewing a specific region or country record,
When I create a new city,
Then the city is automatically linked to that region or country, appears nested under it in the content hierarchy, and is immediately available as a container for POIs and tips.

**Edge Cases:**
- City creation is only available from within a region or country view
- A city can attach directly to a country if no region exists — region is not required
- If I try to save a city with no name creation is blocked
- City names must be unique within the same parent — but the same city name can exist across different parents
- If the parent region is deleted the app warns me the city and its contents will be affected
- No tags are applied automatically

---

**US-004 — Create POI**
*When I come across an interesting place, restaurant, or experience, I want to create a POI linked to the most relevant level of the hierarchy, so that it's captured and findable when I'm building a trip later.*

**Priority:** P0

**Acceptance Criteria:**

Given I am viewing a country, region, or city record,
When I create a POI with at minimum a name,
Then the POI is automatically linked to the current context (country, region, or city), appears under its parent, and is immediately searchable and available for use in trip itineraries.

**Edge Cases:**
- POI name is the only required field — all other fields (description, tags, practical notes) are optional
- A POI can be linked to a country, region, or city — city is not required
- Duplicate POI names within the same parent trigger a warning but do not block creation
- If I add a POI via mobile quick-capture with no parent specified it is saved to an Unassigned inbox for later organization
- A POI can be re-linked to a different parent if it was initially miscategorized
- No tags are applied automatically

---

**US-005 — Create Trip**
*When I want to rough out what a trip could look like, I want to create a Trip with just a name, so that I have a planning container I can fill in over time as interest and detail grows.*

**Priority:** P0

**Acceptance Criteria:**

Given I am in the master library or within a country record,
When I create a new Trip with at minimum a name,
Then the trip is created with status `Sample`, appears in the master library, and is available to be added to a Travel Window.

**Edge Cases:**
- A Trip can reference multiple countries — a multi-country itinerary is valid
- If no country is linked at creation the trip is saved in an Unlinked state and can be linked later
- Trip names do not need to be unique — two trips named "Northern Thailand" can exist at different planning stages
- No tags are applied automatically

---

**US-006 — Create Tip or Lesson Learned**
*When I want to capture useful knowledge about a destination or travel in general, I want to create a Tip or Lesson Learned with a scope and type, so that it's retrievable when researching or planning.*

**Priority:** P1

**Acceptance Criteria:**

Given I am anywhere in the app,
When I create a Tip and specify its type (Tip or Lesson Learned) and scope (General or Destination-specific),
Then it is saved and linked to the appropriate country, city, or trip if destination-specific, or stored in the General Tips library if general.

**Edge Cases:**
- A Lesson Learned can be General or Destination-specific
- A General tip or Lesson Learned with no destination link is valid and lives in its own section
- If I mark a tip as Destination-specific but don't select a destination the app prompts for one before saving
- A Lesson Learned can only be linked to a trip with `Completed` status — linking to an active trip is not permitted
- No tags are applied automatically

---

## 2. Content Enrichment & Editing

---

**US-007 — Enrich a Country Record**
*When my interest in a country grows, I want to add notes, images, and details to a country record, so that it evolves from a stub into a meaningful research base.*

**Priority:** P0

**Acceptance Criteria:**

Given I am viewing a country record,
When I edit the record and add notes, images, or details,
Then all changes are saved immediately and reflected in the country record without requiring a manual save action.

**Edge Cases:**
- A country record can remain a stub indefinitely — no fields are ever required beyond the name
- Images are optional and can be added or removed at any time
- Notes are free-form rich text with no length restriction
- If I navigate away mid-edit the app either auto-saves or prompts me to save before leaving — no silent data loss
- Status does not automatically change when content is added — it must be updated manually

---

**US-007b — Rich Text Editing**
*When I'm adding notes to any content record, I want a rich text editor, so that I can structure my content with formatting that makes it scannable and mirrors how I already organize my research.*

**Priority:** P0

**Acceptance Criteria:**

Given I am editing any free-form notes field on any content type (Country, Region, City, POI, Trip, Tip, Budget),
When I enter text into the notes field,
Then a rich text editor is available supporting at minimum: bold, italic, underline, bullet points, numbered lists, and hyperlinks.

**Edge Cases:**
- Rich text formatting must be preserved when viewing on mobile — both capture and reference modes
- Rich text formatting must render correctly in presentation mode
- Pasting content from OneNote or a web browser should preserve basic formatting where possible — if formatting cannot be preserved plain text is pasted without silent data corruption
- An empty notes field shows a subtle prompt but never forces formatting on the user
- Rich text applies to notes fields only — name and tag fields remain plain text

---

**US-008 — Enrich a Region or City Record**
*When I'm building out research for a destination, I want to add notes and details to a region or city, so that context is captured at the right level of the hierarchy.*

**Priority:** P0

**Acceptance Criteria:**

Given I am viewing a region or city record,
When I edit and add notes, images, or details,
Then changes are saved and reflected immediately within the parent country hierarchy.

**Edge Cases:**
- Notes and images are optional at both region and city level
- Editing a region or city does not affect the status of the parent country
- If a region or city has no content yet it displays an empty state prompt encouraging enrichment
- No tags are applied automatically

---

**US-009 — Enrich a POI Record**
*When I want to capture the full detail of a point of interest, I want to add a description, practical notes, images, and tags to a POI, so that I have enough context to evaluate and use it when planning a trip.*

**Priority:** P0

**Acceptance Criteria:**

Given I am viewing a POI record,
When I add a description, practical notes, images, or tags,
Then all details are saved and the POI is immediately usable in trip itinerary building with its full detail visible.

**Edge Cases:**
- All fields beyond POI name remain optional — a stub POI is valid indefinitely
- Practical notes are free-form rich text (e.g. "1.5hr drive from city", "book in advance", "best at sunset")
- Multiple images can be attached to a single POI
- Tags must be selected from the fixed contextual tag set — none are applied automatically
- A POI can be re-linked to a different parent if it was initially miscategorized

---

**US-010 — Enrich a Tip or Lesson Learned**
*When I want to add context to a tip, I want to edit its description, scope, type, and destination link, so that it remains accurate and useful as my knowledge grows.*

**Priority:** P1

**Acceptance Criteria:**

Given I am viewing a Tip or Lesson Learned record,
When I edit any field including description, scope, type, or destination link,
Then changes are saved immediately and the tip is correctly surfaced in relevant destination views and search results.

**Edge Cases:**
- Changing scope from Destination-specific to General removes the destination link — the app warns before doing so
- A Lesson Learned can be linked to a Completed Trip retroactively — the link is never required at creation
- If a linked destination is deleted the tip is not deleted — it moves to an Unlinked state
- No tags are applied automatically

---

**US-011 — Add Images to Any Content Type**
*When I want to give a destination or POI a visual identity, I want to attach images to any content record, so that they support the presentation mode and give a feel for the place.*

**Priority:** P1

**Acceptance Criteria:**

Given I am viewing any content record,
When I attach one or more images,
Then the images are stored with the record and I can designate any image as the hero image used in presentation mode.

**Edge Cases:**
- Multiple images can be attached to any record
- I can designate any image as the hero image — it does not default to the first one without my input
- If no image is attached presentation mode shows a placeholder rather than a broken image
- Images can be removed or reordered at any time
- No automatic tagging or categorization of images

---

## 3. Trip Building

---

**US-012 — Build a Trip in Location-Block Mode**
*When I'm roughing out a trip concept, I want to add location blocks at any geographic level with rough durations, so that I can see the shape of the itinerary without committing to day-by-day detail.*

**Priority:** P0

**Acceptance Criteria:**

Given I am viewing a Trip record,
When I add a location block and select a country, region, or city as the anchor with an optional duration,
Then the block appears in the itinerary in sequence, the total trip duration is auto-calculated from all blocks that have durations assigned, and the trip remains in location-block mode until I choose to add day-level detail.

**Edge Cases:**
- Location blocks can be reordered via drag or manual sequencing
- A block can be set at country, region, or city level — no level is required over another
- Duration is optional on any block
- A trip can mix blocks at different geographic levels (e.g. "Italy — 3 days" alongside "Chiang Mai — 4 days")
- A block can be refined over time — a country block can be replaced by more specific region or city blocks as planning matures
- Deleting a block does not delete the referenced country, region, or city from the master library
- Multiple blocks can reference the same location (e.g. returning to Rome at the end of an Italy trip)
- A trip can have a single location block (e.g. a Tuscany-only trip)

---

**US-013 — Transition a Trip from Location-Block to Day-by-Day Mode**
*When my trip planning matures, I want to break location blocks into individual days, so that I can build a detailed day-by-day itinerary without losing the high-level structure.*

**Priority:** P0

**Acceptance Criteria:**

Given I am viewing a Trip with at least one location block that has a duration assigned,
When I choose to expand a location block into individual days,
Then the block expands into numbered days matching the assigned duration, the location context is preserved on each day, and unexpanded blocks remain intact in location-block mode.

**Edge Cases:**
- Blocks can be expanded one at a time — not all blocks need to be expanded simultaneously
- If a block has no duration assigned I am prompted to set one before expanding into days
- Expanding a block does not automatically populate days with POIs — days start empty
- If I reduce the duration of a block after expanding the app warns me that day entries beyond the new duration will be removed
- A trip can exist in a mixed state — some blocks expanded into days, others still in location-block mode
- A country or region block expanded into days can have its days later reorganized into sub-blocks if needed

---

**US-014 — Add POIs to a Day**
*When I'm building out a day in my itinerary, I want to search for and add POIs from my library to that day, so that my existing research feeds directly into the trip plan.*

**Priority:** P0

**Acceptance Criteria:**

Given I am viewing a specific day within a Trip itinerary,
When I search for and select a POI from the library,
Then the POI is added to that day with its name, description, and practical notes visible inline, and the POI remains intact in the master library.

**Edge Cases:**
- In-context POI search is scoped to the countries referenced in the trip by default — but can be expanded to the full library
- The same POI can be added to multiple days (e.g. a restaurant visited twice)
- Adding a POI to a day does not remove it from the library or mark it as used
- POIs can be reordered within a day
- A day can have zero POIs — free-form notes on a day are valid without any POI attached
- If a POI is deleted from the library the app warns me it is referenced in one or more trip itineraries before confirming deletion

---

**US-015 — Add Free-Form Notes to a Day**
*When planning a specific day, I want to add free-form notes alongside POIs, so that I can capture logistics, travel details, and context that don't fit neatly into a POI record.*

**Priority:** P0

**Acceptance Criteria:**

Given I am viewing a specific day within a Trip itinerary,
When I add free-form notes to that day,
Then the notes are saved and displayed alongside any POIs on that day in both planning and reference mode.

**Edge Cases:**
- Free-form notes and POIs can coexist on the same day in any combination
- A day can consist entirely of free-form notes with no POIs
- Notes have no length restriction
- Rich text formatting is supported per US-007b

---

**US-016 — Manually Update Trip Status**
*When my trip planning reaches a new stage, I want to manually update the trip status, so that I can track where each trip is in its lifecycle.*

**Priority:** P0

**Acceptance Criteria:**

Given I am viewing a Trip record,
When I manually change the status,
Then the status updates immediately and is reflected everywhere the trip appears including the master library and any Travel Windows it belongs to.

**Edge Cases:**
- Status can be set to any stage at any time — it does not have to advance sequentially
- A trip can be moved back to a previous status (e.g. from `Planning` back to `Sample`)
- Marking a trip as `Completed` does not lock the record — it remains fully editable
- Status change does not trigger any automatic changes to content, tags, or linked records
- Marking a trip as `Completed` within a Travel Window triggers an automatic snapshot per US-034b

---

**US-017 — Manually Update Country Status**
*When my research on a country reaches a new depth, I want to manually update the country status, so that I can see at a glance where each destination sits in my research pipeline.*

**Priority:** P0

**Acceptance Criteria:**

Given I am viewing a Country record,
When I manually change the status,
Then the status updates immediately and is reflected in the master library and any filtered views.

**Edge Cases:**
- Status can be set to any stage in any order — does not need to advance sequentially
- A country status can be moved backwards (e.g. from `Sampling` back to `Researching`)
- Country status is independent of any Trip status — completing a trip does not change the country status
- No content is affected by a status change

---

## 4. Budget

---

**US-018 — Create a Budget**
*When I want to estimate the cost of a destination or trip, I want to create a budget attached to any country, region, or trip, so that I can track costs from a rough estimate through to final actuals.*

**Priority:** P0

**Acceptance Criteria:**

Given I am viewing a Country, Region, or Trip record,
When I create a new budget with at minimum a name,
Then a budget is created linked to that record, starts empty or from a selected template, and is immediately available to add categories and line items to.

**Edge Cases:**
- A record can have multiple budgets (e.g. two budget scenarios for the same destination)
- A budget can be created from scratch or from a saved template
- A budget can be saved with zero categories or line items — no minimum content required
- Budget names must be unique within the same parent record
- No tags are applied automatically

---

**US-019 — Add Categories and Line Items to a Budget**
*When I'm building out a budget, I want to add categories and line items with estimated costs, so that I can organize and track spend across all areas of the trip.*

**Priority:** P0

**Acceptance Criteria:**

Given I am viewing a Budget record,
When I add a category and one or more line items,
Then each line item displays three columns — Budgeted/Estimate, Actual, Difference — and the summary snapshot auto-calculates totals across all categories in real time.

**Edge Cases:**
- A line item requires at minimum a name — all cost columns are optional
- Categories can be added, renamed, reordered, and deleted at any time
- Deleting a category that contains line items prompts a warning — line items are deleted with the category
- Category names must be unique within the same budget
- A budget can have any number of categories and line items with no upper limit enforced
- All amounts are in USD

---

**US-020 — Track Actuals Against Estimates**
*When I start making real bookings, I want to enter actual costs against my estimates, so that I can see exactly where I'm over, under, or close across every line item.*

**Priority:** P0

**Acceptance Criteria:**

Given I am viewing a Budget with at least one line item,
When I enter an actual cost for a line item,
Then the Difference column auto-calculates (Actual minus Budgeted) and a visual indicator shows whether that line item is Under, Over, or Close — at the line item level, category subtotal level, and overall summary level.

**Edge Cases:**
- Under = actual is below budgeted estimate
- Over = actual exceeds budgeted estimate — flagged visually (e.g. red)
- Close = actual is within a defined threshold of the estimate — flagged visually (e.g. yellow). Threshold is user-configurable, defaulting to 10%
- A line item with no actual entered shows Difference as pending — not zero
- A line item with actual but no estimate still calculates and displays the actual with no variance shown
- All amounts are in USD

---

**US-021 — View Budget Summary Snapshot**
*When I want a quick read on trip cost, I want to see a budget summary snapshot on any record, so that I can evaluate affordability at a glance without opening the full budget.*

**Priority:** P0

**Acceptance Criteria:**

Given I am viewing a Country, Region, or Trip record that has a budget attached,
When I view the record,
Then the budget snapshot is visible showing total budgeted, total actual, total difference, cost per person, and cost per day — with overall Under/Over/Close indicator — without needing to open the full budget.

**Edge Cases:**
- If multiple budgets are attached the snapshot shows each separately with its name
- Cost per person requires number of travelers to be set — if not set displays as incomplete
- Cost per day requires trip duration to be set — if not set displays as incomplete
- Snapshot is visible in presentation mode
- Snapshot reflects the most recently updated figures in real time

---

**US-022 — Save and Apply a Budget Template**
*When I've settled on a category structure that works for a type of trip, I want to save it as a template, so that I can start future budgets from a familiar structure without rebuilding it each time.*

**Priority:** P0

**Acceptance Criteria:**

Given I am viewing any Budget,
When I save it as a template with a name,
Then the template captures all category names and structure but not individual amounts or line item values, and is available as a starting point when creating any future budget.

**Edge Cases:**
- Applying a template populates category structure only — all cost columns start empty
- A template can be edited after saving — changes do not retroactively affect budgets already built from it
- Multiple templates can exist (e.g. "Sample Trip Budget", "Full Trip Budget")
- Deleting a template does not affect budgets already created from it
- Template names must be unique — duplicates are blocked
- If I apply a template to a budget that already has categories the app warns me before overwriting

---

## 5. Search & Discovery

---

**US-023 — Global Search Across All Content**
*When I want to find something across my entire library, I want to search by keyword across all content types, so that I can locate any piece of research regardless of where it lives in the hierarchy.*

**Priority:** P0

**Acceptance Criteria:**

Given I am anywhere in the app,
When I enter a keyword into global search,
Then results are returned across all content types (Countries, Regions, Cities, POIs, Trips, Tips, Budgets) ranked by relevance, with each result showing its content type and parent context (e.g. "Wat Phra That — POI — Chiang Mai — Thailand").

**Edge Cases:**
- Search returns results from all text fields including names, notes, and descriptions — not just titles
- If no results are found an empty state is shown with a prompt to refine the search
- Search results return instantly for typical library sizes — no loading state expected
- Single character searches are not blocked but may return broad results
- Search is case insensitive

---

**US-024 — Filter Global Search Results**
*When my search returns too many results, I want to filter by content type, country, tag, and status, so that I can narrow down to exactly what I'm looking for.*

**Priority:** P0

**Acceptance Criteria:**

Given I have results showing in global search,
When I apply one or more filters (content type, country, tag, status),
Then results update immediately to show only matching records and active filters are clearly displayed.

**Edge Cases:**
- Multiple filters can be applied simultaneously
- Filters are additive — Country: Italy AND Tag: Hiking returns only Italian hiking content
- Clearing a single filter does not clear all filters
- If a filter combination returns no results an empty state is shown with a suggestion to broaden filters
- Filter options only surface values that exist in the library — no orphaned filter options
- Filters persist during the same search session but reset when a new search is started

---

**US-025 — Browse Library by Country**
*When I want to explore what I have on a specific destination, I want to browse the full content hierarchy under a country, so that I can see all my research in one organized view.*

**Priority:** P0

**Acceptance Criteria:**

Given I am viewing the master library,
When I select a country,
Then I see all content nested under that country — regions, cities, POIs, trips, tips, and budgets — organized by the content hierarchy with content counts visible at each level.

**Edge Cases:**
- A country with no content beyond its name displays an empty state with prompts to start adding content
- Content counts are visible at each hierarchy level (e.g. "Northern Thailand — 3 Cities, 12 POIs")
- Collapsing and expanding hierarchy levels is supported for navigability
- Trips that reference multiple countries appear under each referenced country
- Filtering within a country view is supported — e.g. show only POIs tagged `Hiking` within Thailand

---

**US-026 — In-Context Search**
*When I'm adding locations or POIs to a trip itinerary, or navigating a large country record, I want to search within that context, so that I can find what I need without leaving my current flow.*

**Priority:** P0

**Acceptance Criteria:**

Given I am in the trip builder OR viewing a country, region, or city record,
When I trigger in-context search (via the search UI or Ctrl+F),
Then results are scoped to the current context — trip-relevant content when in the trip builder, and content within the current record when browsing a country/region/city.

**Edge Cases:**
- In the trip builder results are scoped to countries referenced in the trip by default with an option to expand to the full library
- In a country/region/city record search scopes to all content nested under that record including notes, POIs, tips, and sub-regions
- If the trip has no linked countries yet in-context search defaults to the full library
- Selecting a result in the trip builder adds it without navigating away
- Selecting a result in a country record navigates to or highlights that content within the record
- If a searched POI doesn't exist yet I can create it inline from trip builder search without fully leaving the builder
- Recently used POIs and locations surface at the top of trip builder search before a keyword is entered
- Ctrl+F behavior (app intercept vs. browser native) to be determined by UX best practice research before implementation

---

**US-027 — Filter Master Library by Status**
*When I want to see all destinations at a specific research stage, I want to filter my library by country or trip status, so that I can quickly see what needs attention or what's ready.*

**Priority:** P1

**Acceptance Criteria:**

Given I am viewing the master library,
When I filter by status,
Then only records matching the selected status are shown.

**Edge Cases:**
- Country and trip statuses are filtered independently
- Multiple statuses can be selected simultaneously
- Clearing the status filter restores the full library view
- If no records match the selected status an empty state is shown
- Status filter can be combined with other filters like country or tag

---

**US-028 — Filter Master Library by Tag**
*When I want to find all content matching a specific attribute, I want to filter the library by one or more tags, so that I can discover relevant content across destinations.*

**Priority:** P1

**Acceptance Criteria:**

Given I am viewing the master library or a country view,
When I apply one or more tag filters,
Then only content records carrying those tags are shown with each result displaying its content type and parent context.

**Edge Cases:**
- Tag filters are contextual — filtering by a POI tag only surfaces POIs
- Multiple tags can be applied simultaneously — results match all selected tags
- Tag filters can be combined with keyword search and status filters
- If no content matches the selected tags an empty state is shown
- Removing a tag filter immediately restores unfiltered results
- Filter options only surface tag values that exist in the library

---

**US-029 — Ctrl+F Search Trigger**
*When I want to search quickly without reaching for the mouse, I want Ctrl+F to trigger search in the most relevant context, so that I can find content efficiently using a familiar keyboard shortcut.*

**Priority:** P1

**Acceptance Criteria:**

Given I am anywhere in the app,
When I press Ctrl+F,
Then search is triggered in the most relevant context — in-context search if I am within a trip builder or country record, global search if I am in the master library or a neutral screen.

**Edge Cases:**
- Ctrl+F behavior (app intercept vs. browser native) to be determined by UX best practice research before implementation
- On mobile Ctrl+F is not applicable — search is triggered via search UI only
- The triggered search state is dismissable without losing current position in the app

**Open Decision:** Determine whether intercepting Ctrl+F is best practice for a web app or whether deferring to browser native find-in-page is the better convention. Implementation follows that finding.

---

## 6. Travel Windows

---

**US-030 — Create a Travel Window**
*When my wife and I decide on a travel period, I want to create a Travel Window with a target date, so that I have a named planning event to attach shortlisted trips to.*

**Priority:** P0

**Acceptance Criteria:**

Given I am in the master library or Travel Windows section,
When I create a Travel Window with at minimum a name and target travel period,
Then the Travel Window is created, appears in the Travel Windows section, and is immediately available to add shortlisted trips to.

**Edge Cases:**
- A Travel Window requires at minimum a name — target date is strongly recommended but not blocked if missing
- Multiple Travel Windows can exist simultaneously
- Travel Window names must be unique — duplicates are blocked
- A Travel Window can be created with no trips attached yet
- No tags are applied automatically

---

**US-031 — Add Shortlisted Trips to a Travel Window**
*When I have destination candidates for a travel period, I want to add trips from my master library to a Travel Window, so that I can view, compare, and present them as a shortlist.*

**Priority:** P0

**Acceptance Criteria:**

Given I am viewing a Travel Window,
When I search for and select a trip from the master library to add,
Then the trip is referenced in the Travel Window without being copied or removed from the master library, and is immediately available in both individual and comparison views.

**Edge Cases:**
- A Travel Window supports 2-3 shortlisted trips — adding a 4th prompts a warning but does not hard block
- The same trip cannot be added to the same Travel Window twice — duplicates are blocked
- A trip can be referenced in multiple Travel Windows simultaneously
- Removing a trip from a Travel Window does not delete it from the master library
- If a shortlisted trip is deleted from the master library the app warns me it is referenced in one or more Travel Windows before confirming deletion

---

**US-032 — View Trips Individually in a Travel Window**
*When I want to walk through each destination option one at a time, I want an individual trip view within a Travel Window, so that I can present or review each option with full detail before comparing.*

**Priority:** P0

**Acceptance Criteria:**

Given I am viewing a Travel Window with at least one shortlisted trip,
When I open individual view,
Then I can navigate through each shortlisted trip one at a time seeing the full trip concept, location blocks, budget snapshot, and photos, with clear navigation to move between trips.

**Edge Cases:**
- Individual view is read-only by default — editing is available per US-032b
- If only one trip is shortlisted individual view shows that trip with no next/previous navigation
- Navigation between trips is clear and intuitive — no explanation required for a non-user
- If a trip has no photos a placeholder is shown
- If a trip has no budget attached the budget section shows as not yet estimated

---

**US-032b — Edit a Trip from Within a Travel Window**
*When I need to make a change to a trip while reviewing it in a Travel Window, I want to edit it directly without navigating back to the master library, so that my planning flow is uninterrupted.*

**Priority:** P0

**Acceptance Criteria:**

Given I am viewing a trip within a Travel Window,
When I trigger edit mode,
Then the full trip record is editable from within the Travel Window view, and all changes are immediately reflected in the master library.

**Edge Cases:**
- Edits made from within a Travel Window are the same as edits made from the master library — there is no separate copy
- All content types within the trip are editable — location blocks, days, POIs, notes, budget, status
- Editing does not affect any existing snapshots — snapshots are only created on `Completed` status change
- Exiting edit mode returns me to the Travel Window view without losing position

---

**US-033 — Compare Shortlisted Trips in a Travel Window**
*When I'm deciding between destination options, I want to see all shortlisted trips side by side, so that I can evaluate them together in a single view.*

**Priority:** P0

**Acceptance Criteria:**

Given I am viewing a Travel Window with at least two shortlisted trips,
When I switch to comparison view,
Then all shortlisted trips are displayed side by side showing trip concept, location block summary, budget snapshot, and photos for each option.

**Edge Cases:**
- Comparison view is read-only — no editing controls visible
- If a shortlisted trip has no budget attached its budget column shows as not yet estimated
- If a shortlisted trip has no photos a placeholder is shown
- Comparison view adjusts layout appropriately for desktop and mobile screen sizes
- If only one trip is shortlisted comparison view is accessible but notes that only one option is present

---

**US-034 — Select a Trip from a Travel Window**
*When we've made a decision on our next destination, I want to mark the chosen trip in the Travel Window, so that the decision is recorded while all options remain accessible and editable.*

**Priority:** P0

**Acceptance Criteria:**

Given I am viewing a Travel Window with shortlisted trips,
When I mark one trip as the chosen destination,
Then that trip is visually flagged as selected, the other trips are visually marked as not chosen, and all trips remain fully accessible and editable from within the Travel Window.

**Edge Cases:**
- Marking a trip as chosen does not alter its status — status remains manual
- Marking a trip as chosen does not remove or hide other shortlisted trips from the Travel Window
- The selection can be changed — choosing a different trip updates the chosen flag accordingly
- Unchosen trips remain fully intact in the master library with no changes
- A Travel Window with no selection made is valid — the chosen flag is optional
- Edits made to a trip from within the Travel Window view are reflected in the master library immediately

---

**US-034b — Snapshot a Trip on Completion**
*When a trip is marked as Completed, I want a snapshot of that trip automatically captured, so that I have a permanent historical record of the trip exactly as it was when we finished it.*

**Priority:** P0

**Acceptance Criteria:**

Given a trip exists within a Travel Window and its status is changed to `Completed`,
When the status change is saved,
Then a point-in-time snapshot of the trip is automatically captured and stored within the Travel Window, preserving the full trip state at that moment — location blocks, days, POIs, notes, budget, and photos.

**Edge Cases:**
- The snapshot captures the complete trip state at the moment `Completed` status is set
- Future edits to the live trip after completion do not alter the snapshot
- The snapshot is clearly labeled with the date it was captured
- The snapshot is read-only — it cannot be edited
- If a trip is marked Completed outside of a Travel Window no snapshot is captured — snapshots only apply to trips within a Travel Window
- If a trip's status is moved back from `Completed` to a previous status the snapshot is retained but clearly marked as superseded — it is not automatically deleted

---

**US-035 — Reference Past Travel Windows**
*When I want to look back at a previous planning decision, I want to browse past Travel Windows and see trips exactly as they were when completed, so that I have an accurate historical record of what we experienced.*

**Priority:** P0

**Acceptance Criteria:**

Given I am viewing a past Travel Window that contains a Completed trip with a snapshot,
When I browse its contents,
Then I can toggle between the snapshot view (trip as it was when completed) and the live view (trip as it is today), with a clear visual indicator of which mode I am in.

**Edge Cases:**
- Snapshot view is always read-only — no editing from within snapshot view
- Live view remains fully editable from within the Travel Window
- If no trip in the Travel Window has been marked Completed no snapshot exists — only live view is available
- If a live trip has been deleted after a snapshot was taken the snapshot still displays the preserved state clearly labeled as a historical record
- The toggle between snapshot and live is clearly labeled so it is never ambiguous which version is being viewed
- Photos included in the snapshot are preserved even if later removed from the live trip
- Non-completed trips in the same Travel Window show live view only with no snapshot toggle
- Travel Windows are organized chronologically and persist indefinitely — they are never automatically deleted

---

**US-036 — Archive a Travel Window**
*When my active Travel Windows list gets long, I want to archive older ones, so that my active planning view stays focused without losing historical reference.*

**Priority:** P1

**Acceptance Criteria:**

Given I am viewing a Travel Window,
When I archive it,
Then it moves out of the active Travel Windows view into an archived section, all referenced trips remain intact, and the Travel Window remains fully accessible from the archived view.

**Edge Cases:**
- Archiving does not affect any trips referenced within the Travel Window
- An archived Travel Window can be restored to active at any time
- Archived Travel Windows remain fully navigable in both individual and comparison view
- Deleting a Travel Window is a separate explicit action from archiving and requires confirmation
- Deleting a Travel Window does not delete any referenced trips from the master library

---

## 7. Presentation Mode

---

**US-037 — Enter Presentation Mode from a Travel Window**
*When my wife and I sit down to decide on our next trip, I want to launch presentation mode from a Travel Window, so that we can review our options in a clean focused view without app chrome or editing UI.*

**Priority:** P0

**Acceptance Criteria:**

Given I am viewing a Travel Window with at least one shortlisted trip,
When I launch presentation mode,
Then the view switches to a full clean display showing all shortlisted trips with all navigation, editing controls, and app chrome hidden, and the interface is immediately intuitive to someone who has never used the app.

**Edge Cases:**
- Presentation mode is accessible from any Travel Window regardless of how many trips are shortlisted
- Entering presentation mode does not alter any data or statuses
- Presentation mode is dismissable at any time returning me to exactly where I was in the Travel Window
- If only one trip is shortlisted presentation mode shows that single trip with no comparison navigation
- Presentation mode is accessible on both desktop and mobile with layout adjusting appropriately

---

**US-038 — View Individual Trip in Presentation Mode**
*When presenting a destination to my wife, I want to walk through each option one at a time, so that we can focus on one destination fully before moving to the next.*

**Priority:** P0

**Acceptance Criteria:**

Given I am in presentation mode within a Travel Window,
When I navigate to an individual trip view,
Then the trip is displayed showing trip name, concept/description, location blocks, sample itinerary, budget snapshot, and photos in a clean visually appealing layout with simple next/previous navigation between trips.

**Edge Cases:**
- Individual trip view is read-only — no editing controls visible
- Navigation between trips is simple and obvious — no explanation required for a non-user
- If a trip has no photos a clean placeholder is shown — no broken image states
- If a trip has no budget attached the budget section shows as not yet estimated
- If a trip has no itinerary detail beyond location blocks those blocks are shown as the itinerary
- Rich text formatting in notes and descriptions renders correctly in presentation mode

---

**US-039 — View Comparison in Presentation Mode**
*When we want to evaluate all options together, I want to switch to a side by side comparison view in presentation mode, so that we can directly contrast destinations during our decision conversation.*

**Priority:** P0

**Acceptance Criteria:**

Given I am in presentation mode with at least two shortlisted trips,
When I switch to comparison view,
Then all shortlisted trips are shown side by side with consistent information displayed for each — trip concept, location summary, budget snapshot, vibe tags, and a hero photo.

**Edge Cases:**
- Comparison view is read-only — no editing controls visible
- All trips display the same information fields for consistency — missing fields show as not yet added rather than hidden
- Comparison view layout adjusts for screen size — on mobile trips may stack vertically
- If only one trip is shortlisted comparison view is not available — individual view is shown with a note that only one option exists
- Vibe tags are only shown if at least one trip has tags applied — if no trips have tags the tag row is hidden entirely

---

**US-040 — Display Budget Snapshot in Presentation Mode**
*When discussing destination options, I want the budget snapshot visible in presentation mode, so that cost is part of the conversation without needing to open the full budget.*

**Priority:** P0

**Acceptance Criteria:**

Given I am in presentation mode viewing a trip or comparison,
When the budget snapshot is displayed,
Then it shows total estimated cost, cost per person, and cost per day clearly and legibly, with the Under/Over/Close indicator visible if actuals have been entered.

**Edge Cases:**
- If no budget is attached the snapshot area shows "Budget not yet estimated"
- Cost per person and cost per day only display if number of travelers and trip duration are set
- Budget snapshot in comparison view is consistently positioned across all trips
- Full budget detail is not shown in presentation mode — snapshot only
- If multiple budgets are attached the most recently updated budget snapshot is shown with an indicator that more detail is available

---

**US-041 — Display Photos in Presentation Mode**
*When presenting a destination, I want photos visible as a supporting visual element, so that my wife gets a feel for the place without photos overwhelming the informational content.*

**Priority:** P1

**Acceptance Criteria:**

Given I am in presentation mode viewing a trip or country,
When photos are attached to the trip or its linked content,
Then photos are displayed as a supporting visual element — present and attractive but not the dominant focus of the layout.

**Edge Cases:**
- If no photos are attached a clean neutral placeholder is shown
- The hero image is the designated hero photo if set, otherwise the first available photo from the trip or its linked content
- Photos from any level of the content hierarchy are eligible — trip, country, region, city, or POI
- On mobile photos scale appropriately without distorting the layout

---

**US-042 — Navigate and Exit Presentation Mode**
*When presenting destination options, I want intuitive keyboard and accessibility controls, so that I can navigate the presentation smoothly without reaching for the mouse.*

**Priority:** P0

**Acceptance Criteria:**

Given I am in presentation mode,
When I use keyboard or accessibility controls,
Then the following behaviors are supported:
- **Escape** — exits presentation mode and returns to the Travel Window
- **Arrow keys (left/right)** — navigate between shortlisted trips in individual view
- **Arrow keys (up/down)** — scroll within a trip's full detail view
- **Enter** — opens full trip detail from the summary view
- **Backspace** — returns from full trip detail back to the presentation summary
- **Tab** — cycles through interactive elements within the current view

**Edge Cases:**
- Exit control is always visible on screen but unobtrusive
- Keyboard shortcuts are consistent and do not conflict with browser or OS defaults where possible — conflicts deferred to best practice research
- On mobile navigation is touch/swipe based
- All keyboard navigation is consistent with WCAG accessibility standards
- A visible keyboard shortcut reference is accessible from within presentation mode without disrupting the presentation
- Exiting presentation mode does not trigger any status changes or data updates
- If presentation mode is accidentally exited re-entering returns to the beginning of the presentation

---

**US-043 — Access Full Trip Detail from Presentation Mode**
*When our conversation goes deep on a specific destination, I want to access the full trip detail from within presentation mode, so that I can answer questions and explore specifics without breaking the presentation flow.*

**Priority:** P0

**Acceptance Criteria:**

Given I am in presentation mode viewing a trip,
When I trigger the full detail view for that trip,
Then the complete trip record is shown including all location blocks, day-by-day itinerary, full POI details, tips, and complete budget — accessible from within presentation mode without fully exiting it.

**Edge Cases:**
- Full detail view is accessible from both individual trip view and comparison view
- Full detail view within presentation mode is read-only — editing requires explicitly exiting presentation mode
- Navigating into full detail and back to the presentation summary is seamless — no loss of position
- If accessed from comparison view returning from full detail brings me back to the comparison view
- All rich text formatting renders correctly in full detail view
- On mobile full detail view scrolls naturally without breaking the presentation mode context

---

## 8. Tips & Lessons Learned

---

**US-050 — Create a General Tip**
*When I want to capture universal travel knowledge not tied to any destination, I want to create a General Tip, so that it's stored and retrievable when planning any future trip.*

**Priority:** P1

**Acceptance Criteria:**

Given I am anywhere in the app,
When I create a new Tip and set its scope to General,
Then the tip is saved in the General Tips library with no destination link required, and is immediately searchable and browsable.

**Edge Cases:**
- A General Tip requires only a name/title — all other fields are optional
- Rich text formatting is supported per US-007b
- No tags are applied automatically
- General Tips are accessible from a dedicated Tips section as well as via global search

---

**US-051 — Create a Destination-Specific Tip**
*When I want to capture knowledge about a specific destination, I want to create a Tip linked to a country, region, or city, so that it surfaces when I'm researching or planning for that destination.*

**Priority:** P1

**Acceptance Criteria:**

Given I am viewing a country, region, or city record OR creating a tip from anywhere in the app,
When I create a new Tip and set its scope to Destination-specific with a linked destination,
Then the tip is saved and appears within the linked destination's record as well as in the global Tips section.

**Edge Cases:**
- A destination-specific tip requires a destination link — if scope is set to Destination-specific the app prompts for a destination before saving
- A tip can be linked to any level of the hierarchy — country, region, or city
- If created from within a destination record the destination link is pre-populated automatically
- If the linked destination is deleted the tip is not deleted — it moves to an Unlinked state and is flagged for reassignment
- Rich text formatting is supported per US-007b
- No tags are applied automatically

---

**US-052 — Create a Lesson Learned**
*When I want to capture something I discovered from travel experience, I want to create a Lesson Learned with either a general or destination-specific scope, so that real experience is distinguished from pre-trip research regardless of whether it applies to one place or everywhere.*

**Priority:** P1

**Acceptance Criteria:**

Given I am anywhere in the app,
When I create a new Tip, set its type to Lesson Learned, and set its scope to either General or Destination-specific,
Then it is saved and clearly distinguished from standard Tips in both the relevant destination record (if destination-specific) or General Tips section (if general), with an optional link to a Completed Trip as its source.

**Edge Cases:**
- A General Lesson Learned requires no destination link — scope is explicitly global
- A Destination-specific Lesson Learned requires a destination link — the app prompts for one before saving
- The link to a Completed Trip is optional for both General and Destination-specific Lesson Learned
- A Lesson Learned can only be linked to a trip with `Completed` status — linking to an active trip is not permitted
- If the linked Completed Trip is deleted the Lesson Learned is not deleted — the trip link is removed and flagged
- Lesson Learned and Tip are visually distinguishable in all views regardless of scope
- Rich text formatting is supported per US-007b
- No tags are applied automatically
- Converting a General Lesson Learned to Destination-specific requires adding a destination link

---

**US-053 — Browse Tips and Lessons Learned by Destination**
*When I'm researching or planning for a destination, I want to see all relevant tips and lessons learned within that destination's record, so that accumulated knowledge surfaces exactly where I need it.*

**Priority:** P1

**Acceptance Criteria:**

Given I am viewing a country, region, or city record,
When I navigate to the Tips section of that record,
Then all destination-specific tips and lessons learned linked to that record are displayed, clearly separated by type, with the most recently added surfaced first by default.

**Edge Cases:**
- Tips linked to a child record surface in the parent — a tip linked to Chiang Mai appears when viewing Thailand with its specific city context visible
- If no tips exist for a destination an empty state is shown with a prompt to add the first tip
- Tips and Lessons Learned are visually distinct from each other in the list
- Sorting and filtering within the tips section is supported — by type, date added, and destination level
- General Tips do not appear in destination-specific tip views

---

**US-054 — Browse General Tips**
*When I want to review universal travel knowledge, I want a dedicated General Tips section, so that I can browse and reference advice that applies across all destinations.*

**Priority:** P1

**Acceptance Criteria:**

Given I am in the Tips section of the app,
When I filter to General Tips,
Then only tips with General scope are shown, organized by date added with the most recent first by default.

**Edge Cases:**
- General Tips are clearly separated from destination-specific tips
- Sorting and filtering within General Tips is supported — by date added and keyword search
- If no General Tips exist an empty state is shown with a prompt to add the first one
- General Tips are also surfaced in global search results

---

**US-055 — Convert a Tip to a Lesson Learned**
*When a tip I captured before visiting a destination turns out to be confirmed or contradicted by real experience, I want to convert it or add a Lesson Learned follow-up, so that my knowledge base reflects reality not just research.*

**Priority:** P1

**Acceptance Criteria:**

Given I am viewing a destination-specific Tip,
When I convert it to a Lesson Learned or add a Lesson Learned as a follow-up,
Then the original tip is preserved and the Lesson Learned is linked to it, clearly showing the evolution from pre-trip research to post-trip reality.

**Edge Cases:**
- Converting a Tip to a Lesson Learned prompts for an optional Completed Trip link
- The original Tip content is preserved — conversion is non-destructive
- If added as a follow-up both the Tip and the Lesson Learned exist independently but are visually linked
- A Lesson Learned follow-up inherits the destination link of the original Tip
- A General Tip can be converted to a General Lesson Learned without requiring a destination link

---

## 9. Tagging & Organization

---

**US-056 — Tag a Country Record**
*When I'm organizing my library, I want to apply tags to a country, so that I can filter and discover destinations by their attributes.*

**Priority:** P1

**Acceptance Criteria:**

Given I am creating or editing a Country record,
When I open the tag selector,
Then I see the country-specific tag categories (Region/Continent, Entry Requirements, Language Barrier, Safety) and can select one or more values from each category.

**Edge Cases:**
- Tags are never applied automatically — always manual
- Multiple tags from the same category can be applied
- Tags can be removed at any time without affecting any other content on the record
- No tags are required — a country record is valid with zero tags
- Tag selections are saved immediately without requiring a separate save action

---

**US-056b — Tag a Region Record**
*When I'm organizing destination research, I want to apply tags to a region, so that I can describe its character and filter by it when planning.*

**Priority:** P1

**Acceptance Criteria:**

Given I am creating or editing a Region record,
When I open the tag selector,
Then I see region-specific tag categories (Region Type, Safety, Language Barrier) and can select one or more values from each category.

**Edge Cases:**
- Tags are never applied automatically — always manual
- Region tags are independent of parent country tags — tagging a region does not affect the country record and vice versa
- Multiple values can be applied within the same category
- Tags can be removed at any time without affecting any other content on the record
- No tags are required — a region is valid with zero tags

---

**US-056c — Tag a City Record**
*When I'm capturing detail on a city, I want to apply tags to describe its character and size, so that I can filter and discover cities by type when building a trip.*

**Priority:** P1

**Acceptance Criteria:**

Given I am creating or editing a City record,
When I open the tag selector,
Then I see city-specific tag categories (City Type, Size, Vibe) and can select one or more values from each category.

**Edge Cases:**
- Tags are never applied automatically — always manual
- City tags are independent of parent region and country tags
- Multiple values can be applied within the same category
- Tags can be removed at any time without affecting POIs or trips that reference the city
- No tags are required — a city is valid with zero tags

---

**US-057 — Tag a POI Record**
*When I'm capturing a point of interest, I want to apply tags to a POI, so that it's discoverable by type, price range, and priority when building a trip.*

**Priority:** P1

**Acceptance Criteria:**

Given I am creating or editing a POI record,
When I open the tag selector,
Then I see POI-specific tag categories (POI Type, Price Range, Must-Do Status) and can select one or more values.

**Edge Cases:**
- Tags are never applied automatically — always manual
- A POI can have multiple POI Type tags (e.g. both `Museum` and `Historic Site`)
- Must-Do Status is a single select — only one value applies at a time
- Tags can be removed at any time without affecting the POI record or any trips it is referenced in
- No tags are required — a POI is valid with zero tags

---

**US-058 — Tag a Trip Record**
*When I'm describing what kind of trip a plan represents, I want to apply tags to a trip, so that it's filterable by vibe, activity type, budget tier, and pace.*

**Priority:** P1

**Acceptance Criteria:**

Given I am creating or editing a Trip record,
When I open the tag selector,
Then I see trip-specific tag categories (Vibe, Activity Type, Budget Tier, Trip Pace, Best Season) and can select one or more values from each category.

**Edge Cases:**
- Tags are never applied automatically — always manual
- Multiple tags from the same category can be applied
- Tags applied to a trip do not automatically propagate to linked POIs or countries
- Tags can be removed at any time without affecting the trip record or any Travel Windows it belongs to
- No tags are required — a trip is valid with zero tags

---

**US-059 — Filter Library by Tag**
*When I want to find all content matching a specific attribute, I want to filter my library by one or more tags, so that I can discover relevant content across destinations quickly.*

**Priority:** P1

**Acceptance Criteria:**

Given I am viewing the master library or a country record,
When I apply one or more tag filters,
Then only content records carrying those tags are shown with each result displaying its content type and parent context.

**Edge Cases:**
- Tag filters are contextual to content type — filtering by `Must Do` only surfaces POIs
- Multiple tags can be applied simultaneously — results match all selected tags
- Tag filters can be combined with keyword search and status filters
- If no content matches the selected tags an empty state is shown
- Removing a tag filter immediately restores unfiltered results
- Filter options only surface tag values that exist in the library

---

**US-060 — Filter POIs by Tag Within a Trip**
*When I'm building a trip and looking for the right POIs to add, I want to filter available POIs by tag within the in-context search, so that I can quickly surface the most relevant options.*

**Priority:** P1

**Acceptance Criteria:**

Given I am in the trip builder using in-context search to find POIs,
When I apply tag filters (POI Type, Price Range, Must-Do Status),
Then only POIs matching the selected tags are shown in the search results within the current trip context.

**Edge Cases:**
- Tag filters in in-context search stack with keyword search and geographic scoping
- Clearing tag filters within in-context search restores the full scoped results
- If no POIs match the combined filters an empty state is shown with an option to expand scope or clear filters
- Tag filter selections within in-context search do not persist after closing the search

---

**US-061 — Manage Fixed Tag Values**
*When the default tag set doesn't fully reflect how I think about destinations, I want to add new values to existing tag categories, so that my tagging system evolves with my travel knowledge.*

**Priority:** P2

**Acceptance Criteria:**

Given I am in app settings or tag management,
When I add a new value to an existing tag category,
Then the new value appears in the tag selector for the relevant content type and is immediately available to apply.

**Edge Cases:**
- Tag value names must be unique within the same category — duplicates are blocked
- Deleting a tag value that is currently applied to records prompts a warning — the tag is removed from all affected records if confirmed
- Renaming a tag value updates it across all records where it is applied
- Default tag values cannot be deleted — only custom added values can be removed
- Tag management is accessible from settings not from within content records

---

**US-062 — Add New Tag Categories**
*When my tagging needs grow beyond the default categories, I want to create entirely new tag categories for a content type, so that my organization system reflects how I actually think about travel.*

**Priority:** P2

**Acceptance Criteria:**

Given I am in tag management settings,
When I create a new tag category and assign it to one or more content types,
Then the new category appears in the tag selector for the assigned content types and accepts new values.

**Edge Cases:**
- A new tag category requires a name and at least one content type assignment before saving
- Category names must be unique across all content types — duplicates are blocked
- A custom category can be assigned to multiple content types
- Deleting a custom category that has values applied to records prompts a warning — all applied tags in that category are removed from affected records if confirmed
- Default tag categories cannot be deleted — only custom categories can be removed
- Custom categories are visually distinguishable from default categories in the tag selector

---

## 10. Mobile & Offline

---

**US-044 — Quick Capture a Content Stub on Mobile**
*When I'm on the go and want to capture a destination idea or POI immediately, I want a fast mobile quick-capture flow, so that the idea is saved before it's forgotten without requiring me to fill out a full record.*

**Priority:** P2

**Acceptance Criteria:**

Given I am on mobile anywhere in the app,
When I trigger quick capture,
Then I am presented with a minimal form asking only for content type and name, and the record is saved as a stub in the master library in under 3 taps.

**Edge Cases:**
- Quick capture is accessible from any screen on mobile — never more than one tap away
- All content types are available via quick capture
- If no parent context is specified the stub is saved to an Unassigned inbox for later organization
- Quick capture does not require an internet connection — stubs created offline are queued and synced when connectivity is restored
- If a duplicate name is detected the app warns before saving but does not hard block
- No tags are applied automatically

---

**US-045 — Access Full Library on Mobile**
*When I'm away from my desktop, I want to browse and edit my full library on mobile, so that I'm not limited to quick capture and can do lightweight research or planning from my phone.*

**Priority:** P2

**Acceptance Criteria:**

Given I am on mobile with the app open,
When I browse the master library,
Then the full content hierarchy is accessible and navigable with a mobile-optimized layout.

**Edge Cases:**
- All content types are accessible on mobile — no content is desktop-only
- Editing any record is supported on mobile
- Rich text editor is functional on mobile
- Search and filtering are fully functional on mobile
- Large records with extensive notes scroll naturally without performance degradation

---

**US-046 — Access Trip Itinerary on Mobile While Traveling**
*When I'm in-destination and need to know what's next, I want fast access to my active trip itinerary on mobile, so that I can reference the day's plan, addresses, and POI details without digging through the app.*

**Priority:** P2

**Acceptance Criteria:**

Given I am on mobile viewing an active trip,
When I open the itinerary view,
Then the day-by-day plan is immediately visible with the current day surfaced prominently, showing all POIs, addresses, practical notes, and free-form notes for that day in a clean readable layout.

**Edge Cases:**
- The current day is surfaced automatically based on device date if it falls within the trip's date range — otherwise Day 1 is shown by default
- All POI details are accessible with one tap
- Itinerary view is fully functional offline
- If the trip has no specific dates set the itinerary displays from Day 1
- Photos attached to POIs are accessible but may load on-demand if offline storage is optimized in the future

---

**US-047 — Use App Fully Offline on Mobile**
*When I'm traveling with poor or no connectivity, I want the full library available offline, so that I can reference any research or itinerary detail without needing an internet connection.*

**Priority:** P2

**Acceptance Criteria:**

Given I have previously opened the app with an internet connection,
When I open the app with no internet connection,
Then the full library is available including all countries, regions, cities, POIs, trips, tips, budgets, and photos, with a clear indicator that I am in offline mode.

**Edge Cases:**
- All content is readable offline — no content type is excluded
- Edits made offline are queued and synced automatically when connectivity is restored
- If an offline edit conflicts with a change made on another device the app flags the conflict and prompts resolution — no silent overwrites
- The offline indicator is visible but unobtrusive
- Photos are included in offline storage by default — optimization to text-only offline is a known future lever

---

**US-048 — Sync Changes Across Devices**
*When I switch between my phone and laptop during planning, I want changes synced automatically across devices, so that my library is always up to date regardless of which device I used last.*

**Priority:** P2

**Acceptance Criteria:**

Given I have made changes on one device,
When I open the app on another device with internet connectivity,
Then all changes are reflected automatically with no manual sync action required.

**Edge Cases:**
- Sync happens in the background — no loading screen or manual trigger required
- If connectivity is lost mid-sync the app completes the sync when connectivity is restored without data loss
- If a sync conflict exists between two devices the app flags it clearly and prompts resolution — no silent overwrites
- Large photo uploads may sync separately from text content

---

**US-049 — Offline Indicator and Sync Status**
*When I'm offline or syncing, I want a clear but unobtrusive status indicator, so that I always know the current state of my data without it interrupting my workflow.*

**Priority:** P2

**Acceptance Criteria:**

Given I am using the app in any state (online, offline, or syncing),
When the connectivity or sync state changes,
Then a visible but unobtrusive indicator reflects the current state — Online, Offline, Syncing, or Sync Failed.

**Edge Cases:**
- The indicator never blocks content or interactions — it is informational only
- A Sync Failed state provides a clear explanation and a manual retry option
- If sync has been failing silently for an extended period the app surfaces a more prominent warning
- The indicator is accessible on both mobile and desktop

---

## Story Count Summary

| Section | Stories | P0 | P1 | P2 |
|---|---|---|---|---|
| Content Creation | 6 | 5 | 1 | 0 |
| Content Enrichment & Editing | 5 | 3 | 2 | 0 |
| Trip Building | 6 | 6 | 0 | 0 |
| Budget | 5 | 5 | 0 | 0 |
| Search & Discovery | 7 | 4 | 3 | 0 |
| Travel Windows | 8 | 7 | 1 | 0 |
| Presentation Mode | 7 | 6 | 1 | 0 |
| Tips & Lessons Learned | 6 | 0 | 6 | 0 |
| Tagging & Organization | 9 | 0 | 7 | 2 |
| Mobile & Offline | 6 | 0 | 0 | 6 |
| **Total** | **65** | **36** | **21** | **8** |