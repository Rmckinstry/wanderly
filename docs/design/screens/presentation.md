---
title: Presentation mode
status: ready for build
stories: [US-037, US-038, US-039, US-040, US-041, US-042, US-043]
last-updated: 2026-10-09
---

# Presentation mode

Hero screen (D-01): the only screen the Co-Decider sees. It must need zero explanation (FR12) and be drivable with arrow keys alone. Follows the palette and theme chosen in Settings (D-05).

Wireframes: PresSummary, PresDetail, PresCompare, PresSingle. Hi-fi: Presentation (Paper light, Paper dark, Atlas light).

---

## Summary (one trip)

Screen:      Presentation — summary
Serves:      US-037, US-038, US-040, US-041, US-042, D-14, D-43
Components:  PresentationShell, RouteStrip (`strip`, `highlights`), BudgetSnapshot (`ledger`, `split`), PhotoGrid (presentation hero + 3), TagChip, Button (`lg`)

States:
- default — no app chrome, no nav, no edit controls. Header: "November 2027 · Option 1 of 3" (left, `small`); "Shortcuts ?" and "Exit Esc" quiet buttons (right). Left column: trip title in `trip-title` (64px Newsreader 400), the concept in `lede` (18/1.6, max 620px), "The route" with RouteStrip `strip` (segments ∝ days; stops alternate `accent` / `cat-5`; travel days hatched with `border`) and labels under each segment; "Highlights by stop" (RouteStrip `highlights`: one column per stop with a 3px top rule in its segment colour, top 2 POIs Must-do first, "+N more places"). Right column (440px): hero photo 256px on its `mat`, three 72px thumbnails, facts ledger (Shared estimate `money-lg`, Works out to $/day, Feels like: tag chips), "Where the money goes" split bar + labelled legend. Footer: option dots (current filled `accent`, others outlined) with trip names, "Full trip detail ↵", "Compare all three C", primary "Next: Portugal →".
- first trip — there is no Previous button anywhere (← simply does nothing on the first trip). Last trip — "Next" is removed and "Compare all three C" becomes the primary (accent) button, so the natural end of the walk-through is the comparison.
- no photos — hero area shows a neutral `mat` block with the trip's first letter in `text-muted`; thumbnails omitted.
- no budget — ledger reads "Budget not yet estimated" in place of the three figures; split bar omitted.
- per day incomplete — "Works out to" shows "Needs every stop's length".
- no tags on this trip — "Feels like" row hidden.
- trip with only location blocks (no days) — highlights come only from POIs placed on the trip's days, never from the library; a stop with none shows its duration and "No places picked yet" (`text-muted`). US-038: the blocks themselves are the itinerary.
- single trip in the window (US-037) — see "Single trip" below.

Interactions: buttons as labelled; clicking an option dot jumps to that trip.

Keyboard:
- → / ← next / previous trip; ↵ full trip detail; C comparison (2+ trips); ? shortcuts sheet; Esc exits to exactly where the Travel Window was.
- Tab cycles: Shortcuts, Exit, then footer controls (dots, Full trip detail, Compare, Next). Focus starts on the primary (Next).
- Keys are inert while the shortcuts sheet is open, except Esc (closes the sheet).

Motion: entering presentation fades in over `duration-slow`; changing trips cross-fades `duration-base`; instant with reduced motion.

