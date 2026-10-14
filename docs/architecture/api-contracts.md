# Wanderly — API Contracts

## Global Conventions

**Transport:** Electron IPC via contextBridge. All contracts are expressed in REST
style for readability. Each operation is one `ipcMain.handle` channel, named on the
`Channel:` line directly under its heading.

**IPC channel convention:** `<module>:<resource>:<action>`
e.g. `content:country:create`, `trip:block:expand`, `search:global:query`

**Reading a contract as an IPC call:**

- **Payload.** Each channel takes one object. Path parameters, query parameters and the
  request body are merged into it under the names shown. `PATCH /api/countries/:id`
  with body `{ name }` is the channel `content:country:update` with payload `{ id, name }`.
- **Result envelope.** Every channel returns one of:

  ```
  { ok: true,  data: <the Response 2xx body shown> }
  { ok: false, error: { code: string, message: string, ...context_fields } }
  ```

  Handlers return errors as values and never throw across IPC, because Electron would
  drop everything except the message. The renderer's IPC client unwraps the envelope
  and throws a typed `ApiError` carrying the full error object.
- **Status numbers.** `Response 200 / 201` means `ok: true`. `Response 4xx` means
  `ok: false`; the number is a reading aid only and is not transmitted — the renderer
  branches on `error.code`.
- **Confirm steps.** Where a contract shows a second `.../confirm` endpoint, it is the
  **same channel** called again with `confirm: true` added to the payload. The first
  call returns a confirmation-required error and changes nothing; the renderer shows
  the preview, and on acceptance repeats the call with `confirm: true`. A call that
  needs no confirmation succeeds the first time.
- **Events (main → renderer).** A few long operations report progress. The main
  process sends them with `webContents.send` on a channel named
  `<module>:<resource>:<event>`, and the preload script exposes a typed `on(channel,
  listener)` that returns an unsubscribe function. Events carry data only; they never
  replace the operation's own result. Each is listed under the operation that emits it
  as `Emits:`.

**Authentication:** None — single-user local app. Omitted from all endpoints.

**IDs:** All record IDs are cuid2 strings.

**Timestamps:** All timestamps are ISO 8601 strings.

**Rich text fields:** All `notes`, `description`, `content`, and `practical_notes`
fields store and return TipTap JSON strings unless explicitly noted otherwise.

**Error object shape** (universal across all endpoints; delivered inside the envelope):

```
{ error: { code: string, message: string, ...context_fields } }
```

**Error codes** (complete list — an endpoint may only return codes listed here):

- `VALIDATION_ERROR` — missing or invalid field in request
- `NOT_FOUND` — record does not exist
- `DUPLICATE_NAME` — unique name constraint violated
- `DUPLICATE_LINK` — many-to-many link already exists
- `DUPLICATE_TAG` — tag already applied to this record
- `DUPLICATE_TRIP` — trip already shortlisted in this Travel Window
- `HIERARCHY_MISMATCH` — geographic FK inconsistency
- `TYPE_MISMATCH` — tag applied to wrong content type
- `RECORD_IN_USE` — delete refused because other records depend on this one; cannot be confirmed past
- `RECORD_IS_DELETED` — operation not permitted on a preserved historical record
- `INVALID_TRIP_STATUS` — operation needs a trip in a different status (e.g. Lesson Learned source must be Completed)
- `ALREADY_EXPANDED` — block already has days
- `ALREADY_ARCHIVED` — Travel Window is already archived
- `NOT_ARCHIVED` — Travel Window is not archived
- `EMPTY_WINDOW` — Travel Window has no shortlisted trips
- `INSUFFICIENT_TRIPS` — comparison view needs at least two trips
- `FILE_NOT_FOUND` — source file missing at the given path
- `FILE_PROCESSING_ERROR` — image could not be read or processed
- `BACKUP_FOLDER_UNAVAILABLE` — no backup folder configured, or it cannot be reached
- `INVALID_BACKUP` — restore point is damaged or was made by a newer app version
- `INTERNAL_ERROR` — unexpected failure; details are in the log file, not the response

**Confirmation-required codes** (returned when `confirm` is absent; repeat the call
with `confirm: true` to proceed):

- `DELETE_REQUIRES_CONFIRMATION` — destructive delete; carries a preview of what will be affected
- `DURATION_CONFLICT` — reducing a block's duration would remove days
- `TEMPLATE_OVERWRITE_CONFLICT` — applying a template would replace existing categories
- `SHORTLIST_LIMIT_WARNING` — adding a fourth or later trip to a Travel Window
- `DUPLICATE_NAME_WARNING` — creating a POI whose name already exists under the same parent
- `RESTORE_REQUIRES_CONFIRMATION` — restoring a backup replaces the current library

---

## Module 1 — Content (Countries, Regions, Cities, POIs)

**IPC prefix:** `content`

---

### Countries

```
GET /api/countries
Channel: content:country:list
Query params:  none — countries have no status to filter by (D-09); tag filtering
               on this list goes through GET /api/search (filter-only browsing)
Response 200:
  {
    countries: [
      {
        id:           string,
        name:         string,
        pin_position: number | null,   -- null = not pinned
        created_at:   string,
        updated_at:   string,
        counts: {
          regions: number,
          cities:  number,
          pois:    number,
          trips:   number
        },
        tags:          [{ id: string, category: string, value: string }],
        hero_thumbnail_url: string | null
          -- the country's designated hero, else its first photo by position;
          -- null = Grid view shows the initial on a neutral block (D-24)
      }
    ]
  }
Side effects: none
  -- Order: pinned countries first by pin_position, then the rest newest first
     (created_at descending) — the side navigation's order (FR14). The Countries
     page's other sorts (Name A–Z, Most POIs) are applied by the renderer to this
     list; every country is returned, so no paging is needed
```

```
GET /api/countries/:id
Channel: content:country:get
Response 200:
  {
    id:           string,
    name:         string,
    pin_position: number | null,
    notes:        string | null,   -- TipTap JSON
    created_at: string,
    updated_at: string,
    counts: {
      regions: number,
      cities:  number,
      pois:    number,
      trips:   number,
      tips:    number,
      budgets: number
    }
  }
Response 404: { error: { code: "NOT_FOUND", message: "Country not found" } }
Side effects: none
```

```
POST /api/countries
Channel: content:country:create
Request body:
  {
    name:   string  required  -- non-empty, unique across all countries
    notes:  string?           -- TipTap JSON
  }
Response 201:
  {
    id:           string,
    name:         string,
    pin_position: null,
    notes:        string | null,
    created_at:   string,
    updated_at:   string
  }
Response 400: name is empty
  { error: { code: "VALIDATION_ERROR", message: "Name is required" } }
Response 409: country with this name already exists
  { error: { code: "DUPLICATE_NAME", message: "A country named '{name}' already exists", existing_id: string } }
Side effects: search_index updated synchronously
```

```
PATCH /api/countries/:id
Channel: content:country:update
Request body: (all fields optional — only provided fields updated)
  {
    name:   string?
    notes:  string?  -- TipTap JSON
  }
  -- Pinning is changed only through PUT /api/countries/:id/pin
Response 200: same shape as GET /api/countries/:id
Response 400: name is empty string
  { error: { code: "VALIDATION_ERROR", message: "Name cannot be empty" } }
Response 404: { error: { code: "NOT_FOUND", message: "Country not found" } }
Response 409: name conflicts with existing country
  { error: { code: "DUPLICATE_NAME", message: "A country named '{name}' already exists", existing_id: string } }
Side effects: search_index updated synchronously
```

```
DELETE /api/countries/:id
Channel: content:country:delete
Response 409: country is used by trip location blocks — delete refused
  {
    error: {
      code: "RECORD_IN_USE",
      message: "This country is used in trip itineraries. Remove it from those trips first.",
      blocking_trips: [{ id: string, name: string, status: string, blocks: number }]
    }
  }
  -- Checked first. There is no confirm path past this response.
Response 409: delete would affect other records — confirm required
  {
    error: {
      code: "DELETE_REQUIRES_CONFIRMATION",
      message: "This country has nested content that will be deleted",
      cascade_preview: {
        regions:             number,
        cities:              number,
        pois:                number,
        media:               number,
        tips:                number,  -- tips linked to the country or anything beneath it; unlinked, not deleted
        trip_links:          number,  -- trips that list this country without a block for it
        trip_day_references: number,  -- trip-day entries for POIs in this country
        affected_trips:      [{ id: string, name: string }]  -- trips losing a link or a day entry
      }
    }
  }
Response 200: (every cascade_preview count is zero — no confirm needed)
  {
    deleted_id: string,
    cascade_summary: {
      regions_deleted:             number,
      cities_deleted:              number,
      pois_deleted:                number,
      media_deleted:               number,
      tips_unlinked:               number,
      trip_links_removed:          number,
      trip_day_references_removed: number
    }
  }
Response 404: { error: { code: "NOT_FOUND", message: "Country not found" } }
Side effects: none until confirmed
```

```
DELETE /api/countries/:id/confirm
Channel: content:country:delete  + confirm: true
Response 200: same shape as DELETE /api/countries/:id success response
Response 404: { error: { code: "NOT_FOUND", message: "Country not found" } }
Response 409: RECORD_IN_USE — same shape as DELETE /api/countries/:id
Side effects:
  -- Regions, cities, POIs and their media and tag assignments hard deleted
  -- Tips linked to the country or anything beneath it become Unlinked:
     destination FK nulled, former_destination set, scope retained
  -- trip_countries links for this country removed
  -- trip_day_pois rows for the deleted POIs removed
  -- search_index entries for all deleted records removed synchronously
  -- If the country was pinned, the pinned positions after it close up
```

```
PUT /api/countries/:id/pin
Channel: content:country:pin
  -- Pins, unpins or moves a country within the pinned order (FR14, US-025):
  -- "Pin to sidebar" / "Unpin" in More ▾, drag in the Pinned group, Alt+Up/Down.
Request body:
  {
    pinned:   boolean  required
    position: number?  -- 1-based place in the pinned order; only with pinned: true.
                       -- Omitted when pinning = last. Larger than the pinned count = last
  }
Response 200:
  {
    pinned: [{ id: string, name: string, pin_position: number }]  -- the whole pinned order
  }
Response 400: position given with pinned: false, or not a positive integer
  { error: { code: "VALIDATION_ERROR", message: "position must be a positive integer and needs pinned: true" } }
Response 404: { error: { code: "NOT_FOUND", message: "Country not found" } }
Side effects:
  -- Other pinned countries renumbered so positions stay 1..n with no gaps
  -- Pinning an already-pinned country with a position moves it; without one, no change
  -- Unpinning an unpinned country is a no-op that returns the current order
```

---

### Regions

```
GET /api/countries/:country_id/regions
Channel: content:region:list
Response 200:
  {
    regions: [
      {
        id:         string,
        country_id: string,
        name:       string,
        created_at: string,
        updated_at: string,
        counts: { cities: number, pois: number }
      }
    ]
  }
Response 404: { error: { code: "NOT_FOUND", message: "Country not found" } }
Side effects: none
```

```
GET /api/regions/:id
Channel: content:region:get
Response 200:
  {
    id:         string,
    country_id: string,
    name:       string,
    notes:      string | null,  -- TipTap JSON
    created_at: string,
    updated_at: string,
    counts: { cities: number, pois: number, tips: number }
  }
Response 404: { error: { code: "NOT_FOUND", message: "Region not found" } }
Side effects: none
```

```
POST /api/countries/:country_id/regions
Channel: content:region:create
Request body:
  {
    name:  string  required  -- non-empty, unique within this country
    notes: string?           -- TipTap JSON
  }
Response 201:
  {
    id:         string,
    country_id: string,
    name:       string,
    notes:      string | null,
    created_at: string,
    updated_at: string
  }
Response 400: name is empty
  { error: { code: "VALIDATION_ERROR", message: "Name is required" } }
Response 404: { error: { code: "NOT_FOUND", message: "Country not found" } }
Response 409: region name already exists within this country
  { error: { code: "DUPLICATE_NAME", message: "A region named '{name}' already exists in this country", existing_id: string } }
Side effects: search_index updated synchronously
```

```
PATCH /api/regions/:id
Channel: content:region:update
Request body:
  {
    name:  string?
    notes: string?  -- TipTap JSON
  }
Response 200: same shape as GET /api/regions/:id
Response 400: name is empty string
  { error: { code: "VALIDATION_ERROR", message: "Name cannot be empty" } }
Response 404: { error: { code: "NOT_FOUND", message: "Region not found" } }
Response 409: name conflicts with existing region in same country
  { error: { code: "DUPLICATE_NAME", message: "A region named '{name}' already exists in this country", existing_id: string } }
Side effects: search_index updated synchronously
```

```
DELETE /api/regions/:id
Channel: content:region:delete
Response 409: delete would affect other records — confirm required
  {
    error: {
      code: "DELETE_REQUIRES_CONFIRMATION",
      message: "This region has nested content that will be deleted",
      cascade_preview: {
        cities:              number,
        pois:                number,
        media:               number,
        tips:                number,  -- tips linked to the region or its cities; unlinked, not deleted
        trip_blocks_widened: number,  -- blocks on this region or its cities; kept at country level
        trip_day_references: number,  -- trip-day entries for POIs in this region
        affected_trips:      [{ id: string, name: string }]
      }
    }
  }
Response 200: (every cascade_preview count is zero — no confirm needed)
  {
    deleted_id: string,
    cascade_summary: {
      cities_deleted:              number,
      pois_deleted:                number,
      media_deleted:               number,
      tips_unlinked:               number,
      trip_blocks_widened:         number,
      trip_day_references_removed: number
    }
  }
Response 404: { error: { code: "NOT_FOUND", message: "Region not found" } }
Side effects: none until confirmed
```

```
DELETE /api/regions/:id/confirm
Channel: content:region:delete  + confirm: true
Response 200: same shape as DELETE /api/regions/:id success response
Response 404: { error: { code: "NOT_FOUND", message: "Region not found" } }
Side effects:
  -- Cities, POIs and their media and tag assignments hard deleted
  -- Tips linked to the region or its cities become Unlinked:
     destination FK nulled, former_destination set, scope retained
  -- Trip blocks on the region or its cities are kept and widened to country level
     (region_id and city_id nulled); their days and duration are unchanged
  -- trip_day_pois rows for the deleted POIs removed
  -- search_index entries for all deleted records removed synchronously
```

---

### Cities

```
GET /api/countries/:country_id/cities
Channel: content:city:list  (payload carries country_id or region_id)
  -- Also callable as GET /api/regions/:region_id/cities
Query params:
  scope  string?  -- all | direct; default: all. Country parent only:
                  --   all    = every city in the country, in any region or none
                  --   direct = only cities attached straight to the country (no region)
                  -- Ignored for a region parent, which has no nested level of cities
Response 200:
  {
    cities: [
      {
        id:          string,
        country_id:  string,
        region_id:   string | null,
        region_name: string | null,
        name:        string,
        created_at:  string,
        updated_at:  string,
        counts: { pois: number, tips: number }
      }
    ]
  }
  -- Ordered by name ascending. Not paginated.
Response 400: scope is not a valid value
  { error: { code: "VALIDATION_ERROR", message: "Invalid scope value" } }
Response 404: { error: { code: "NOT_FOUND", message: "Country not found" } }
Side effects: none
```

