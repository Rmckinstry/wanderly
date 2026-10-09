# ADR-002: Drizzle ORM over Prisma

**Status:** Accepted  
**Date:** 2026-04-18  
**Applies to:** Database layer — all phases

---

## Context

Wanderly uses SQLite via `better-sqlite3` (a synchronous native Node.js module) as
its database. An ORM or query builder is needed to provide type-safe database access,
schema management, and migration tooling.

Two mature TypeScript ORM options were evaluated: Drizzle ORM and Prisma. Both
support SQLite and provide strong TypeScript inference.

Note: An earlier version of this ADR cited Prisma's incompatibility with
`better-sqlite3`'s synchronous driver as the deciding factor. This was inaccurate —
`@prisma/adapter-better-sqlite3` exists, is actively maintained, and resolves that
compatibility issue. The rationale has been corrected to reflect the actual basis
for the decision.

---

## Decision

Use **Drizzle ORM** with **Drizzle Kit** for migrations. Versions are in the Tech Stack
table in `architecture.md`.

---

## What Was Debated

Both Drizzle and Prisma are viable for this project after `@prisma/adapter-better-sqlite3`
resolved the prior sync driver incompatibility. The choice comes down to architectural
weight and maintenance burden for a solo developer building a long-lived personal tool.

The decisive factors are:

1. **No generated client.** Prisma generates a client binary at build time and requires
   `prisma generate` to be run after schema changes. In an Electron app this adds a build
   step and a binary that must be managed during packaging. Drizzle has no generated client —
   queries are written directly against the schema in TypeScript.

2. **SQL-close query syntax.** Drizzle's query API closely mirrors SQL, which makes
   the queries easier to reason about during long-term solo maintenance. Prisma's
   abstraction is higher-level, which is a benefit in team settings but introduces
   opacity for a developer who prefers to understand exactly what queries are being sent.

3. **Lighter footprint.** Drizzle adds less to the packaged Electron app than Prisma's
   client and engine.

---

## Alternatives Considered

**Prisma 7.x with @prisma/adapter-better-sqlite3** — rejected because:

- Requires a generated client and `prisma generate` step in the build pipeline —
  additional complexity in Electron packaging with no compensating benefit
- Higher abstraction level makes it harder to audit exact SQL during solo maintenance
- Heavier overall footprint in the packaged binary
- Prisma's strengths (team collaboration, relation graph, Prisma Studio) are
  not relevant for a single-user local app

**Raw SQL with better-sqlite3 directly** — not evaluated as a primary option; lack of
type safety and schema management would create long-term maintenance burden for a
complex schema with 15+ tables.

---

## Consequences

**Positive:**

- No generated client — schema changes are reflected immediately without a build step
- Type-safe queries written directly in TypeScript against the Drizzle schema
- Drizzle Kit generates versioned migration files cleanly; Drizzle ORM's runtime migrator
  applies them on app startup
- Lighter Electron bundle — no generated Prisma client to package
- SQL-close syntax; queries are auditable and predictable during solo maintenance

**Negative:**

- Drizzle's ecosystem is smaller than Prisma's — fewer community resources and examples
- Drizzle Studio (the visual DB browser equivalent of Prisma Studio) is less mature
- If the project grows to require a team or moves to a hosted database, Prisma's
  higher-level abstractions and relation graph may become more valuable — migration
  to Prisma at that point would require rewriting the repository layer
- `@prisma/adapter-better-sqlite3` would have been viable; this decision is a preference
  call, not a hard technical requirement

---

## Revision History

- **2026-10-08** — version numbers removed from the Decision (the Tech Stack table in `architecture.md` is the single source); the last reference to a "Prisma engine binary" corrected, completing the rationale correction described in Context. The decision itself is unchanged.
