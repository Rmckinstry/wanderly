# Travel App — API Contracts

## Global Conventions

**Transport:** Electron IPC via contextBridge. All contracts expressed in REST style
for readability; each maps to an `ipcMain.handle` channel in the main process.

**IPC channel convention:** `<module>:<resource>:<action>`
e.g. `content:country:create`, `trip:block:expand`, `search:global`

**Authentication:** None — single-user local app. Omitted from all endpoints.

**IDs:** All record IDs are cuid2 strings.

**Timestamps:** All timestamps are ISO 8601 strings.

**Rich text fields:** All `notes`, `description`, `content`, and `practical_notes`
fields store and return TipTap JSON strings unless explicitly noted otherwise.

**Error response shape** (universal across all endpoints):

```
{ error: { code: string, message: string, ...context_fields } }
```

**Error codes used across modules:**

- `VALIDATION_ERROR` — missing or invalid field in request
- `NOT_FOUND` — record does not exist
- `DUPLICATE_NAME` — unique name constraint violated
- `DUPLICATE_LINK` — many-to-many link already exists
- `HIERARCHY_MISMATCH` — geographic FK inconsistency
- `DELETE_REQUIRES_CONFIRMATION` — destructive action needs confirm call
- `TYPE_MISMATCH` — tag applied to wrong content type
- `RECORD_IS_DELETED` — operation not permitted on a preserved historical record

---

## Module 1 — Content (Countries, Regions, Cities, POIs)

**IPC prefix:** `content`

---

### Countries

```
GET /api/countries
Query params:  status  string?  -- Wishlist | Researching | Sampling | Planning
Response 200:
  {
    countries: [
      {
        id:         string,
        name:       string,
        status:     string,
        created_at: string,
        updated_at: string,
        counts: {
          regions: number,
          cities:  number,
          pois:    number,
          trips:   number
        }
      }
    ]
  }
Side effects: none
```

```
GET /api/countries/:id
Response 200:
  {
    id:         string,
    name:       string,
    status:     string,
    notes:      string | null,   -- TipTap JSON
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
Request body:
  {
    name:   string  required  -- non-empty, unique across all countries
    status: string?           -- default: Wishlist
    notes:  string?           -- TipTap JSON
  }
Response 201:
  {
    id:         string,
    name:       string,
    status:     string,
    notes:      string | null,
    created_at: string,
    updated_at: string
  }
Response 400: name is empty
  { error: { code: "VALIDATION_ERROR", message: "Name is required" } }
Response 409: country with this name already exists
  { error: { code: "DUPLICATE_NAME", message: "A country named '{name}' already exists", existing_id: string } }
Side effects: search_index updated synchronously
```

```
PATCH /api/countries/:id
Request body: (all fields optional — only provided fields updated)
  {
    name:   string?
    status: string?  -- Wishlist | Researching | Sampling | Planning
    notes:  string?  -- TipTap JSON
  }
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
Response 409: country has nested content — confirm required
  {
    error: {
      code: "DELETE_REQUIRES_CONFIRMATION",
      message: "This country has nested content that will be deleted",
      cascade_preview: {
        regions: number,
        cities:  number,
        pois:    number,
        tips:    number,
        media:   number
      }
    }
  }
Response 200: (empty country — no confirm needed)
  {
    deleted_id: string,
    cascade_summary: {
      regions_deleted: number,
      cities_deleted:  number,
      pois_deleted:    number,
      tips_unlinked:   number,
      media_deleted:   number
    }
  }
Response 404: { error: { code: "NOT_FOUND", message: "Country not found" } }
Side effects: none until confirmed
```

```
DELETE /api/countries/:id/confirm
Response 200:
  {
    deleted_id: string,
    cascade_summary: {
      regions_deleted: number,
      cities_deleted:  number,
      pois_deleted:    number,
      tips_unlinked:   number,
      media_deleted:   number
    }
  }
Response 404: { error: { code: "NOT_FOUND", message: "Country not found" } }
Side effects:
  -- Cascades to regions, cities, POIs, media (hard delete)
  -- Tips with this country as destination move to unlinked state
  -- search_index entries removed synchronously
```

---

### Regions

```
GET /api/countries/:country_id/regions
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
Response 409: region has nested content — confirm required
  {
    error: {
      code: "DELETE_REQUIRES_CONFIRMATION",
      message: "This region has nested content that will be deleted",
      cascade_preview: { cities: number, pois: number, tips: number, media: number }
    }
  }
Response 200: (empty region — no confirm needed)
  {
    deleted_id: string,
    cascade_summary: { cities_deleted: number, pois_deleted: number, tips_unlinked: number, media_deleted: number }
  }
Response 404: { error: { code: "NOT_FOUND", message: "Region not found" } }
Side effects: none until confirmed
```

