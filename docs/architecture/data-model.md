# Travel App — Data Model

**Paradigm:** Relational (SQLite via Drizzle ORM / better-sqlite3)
**Database version:** SQLite 3.x (bundled with Electron)
**Migration tool:** Drizzle Kit — versioned SQL migration files, run automatically on app startup

---

## Schema

### Core Geographic Hierarchy

```sql
CREATE TABLE countries (
  id          TEXT    NOT NULL PRIMARY KEY,  -- cuid2
  name        TEXT    NOT NULL UNIQUE,
  status      TEXT    NOT NULL DEFAULT 'Wishlist',
    -- CHECK: Wishlist | Researching | Sampling | Planning
    -- No Completed state — countries have no completion lifecycle
  notes       TEXT,                          -- TipTap JSON
  created_at  TEXT    NOT NULL,
  updated_at  TEXT    NOT NULL
);

CREATE TABLE regions (
  id          TEXT    NOT NULL PRIMARY KEY,
  country_id  TEXT    NOT NULL REFERENCES countries(id),
  name        TEXT    NOT NULL,
  notes       TEXT,                          -- TipTap JSON
  created_at  TEXT    NOT NULL,
  updated_at  TEXT    NOT NULL,
  UNIQUE (country_id, name)
  -- No status — depth implied by content volume per requirements
);

CREATE TABLE cities (
  id          TEXT    NOT NULL PRIMARY KEY,
  country_id  TEXT    NOT NULL REFERENCES countries(id),
  region_id   TEXT    REFERENCES regions(id),  -- nullable: city can attach directly to country
  name        TEXT    NOT NULL,
  notes       TEXT,                          -- TipTap JSON
  created_at  TEXT    NOT NULL,
  updated_at  TEXT    NOT NULL
  -- No status — depth implied by content volume per requirements
);

CREATE TABLE pois (
  id              TEXT    NOT NULL PRIMARY KEY,
  name            TEXT    NOT NULL,
  country_id      TEXT    NOT NULL REFERENCES countries(id),
    -- country_id is always required — it is the top-level geographic anchor
    -- region_id and city_id optionally narrow the attachment within that country
  region_id       TEXT    REFERENCES regions(id),
  city_id         TEXT    REFERENCES cities(id),
    -- Application layer enforces: if city_id is set, region_id must match that
    -- city's region (or be null). country_id always required regardless.
  description     TEXT,                      -- TipTap JSON
  practical_notes TEXT,                      -- TipTap JSON
  created_at      TEXT    NOT NULL,
  updated_at      TEXT    NOT NULL
);
```

---

### Tags

```sql
CREATE TABLE tag_definitions (
  id            TEXT    NOT NULL PRIMARY KEY,
  content_type  TEXT    NOT NULL,
    -- country | region | city | poi | trip
  category      TEXT    NOT NULL,
    -- e.g. 'Region/Continent', 'POI Type', 'Vibe'
  value         TEXT    NOT NULL,
    -- e.g. 'Southeast Asia', 'Restaurant', 'Relaxing'
  is_default    INTEGER NOT NULL DEFAULT 1,  -- 1 = system default, 0 = user-added (P2)
  UNIQUE (content_type, category, value)
);

CREATE TABLE content_tags (
  id          TEXT    NOT NULL PRIMARY KEY,
  tag_id      TEXT    NOT NULL REFERENCES tag_definitions(id),
  -- Polymorphic target: exactly one of the following FK columns is set.
  -- Application layer enforces single-FK rule — SQLite cannot express this constraint.
  country_id  TEXT    REFERENCES countries(id),
  region_id   TEXT    REFERENCES regions(id),
  city_id     TEXT    REFERENCES cities(id),
  poi_id      TEXT    REFERENCES pois(id),
  trip_id     TEXT    REFERENCES trips(id),
  created_at  TEXT    NOT NULL,
  UNIQUE (tag_id, country_id),
  UNIQUE (tag_id, region_id),
  UNIQUE (tag_id, city_id),
  UNIQUE (tag_id, poi_id),
  UNIQUE (tag_id, trip_id)
);
```

---

### Trips