Edge cases:
- US-037 accessible regardless of count → Present works with 1+ trips; with 0 trips the Present button is disabled with tooltip "Add a trip first".
- US-037 no data or status changes; Esc returns exactly where you were → re-entering starts at the beginning (US-042).
- US-038 read-only, simple navigation, placeholders, rich text renders → as above.
- US-040 snapshot: total, per day, Under/Close/Over once actuals exist (Variance on the ledger's estimate line); per person removed (D-28); "Budget not yet estimated"; same position in comparison; snapshot only; primary budget with "1 of 2 budgets" note in `micro` under the estimate.
- US-041 photos supporting, hero = designated else first available from the trip or its linked places (handoff 12).

Handoff notes: side padding `space-14`; header and footer `space-6` vertical; option dots 8px; hero photo `radius-sm` on a `mat` frame with `space-2` inset (critique nit — confirm or drop the mat).

---

## Full trip detail

Screen:      Presentation — full trip detail
Serves:      US-043, D-11, D-22
Components:  PresentationShell (full-detail header, section rail, footer hints), ItineraryBlock (`read`), PoiCard (`read`), Banner, BudgetTable (read-only), PhotoGrid

States:
- default — header "← Summary ⌫" + title in `trip-title-sm`; Shortcuts / Exit. Left rail: 1 Itinerary (with stops listed beneath), 2 Places, 3 Tips, 4 Budget, 5 Photos — current section `surface-active` with its number in `accent`. Content scrolls: travel blocks as read rows, each stop's heading with "Days 4–9 · 6 days", stop notes, DayRows with PoiCards `read`, empty days read "Free day". Footer hints: "↑↓ scroll · 1–5 jump to section · ⌫ back to summary · ←→ other trips · Read-only · exit presentation to edit".
- Budget section — full BudgetTable, read-only (US-043 complete budget).
- preserved record — Banner under the title "Historical record · deleted 3 Mar 2026"; no links.

Keyboard: ↑ / ↓ scroll; 1–5 jump to section; ⌫ back to the summary (or to comparison when entered from comparison, US-043); ← → previous/next trip, staying in full detail; Esc exits presentation.

Edge cases:
- US-043 reachable from summary and comparison; read-only; seamless return; returns to comparison when entered from there; rich text renders → as above.

---

## Comparison

Screen:      Presentation — comparison (2+ trips)
Serves:      US-039, US-040, FR10, D-02
Components:  PresentationShell, ComparisonColumn, RouteStrip (`strip`, compact), TagChip

States:
- default — header "November 2027 · Comparing 3 options". Row labels at the left (The idea · Route · Cost · Feels like); one ComparisonColumn per trip: hero photo, title (`trip-title-sm`), concept (2 lines), compact route strip + route text + days, estimate (`money-md`) + per day, tags. Focused column: 2px `text` outline on `surface`. Footer hints "←→ move between options · ↵ full detail of the focused option · C back to one at a time".
- missing field — "Budget not yet estimated", "No tags yet" — never a hidden row; the Feels like row is hidden only if no trip has tags (US-039).
- chosen trip (past windows) — `accent` top rule + "Chosen".

Keyboard: ← → move focus between columns; ↵ opens that trip's full detail; C returns to the summary of the focused trip; Esc exits.

Edge cases:
- US-039 only with 2+ trips → C and "Compare all" absent with one trip (D-02).
- US-039 same fields, missing shown as not yet added, tags row rule → as above.

---

## Single trip and the shortcuts sheet

Screen:      Presentation — single trip; Shortcuts sheet
Serves:      US-037, US-042, D-02
Components:  PresentationShell, FormDialog-style sheet (read-only list)

States:
- single trip — header "February 2027 · One option"; footer shows "The only trip in this window" and primary "Full trip detail ↵"; no dots, no Compare, no Next.
- shortcuts sheet (?) — dialog "Keyboard shortcuts" with two columns, Summary (← → next/previous, ↵ full detail, C compare all, Esc exit) and Full detail (↑↓ scroll, 1–5 jump to section, ⌫ back to summary, ? this sheet). Keys that don't apply now are shown at 45% opacity, not hidden.

Keyboard: ? opens; Esc or ? closes and returns focus to where it was; the sheet traps focus.

Edge cases:
- US-042 exit control always visible but unobtrusive → header "Exit Esc".
- US-042 shortcut reference without disrupting → sheet over the slide.
- US-042 accidental exit → re-entering starts at the beginning.
- US-042 keys don't conflict with OS defaults → no ⌘/Ctrl combos inside presentation; Backspace is captured (no browser-style back).

Handoff notes: all presentation type sizes are fixed (no scaling with window size in v1); on very wide windows the content keeps `space-14` side padding and grows its columns proportionally.
