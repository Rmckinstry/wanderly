# Wanderly — Data Model

**Paradigm:** Relational (SQLite via Drizzle ORM / better-sqlite3)
**Database version:** SQLite 3.x (bundled with Electron)
**Migration tool:** Drizzle Kit generates versioned SQL migration files during development; Drizzle ORM's runtime migrator applies them automatically on app startup

---

## Schema

### Core Geographic Hierarchy

```sql
CREATE TABLE countries (
  id          TEXT    NOT NULL PRIMARY KEY,  -- cuid2
  name        TEXT    NOT NULL UNIQUE,
  status      TEXT    NOT NULL DEFAULT 'Wishlist'
    CHECK (status IN ('Wishlist', 'Researching', 'Sampling', 'Planning')),
    -- No Completed state — countries have no completion lifecycle
  notes       TEXT,                          -- TipTap JSON
  created_at  TEXT    NOT NULL,
  updated_at  TEXT    NOT NULL
);

CREATE TABLE regions (
  id          TEXT    NOT NULL PRIMARY KEY,
  country_id  TEXT    NOT NULL REFERENCES countries(id) ON DELETE CASCADE,
  name        TEXT    NOT NULL,
  notes       TEXT,                          -- TipTap JSON
  created_at  TEXT    NOT NULL,
  updated_at  TEXT    NOT NULL,
  UNIQUE (country_id, name)
  -- No status — depth implied by content volume per requirements
);

CREATE TABLE cities (
  id          TEXT    NOT NULL PRIMARY KEY,
  country_id  TEXT    NOT NULL REFERENCES countries(id) ON DELETE CASCADE,
  region_id   TEXT    REFERENCES regions(id) ON DELETE CASCADE,
    -- nullable: city can attach directly to country
  name        TEXT    NOT NULL,
  notes       TEXT,                          -- TipTap JSON
  created_at  TEXT    NOT NULL,
  updated_at  TEXT    NOT NULL
  -- No status — depth implied by content volume per requirements
  -- Name is unique within the same parent (US-003) — see the two partial unique
  -- indexes on cities under Indexes. A plain UNIQUE cannot do it because
  -- region_id is nullable.
);

CREATE TABLE pois (
  id              TEXT    NOT NULL PRIMARY KEY,
  name            TEXT    NOT NULL,
  country_id      TEXT    NOT NULL REFERENCES countries(id) ON DELETE CASCADE,
    -- country_id is always required — it is the top-level geographic anchor
    -- region_id and city_id optionally narrow the attachment within that country
  region_id       TEXT    REFERENCES regions(id) ON DELETE CASCADE,
  city_id         TEXT    REFERENCES cities(id) ON DELETE CASCADE,
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
  single_select INTEGER NOT NULL DEFAULT 0,
    -- 1 = a record can carry at most one value from this tag's category.
    -- Same value on every row of a (content_type, category). In the default tag
    -- set only POI 'Must-Do Status' is single-select (US-057).
    -- Application layer enforces it: applying a second value replaces the first.
  UNIQUE (content_type, category, value)
);

CREATE TABLE content_tags (
  id          TEXT    NOT NULL PRIMARY KEY,
  tag_id      TEXT    NOT NULL REFERENCES tag_definitions(id) ON DELETE CASCADE,
  -- Polymorphic target: exactly one of the following FK columns is set
  -- (enforced by the CHECK below).
  country_id  TEXT    REFERENCES countries(id) ON DELETE CASCADE,
  region_id   TEXT    REFERENCES regions(id) ON DELETE CASCADE,
  city_id     TEXT    REFERENCES cities(id) ON DELETE CASCADE,
  poi_id      TEXT    REFERENCES pois(id) ON DELETE CASCADE,
  trip_id     TEXT    REFERENCES trips(id) ON DELETE CASCADE,
  created_at  TEXT    NOT NULL,
  CHECK ((country_id IS NOT NULL) + (region_id IS NOT NULL) + (city_id IS NOT NULL)
       + (poi_id IS NOT NULL) + (trip_id IS NOT NULL) = 1),
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
  status        TEXT    NOT NULL DEFAULT 'Sample'
    CHECK (status IN ('Sample', 'Planning', 'Ready', 'Completed')),
  completed_at  TEXT,
    -- ISO date. Defaults to today when status is first set to Completed (if currently null).
    -- Editable only while the trip is Completed — any valid date, including past
    -- dates (for historical trips) and future dates (US-016b).
    -- Retained (not cleared, not editable) when status moves back from Completed —
    -- restored automatically if trip is marked Completed again.
    -- Drives the Completed badge display in Travel Windows (US-035).
  notes         TEXT,                        -- TipTap JSON (trip concept / description)
  created_at    TEXT    NOT NULL,
  updated_at    TEXT    NOT NULL
);

-- Many-to-many: which countries a trip references.
-- Used to scope in-context search to relevant countries (US-026).
-- Always a superset of the countries its blocks use: a block adds its country's
-- link, and a link cannot be removed while a block uses that country. Links with
-- no block behind them are allowed (a trip can name a country before it has blocks).
CREATE TABLE trip_countries (
  trip_id     TEXT    NOT NULL REFERENCES trips(id) ON DELETE CASCADE,
  country_id  TEXT    NOT NULL REFERENCES countries(id) ON DELETE CASCADE,
    -- A country can only be deleted when no trip_blocks row uses it (see below),
    -- so this cascade only ever removes links that have no block behind them.
  PRIMARY KEY (trip_id, country_id)
);

-- Location blocks: high-level itinerary mode (US-012)
CREATE TABLE trip_blocks (
  id            TEXT    NOT NULL PRIMARY KEY,
  trip_id       TEXT    NOT NULL REFERENCES trips(id) ON DELETE CASCADE,
  position      INTEGER NOT NULL,            -- manual ordering within the trip
  country_id    TEXT    NOT NULL REFERENCES countries(id) ON DELETE RESTRICT,
    -- country_id always required — enables efficient trip-country scoping for search.
    -- RESTRICT: a country in use by any block cannot be deleted. The application
    -- checks first and returns RECORD_IN_USE; the constraint is the backstop.
    -- region_id and city_id optionally narrow the block's geographic focus.
    -- SET NULL: deleting the region or city widens the block to its parent level
    -- (city → region or country, region → country). Days and duration are kept.
  region_id     TEXT    REFERENCES regions(id) ON DELETE SET NULL,
  city_id       TEXT    REFERENCES cities(id) ON DELETE SET NULL,
  duration_days INTEGER,                     -- nullable until explicitly set
  notes         TEXT,                        -- TipTap JSON
  created_at    TEXT    NOT NULL,
  updated_at    TEXT    NOT NULL
);

-- Individual days expanded from a block (US-013)
CREATE TABLE trip_days (
  id          TEXT    NOT NULL PRIMARY KEY,
  block_id    TEXT    NOT NULL REFERENCES trip_blocks(id) ON DELETE CASCADE,
  trip_id     TEXT    NOT NULL REFERENCES trips(id) ON DELETE CASCADE,
    -- trip_id is intentionally denormalized here for query convenience.
    -- Fetching all days for a trip is a common operation; avoiding the join
    -- through trip_blocks on every call is worth the redundancy.
    -- Application layer keeps trip_id consistent with block_id.trip_id.
  day_number  INTEGER NOT NULL,              -- 1-based, scoped to block
    -- Always contiguous 1..n within a block, and n = the block's duration_days.
    -- Inserting, deleting, reordering or moving a day renumbers its siblings in
    -- the same transaction (renumber via a temporary offset to satisfy the
    -- UNIQUE constraint mid-update).
  date        TEXT,                          -- optional ISO date string
  notes       TEXT,                          -- TipTap JSON
  created_at  TEXT    NOT NULL,
  updated_at  TEXT    NOT NULL,
  UNIQUE (block_id, day_number)
);

-- POIs attached to a specific day (US-014)
CREATE TABLE trip_day_pois (
  id          TEXT    NOT NULL PRIMARY KEY,
  day_id      TEXT    NOT NULL REFERENCES trip_days(id) ON DELETE CASCADE,
  poi_id      TEXT    NOT NULL REFERENCES pois(id) ON DELETE CASCADE,
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
  trip_id         TEXT    REFERENCES trips(id) ON DELETE CASCADE,
    -- Required for non-template budgets. Null only when is_template = 1.
    -- Budgets attach to trips only — country and region attachment removed (FR1 update).
    -- UNIQUE (trip_id, name) — budget names must be unique within the same trip.
    -- Template names (trip_id = null) are covered by a partial unique index
    -- instead (see Indexes), because UNIQUE treats NULLs as distinct.
  notes           TEXT,                      -- TipTap JSON (US-007b)
  is_primary      INTEGER NOT NULL DEFAULT 0,
    -- 1 = the budget shown for this trip in presentation mode and comparison view.
    -- A trip with any budgets has exactly one primary: the first budget created
    -- becomes primary; the user can move the flag; deleting the primary promotes
    -- the oldest remaining budget. Always 0 for templates.
  num_travelers   INTEGER,
    -- Per-budget (not per-trip) because multiple budget scenarios on the same trip
    -- may model different traveler counts. Used for cost-per-person calculation.
  close_threshold REAL    NOT NULL DEFAULT 0.10,
    -- User-configurable variance threshold for the Close indicator. Default 10%.
  is_template     INTEGER NOT NULL DEFAULT 0,
    -- 1 = saved template (trip_id = null). 0 = live budget (trip_id required).
  created_at      TEXT    NOT NULL,
  updated_at      TEXT    NOT NULL,
  CHECK ((is_template = 1) = (trip_id IS NULL)),   -- templates have no trip; live budgets must
  CHECK (is_template = 0 OR is_primary = 0),       -- a template is never primary
  UNIQUE (trip_id, name)
);

CREATE TABLE budget_categories (
  id          TEXT    NOT NULL PRIMARY KEY,
  budget_id   TEXT    NOT NULL REFERENCES budgets(id) ON DELETE CASCADE,
  name        TEXT    NOT NULL,
  position    INTEGER NOT NULL,
  created_at  TEXT    NOT NULL,
  updated_at  TEXT    NOT NULL,
  UNIQUE (budget_id, name)
);

CREATE TABLE budget_line_items (
  id          TEXT    NOT NULL PRIMARY KEY,
  category_id TEXT    NOT NULL REFERENCES budget_categories(id) ON DELETE CASCADE,
  name        TEXT    NOT NULL,
  position    INTEGER NOT NULL,
  budgeted    INTEGER,                       -- whole US cents; nullable until entered
  actual      INTEGER,                       -- whole US cents; nullable until entered
  -- difference (actual - budgeted) is always computed at the application layer.
  -- Under/Over/Close indicator is also computed, never stored.
  notes       TEXT,
    -- Plain text. A short per-line annotation, not a rich text field —
    -- the budget's own notes column is the rich text field for a budget.
  created_at  TEXT    NOT NULL,
  updated_at  TEXT    NOT NULL,
  CHECK (budgeted IS NULL OR budgeted >= 0),
  CHECK (actual   IS NULL OR actual   >= 0)
);
```

