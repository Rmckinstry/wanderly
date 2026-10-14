---
title: Recovery and first run
status: ready for build
stories: [NFR6]
last-updated: 2026-10-13
---

# Recovery and first run

Full-window screens without the app shell. Wireframes: Recovery, FirstRun, FirstRunRestore. Decision D-35 (empty first library).

---

## Recovery

Screen:      Recovery — library failed its startup check
Serves:      NFR6, handoff 21
Components:  Button, SegmentedControl-style radio list (restore points)

States:
- integrity failure — "Wanderly" wordmark; "Your library couldn't be opened"; "The library file failed its check when Wanderly started, so nothing was opened or changed. Pick a copy to restore — the newest is selected." Radio list of restore points (When · source; counts; "Newest" on the first). Primary "Restore and open ↵", "Restore from another folder…", quiet "Quit". Note: "The damaged file is kept beside the library as wanderly-damaged-2026-10-09.db — nothing is deleted. Show details" (details disclose the check's error text).
- failed migration (D-50) — "Wanderly couldn't update your library" / "It's safe and unchanged. This version of Wanderly can't open it until the update works — reinstall the previous version to keep using your library until a fix is out." Primary "Quit"; secondary "Copy details" (copies the error, app version and schema versions for a bug report; Toast "Details copied"); quiet "Show library file" (reveals it in Finder / Explorer). No restore list — every snapshot is on the older format too.
- no restore points — "No snapshots were found on this computer." + "Restore from another folder…" + "Quit".
- restoring — primary shows "Restoring…" disabled; then the app restarts.

Keyboard: focus starts on the selected restore point; Up/Down change the choice; Enter restores; Tab reaches the other buttons. Failed migration: focus starts on "Quit"; Tab moves to "Copy details" and "Show library file".

---

## First run

Screen:      First run — fresh install
Serves:      NFR6, D-35
Components:  Button, card-style choices

States: "Welcome. How do you want to start?" Two choices: "Start a new library — An empty library. Add your first country and go from there." (primary "Start fresh ↵") and "Restore from a backup — Moving to a new computer? Point Wanderly at your backup folder — library and photos come across." ("Choose backup folder…").

Interactions: Start fresh → Countries list in its empty state. Choose backup folder… → system folder picker → First run restore.

Keyboard: Tab between the two buttons; Enter on the focused one; the first is focused on open.

Edge cases: D-35 no seeded sample content; tutorial deferred (handoff 20).

---

## First run — restore found

Screen:      First run — restore from a backup folder
Serves:      NFR6, handoff 21
Components:  Field (read-only path), Button

States:
- found — path + "Change…"; a summary box "Backup from 9 Oct 2026, 21:40 · Includes photos · 23 countries · 412 POIs · 11 trips · 1.8 GB"; primary "Restore and open ↵", quiet "Back"; note "This folder then becomes your backup folder on this computer too. You can change it in Settings."
- no backup in folder — "No Wanderly backup in this folder. Pick the folder named Wanderly Backup, or the drive that holds it." (Wanderly looks one level down.)
- newer version — "This backup was made by a newer version. Update Wanderly, then try again." No restore button.
- restoring, copying photos (D-51) — "Restoring… copying photos (1.2 of 1.8 GB)" with a determinate bar and a secondary "Cancel — nothing has changed yet". Cancel deletes the partly copied files and returns to this screen with the folder still chosen.
- restoring, finishing — "Finishing…" while the library file is swapped in (a few seconds); no Cancel; then the app relaunches.

Keyboard: Enter restores; Esc goes Back. While copying, focus sits on Cancel (Esc also cancels); while finishing, keys do nothing.