```
GET /api/cities/:id
Channel: content:city:get
Response 200:
  {
    id:         string,
    country_id: string,
    region_id:  string | null,
    name:       string,
    notes:      string | null,  -- TipTap JSON
    created_at: string,
    updated_at: string,
    counts: { pois: number, tips: number }
  }
Response 404: { error: { code: "NOT_FOUND", message: "City not found" } }
Side effects: none
```

```
POST /api/countries/:country_id/cities
Channel: content:city:create  (payload carries country_id or region_id)
  -- Also callable as POST /api/regions/:region_id/cities
  -- Parent context determines the FK set on creation
Request body:
  {
    name:      string  required
    region_id: string?           -- must belong to country_id if provided
    notes:     string?           -- TipTap JSON
  }
Response 201:
  {
    id:         string,
    country_id: string,
    region_id:  string | null,
    name:       string,
    notes:      string | null,
    created_at: string,
    updated_at: string
  }
Response 400: name is empty
  { error: { code: "VALIDATION_ERROR", message: "Name is required" } }
Response 404: { error: { code: "NOT_FOUND", message: "Country not found" } }
Response 409: region_id does not belong to country_id
  { error: { code: "HIERARCHY_MISMATCH", message: "Region does not belong to the specified country" } }
Response 409: a city with this name already exists under the same parent
  { error: { code: "DUPLICATE_NAME", message: "A city named '{name}' already exists here", existing_id: string } }
  -- Same parent = the same region, or the same country for cities with no region.
     The same name under a different parent is allowed.
Side effects: search_index updated synchronously
```

```
PATCH /api/cities/:id
Channel: content:city:update
Request body:
  {
    name:      string?
    region_id: string?  -- pass null to detach city to country level
    notes:     string?  -- TipTap JSON
  }
Response 200: same shape as GET /api/cities/:id
Response 404: { error: { code: "NOT_FOUND", message: "City not found" } }
Response 409: region_id does not belong to city's country
  { error: { code: "HIERARCHY_MISMATCH", message: "Region does not belong to this city's country" } }
Response 409: renaming or moving the city would duplicate a name under its parent
  { error: { code: "DUPLICATE_NAME", message: "A city named '{name}' already exists here", existing_id: string } }
Side effects: search_index updated synchronously
```

```
DELETE /api/cities/:id
Channel: content:city:delete
Response 409: delete would affect other records — confirm required
  {
    error: {
      code: "DELETE_REQUIRES_CONFIRMATION",
      message: "This city has nested content that will be deleted",
      cascade_preview: {
        pois:                number,
        media:               number,
        tips:                number,  -- unlinked, not deleted
        trip_blocks_widened: number,  -- blocks on this city; kept at region or country level
        trip_day_references: number,  -- trip-day entries for POIs in this city
        affected_trips:      [{ id: string, name: string }]
      }
    }
  }
Response 200: (every cascade_preview count is zero — no confirm needed)
  {
    deleted_id: string,
    cascade_summary: {
      pois_deleted:                number,
      media_deleted:               number,
      tips_unlinked:               number,
      trip_blocks_widened:         number,
      trip_day_references_removed: number
    }
  }
Response 404: { error: { code: "NOT_FOUND", message: "City not found" } }
Side effects: none until confirmed
```

```
DELETE /api/cities/:id/confirm
Channel: content:city:delete  + confirm: true
Response 200: same shape as DELETE /api/cities/:id success response
Response 404: { error: { code: "NOT_FOUND", message: "City not found" } }
Side effects:
  -- POIs and their media and tag assignments hard deleted
  -- Tips linked to the city become Unlinked:
     destination FK nulled, former_destination set, scope retained
  -- Trip blocks on the city are kept and widened to the city's region, or to the
     country if it has none (city_id nulled); their days and duration are unchanged
  -- trip_day_pois rows for the deleted POIs removed
  -- search_index entries for all deleted records removed synchronously
```

---

### POIs

```
GET /api/pois
Channel: content:poi:list
Query params:
  -- Exactly one parent required:
  country_id  string?
  region_id   string?
  city_id     string?
  scope       string?  -- all | direct; default: all
                       --   all    = POIs attached to the parent or to anything beneath it
                       --   direct = only POIs attached at exactly that level
                       --            (country: no region and no city; region: no city)
  tag_id      string?  -- repeatable: tag_id=x&tag_id=y; AND-ed (US-025)
Response 200:
  {
    pois: [
      {
        id:                 string,
        name:               string,
        country_id:         string,
        region_id:          string | null,
        region_name:        string | null,
        city_id:            string | null,
        city_name:          string | null,
        tags:               [{ id: string, category: string, value: string }],
        hero_thumbnail_url: string | null,  -- app://media/thumbnails/{filename}; null if no hero image
        has_description:    boolean,
        has_notes:          boolean,        -- practical_notes present
        created_at:         string,
        updated_at:         string
      }
    ]
  }
  -- Ordered by name ascending. Not paginated.
Response 400: zero or more than one parent provided, or scope is not a valid value
  { error: { code: "VALIDATION_ERROR", message: "Exactly one parent record must be specified" } }
Response 404: { error: { code: "NOT_FOUND", message: "Parent record not found" } }
Side effects: none
  -- scope = all is resolved through the live hierarchy: a POI attached only to a city
     is returned for that city's region and country
  -- region_name is the POI's own region, or its city's region when the POI has none
  -- Rich text fields are not returned; fetch GET /api/pois/:id for the full record
```

```
GET /api/pois/:id
Channel: content:poi:get
Response 200:
  {
    id:               string,
    name:             string,
    country_id:       string,
    region_id:        string | null,
    city_id:          string | null,
    description:      string | null,      -- TipTap JSON
    practical_notes:  string | null,      -- TipTap JSON
    tags:             [{ id: string, category: string, value: string }],
    media:            [{ id: string, app_url: string, thumbnail_url: string, is_hero: boolean, position: number }],
    created_at:       string,
    updated_at:       string
  }
Response 404: { error: { code: "NOT_FOUND", message: "POI not found" } }
Side effects: none
```

```
POST /api/pois
Channel: content:poi:create
Request body:
  {
    name:            string  required
    country_id:      string  required
    region_id:       string?
    city_id:         string?
    description:     string?  -- TipTap JSON
    practical_notes: string?  -- TipTap JSON
  }
Response 201: same shape as GET /api/pois/:id (tags and media arrays empty)
Response 400: name or country_id is missing
  { error: { code: "VALIDATION_ERROR", message: "Name and country_id are required" } }
Response 404: country, region, or city not found
  { error: { code: "NOT_FOUND", message: "Country not found" } }
Response 409: region or city does not belong to country_id
  { error: { code: "HIERARCHY_MISMATCH", message: "Region or city does not belong to the specified country" } }
Response 409: a POI with this name already exists under the same parent — soft warning (US-004)
  {
    error: {
      code: "DUPLICATE_NAME_WARNING",
      message: "A POI named '{name}' already exists here",
      existing: [{ id: string, name: string }]
    }
  }
  -- Not a block: repeat the call with confirm: true to create it anyway.
  -- Same parent = the most specific level the POI is attached to (city, else region,
     else country). Checked on create only.
Side effects: search_index updated synchronously
```

```
PATCH /api/pois/:id
Channel: content:poi:update
Request body:
  {
    name:            string?
    country_id:      string?  -- re-linking POI to a different country
    region_id:       string?  -- pass null to detach
    city_id:         string?  -- pass null to detach
    description:     string?  -- TipTap JSON
    practical_notes: string?  -- TipTap JSON
  }
Response 200: same shape as GET /api/pois/:id
Response 404: { error: { code: "NOT_FOUND", message: "POI not found" } }
Response 409: hierarchy mismatch on re-link
  { error: { code: "HIERARCHY_MISMATCH", message: "Region or city does not belong to the specified country" } }
Side effects: search_index updated synchronously
```

```
DELETE /api/pois/:id
Channel: content:poi:delete
Response 409: POI referenced in trip itineraries — confirm required
  {
    error: {
      code: "DELETE_REQUIRES_CONFIRMATION",
      message: "This POI is used in trip itineraries and will be removed from them",
      affected_trips: [{ id: string, name: string }]
    }
  }
Response 200: (unreferenced POI — no confirm needed)
  {
    deleted_id: string,
    cascade_summary: { trip_day_references_removed: number, media_deleted: number }
  }
Response 404: { error: { code: "NOT_FOUND", message: "POI not found" } }
Side effects: none until confirmed
```

```
DELETE /api/pois/:id/confirm
Channel: content:poi:delete  + confirm: true
Response 200:
  {
    deleted_id: string,
    cascade_summary: { trip_day_references_removed: number, media_deleted: number }
  }
Response 404: { error: { code: "NOT_FOUND", message: "POI not found" } }
Side effects:
  -- trip_day_pois rows removed
  -- Media hard deleted
  -- search_index updated synchronously
```

---

### Tags

```
GET /api/tag-definitions
Channel: content:tag-definition:list
Query params: content_type  string?  -- country | region | city | poi | trip
Response 200:
  {
    tags: [
      {
        id:           string,
        content_type: string,
        category:     string,
        value:        string,
        is_default:   boolean,
        single_select: boolean   -- true = a record carries at most one value from this category
      }
    ]
  }
Side effects: none
```

```
POST /api/content-tags
Channel: content:tag:apply
Request body:
  {
    tag_id:     string  required
    -- Exactly one of the following:
    country_id: string?
    region_id:  string?
    city_id:    string?
    poi_id:     string?
    trip_id:    string?
  }
Response 201: { id: string, tag_id: string, [parent_field]: string, created_at: string,
                replaced_id: string | null }
  -- replaced_id: the assignment this one replaced, when the tag's category is single-select
Response 400: zero or more than one parent FK provided
  { error: { code: "VALIDATION_ERROR", message: "Exactly one parent record must be specified" } }
Response 404: tag or parent record not found
  { error: { code: "NOT_FOUND", message: "Tag definition not found" } }
Response 409: tag already applied to this record
  { error: { code: "DUPLICATE_TAG", message: "This tag is already applied to this record" } }
Response 409: tag content_type does not match parent record type
  { error: { code: "TYPE_MISMATCH", message: "This tag cannot be applied to this content type" } }
Side effects:
  -- If the tag's category is single-select (e.g. POI Must-Do Status) and the record
     already carries another value from it, that assignment is removed and this one
     takes its place (US-057)
  -- Tags are not indexed text. Search filters by tag through a live join on
     content_tags, so tag changes need no search index update.
```

```
DELETE /api/content-tags/:id
Channel: content:tag:remove
Response 200: { deleted_id: string }
Response 404: { error: { code: "NOT_FOUND", message: "Tag assignment not found" } }
Side effects: none
```

---

## Module 2 — Trips (Trip CRUD, Blocks, Days, Day-POIs)

**IPC prefix:** `trip`

**Key behaviours:**

- **Two kinds of block** (D-27). A `location` block is a stay at a country, region or
  city and can be expanded into days. A `travel` block is a flight, train, bus, ferry
  or drive with free-text `from_label` / `to_label`; it has no place, no days and no
  POIs, and never creates budget lines. `kind` is fixed when the block is created
- **Trip length** = the sum of every block's `duration_days`, location and travel.
  `length.complete` is false when any block has no duration; `length.days` is then the
  sum of the durations that are set (the renderer shows "12+ days"). A trip with no
  blocks has length `{ days: 0, complete: false }`
- **Day numbering** is derived, never stored. Walking the blocks in order, each block
  covers the next `duration_days` days. A 0-day travel block in position 1 is **Day 0**
  and adds nothing to the length; a 0-day travel block elsewhere sits between two days
  ("Day 2→3"). After a block with no duration, later blocks have no known day numbers
  and the renderer shows their length instead ("4 days"). `trip_days.day_number` is the
  day's number within its own block
- **Days and collapsing.** A block either has days (`has_days`) or not. Days are created
  by expanding a block and stay until the block is deleted, its duration is lowered, or
  the day is deleted. Showing a block open or collapsed in the trip builder is UI state
  (Module 10), not a call to this module: collapsing never removes days (US-013)

---

### Trips

```
GET /api/trips
Channel: trip:trip:list
Query params:
  status     string?  -- Sample | Planning | Ready | Completed
                      -- repeatable: status=x&status=y; OR-ed. The Trips list sends one
                      -- (single-select, US-027); repeating it stays valid
  country_id string?  -- repeatable; trips referencing any of these countries
  tag_id     string?  -- repeatable; trips carrying all of these tags
                      -- Different filters combine with AND
Response 200:
  {
    trips: [
      {
        id:              string,
        name:            string,
        status:          string,
        completed_at:    string | null,
        pin_position:    number | null,
        created_at:      string,
        updated_at:      string,
        countries:       [{ id: string, name: string }],
        counts:          { blocks: number, days: number, windows: number },
        length:          { days: number, complete: boolean },
          -- see Trip length in Key behaviours; complete = false shows "12+ days"
        estimate:        number | null,  -- primary budget's total_budgeted, in cents
        hero_thumbnail_url: string | null  -- hero rule in Module 8 Key behaviours
      }
    ]
  }
Side effects: none
  -- Order: pinned trips first by pin_position, then newest first (FR14). Other sorts
     on the Trips page are applied by the renderer to this list
```

```
GET /api/trips/:id
Channel: trip:trip:get
Response 200:
  {
    id:           string,
    name:         string,
    status:       string,
    completed_at: string | null,
    pin_position: number | null,
    notes:        string | null,  -- TipTap JSON
    created_at:   string,
    updated_at:   string,
    countries:    [{ id: string, name: string }],
    tags:         [{ id: string, category: string, value: string }],
    media:        [{ id: string, app_url: string, thumbnail_url: string, is_hero: boolean, position: number }],
    counts: {
      blocks:        number,   -- location and travel
      travel_blocks: number,
      days:          number,
      budgets:       number
    },
    length: { days: number, complete: boolean }   -- see Trip length in Key behaviours
  }
Response 404: { error: { code: "NOT_FOUND", message: "Trip not found" } }
Side effects: none
```

```
POST /api/trips
Channel: trip:trip:create
Request body:
  {
    name:        string    required  -- non-empty; duplicate names permitted
    country_ids: string[]?           -- countries this trip references; can be empty
    notes:       string?             -- TipTap JSON
  }
Response 201: same shape as GET /api/trips/:id (tags, media, counts empty)
Response 400: name is empty
  { error: { code: "VALIDATION_ERROR", message: "Name is required" } }
Response 404: any country_id in country_ids not found
  { error: { code: "NOT_FOUND", message: "Country '{id}' not found" } }
Side effects: trip_countries rows created; search_index updated synchronously
  -- "Start a trip" from a country (D-29) is this call with
     { name: "<Country> trip", country_ids: [<country id>] }: the trip links only that
     country and has no blocks. No block is created for it
```