```sql
CREATE TABLE trips (
  id            TEXT    NOT NULL PRIMARY KEY,
  name          TEXT    NOT NULL,
  status        TEXT    NOT NULL DEFAULT 'Sample',
    -- CHECK: Sample | Planning | Ready | Completed
  completed_at  TEXT,
    -- Defaults to today when status is first set to Completed (if currently null).
    -- Independently editable at any time regardless of status — any date is valid
    -- including past dates (for historical trips) and future dates (US-016b).
    -- Retained (not cleared) when status moves back from Completed — restored
    -- automatically if trip is marked Completed again.
    -- Drives the Completed badge display in Travel Windows (US-035).
  notes         TEXT,                        -- TipTap JSON (trip concept / description)
  created_at    TEXT    NOT NULL,
  updated_at    TEXT    NOT NULL
);

-- Many-to-many: which countries a trip references.
-- Used to scope in-context search to relevant countries (US-026).
CREATE TABLE trip_countries (
  trip_id     TEXT    NOT NULL REFERENCES trips(id),
  country_id  TEXT    NOT NULL REFERENCES countries(id),
  PRIMARY KEY (trip_id, country_id)
);

-- Location blocks: high-level itinerary mode (US-012)
CREATE TABLE trip_blocks (
  id            TEXT    NOT NULL PRIMARY KEY,
  trip_id       TEXT    NOT NULL REFERENCES trips(id),
  position      INTEGER NOT NULL,            -- manual ordering within the trip
  country_id    TEXT    NOT NULL REFERENCES countries(id),
    -- country_id always required — enables efficient trip-country scoping for search.
    -- region_id and city_id optionally narrow the block's geographic focus.
  region_id     TEXT    REFERENCES regions(id),
  city_id       TEXT    REFERENCES cities(id),
  duration_days INTEGER,                     -- nullable until explicitly set
  notes         TEXT,                        -- TipTap JSON
  created_at    TEXT    NOT NULL,
  updated_at    TEXT    NOT NULL
);

-- Individual days expanded from a block (US-013)
CREATE TABLE trip_days (
  id          TEXT    NOT NULL PRIMARY KEY,
  block_id    TEXT    NOT NULL REFERENCES trip_blocks(id),
  trip_id     TEXT    NOT NULL REFERENCES trips(id),
    -- trip_id is intentionally denormalized here for query convenience.
    -- Fetching all days for a trip is a common operation; avoiding the join
    -- through trip_blocks on every call is worth the redundancy.
    -- Application layer keeps trip_id consistent with block_id.trip_id.
  day_number  INTEGER NOT NULL,              -- 1-based, scoped to block
  date        TEXT,                          -- optional ISO date string
  notes       TEXT,                          -- TipTap JSON
  created_at  TEXT    NOT NULL,
  updated_at  TEXT    NOT NULL,
  UNIQUE (block_id, day_number)
);

-- POIs attached to a specific day (US-014)
CREATE TABLE trip_day_pois (
  id          TEXT    NOT NULL PRIMARY KEY,
  day_id      TEXT    NOT NULL REFERENCES trip_days(id),
  poi_id      TEXT    NOT NULL REFERENCES pois(id),
  position    INTEGER NOT NULL,              -- ordering within the day
  notes       TEXT,                          -- TipTap JSON (day-specific override notes for this POI)
  created_at  TEXT    NOT NULL
);
```

---

### Budgets

```sql
CREATE TABLE budgets (
  id              TEXT    NOT NULL PRIMARY KEY,
  name            TEXT    NOT NULL,
  trip_id         TEXT    REFERENCES trips(id),
    -- Required for non-template budgets. Null only when is_template = 1.
    -- Budgets attach to trips only — country and region attachment removed (FR1 update).
    -- Application layer enforces: is_template = 0 requires trip_id set.
    -- UNIQUE (trip_id, name) — budget names must be unique within the same trip.
    -- Template names (trip_id = null) are enforced separately at app layer
    -- because SQLite treats NULLs as distinct in UNIQUE constraints.
  num_travelers   INTEGER,
    -- Per-budget (not per-trip) because multiple budget scenarios on the same trip
    -- may model different traveler counts. Used for cost-per-person calculation.
  close_threshold REAL    NOT NULL DEFAULT 0.10,
    -- User-configurable variance threshold for the Close indicator. Default 10%.
  is_template     INTEGER NOT NULL DEFAULT 0,
    -- 1 = saved template (trip_id = null). 0 = live budget (trip_id required).
  created_at      TEXT    NOT NULL,
  updated_at      TEXT    NOT NULL,
  UNIQUE (trip_id, name)
);

CREATE TABLE budget_categories (
  id          TEXT    NOT NULL PRIMARY KEY,
  budget_id   TEXT    NOT NULL REFERENCES budgets(id),
  name        TEXT    NOT NULL,
  position    INTEGER NOT NULL,
  created_at  TEXT    NOT NULL,
  updated_at  TEXT    NOT NULL,
  UNIQUE (budget_id, name)
);

CREATE TABLE budget_line_items (
  id          TEXT    NOT NULL PRIMARY KEY,
  category_id TEXT    NOT NULL REFERENCES budget_categories(id),
  name        TEXT    NOT NULL,
  position    INTEGER NOT NULL,
  budgeted    REAL,                          -- nullable until entered
  actual      REAL,                          -- nullable until entered
  -- difference (actual - budgeted) is always computed at the application layer.
  -- Under/Over/Close indicator is also computed, never stored.
  notes       TEXT,
  created_at  TEXT    NOT NULL,
  updated_at  TEXT    NOT NULL
);
```

