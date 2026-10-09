# Architecture — Wanderly (Local Desktop)

## 1. Overview

### System Diagram

```
┌─────────────────────────────────────────────────────────────────┐
│  Renderer Process                                               │
│                                                                 │
│  React 19 · Zustand 5 · TipTap 3 · @dnd-kit · Tailwind 4      │
│                                                                 │
│  ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌────────────────────┐ │
│  │ Library  │ │  Trip    │ │ Travel   │ │  Presentation      │ │
│  │  Views   │ │ Builder  │ │ Windows  │ │  Mode              │ │
│  └──────────┘ └──────────┘ └──────────┘ └────────────────────┘ │
│                                                                 │
│  — no direct DB or file system access —                        │
└───────────────────────────┬─────────────────────────────────────┘
                            │
                 contextBridge IPC (typed)
                 ipcRenderer.invoke / ipcMain.handle
                            │
┌───────────────────────────▼─────────────────────────────────────┐
│  Main Process                                                   │
│                                                                 │
│  ┌────────────┐ ┌──────────┐ ┌──────────┐ ┌─────────────────┐ │
│  │  Content   │ │   Trip   │ │  Budget  │ │ Travel Windows  │ │
│  │  Module    │ │  Module  │ │  Module  │ │    Module       │ │
│  └────────────┘ └──────────┘ └──────────┘ └─────────────────┘ │
│                                                                 │
│  ┌────────────┐ ┌──────────┐ ┌──────────┐ ┌─────────────────┐ │
│  │    Tips    │ │  Search  │ │  Media   │ │  Presentation   │ │
│  │   Module   │ │  Module  │ │  Module  │ │    Module       │ │
│  └────────────┘ └──────────┘ └──────────┘ └─────────────────┘ │
│                                                                 │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │  Database Layer — Drizzle ORM · better-sqlite3 · SQLite │   │
│  └─────────────────────────────────────────────────────────┘   │
│                                                                 │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │  Media Layer — Sharp · app:// protocol · app data dir   │   │
│  └─────────────────────────────────────────────────────────┘   │
│                                                                 │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │  Backup — local snapshots · backup folder · restore     │   │
│  └─────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────┘
```

---

### Tech Stack

| Layer                  | Choice                  | Version | Primary Reason                                                                                                                                                    | Alternative Rejected                                                                                                                                  |
| ---------------------- | ----------------------- | ------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------- |
| App shell              | Electron                | 44.x    | Battle-tested for rich local data apps; Node.js backend gives direct SQLite and file system access with no bridge layer                                           | Tauri — Rust backend adds friction for file ops and SQLite with no meaningful benefit for a single-user personal tool where bundle size is irrelevant |
| Frontend framework     | React                   | 19.x    | Industry standard for complex interactive UIs; hooks model handles trip builder state, presentation mode transitions, and drag-and-drop naturally                 | Vue 3 — viable but smaller ecosystem; React better matched to the complexity of nested stateful views                                                 |
| State management       | Zustand                 | 5.x     | Lightweight, boilerplate-free; no multi-user or real-time requirements to justify Redux overhead                                                                  | Redux Toolkit — overkill for this scale; significant boilerplate with no benefit                                                                      |
| Rich text editor       | TipTap                  | 3.x     | ProseMirror-based, React-native, stores content as portable JSON; actively maintained                                                                             | Quill — Delta format less portable; maintenance trajectory uncertain                                                                                  |
| Database               | SQLite (better-sqlite3) | 13.x    | Named in requirements; synchronous API fits Electron's main process model cleanly; Node-API build ships prebuilt binaries that load in Electron without a rebuild | PostgreSQL — requires a running server process, wrong for a desktop app                                                                               |
| ORM                    | Drizzle ORM             | 0.45.x  | Type-safe SQL, lightweight, no generated client or generate step, works with better-sqlite3 natively; its runtime migrator applies migrations on startup          | Prisma — generated client and `prisma generate` step add build and packaging work; heavier abstraction (ADR-002)                                      |
| Migrations             | Drizzle Kit             | 0.31.x  | Pairs with Drizzle ORM; development-time CLI that generates versioned SQL migration files from the schema                                                         | Manual SQL — error-prone across app versions; no rollback story                                                                                       |
| Bundler                | Vite                    | 7.x     | Fast HMR; the newest major that electron-vite supports                                                                                                            | Webpack — slower build times, more config overhead                                                                                                    |
| Electron build tooling | electron-vite           | 5.x     | Handles the main / preload / renderer builds and dev server with minimal config                                                                                   | Hand-rolled Vite configs per process — more wiring to maintain for no benefit                                                                         |
| Styling                | Tailwind CSS            | 4.x     | Utility-first; no context switching between CSS files and components; right scale for solo development                                                            | CSS Modules — more verbose for this project size                                                                                                      |
| Drag and drop          | @dnd-kit/core           | 6.x     | Accessible, headless, React-native; handles itinerary block and POI reordering                                                                                    | react-beautiful-dnd — effectively deprecated                                                                                                          |
| Image processing       | Sharp                   | 0.35.x  | Runs in Electron main process; handles copy-on-import, thumbnail generation, and file type detection; Node-API prebuilt binaries                                  | Canvas API — cannot write to file system from renderer                                                                                                |
| Packaging              | electron-builder        | 26.x    | Handles macOS and Windows packaging with code signing hooks; rebuilds or unpacks native modules as part of packaging                                              | electron-forge — viable but electron-builder has broader format support                                                                               |

