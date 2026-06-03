# Travel App — Vision & Requirements

---

## Product Vision

A personal travel intelligence hub where years of curated research, planning, and experience become instantly accessible, searchable, and reusable — instead of buried in a notebook app.

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
              └── POI
```

**First-class content types:**
- **Country** — top-level container, tracks research status
- **Region** — organizational container within a country
- **City** — primary research unit; depth implied by content inside it
- **POI (Point of Interest)** — specific place, experience, restaurant, activity
- **Trip / Sample Trip** — itinerary referencing any Cities and POIs across the library
- **Budget** — flexible cost estimate, attachable to a Country, Location, or Trip
- **Tip / Lesson Learned** — scoped as General or Destination-specific; typed as Tip or Lesson Learned
- **Travel Window** — named planning event (e.g. "November 2027") containing shortlisted trips

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
- **FR1** — Create, edit, and delete any content type: Country, Region, City, POI, Trip, Budget, Tip, Travel Window. Budget is a standalone first-class type, attachable to a Country, Location, or Trip.
- **FR2** — All content starts as a stub (title only) and can be progressively enriched. No required fields beyond a title. Applies to all content types including Budget.
- **FR3** — Content types are interconnected. POIs belong to Cities, Cities belong to Regions, Regions belong to Countries. Trips reference any POIs, Cities, and Regions across the library regardless of geographic scope.

### Organization & Discovery
- **FR4** — Structured, contextual tags per content type. Tag categories are specific to each content type (e.g. POI Type, Price Range, Must-Do for POIs; Vibe, Activity Type, Budget Tier, Pace for Trips; Region/Continent, Entry Requirements, Language, Safety for Countries). Free-form tags and expanded categories are a planned future enhancement.
- **FR5a** — Global search across all content types, filterable by type, country, tag, and status.
- **FR5b** — In-context search/picker when creating or editing content, scoped intelligently to relevant content (e.g. building an Italy trip surfaces Italian POIs and Cities first).
- **FR6** — Manual status indicators at Country level (`Wishlist → Researching → Sampling → Planning`) and Trip level (`Sample → Planning → Ready → Completed`). Status is never auto-derived.

### Trip Planning
- **FR7** — Itinerary builder supports two modes: high-level city/region blocks with rough durations, and day-by-day detailed view. A trip can live in either state and transition fluidly between them as planning matures. City-block mode maps to `Sample` status; day-by-day maps to `Planning` and beyond.
- **FR8** — Budget is a flexible spreadsheet-like structure with customizable categories and line items. Supports two phases: Rough Estimate (high-level categories, single estimates per category, attached to Country or Sample Trip) and Detailed (expanded customizable categories, line items, estimated vs. actual columns, attached to a Planning or Ready Trip). Budget templates allow reuse of a working structure across trips. A summary snapshot (total, per person, per day) is auto-calculated and surfaced in presentation mode.

### Presentation Mode
- **FR9** — Any Travel Window can be flipped into a clean presentation mode designed for a destination decision conversation. Surfaces trip concept, sample itinerary, budget snapshot, and photos. Designed to support an honest pros/cons conversation — not a highlight reel. No app chrome, no editing UI.
- **FR10** — Presentation mode includes a side-by-side comparison view of 2-3 shortlisted trips within a Travel Window.
- **FR11** — Travel Window is a first-class concept. Contains a target travel date/window, 2-3 shortlisted trips referenced (not copied) from the master library, a comparison/presentation view, and a selection flow that dismisses unchosen options without deleting them from the master library.
- **FR12** — Presentation mode requires zero explanation to navigate. A non-user should be able to scroll through it intuitively without guidance.

### Migration
- No technical import from OneNote. Migration is handled manually by the primary user, used intentionally as a testing and refinement phase for the content model.

---

## Non-Functional Requirements

- **NFR1** — Search and filtering must feel instant. This is a reference tool used mid-planning and mid-travel.
- **NFR2** — Three distinct device contexts, each with its own UX consideration:
  - **Desktop/Laptop** — primary planning environment; full content creation, itinerary building, budget work, presentation mode
  - **Mobile Capture** — quick add of stubs, POIs, tips; minimal friction, fast
  - **Mobile Reference** — read-only in-destination use; optimized for finding specific information quickly (addresses, itinerary details, bookings)
- **NFR3** — Full library available offline on mobile. Photo optimization (text-only offline, photos on-demand) is a known future scaling lever.
- **NFR4** *(Post-MVP)* — Full data export in JSON format, preserving content relationships. Target for longevity and portability.

---

## Out of Scope (Current Version)
- Collaborative editing or sharing
- OneNote API connector or automated import
- Free-form tagging
- Social or community features
- Data export (post-MVP)