```
DELETE /api/regions/:id/confirm
Response 200:
  {
    deleted_id: string,
    cascade_summary: { cities_deleted: number, pois_deleted: number, tips_unlinked: number, media_deleted: number }
  }
Response 404: { error: { code: "NOT_FOUND", message: "Region not found" } }
Side effects:
  -- Cascades to cities, POIs, media (hard delete)
  -- Tips unlinked (destination FKs nulled)
  -- search_index updated synchronously
```

---

### Cities

```
GET /api/cities/:id
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
Side effects: search_index updated synchronously
```

```
PATCH /api/cities/:id
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
Side effects: search_index updated synchronously
```

```
DELETE /api/cities/:id
Response 409: city has nested content — confirm required
  {
    error: {
      code: "DELETE_REQUIRES_CONFIRMATION",
      message: "This city has nested content that will be deleted",
      cascade_preview: { pois: number, tips: number, media: number }
    }
  }
Response 200: (empty city — no confirm needed)
  {
    deleted_id: string,
    cascade_summary: { pois_deleted: number, tips_unlinked: number, media_deleted: number }
  }
Response 404: { error: { code: "NOT_FOUND", message: "City not found" } }
Side effects: none until confirmed
```

```
DELETE /api/cities/:id/confirm
Response 200:
  {
    deleted_id: string,
    cascade_summary: { pois_deleted: number, tips_unlinked: number, media_deleted: number }
  }
Response 404: { error: { code: "NOT_FOUND", message: "City not found" } }
Side effects:
  -- Cascades to POIs, media (hard delete)
  -- Tips unlinked
  -- search_index updated synchronously
```

---

### POIs

```
GET /api/pois/:id
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
    media:            [{ id: string, filename: string, is_hero: boolean, position: number }],
    created_at:       string,
    updated_at:       string
  }
Response 404: { error: { code: "NOT_FOUND", message: "POI not found" } }
Side effects: none
```

```
POST /api/pois
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
Side effects: search_index updated synchronously
```

```
PATCH /api/pois/:id
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
Query params: content_type  string?  -- country | region | city | poi | trip
Response 200:
  {
    tags: [
      {
        id:           string,
        content_type: string,
        category:     string,
        value:        string,
        is_default:   boolean
      }
    ]
  }
Side effects: none
```

```
POST /api/content-tags
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
Response 201: { id: string, tag_id: string, [parent_field]: string, created_at: string }
Response 400: zero or more than one parent FK provided
  { error: { code: "VALIDATION_ERROR", message: "Exactly one parent record must be specified" } }
Response 404: tag or parent record not found
  { error: { code: "NOT_FOUND", message: "Tag definition not found" } }
Response 409: tag already applied to this record
  { error: { code: "DUPLICATE_TAG", message: "This tag is already applied to this record" } }
Response 409: tag content_type does not match parent record type
  { error: { code: "TYPE_MISMATCH", message: "This tag cannot be applied to this content type" } }
Side effects: none
```

```
DELETE /api/content-tags/:id
Response 200: { deleted_id: string }
Response 404: { error: { code: "NOT_FOUND", message: "Tag assignment not found" } }
Side effects: none
```

---

## Module 2 — Trips (Trip CRUD, Blocks, Days, Day-POIs)

**IPC prefix:** `trip`

---

### Trips

```
GET /api/trips
Query params:
  status     string?  -- Sample | Planning | Ready | Completed
  country_id string?  -- filter to trips referencing this country
Response 200:
  {
    trips: [
      {
        id:           string,
        name:         string,
        status:       string,
        completed_at: string | null,
        created_at:   string,
        updated_at:   string,
        countries:    [{ id: string, name: string }],
        counts: { blocks: number, days: number }
      }
    ]
  }
Side effects: none
```

```
GET /api/trips/:id
Response 200:
  {
    id:           string,
    name:         string,
    status:       string,
    completed_at: string | null,
    notes:        string | null,  -- TipTap JSON
    created_at:   string,
    updated_at:   string,
    countries:    [{ id: string, name: string }],
    tags:         [{ id: string, category: string, value: string }],
    media:        [{ id: string, filename: string, is_hero: boolean, position: number }],
    counts: { blocks: number, days: number, budgets: number }
  }
Response 404: { error: { code: "NOT_FOUND", message: "Trip not found" } }
Side effects: none
```