---

### Tips & Lessons Learned

```sql
CREATE TABLE tips (
  id              TEXT    NOT NULL PRIMARY KEY,
  type            TEXT    NOT NULL,          -- Tip | LessonLearned
  scope           TEXT    NOT NULL,          -- General | Destination
  title           TEXT    NOT NULL,
  content         TEXT,                      -- TipTap JSON
  -- Destination FK columns (all null when scope = General).
  -- When scope = Destination, exactly one of the three must be set.
  -- Application layer enforces both rules.
  country_id      TEXT    REFERENCES countries(id),
  region_id       TEXT    REFERENCES regions(id),
  city_id         TEXT    REFERENCES cities(id),
  -- Source trip link (LessonLearned only, optional).
  -- Application layer enforces: source_trip_id may only reference a trip
  -- with status = Completed. Linking to an active trip is not permitted.
  source_trip_id  TEXT    REFERENCES trips(id),
  created_at      TEXT    NOT NULL,
  updated_at      TEXT    NOT NULL
);
```

---

### Media

```sql
CREATE TABLE media (
  id            TEXT    NOT NULL PRIMARY KEY,
  filename      TEXT    NOT NULL,
    -- Stored filename within the app data directory.
    -- File is copied into the app data dir on import (Sharp handles copy + thumbnail).
    -- Deleting a media record does not auto-delete the file — explicit purge required.
  original_name TEXT    NOT NULL,            -- original filename at time of import
  mime_type     TEXT    NOT NULL,
  size_bytes    INTEGER NOT NULL,
  -- Polymorphic attachment: exactly one FK is set.
  -- Application layer enforces single-FK rule.
  country_id    TEXT    REFERENCES countries(id),
  region_id     TEXT    REFERENCES regions(id),
  city_id       TEXT    REFERENCES cities(id),
  poi_id        TEXT    REFERENCES pois(id),
  trip_id       TEXT    REFERENCES trips(id),
  is_hero       INTEGER NOT NULL DEFAULT 0,
    -- 1 = designated hero image for the parent record.
    -- Application layer enforces only one is_hero = 1 per parent record.
    -- SQLite cannot express this constraint across a polymorphic FK column.
  position      INTEGER NOT NULL DEFAULT 0,  -- display ordering within the record
  created_at    TEXT    NOT NULL
);
```

---

### Travel Windows

```sql
CREATE TABLE travel_windows (
  id            TEXT    NOT NULL PRIMARY KEY,
  name          TEXT    NOT NULL UNIQUE,
  target_date   TEXT,                        -- ISO date or freeform string; nullable
  is_archived   INTEGER NOT NULL DEFAULT 0,
  created_at    TEXT    NOT NULL,
  updated_at    TEXT    NOT NULL
);

CREATE TABLE travel_window_trips (
  id                TEXT    NOT NULL PRIMARY KEY,
  window_id         TEXT    NOT NULL REFERENCES travel_windows(id),
  trip_id           TEXT    REFERENCES trips(id),
    -- Nullable: set to NULL when the master trip is deleted.
    -- UNIQUE (window_id, trip_id) allows multiple NULLs in the trip_id column —
    -- SQLite treats each NULL as distinct, so multiple deleted trips in the same
    -- window do not violate the constraint. This is intentional.
  position          INTEGER NOT NULL,
  is_chosen         INTEGER NOT NULL DEFAULT 0,
  -- Deletion preservation (US-034b):
  -- When a Completed trip is deleted from the master library:
  --   1. snapshot_data is populated with the full trip state as a JSON blob.
  --   2. trip_id is set to NULL.
  --   3. is_deleted_record is set to 1.
  --   4. deleted_at is set to the deletion timestamp.
  -- Non-completed trips that are deleted: row is removed from this table entirely.
  snapshot_data     TEXT,                    -- JSON blob; null for live trips
  is_deleted_record INTEGER NOT NULL DEFAULT 0,
  deleted_at        TEXT,                    -- timestamp of master trip deletion
  created_at        TEXT    NOT NULL,
  UNIQUE (window_id, trip_id)
);
```