**Version source of truth:** All downstream documents reference versions from this table.
Downstream files must never restate version numbers independently.

**Versions validated:** 2026-10-08, against the npm registry and the Electron release
schedule. This is a greenfield project, so each row is the current stable line unless a
note below says otherwise.

**Version notes:**

- **Electron** supports only its latest three majors and ships a new major about every
  eight weeks, so any pinned major is unsupported roughly six months after release.
  Scaffold on the newest stable major (44 today; 45 is due 2026-10-20) and plan a major
  upgrade about twice a year. Update this table with each upgrade. Electron 44 runs
  Node.js 24 and no longer supports macOS 12 or 32-bit Windows.
- **Vite** is held one major behind its latest (8.x) because electron-vite 5.x supports
  Vite 5–7 only. Move to Vite 8 when a stable electron-vite release supports it.
- **Drizzle ORM / Drizzle Kit** stay on the 0.x line. 1.0 is at release-candidate stage
  and changes the migration folder layout; adopt it as a deliberate upgrade after it is
  stable, following its upgrade guide.
- **TipTap 3** keeps the ProseMirror JSON document format, so stored content is unaffected
  by the editor's major version.
- **@dnd-kit/core** is the stable package. Its successor (`@dnd-kit/react`) is still
  pre-1.0 and is not used.

---

### Non-Negotiable Constraints

| Constraint                                                              | Failure Mode if Violated                                                          |
| ----------------------------------------------------------------------- | --------------------------------------------------------------------------------- |
| Local storage only — SQLite on the local file system, no cloud database | Data leaves the user's machine; privacy violated; app no longer functions offline |
| Single-user — no authentication layer                                   | N/A for this version; multi-user deferred explicitly                              |
| Desktop app only (macOS and Windows) — no mobile layout required        | UI breaks on mobile; deferred scope activated prematurely                         |
| No internet connection required to function                             | App unusable in offline environments; defeats the purpose of a local tool         |
| Electron main process owns all DB and file system access                | Renderer gets direct Node access (nodeIntegration: true); security anti-pattern   |
| Images copied into app data directory on import                         | Source file deletion breaks app image references                                  |

---

## 2. Modules

### Module Map

| Module             | Responsibility                                                                                                                                                                                                                                                                                              |
| ------------------ | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Content**        | CRUD for the geographic hierarchy: Country, Region, City, POI, and their tag assignments. Owns hierarchy consistency, country status management, and the warn-or-refuse checks before destination deletes. Largest module.                                                                                  |
| **Tips**           | CRUD for Tips and Lessons Learned: scope and destination linking, source-trip linking, conversion, and the Unlinked state. Separate from Content because tips have their own lifecycle and survive destination deletes.                                                                                     |
| **Trip**           | Trip lifecycle: location-block management, day-by-day expansion, POI attachment to days, status transitions, and `completed_at` tracking. Distinct from Content because trip planning logic — block ordering, duration calculation, mixed-state management — is complex enough to warrant its own boundary. |
| **Budget**         | Budget CRUD, category and line item management, template save/apply, variance calculation (Under/Over/Close), and summary snapshot generation. Attached to trips only.                                                                                                                                      |
| **Travel Windows** | Travel Window CRUD, shortlist management, chosen-trip flagging, deletion snapshot capture for Completed trips, and archive/restore. Depends on Trip module for trip data; owns the Travel Window lifecycle entirely.                                                                                        |
| **Search**         | FTS5 full-text search index management, global search, in-context scoped search, and filter application. Write-through sync — index updated in the same transaction as every content write.                                                                                                                 |
| **Media**          | Image import (copy to app data directory), thumbnail generation via Sharp, hero image designation, and serving local images to the renderer via the `app://` Electron protocol. Deletion is soft — file purged explicitly, not on record delete.                                                            |
| **Presentation**   | Read-only orchestration layer. Assembles trip concept, location summary, budget snapshot, and hero images into presentation payloads. Reads through the other modules' service interfaces. Owns the trip snapshot shape.                                                                                    |
| **Backup**         | Local database snapshots, the optional backup folder (database + media), the startup integrity check, and restore. Works on the database file and media directory as a whole; has no knowledge of domain tables.                                                                                            |

### Module Boundary Rules

Each module has one public surface: the **service interface** exported from its
`index.ts` — plain TypeScript functions and types. Everything else in the module
directory is private.

1. **Main-process code calls other modules through their service interface**, as a
   normal function call. IPC is only the renderer↔main transport; main-process code
   never goes through `ipcMain` to reach another module.