```
PATCH /api/trips/:id
Channel: trip:trip:update
Request body:
  {
    name:         string?
    status:       string?   -- Sample | Planning | Ready | Completed
    completed_at: string?   -- ISO date (YYYY-MM-DD). Accepted only when the trip is
                            -- Completed, or status: Completed is in the same request.
                            -- Any valid date, past or future.
                            -- Pass null to explicitly clear.
    notes:        string?   -- TipTap JSON
    country_ids:  string[]? -- replaces full set of linked countries
  }
Response 200: same shape as GET /api/trips/:id
Response 400: name is empty, or status is not a valid value, or completed_at is not a valid date
  { error: { code: "VALIDATION_ERROR", message: "Invalid field value" } }
Response 404: trip not found, or any country_id not found
  { error: { code: "NOT_FOUND", message: "Trip not found" } }
Response 409: completed_at provided but the trip is not, and is not becoming, Completed
  { error: { code: "INVALID_TRIP_STATUS", message: "A completion date can only be set on a Completed trip" } }
Response 409: country_ids omits a country that one of the trip's blocks uses
  {
    error: {
      code: "RECORD_IN_USE",
      message: "A country used by this trip's itinerary cannot be unlinked",
      blocking_blocks: [{ id: string, position: number, country_id: string, country_name: string }]
    }
  }
Side effects:
  -- If status changes to Completed AND completed_at is currently null:
     completed_at set to today's date automatically
  -- If status changes to Completed AND completed_at already has a value:
     completed_at left unchanged — prior date restored
  -- If status changes away from Completed: completed_at retained, not cleared.
     It is not editable again until the trip is Completed (US-016b)
  -- If completed_at is provided together with status: Completed, the provided
     date is used instead of the default or the retained one
  -- trip_countries replaced if country_ids provided
  -- search_index updated synchronously
```

```
DELETE /api/trips/:id
Channel: trip:trip:delete
Response 409: trip referenced in Travel Windows — confirm required
  {
    error: {
      code: "DELETE_REQUIRES_CONFIRMATION",
      message: "This trip is referenced in Travel Windows",
      affected_windows: [{ id: string, name: string }],
      is_completed: boolean
        -- true = snapshot preserved in Travel Windows per US-034b
        -- false = Travel Window references removed entirely
    }
  }
Response 200: (unreferenced trip — no confirm needed)
  {
    deleted_id: string,
    cascade_summary: {
      blocks_deleted:         number,
      days_deleted:           number,
      day_poi_refs_deleted:   number,
      budgets_deleted:        number,
      media_deleted:          number,
      lessons_learned_unlinked: number,
        -- Lessons Learned that named this trip as their source; kept, link removed
      windows_snapshot_taken: number,
        -- Travel Window entries now showing the preserved record (Completed trip)
      windows_refs_removed:   number
        -- Travel Window entries removed (non-Completed trip)
    }
  }
Response 404: { error: { code: "NOT_FOUND", message: "Trip not found" } }
Side effects: none until confirmed
```

```
DELETE /api/trips/:id/confirm
Channel: trip:trip:delete  + confirm: true
Response 200: same shape as DELETE /api/trips/:id success response
Response 404: { error: { code: "NOT_FOUND", message: "Trip not found" } }
Side effects:
  -- Lessons Learned with this trip as source: source_trip_id nulled,
     former_source_trip set to the trip's name (US-052). The tips are not deleted.
  -- Completed trip in at least one Travel Window: one trip_snapshots row written
     (full-detail presentation payload, versioned); the trip's media re-parented to
     the snapshot and the POI/country media it shows copied to it; each
     travel_window_trips row switched from trip_id to snapshot_id
  -- Completed trip in no Travel Window: no snapshot — deleted like any other trip
  -- Non-Completed trip: travel_window_trips rows removed
  -- All blocks, days, day_poi_refs, budgets, trip_countries links and tag
     assignments hard deleted; media hard deleted unless re-parented to a snapshot
  -- search_index entries for the trip and its budgets removed synchronously
  -- If the trip was pinned, the pinned positions after it close up
```

```
PUT /api/trips/:id/pin
Channel: trip:trip:pin
  -- Same contract as PUT /api/countries/:id/pin, for trips (FR14, US-005).
Request body:  { pinned: boolean  required, position: number? }
Response 200:  { pinned: [{ id: string, name: string, pin_position: number }] }
Response 400:  { error: { code: "VALIDATION_ERROR", message: "position must be a positive integer and needs pinned: true" } }
Response 404:  { error: { code: "NOT_FOUND", message: "Trip not found" } }
Side effects:  other pinned trips renumbered so positions stay 1..n with no gaps
```

---

### Trip Countries

```
POST /api/trips/:trip_id/countries
Channel: trip:country:add
Request body:  { country_id: string  required }
Response 201:  { trip_id: string, country_id: string, country_name: string }
Response 404:  { error: { code: "NOT_FOUND", message: "Trip or country not found" } }
Response 409:  { error: { code: "DUPLICATE_LINK", message: "Country is already linked to this trip" } }
Side effects: none
```

```
DELETE /api/trips/:trip_id/countries/:country_id
Channel: trip:country:remove
Response 200:  { trip_id: string, country_id: string }
Response 404:  { error: { code: "NOT_FOUND", message: "Country link not found" } }
Response 409:  one or more of the trip's blocks use this country
  {
    error: {
      code: "RECORD_IN_USE",
      message: "A country used by this trip's itinerary cannot be unlinked",
      blocking_blocks: [{ id: string, position: number }]
    }
  }
Side effects: none
  -- Invariant: every country used by one of a trip's blocks is linked to the trip.
     Blocks add the link automatically; it can be removed only once no block uses
     the country. A link with no block behind it is the user's to keep or remove.
```

---

### Trip Blocks

```
GET /api/trips/:trip_id/blocks
Channel: trip:block:list
Response 200:
  {
    blocks: [
      {
        id:            string,
        trip_id:       string,
        position:      number,
        kind:          string,         -- location | travel
        country_id:    string | null,  -- null on a travel block
        country_name:  string | null,
        region_id:     string | null,
        region_name:   string | null,
        city_id:       string | null,
        city_name:     string | null,
        travel_mode:   string | null,  -- flight | train | bus | ferry | drive; travel only
        from_label:    string | null,  -- travel only
        to_label:      string | null,  -- travel only
        duration_days: number | null,
        notes:         string | null,  -- TipTap JSON
        has_days:      boolean,        -- true once the block has been expanded into days
        created_at:    string,
        updated_at:    string
      }
    ]
  }
Response 404: { error: { code: "NOT_FOUND", message: "Trip not found" } }
Side effects: none
```

```
POST /api/trips/:trip_id/blocks
Channel: trip:block:create
Request body:
  {
    kind:          string?  -- location | travel; default: location
    -- location block:
    country_id:    string   required for a location block; not allowed on a travel block
    region_id:     string?
    city_id:       string?
    -- travel block:
    travel_mode:   string?  -- flight | train | bus | ferry | drive; default: flight
    from_label:    string?  -- free text
    to_label:      string?  -- free text
    -- either kind:
    duration_days: number?  -- whole days: ≥ 1 on a location block, ≥ 0 on a travel block
    notes:         string?  -- TipTap JSON
    position:      number?  -- if omitted, appended to end
  }
Response 201:
  {
    id:            string,
    trip_id:       string,
    position:      number,
    kind:          string,
    country_id:    string | null,
    country_name:  string | null,
    region_id:     string | null,
    region_name:   string | null,
    city_id:       string | null,
    city_name:     string | null,
    travel_mode:   string | null,
    from_label:    string | null,
    to_label:      string | null,
    duration_days: number | null,
    notes:         string | null,  -- TipTap JSON
    has_days:      false,
    created_at:    string,
    updated_at:    string
  }
Response 400: location block without country_id
  { error: { code: "VALIDATION_ERROR", message: "country_id is required for a location block" } }
Response 400: travel block with a country, region or city; or an unknown kind or travel_mode
  { error: { code: "VALIDATION_ERROR", message: "Travel blocks don't take a place" } }
Response 400: duration_days not a whole number, below 1 on a location block, or below 0
  on a travel block
  { error: { code: "VALIDATION_ERROR", message: "duration_days must be a whole number of days" } }
Response 404: trip, country, region, or city not found
  { error: { code: "NOT_FOUND", message: "Country not found" } }
Response 409: region or city does not belong to specified country
  { error: { code: "HIERARCHY_MISMATCH", message: "Region or city does not belong to the specified country" } }
Side effects:
  -- Sibling block positions incremented if inserted mid-sequence
  -- Location block: its country added to trip_countries if not already linked.
     A travel block adds no link, so it can be added to a trip with no countries
```

```
PATCH /api/trips/blocks/:id
Channel: trip:block:update
Request body:
  {
    -- location block only ("Change place…", US-012):
    country_id:    string?
    region_id:     string?  -- pass null to detach
    city_id:       string?  -- pass null to detach
    -- travel block only:
    travel_mode:   string?
    from_label:    string?  -- pass null to clear
    to_label:      string?  -- pass null to clear
    -- either kind:
    duration_days: number?  -- pass null to clear (not allowed on a block with days);
                            -- on a block with days, raising it appends empty days and
                            -- lowering it removes days from the end (confirm required)
    notes:         string?  -- TipTap JSON
    position:      number?
  }
  -- kind cannot be changed; delete the block and add one of the other kind
Response 200: same shape as POST /api/trips/:trip_id/blocks response
Response 400: duration_days not a whole number, below the minimum for the block's kind,
  or null on a block with days
  { error: { code: "VALIDATION_ERROR", message: "duration_days must be a whole number of days" } }
Response 400: place fields sent for a travel block, or travel fields for a location block
  { error: { code: "VALIDATION_ERROR", message: "That field does not apply to this kind of block" } }
Response 404: { error: { code: "NOT_FOUND", message: "Block not found" } }
Response 409: hierarchy mismatch
  { error: { code: "HIERARCHY_MISMATCH", message: "Region or city does not belong to the specified country" } }
Response 409: duration_days reduced below existing expanded day count
  {
    error: {
      code: "DURATION_CONFLICT",
      message: "Reducing duration will remove existing days",
      days_that_will_be_removed: [{ id: string, day_number: number }]
    }
  }
Side effects:
  -- Sibling positions recompacted if position changed
  -- Duration conflict requires separate confirm call
  -- Block with days: it always has exactly duration_days days.
     Raising duration_days appends empty days numbered after the existing ones
  -- Changing the place (country_id / region_id / city_id) never touches the block's
     days, duration or notes (handoff 29). The trip builder's "Change place…" offers
     places in the block's own country, which is the case the design specifies; a
     change to another country is also accepted and keeps the days
  -- If country_id changes, the new country is added to trip_countries if not linked.
     The old country stays linked (a link with no block behind it is allowed)
```

```
PATCH /api/trips/blocks/:id/confirm-duration-reduce
Channel: trip:block:update  + confirm: true
Request body:  { duration_days: number  required }
Response 200:  same shape as PATCH /api/trips/blocks/:id response
Response 404:  { error: { code: "NOT_FOUND", message: "Block not found" } }
Side effects:  Days beyond new duration hard deleted including their day_poi_refs
```

```
DELETE /api/trips/blocks/:id
Channel: trip:block:delete
Response 200:
  {
    deleted_id: string,
    cascade_summary: { days_deleted: number, day_poi_refs_deleted: number }
  }
Response 404: { error: { code: "NOT_FOUND", message: "Block not found" } }
Side effects:
  -- All trip_days and trip_day_pois for this block hard deleted
  -- Sibling block positions compacted
```

---

### Block Expansion

```
POST /api/trips/blocks/:id/expand
Channel: trip:block:expand
  -- Creates the days of a location block that has none ("Expand to days").
  -- Opening a block that already has days is UI state only — no call.
Request body:  { duration_days: number  required }
Response 200:
  {
    block_id: string,
    days: [
      {
        id:         string,
        block_id:   string,
        trip_id:    string,
        day_number: number,
        date:       string | null,
        notes:      string | null,  -- TipTap JSON
        pois:       [],
        created_at: string,
        updated_at: string
      }
    ]
  }
Response 400: duration_days missing or not a positive integer
  { error: { code: "VALIDATION_ERROR", message: "duration_days is required and must be a positive integer" } }
Response 400: the block is a travel block
  { error: { code: "VALIDATION_ERROR", message: "Travel blocks don't expand into days" } }
Response 404: { error: { code: "NOT_FOUND", message: "Block not found" } }
Response 409: block already has days
  { error: { code: "ALREADY_EXPANDED", message: "Block already has days", existing_day_count: number } }
Side effects: trip_days rows created; block duration_days updated to match
```

```
POST /api/trips/:trip_id/blocks/expand-all
Channel: trip:block:expand-all
  -- "Expand all" (D-44, ⌥⌘↓): creates days for every location block that has a
  -- duration and no days yet, in one transaction (one undo step).
Request body:  none
Response 200:
  {
    trip_id: string,
    expanded: [{ block_id: string, days_created: number }],  -- blocks that gained days
    skipped:  {
      no_duration: number,  -- location blocks without a duration (stay as they are)
      travel:      number,  -- travel blocks never have days
      had_days:    number   -- already had days; nothing created
    }
  }
Response 404: { error: { code: "NOT_FOUND", message: "Trip not found" } }
Side effects: trip_days rows created for each block in `expanded`
  -- Which blocks the builder then shows open is UI state; the renderer opens every
     block with days after this call
```

**Collapsing a block has no channel.** "Collapse to block", "Collapse all" and Enter on
an open block only change which blocks the trip builder shows open — UI state, stored per
trip by Module 10. The block's days, notes and POIs are untouched and reappear when the
block is opened again (US-013; handoff 30). The warning before collapsing a block whose
days have content is shown by the renderer from data it already holds.

```
POST /api/trips/blocks/:id/split
Channel: trip:block:split
  -- Splits a location block that has days in two, so a broad block's days can be reorganized into
  -- more specific blocks (US-013): a 7-day Thailand block becomes a 3-day Bangkok
  -- block and a 4-day Chiang Mai block.
Request body:
  {
    first_day_number: number  required  -- first day that moves to the new block; 2..day count
    -- Location of the new block; default: same as the original
    country_id: string?
    region_id:  string?
    city_id:    string?
  }
Response 200:
  { blocks: [ <original block>, <new block> ] }   -- each: same shape as the block create response
Response 400: block has no days (or is a travel block), or first_day_number is out of range
  { error: { code: "VALIDATION_ERROR", message: "first_day_number must be between 2 and the block's day count" } }
Response 404: block, country, region, or city not found
  { error: { code: "NOT_FOUND", message: "Block not found" } }
Response 409: { error: { code: "HIERARCHY_MISMATCH", message: "Region or city does not belong to the specified country" } }
Side effects:
  -- New block inserted directly after the original; later block positions incremented
  -- Days from first_day_number onward move to the new block and are renumbered from 1,
     keeping their POIs, notes and dates
  -- duration_days of both blocks set to their day counts
  -- New block's country added to trip_countries if not already linked
  -- To change the original block's own location, use PATCH /api/trips/blocks/:id
```