---

### Full-Text Search

```sql
-- FTS5 virtual table — indexes searchable text across all content types.
-- Kept in sync via write-through updates in the same SQLite transaction as
-- every create, update, or delete. No deferred or background indexing.
CREATE VIRTUAL TABLE search_index USING fts5(
  content_type,   -- country | region | city | poi | trip | tip | budget
  record_id,      -- ID of the source record (not FK-enforced by FTS5)
  parent_context, -- e.g. "Thailand > Northern Thailand > Chiang Mai"
  title,          -- name/title of the record
  body,           -- concatenated plain-text extraction of all searchable fields:
    -- country:  notes (plain text stripped from TipTap JSON)
    -- region:   notes
    -- city:     notes
    -- poi:      description + practical_notes
    -- trip:     notes
    -- tip:      title + content
    -- budget:   name + all category names + all line item names
  tokenize = 'porter unicode61'
);
```

---

## Indexes

```sql
-- Geographic hierarchy traversal
CREATE INDEX idx_regions_country      ON regions (country_id);
CREATE INDEX idx_cities_country       ON cities (country_id);
CREATE INDEX idx_cities_region        ON cities (region_id);
CREATE INDEX idx_pois_country         ON pois (country_id);
CREATE INDEX idx_pois_region          ON pois (region_id);
CREATE INDEX idx_pois_city            ON pois (city_id);

-- Trip structure
CREATE INDEX idx_trip_blocks_trip     ON trip_blocks (trip_id, position);
CREATE INDEX idx_trip_days_block      ON trip_days (block_id, day_number);
CREATE INDEX idx_trip_days_trip       ON trip_days (trip_id);
CREATE INDEX idx_trip_day_pois_day    ON trip_day_pois (day_id, position);
CREATE INDEX idx_trip_countries_trip  ON trip_countries (trip_id);

-- Budget
CREATE INDEX idx_budgets_trip         ON budgets (trip_id);
CREATE INDEX idx_budget_cats_budget   ON budget_categories (budget_id, position);
CREATE INDEX idx_budget_items_cat     ON budget_line_items (category_id, position);

-- Tags
CREATE INDEX idx_content_tags_tag     ON content_tags (tag_id);
CREATE INDEX idx_content_tags_trip    ON content_tags (trip_id);
CREATE INDEX idx_content_tags_poi     ON content_tags (poi_id);
CREATE INDEX idx_content_tags_city    ON content_tags (city_id);
CREATE INDEX idx_content_tags_country ON content_tags (country_id);

-- Tips
CREATE INDEX idx_tips_country         ON tips (country_id);
CREATE INDEX idx_tips_region          ON tips (region_id);
CREATE INDEX idx_tips_city            ON tips (city_id);
CREATE INDEX idx_tips_source_trip     ON tips (source_trip_id);

-- Media
CREATE INDEX idx_media_country        ON media (country_id);
CREATE INDEX idx_media_region         ON media (region_id);
CREATE INDEX idx_media_city           ON media (city_id);
CREATE INDEX idx_media_poi            ON media (poi_id);
CREATE INDEX idx_media_trip           ON media (trip_id);

-- Travel Windows
CREATE INDEX idx_tw_trips_window      ON travel_window_trips (window_id, position);
CREATE INDEX idx_tw_trips_trip        ON travel_window_trips (trip_id);

-- Status filtering (master library views)
CREATE INDEX idx_countries_status     ON countries (status);
CREATE INDEX idx_trips_status         ON trips (status);
CREATE INDEX idx_trips_completed_at   ON trips (completed_at);
```