---

### Tips & Lessons Learned

```sql
CREATE TABLE tips (
  id              TEXT    NOT NULL PRIMARY KEY,
  type            TEXT    NOT NULL CHECK (type  IN ('Tip', 'LessonLearned')),
  scope           TEXT    NOT NULL CHECK (scope IN ('General', 'Destination')),
  title           TEXT    NOT NULL,
  content         TEXT,                      -- TipTap JSON
  -- Destination FK columns (all null when scope = General).
  -- When scope = Destination, at most one of the three is set:
  --   exactly one set  = Linked
  --   none set         = Unlinked (the destination was deleted; US-010, US-051)
  -- Unlinked is a derived state, not a stored flag. A tip can only become
  -- Unlinked through a destination delete — never through create or edit.
  -- The CHECKs at the end of the table enforce "at most one" and "none when
  -- General"; the application layer enforces "exactly one on create".
  country_id      TEXT    REFERENCES countries(id) ON DELETE SET NULL,
  region_id       TEXT    REFERENCES regions(id) ON DELETE SET NULL,
  city_id         TEXT    REFERENCES cities(id) ON DELETE SET NULL,
  former_destination TEXT,
    -- Display path of the deleted destination, e.g. "Thailand > Chiang Mai".
    -- Written in the same transaction as the destination delete so the user can
    -- see what an Unlinked tip used to belong to. Cleared when the tip is
    -- re-linked or its scope changes to General.
  -- Source trip link (LessonLearned only, optional).
  -- Application layer enforces: source_trip_id may only reference a trip
  -- with status = Completed at the moment the link is made. The link is kept
  -- if that trip later moves back from Completed (as completed_at is).
  source_trip_id  TEXT    REFERENCES trips(id) ON DELETE SET NULL,
  former_source_trip TEXT,
    -- Name of the deleted source trip. Non-null = "trip link removed" flag (US-052).
    -- Written in the same transaction as the trip delete. Cleared when a new
    -- source trip is linked, the type changes to Tip, or the user dismisses it.
  -- Follow-up link (LessonLearned only, optional; US-055).
  -- Set on a Lesson Learned that was created as a follow-up to a Tip. The original
  -- Tip is never modified. One Tip can have several follow-ups.
  origin_tip_id   TEXT    REFERENCES tips(id) ON DELETE SET NULL,
  created_at      TEXT    NOT NULL,
  updated_at      TEXT    NOT NULL,
  CHECK ((country_id IS NOT NULL) + (region_id IS NOT NULL) + (city_id IS NOT NULL) <= 1),
  CHECK (scope = 'Destination'
         OR (country_id IS NULL AND region_id IS NULL AND city_id IS NULL
             AND former_destination IS NULL)),
  CHECK (type = 'LessonLearned' OR (source_trip_id IS NULL AND origin_tip_id IS NULL))
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
    -- Not unique: snapshot-owned rows share a file with the library row they were
    -- copied from. Purge removes a file only when no media row references it.
  original_name TEXT    NOT NULL,            -- original filename at time of import
  mime_type     TEXT    NOT NULL,
  size_bytes    INTEGER NOT NULL,
  -- Polymorphic attachment: exactly one FK is set (enforced by the CHECK below).
  country_id    TEXT    REFERENCES countries(id) ON DELETE CASCADE,
  region_id     TEXT    REFERENCES regions(id) ON DELETE CASCADE,
  city_id       TEXT    REFERENCES cities(id) ON DELETE CASCADE,
  poi_id        TEXT    REFERENCES pois(id) ON DELETE CASCADE,
  trip_id       TEXT    REFERENCES trips(id) ON DELETE CASCADE,
  snapshot_id   TEXT    REFERENCES trip_snapshots(id) ON DELETE CASCADE,
    -- Set on rows owned by a preserved deleted trip (see trip_snapshots).
    -- These rows keep the image files alive for as long as the snapshot exists.
    -- They are read-only and are excluded from the hero and position rules.
  is_hero       INTEGER NOT NULL DEFAULT 0,
    -- 1 = designated hero image for the parent record.
    -- At most one per parent, enforced by one partial unique index per parent
    -- column (see Indexes). Changing the hero therefore demotes the old one
    -- before promoting the new one, inside one transaction.
  position      INTEGER NOT NULL DEFAULT 0,  -- display ordering within the record
  created_at    TEXT    NOT NULL,
  CHECK ((country_id IS NOT NULL) + (region_id IS NOT NULL) + (city_id IS NOT NULL)
       + (poi_id IS NOT NULL) + (trip_id IS NOT NULL) + (snapshot_id IS NOT NULL) = 1),
  CHECK (snapshot_id IS NULL OR is_hero = 0)   -- snapshot-owned rows are never a hero
);
```

