# Wanderly — Documentation

Wanderly is a local desktop app (Electron, macOS and Windows) for keeping travel research,
trip plans and budgets in one searchable library. These documents define the product and
its architecture. There is no code yet; they are the specification the build follows.

## Reading order

| # | Document | What it answers |
| - | -------- | --------------- |
| 1 | [product/requirements.md](product/requirements.md) | Vision, content model, functional and non-functional requirements, deferred scope |
| 2 | [product/personas.md](product/personas.md) | Who the app is for |
| 3 | [product/user-stories.md](product/user-stories.md) | Behaviour in detail: acceptance criteria and edge cases per story |
| 4 | [architecture/architecture.md](architecture/architecture.md) | Tech stack, modules, cross-cutting patterns, security, performance, open items |
| 5 | [architecture/data-model.md](architecture/data-model.md) | SQLite schema, indexes, integrity rules, delete behaviour |
| 6 | [architecture/api-contracts.md](architecture/api-contracts.md) | Every renderer↔main operation: channel, payload, result, errors, side effects |
| 7 | [architecture/adrs/](architecture/adrs/) | Decision records for the contested choices (Electron over Tauri, Drizzle over Prisma) |

## Source of truth

Each fact lives in one place. Other documents link to it and do not restate it.

| Fact | Lives in |
| ---- | -------- |
| What the product must do, and priorities | `product/requirements.md` |
| Exact behaviour and edge cases | `product/user-stories.md` |
| Library and framework versions | Tech Stack table in `architecture/architecture.md` |
| Module boundaries and who may call whom | Modules section of `architecture/architecture.md` |
| Tables, columns, constraints, what a delete does | `architecture/data-model.md` |
| Channel names, payloads, error codes | `architecture/api-contracts.md` |
| Items that block implementation | Open Action Items in `architecture/architecture.md` |

When two documents disagree, that is a defect in the docs: fix both rather than picking
one silently. For behaviour, the user story states the intent and the architecture
documents must be brought into line with it, or the story changed deliberately.

## Conventions

- Story IDs (`US-012`) and requirement IDs (`FR7`, `NFR4`) are stable. A retired ID is
  left in place with a note and never reused.
- API operations are written in REST style for readability but are Electron IPC calls.
  The Global Conventions section of `api-contracts.md` explains how to read them.
- A decision gets an ADR only when real alternatives were weighed and someone might
  reasonably want to reverse it later. Everything else is documented where it applies.
