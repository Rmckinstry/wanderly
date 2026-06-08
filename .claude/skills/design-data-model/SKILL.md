---
name: design-data-model
description: Use this skill when the user asks to design, model, or document a database schema, tables, collections, entities, relationships, or data persistence layer for either a relational or document database.
---

# Design Data Model

## Step 0 — Identify paradigm

Pick the branch based on the database selected in `select-technology-stack`:

- **Relational** — PostgreSQL, MySQL, SQL Server, etc. Schema-first, normalized by default, joins are cheap.
- **Document** — MongoDB, DocumentDB, Firestore, etc. Access-pattern-first, denormalized by default, joins are expensive or unavailable.

If the project uses **both** (e.g., Postgres as system of record + a document store for search/cache), run both branches for the respective tables/collections.

---

## Step 1 — Model the schema (paradigm-specific)

### Relational branch

For every table, emit full DDL. Vague descriptions ("stores user data") are defects.

```sql
CREATE TABLE <name> (
  <column>  <type>  <NULL/NOT NULL>  <DEFAULT ...>  <constraints>,
  ...
  PRIMARY KEY (...),
  FOREIGN KEY (...) REFERENCES ...
);
```

### Document branch

For every collection, emit three things:

1. **Schema validator** — JSON Schema outline with field types, required fields, enums:
   ```js
   { $jsonSchema: {
       bsonType: "object",
       required: ["customer_id", "created_at"],
       properties: { customer_id: { bsonType: "objectId" }, ... }
   } }
   ```
2. **Embed vs. reference decision** — for every relationship, state:
   - The choice (embed sub-document / store ObjectId reference)
   - The rationale (read-together frequency, write contention, unbounded growth risk)
   - Example: `order.lineItems → embed (always read with order, bounded ≤50 items)` vs `order.customer → reference (customer doc is large and shared)`
3. **Access patterns** — list the queries this shape is optimized for. If a query requires `$lookup` or app-side joins, justify why the shape doesn't co-locate the data.

---

## Step 2 — Specify every index with rationale

Universal rule: every index must name the query it serves. Indexes without a named query are speculative — drop them or justify them.

**Relational:**
```
INDEX idx_orders_customer_created ON orders (customer_id, created_at DESC)
  Serves: "customer order history page, newest first"
```

**Document:**
```
db.orders.createIndex({ customer_id: 1, created_at: -1 })
  Serves: "customer order history page, newest first"
```

---

## Step 3 — Declare integrity layer

For every cross-module relationship, state where integrity is enforced.

- **Relational:** DB layer (FK constraint) — preferred when modules share a database. Application layer (event-driven, eventual consistency) — required when modules own separate databases.
- **Document:** Application layer **always** (FKs do not exist). Name the service or module that owns the consistency contract and the recovery mechanism for orphaned references.

Mixing enforcement layers silently causes bugs. Document the choice.

---

## Step 4 — Encode compliance-driven design

Surface these explicitly when they apply.

| Concern | Relational mechanism | Document mechanism |
|---|---|---|
| **Append-only** (e.g., audit log) | Revoke UPDATE/DELETE grants; add trigger blocking modifications | Deny `update` / `delete` actions in role/access rules; write-once collection convention |
| **Encryption at rest** | Column-level (pgcrypto) vs. tablespace-level | Field-level (Client-Side Field Level Encryption) vs. collection/cluster-level |
| **Role-restricted access** | DB roles + GRANTs per table | Built-in role-based access per collection / action |
| **Soft vs. hard delete** | Justify per table; flag any compliance mandate | Justify per collection; flag any compliance mandate |

---

## Step 5 — Document migration strategy

State the tool, naming convention, and rollback policy.

- **Relational:** Flyway, Liquibase, or framework-native (e.g., Alembic, ActiveRecord). Note tables large enough to need zero-downtime patterns (online DDL, dual-write).
- **Document:** Schema-version field on each document + idempotent migration scripts, OR Mongo's `collMod` for validator changes. Note collections large enough to need batched background migration.

---

## Step 6 — Version-pin the database

State the exact major version (e.g., `PostgreSQL 17`, `MongoDB 7.0`). This must match `select-technology-stack`'s version table.