2. **IPC handlers are a thin layer over the service interface.** Each module's `ipc.ts`
   registers its channels; a handler validates the payload, calls a service function,
   and wraps the result (see IPC Communication Pattern). No business logic lives there.
3. **A module writes only its own tables.** Reading another module's data goes through
   that module's interface. The one exception is Search, whose queries join hits to
   every indexed table (see Search Query Pattern).
4. **Dependencies run one way.** A module may call only the modules listed for it
   below. This keeps the module graph free of cycles.

| Module         | May call                                              |
| -------------- | ----------------------------------------------------- |
| Search         | — (reads indexed tables in its query joins)           |
| Media          | —                                                     |
| Backup         | — (database connection and media directory only)      |
| Content        | Search                                                |
| Tips           | Content, Search                                       |
| Trip           | Content, Search                                       |
| Budget         | Trip, Search                                          |
| Travel Windows | Trip, Budget, Media                                   |
| Presentation   | Content, Tips, Trip, Budget, Travel Windows, Media    |
| Workflows      | any module                                            |

**Workflows** hold the few operations that span modules in ways the table does not
allow — the destination deletes and the trip delete, which must stamp tips, capture a
snapshot, delete, and clear search entries as one unit. Each workflow is one function in
`src/main/workflows/` that calls the modules' service interfaces in order, and it
registers the IPC channel for that operation itself. Modules never import workflows.

**Transactions.** `better-sqlite3` is synchronous and uses one connection, so every
service call made inside `db.transaction(() => { ... })` joins that transaction. The
outermost caller — an IPC handler or a workflow — opens it; service functions must work
whether or not one is already open. This is how a content write and its search-index
update commit or roll back together across two modules.

**Enforcement mechanism:** ESLint `no-restricted-imports` rules. From outside a module,
only `src/main/modules/<name>` (its `index.ts`) may be imported; any deeper path such
as `src/main/modules/trip/trip.repository` is an error. A second rule set encodes the
"may call" table. Code review alone is not sufficient enforcement.

**Shared infrastructure exception:** A shared `db/` layer (Drizzle schema definitions,
migration runner, and database connection) is importable by all modules. It is
infrastructure, not a domain module, and does not enforce business rules.

### Directory Pattern

```
src/
├── main/              ← Electron main process (Node.js)
│   ├── modules/       ← One directory per domain module
│   │   └── <module>/
│   │       ├── index.ts        ← PUBLIC surface — the service interface
│   │       ├── ipc.ts          ← IPC channel registration (thin; calls index.ts)
│   │       ├── *.service.ts    ← Business logic (PRIVATE)
│   │       └── *.repository.ts ← DB queries (PRIVATE)
│   ├── workflows/     ← Cross-module operations (deletes that span modules)
│   ├── ipc/           ← Shared handler wrapper: result envelope, error mapping, logging
│   ├── db/            ← Shared: schema, migrations, connection
│   └── media/         ← Sharp processing, app:// protocol registration
├── preload/           ← contextBridge API exposed to the renderer
├── renderer/          ← Electron renderer process (React)
│   ├── components/
│   ├── stores/        ← Zustand stores
│   └── ipc/           ← Typed IPC client wrappers (mirrors main module surface)
└── shared/
    └── types/         ← Types shared across main and renderer (no logic):
                          channel map, payload and result types, error codes
```

---

## 3. Patterns & Flows

### IPC Communication Pattern

The renderer never calls Node.js APIs directly. All communication goes through
the contextBridge-exposed IPC client.

```
Renderer                          Main Process
────────                          ────────────
api.content.country.create(       handle('content:country:create',
  { name: 'Thailand' }              (payload) => content.createCountry(payload)
)                                 )
  → resolves with the country       → { ok: true, data: country }
  → or throws ApiError              → { ok: false, error: { code, message, ... } }
```

**One payload object in, one result envelope out.** Every channel takes a single
payload object and returns:

```
{ ok: true,  data: T }
{ ok: false, error: { code: string, message: string, ...context_fields } }
```

**Errors are returned, never thrown, across IPC.** When an `ipcMain.handle` handler
throws, Electron passes only the error's message to the renderer — the `code` and
context fields such as `cascade_preview` or `existing_id` are lost, and every
confirm-and-retry flow depends on them. So:

- Service functions signal expected failures (validation, not found, confirmation
  required) with a typed `AppError` carrying `code`, `message` and context fields.
- The shared `handle()` wrapper in `src/main/ipc/` catches it and returns the
  `ok: false` envelope. Any other exception is logged and returned as
  `{ code: "INTERNAL_ERROR" }` with no internal detail.
- The renderer's IPC client unwraps the envelope: it resolves with `data`, or throws an
  `ApiError` that carries the full error object.

**Channels and types.** Channel names follow `<module>:<resource>:<action>` and are
listed per operation in `api-contracts.md`. One channel map in `src/shared/types/`
types the payload and result of every channel; the preload script, the main handlers
and the renderer's `ipc/` wrappers are all typed from it, so the renderer never
constructs raw channel name strings.