---

### Trip Days

```
GET /api/trips/:trip_id/days
Channel: trip:day:list
Response 200:
  {
    days: [
      {
        id:         string,
        block_id:   string,
        trip_id:    string,
        day_number: number,
        date:       string | null,
        notes:      string | null,  -- TipTap JSON
        created_at: string,
        updated_at: string,
        pois: [
          {
            id:       string,
            poi_id:   string,
            poi_name: string,
            position: number,
            notes:    string | null  -- TipTap JSON (day-specific override notes)
          }
        ]
      }
    ]
  }
Response 404: { error: { code: "NOT_FOUND", message: "Trip not found" } }
Side effects: none
```

```
GET /api/trips/days/:id
Channel: trip:day:get
Response 200: single day object — same shape as item in GET /api/trips/:trip_id/days
Response 404: { error: { code: "NOT_FOUND", message: "Day not found" } }
Side effects: none
```

```
POST /api/trips/blocks/:block_id/days
Channel: trip:day:create
  -- Inserts one empty day into a block that has days.
Request body:
  {
    day_number: number?  -- position to insert at; default: after the last day
    date:       string?  -- ISO date string
    notes:      string?  -- TipTap JSON
  }
Response 201: same shape as GET /api/trips/days/:id
Response 400: block has no days (or is a travel block), or day_number is out of range
  { error: { code: "VALIDATION_ERROR", message: "Expand the block before adding days" } }
Response 404: { error: { code: "NOT_FOUND", message: "Block not found" } }
Side effects:
  -- Days at and after day_number are renumbered up by one
  -- Block duration_days incremented
```

```
PATCH /api/trips/days/:id
Channel: trip:day:update
Request body:
  {
    date:       string?  -- ISO date string; pass null to clear
    notes:      string?  -- TipTap JSON
    day_number: number?  -- move the day to this position within its block
    block_id:   string?  -- move the day to another block with days in the same trip;
                         -- lands at day_number, or at the end if day_number is omitted
  }
Response 200: same shape as GET /api/trips/days/:id
Response 400: date is not a valid ISO date string, or day_number is out of range
  { error: { code: "VALIDATION_ERROR", message: "date must be a valid ISO date string" } }
Response 400: block_id is in another trip, has no days, or the day is its block's only day
  { error: { code: "VALIDATION_ERROR", message: "This day cannot be moved to that block" } }
Response 404: day or target block not found
  { error: { code: "NOT_FOUND", message: "Day not found" } }
Side effects:
  -- Reorder: the other days in the block are renumbered so numbers stay 1..n
  -- Move: both blocks renumbered; source duration_days decremented, target incremented
  -- A day keeps its POIs, notes and date when it is reordered or moved
  -- A block's only day cannot be moved out — delete the block instead
```

```
DELETE /api/trips/days/:id
Channel: trip:day:delete
Response 409: day has content — confirm required
  {
    error: {
      code: "DELETE_REQUIRES_CONFIRMATION",
      message: "This day has content that will be deleted",
      day_poi_refs: number,
      has_notes:    boolean
    }
  }
Response 200: (empty day, or confirm: true)
  { deleted_id: string, block_id: string, cascade_summary: { day_poi_refs_deleted: number } }
Response 400: the day is its block's only day
  { error: { code: "VALIDATION_ERROR", message: "Delete the block to remove its last day" } }
Response 404: { error: { code: "NOT_FOUND", message: "Day not found" } }
Side effects:
  -- Day and its trip_day_pois hard deleted; later days renumbered down by one
  -- Block duration_days decremented
```

---

### Day POIs

```
POST /api/trips/days/:day_id/pois
Channel: trip:day-poi:add
Request body:
  {
    poi_id:   string   required
    position: number?  -- if omitted, appended to end of day
    notes:    string?  -- TipTap JSON (day-specific override notes)
  }
Response 201:
  {
    id:         string,
    day_id:     string,
    poi_id:     string,
    poi_name:   string,
    position:   number,
    notes:      string | null,  -- TipTap JSON
    created_at: string
  }
Response 404: day or POI not found
  { error: { code: "NOT_FOUND", message: "POI not found" } }
Side effects:
  -- Sibling POI positions incremented if inserted mid-sequence
  -- Same POI can be added to same day multiple times — no duplicate check
```

```
PATCH /api/trips/days/:day_id/pois/:id
Channel: trip:day-poi:update
Request body:
  {
    position: number?
    notes:    string?  -- TipTap JSON; pass null to clear
  }
Response 200: same shape as POST /api/trips/days/:day_id/pois response
Response 404: { error: { code: "NOT_FOUND", message: "Day POI not found" } }
Side effects: sibling positions recompacted if position changed
```

```
DELETE /api/trips/days/:day_id/pois/:id
Channel: trip:day-poi:remove
Response 200:  { deleted_id: string }
Response 404:  { error: { code: "NOT_FOUND", message: "Day POI not found" } }
Side effects:
  -- trip_day_pois reference deleted; POI master record unaffected
  -- Sibling positions compacted
```

---

## Module 3 — Budget

**IPC prefix:** `budget`

**Key behaviours:**

- A non-template budget is searchable by its name and by the names of its categories
  and line items. Every write that changes any of those re-indexes the budget in the
  same transaction. Amounts and line item notes are not indexed
- Templates are not indexed
- **Money.** Every amount in requests and responses is an integer number of US cents
  (`1250` = $12.50). A fractional or negative amount is a `VALIDATION_ERROR`.
  `cost_per_day` is rounded to the nearest cent
- **Thresholds** are two global settings read from Module 10 (D-18): `close` (default
  0.10) and `way_over` (default 0.25, always greater than `close`). They are not stored
  on budgets. A change recolours every budget on its next read; nothing is rewritten
- **Variance status** for a line item — first matching rule wins:

  | # | Condition                                                   | Status  |
  | - | ----------------------------------------------------------- | ------- |
  | 1 | `budgeted` is null or `actual` is null                      | null    |
  | 2 | `budgeted` = 0 and `actual` = 0                             | Close   |
  | 3 | `budgeted` = 0 and `actual` > 0                             | WayOver |
  | 4 | \|`actual` − `budgeted`\| ≤ `close` × `budgeted`            | Close   |
  | 5 | `actual` < `budgeted`                                       | Under   |
  | 6 | `actual` − `budgeted` > `way_over` × `budgeted`             | WayOver |
  | 7 | otherwise                                                   | Over    |

  Close therefore takes precedence over Under and Over, and an exact match is Close.
  `variance_status` is one of `Under | Close | Over | WayOver | null` everywhere it
  appears. The renderer draws it on the Difference value (glyph + word + colour); there
  is no status column
- **Category subtotals and the budget snapshot:**
  - `budgeted` / `total_budgeted` = sum of the line items' budgeted amounts; null if none has one.
    `actual` / `total_actual` likewise
  - `difference` / `total_difference` and `variance_status` compare like with like: they are
    computed only over line items that have **both** amounts, using the table above on
    those two sums. Null if no line item has both
  - `actuals_complete` is true when every line item with a budgeted amount also has an
    actual. While it is false, the difference and status cover only the lines that have
    both amounts; pending lines show as Pending and no "partial" label is added (D-20)
- **Cost per person** is not part of this version: budgets are shared (one pot, D-28).
  It returns with split budgets (US-064, P1; see Planned Schema Changes in `data-model.md`)
- **Cost per day** = `total_budgeted` ÷ the trip's length (sum of every block's
  `duration_days`, location and travel — see Module 2). Null unless the trip has at least
  one block, **every** block has a duration, and the length is above 0 — a
  partly-planned trip shows "Needs every stop's length", never a figure based on some
  of its days
- **Primary budget.** A trip with budgets has exactly one primary budget. Presentation
  mode and the comparison view show the primary; the trip record lists all of them

---

### Budgets

```
GET /api/budgets
Channel: budget:budget:list
Query params:
  trip_id:     string?   -- filter to budgets attached to this trip
  is_template: boolean?  -- true = templates only; false = live budgets only
Response 200:
  {
    budgets: [
      {
        id:              string,
        name:            string,
        trip_id:         string | null,
        is_template:     boolean,
        is_primary:      boolean,        -- always false for templates
        notes:           string | null,  -- TipTap JSON
        created_at:      string,
        updated_at:      string,
        snapshot: {
          total_budgeted:   number | null,
          total_actual:     number | null,
          total_difference: number | null,
          actuals_complete: boolean,
          cost_per_day:     number | null,
          variance_status:  string | null   -- Under | Close | Over | WayOver
        }
      }
    ]
  }
Side effects: none
```

```
GET /api/budgets/:id
Channel: budget:budget:get
Response 200:
  {
    id:              string,
    name:            string,
    trip_id:         string | null,
    is_template:     boolean,
    is_primary:      boolean,        -- always false for templates
    notes:           string | null,  -- TipTap JSON
    created_at:      string,
    updated_at:      string,
    snapshot: {
      total_budgeted:   number | null,
      total_actual:     number | null,
      total_difference: number | null,
      actuals_complete: boolean,
      cost_per_day:     number | null,  -- null unless every block on the trip has a duration
      variance_status:  string | null
    },
    categories: [
      {
        id:       string,
        name:     string,
        position: number,
        subtotal: {
          budgeted:        number | null,
          actual:          number | null,
          difference:      number | null,
          actuals_complete: boolean,
          variance_status: string | null
        },
        line_items: [
          {
            id:              string,
            name:            string,
            position:        number,
            budgeted:        number | null,
            actual:          number | null,
            difference:      number | null,  -- computed: actual - budgeted
            variance_status: string | null,  -- Under | Close | Over | WayOver | null if no actual
            notes:           string | null,
            created_at:      string,
            updated_at:      string
          }
        ]
      }
    ]
  }
Response 404: { error: { code: "NOT_FOUND", message: "Budget not found" } }
Side effects: none
  -- difference and variance_status always computed at application layer, never stored.
  -- Rules for variance_status, subtotals, the snapshot and cost_per_day are in
     Key behaviours at the top of this module.
```

```
POST /api/budgets
Channel: budget:budget:create
Request body:
  {
    name:            string   required  -- unique within same trip; templates: unique among templates
    trip_id:         string   required  -- unless is_template = true
    notes:           string?            -- TipTap JSON
    is_template:     boolean?           -- default: false
    template_id:     string?            -- copies category and line item names from this template
  }
  -- New template in Settings › Budget: "Blank" = is_template: true;
     "Copy of a template" = is_template: true + template_id;
     "An existing budget" = POST /api/budgets/:id/save-as-template
Response 201: same shape as GET /api/budgets/:id (categories empty unless template applied)
Response 400: trip_id missing for non-template budget
  { error: { code: "VALIDATION_ERROR", message: "trip_id is required for non-template budgets" } }
Response 400: trip_id provided for template budget
  { error: { code: "VALIDATION_ERROR", message: "Template budgets cannot be attached to a trip" } }
Response 404: trip_id or template_id not found
  { error: { code: "NOT_FOUND", message: "Trip not found" } }
Response 409: budget name already exists on this trip, or template name already taken
  { error: { code: "DUPLICATE_NAME", message: "A budget named '{name}' already exists on this trip" } }
Side effects:
  -- If template_id provided: category names and line item names copied; all cost fields
     null. The new budget is an independent copy — later template edits never reach it
  -- The first budget created on a trip becomes its primary budget
  -- Non-template budget: search_index entry created synchronously
```

```
PATCH /api/budgets/:id
Channel: budget:budget:update
Request body:
  {
    name:            string?
    notes:           string?  -- TipTap JSON; pass null to clear
  }
  -- trip_id and is_template are immutable after creation
  -- is_primary is changed only through POST /api/budgets/:id/set-primary
Response 200: same shape as GET /api/budgets/:id
Response 404: { error: { code: "NOT_FOUND", message: "Budget not found" } }
Response 409: name conflicts with existing budget on same trip
  { error: { code: "DUPLICATE_NAME", message: "A budget named '{name}' already exists on this trip" } }
Side effects: search_index updated synchronously if name or notes changed
```

```
DELETE /api/budgets/:id
Channel: budget:budget:delete
Response 200:
  {
    deleted_id: string,
    cascade_summary: { categories_deleted: number, line_items_deleted: number }
  }
Response 404: { error: { code: "NOT_FOUND", message: "Budget not found" } }
Side effects:
  -- All categories and line items hard deleted
  -- Deleting a template does not affect budgets already built from it
  -- If the deleted budget was the trip's primary, the oldest remaining budget on
     that trip becomes primary
  -- search_index entry removed synchronously
```

```
POST /api/budgets/:id/set-primary
Channel: budget:budget:set-primary
Request body:  none
Response 200:  same shape as GET /api/budgets/:id with is_primary: true
Response 400:  budget is a template
  { error: { code: "VALIDATION_ERROR", message: "A template cannot be a primary budget" } }
Response 404:  { error: { code: "NOT_FOUND", message: "Budget not found" } }
Side effects:
  -- This budget set to is_primary = 1; the trip's previous primary set to 0
```

```
POST /api/budgets/:id/duplicate
Channel: budget:budget:duplicate
  -- "Duplicate" in a budget's More ▾ (a second scenario for the same trip) and on a
  -- template's row in Settings › Budget.
Request body:  { name: string  required }  -- the renderer proposes "<name> copy"
Response 201:  same shape as GET /api/budgets/:id
Response 400:  { error: { code: "VALIDATION_ERROR", message: "Name is required" } }
Response 404:  { error: { code: "NOT_FOUND", message: "Budget not found" } }
Response 409:  name already used on the same trip, or by another template
  { error: { code: "DUPLICATE_NAME", message: "A budget named '{name}' already exists on this trip" } }
Side effects:
  -- Live budget: copied onto the same trip with every category, line item, amount and
     note; the copy is never primary
  -- Template: copied as a new template (names only, as templates hold no amounts)
  -- Live budget copy: search_index entry created synchronously
```

---

### Apply Template to Existing Budget

```
POST /api/budgets/:id/apply-template
Channel: budget:budget:apply-template
Request body:  { template_id: string  required }
Response 409: budget already has categories — confirm required
  {
    error: {
      code: "TEMPLATE_OVERWRITE_CONFLICT",
      message: "Applying this template will replace existing categories",
      existing_category_count: number,
      existing_line_item_count: number
    }
  }
Response 200: same shape as GET /api/budgets/:id (no conflict — applied immediately)
Response 404: { error: { code: "NOT_FOUND", message: "Template not found" } }
Side effects: none on conflict; on clean apply the category structure is replaced
  and the budget's search_index entry updated synchronously
```