---

### Travel Windows

```sql
CREATE TABLE travel_windows (
  id            TEXT    NOT NULL PRIMARY KEY,
  name          TEXT    NOT NULL UNIQUE,
  target_date   TEXT,
    -- ISO date (YYYY-MM-DD); nullable. Used for chronological ordering (US-035).
    -- For a period rather than a day, store its first day (2027-11-01 for
    -- "November 2027").
  target_label  TEXT,
    -- Optional free text shown instead of the date, e.g. "November 2027" or
    -- "Thanksgiving week". Never used for sorting.
  is_archived   INTEGER NOT NULL DEFAULT 0,
  created_at    TEXT    NOT NULL,
  updated_at    TEXT    NOT NULL
);

-- Preserved copy of a deleted Completed trip (US-034b).
-- One row per deleted trip, shared by every Travel Window the trip was in.
-- Created only when a Completed trip that is in at least one Travel Window is deleted.
CREATE TABLE trip_snapshots (
  id                TEXT    NOT NULL PRIMARY KEY,
  trip_name         TEXT    NOT NULL,        -- for listings without parsing data
  snapshot_version  INTEGER NOT NULL,        -- version of the data shape; starts at 1
  data              TEXT    NOT NULL,
    -- JSON blob. Shape = the Presentation full-detail trip payload
    -- (GET /api/presentation/:window_id/trips/:id) without the per-window fields.
    -- Fully denormalised: names, rich text, budget figures and tips are embedded
    -- as they were at deletion. IDs inside are historical and never dereferenced.
    -- Media entries point at the snapshot-owned media rows (media.snapshot_id).
  deleted_at        TEXT    NOT NULL         -- timestamp of master trip deletion
);

CREATE TABLE travel_window_trips (
  id                TEXT    NOT NULL PRIMARY KEY,
  window_id         TEXT    NOT NULL REFERENCES travel_windows(id) ON DELETE CASCADE,
  -- Exactly one of trip_id / snapshot_id is set:
  --   trip_id set      = live trip
  --   snapshot_id set  = preserved historical record (deleted Completed trip)
  trip_id           TEXT    REFERENCES trips(id) ON DELETE CASCADE,
    -- CASCADE removes the row when a non-Completed trip is deleted.
    -- For a Completed trip the application swaps trip_id for snapshot_id
    -- before deleting the trip, so the row survives.
  snapshot_id       TEXT    REFERENCES trip_snapshots(id) ON DELETE RESTRICT,
    -- The application deletes a snapshot only after the last row that
    -- references it is gone.
  position          INTEGER NOT NULL,
  is_chosen         INTEGER NOT NULL DEFAULT 0,
  created_at        TEXT    NOT NULL,
  CHECK ((trip_id IS NULL) <> (snapshot_id IS NULL)),
  UNIQUE (window_id, trip_id),
  UNIQUE (window_id, snapshot_id)
    -- SQLite treats NULLs as distinct, so each constraint only applies to the
    -- rows where its column is set.
);
```