**Logging.** The main process writes a rotating log file under
`app.getPath('userData')/logs/`. Unexpected errors are logged with the channel name and
stack trace; expected `AppError`s are not. Nothing is sent off the machine.

**Security:** `nodeIntegration: false`, `contextIsolation: true` on all
BrowserWindow instances. The renderer has no access to Node.js APIs.

---

### Status Transition Model

**Country status** (manual, no enforcement on transitions):

```
Wishlist → Researching → Sampling → Planning
  ↑_______________|_______________|__________↑  (any direction permitted)
```

**Trip status** (manual, no enforcement on transitions):

```
Sample → Planning → Ready → Completed
  ↑_________|_________|_________|  (any direction permitted)
```

- Status transitions never trigger automatic content changes
- Moving to `Completed` sets `completed_at` to today if currently null;
  moving away from `Completed` retains `completed_at` (not cleared)
- `completed_at` is editable only while the trip is `Completed` (or in the same save
  that marks it `Completed`); the retained date is hidden, not editable, otherwise

---

### Trip Itinerary State Machine

A trip block moves through two modes. A trip can be in a mixed state — some blocks
expanded into days, others remaining as location blocks.

```
  Location-Block Mode          Day-by-Day Mode
  ───────────────────          ───────────────
  [Block: Thailand, 7d]   →   [Day 1] [Day 2] ... [Day 7]
  [Block: Laos, 4d]            ↑ expand() requires duration set
  [Block: Vietnam, 5d]    →   [Day 1] [Day 2] ... [Day 5]
                               ↑ expanded independently
```

Transitions:

- `expand(block_id, duration_days)` — creates `trip_days` rows; sets block `duration_days`
- Expanding a block does not auto-populate days with POIs — days start empty
- `collapse(block_id)` — removes the block's days and returns it to location-block
  mode, keeping its duration; requires confirmation if any day has content

Rules for an expanded block:

- It always has exactly `duration_days` days, numbered 1..n with no gaps
- Raising `duration_days` appends empty days; lowering it removes days from the end and
  requires explicit confirmation
- Individual days can be inserted, deleted, reordered, and moved to another expanded
  block of the same trip. Each of these keeps both invariants: siblings are renumbered
  and the block's `duration_days` is adjusted
- `split(block_id, first_day_number)` — divides the block into two consecutive blocks,
  which is how a broad block's days are reorganized into more specific ones (US-013)
- A block's only day cannot be deleted or moved out; collapse or delete the block instead

---

### Deletion Cascade and Snapshot Pattern

Every FK column declares its `ON DELETE` action in the schema; the full reference
matrix and the per-record delete paths are in `data-model.md` (Integrity Layer).
The application layer wraps each delete with the steps the database cannot do:
warn or refuse, stamp tips, capture a snapshot, and clear `search_index` rows.
Everything for one delete runs in a single transaction.

**Destination deletes protect trips.**

```
DELETE country                        DELETE region / city
────────────────────────────          ──────────────────────────────────
Used by any trip block?               Warn: nested content, tips to unlink,
  yes → refuse (RECORD_IN_USE,          blocks to widen, trip-day POI entries
        list the trips)                         ↓ confirm
  no  → warn: nested content,         Stamp tips with former_destination
        tips to unlink                Delete (FK actions do the rest):
         ↓ confirm                      nested cities/POIs/media removed
Stamp tips with former_destination      tips → Unlinked
Delete (FK actions do the rest)         blocks widen to the parent level
```

Tips are never deleted by a destination delete. A destination-scoped tip with no
destination link is Unlinked; it stays editable and is listed for reassignment.

**Trip deletes preserve Completed trips.**

```
DELETE trip (non-Completed)          DELETE trip (Completed)
────────────────────────────         ──────────────────────────────────
Warn: referenced in N windows        Warn: referenced in N windows
         ↓ confirm                            ↓ confirm
Stamp linked Lessons Learned         Stamp linked Lessons Learned
Delete trip                          If in at least one Travel Window:
  (travel_window_trips rows,           1. Build full-detail presentation payload
   blocks, days, budgets, media           → one trip_snapshots row (versioned)
   removed by FK cascade)              2. Re-parent trip media to the snapshot;
                                          copy POI/country media rows it shows
                                       3. On each travel_window_trips row:
                                          set snapshot_id, null trip_id
                                     Delete trip
```

**Snapshot shape.** A snapshot is the Presentation full-detail trip payload
(`GET /api/presentation/:window_id/trips/:id`), stored fully denormalised with a
`snapshot_version`. The summary, comparison and Travel Window views are projections
of that same payload, so the renderer handles live and preserved trips identically.
The Travel Windows Module captures and stores the snapshot. The Presentation Module
owns the shape: it supplies the payload at deletion and upgrades older versions
when they are read. Stored snapshots are never rewritten.

