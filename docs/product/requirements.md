# Travel App — Vision & Requirements

---

## Product Vision

A personal travel intelligence hub where years of curated research, planning, and experience become instantly accessible, searchable, and reusable — instead of buried in a notebook app.

---

## Deployment Context

**This version is a local desktop web app.** It runs on your machine (localhost), uses local storage (SQLite or flat-file), requires no internet connection to function, and is single-user with no authentication. All data lives on your computer.

Cloud hosting, multi-device sync, authentication, and mobile support are explicitly deferred. See [Deferred Scope](#deferred-scope) at the end of this document.

---

## Problem Statement

Avid travelers accumulate rich, hard-won knowledge across trip research, budgeting, POIs, and itineraries — but OneNote and similar tools turn that knowledge into a graveyard. Content is hard to search across, impossible to cross-reference, and painful to reuse when planning a new trip. Existing research decays instead of compounding.

---

## Solution Hypothesis

A structured travel knowledge base app purpose-built around travel content types — with first-class tagging, smart search, templated content creation, and a clear data model (Countries → Regions → Cities → POIs, Trips, Budgets, Tips) that makes everything interconnectable and reusable. A Travel Window concept formalizes the shortlisting and decision workflow, and a presentation mode supports honest destination decision conversations with a travel partner.

---

## Content Model

The core hierarchy:

```
Country
  └── Region
        └── City
              └── POI (can also attach directly to Country or Region)
```

**First-class content types:**
- **Country** — top-level container, tracks research status
- **Region** — organizational container within a country
- **City** — primary research unit; depth implied by content inside it
- **POI (Point of Interest)** — specific place, experience, restaurant, activity. Can attach to a Country, Region, or City — city-level attachment is not required
- **Trip** — itinerary referencing any Countries, Regions, Cities, and POIs across the library. Starts as a stub (status: `Sample`) and matures progressively. There is no separate "Sample Trip" type — maturity is expressed through status.
- **Budget** — flexible cost estimate, attachable to a Country, Region, or Trip
- **Tip / Lesson Learned** — scoped as General or Destination-specific; typed as Tip or Lesson Learned. Both types can be General or Destination-specific.
- **Travel Window** — named planning event (e.g. "November 2027") containing shortlisted trips. Persists permanently as a historical record.

---

## Status Model

**Country status** — tracks research investment (set manually):
`Wishlist` → `Researching` → `Sampling` → `Planning`

**Trip status** — tracks lifecycle (set manually):
`Sample` → `Planning` → `Ready` → `Completed`

Regions and Cities do not have formal status — depth is implied by content volume.

---

## Functional Requirements

### Content Management
- **FR1** — Create, edit, and delete any content type: Country, Region, City, POI, Trip, Budget, Tip, Travel Window. Budget is a standalone first-class type, attachable to a Country, Region, or Trip.
- **FR2** — All content starts as a stub (title only) and can be progressively enriched. No required fields beyond a title. Applies to all content types including Budget.
- **FR3** — Content types are interconnected. POIs can attach to a Country, Region, or City — city-level is not required. Cities belong to Regions or Countries. Regions belong to Countries. Trips reference any Countries, Regions, Cities, and POIs across the library regardless of geographic scope. A Trip is not constrained to a single country or region.

### Organization & Discovery
- **FR4** — Structured, contextual tags per content type. Tag categories are specific to each content type:
  - **Country:** Region/Continent, Entry Requirements, Language Barrier, Safety
  - **Region:** Region Type (Coastal, Mountain, Urban, Rural, Island), Safety, Language Barrier
  - **City:** City Type (Capital, Historic, Beach Town, Mountain Town, Village, Resort), Size (Major City, Mid-Size, Small Town), Vibe
  - **POI:** POI Type (Restaurant, Hotel, Museum, Beach, Hike, Market, Bar, Viewpoint, Day Trip), Price Range (Free, $, $$, $$$), Must-Do Status (Must Do, Nice to Have, Skip if Short on Time)
  - **Trip:** Vibe (Relaxing, Adventure, Cultural, Romantic, Family, Party/Nightlife, Off the Beaten Path), Activity Type (Beach, Hiking, Food & Drink, History & Architecture, Wildlife, Water Sports, Skiing, City Exploration, Road Trip), Budget Tier (Budget, Mid-Range, Luxury), Trip Pace (Slow Travel, Packed Itinerary, Weekend Getaway, Long Haul), Best Season (Spring, Summer, Fall, Winter, Year Round, Avoid Crowds)

  Free-form tags and expanded categories are a planned P2 enhancement.
- **FR5a** — Global search across all content types, filterable by type, country, tag, and status.
- **FR5b** — In-context search applies in two places: (1) the trip builder, scoped by default to countries referenced in the trip with an option to expand to the full library; (2) within country, region, or city records, scoped to all content nested under that record. Ctrl+F triggers in-context search — final behavior (app intercept vs. browser native) to be determined by UX best practice research.
- **FR6** — Manual status indicators at Country level (`Wishlist → Researching → Sampling → Planning`) and Trip level (`Sample → Planning → Ready → Completed`). Status is never auto-derived.

### Trip Planning
- **FR7** — Itinerary builder supports two modes: high-level location blocks with rough durations, and day-by-day detailed view. Location blocks can be set at any geographic level — Country, Region, or City. A trip can mix blocks at different levels and transition fluidly from location-block mode to day-by-day as planning matures. A trip can exist in a mixed state with some blocks expanded into days and others not. Location-block mode maps to `Sample` status; day-by-day maps to `Planning` and beyond.
- **FR8** — Budget is a single flexible content type with no Rough vs. Detailed distinction. It starts as simple as a name and grows in detail over time. Structure: customizable categories containing line items, each with three columns — Budgeted/Estimate, Actual, Difference. Visual indicators (Under, Over, Close) appear at line item, category, and summary levels. Close threshold is user-configurable, defaulting to 10%. A summary snapshot (total budgeted, total actual, total difference, cost per person, cost per day) is auto-calculated and surfaced in presentation mode. All amounts are in USD. Budget templates allow reuse of category structure across trips without carrying over amounts.

### Presentation Mode
- **FR9** — Any Travel Window can be flipped into presentation mode designed for a destination decision conversation. Supports two views: individual trip view (one trip at a time with next/previous navigation) and comparison view (all shortlisted trips side by side). Full trip detail is accessible from within presentation mode without exiting. Presentation mode is read-only by default. Surfaces trip concept, sample itinerary, budget snapshot, and photos. Designed to support an honest decision conversation — not a highlight reel. No app chrome or editing UI visible. Fully navigable via keyboard (arrow keys, Enter, Backspace, Escape, Tab).
- **FR10** — Presentation mode includes a side-by-side comparison view of 2-3 shortlisted trips within a Travel Window.
- **FR11** — Travel Window is a first-class concept. Contains a target travel date/window and 2-3 shortlisted trips referenced (not copied) from the master library. Trips are fully editable from within the Travel Window view — edits reflect immediately in the master library. A chosen trip can be flagged within the Travel Window; unchosen trips remain visible and intact. Travel Windows persist permanently as historical records and are never automatically deleted or hidden. When a trip within a Travel Window is marked `Completed`, a point-in-time snapshot of that trip is automatically captured and stored — preserving the full trip state (location blocks, days, POIs, notes, budget, photos) at that moment. Future edits to the live trip do not alter the snapshot. Users can toggle between snapshot view and live view within a Travel Window.
- **FR12** — Presentation mode requires zero explanation to navigate. A non-user should be able to scroll through it intuitively without guidance.

### Migration
- No technical import from OneNote. Migration is handled manually by the primary user, used intentionally as a testing and refinement phase for the content model.

### Content Editing
- **FR13** — All free-form notes fields across every content type support rich text editing: bold, italic, underline, bullet points, numbered lists, and hyperlinks. Rich text formatting is preserved on desktop and renders correctly in presentation mode. Pasting from external sources (OneNote, browser) preserves basic formatting where possible — if not, plain text is pasted without silent data corruption.

---

## Non-Functional Requirements

- **NFR1** — Search and filtering must feel instant. This is a reference tool used mid-planning.
- **NFR2** — Desktop/laptop browser is the only supported environment (P0). The app is not required to be responsive or functional on mobile. See [Deferred Scope](#deferred-scope) for mobile plans.
- **NFR3** — Single-user, no authentication required. The app assumes it is running on the owner's local machine.
- **NFR4** — Local data storage only. Data persists via SQLite or a structured flat-file format on the local filesystem. No cloud database, no sync service.
- **NFR5** *(P2)* — Full data export in JSON format, preserving content relationships. Lower urgency in a local-first context (data is already portable) but worth building to avoid future lock-in.

---

## Out of Scope (Current Version)
- Collaborative editing or sharing
- OneNote API connector or automated import
- Free-form tagging (planned P2 enhancement)
- Custom tag categories and values (planned P2 enhancement)
- Social or community features
- Mobile app or mobile-optimized layout
- Cloud hosting or remote access
- Multi-device sync
- User authentication
- Data export — planned P2 in JSON format

---

## Deferred Scope

The following capabilities are explicitly out of scope for the local version and documented here for future expansion planning.

### Cloud Hosting & Authentication
**What it is:** Moving the app from localhost to a hosted URL, with a login system so the data is accessible from anywhere.
**Why deferred:** Adds infrastructure complexity (hosting, auth provider, secrets management) with no benefit for a single-user local tool.
**What it unlocks when built:** Remote access, sharing, and the ability to support additional users.
**Rough prerequisites:** Backend API layer, auth provider integration (e.g. Auth0 or Supabase Auth), cloud database migration from local SQLite.

### Multi-Device Sync
**What it is:** Changes made on one device automatically appear on another — phone, tablet, second laptop.
**Why deferred:** Requires a cloud backend and conflict resolution logic. Not applicable to a single local machine.
**What it unlocks when built:** Seamless switching between devices mid-planning or mid-travel.
**Rough prerequisites:** Cloud hosting + auth (above), sync engine, conflict resolution strategy (last-write-wins vs. merge).

### Mobile App (Capture, Reference, Offline)
**What it is:** Three mobile use cases — quick stub capture on the go, in-destination reference of itinerary and POI details, full offline library access.
**Why deferred:** Requires responsive layout design, mobile-specific UX patterns, offline-first architecture (service workers or native app wrapper), and sync infrastructure.
**What it unlocks when built:** The full travel lifecycle — research and planning on desktop, reference and capture while actually traveling.
**Rough prerequisites:** Multi-device sync (above), responsive UI overhaul or native app wrapper, offline storage strategy (text-first with on-demand photos).
**Stories on hold:** US-044, US-045, US-046, US-047, US-048, US-049.