```
POST /api/trips
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
```

```
PATCH /api/trips/:id
Request body:
  {
    name:         string?
    status:       string?   -- Sample | Planning | Ready | Completed
    completed_at: string?   -- ISO date string; editable independently of status.
                            -- Any date is valid (past or future).
                            -- Pass null to explicitly clear.
    notes:        string?   -- TipTap JSON
    country_ids:  string[]? -- replaces full set of linked countries
  }
Response 200: same shape as GET /api/trips/:id
Response 400: name is empty, or status is not a valid value, or completed_at is not a valid date
  { error: { code: "VALIDATION_ERROR", message: "Invalid field value" } }
Response 404: trip not found, or any country_id not found
  { error: { code: "NOT_FOUND", message: "Trip not found" } }
Side effects:
  -- If status changes to Completed AND completed_at is currently null:
     completed_at set to today's date automatically
  -- If status changes to Completed AND completed_at already has a value:
     completed_at left unchanged — prior date restored
  -- If status changes away from Completed: completed_at retained, not cleared
  -- If completed_at explicitly provided in request body: always used as-is
     regardless of status change direction
  -- trip_countries replaced if country_ids provided
  -- search_index updated synchronously
```

```
DELETE /api/trips/:id
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
      windows_snapshot_taken: number,
      windows_refs_removed:   number
    }
  }
Response 404: { error: { code: "NOT_FOUND", message: "Trip not found" } }
Side effects: none until confirmed
```

```
DELETE /api/trips/:id/confirm
Response 200: same shape as DELETE /api/trips/:id success response
Response 404: { error: { code: "NOT_FOUND", message: "Trip not found" } }
Side effects:
  -- Completed trip: snapshot_data populated on travel_window_trips rows,
     trip_id nulled, is_deleted_record = 1, deleted_at set
  -- Non-Completed trip: travel_window_trips rows removed
  -- All blocks, days, day_poi_refs, budgets, media hard deleted
  -- search_index updated synchronously
```

---

### Trip Countries

```
POST /api/trips/:trip_id/countries
Request body:  { country_id: string  required }
Response 201:  { trip_id: string, country_id: string, country_name: string }
Response 404:  { error: { code: "NOT_FOUND", message: "Trip or country not found" } }
Response 409:  { error: { code: "DUPLICATE_LINK", message: "Country is already linked to this trip" } }
Side effects: none
```

```
DELETE /api/trips/:trip_id/countries/:country_id
Response 200:  { trip_id: string, country_id: string }
Response 404:  { error: { code: "NOT_FOUND", message: "Country link not found" } }
Side effects: none
  -- Note: does not remove blocks referencing this country. Application layer
     warns the user if unlinking a country that still has blocks referencing it.
```

---

### Trip Blocks

```
GET /api/trips/:trip_id/blocks
Response 200:
  {
    blocks: [
      {
        id:            string,
        trip_id:       string,
        position:      number,
        country_id:    string,
        country_name:  string,
        region_id:     string | null,
        region_name:   string | null,
        city_id:       string | null,
        city_name:     string | null,
        duration_days: number | null,
        notes:         string | null,  -- TipTap JSON
        days_expanded: boolean,
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
Request body:
  {
    country_id:    string   required
    region_id:     string?
    city_id:       string?
    duration_days: number?  -- positive integer
    notes:         string?  -- TipTap JSON
    position:      number?  -- if omitted, appended to end
  }
Response 201:
  {
    id:            string,
    trip_id:       string,
    position:      number,
    country_id:    string,
    country_name:  string,
    region_id:     string | null,
    region_name:   string | null,
    city_id:       string | null,
    city_name:     string | null,
    duration_days: number | null,
    notes:         string | null,  -- TipTap JSON
    days_expanded: false,
    created_at:    string,
    updated_at:    string
  }
Response 400: country_id missing, or duration_days not a positive integer
  { error: { code: "VALIDATION_ERROR", message: "country_id is required" } }
Response 404: trip, country, region, or city not found
  { error: { code: "NOT_FOUND", message: "Country not found" } }
Response 409: region or city does not belong to specified country
  { error: { code: "HIERARCHY_MISMATCH", message: "Region or city does not belong to the specified country" } }
Side effects:
  -- Sibling block positions incremented if inserted mid-sequence
  -- Country added to trip_countries if not already linked
```