---

### Full-Text Search

```sql
-- One row per searchable record. Maps a record to its row in the FTS table.
-- A regular table, so lookups and deletes by record are indexed.
CREATE TABLE search_docs (
  id            INTEGER PRIMARY KEY,         -- = search_index.rowid
  content_type  TEXT    NOT NULL,
    -- country | region | city | poi | trip | tip | budget
  record_id     TEXT    NOT NULL,            -- ID of the source record (no FK: polymorphic)
  UNIQUE (content_type, record_id)
);

-- FTS5 virtual table — holds only the searchable text. No metadata columns:
-- every column in an FTS5 table is tokenized unless marked UNINDEXED, so type
-- and ID live in search_docs instead and are joined by rowid.
CREATE VIRTUAL TABLE search_index USING fts5(
  title,          -- the record's own name/title
  body,           -- plain text of the record's own searchable fields (see table below)
  tokenize = 'porter unicode61'
);
```

**What is indexed, per content type:**

| Content type | `title` | `body`                                                             |
| ------------ | ------- | ------------------------------------------------------------------ |
| country      | name    | notes                                                              |
| region       | name    | notes                                                              |
| city         | name    | notes                                                              |
| poi          | name    | description + practical_notes                                      |
| trip         | name    | notes                                                              |
| tip          | title   | content                                                            |
| budget       | name    | notes + all category names + all line item names (templates not indexed) |