**Snapshot lifetime.** One snapshot is shared by every Travel Window the trip was
in. It is deleted, with its media rows, when the last of those windows is deleted.

---

### FTS5 Write-Through Sync Pattern

Every write that changes a record's indexed text updates the search index in the
same SQLite transaction. No background jobs. No eventual consistency.

The index is two tables (see `data-model.md`): `search_docs` maps each searchable
record to a rowid, and the FTS5 table `search_index` holds only `title` and `body`
for that rowid.

```
createCountry(payload)
  ├── INSERT INTO countries ...
  └── search.upsert('country', id, name, notes_plain_text)
        ├── INSERT INTO search_docs ... ON CONFLICT DO NOTHING   → doc_id
        ├── DELETE FROM search_index WHERE rowid = doc_id
        └── INSERT INTO search_index (rowid, title, body) VALUES (doc_id, ?, ?)
  └── COMMIT  ← all writes succeed or all roll back
```

Plain text for `body` is extracted from TipTap JSON at write time by a shared
`extractPlainText(tiptapJson)` utility. Never stored separately — always derived
on write.

**When a write touches the index:**

| Write                                                              | Index action                                |
| ------------------------------------------------------------------ | ------------------------------------------- |
| Create a country, region, city, POI, trip, tip or non-template budget | `upsert` that record                     |
| Update that changes an indexed field (name/title, notes, description, practical notes, content) | `upsert` that record |
| Update that changes no indexed field (status, position, dates, links, tags) | none                               |
| Create, rename or delete a budget category or line item; apply a template | `upsert` the parent budget           |
| Delete a record                                                    | `remove` it and every indexed record the cascade deletes |
| Rename or move a parent (country, region, city, trip)              | none — parent context is read live at query time |

The index stores only a record's own text, so no write to one record can make
another record's entry stale. The single exception is budgets, whose body is built
from their categories and line items (row 4).

**Consistency check.** On startup the Search Module compares the number of
`search_docs` rows with the number of indexable records. If they differ it runs a
full rebuild (`POST /api/search/index/rebuild`). The same rebuild runs after any
migration that changes what is indexed.

---

### Search Query Pattern

A search runs the FTS match once and joins each hit to its **live source row**.
Display fields and filters come from the source tables, never from the index.

```
search(q, filters)
  1. Build the MATCH expression from user input (never passed through raw):
       each whitespace-separated term → double-quoted, embedded quotes doubled
       last term gets a trailing * (prefix match, for search-as-you-type)
       "wat phra-that"  →  "wat" "phra-that"*
  2. One query arm per content type in scope, UNION ALL-ed:
       search_index MATCH ?
         JOIN search_docs d ON d.id = search_index.rowid
         JOIN <source table> s ON d.content_type = '<type>' AND s.id = d.record_id
       WHERE <filters, as conditions on s and its live relations>
  3. ORDER BY bm25(search_index, 10.0, 1.0)   -- title weighted 10× body
     snippet() taken from the body column
  4. parent_context assembled from the live hierarchy for the page of results
```

Because every arm inner-joins its source table, a search entry whose record no
longer exists can never appear in results.

**Filter-only browsing.** When a search has filters but no keyword, the same arms run
without the FTS match: each selects straight from its source table with the filter
conditions, ordered by title. This serves "filter the library by tag or status" with one
code path and one result shape.

Filters are conditions on the live source row, so each content type states what a
filter means for it. A filter that has no meaning for a content type excludes that
type from the results.

| Content type | Country / region / city scope                            | Status           | Tag                       | Parent context shown                       |
| ------------ | -------------------------------------------------------- | ---------------- | ------------------------- | ------------------------------------------ |
| country      | is that country                                          | `countries.status` | `content_tags.country_id` | —                                          |
| region       | its `country_id` / is that region                        | excluded         | `content_tags.region_id`  | Country                                    |
| city         | its `country_id` / `region_id` / is that city            | excluded         | `content_tags.city_id`    | Country > Region                           |
| poi          | its `country_id`, or its region/city resolved through the hierarchy | excluded | `content_tags.poi_id`   | Country > Region > City                    |
| trip         | linked through `trip_countries` (country scope only)     | `trips.status`   | `content_tags.trip_id`    | Linked country names                       |
| tip          | its destination, resolved up the hierarchy; General and Unlinked tips excluded | excluded | excluded | Destination path, "General", or "Unlinked" |
| budget       | its trip's countries (country scope only)                | excluded         | excluded                  | Trip name                                  |

---

### Image Import and Serving Pattern

The renderer cannot read file paths from the DOM (`File.path` no longer exists in
Electron), and it never reads image bytes itself. A path reaches the main process one
of two ways:

```
"Add image" button                       Drag and drop onto a record
──────────────────                       ───────────────────────────
Renderer calls media:pick                Preload calls webUtils.getPathForFile(file)
Main opens dialog.showOpenDialog         for each dropped File and returns the paths
  (image filters, multi-select)          to the renderer
Main returns the chosen paths
                     ↓                          ↓
              Renderer calls media:import with { source_path, parent }
                                   ↓
Media Module (main process)
  1. Read source file from source_path
  2. Detect the image type from the file's contents (Sharp metadata) —
     never from the extension or anything the renderer says
  3. Reject anything that is not jpeg / png / webp / gif
  4. Generate cuid2 filename
  5. Sharp: copy + optimise → app data dir /media/{filename}
  6. Sharp: generate thumbnail → app data dir /media/thumbnails/{filename}
  7. Insert media record into DB
  8. Return { app_url: 'app://media/{filename}', thumbnail_url: '...' }
                                   ↓
app:// protocol (registered in main process via protocol.handle)
  Maps app://media/{filename} → a file in the app data dir, served with net.fetch
  Renderer uses app_url directly in <img src> — never a raw file:// path
```

Both entry points are supported because both are expected in a desktop app; they
converge on one import call so validation lives in one place.

Orphaned files (files on disk that no media row references) are cleaned up
by `POST /api/media/purge`, which runs automatically on every app startup and
is also available to trigger manually. A file can be referenced by more than one
media row: a snapshot of a deleted Completed trip holds its own rows for the
images it shows, so those files outlive the library records they came from.

---

### Backup and Recovery Pattern

The whole library is one SQLite file and one media directory. Two layers protect it.

| Layer            | Contains                    | Where                                  | Protects against                                    |
| ---------------- | --------------------------- | -------------------------------------- | --------------------------------------------------- |
| Local snapshots  | Database only               | `userData/backups/` (automatic)        | Failed migration, corruption, a bad edit or delete  |
| Backup folder    | Database + all media files  | A folder the user chooses (optional)   | Disk loss, OS reinstall, moving to a new machine    |

**Database settings.** The connection runs in WAL mode (`journal_mode = WAL`,
`synchronous = NORMAL`, `foreign_keys = ON`). In WAL mode the database is three files,
so it is **never copied with a plain file copy** while open. Every copy is made with
SQLite's online backup API (`db.backup()` in better-sqlite3), which produces a single
consistent file.

**Local snapshots** (always on):

```
App startup
  1. PRAGMA quick_check          → on failure: recovery screen (restore a snapshot)
  2. Pending migrations?
       yes → snapshot  backups/pre-migration-<timestamp>.db
             run migrations in one transaction
             on failure: roll back, restore that snapshot, show the error, do not open
  3. No daily snapshot in the last 24h?
       yes → snapshot  backups/daily-<date>.db
  4. Prune: keep the 7 newest daily and the 5 newest pre-migration snapshots
```

**Backup folder** (optional, strongly prompted): the user picks a folder in Settings —
an external drive, or a folder their own sync or backup tool covers. The app writes:

```
<chosen folder>/Wanderly Backup/
  ├── manifest.json            ← app version, schema version, timestamps, record counts
  ├── db/wanderly-<timestamp>.db   ← the 3 newest database copies
  └── media/                   ← mirror of the media directory, including thumbnails
```

- Runs when the app quits (if anything changed since the last run) and on "Back up now".
- Media files never change after import, so the mirror only copies files it does not
  have. A file is removed from the mirror only when none of the retained database copies
  references it — so every retained copy can be restored in full.
- Each database copy is written to a temporary name and renamed when complete.
- If the folder is unavailable (drive unplugged), the run is skipped and the status shows
  the last successful backup. Settings warns when that is more than 7 days old, and
  shows a standing notice while no folder is configured.
- The backup folder is a destination only. The live database always stays in `userData`
  and is never opened from a synced or removable location.

**Restore** (Settings, the recovery screen, or first run on a new machine):

```
Choose a restore point (local snapshot, or a backup folder)
  1. Validate: quick_check passes; schema version is not newer than this app
  2. Confirm with the user — the current library will be replaced
  3. Snapshot the current database to backups/pre-restore-<timestamp>.db
  4. Close the connection; swap in the restored database
     (backup folder only: copy its media into the media directory)
  5. Relaunch → pending migrations run → search index rebuilt
```

A local snapshot restores the database only; media files stay as they are, and any
images referenced by the restored database but since purged show as missing. A backup
folder restores both.

**Where settings live.** The backup folder path and last-backup times are stored in
`userData/settings.json`, not in the database, so restoring a database never changes
where backups go.

**Moving machines.** Nothing in the database stores an absolute path: media rows hold a
filename and the `app://` handler resolves it against the current media directory. The
data directory is always `app.getPath('userData')`; moving to a new machine is install,
then restore from the backup folder.

---

## 4. Security

### Process Isolation

| Setting                       | Value   | Reason                                                        |
| ----------------------------- | ------- | ------------------------------------------------------------- |
| `nodeIntegration`             | `false` | Renderer cannot access Node.js APIs directly                  |
| `contextIsolation`            | `true`  | contextBridge is the only renderer↔main communication surface |
| `webSecurity`                 | `true`  | Default; not disabled                                         |
| `allowRunningInsecureContent` | `false` | Default                                                       |