```
PATCH /api/trips/blocks/:id
Request body:
  {
    country_id:    string?
    region_id:     string?  -- pass null to detach
    city_id:       string?  -- pass null to detach
    duration_days: number?  -- pass null to clear; reducing below day count requires confirm
    notes:         string?  -- TipTap JSON
    position:      number?
  }
Response 200: same shape as POST /api/trips/:trip_id/blocks response
Response 400: duration_days not a positive integer
  { error: { code: "VALIDATION_ERROR", message: "duration_days must be a positive integer" } }
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
```

```
PATCH /api/trips/blocks/:id/confirm-duration-reduce
Request body:  { duration_days: number  required }
Response 200:  same shape as PATCH /api/trips/blocks/:id response
Response 404:  { error: { code: "NOT_FOUND", message: "Block not found" } }
Side effects:  Days beyond new duration hard deleted including their day_poi_refs
```

```
DELETE /api/trips/blocks/:id
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
Response 404: { error: { code: "NOT_FOUND", message: "Block not found" } }
Response 409: block already has expanded days
  { error: { code: "ALREADY_EXPANDED", message: "Block already has expanded days", existing_day_count: number } }
Side effects: trip_days rows created; block duration_days updated to match
```

---

### Trip Days

```
GET /api/trips/:trip_id/days
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
Response 200: single day object — same shape as item in GET /api/trips/:trip_id/days
Response 404: { error: { code: "NOT_FOUND", message: "Day not found" } }
Side effects: none
```

```
PATCH /api/trips/days/:id
Request body:
  {
    date:  string?  -- ISO date string; pass null to clear
    notes: string?  -- TipTap JSON
  }
Response 200: same shape as GET /api/trips/days/:id
Response 400: date is not a valid ISO date string
  { error: { code: "VALIDATION_ERROR", message: "date must be a valid ISO date string" } }
Response 404: { error: { code: "NOT_FOUND", message: "Day not found" } }
Side effects: none
```

---

### Day POIs

```
POST /api/trips/days/:day_id/pois
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
Response 200:  { deleted_id: string }
Response 404:  { error: { code: "NOT_FOUND", message: "Day POI not found" } }
Side effects:
  -- trip_day_pois reference deleted; POI master record unaffected
  -- Sibling positions compacted
```

---

## Module 3 — Budget

**IPC prefix:** `budget`

---

### Budgets

```
GET /api/budgets
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
        num_travelers:   number | null,
        close_threshold: number,
        is_template:     boolean,
        created_at:      string,
        updated_at:      string,
        snapshot: {
          total_budgeted:   number | null,
          total_actual:     number | null,
          total_difference: number | null,
          cost_per_person:  number | null,
          cost_per_day:     number | null,
          variance_status:  string | null   -- Under | Over | Close
        }
      }
    ]
  }
Side effects: none
```

```
GET /api/budgets/:id
Response 200:
  {
    id:              string,
    name:            string,
    trip_id:         string | null,
    num_travelers:   number | null,
    close_threshold: number,
    is_template:     boolean,
    created_at:      string,
    updated_at:      string,
    snapshot: {
      total_budgeted:   number | null,
      total_actual:     number | null,
      total_difference: number | null,
      cost_per_person:  number | null,  -- null if num_travelers not set
      cost_per_day:     number | null,  -- null if trip block durations not set
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
            variance_status: string | null,  -- Under | Over | Close | null if no actual
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
  -- Under: actual < budgeted.
  -- Over: actual > budgeted.
  -- Close: |actual - budgeted| / budgeted <= close_threshold.
  -- variance_status null if budgeted is null (no baseline) or actual is null.
```

```
POST /api/budgets
Request body:
  {
    name:            string   required  -- unique within same trip
    trip_id:         string   required  -- unless is_template = true
    num_travelers:   number?            -- positive integer
    close_threshold: number?            -- default: 0.10
    is_template:     boolean?           -- default: false
    template_id:     string?            -- copies category structure from this template
  }
Response 201: same shape as GET /api/budgets/:id (categories empty unless template applied)
Response 400: trip_id missing for non-template budget
  { error: { code: "VALIDATION_ERROR", message: "trip_id is required for non-template budgets" } }
Response 400: trip_id provided for template budget
  { error: { code: "VALIDATION_ERROR", message: "Template budgets cannot be attached to a trip" } }
Response 400: num_travelers not a positive integer
  { error: { code: "VALIDATION_ERROR", message: "num_travelers must be a positive integer" } }
Response 404: trip_id or template_id not found
  { error: { code: "NOT_FOUND", message: "Trip not found" } }
Response 409: budget name already exists on this trip
  { error: { code: "DUPLICATE_NAME", message: "A budget named '{name}' already exists on this trip" } }
Side effects:
  -- If template_id provided: category and line item structure copied; all cost fields null
```