```
POST /api/budgets/:id/apply-template/confirm
Channel: budget:budget:apply-template  + confirm: true
Request body:  { template_id: string  required }
Response 200:  same shape as GET /api/budgets/:id
Response 404:  { error: { code: "NOT_FOUND", message: "Template not found" } }
Side effects:
  -- All existing categories and line items deleted
  -- Category and line item structure copied from template; all cost fields null
  -- search_index entry for the budget updated synchronously
```

---

### Save Budget as Template

```
POST /api/budgets/:id/save-as-template
Channel: budget:budget:save-as-template
Request body:  { name: string  required }
Response 201:
  {
    id:          string,
    name:        string,
    is_template: true,
    created_at:  string,
    categories: [
      {
        id:         string,
        name:       string,
        position:   number,
        line_items: [{ id: string, name: string, position: number }]
      }
    ]
  }
Response 400: name is empty
  { error: { code: "VALIDATION_ERROR", message: "Template name is required" } }
Response 404: { error: { code: "NOT_FOUND", message: "Budget not found" } }
Response 409: template name already exists
  { error: { code: "DUPLICATE_NAME", message: "A template named '{name}' already exists" } }
Side effects:
  -- New budget row created with is_template = 1, trip_id = null
  -- Category and line item structure copied; all cost fields null
  -- No search_index entry — templates are not indexed
```

---

### Budget Categories

```
POST /api/budgets/:budget_id/categories
Channel: budget:category:create
Request body:
  {
    name:     string   required  -- unique within this budget
    position: number?
  }
Response 201:
  {
    id:         string,
    budget_id:  string,
    name:       string,
    position:   number,
    created_at: string,
    updated_at: string
  }
Response 400: name is empty
  { error: { code: "VALIDATION_ERROR", message: "Name is required" } }
Response 404: { error: { code: "NOT_FOUND", message: "Budget not found" } }
Response 409: { error: { code: "DUPLICATE_NAME", message: "A category named '{name}' already exists in this budget" } }
Side effects:
  -- Sibling positions incremented if inserted mid-sequence
  -- Parent budget's search_index entry updated synchronously
```

```
PATCH /api/budgets/categories/:id
Channel: budget:category:update
Request body:  { name: string?, position: number? }
Response 200:  same shape as POST /api/budgets/:budget_id/categories response
Response 404:  { error: { code: "NOT_FOUND", message: "Category not found" } }
Response 409:  { error: { code: "DUPLICATE_NAME", message: "A category named '{name}' already exists in this budget" } }
Side effects:
  -- Sibling positions recompacted if position changed
  -- Parent budget's search_index entry updated synchronously if name changed
```

```
DELETE /api/budgets/categories/:id
Channel: budget:category:delete
Response 409: category contains line items — confirm required
  {
    error: {
      code: "DELETE_REQUIRES_CONFIRMATION",
      message: "This category contains line items that will be deleted",
      line_item_count: number
    }
  }
Response 200: (empty category — no confirm needed)
  { deleted_id: string, cascade_summary: { line_items_deleted: number } }
Response 404: { error: { code: "NOT_FOUND", message: "Category not found" } }
Side effects: none until confirmed
```

```
DELETE /api/budgets/categories/:id/confirm
Channel: budget:category:delete  + confirm: true
Response 200: { deleted_id: string, cascade_summary: { line_items_deleted: number } }
Response 404: { error: { code: "NOT_FOUND", message: "Category not found" } }
Side effects:
  -- All line items hard deleted; sibling positions compacted
  -- Parent budget's search_index entry updated synchronously
  -- Applies equally when an empty category is deleted without the confirm step
```

---

### Budget Line Items

```
POST /api/budgets/categories/:category_id/line-items
Channel: budget:line-item:create
Request body:
  {
    name:     string   required
    budgeted: number?  -- non-negative; null = not yet entered
    actual:   number?  -- non-negative; null = not yet entered
    notes:    string?
    position: number?
  }
Response 201:
  {
    id:              string,
    category_id:     string,
    name:            string,
    position:        number,
    budgeted:        number | null,
    actual:          number | null,
    difference:      number | null,  -- computed
    variance_status: string | null,  -- computed
    notes:           string | null,
    created_at:      string,
    updated_at:      string
  }
Response 400: name is empty, or budgeted/actual is negative
  { error: { code: "VALIDATION_ERROR", message: "Cost values cannot be negative" } }
Response 404: { error: { code: "NOT_FOUND", message: "Category not found" } }
Side effects:
  -- Sibling positions incremented if inserted mid-sequence
  -- Parent budget's search_index entry updated synchronously
```

```
PATCH /api/budgets/line-items/:id
Channel: budget:line-item:update
Request body:
  {
    name:     string?
    budgeted:    number?  -- pass null to clear
    actual:      number?  -- pass null to clear
    notes:       string?
    position:    number?
    category_id: string?  -- move to another category of the same budget (drag between
                          -- categories); lands at position, or at the end
  }
Response 200: same shape as POST /api/budgets/categories/:category_id/line-items response
Response 400: budgeted or actual is negative
  { error: { code: "VALIDATION_ERROR", message: "Cost values cannot be negative" } }
Response 400: category_id belongs to another budget
  { error: { code: "VALIDATION_ERROR", message: "A line can only move within its own budget" } }
Response 404: { error: { code: "NOT_FOUND", message: "Line item not found" } }
Side effects:
  -- Sibling positions recompacted if position changed; on a move, both categories
  -- Parent budget's search_index entry updated synchronously if name changed
```

```
DELETE /api/budgets/line-items/:id
Channel: budget:line-item:delete
Response 200:  { deleted_id: string }
Response 404:  { error: { code: "NOT_FOUND", message: "Line item not found" } }
Side effects:
  -- Sibling positions compacted
  -- Parent budget's search_index entry updated synchronously
```

---

## Module 4 — Tips & Lessons Learned

**IPC prefix:** `tips`

Tips are their own module (see the Module Map in `architecture.md`), not part of Content.

---

### Tips

```
GET /api/tips
Channel: tips:tip:list
Query params:
  type:       string?  -- Tip | LessonLearned
  scope:      string?  -- General | Destination
  country_id: string?
  region_id:  string?
  city_id:    string?
  trip_id:    string?  -- filter to tips sourced from this trip (LessonLearned only)
  unlinked:   boolean? -- true = only Unlinked tips (scope = Destination, destination
                       --        deleted); false = exclude them
  source_trip_removed: boolean? -- true = only Lessons Learned whose source trip was deleted
Response 200:
  {
    tips: [
      {
        id:               string,
        type:             string,
        scope:            string,
        title:            string,
        content:          string | null,   -- TipTap JSON
        country_id:       string | null,
        country_name:     string | null,
        region_id:        string | null,
        region_name:      string | null,
        city_id:          string | null,
        city_name:        string | null,
        is_unlinked:        boolean,
          -- true when scope = Destination and no destination is linked.
          -- Shown as "needs reassignment" (US-051).
        former_destination: string | null,
          -- display path of the deleted destination, e.g. "Thailand > Chiang Mai"
        source_trip_id:   string | null,
        source_trip_name: string | null,
        former_source_trip: string | null,
          -- name of the deleted source trip; non-null = "trip link removed" flag (US-052)
        origin_tip:       { id: string, title: string } | null,
          -- LessonLearned only: the Tip this was written as a follow-up to (US-055)
        follow_ups:       [{ id: string, title: string, created_at: string }],
          -- Tip only: Lessons Learned written as follow-ups to it, oldest first
        created_at:       string,
        updated_at:       string
      }
    ]
  }
Side effects: none
```

```
GET /api/tips/:id
Channel: tips:tip:get
Response 200: single tip object — same shape as item in GET /api/tips
Response 404: { error: { code: "NOT_FOUND", message: "Tip not found" } }
Side effects: none
```

```
POST /api/tips
Channel: tips:tip:create
Request body:
  {
    type:           string   required  -- Tip | LessonLearned
    scope:          string   required  -- General | Destination
    title:          string   required
    content:        string?            -- TipTap JSON
    -- Destination FK: exactly one required when scope = Destination;
    -- all must be omitted when scope = General
    country_id:     string?
    region_id:      string?
    city_id:        string?
    -- Source trip: optional; only valid when type = LessonLearned;
    -- referenced trip must have status = Completed
    source_trip_id: string?
  }
Response 201: same shape as GET /api/tips/:id
Response 400: title is empty
  { error: { code: "VALIDATION_ERROR", message: "Title is required" } }
Response 400: scope = Destination but no destination FK provided
  { error: { code: "VALIDATION_ERROR", message: "A destination must be specified for destination-scoped tips" } }
Response 400: scope = Destination and more than one destination FK provided
  { error: { code: "VALIDATION_ERROR", message: "Only one destination level can be specified" } }
Response 400: scope = General but a destination FK was provided
  { error: { code: "VALIDATION_ERROR", message: "General tips cannot be linked to a destination" } }
Response 400: type = Tip but source_trip_id provided
  { error: { code: "VALIDATION_ERROR", message: "source_trip_id is only valid for Lesson Learned type" } }
Response 404: destination record or source_trip_id not found
  { error: { code: "NOT_FOUND", message: "Destination record not found" } }
Response 409: source_trip_id references a non-Completed trip
  { error: { code: "INVALID_TRIP_STATUS", message: "Lesson Learned source trip must have Completed status" } }
Side effects: search_index updated synchronously
```

```
PATCH /api/tips/:id
Channel: tips:tip:update
Request body:
  {
    type:           string?
    scope:          string?
    title:          string?
    content:        string?  -- TipTap JSON
    country_id:     string?  -- pass null to detach
    region_id:      string?  -- pass null to detach
    city_id:        string?  -- pass null to detach
    source_trip_id: string?  -- pass null to detach
    former_source_trip: null? -- pass null to dismiss the "trip link removed" flag
  }
Response 200: same shape as GET /api/tips/:id
Response 400: title is empty string
  { error: { code: "VALIDATION_ERROR", message: "Title cannot be empty" } }
Response 400: scope changing to General but destination FKs still set
  { error: { code: "VALIDATION_ERROR", message: "Clear destination links before changing scope to General" } }
Response 400: scope changing to Destination but no destination FK provided or remaining
  { error: { code: "VALIDATION_ERROR", message: "A destination must be specified for destination-scoped tips" } }
Response 400: destination FKs cleared on a linked destination-scoped tip without setting another
  { error: { code: "VALIDATION_ERROR", message: "A destination must be specified for destination-scoped tips" } }
Response 400: type changing to Tip but source_trip_id is set
  { error: { code: "VALIDATION_ERROR", message: "Remove source_trip_id before changing type to Tip" } }
Response 400: type changing to LessonLearned on a Tip that has follow-ups
  { error: { code: "VALIDATION_ERROR", message: "This tip has Lesson Learned follow-ups and must stay a Tip" } }
Response 400: more than one destination FK set after patch applied
  { error: { code: "VALIDATION_ERROR", message: "Only one destination level can be specified" } }
Response 404: tip, destination record, or source_trip_id not found
  { error: { code: "NOT_FOUND", message: "Tip not found" } }
Response 409: source_trip_id references a non-Completed trip
  { error: { code: "INVALID_TRIP_STATUS", message: "Lesson Learned source trip must have Completed status" } }
Side effects:
  -- Server validates resulting state after patch applied — not just fields sent
  -- Client must explicitly clear destination FKs in same request as scope change
  -- Unlinked tips: a tip that is already Unlinked may be saved with no destination,
     so title, content and type stay editable. Setting a destination re-links it;
     changing scope to General is also allowed. Both clear former_destination.
     A linked tip cannot be made Unlinked through this endpoint.
  -- The Completed-status check on source_trip_id applies only when the link is
     set or changed. An existing link is kept if that trip later leaves Completed.
  -- former_source_trip cleared when source_trip_id is set or type changes to Tip
  -- Changing a follow-up's type to Tip also clears its origin_tip_id. Changing type
     here corrects a misfiled record; to record what a trip taught you about a
     tip, add a follow-up instead
  -- search_index updated synchronously
```

```
DELETE /api/tips/:id
Channel: tips:tip:delete
Response 200:  { deleted_id: string }
Response 404:  { error: { code: "NOT_FOUND", message: "Tip not found" } }
Side effects:
  -- search_index entry removed synchronously
  -- Note: deleting a destination record does not cascade-delete its tips —
     tips become Unlinked (destination FK nulled, former_destination set, scope
     retained). Find them with GET /api/tips?unlinked=true.
     This endpoint is for explicit tip deletion only.
  -- Deleting a Tip that has follow-ups keeps them; their origin_tip becomes null
```

---

### Lesson Learned Follow-up

A tip is never converted in place. Recording what a trip taught you about a tip creates
a **new** Lesson Learned linked to it, so the pre-trip research and the post-trip
reality stay side by side (US-055).

```
POST /api/tips/:id/follow-ups
Channel: tips:tip:add-follow-up
  -- :id is the original Tip
Request body:
  {
    title:          string?  -- default: the original tip's title
    content:        string?  -- TipTap JSON
    source_trip_id: string?  -- optional Completed trip the lesson came from
  }
Response 201:  same shape as GET /api/tips/:id — the new Lesson Learned, with
               origin_tip set to the original
Response 400:  the original is a Lesson Learned
  { error: { code: "VALIDATION_ERROR", message: "A follow-up can only be added to a Tip" } }
Response 400:  the original is Unlinked
  { error: { code: "VALIDATION_ERROR", message: "Reassign this tip to a destination before adding a follow-up" } }
Response 404:  tip or source_trip_id not found
  { error: { code: "NOT_FOUND", message: "Tip not found" } }
Response 409:  source_trip_id references a non-Completed trip
  { error: { code: "INVALID_TRIP_STATUS", message: "Source trip must have Completed status" } }
Side effects:
  -- New tip created with type = LessonLearned, origin_tip_id = :id
  -- Scope and destination copied from the original (a General tip yields a General
     Lesson Learned); they can be edited afterwards like any other tip
  -- The original Tip is not modified
  -- search_index entry created synchronously
```

---

## Module 5 — Media

**IPC prefix:** `media`

**Key behaviours:**

- Files copied into app data directory on import — original path not stored
- Deleting a media record does not delete the file immediately — purge required
- A file may be referenced by more than one media record. A preserved deleted trip
  (trip snapshot) owns its own records for the images it shows; they are read-only,
  are not returned by `GET /api/media`, and are removed only with the snapshot
- Sharp runs in Electron main process for all image processing
- Images served to renderer via custom `app://` Electron protocol — never raw file:// paths
- `app://` protocol is registered in the Electron main process via `protocol.handle`
- The main process decides what a file is. The image type is detected from the file's
  contents; the renderer never supplies a mime type
