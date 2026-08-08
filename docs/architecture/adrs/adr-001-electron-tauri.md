# ADR-001: Electron over Tauri

**Status:** Accepted  
**Date:** 2026-04-18  
**Applies to:** App shell — all phases

---

## Context

Wanderly is a local desktop app requiring a packaged binary that runs on macOS and
Windows without a network connection. The app needs direct access to the local file
system (image copy-on-import, app data directory management) and SQLite via
`better-sqlite3`, which is a synchronous native Node.js module.

Two viable cross-platform desktop frameworks were evaluated: Electron and Tauri.

The team is a single developer working in TypeScript/JavaScript. Bundle size and
memory footprint are low priorities — this is a personal single-user tool, not a
mass-distributed commercial product.

---

## Decision

Use **Electron 33.x** as the app shell.

---

## What Was Debated

The choice was non-obvious because Tauri has meaningful advantages in bundle size
(~10MB vs ~150MB) and memory footprint. For a consumer product distributed at scale,
Tauri's lightweight profile would be a compelling reason to accept its constraints.

The decisive constraint for this project is the backend language. Tauri's backend is
Rust. All file system operations, SQLite access, and IPC business logic would require
either writing Rust or bridging everything through Tauri's JS IPC layer — the latter
of which effectively negates Tauri's performance advantages and adds significant
indirection. Electron's backend is Node.js, which means `better-sqlite3`, Sharp, and
the entire module layer are written in the same language as the renderer with no
additional runtime or context switch.

---

## Alternatives Considered

**Tauri** — rejected because:

- Backend is Rust; the solo developer has no Rust experience
- SQLite access via `better-sqlite3` (synchronous Node.js native module) is not
  directly usable in Tauri's Rust backend — would require a Rust SQLite crate instead,
  or a JS bridge adding latency and complexity
- File system operations (Sharp image processing, app data directory management) would
  require Rust implementations or JS bridge calls for every operation
- The 140MB bundle size difference is irrelevant for a personal single-user desktop tool
- Tauri's ecosystem is younger; fewer resources for the specific patterns this app needs

---

## Consequences

**Positive:**

- Entire codebase is TypeScript end-to-end — main process and renderer share types
- `better-sqlite3`, Sharp, and all Node.js tooling work without bridging or wrapping
- Electron's IPC model (contextBridge, ipcMain/ipcRenderer) maps cleanly to the
  module architecture defined in `architecture.md`
- Large ecosystem; mature documentation for all patterns used in this app
- electron-builder handles macOS and Windows packaging straightforwardly

**Negative:**

- Bundle size: ~150MB installer vs ~10MB for Tauri — accepted; irrelevant for this use case
- Memory footprint: Electron ships its own Chromium — accepted; single-user desktop tool
- `better-sqlite3` is a native module requiring rebuild against Electron's Node ABI
  via `electron-rebuild` — documented as Open Action Item B in `architecture.md`
- Electron's security posture requires discipline (nodeIntegration: false,
  contextIsolation: true) — enforced via BrowserWindow configuration; documented
  in the Security section of `architecture.md`