```
PATCH /api/budgets/:id
Request body:
  {
    name:            string?
    num_travelers:   number?  -- pass null to clear
    close_threshold: number?
  }
  -- trip_id and is_template are immutable after creation
Response 200: same shape as GET /api/budgets/:id
Response 400: num_travelers not a positive integer
  { error: { code: "VALIDATION_ERROR", message: "num_travelers must be a positive integer" } }
Response 404: { error: { code: "NOT_FOUND", message: "Budget not found" } }
Response 409: name conflicts with existing budget on same trip
  { error: { code: "DUPLICATE_NAME", message: "A budget named '{name}' already exists on this trip" } }
Side effects: none
```

```
DELETE /api/budgets/:id
Response 200:
  {
    deleted_id: string,
    cascade_summary: { categories_deleted: number, line_items_deleted: number }
  }
Response 404: { error: { code: "NOT_FOUND", message: "Budget not found" } }
Side effects:
  -- All categories and line items hard deleted
  -- Deleting a template does not affect budgets already built from it
```

---

### Apply Template to Existing Budget

```
POST /api/budgets/:id/apply-template
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
Side effects: none on conflict; category structure replaced on clean apply
```

```
POST /api/budgets/:id/apply-template/confirm
Request body:  { template_id: string  required }
Response 200:  same shape as GET /api/budgets/:id
Response 404:  { error: { code: "NOT_FOUND", message: "Template not found" } }
Side effects:
  -- All existing categories and line items deleted
  -- Category and line item structure copied from template; all cost fields null
```

---

### Save Budget as Template

```
POST /api/budgets/:id/save-as-template
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
```

---

### Budget Categories

```
POST /api/budgets/:budget_id/categories
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
Side effects: sibling positions incremented if inserted mid-sequence
```

```
PATCH /api/budgets/categories/:id
Request body:  { name: string?, position: number? }
Response 200:  same shape as POST /api/budgets/:budget_id/categories response
Response 404:  { error: { code: "NOT_FOUND", message: "Category not found" } }
Response 409:  { error: { code: "DUPLICATE_NAME", message: "A category named '{name}' already exists in this budget" } }
Side effects:  sibling positions recompacted if position changed
```

```
DELETE /api/budgets/categories/:id
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
Response 200: { deleted_id: string, cascade_summary: { line_items_deleted: number } }
Response 404: { error: { code: "NOT_FOUND", message: "Category not found" } }
Side effects: all line items hard deleted; sibling positions compacted
```

---

### Budget Line Items

```
POST /api/budgets/categories/:category_id/line-items
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
Side effects: sibling positions incremented if inserted mid-sequence
```

```
PATCH /api/budgets/line-items/:id
Request body:
  {
    name:     string?
    budgeted: number?  -- pass null to clear
    actual:   number?  -- pass null to clear
    notes:    string?
    position: number?
  }
Response 200: same shape as POST /api/budgets/categories/:category_id/line-items response
Response 400: budgeted or actual is negative
  { error: { code: "VALIDATION_ERROR", message: "Cost values cannot be negative" } }
Response 404: { error: { code: "NOT_FOUND", message: "Line item not found" } }
Side effects: sibling positions recompacted if position changed
```

```
DELETE /api/budgets/line-items/:id
Response 200:  { deleted_id: string }
Response 404:  { error: { code: "NOT_FOUND", message: "Line item not found" } }
Side effects:  sibling positions compacted
```

---

## Module 4 — Tips & Lessons Learned

**IPC prefix:** `tips`

---

### Tips

```
GET /api/tips
Query params:
  type:       string?  -- Tip | LessonLearned
  scope:      string?  -- General | Destination
  country_id: string?
  region_id:  string?
  city_id:    string?
  trip_id:    string?  -- filter to tips sourced from this trip (LessonLearned only)
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
        source_trip_id:   string | null,
        source_trip_name: string | null,
        created_at:       string,
        updated_at:       string
      }
    ]
  }
Side effects: none
```

```
GET /api/tips/:id
Response 200: single tip object — same shape as item in GET /api/tips
Response 404: { error: { code: "NOT_FOUND", message: "Tip not found" } }
Side effects: none
```