- The renderer obtains a source path either from `POST /api/media/pick` (native file
  dialog) or, for drag and drop, from the preload helper that wraps
  `webUtils.getPathForFile`. Web `File` objects carry no path in Electron

---

### Media Import

```
POST /api/media/pick
Channel: media:image:pick
  -- Opens the native file dialog from the main process (dialog.showOpenDialog),
  -- filtered to supported image types. Nothing is imported by this call.
Request body:  { multiple: boolean? }   -- default: true
Response 200:  { paths: [string] }      -- absolute paths; empty array if the user cancelled
Side effects:  none
```

```
POST /api/media/import
Channel: media:image:import
Request body:
  {
    source_path:  string   required  -- absolute local file path, from /api/media/pick
                                     -- or the drag-and-drop preload helper
    -- Exactly one parent FK required:
    country_id:   string?
    region_id:    string?
    city_id:      string?
    poi_id:       string?
    trip_id:      string?
    is_hero:      boolean?           -- default: false
    position:     number?            -- if omitted, appended to end
  }
Response 201:
  {
    id:            string,
    filename:      string,
    original_name: string,
    mime_type:     string,
    size_bytes:    number,
    app_url:       string,        -- app://media/{filename}
    thumbnail_url: string,        -- app://media/thumbnails/{filename}
    country_id:    string | null,
    region_id:     string | null,
    city_id:       string | null,
    poi_id:        string | null,
    trip_id:       string | null,
    is_hero:       boolean,
    position:      number,
    created_at:    string
  }
Response 400: source_path missing, or the file's contents are not a supported image type
  { error: { code: "VALIDATION_ERROR", message: "Unsupported file type. Supported: jpeg, png, webp, gif" } }
Response 400: zero or more than one parent FK provided
  { error: { code: "VALIDATION_ERROR", message: "Exactly one parent record must be specified" } }
Response 404: source file not found at source_path
  { error: { code: "FILE_NOT_FOUND", message: "Source file not found at the specified path" } }
Response 404: parent record not found
  { error: { code: "NOT_FOUND", message: "Parent record not found" } }
Response 422: file corrupt or unprocessable by Sharp
  { error: { code: "FILE_PROCESSING_ERROR", message: "File could not be processed. It may be corrupt or in an unsupported format." } }
Side effects:
  -- Image type detected from file contents by the main process; mime_type in the
     response is the detected type
  -- File copied to app data dir with cuid2-based filename
  -- Thumbnail generated and written to app data dir /thumbnails/
  -- If is_hero = true: existing is_hero = 1 on this parent demoted to 0
  -- Sibling positions incremented if inserted mid-sequence
```

---

### Media Retrieval

```
GET /api/media
Channel: media:image:list
Query params: exactly one parent FK required
  country_id: string?
  region_id:  string?
  city_id:    string?
  poi_id:     string?
  trip_id:    string?
Response 200:
  {
    media: [
      {
        id:            string,
        filename:      string,
        original_name: string,
        mime_type:     string,
        size_bytes:    number,
        app_url:       string,
        thumbnail_url: string,
        is_hero:       boolean,
        position:      number,
        created_at:    string
      }
    ]
  }
Response 400: no parent FK or more than one provided
  { error: { code: "VALIDATION_ERROR", message: "Exactly one parent record must be specified" } }
Response 404: { error: { code: "NOT_FOUND", message: "Parent record not found" } }
Side effects: none
```

```
GET /api/media/:id
Channel: media:image:get
Response 200: single media object — same shape as item in GET /api/media
Response 404: { error: { code: "NOT_FOUND", message: "Media record not found" } }
Side effects: none
```

---

### Media Updates

```
PATCH /api/media/:id
Channel: media:image:update
Request body:
  {
    is_hero:  boolean?
    position: number?
  }
Response 200: same shape as GET /api/media/:id
Response 400: no fields provided
  { error: { code: "VALIDATION_ERROR", message: "At least one field must be provided" } }
Response 404: { error: { code: "NOT_FOUND", message: "Media record not found" } }
Response 409: { error: { code: "RECORD_IS_DELETED", message: "Cannot update media on a preserved historical record" } }
Side effects:
  -- If is_hero = true: existing hero on same parent demoted to is_hero = 0
  -- If is_hero = false and this was the hero: no replacement hero designated automatically
  -- Sibling positions recompacted if position changed
```

---

### Hero Image

```
POST /api/media/:id/set-hero
Channel: media:image:set-hero
Request body:  none
Response 200:  same shape as GET /api/media/:id with is_hero: true
Response 404:  { error: { code: "NOT_FOUND", message: "Media record not found" } }
Response 409:  { error: { code: "RECORD_IS_DELETED", message: "Cannot update media on a preserved historical record" } }
Side effects:
  -- Existing hero on same parent set to is_hero = 0
  -- This record set to is_hero = 1
```

---

### Media Deletion

```
DELETE /api/media/:id
Channel: media:image:delete
Response 200:
  {
    deleted_id: string,
    filename:   string,   -- queued for purge; not yet deleted from disk
    was_hero:   boolean
  }
Response 404: { error: { code: "NOT_FOUND", message: "Media record not found" } }
Response 409: { error: { code: "RECORD_IS_DELETED", message: "Cannot delete media on a preserved historical record" } }
Side effects:
  -- media row deleted; file remains on disk until purge
  -- The file is kept by purge if another media row (a snapshot copy) still references it
  -- Sibling positions compacted
  -- If was_hero = true: no replacement hero designated automatically
```

---

### Purge Orphaned Files

```
POST /api/media/purge
Channel: media:storage:purge
  -- Removes files (and their thumbnails) in the app data media directory whose
  -- filename is not referenced by any media row, including snapshot-owned rows.
  -- Runs automatically on app startup; also available to trigger manually.
Request body:  none
Response 200:
  {
    files_scanned: number,
    files_deleted: number,
    bytes_freed:   number,
    errors: [{ filename: string, error: string }]
  }
Side effects: orphaned image and thumbnail files permanently deleted from disk
```

---

## Module 6 — Search

**IPC prefix:** `search`

**Key behaviours:**

- All queries run against the FTS5 `search_index` table, joined through `search_docs`
  to each hit's live source row (Search Query Pattern in `architecture.md`)
- Searchable text is each record's own name/title and its notes, description or content.
  Budgets also match on their category and line item names. Parent names and tag
  values are not searchable text — use the country, scope and tag filters for those
- `title`, `parent_context` and all filters are read from the live records when the
  query runs, so results never show a renamed parent's old name or a stale status
- `q` is treated as plain words, never as FTS5 query syntax. Every term must match
  (AND); the last term also matches as a prefix, so results narrow as the user types.
  Quotes, hyphens and words like AND / OR / NOT in `q` are searched literally
- Ranked by relevance with a match in the title weighted above a match in the body
- Search is always synchronous — no loading state expected for typical library sizes
- Case-insensitive; porter stemming applied (e.g. "hiking" matches "hike")
- **Filter-only browsing.** `GET /api/search` also works with no keyword when at least
  one of `country_id`, `status` or `tag_id` is given. This is how the Countries list
  and country views filter by tag without a search term (US-028). The Trips list
  filters through `GET /api/trips` instead. Results come straight from the source
  tables, ordered by title
- An empty or whitespace-only query string with no filters returns an empty result set —
  not an error
- Single-character queries permitted
- A filter that has no meaning for a content type excludes that type: `status` returns
  only trips (the only content with a status, D-09); `tag_id` returns only the content
  type the tag belongs to;
  `country_id` excludes General and Unlinked tips

---

### Global Search

```
GET /api/search
Channel: search:global:query
Query params:
  q            string?            -- keyword; optional when country_id, status or tag_id is given
  content_type string?            -- country | region | city | poi | trip | tip | budget
  country_id   string?            -- filter to content under this country
  status       string?            -- trip status (Sample | Planning | Ready | Completed);
                                  -- returns trips only. Repeatable; OR-ed
  tag_id       string?            -- repeatable: tag_id=x&tag_id=y; AND-ed
  limit        number?            -- default: 50; max: 200
  offset       number?            -- default: 0
Response 200:
  {
    query:   string,
    total:   number,
    limit:   number,
    offset:  number,
    results: [
      {
        record_id:      string,
        content_type:   string,
        title:          string,
        parent_context: string,  -- built from the live hierarchy at query time:
                                 --   region / city / poi: "Thailand > Northern Thailand > Chiang Mai"
                                 --   trip:    linked country names, e.g. "Thailand, Laos"
                                 --   budget:  its trip's name
                                 --   tip:     destination path, "General", or "Unlinked"
                                 --   country: empty string
        snippet:        string,  -- FTS5 snippet ~30 words, taken from the body text;
                                 -- empty when the record has no body text
        rank:           number   -- bm25 relevance (title weighted 10x body); lower = more relevant
      }
    ]
  }
Response 400: content_type is not a valid value
  { error: { code: "VALIDATION_ERROR", message: "Invalid content_type value" } }
Side effects: none
  -- Multiple tag_ids are AND-ed — results must carry all specified tags.
  -- A tag belongs to one content type, so any tag_id limits results to that type
     (US-028). tag_ids from two different content types therefore return nothing.
  -- country_id matches: the country itself; regions, cities and POIs in it; trips
     linked to it; budgets on those trips; tips whose destination is in it.
  -- status matches trips.status only. Several status values are OR-ed.
  -- Without q: results are ordered by title, snippet is an empty string and rank is 0.
```

---

### In-Context Search — Trip Builder

```
GET /api/search/trip/:trip_id
Channel: search:trip:query
Query params:
  q            string   required
  content_type string?            -- poi | country | region | city; default: poi
  expand_scope boolean?           -- default: false; true = search full library
  tag_id       string?            -- repeatable; filters POIs by tag
  limit        number?            -- default: 30
Response 200:
  {
    query:       string,
    trip_id:     string,
    scope:       string,          -- trip_countries | full_library
    country_ids: [string],
    results: [
      {
        record_id:      string,
        content_type:   string,
        title:          string,
        parent_context: string,
        snippet:        string,
        rank:           number,
        poi_detail: {
          country_id:      string,
          country_name:    string,
          city_id:         string | null,
          city_name:       string | null,
          tags:            [{ id: string, category: string, value: string }],
          has_description: boolean,
          has_notes:       boolean
        } | null   -- present only when content_type = poi
      }
    ]
  }
Response 400: { error: { code: "VALIDATION_ERROR", message: "Search query is required" } }
Response 404: { error: { code: "NOT_FOUND", message: "Trip not found" } }
Side effects: none
  -- If trip has no linked countries, scope defaults to full_library regardless
     of expand_scope value.
```

---

### In-Context Search — Country / Region / City

```
GET /api/search/context
Channel: search:context:query
Query params:
  q            string   required
  country_id   string?            -- scope to this country and all nested content
  region_id    string?            -- scope to this region and all nested content
  city_id      string?            -- scope to this city and all nested content
  -- Exactly one scope FK required
  content_type string?
  limit        number?            -- default: 30
Response 200:
  {
    query:      string,
    scope_type: string,           -- country | region | city
    scope_id:   string,
    results: [
      {
        record_id:      string,
        content_type:   string,
        title:          string,
        parent_context: string,
        snippet:        string,
        rank:           number
      }
    ]
  }
Response 400: zero or more than one scope FK provided
  { error: { code: "VALIDATION_ERROR", message: "Exactly one scope must be specified" } }
Response 400: { error: { code: "VALIDATION_ERROR", message: "Search query is required" } }
Response 404: { error: { code: "NOT_FOUND", message: "Scope record not found" } }
Side effects: none
  -- City scope: POIs and tips linked to that city
  -- Region scope: cities, POIs, and tips in that region or its cities
  -- Country scope: all nested content
  -- Scope is resolved through the live hierarchy, so a POI linked only to a city
     is found by a search scoped to that city's region
  -- Trips and budgets appear in country scope only
```

---

### Recently Used (Trip Builder)

```
GET /api/search/trip/:trip_id/recent
Channel: search:trip:recent
Query params:
  content_type string?  -- poi | country | region | city; default: poi
  limit        number?  -- default: 10
Response 200:
  {
    trip_id: string,
    results: [
      {
        record_id:      string,
        content_type:   string,
        title:          string,
        parent_context: string,
        last_used_at:   string   -- most recent use in any trip day across all trips
      }
    ]
  }
Response 404: { error: { code: "NOT_FOUND", message: "Trip not found" } }
Side effects: none
```

---

### Search Index Rebuild

```
POST /api/search/index/rebuild
Channel: search:index:rebuild
Request body:  none
Response 200:
  {
    records_indexed: number,
    duration_ms:     number
  }
Side effects:
  -- search_docs and search_index emptied and rebuilt from the source tables
     in one transaction
  -- Synchronous; blocks until complete
  -- Runs automatically on startup when the number of search_docs rows does not
     match the number of indexable records, and after a migration that changes
     what is indexed. Not otherwise needed during normal operation
```

---

## Module 7 — Travel Windows

**IPC prefix:** `travelwindow`

---

### Travel Windows

```
GET /api/travel-windows
Channel: travelwindow:window:list
Query params:  is_archived  boolean?  -- default: false
Response 200:
  {
    travel_windows: [
      {
        id:          string,
        name:        string,
        target_date: string | null,   -- ISO date (YYYY-MM-DD)
        target_label: string | null,  -- optional display text, e.g. "November 2027"
        is_archived: boolean,
        created_at:  string,
        updated_at:  string,
        counts: { trips: number, chosen: number, completed: number }
      }
    ]
  }
  -- Ordered by target_date ascending; windows with no target_date come after every
     dated one, oldest created first, and the list shows them as "No date yet" (US-030).
  -- target_label is display text only and never affects the order.
  -- The renderer splits Upcoming and Past by target_date against today; an undated
     window is always Upcoming.
Side effects: none
```

```
GET /api/travel-windows/:id
Channel: travelwindow:window:get
Response 200:
  {
    id:          string,
    name:        string,
    target_date: string | null,   -- ISO date (YYYY-MM-DD)
    target_label: string | null,  -- optional display text, e.g. "November 2027"
    is_archived: boolean,
    created_at:  string,
    updated_at:  string,
    trips: [
      {
        id:                string,        -- travel_window_trips.id
        trip_id:           string | null,
        trip_name:         string,
        position:          number,
        is_chosen:         boolean,
        is_deleted_record: boolean,
        deleted_at:        string | null,
        -- For a preserved record (is_deleted_record = true) trip_id is null and the
        -- fields below are read from the trip snapshot, in the same shape as a live
        -- trip. IDs inside them are historical and must not be dereferenced.
        status:            string,
        completed_at:      string | null,
        notes:             string | null,  -- TipTap JSON
        countries:         [{ id: string, name: string }],
        tags:              [{ id: string, category: string, value: string }],
        length:            { days: number, complete: boolean },
        blocks:            [ <summary block> ],  -- same shape as in GET /api/presentation/:window_id,
                                                 -- without highlights; drives the route text
        hero_image:        { app_url: string, thumbnail_url: string } | null,
                                                 -- hero rule in Module 8 Key behaviours
        budget_snapshots: [
          {
            id:               string,
            name:             string,
            is_primary:       boolean,
            total_budgeted:   number | null,
            total_actual:     number | null,
            total_difference: number | null,
            actuals_complete: boolean,
            cost_per_day:     number | null,
            variance_status:  string | null
          }
        ]
      }
    ]
  }
Response 404: { error: { code: "NOT_FOUND", message: "Travel Window not found" } }
Side effects: none
  -- trips are in position order — the order presentation uses
```