Rich text fields are reduced to plain text by `extractPlainText(tiptapJson)` at write time.

**What is deliberately not in the index:**

- **Parent names and paths.** A result's parent context ("Thailand > Northern Thailand >
  Chiang Mai") is built from the live hierarchy when the query runs, so renaming or
  moving a parent never leaves stale text behind. Parent names are therefore not
  searchable text: a search for "Thailand" finds the Thailand record and anything that
  mentions Thailand, not every POI inside it. The country filter and in-context search
  cover "everything under Thailand".
- **Scope, status and tags.** Filters are applied by joining each hit to its live source
  row (see Search Query Pattern in `architecture.md`), not by copying those values into
  the index. Tag assignments, status changes and re-parenting need no index update.

**Sync operations** (Search Module; always inside the caller's transaction):

```sql
-- upsert(content_type, record_id, title, body)
INSERT INTO search_docs (content_type, record_id) VALUES (?, ?)
  ON CONFLICT (content_type, record_id) DO NOTHING;
-- doc_id := SELECT id FROM search_docs WHERE content_type = ? AND record_id = ?
DELETE FROM search_index WHERE rowid = :doc_id;
INSERT INTO search_index (rowid, title, body) VALUES (:doc_id, ?, ?);

-- remove(content_type, record_id)
DELETE FROM search_index WHERE rowid = :doc_id;
DELETE FROM search_docs  WHERE id = :doc_id;
```

Both are primary-key operations — no scan of the FTS table.

---

## Indexes

```sql
-- Geographic hierarchy traversal
CREATE INDEX idx_regions_country      ON regions (country_id);
CREATE INDEX idx_cities_country       ON cities (country_id);
CREATE INDEX idx_cities_region        ON cities (region_id);

-- City names unique within the same parent (US-003)
CREATE UNIQUE INDEX idx_cities_name_in_region  ON cities (region_id, name)  WHERE region_id IS NOT NULL;
CREATE UNIQUE INDEX idx_cities_name_in_country ON cities (country_id, name) WHERE region_id IS NULL;
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
CREATE UNIQUE INDEX idx_budgets_primary ON budgets (trip_id) WHERE is_primary = 1;
  -- at most one primary budget per trip
CREATE UNIQUE INDEX idx_budget_template_name ON budgets (name) WHERE trip_id IS NULL;
  -- template names are unique among templates
CREATE INDEX idx_budget_cats_budget   ON budget_categories (budget_id, position);
CREATE INDEX idx_budget_items_cat     ON budget_line_items (category_id, position);

-- Tags
CREATE INDEX idx_content_tags_tag     ON content_tags (tag_id);
CREATE INDEX idx_content_tags_trip    ON content_tags (trip_id);
CREATE INDEX idx_content_tags_poi     ON content_tags (poi_id);
CREATE INDEX idx_content_tags_city    ON content_tags (city_id);
CREATE INDEX idx_content_tags_country ON content_tags (country_id);
CREATE INDEX idx_content_tags_region  ON content_tags (region_id);

-- Tips
CREATE INDEX idx_tips_country         ON tips (country_id);
CREATE INDEX idx_tips_region          ON tips (region_id);
CREATE INDEX idx_tips_city            ON tips (city_id);
CREATE INDEX idx_tips_source_trip     ON tips (source_trip_id);
CREATE INDEX idx_tips_origin          ON tips (origin_tip_id);

-- Media
CREATE INDEX idx_media_country        ON media (country_id);
CREATE INDEX idx_media_region         ON media (region_id);
CREATE INDEX idx_media_city           ON media (city_id);
CREATE INDEX idx_media_poi            ON media (poi_id);
CREATE INDEX idx_media_trip           ON media (trip_id);
CREATE INDEX idx_media_snapshot       ON media (snapshot_id);
CREATE INDEX idx_media_filename       ON media (filename);  -- purge reference check

-- At most one hero image per parent record
CREATE UNIQUE INDEX idx_media_hero_country ON media (country_id) WHERE is_hero = 1;
CREATE UNIQUE INDEX idx_media_hero_region  ON media (region_id)  WHERE is_hero = 1;
CREATE UNIQUE INDEX idx_media_hero_city    ON media (city_id)    WHERE is_hero = 1;
CREATE UNIQUE INDEX idx_media_hero_poi     ON media (poi_id)     WHERE is_hero = 1;
CREATE UNIQUE INDEX idx_media_hero_trip    ON media (trip_id)    WHERE is_hero = 1;

-- Travel Windows
CREATE INDEX idx_travel_windows_date   ON travel_windows (target_date);
CREATE INDEX idx_tw_trips_window      ON travel_window_trips (window_id, position);
CREATE INDEX idx_tw_trips_trip        ON travel_window_trips (trip_id);
CREATE INDEX idx_tw_trips_snapshot    ON travel_window_trips (snapshot_id);

-- Trip references to library records (in-use check on country delete,
-- widening on region/city delete, itinerary warning on POI delete)
CREATE INDEX idx_trip_blocks_country  ON trip_blocks (country_id);
CREATE INDEX idx_trip_blocks_region   ON trip_blocks (region_id);
CREATE INDEX idx_trip_blocks_city     ON trip_blocks (city_id);
CREATE INDEX idx_trip_day_pois_poi    ON trip_day_pois (poi_id);

-- Status filtering (master library views)
CREATE INDEX idx_countries_status     ON countries (status);
CREATE INDEX idx_trips_status         ON trips (status);
CREATE INDEX idx_trips_completed_at   ON trips (completed_at);
```

---

## Integrity Layer

**Connection settings:** every connection sets `journal_mode = WAL`, `synchronous = NORMAL`
and `foreign_keys = ON`. The app uses a single connection in the main process.

**FK enforcement:** `PRAGMA foreign_keys = ON` is set on every connection. Every FK
column declares its `ON DELETE` action in the schema, so the database itself guarantees
that no delete can leave a dangling reference. The application layer adds what the
database cannot do on its own: warn and ask for confirmation, refuse a delete with a
useful message, record what a tip used to be linked to, capture a snapshot, and remove
`search_index` rows (FTS5 tables do not take part in FK cascades).

### Reference matrix

Every FK column and what happens to the row when the referenced record is deleted.

| Referenced record | FK column                         | ON DELETE | Effect                                                                 |
| ----------------- | --------------------------------- | --------- | ---------------------------------------------------------------------- |
| Country           | `regions.country_id`              | CASCADE   | Region deleted                                                         |
| Country           | `cities.country_id`               | CASCADE   | City deleted                                                           |
| Country           | `pois.country_id`                 | CASCADE   | POI deleted                                                            |
| Country           | `trip_blocks.country_id`          | RESTRICT  | Delete refused while any block uses the country                        |
| Country           | `trip_countries.country_id`       | CASCADE   | Trip–country link removed (only links with no block can exist here)    |
| Country           | `tips.country_id`                 | SET NULL  | Tip becomes Unlinked                                                   |
| Country           | `content_tags.country_id`         | CASCADE   | Tag assignment removed                                                 |
| Country           | `media.country_id`                | CASCADE   | Media row deleted; file removed by purge once unreferenced             |
| Region            | `cities.region_id`                | CASCADE   | City deleted                                                           |
| Region            | `pois.region_id`                  | CASCADE   | POI deleted                                                            |
| Region            | `trip_blocks.region_id`           | SET NULL  | Block widens to country level                                          |
| Region            | `tips.region_id`                  | SET NULL  | Tip becomes Unlinked                                                   |
| Region            | `content_tags.region_id`          | CASCADE   | Tag assignment removed                                                 |
| Region            | `media.region_id`                 | CASCADE   | Media row deleted                                                      |
| City              | `pois.city_id`                    | CASCADE   | POI deleted                                                            |
| City              | `trip_blocks.city_id`             | SET NULL  | Block widens to its region, or to the country if it has no region      |
| City              | `tips.city_id`                    | SET NULL  | Tip becomes Unlinked                                                   |
| City              | `content_tags.city_id`            | CASCADE   | Tag assignment removed                                                 |
| City              | `media.city_id`                   | CASCADE   | Media row deleted                                                      |
| POI               | `trip_day_pois.poi_id`            | CASCADE   | POI removed from every trip day that used it                           |
| POI               | `content_tags.poi_id`             | CASCADE   | Tag assignment removed                                                 |
| POI               | `media.poi_id`                    | CASCADE   | Media row deleted                                                      |
| Trip              | `trip_countries.trip_id`          | CASCADE   | Link removed                                                           |
| Trip              | `trip_blocks.trip_id`             | CASCADE   | Block deleted                                                          |
| Trip              | `trip_days.trip_id`               | CASCADE   | Day deleted                                                            |
| Trip              | `budgets.trip_id`                 | CASCADE   | Budget deleted (templates have no `trip_id` and are unaffected)        |
| Trip              | `tips.source_trip_id`             | SET NULL  | Lesson Learned kept; trip link removed and flagged                     |
| Trip              | `content_tags.trip_id`            | CASCADE   | Tag assignment removed                                                 |
| Trip              | `media.trip_id`                   | CASCADE   | Media row deleted — unless re-parented to a snapshot first (see below) |
| Trip              | `travel_window_trips.trip_id`     | CASCADE   | Row removed — unless swapped to a snapshot first (see below)           |
| Trip block        | `trip_days.block_id`              | CASCADE   | Day deleted                                                            |
| Trip day          | `trip_day_pois.day_id`            | CASCADE   | Day–POI entry deleted                                                  |
| Budget            | `budget_categories.budget_id`     | CASCADE   | Category deleted                                                       |
| Budget category   | `budget_line_items.category_id`   | CASCADE   | Line item deleted                                                      |
| Tag definition    | `content_tags.tag_id`             | CASCADE   | Tag removed from every record carrying it                              |
| Travel Window     | `travel_window_trips.window_id`   | CASCADE   | Row removed                                                            |
| Trip snapshot     | `travel_window_trips.snapshot_id` | RESTRICT  | Snapshot cannot be deleted while a Travel Window still shows it        |
| Trip snapshot     | `media.snapshot_id`               | CASCADE   | Snapshot-owned media rows deleted                                      |
| Tip               | `tips.origin_tip_id`              | SET NULL  | Follow-up Lesson Learned kept; it no longer points at an original Tip  |

Cascades chain. Deleting a country deletes its regions, which deletes their cities,
which deletes their POIs, which removes those POIs from trip days.

### Delete paths

What the application does around each delete. All steps for one delete run in a single
transaction. "Warn" means the delete call returns `DELETE_REQUIRES_CONFIRMATION` with a
preview and nothing is changed until the confirm call.

| Deleted record       | Application behaviour                                                                                                                                                                                                                                                                                                                    |
| -------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Country              | **Refuse** with `RECORD_IN_USE` and the list of trips if any `trip_blocks` row uses the country; confirming does not override this. Otherwise warn with counts of regions, cities, POIs, media, tips that will be unlinked, trip–country links, and trip-day POI entries (with the affected trips). On confirm: stamp tips, then delete. |
| Region               | Warn with counts of cities, POIs, media, tips that will be unlinked, blocks that will widen, and trip-day POI entries (with the affected trips). On confirm: stamp tips, then delete.                                                                                                                                                    |
| City                 | Same as Region, without the city count.                                                                                                                                                                                                                                                                                                  |
| POI                  | Warn if referenced in any `trip_day_pois`, listing the trips. Delete on confirm.                                                                                                                                                                                                                                                         |
| Trip (non-Completed) | Warn if referenced in Travel Windows. On confirm: stamp linked Lessons Learned, then delete. The trip's `travel_window_trips` rows go with it.                                                                                                                                                                                           |
| Trip (Completed)     | Warn if referenced in Travel Windows. On confirm: stamp linked Lessons Learned; if the trip is in at least one Travel Window, capture a snapshot (see below); then delete.                                                                                                                                                               |
| Travel Window        | Warn if the window holds any preserved historical record, because a record that is in no other window is destroyed with it. On confirm: delete the window, then delete every `trip_snapshots` row that no remaining `travel_window_trips` row references.                                                                                |
| Budget template      | No cascade. Budgets already built from this template are unaffected.                                                                                                                                                                                                                                                                     |
| Tag definition (P2)  | Warn: tag will be removed from all records carrying it. Delete on confirm.                                                                                                                                                                                                                                                               |

**Stamp tips** (destination deletes): before the delete, for every tip linked to the
record being deleted or to any region or city beneath it, set `former_destination` to
the destination's display path. The FK action then nulls the link and the tip is
Unlinked. Tips are never deleted by a destination delete.

**Stamp linked Lessons Learned** (trip deletes): before the delete, for every tip with
`source_trip_id` = the trip, set `former_source_trip` to the trip's name. The FK action
then nulls the link.

**Search index:** before the delete, remove the search entries (`search_docs` and
`search_index` rows) for the record and for every indexed record the cascade will remove
(regions, cities, POIs, trips, budgets). Tips are kept, so their entries stay. An entry
that is missed cannot surface in results, because every query joins hits to their live
source rows; the startup consistency check then clears it.

**Capture a snapshot** (Completed trip in at least one Travel Window):

1. Build the Presentation full-detail payload for the trip and insert it as one
   `trip_snapshots` row with the current `snapshot_version`.
2. Give the snapshot its own media rows, so its images survive later library deletes:
   re-parent the trip's own media rows (`trip_id` → NULL, `snapshot_id` set), and insert
   a copy row (same `filename`, `snapshot_id` set) for each POI or country image the
   payload includes. The payload's media entries reference these rows.
3. On each of the trip's `travel_window_trips` rows, set `snapshot_id` and null `trip_id`.
4. Delete the trip.

A Completed trip that is in no Travel Window has nowhere to be preserved and is deleted
without a snapshot.

**Rules enforced by the schema** (CHECK constraints and partial unique indexes, declared
above — the database rejects a violation whatever the application does):

- `countries`, `trips`: `status` is one of the allowed values. `tips`: `type` and `scope` likewise
- `content_tags`, `media`: exactly one parent FK column set per row
- `media`: at most one hero image per parent; a snapshot-owned row is never a hero
- `budgets`: `trip_id` is null exactly when `is_template = 1`; a template is never primary; at most one primary budget per trip; template names unique among templates
- `budget_line_items`: amounts are non-negative
- `cities`: name unique within the same region, or within the country for cities with no region
- `tips`: at most one destination FK; no destination when `scope = General`; `source_trip_id` and `origin_tip_id` only on a `LessonLearned`
- `travel_window_trips`: exactly one of `trip_id` / `snapshot_id`

**Rules enforced by the application layer** (they depend on another row, or on what the
row was before the write, which a CHECK cannot see):

- `pois`: `city_id` and `region_id` must be consistent with `country_id` hierarchy
- `trip_blocks`: `region_id` and `city_id` must be consistent with `country_id` hierarchy
- `content_tags`: the tag's `content_type` matches the kind of record it is applied to; at most one value per record from a `single_select` category
- `media`: snapshot-owned rows are never modified
- `tips`: when `scope = Destination`, create requires exactly one destination FK. An edit may leave none set only if the tip was already Unlinked before the edit
- `tips`: `source_trip_id` may only be set to a trip with `status = Completed`; checked when the link is made, not afterwards
- `tips`: `origin_tip_id` must reference a tip of type `Tip`
- `budgets`: a trip with one or more budgets has at least one primary (the index guarantees at most one)
- `budget_line_items`: `budgeted` and `actual` are whole cents
- `trip_days`: `trip_id` must match `block_id → trip_blocks.trip_id` (denormalization consistency)
- `trip_days`: within an expanded block, `day_number` runs 1..n with no gaps and n equals the block's `duration_days`
- `trip_countries`: contains every country used by one of the trip's blocks
- `trips`: `completed_at` is changed only while `status = Completed`
- `trip_snapshots`: rows are written once, at deletion of a Completed trip, and never updated

---

## Migration Strategy

Drizzle Kit (a development-time CLI) generates versioned SQL migration files under
`src/main/db/migrations/`. They are packaged with the app as resources. On startup, before
the main window opens, Drizzle ORM's runtime migrator (`migrate` from
`drizzle-orm/better-sqlite3/migrator`) applies any that have not yet run.
Before any pending migration runs, the app takes a snapshot of the database with SQLite's
online backup API. The migrations then run in a single transaction: if one fails, the
transaction rolls back, the snapshot is restored, and the app reports the error instead of
opening on a half-migrated database. Migrations are additive where possible. See Backup and
Recovery Pattern in `architecture.md`.

