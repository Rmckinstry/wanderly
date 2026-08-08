# Architecture — Wanderly (Local Desktop)

## 1. Overview

### System Diagram

```
┌─────────────────────────────────────────────────────────────────┐
│  Renderer Process                                               │
│                                                                 │
│  React 19 · Zustand 5 · TipTap 2 · @dnd-kit · Tailwind 4      │
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
│  ┌────────────┐ ┌──────────┐ ┌──────────┐                      │
│  │   Search   │ │  Media   │ │ Present- │                      │
│  │   Module   │ │  Module  │ │  ation   │                      │
│  └────────────┘ └──────────┘ └──────────┘                      │
│                                                                 │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │  Database Layer — Drizzle ORM · better-sqlite3 · SQLite │   │
│  └─────────────────────────────────────────────────────────┘   │
│                                                                 │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │  Media Layer — Sharp · app:// protocol · app data dir   │   │
│  └─────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────┘
```

---

### Tech Stack

| Layer              | Choice                  | Version | Primary Reason                                                                                                                                    | Alternative Rejected                                                                                                                                  |
| ------------------ | ----------------------- | ------- | ------------------------------------------------------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------- |
| App shell          | Electron                | 33.x    | Battle-tested for rich local data apps; Node.js backend gives direct SQLite and file system access with no bridge layer                           | Tauri — Rust backend adds friction for file ops and SQLite with no meaningful benefit for a single-user personal tool where bundle size is irrelevant |
| Frontend framework | React                   | 19.x    | Industry standard for complex interactive UIs; hooks model handles trip builder state, presentation mode transitions, and drag-and-drop naturally | Vue 3 — viable but smaller ecosystem; React better matched to the complexity of nested stateful views                                                 |
| State management   | Zustand                 | 5.x     | Lightweight, boilerplate-free; no multi-user or real-time requirements to justify Redux overhead                                                  | Redux Toolkit — overkill for this scale; significant boilerplate with no benefit                                                                      |
| Rich text editor   | TipTap                  | 2.x     | ProseMirror-based, React-native, stores content as portable JSON; actively maintained                                                             | Quill — Delta format less portable; maintenance trajectory uncertain                                                                                  |
| Database           | SQLite (better-sqlite3) | 9.x     | Named in requirements; synchronous API fits Electron's main process model cleanly                                                                 | PostgreSQL — requires a running server process, wrong for a desktop app                                                                               |
| ORM                | Drizzle ORM             | 0.36.x  | Type-safe SQL, lightweight, no generated client binary, works with better-sqlite3 natively                                                        | Prisma — query engine binary complicates Electron packaging; heavier abstraction                                                                      |
| Migrations         | Drizzle Kit             | 0.28.x  | Pairs with Drizzle ORM; generates versioned SQL migration files                                                                                   | Manual SQL — error-prone across app versions; no rollback story                                                                                       |
| Build / bundler    | Vite + electron-vite    | 6.x     | Fast HMR; electron-vite handles dual main/renderer build with minimal config                                                                      | Webpack — slower build times, more config overhead                                                                                                    |
| Styling            | Tailwind CSS            | 4.x     | Utility-first; no context switching between CSS files and components; right scale for solo development                                            | CSS Modules — more verbose for this project size                                                                                                      |
| Drag and drop      | @dnd-kit/core           | 6.x     | Accessible, headless, React-native; handles itinerary block and POI reordering                                                                    | react-beautiful-dnd — effectively deprecated                                                                                                          |
| Image processing   | Sharp                   | 0.33.x  | Runs in Electron main process; handles copy-on-import, thumbnail generation, hero image designation                                               | Canvas API — cannot write to file system from renderer                                                                                                |
| Packaging          | electron-builder        | 25.x    | Handles macOS, Windows, and Linux packaging with code signing hooks                                                                               | electron-forge — viable but electron-builder has broader format support                                                                               |

**Version source of truth:** All downstream documents reference versions from this table.
Downstream files must never restate version numbers independently.

---

### Non-Negotiable Constraints

| Constraint                                                              | Failure Mode if Violated                                                          |
| ----------------------------------------------------------------------- | --------------------------------------------------------------------------------- |
| Local storage only — SQLite on the local file system, no cloud database | Data leaves the user's machine; privacy violated; app no longer functions offline |
| Single-user — no authentication layer                                   | N/A for this version; multi-user deferred explicitly                              |
| Desktop browser only — no mobile layout required                        | UI breaks on mobile; deferred scope activated prematurely                         |
| No internet connection required to function                             | App unusable in offline environments; defeats the purpose of a local tool         |
| Electron main process owns all DB and file system access                | Renderer gets direct Node access (nodeIntegration: true); security anti-pattern   |
| Images copied into app data directory on import                         | Source file deletion breaks app image references                                  |

---

## 2. Modules

### Module Map