```
POST /api/tips
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
  }
Response 200: same shape as GET /api/tips/:id
Response 400: title is empty string
  { error: { code: "VALIDATION_ERROR", message: "Title cannot be empty" } }
Response 400: scope changing to General but destination FKs still set
  { error: { code: "VALIDATION_ERROR", message: "Clear destination links before changing scope to General" } }
Response 400: scope changing to Destination but no destination FK provided or remaining
  { error: { code: "VALIDATION_ERROR", message: "A destination must be specified for destination-scoped tips" } }
Response 400: type changing to Tip but source_trip_id is set
  { error: { code: "VALIDATION_ERROR", message: "Remove source_trip_id before changing type to Tip" } }
Response 400: more than one destination FK set after patch applied
  { error: { code: "VALIDATION_ERROR", message: "Only one destination level can be specified" } }
Response 404: tip, destination record, or source_trip_id not found
  { error: { code: "NOT_FOUND", message: "Tip not found" } }
Response 409: source_trip_id references a non-Completed trip
  { error: { code: "INVALID_TRIP_STATUS", message: "Lesson Learned source trip must have Completed status" } }
Side effects:
  -- Server validates resulting state after patch applied — not just fields sent
  -- Client must explicitly clear destination FKs in same request as scope change
  -- search_index updated synchronously
```

```
DELETE /api/tips/:id
Response 200:  { deleted_id: string }
Response 404:  { error: { code: "NOT_FOUND", message: "Tip not found" } }
Side effects:
  -- search_index entry removed synchronously
  -- Note: deleting a destination record does not cascade-delete its tips —
     tips move to unlinked state (destination FKs nulled, scope retained).
     This endpoint is for explicit tip deletion only.
```

---

### Tip Conversion

```
POST /api/tips/:id/convert
Request body:  { source_trip_id: string? }
Response 200:  same shape as GET /api/tips/:id with type = LessonLearned
Response 404:  { error: { code: "NOT_FOUND", message: "Tip not found" } }
Response 409:  tip is already a LessonLearned
  { error: { code: "ALREADY_CONVERTED", message: "This tip is already a Lesson Learned" } }
Response 409:  source_trip_id references a non-Completed trip
  { error: { code: "INVALID_TRIP_STATUS", message: "Source trip must have Completed status" } }
Side effects:
  -- type updated to LessonLearned in place — no new record created
  -- source_trip_id set if provided
  -- search_index updated synchronously
```

---

## Module 5 — Media

**IPC prefix:** `media`

**Key behaviours:**

- Files copied into app data directory on import — original path not stored
- Deleting a media record does not delete the file immediately — purge required
- Sharp runs in Electron main process for all image processing
- Images served to renderer via custom `app://` Electron protocol — never raw file:// paths
- `app://` protocol must be registered in Electron main process via `protocol.registerFileProtocol`

---

### Media Import

```
POST /api/media/import
Request body:
  {
    source_path:  string   required  -- absolute local file path
    mime_type:    string   required  -- image/jpeg | image/png | image/webp | image/gif
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
Response 400: source_path missing or mime_type unsupported
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
  -- File copied to app data dir with cuid2-based filename
  -- Thumbnail generated and written to app data dir /thumbnails/
  -- If is_hero = true: existing is_hero = 1 on this parent demoted to 0
  -- Sibling positions incremented if inserted mid-sequence
```

---

### Media Retrieval

```
GET /api/media
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
Response 200: single media object — same shape as item in GET /api/media
Response 404: { error: { code: "NOT_FOUND", message: "Media record not found" } }
Side effects: none
```

---

### Media Updates

```
PATCH /api/media/:id
Request body:
  {
    is_hero:  boolean?
    position: number?
  }
Response 200: same shape as GET /api/media/:id
Response 400: no fields provided
  { error: { code: "VALIDATION_ERROR", message: "At least one field must be provided" } }
Response 404: { error: { code: "NOT_FOUND", message: "Media record not found" } }
Side effects:
  -- If is_hero = true: existing hero on same parent demoted to is_hero = 0
  -- If is_hero = false and this was the hero: no replacement hero designated automatically
  -- Sibling positions recompacted if position changed
```

---

### Hero Image

```
POST /api/media/:id/set-hero
Request body:  none
Response 200:  same shape as GET /api/media/:id with is_hero: true
Response 404:  { error: { code: "NOT_FOUND", message: "Media record not found" } }
Side effects:
  -- Existing hero on same parent set to is_hero = 0
  -- This record set to is_hero = 1
```

---

### Media Deletion