---

## Design Decisions

**IDs as TEXT (cuid2), not INTEGER.** Avoids rowid reuse after deletion. Safe for the IPC
layer without sequential guessability concerns.

**TipTap content stored as TEXT (JSON).** ProseMirror JSON is portable and readable.
Not directly indexed in FTS5 — the Search Module extracts plain text at write time and
populates the `body` column in `search_index`.

**`difference` and variance indicators computed, not stored.** `actual - budgeted` is always
derivable. Storing it introduces sync risk with no query benefit.

**Money as integer cents.** All amounts are USD (FR8) and are stored as whole cents in
`INTEGER` columns. Sums and differences are then exact; floating-point `REAL` values would
accumulate rounding error across line items. Only the derived per-person and per-day
figures involve division, and they are rounded to the nearest cent when computed.

**`trip_days.trip_id` is intentional denormalization.** Fetching all days for a trip is
frequent enough to justify avoiding a join through `trip_blocks` on every call.

**Snapshot as a JSON blob in `trip_snapshots`.** A frozen copy of a deleted Completed
trip's full presentation detail. Read-only by definition — no relational query flexibility
needed, so a normalized snapshot schema would add complexity with no benefit. It lives in
its own table rather than on `travel_window_trips` because a trip can be in several
Travel Windows: one snapshot is shared by all of them, and it gives the snapshot's media
rows a single owner.