### Custom Protocol Security

The `app://` scheme is declared with `protocol.registerSchemesAsPrivileged` (standard,
secure) before the app is ready, and served with `protocol.handle`. The older
`protocol.registerFileProtocol` API is deprecated and is not used. The handler:

- Serves files only from the app data `/media/` directory
- Accepts only two path shapes — `media/{filename}` and `media/thumbnails/{filename}` —
  where `{filename}` matches the generated pattern (lowercase alphanumeric ID plus a
  known image extension). Anything else returns 404
- As a second check, resolves the final absolute path and confirms it is inside the
  media directory before reading. This covers encoded sequences and Windows
  backslashes, which a search for `../` in the string would miss
- Is not accessible from external web content

### Content Security Policy

Applied to all BrowserWindow instances in the packaged app:

```
default-src 'self';
script-src 'self';
style-src 'self';
img-src 'self' app: data:;
font-src 'self' data:;
connect-src 'none';
```

`connect-src 'none'` enforces that the renderer makes no network requests — all
data comes through IPC.

No `'unsafe-inline'` is needed in production. Tailwind compiles to a static stylesheet
at build time. Styles that React and @dnd-kit set on elements go through the DOM style
API, which CSP does not restrict. TipTap is configured with `injectCSS: false` and its
base editor styles are included in the app stylesheet, because its default behaviour of
injecting a `<style>` element would be blocked.

**Development only:** the Vite dev server needs a looser policy — inline styles and
scripts for hot module replacement, and a connection back to the dev server:

```
script-src 'self' 'unsafe-inline';
style-src 'self' 'unsafe-inline';
connect-src ws://localhost:* http://localhost:*;
```

The policy is chosen by build mode in one place. The development policy must never be
present in a packaged build.

### Data Residency

All data remains on the user's local machine:

- SQLite database file: Electron `app.getPath('userData')/wanderly.db` (plus its
  `-wal` and `-shm` companions while open)
- Media files: Electron `app.getPath('userData')/media/`
- Local snapshots: `app.getPath('userData')/backups/`
- App settings and logs: `app.getPath('userData')/settings.json` and `logs/`
- Backup folder (optional): a local path chosen by the user. The app only writes files
  there; it never uploads anything
- No telemetry, no analytics, no external network calls from the app

### Dependency Security

- No third-party authentication libraries required (single-user, no auth)
- No network-facing server process — attack surface is local only

### Native Modules and Packaging

Two dependencies contain native code: `better-sqlite3` and Sharp. Both are Node-API
modules that ship prebuilt binaries, so they load in Electron without being compiled
against a specific Electron version. Packaging still has to handle them:

- **Unpack from the ASAR archive.** Native binaries cannot be loaded from inside
  `app.asar`. electron-builder's `asarUnpack` must list `better-sqlite3`, `sharp` and
  Sharp's platform packages (`@img/*`).
- **Keep them external to the bundle.** electron-vite must not bundle either module into
  the main-process output; they stay as runtime `node_modules` dependencies.
- **Right binary per target.** The packaged app must contain the binaries for the
  platform and CPU it is built for (macOS arm64 and x64, Windows x64). Build each
  target on, or explicitly for, that platform.
- **Fallback.** If a prebuilt binary is ever missing for a target, the module is
  compiled from source with `@electron/rebuild` (the successor to `electron-rebuild`),
  which electron-builder invokes during packaging.

**Migration files ship with the app.** The SQL migrations generated by Drizzle Kit are
copied into the package as `extraResources` and read from `process.resourcesPath` at
startup. Drizzle Kit itself is a development dependency and is not shipped.

---

## 5. Performance

Targets apply on both supported platforms. Reference machines: a mid-range MacBook
(Apple M-series, 16GB RAM) and a mid-range Windows 11 laptop (4-core, 16GB RAM, SSD).
Measurement mechanisms stated per target.

| Target          | Metric                                                 | Value          | Measurement                                                                                |
| --------------- | ------------------------------------------------------ | -------------- | ------------------------------------------------------------------------------------------ |
| Search response | Time from IPC call to response in main process         | < 100ms        | Measured via `performance.now()` wrapping the IPC handler; logged in development mode      |
| App startup     | Cold start to interactive window                       | < 3 seconds    | Measured from process launch to `BrowserWindow` `did-finish-load` event; logged on startup |
| Content write   | Time from IPC call to confirmed DB write + FTS5 sync   | < 50ms         | Measured via `performance.now()` in main process; logged in development mode               |
| Library size    | No degradation in search or navigation up to this size | 10,000 records | Verified with a seeded test database of 10,000 records across all content types            |
| Image import    | Time from file selection to thumbnail visible in UI    | < 2 seconds    | Measured from IPC call to renderer receiving the media record response                     |

**Search library scale assumption:** The target user accumulates research over years.
10,000 records covers an aggressive multi-year research library across all content types.
SQLite FTS5 handles this volume comfortably; no query optimisation beyond the defined
indexes is expected to be necessary.