| Module             | Responsibility                                                                                                                                                                                                                                                                                              |
| ------------------ | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Content**        | CRUD for all core content types: Country, Region, City, POI, Tip/Lesson Learned. Owns the geographic hierarchy, inter-content linking, status management, and cascade-warning logic before destructive operations. Largest module.                                                                          |
| **Trip**           | Trip lifecycle: location-block management, day-by-day expansion, POI attachment to days, status transitions, and `completed_at` tracking. Distinct from Content because trip planning logic — block ordering, duration calculation, mixed-state management — is complex enough to warrant its own boundary. |
| **Budget**         | Budget CRUD, category and line item management, template save/apply, variance calculation (Under/Over/Close), and summary snapshot generation. Attached to trips only.                                                                                                                                      |
| **Travel Windows** | Travel Window CRUD, shortlist management, chosen-trip flagging, deletion snapshot capture for Completed trips, and archive/restore. Depends on Trip module for trip data; owns the Travel Window lifecycle entirely.                                                                                        |
| **Search**         | FTS5 full-text search index management, global search, in-context scoped search, and filter application. Write-through sync — index updated in the same transaction as every content write.                                                                                                                 |
| **Media**          | Image import (copy to app data directory), thumbnail generation via Sharp, hero image designation, and serving local images to the renderer via the `app://` Electron protocol. Deletion is soft — file purged explicitly, not on record delete.                                                            |
| **Presentation**   | Read-only orchestration layer. Assembles trip concept, location summary, budget snapshot, and hero images into presentation payloads. Delegates all data retrieval to other modules.                                                                                                                        |

### Module Boundary Rules

Modules communicate only through their public interface — exported handler functions
registered on `ipcMain`. No module imports directly from another module's internal files.

**Enforcement mechanism:** ESLint `no-restricted-imports` rules. Example:
`src/main/modules/budget/` cannot import from `src/main/modules/trip/internal/`.
Cross-module data needs go through the IPC handler layer or a shared types package.
Code review alone is not sufficient enforcement.

**Shared infrastructure exception:** A shared `db/` layer (Drizzle schema definitions,
migration runner, and database connection) is importable by all modules. It is
infrastructure, not a domain module, and does not enforce business rules.

### Directory Pattern

```
src/
├── main/              ← Electron main process (Node.js)
│   ├── modules/       ← One directory per domain module
│   │   └── <module>/
│   │       ├── index.ts        ← PUBLIC surface — IPC handlers registered here
│   │       ├── *.service.ts    ← Business logic (PRIVATE)
│   │       └── *.repository.ts ← DB queries (PRIVATE)
│   ├── db/            ← Shared: schema, migrations, connection
│   └── media/         ← Sharp processing, app:// protocol registration
├── renderer/          ← Electron renderer process (React)
│   ├── components/
│   ├── stores/        ← Zustand stores
│   └── ipc/           ← Typed IPC client wrappers (mirrors main module surface)
└── shared/
    └── types/         ← Types shared across main and renderer (no logic)
```

---

## 3. Patterns & Flows

### IPC Communication Pattern

The renderer never calls Node.js APIs directly. All communication goes through
the contextBridge-exposed IPC client.

```
Renderer                        Main Process
────────                        ────────────
ipcRenderer.invoke(             ipcMain.handle(
  'content:country:create',       'content:country:create',
  { name: 'Thailand' }            async (event, payload) => {
)                                   return contentModule.createCountry(payload)
  .then(result => ...)            }
                                )
```

The contextBridge API is typed end-to-end. The renderer's `ipc/` wrappers mirror
the main module surface with matching TypeScript types, so the renderer never
constructs raw channel name strings.

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
- `completed_at` is independently editable at any time regardless of status

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
- Reducing `duration_days` below existing day count requires explicit confirmation
- Expanding a block does not auto-populate days with POIs — days start empty
- A block cannot be un-expanded once days have content (delete days first)

---

### Deletion Cascade and Snapshot Pattern

Two deletion paths exist for trips referenced in Travel Windows:

```
DELETE trip (non-Completed)          DELETE trip (Completed)
────────────────────────────         ──────────────────────────────────
Warn: referenced in N windows        Warn: referenced in N windows
         ↓ confirm                            ↓ confirm
Remove travel_window_trips rows      For each travel_window_trips row:
                                       1. Serialize full trip state → JSON
                                       2. Write to snapshot_data column
                                       3. Set trip_id = NULL
                                       4. Set is_deleted_record = 1
                                       5. Set deleted_at = now()
                                     Then hard delete trip record
```

The snapshot shape is identical to the live trip API response shape. The
Presentation Module deserialises `snapshot_data` and returns it with the same
structure as a live trip — the renderer handles both cases identically.

---

### FTS5 Write-Through Sync Pattern

Every content write updates the `search_index` in the same SQLite transaction.
No background jobs. No eventual consistency. Search is always current.

```
createCountry(payload)
  ├── INSERT INTO countries ...
  ├── DELETE FROM search_index WHERE record_id = ?
  └── INSERT INTO search_index (content_type, record_id, parent_context, title, body)
        VALUES ('country', id, '', name, notes_plain_text)
  └── COMMIT  ← both writes succeed or both roll back
```