```
POST /api/travel-windows
Channel: travelwindow:window:create
Request body:
  {
    name:        string  required
    target_date:  string?  -- ISO date (YYYY-MM-DD); for a period, its first day
    target_label: string?  -- optional display text, e.g. "November 2027"
  }
Response 201:
  {
    id:          string,
    name:        string,
    target_date: string | null,   -- ISO date (YYYY-MM-DD)
    target_label: string | null,  -- optional display text, e.g. "November 2027"
    is_archived: boolean,
    created_at:  string,
    updated_at:  string,
    trips:       []
  }
Response 400: { error: { code: "VALIDATION_ERROR", message: "Name is required" } }
Response 400: target_date is not a valid ISO date
  { error: { code: "VALIDATION_ERROR", message: "target_date must be a date in YYYY-MM-DD format" } }
Response 409: { error: { code: "DUPLICATE_NAME", message: "A Travel Window named '{name}' already exists" } }
Side effects: none
```

```
PATCH /api/travel-windows/:id
Channel: travelwindow:window:update
Request body:
  {
    name:        string?
    target_date:  string?  -- ISO date (YYYY-MM-DD); pass null to clear
    target_label: string?  -- pass null to clear
  }
Response 200: same shape as GET /api/travel-windows/:id
Response 400: { error: { code: "VALIDATION_ERROR", message: "Name cannot be empty" } }
Response 404: { error: { code: "NOT_FOUND", message: "Travel Window not found" } }
Response 409: { error: { code: "DUPLICATE_NAME", message: "A Travel Window named '{name}' already exists" } }
Side effects: none
```

```
DELETE /api/travel-windows/:id
Channel: travelwindow:window:delete
Response 409: window holds preserved historical records — confirm required
  {
    error: {
      code: "DELETE_REQUIRES_CONFIRMATION",
      message: "This Travel Window holds preserved records of deleted trips that will be permanently lost",
      preserved_records: [{ id: string, trip_name: string, deleted_at: string, in_other_windows: boolean }]
        -- in_other_windows = true: the record survives in another Travel Window
    }
  }
Response 200: (no preserved records — no confirm needed)
  {
    deleted_id: string,
    cascade_summary: { trip_references_removed: number, preserved_records_destroyed: number }
  }
Response 404: { error: { code: "NOT_FOUND", message: "Travel Window not found" } }
Side effects: none until confirmed
  -- The user-facing "are you sure" prompt for a window with no preserved records
     (US-036) is a renderer concern
```

```
DELETE /api/travel-windows/:id/confirm
Channel: travelwindow:window:delete  + confirm: true
Response 200: same shape as DELETE /api/travel-windows/:id success response
Response 404: { error: { code: "NOT_FOUND", message: "Travel Window not found" } }
Side effects:
  -- travel_window_trips rows hard deleted
  -- Each trip snapshot no longer shown in any Travel Window is deleted with its
     media rows; its image files are removed by the next purge unless a library
     record still references them
  -- Master trip records unaffected
```

---

### Archive / Restore

```
POST /api/travel-windows/:id/archive
Channel: travelwindow:window:archive
Response 200:  { id: string, is_archived: true, updated_at: string }
Response 404:  { error: { code: "NOT_FOUND", message: "Travel Window not found" } }
Response 409:  { error: { code: "ALREADY_ARCHIVED", message: "Travel Window is already archived" } }
Side effects:  is_archived set to 1
```

```
POST /api/travel-windows/:id/restore
Channel: travelwindow:window:restore
Response 200:  { id: string, is_archived: false, updated_at: string }
Response 404:  { error: { code: "NOT_FOUND", message: "Travel Window not found" } }
Response 409:  { error: { code: "NOT_ARCHIVED", message: "Travel Window is not archived" } }
Side effects:  is_archived set to 0
```

---

### Shortlisted Trips

```
POST /api/travel-windows/:window_id/trips
Channel: travelwindow:trip:add
Request body:
  {
    trip_id:  string  required
    position: number?
  }
Response 201:
  {
    id:                string,
    window_id:         string,
    trip_id:           string,
    trip_name:         string,
    position:          number,
    is_chosen:         boolean,
    is_deleted_record: false,
    created_at:        string
  }
Response 400: { error: { code: "VALIDATION_ERROR", message: "trip_id is required" } }
Response 404: { error: { code: "NOT_FOUND", message: "Trip not found" } }
Response 409: trip already in this window
  { error: { code: "DUPLICATE_TRIP", message: "This trip is already shortlisted in this Travel Window" } }
Response 409: window already has 3+ trips — soft warning
  {
    error: {
      code: "SHORTLIST_LIMIT_WARNING",
      message: "Travel Windows work best with 2–3 trips. Adding a 4th is allowed but not recommended.",
      current_count: number
    }
  }
  -- Soft warning — use /confirm endpoint to proceed after user acknowledges
Side effects: sibling positions incremented if inserted mid-sequence
```

```
POST /api/travel-windows/:window_id/trips/confirm
Channel: travelwindow:trip:add  + confirm: true
  -- Confirms adding 4th+ trip after soft warning acknowledged
Request body:  { trip_id: string  required, position: number? }
Response 201:  same shape as POST /api/travel-windows/:window_id/trips
Response 404:  { error: { code: "NOT_FOUND", message: "Trip not found" } }
Response 409:  { error: { code: "DUPLICATE_TRIP", message: "This trip is already shortlisted" } }
Side effects:  same as POST /api/travel-windows/:window_id/trips
```

```
PATCH /api/travel-windows/trips/:id
Channel: travelwindow:trip:update
Request body:
  {
    is_chosen: boolean?
    position:  number?
  }
Response 200:
  {
    id:         string,
    window_id:  string,
    trip_id:    string | null,
    is_chosen:  boolean,
    position:   number,
    updated_at: string
  }
Response 400: { error: { code: "VALIDATION_ERROR", message: "At least one field must be provided" } }
Response 404: { error: { code: "NOT_FOUND", message: "Travel Window trip not found" } }
Response 409: { error: { code: "RECORD_IS_DELETED", message: "Cannot update a preserved historical record" } }
Side effects:
  -- is_chosen does not automatically de-choose others — use /choose endpoint
  -- Sibling positions recompacted if position changed
```

```
DELETE /api/travel-windows/trips/:id
Channel: travelwindow:trip:remove
Response 200:  { deleted_id: string, trip_id: string }
Response 404:  { error: { code: "NOT_FOUND", message: "Travel Window trip not found" } }
Response 409:  { error: { code: "RECORD_IS_DELETED", message: "Cannot remove a preserved historical record individually. Delete the Travel Window to remove all records." } }
Side effects:
  -- travel_window_trips row hard deleted; sibling positions compacted
  -- Master trip record unaffected
```

---

### Chosen Trip Flag

```
POST /api/travel-windows/:window_id/trips/:id/choose
Channel: travelwindow:trip:choose
  -- Atomically marks one trip as chosen and de-chooses all others in window
Request body:  none
Response 200:
  {
    window_id:  string,
    chosen_id:  string,    -- travel_window_trips.id now chosen
    de_chosen:  [string]   -- travel_window_trips.ids de-chosen
  }
Response 404: { error: { code: "NOT_FOUND", message: "Travel Window trip not found" } }
Response 409: { error: { code: "RECORD_IS_DELETED", message: "Cannot choose a preserved historical record" } }
Side effects: all other is_chosen flags in window set to 0; this entry set to 1
```

```
POST /api/travel-windows/:window_id/unchoose
Channel: travelwindow:trip:unchoose
  -- Clears chosen flag on all trips in window
Request body:  none
Response 200:  { window_id: string, de_chosen: [string] }
Response 404:  { error: { code: "NOT_FOUND", message: "Travel Window not found" } }
Side effects:  all is_chosen flags in window set to 0
```

---

## Module 8 — Presentation Mode

**IPC prefix:** `presentation`

**Key behaviours:**

- All endpoints are read-only — no mutations permitted
- Data assembled from underlying records — no separate presentation storage
- Deleted-record entries render from the trip snapshot with identical response shape to live trips
- The trip snapshot **is** the full-detail payload below (minus the per-window fields),
  stored at deletion with a `snapshot_version`. Summary and comparison entries for a
  preserved record are projections of it. This module supplies the payload when the
  Travel Windows module captures a snapshot, and upgrades older versions when they are read
- Keyboard navigation state managed entirely client-side
- **Hero image** (US-041; handoff 12). Every `hero_image` in this module, the Travel
  Window detail and the Trips list is chosen by the same rule, first match wins:
  1. the trip's designated hero (`media.is_hero` on the trip)
  2. the trip's first photo by `position`
  3. walking the trip's location blocks in order, the block's most specific place first
     (city, then region, then country): that place's designated hero, else its first photo
  4. the first photo of a POI placed on the trip's days, in day and position order
  5. a country linked to the trip with no block behind it: hero, else first photo
  6. none — `hero_image: null`, and the renderer shows the neutral placeholder

  `hero_image.source` names where it came from (`trip | city | region | country | poi`).
  Nothing is ever designated automatically: the rule only chooses what to show
- **The window must have a trip.** Presentation needs at least one shortlisted trip;
  the Travel Window disables Present with "Add a trip first" (US-037), and these
  channels return `EMPTY_WINDOW` as a backstop

---

### Load Presentation

```
GET /api/presentation/:window_id
Channel: presentation:window:get
  -- Full presentation payload; single call, no further round trips needed
  -- for summary and comparison views.
Response 200:
  {
    window_id:   string,
    window_name: string,
    target_date: string | null,   -- ISO date (YYYY-MM-DD)
    target_label: string | null,  -- optional display text, e.g. "November 2027"
    trip_count:  number,
    trips: [
      {
        id:                string,
        trip_id:           string | null,
        position:          number,
        is_chosen:         boolean,
        is_deleted_record: boolean,
        deleted_at:        string | null,
        name:              string,
        status:            string | null,
        completed_at:      string | null,
        notes:             string | null,   -- TipTap JSON
        countries:         [{ id: string, name: string }],
        tags:              [{ id: string, category: string, value: string }],
        length:            { days: number, complete: boolean },
        blocks: [
          {
            -- <summary block>
            id:            string,
            position:      number,
            kind:          string,         -- location | travel
            country_name:  string | null,
            region_name:   string | null,
            city_name:     string | null,
            travel_mode:   string | null,
            from_label:    string | null,
            to_label:      string | null,
            duration_days: number | null,
            has_days:      boolean,
            highlights: {                  -- "Highlights by stop" (D-14); location blocks only
              pois: [{ poi_id: string, poi_name: string, is_must_do: boolean }],
                -- up to 2 POIs placed on this block's days, Must Do first, then by day
                   and position. Never drawn from the library — a block with no days
                   or no POIs has an empty list ("No places picked yet")
              more_count: number           -- further distinct POIs on its days ("+N more places")
            } | null                       -- null on travel blocks
          }
        ],
        budget_snapshots: [
          {
            id:               string,
            name:             string,
            is_primary:       boolean,
            total_budgeted:   number | null,
            total_actual:     number | null,
            total_difference: number | null,
            actuals_complete: boolean,
            cost_per_day:     number | null,
            variance_status:  string | null,
            category_split: [              -- "Where the money goes" (D-14); primary only,
              {                            -- empty on other budgets
                name:     string,
                budgeted: number           -- category subtotal in cents; categories with
              }                            -- no budgeted amount are left out
            ]
          }
        ],
        hero_image: {
          app_url:       string,
          thumbnail_url: string,
          source:        string            -- trip | city | region | country | poi
        } | null,
        thumbnails: [{ app_url: string, thumbnail_url: string }],
          -- up to 3 more photos for the summary strip, taken by the same walk as the
             hero rule, skipping the hero; empty when there are none
        media: [
          {
            id:            string,
            app_url:       string,
            thumbnail_url: string,
            is_hero:       boolean,
            position:      number
          }
        ]
      }
    ]
  }
Response 404: { error: { code: "NOT_FOUND", message: "Travel Window not found" } }
Response 400: travel window has no shortlisted trips
  { error: { code: "EMPTY_WINDOW", message: "This Travel Window has no shortlisted trips" } }
Side effects: none
  -- budget_snapshots lists every budget on the trip, primary first. The individual
     trip view shows the primary and indicates when others exist (US-040)
  -- blocks array is the itinerary summary (location and travel) with highlights —
     day detail fetched separately
  -- Deleted-record entries populated from the trip snapshot; shape is identical
```

---

### Full Trip Detail

```
GET /api/presentation/:window_id/trips/:id
Channel: presentation:trip:get
  -- :id is travel_window_trips.id
  -- Full drill-down detail for a single trip within presentation mode
Response 200:
  {
    -- All fields from GET /api/presentation/:window_id trips array entry, plus:
    blocks: [
      {
        -- every <summary block> field, plus:
        notes:         string | null,  -- TipTap JSON
        days: [                        -- empty for travel blocks and blocks without days.
                                       -- Every day that exists is included, whether or not
                                       -- the block is open in the trip builder
          {
            id:         string,
            day_number: number,
            date:       string | null,
            notes:      string | null,  -- TipTap JSON
            pois: [
              {
                id:              string,
                poi_id:          string,
                poi_name:        string,
                position:        number,
                notes:           string | null,  -- TipTap JSON (day-specific override)
                description:     string | null,  -- TipTap JSON (from POI record)
                practical_notes: string | null,  -- TipTap JSON (from POI record)
                tags:            [{ id: string, category: string, value: string }],
                media: [
                  {
                    id:            string,
                    app_url:       string,
                    thumbnail_url: string,
                    is_hero:       boolean
                  }
                ]
              }
            ]
          }
        ]
      }
    ],
    budgets: [
      {
        id:              string,
        name:            string,
        is_primary:      boolean,
        notes:           string | null,  -- TipTap JSON
        snapshot: {
          total_budgeted:   number | null,
          total_actual:     number | null,
          total_difference: number | null,
          actuals_complete: boolean,
          cost_per_day:     number | null,
          variance_status:  string | null
        },
        categories: [
          {
            id:       string,
            name:     string,
            position: number,
            subtotal: {
              budgeted:        number | null,
              actual:          number | null,
              difference:      number | null,
              actuals_complete: boolean,
              variance_status: string | null
            },
            line_items: [
              {
                id:              string,
                name:            string,
                position:        number,
                budgeted:        number | null,
                actual:          number | null,
                difference:      number | null,
                variance_status: string | null,
                notes:           string | null
              }
            ]
          }
        ]
      }
    ],
    tips: [
      {
        id:           string,
        type:         string,
        scope:        string,
        title:        string,
        content:      string | null,   -- TipTap JSON
        country_name: string | null,
        region_name:  string | null,
        city_name:    string | null
      }
    ],
    media: [
      {
        id:            string,
        app_url:       string,
        thumbnail_url: string,
        is_hero:       boolean,
        position:      number,
        source:        string   -- trip | country | region | city | poi
      }
    ]
  }
Response 404: { error: { code: "NOT_FOUND", message: "Travel Window trip not found" } }
Side effects: none
  -- tips scoped to countries referenced in the trip — general tips excluded
  -- Deleted-record entries sourced from the trip snapshot; shape identical to live trips

Trip snapshot contents (trip_snapshots.data):
  -- Everything in this response except the per-window fields, which are read from
     the travel_window_trips row: id, trip_id (null), position, is_chosen,
     is_deleted_record (true), deleted_at
  -- Fully denormalised and frozen at deletion: names, rich text, budget totals and
     variance statuses, and tips are stored as values, never recomputed. Variance
     statuses therefore keep the thresholds that were set on the day of deletion
  -- All ids inside are historical. The renderer must not link from a preserved
     record to library records, which may have changed or been deleted
  -- media entries carry the app_url / thumbnail_url of snapshot-owned media rows,
     so images survive deletion of the trip, its POIs and its countries
```