```
DELETE /api/media/:id
Response 200:
  {
    deleted_id: string,
    filename:   string,   -- queued for purge; not yet deleted from disk
    was_hero:   boolean
  }
Response 404: { error: { code: "NOT_FOUND", message: "Media record not found" } }
Side effects:
  -- media row deleted; file remains on disk until purge
  -- Sibling positions compacted
  -- If was_hero = true: no replacement hero designated automatically
```

---

### Purge Orphaned Files

```
POST /api/media/purge
  -- Removes files in the app data media directory with no corresponding record.
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

- All queries run against FTS5 `search_index` virtual table
- Search is always synchronous — no loading state expected for typical library sizes
- Case-insensitive; porter stemming applied (e.g. "hiking" matches "hike")
- Empty query string returns empty result set — not an error
- Single-character queries permitted

---

### Global Search

```
GET /api/search
Query params:
  q            string   required
  content_type string?            -- country | region | city | poi | trip | tip | budget
  country_id   string?            -- filter to content under this country
  status       string?            -- filter by status (countries and trips only)
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
        parent_context: string,  -- e.g. "Thailand > Northern Thailand > Chiang Mai"
        snippet:        string,  -- FTS5 snippet ~30 words
        rank:           number   -- FTS5 relevance rank; lower = more relevant
      }
    ]
  }
Response 400: q is missing
  { error: { code: "VALIDATION_ERROR", message: "Search query is required" } }
Response 400: content_type is not a valid value
  { error: { code: "VALIDATION_ERROR", message: "Invalid content_type value" } }
Side effects: none
  -- Multiple tag_ids are AND-ed — results must carry all specified tags.
  -- Tag filtering without content_type is applied per content type independently.
```

---

### In-Context Search — Trip Builder

```
GET /api/search/trip/:trip_id
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
```

---

### Recently Used (Trip Builder)

```
GET /api/search/trip/:trip_id/recent
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
Request body:  none
Response 200:
  {
    records_indexed: number,
    duration_ms:     number
  }
Side effects:
  -- search_index dropped and rebuilt from scratch
  -- Synchronous; blocks until complete
  -- Recovery tool only — not needed during normal operation
```

---

## Module 7 — Travel Windows

**IPC prefix:** `travelwindow`

---

### Travel Windows

```
GET /api/travel-windows
Query params:  is_archived  boolean?  -- default: false
Response 200:
  {
    travel_windows: [
      {
        id:          string,
        name:        string,
        target_date: string | null,
        is_archived: boolean,
        created_at:  string,
        updated_at:  string,
        counts: { trips: number, chosen: number, completed: number }
      }
    ]
  }
  -- Ordered by target_date ascending; no target_date sorted to end by created_at
Side effects: none
```

```
GET /api/travel-windows/:id
Response 200:
  {
    id:          string,
    name:        string,
    target_date: string | null,
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
        -- Live trip fields (null when is_deleted_record = true):
        status:            string | null,
        completed_at:      string | null,
        notes:             string | null,  -- TipTap JSON
        countries:         [{ id: string, name: string }] | null,
        tags:              [{ id: string, category: string, value: string }] | null,
        budget_snapshots: [
          {
            id:               string,
            name:             string,
            total_budgeted:   number | null,
            total_actual:     number | null,
            total_difference: number | null,
            cost_per_person:  number | null,
            cost_per_day:     number | null,
            variance_status:  string | null
          }
        ] | null,
        -- Deleted record fields (present when is_deleted_record = true):
        snapshot_data:     object | null
      }
    ]
  }
Response 404: { error: { code: "NOT_FOUND", message: "Travel Window not found" } }
Side effects: none
```

```
POST /api/travel-windows
Request body:
  {
    name:        string  required
    target_date: string?
  }
Response 201:
  {
    id:          string,
    name:        string,
    target_date: string | null,
    is_archived: boolean,
    created_at:  string,
    updated_at:  string,
    trips:       []
  }
Response 400: { error: { code: "VALIDATION_ERROR", message: "Name is required" } }
Response 409: { error: { code: "DUPLICATE_NAME", message: "A Travel Window named '{name}' already exists" } }
Side effects: none
```

```
PATCH /api/travel-windows/:id
Request body:
  {
    name:        string?
    target_date: string?  -- pass null to clear
  }
Response 200: same shape as GET /api/travel-windows/:id
Response 400: { error: { code: "VALIDATION_ERROR", message: "Name cannot be empty" } }
Response 404: { error: { code: "NOT_FOUND", message: "Travel Window not found" } }
Response 409: { error: { code: "DUPLICATE_NAME", message: "A Travel Window named '{name}' already exists" } }
Side effects: none
```

```
DELETE /api/travel-windows/:id
Response 200:
  {
    deleted_id: string,
    cascade_summary: { trip_references_removed: number }
  }