Plain text for the `body` field is extracted from TipTap JSON at write time by
a shared `extractPlainText(tiptapJson)` utility. Never stored separately —
always derived on write.

---

### Image Import and Serving Pattern

```
User selects file
      ↓
Media Module (main process)
  1. Read source file from local path
  2. Validate mime type
  3. Generate cuid2 filename
  4. Sharp: copy + optimise → app data dir /media/{filename}
  5. Sharp: generate thumbnail → app data dir /media/thumbnails/{filename}
  6. Insert media record into DB
  7. Return { app_url: 'app://media/{filename}', thumbnail_url: '...' }
      ↓
app:// protocol (registered in main process via protocol.registerFileProtocol)
  Maps app://media/{filename} → absolute path in app data dir
  Renderer uses app_url directly in <img src> — never a raw file:// path
```

Orphaned files (media rows deleted, files remaining on disk) are cleaned up
by `POST /api/media/purge`, which runs automatically on every app startup and
is also available to trigger manually.

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

The `app://` protocol is registered as a privileged scheme that:

- Serves files only from the app data `/media/` directory
- Rejects path traversal attempts (any `../` in the requested path returns 404)
- Is not accessible from external web content

### Content Security Policy

Applied to all BrowserWindow instances:

```
default-src 'self';
script-src 'self';
style-src 'self' 'unsafe-inline';
img-src 'self' app: data:;
font-src 'self' data:;
connect-src 'none';
```

`unsafe-inline` on `style-src` is required for Tailwind's runtime class generation.
`connect-src 'none'` enforces that the renderer makes no network requests — all
data comes through IPC.

### Data Residency

All data remains on the user's local machine:

- SQLite database file: Electron `app.getPath('userData')/wanderly.db`
- Media files: Electron `app.getPath('userData')/media/`
- No telemetry, no analytics, no external network calls from the app

### Dependency Security

- `better-sqlite3` is a native module — must be rebuilt against Electron's Node ABI
  using `electron-rebuild` as part of the build pipeline
- No third-party authentication libraries required (single-user, no auth)
- No network-facing server process — attack surface is local only

---

## 5. Performance

All targets measured on a mid-range MacBook (Apple M-series, 16GB RAM).
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

**Startup migration time:** Drizzle Kit migrations run synchronously on startup before
the main window opens. Migration time is expected to be negligible (< 200ms) for the
schema defined — all migrations are additive. Destructive migrations that could take
longer require a separate confirmed backup step surfaced to the user before running.

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

- **Desktop browser only** — no responsive breakpoints required; minimum supported
  viewport width is 1280px
- **No mobile layout** — mobile support is deferred scope; do not build responsive
  variants
- **Presentation mode** is a distinct full-screen route with all app chrome hidden;
  keyboard-navigable per US-042 (Escape, Arrow keys, Enter, Backspace, Tab)
- **TipTap editors** render in all notes fields across all content types; rich text
  must render correctly in presentation mode (read-only TipTap instance)
- **Drag and drop** via @dnd-kit: itinerary block reordering (trip builder),
  day POI reordering, budget category and line item reordering
- **Ctrl+F** behaviour (app intercept vs. browser native find-in-page) is an open
  decision — see Open Action Items

---

## 7. Open Action Items

Items in this table block implementation of the affected components.
No work should begin on an affected area until the item is resolved.

| #   | Item                                                                                                                                                                                                               | Owner               | Needed Before                | Impact if Unresolved                                                     |
| --- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ | ------------------- | ---------------------------- | ------------------------------------------------------------------------ |
| A   | **Ctrl+F behaviour** — determine whether intercepting Ctrl+F for in-context search is best practice for an Electron web app, or whether deferring to browser native find-in-page is the better convention (US-029) | Frontend Engineer   | Trip builder implementation  | In-context search trigger undefined; cannot implement US-026/US-029      |
| B   | **electron-rebuild pipeline** — confirm `better-sqlite3` native module rebuild against Electron's Node ABI is integrated into the CI/build pipeline before first packaging attempt                                 | DevOps / Build      | First packaged build         | App crashes on launch if native module compiled against wrong ABI        |
| C   | **App data directory migration** — define strategy for handling the app data directory path if the user moves or renames it (e.g. on a new machine or OS migration)                                                | Architect / Backend | Media module implementation  | Media files become unreachable; app_url references break                 |
| D   | **TipTap plain text extraction** — confirm the `extractPlainText(tiptapJson)` utility correctly handles all TipTap node types used in the app (bold, italic, lists, links, etc.) before FTS5 indexing is built     | Backend Engineer    | Search module implementation | FTS5 index contains malformed body text; search returns degraded results |
| E   | **Budget cost-per-day with mixed block state** — define calculation when a trip has some blocks with duration set and others without (partial duration coverage)                                                   | PM + Backend        | Budget module implementation | cost_per_day returns null or incorrect value for partially-planned trips |