---

## Integrity Layer

**FK enforcement:** `PRAGMA foreign_keys = ON` is set on every connection. Enforces all
standard FK relationships. Polymorphic FK columns (media, tags, budgets, tips, travel_window_trips)
are enforced at the application layer — SQLite cannot express single-FK-set constraints
across nullable columns.

**Cascade behavior on deletion — handled explicitly by the application layer:**

| Deleted record       | Behavior                                                                                                                                                                        |
| -------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Country              | Warn user: all nested regions, cities, POIs, media, and tips will be affected. Hard cascade on confirm.                                                                         |
| Region               | Warn: nested cities, POIs, and tips affected. Hard cascade on confirm. Tips move to Unlinked state if destination-specific.                                                     |
| City                 | Warn: nested POIs and tips affected. Hard cascade on confirm.                                                                                                                   |
| POI                  | Warn if referenced in any `trip_day_pois`. Remove those references on confirm.                                                                                                  |
| Trip (non-Completed) | Warn if referenced in Travel Windows. Remove `travel_window_trips` rows on confirm.                                                                                             |
| Trip (Completed)     | Warn if referenced in Travel Windows. On confirm: populate `snapshot_data`, set `is_deleted_record = 1`, null `trip_id` on `travel_window_trips` rows. Then delete master trip. |
| Budget template      | No cascade. Budgets already built from this template are unaffected.                                                                                                            |
| Tag definition (P2)  | Warn: tag will be removed from all records carrying it. Remove all `content_tags` rows for this tag on confirm.                                                                 |

**Application-layer rules not expressible in SQLite:**

- `content_tags`: exactly one FK column set per row
- `media`: exactly one FK column set per row; exactly one `is_hero = 1` per parent record
- `budgets`: `trip_id` must be set when `is_template = 0`; `trip_id` must be null when `is_template = 1`
- `pois`: `city_id` and `region_id` must be consistent with `country_id` hierarchy
- `tips`: when `scope = Destination`, exactly one of `country_id`, `region_id`, `city_id` is set; when `scope = General`, all three are null
- `tips`: `source_trip_id` may only reference a trip with `status = Completed`
- `trip_days`: `trip_id` must match `block_id → trip_blocks.trip_id` (denormalization consistency)
- `travel_window_trips`: `snapshot_data` populated only at deletion of a Completed trip

---

## Migration Strategy

Drizzle Kit generates versioned SQL migration files under `src/main/db/migrations/`.
Migrations run automatically on app startup via `db.migrate()` before the main window opens.
All migrations are additive where possible — destructive changes (column drops, table renames)
are gated behind a confirmed data backup step surfaced to the user.

---

## Design Decisions

**IDs as TEXT (cuid2), not INTEGER.** Avoids rowid reuse after deletion. Safe for the IPC
layer without sequential guessability concerns.

**TipTap content stored as TEXT (JSON).** ProseMirror JSON is portable and readable.
Not directly indexed in FTS5 — the Search Module extracts plain text at write time and
populates the `body` field in `search_index`.

**`difference` and variance indicators computed, not stored.** `actual - budgeted` is always
derivable. Storing it introduces sync risk with no query benefit.

**`trip_days.trip_id` is intentional denormalization.** Fetching all days for a trip is
frequent enough to justify avoiding a join through `trip_blocks` on every call.

**`snapshot_data` as JSON blob on `travel_window_trips`.** A frozen copy of a deleted
Completed trip's full state. Read-only by definition — no relational query flexibility
needed, so a normalized snapshot schema would add complexity with no benefit.

**`completed_at` on `trips` only.** Countries, regions, and cities have no completion
lifecycle. `completed_at` is absent from those tables entirely.

**FTS5 write-through sync.** Every content write triggers a synchronous FTS5 index update
in the same transaction. Search is always current — no background jobs, no eventual
consistency, no stale results.

**P2 — Full data export (NFR5).** JSON export of the full library preserving content
relationships is planned as a P2 enhancement. When built, export will be handled in the
main process by serialising all tables in hierarchy order (countries → regions → cities →
POIs → trips → budgets → tips → media metadata) to a single JSON file. Media files
themselves will require a separate archive step. No architecture changes are required
to support this — the schema and module boundaries are already compatible.