**Snapshots are versioned, not migrated.** `snapshot_version` records the shape the blob
was written in. Schema migrations never rewrite stored snapshots. When the presentation
payload shape changes, the version is bumped and the Presentation Module gains an
upgrade step (version N → N+1) that is applied when an older snapshot is read.

**Unlinked tips are a derived state.** A destination-scoped tip with no destination FK
is Unlinked. No flag column is needed to find them, and the FK `SET NULL` action produces
the state by itself. `former_destination` only records what the tip used to belong to.

**Trips are protected from destination deletes.** A country cannot be deleted while a
trip block uses it. Deleting a region or city keeps the block and widens it to the parent
level, so no itinerary loses days because library content was tidied up.

**`completed_at` on `trips` only.** Countries, regions, and cities have no completion
lifecycle. `completed_at` is absent from those tables entirely.

**FTS5 write-through sync.** Every write that changes a record's indexed text updates
the index synchronously in the same transaction. Search is always current — no
background jobs, no eventual consistency, no stale results.

**The index holds only a record's own text.** Type and ID sit in `search_docs`; parent
context, scope, status and tags are read from the live tables at query time. The one
exception is a budget, whose body includes its category and line item names — so every
category and line item write re-indexes the parent budget. Keeping derived data out of
the index is what makes "always current" true: there is nothing in it that another
record's edit can invalidate.

**`search_docs` instead of `UNINDEXED` columns.** Marking `content_type` and `record_id`
as `UNINDEXED` in the FTS table would stop them being matched as text, but deleting or
replacing a record's entry would still scan the whole FTS table on every write. A
regular mapping table gives an indexed lookup and a rowid delete.

**P2 — Full data export (NFR5).** JSON export of the full library preserving content
relationships is planned as a P2 enhancement. When built, export will be handled in the
main process by serialising all tables in hierarchy order (countries → regions → cities →
POIs → trips → budgets → tips → media metadata) to a single JSON file. Media files
themselves will require a separate archive step. No architecture changes are required
to support this — the schema and module boundaries are already compatible.
