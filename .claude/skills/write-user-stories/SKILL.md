---
name: write-user-stories
description: Use this skill when the user asks to write, draft, create, refine, or expand user stories or acceptance criteria — whether a single story, a batch of stories for a feature, refinement of existing stories, or stories spanning multiple features.
---

# Write User Stories

Flexible user-story authoring. Dispatch on mode in Step 0, then apply the shared format rules.

## Step 0 — Pick mode

| Mode | Trigger phrasing | What to produce |
|---|---|---|
| **Single** | "write a story for X", "draft a user story about Y" | One story with full acceptance criteria + priority |
| **Batch** | "draft user stories for [feature]", "write the stories for the onboarding flow" | A coherent set (3–7 typically), grouped by persona, covering the feature end-to-end |
| **Refine** | "refine story X", "tighten acceptance criteria on Y", "improve this story: [text]" | Improved version of the input story — sharper job-story wording, missing edge cases surfaced, priority validated |
| **Expand** | "write stories across these features", "author stories for the whole roadmap" | Stories across multiple features, ensuring every persona has coverage and no feature is unspec'd |

If the mode is ambiguous, default to **Single** and ask the user whether they want more.

---

## Step 1 — Apply the story format

Use **job-story format** by default:

> *When [situation], I want to [action], so that [outcome].*

Classic `As a [persona], I want to [action], so that I can [benefit]` is acceptable when explicitly requested (e.g., the project already uses it).

---

## Step 2 — Apply the spec block

Each story gets:

```
Story: [Job-story sentence]
Persona: [Name from personas.md if it exists, else descriptive role]
Acceptance Criteria:
  - Given [context], when [action], then [outcome]
  - Given [edge-case context], when [action], then [outcome]
Priority: P0 | P1 | P2 — [one-line justification]
Dependencies: [Other story IDs, or "none"]
```

Optional fields, include only when applicable:
- `Story ID:` — only if the project uses IDs (e.g., `STORY-01`, `FEAT-04`)
- `Technical Constraints:` — only if a known limitation shapes the story
- `UX Considerations:` — only if a specific interaction pattern matters

---

## Step 3 — Apply mode-specific structure

**Single:** Emit one spec block. Done.

**Batch:** Group by persona. Within each persona, order stories by user journey (discovery → activation → core use → edge). Emit a one-line feature summary at the top.

**Refine:** Show the original story, then the improved version, then a brief "what changed and why" note (1–3 bullets). Don't silently rewrite — make the diff visible so the user can accept or push back.

**Expand:** Produce a coverage matrix first (persona × feature → story count), then the stories themselves grouped by feature. Flag any persona/feature cell with zero coverage as a gap to address.

---

## Step 4 — Output for integration

Output is structured to append cleanly into `docs/product/user-stories.md`. Use markdown headings that match the file's existing grouping convention (by persona or by epic — pick whichever the existing file uses; default to by-persona if no file exists yet).

If the user hasn't asked you to write to a file, just return the formatted text in the response — they'll paste it themselves.