**Synchronous main-process work — accepted trade-off.** `better-sqlite3` is synchronous
and runs in the main process, so the main process is blocked for the duration of each
query. At the target library size every normal read and write is a few milliseconds,
well inside the targets above, and a synchronous API is what makes cross-module
transactions simple. Three operations can take noticeably longer — a full search index
rebuild, the startup integrity check, and restoring a backup — and each runs only at
startup or behind an explicit user action with a progress state. Image processing
(Sharp) and database backups (SQLite online backup) are asynchronous and do not block.
If the library grows well past the target or a query misses its target, the remedy is to
move the database to a worker thread behind the same service interfaces.

**Startup migration time:** Migrations are applied synchronously on startup by Drizzle
ORM's runtime migrator, before the main window opens. Migration time is expected to be negligible (< 200ms) for the
schema defined. A database snapshot is taken automatically before any pending migration
runs (see Backup and Recovery Pattern); for a library of the target size this adds well
under a second.

---

## 6. Frontend

### Route Inventory

| Route                         | View                 | Notes                                          |
| ----------------------------- | -------------------- | ---------------------------------------------- |
| `/`                           | Master Library       | Default view; country list with status filters |
| `/countries/:id`              | Country Detail       | Regions, cities, POIs, tips, budgets nested    |
| `/regions/:id`                | Region Detail        | Cities, POIs, tips nested                      |
| `/cities/:id`                 | City Detail          | POIs, tips nested                              |
| `/pois/:id`                   | POI Detail           | Full POI record with media                     |
| `/trips`                      | Trip Library         | All trips with status filter                   |
| `/trips/:id`                  | Trip Detail          | Itinerary builder — block and day-by-day modes |
| `/trips/:id/budget`           | Budget View          | Budget categories, line items, summary         |
| `/travel-windows`             | Travel Windows       | Active and archived windows                    |
| `/travel-windows/:id`         | Travel Window Detail | Shortlisted trips, chosen flag                 |
| `/travel-windows/:id/present` | Presentation Mode    | Full-screen; no app chrome                     |
| `/tips`                       | Tips Library         | General and destination tips, filterable       |
| `/search`                     | Search Results       | Global search with filters                     |

### Key Frontend Constraints

- **Desktop only** — no responsive breakpoints required; minimum supported
  window width is 1280px
- **No mobile layout** — mobile support is deferred scope; do not build responsive
  variants
- **Presentation mode** is a distinct full-screen route with all app chrome hidden;
  keyboard-navigable per US-042 (Escape, Arrow keys, Enter, Backspace, Tab)
- **TipTap editors** render in all notes fields across all content types; rich text
  must render correctly in presentation mode (read-only TipTap instance)
- **Autosave** (US-007): edits to text fields are saved 1 second after the last
  keystroke, and immediately on blur, route change and window close. The app does not
  quit until pending saves have completed. One save is one IPC call and one transaction
- **Drag and drop** via @dnd-kit: itinerary block reordering (trip builder),
  day POI reordering, budget category and line item reordering
- **Ctrl+F** (Cmd+F on macOS) is handled by the app and opens search: in-context
  search inside the trip builder or a country, region or city record, global search
  everywhere else (US-026, US-029). Registered as a `CmdOrCtrl+F` menu accelerator.
  Electron has no built-in find bar, so there is no native behaviour to defer to;
  highlighting matches inside a long note (find-in-page) is not part of this version

---

## 7. Open Action Items

Items in this table block implementation of the affected components.
No work should begin on an affected area until the item is resolved.

| #   | Item                                                                                                                                                                                                               | Owner               | Needed Before                | Impact if Unresolved                                                     |
| --- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ | ------------------- | ---------------------------- | ------------------------------------------------------------------------ |
| B   | **Native module packaging** — on the first packaged build for each target (macOS arm64, macOS x64, Windows x64), confirm `better-sqlite3` and Sharp load from the unpacked ASAR directory and that the database opens and an image imports | DevOps / Build      | First packaged build         | App crashes on launch, or image import fails, in the packaged app only   |
| D   | **TipTap plain text extraction** — confirm the `extractPlainText(tiptapJson)` utility correctly handles all TipTap node types used in the app (bold, italic, lists, links, etc.) before FTS5 indexing is built     | Backend Engineer    | Search module implementation | FTS5 index contains malformed body text; search returns degraded results |

**Resolved:**

- **A — Ctrl+F behaviour.** The app handles Ctrl+F / Cmd+F and opens the most relevant
  search. See Key Frontend Constraints.
- **C — App data directory migration.** The data directory is fixed at
  `app.getPath('userData')` and no absolute paths are stored. Moving to a new machine is
  a restore from the backup folder (Backup and Recovery Pattern).
- **E — Budget cost-per-day with mixed block state.** Cost per day is null ("incomplete")
  unless every block on the trip has a duration. See the Budget module's Key behaviours in
  `api-contracts.md`.