---

### Comparison View

```
GET /api/presentation/:window_id/comparison
Channel: presentation:comparison:get
Response 200:
  {
    window_id: string,
    trips: [
      {
        id:                string,
        trip_id:           string | null,
        position:          number,
        is_chosen:         boolean,
        is_deleted_record: boolean,
        name:              string,
        status:            string | null,
        completed_at:      string | null,
        notes:             string | null,  -- TipTap JSON: the trip concept ("The idea" row,
                                           -- FR10; handoff 10). The renderer clamps it to 2 lines
        countries:         [{ id: string, name: string }],
        length:            { days: number, complete: boolean },
        blocks:            [ <summary block> ],  -- without highlights; drives the compact
                                                 -- route strip and route text
        tags: [{ id: string, category: string, value: string }],
        budget_count:    number,   -- all budgets on the trip; > 1 = more detail available
        budget_snapshot: {
          name:             string | null,
          total_budgeted:   number | null,
          total_actual:     number | null,
          cost_per_day:     number | null,
          variance_status:  string | null
        } | null,
        hero_image: {
          app_url:       string,
          thumbnail_url: string,
          source:        string    -- hero rule in Key behaviours
        } | null
      }
    ]
  }
Response 404: { error: { code: "NOT_FOUND", message: "Travel Window not found" } }
Response 400: fewer than 2 shortlisted trips
  {
    error: {
      code: "INSUFFICIENT_TRIPS",
      message: "Comparison view requires at least 2 shortlisted trips",
      current_count: number
    }
  }
Side effects: none
  -- Tags row hidden by renderer if no trips have any tags applied
  -- budget_snapshot is the trip's primary budget; null if the trip has no budget
```

---

## Module 9 — Backup

**IPC prefix:** `backup`

**Key behaviours:**

- Design is in `architecture.md` (Backup and Recovery Pattern). Local snapshots hold the
  database only; the backup folder holds the database and all media
- Local snapshots are taken automatically (before migrations, and daily). These
  endpoints add manual control, the backup folder, and restore
- The backup folder path and backup times are kept in the `backup` section of
  `userData/settings.json` (through Module 10), not in the database
- A restore replaces the current library and relaunches the app
- **Launch state.** Before the main window loads, the main process decides which of
  three screens the renderer opens: the normal app, **first run** (no library file
  existed, so an empty one was created and nothing has been chosen yet) or
  **recovery** (the startup check or a migration failed). The renderer reads this
  once with `backup:launch:get`

---

### Launch State

```
GET /api/backup/launch
Channel: backup:launch:get
Response 200:
  {
    mode: string,     -- normal | first_run | recovery
    recovery: {
      reason:       string,         -- integrity_failed | migration_failed
      detail:       string,         -- the check's or migration's error text ("Show details")
      damaged_file: string | null,  -- integrity_failed: the kept copy's file name,
                                    -- e.g. "wanderly-damaged-2026-10-09.db"
      app_version:  string          -- migration_failed: the version that could not migrate
    } | null          -- present only when mode = recovery
  }
Side effects: none
  -- integrity_failed: the damaged database was moved aside (not deleted) to
     userData/wanderly-damaged-<date>.db before this screen opens, so a restore never
     overwrites it (handoff 21). If that name exists, a counter is added (-2, -3, …)
  -- migration_failed: the migration was rolled back and the pre-migration snapshot
     restored, so the library is intact but on the old schema. This version of the app
     cannot open it; the screen can only show the error and quit (see open item F in
     architecture.md)
```

```
POST /api/backup/first-run/start-fresh
Channel: backup:first-run:start-fresh
  -- "Start fresh" on the first-run screen (D-35): keeps the empty library.
Request body:  none
Response 200:  { mode: "normal" }
Response 400:  launch mode is not first_run
  { error: { code: "VALIDATION_ERROR", message: "Not in first run" } }
Side effects:  first-run flag cleared in settings.json; the library stays empty
  -- No sample content is created (D-35)
```

### Status

```
GET /api/backup/status
Channel: backup:status:get
Response 200:
  {
    local: {
      last_snapshot_at: string | null,
      snapshot_count:   number
    },
    folder: {
      path:            string | null,   -- null = no backup folder configured
      available:       boolean,         -- false if configured but not reachable right now
      last_backup_at:  string | null,   -- last successful run
      last_error:      string | null,   -- message from the most recent failed run
      is_stale:        boolean          -- true if no successful run in the last 7 days
    }
  }
Side effects: none
```

---

### Backup Folder

```
POST /api/backup/folder
Channel: backup:folder:set
  -- Opens the native folder dialog from the main process and saves the chosen folder.
Request body:  none
Response 200:  { path: string | null }   -- null if the user cancelled; setting unchanged
Response 400:  folder is inside the app data directory, or is not writable
  { error: { code: "VALIDATION_ERROR", message: "Choose a writable folder outside the app's data directory" } }
Side effects:  path saved to settings.json; a "Wanderly Backup" subfolder is created
```

```
POST /api/backup/folder/pick
Channel: backup:folder:pick
  -- Opens the native folder dialog to choose a folder to restore from (first run,
  -- recovery, "Restore from another folder…"). Nothing is saved by this call.
Request body:  none
Response 200:
  {
    picked_path:   string | null,   -- null if the user cancelled
    backup_path:   string | null,   -- the "Wanderly Backup" folder that was found:
                                    -- the picked folder itself, or a child of it
                                    -- named "Wanderly Backup" (one level down, so
                                    -- picking the drive works); null if neither
    summary: {
      created_at:     string,       -- newest database copy in the folder
      includes_media: true,
      counts:         { countries: number, pois: number, trips: number },
      size_bytes:     number,       -- database copy + media mirror
      restorable:     boolean       -- false if made by a newer app version
    } | null                        -- null when backup_path is null
  }
Side effects: none
```

```
DELETE /api/backup/folder
Channel: backup:folder:clear
Response 200:  { path: null }
Side effects:  setting cleared; files already in the folder are left untouched
```

---

### Run a Backup

```
POST /api/backup/run
Channel: backup:backup:run
Request body:
  {
    target: string  required  -- local | folder
  }
Response 200:
  {
    target:        string,
    completed_at:  string,
    database_bytes: number,
    media_files_copied: number   -- always 0 for target = local
  }
Response 409: target = folder but none is configured or it cannot be reached
  { error: { code: "BACKUP_FOLDER_UNAVAILABLE", message: "The backup folder is not available" } }
Side effects:
  -- local:  database snapshot written to userData/backups/; old snapshots pruned
  -- folder: database copy written; new media files mirrored; manifest.json updated;
             database copies beyond the newest 3 removed, then media no retained
             copy references
  -- The folder run also happens automatically when the app quits, if anything changed
```

---

### Restore

```
GET /api/backup/restore-points
Channel: backup:restore-point:list
Query params:
  folder_path  string?   -- inspect this folder instead of the configured one
                         -- (first run on a new machine); obtain via the folder dialog
Response 200:
  {
    restore_points: [
      {
        id:             string,
        source:         string,          -- local | folder
        kind:           string,          -- daily | pre-migration | pre-restore | manual | folder
        created_at:     string,
        schema_version: string,
        includes_media: boolean,         -- true only for source = folder
        restorable:     boolean,         -- false if made by a newer app version
        unrestorable_reason: string | null,  -- tooltip text when restorable = false
        counts: { countries: number, pois: number, trips: number },
          -- written beside each copy when it is made (a small JSON sidecar for local
             snapshots, manifest.json for the folder), so listing never opens a database.
             A copy without one is opened read-only once and counted
        size_bytes:     number
      }
    ]
  }
Side effects: none
  -- folder_path may be the picked folder or its "Wanderly Backup" child; the same
     one-level-down lookup as POST /api/backup/folder/pick applies
  -- Newest first
```

```
POST /api/backup/restore
Channel: backup:restore:start
Request body:
  {
    restore_point_id: string   required
    folder_path:      string?  -- when restoring from a folder that is not the configured one
  }
Response 409: confirm required — always
  {
    error: {
      code: "RESTORE_REQUIRES_CONFIRMATION",
      message: "Restoring will replace your current library",
      restore_point: { id: string, created_at: string, includes_media: boolean },
      current_library_last_modified: string
    }
  }
Response 200: (with confirm: true)  { restarting: true }
Response 404: { error: { code: "NOT_FOUND", message: "Restore point not found" } }
Response 422: integrity check failed, or made by a newer app version
  { error: { code: "INVALID_BACKUP", message: "This backup cannot be restored", reason: string } }
Side effects (with confirm: true):
  -- Current database snapshotted to userData/backups/ as a pre-restore snapshot
     (skipped in first run, where the library is empty, and in recovery, where the
     damaged file has already been kept aside)
  -- Folder restore point: media copied into the media directory first, then the
     database. The live library is replaced only after every file has copied, so a
     failure part-way leaves the current library untouched
  -- First run: the folder restored from becomes the configured backup folder
     (handoff 21), and the preferences saved in its manifest.json (palette, theme,
     thresholds, keymap) are applied, so a new machine looks and behaves like the old one
  -- App relaunches; pending migrations run; search index rebuilt
  -- The confirm step is skipped in first run and recovery — there is no library to
     lose; the renderer calls once with confirm: true
Emits: backup:restore:progress (while copying; see below)
```

```
Event: backup:restore:progress            (main → renderer; handoff 31)
Payload:
  {
    phase:       string,   -- validating | copying_media | copying_database | finishing
    bytes_done:  number,
    bytes_total: number,   -- known before copying starts, so the bar is determinate
    files_done:  number,
    files_total: number
  }
  -- Sent at most every 100 ms and once at the end of each phase. bytes_total covers
     the media mirror plus the database copy. A local snapshot restore copies the
     database only, so it reports one short copying_database phase
  -- Cancelling: there is no cancel channel in v1, matching the design (open question F).
     Until the database swap at the end nothing has been replaced, so a cancel could be
     added safely by deleting the partly-copied files
```

---

## Module 10 — Settings & UI State

**IPC prefix:** `settings`

**Key behaviours:**

- One file, `userData/settings.json`, owned by this module (UI State Store in
  `architecture.md`). Nothing here is in the database, so restoring a library never
  changes the user's appearance, thresholds or layout
- Three sections: `preferences` (chosen in Settings), `ui` (remembered as the user
  works — no Settings control) and `backup` (written by Module 9 through this module,
  read-only to the renderer)
- Every value has a default. A missing, unknown or invalid value is replaced by its
  default when the file is read and the problem is logged; a file that cannot be parsed
  is renamed to `settings-unreadable-<date>.json` and defaults are used — a bad settings
  file never stops the app from opening
- Writes are atomic (write a temporary file, then rename) and coalesced: several
  updates within 250 ms are written once. The renderer applies a change immediately and
  does not wait for the write
- Budget variance (Module 3) reads `preferences.budget_thresholds` through this module's
  service interface

```
GET /api/settings
Channel: settings:all:get
Response 200:
  {
    preferences: {
      palette:           string,   -- paper | atlas; default paper (D-10)
      theme:             string,   -- light | dark | system; default system (D-05)
      budget_thresholds: {
        close:    number,          -- fraction, default 0.10 (D-18)
        way_over: number           -- fraction, default 0.25; always > close
      },
      keymap: { [action_id: string]: string }   -- P2 (US-067); {} until then.
                                                  -- Only overrides are stored
    },
    ui: {
      nav_collapsed:      boolean,                          -- D-17
      nav_sections:       { [section: string]: boolean },   -- expanded per nav section
      list_views:         { [list: string]: string },       -- list | grid, per list (D-24, D-34)
      list_sorts:         { [list: string]: string },       -- sort id per list page
      window_views:       { [window_id: string]: string },  -- one_at_a_time | side_by_side (D-30)
      expanded_blocks:    { [trip_id: string]: string[] },  -- open block ids per trip (D-44)
      contents_collapsed: { [place_id: string]: string[] }, -- collapsed ContentsIndex rows
      tutorial_seen:      boolean                           -- P2 (US-066); false until then
    },
    backup: {
      folder_path:     string | null,
      last_backup_at:  string | null,
      first_run:       boolean
    }
  }
Side effects: none
```

```
PATCH /api/settings
Channel: settings:all:update
Request body:
  {
    preferences: { ...any subset of preferences }?,
    ui:          { ...any subset of ui }?
  }
  -- Merged one level deep: { ui: { list_views: { trips: "grid" } } } changes only that
     key. A map value of null removes that key (e.g. a deleted trip's expanded_blocks)
  -- backup is not writable through this channel
Response 200: same shape as GET /api/settings
Response 400: a value of the wrong type, an unknown palette/theme/view, a threshold
  outside 0.01–1.00, or way_over ≤ close
  { error: { code: "VALIDATION_ERROR", message: "Way over must be higher than Close (10%)", field: string } }
  -- Nothing in the request is applied when any field is invalid; the previous values
     stay in effect (Settings › Budget "invalid" state)
Side effects:
  -- settings.json rewritten (coalesced, atomic)
  -- A thresholds change takes effect on the next budget read; open budget screens
     re-fetch
```

**Housekeeping.** When a trip, Travel Window or place is deleted, its keys in
`ui.expanded_blocks`, `ui.window_views` and `ui.contents_collapsed` are removed by the
deleting module through this module's service interface. Stale keys are harmless —
they are also dropped on read when the id no longer exists.