Response 404: { error: { code: "NOT_FOUND", message: "Travel Window not found" } }
Side effects:
  -- travel_window_trips rows hard deleted
  -- snapshot_data permanently lost for any deleted-record rows
  -- Master trip records unaffected
```

---

### Archive / Restore

```
POST /api/travel-windows/:id/archive
Response 200:  { id: string, is_archived: true, updated_at: string }
Response 404:  { error: { code: "NOT_FOUND", message: "Travel Window not found" } }
Response 409:  { error: { code: "ALREADY_ARCHIVED", message: "Travel Window is already archived" } }
Side effects:  is_archived set to 1
```

```
POST /api/travel-windows/:id/restore
Response 200:  { id: string, is_archived: false, updated_at: string }
Response 404:  { error: { code: "NOT_FOUND", message: "Travel Window not found" } }
Response 409:  { error: { code: "NOT_ARCHIVED", message: "Travel Window is not archived" } }
Side effects:  is_archived set to 0
```

---

### Shortlisted Trips

```
POST /api/travel-windows/:window_id/trips
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
  -- Confirms adding 4th+ trip after soft warning acknowledged
Request body:  { trip_id: string  required, position: number? }
Response 201:  same shape as POST /api/travel-windows/:window_id/trips
Response 404:  { error: { code: "NOT_FOUND", message: "Trip not found" } }
Response 409:  { error: { code: "DUPLICATE_TRIP", message: "This trip is already shortlisted" } }
Side effects:  same as POST /api/travel-windows/:window_id/trips
```

```
PATCH /api/travel-windows/trips/:id
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
- Deleted-record entries render from snapshot_data with identical response shape to live trips
- Keyboard navigation state managed entirely client-side

---

### Load Presentation

```
GET /api/presentation/:window_id
  -- Full presentation payload; single call, no further round trips needed
  -- for summary and comparison views.
Response 200:
  {
    window_id:   string,
    window_name: string,
    target_date: string | null,
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
        blocks: [
          {
            id:            string,
            position:      number,
            country_name:  string,
            region_name:   string | null,
            city_name:     string | null,
            duration_days: number | null,
            days_expanded: boolean
          }
        ],
        budget_snapshots: [
          {
            id:               string,
            name:             string,
            num_travelers:    number | null,
            total_budgeted:   number | null,
            total_actual:     number | null,
            total_difference: number | null,
            cost_per_person:  number | null,
            cost_per_day:     number | null,
            variance_status:  string | null
          }
        ],
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
  -- blocks array is location-block summary only — day detail fetched separately
  -- Deleted-record entries populated from snapshot_data; shape is identical
```

---

### Full Trip Detail

```
GET /api/presentation/:window_id/trips/:id
  -- :id is travel_window_trips.id
  -- Full drill-down detail for a single trip within presentation mode
Response 200:
  {
    -- All fields from GET /api/presentation/:window_id trips array entry, plus:
    blocks: [
      {
        id:            string,
        position:      number,
        country_name:  string,
        region_name:   string | null,
        city_name:     string | null,
        duration_days: number | null,
        days_expanded: boolean,
        notes:         string | null,  -- TipTap JSON
        days: [
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
        num_travelers:   number | null,
        close_threshold: number,
        snapshot: {
          total_budgeted:   number | null,
          total_actual:     number | null,
          total_difference: number | null,
          cost_per_person:  number | null,
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
        source:        string   -- trip | country | poi
      }
    ]
  }
Response 404: { error: { code: "NOT_FOUND", message: "Travel Window trip not found" } }
Side effects: none
  -- tips scoped to countries referenced in the trip — general tips excluded
  -- Deleted-record entries sourced from snapshot_data; shape identical to live trips
```

---

### Comparison View

```
GET /api/presentation/:window_id/comparison
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
        countries:         [{ id: string, name: string }],
        blocks: [
          {
            position:      number,
            country_name:  string,
            region_name:   string | null,
            city_name:     string | null,
            duration_days: number | null
          }
        ],
        tags: [{ id: string, category: string, value: string }],
        budget_snapshot: {
          name:             string | null,
          total_budgeted:   number | null,
          total_actual:     number | null,
          cost_per_person:  number | null,
          cost_per_day:     number | null,
          variance_status:  string | null
        } | null,
        hero_image: {
          app_url:       string,
          thumbnail_url: string
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
  -- If trip has multiple budgets, comparison shows first by position only
```
