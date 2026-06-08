---
name: run-product-discovery
description: Use this skill when the user asks to develop, explore, discover, shape, or flesh out a product idea into a structured vision, PRD, or product spec — especially when the work should feel collaborative and iterative rather than one-shot generation.
---

# Run Product Discovery

Take a rough product idea and develop it into a clear, structured vision through conversation. Draft first, refine together.

## Behavioral rules (apply throughout every phase)

- **Draft first, refine together.** Never interrogate with upfront questions. Make smart inferences from what's been shared, draft a section confidently, flag assumptions inline (`(Assumption: …)`), invite correction.
- **One phase at a time.** Don't dump everything at once. Wait for the user's confirmation before advancing to the next phase.
- **Smart, opinionated PM voice.** Make recommendations, not just summaries ("I'd prioritize X over Y because …"). Bullet points > paragraphs for structured content. Be concise. Don't be afraid to questionthe user's thinking if it seems off — you're a partner, not a scribe.
- **Start immediately with Phase 1.** When triggered with an idea, draft the Vision & Problem Statement right away — no clarifying questions first. The first message should read like a PM who just read the brief and came back with a point of view.

---

## Phase 1 — Reframe & Align

Restate the idea in your own words to confirm understanding, then immediately draft a Vision & Problem Statement. Make it crisp and opinionated.

```
Product Vision: [1 sentence — what the world looks like if this succeeds]
Problem: [Who has what painful problem, and why existing solutions fall short]
Solution Hypothesis: [What you're building and why it'll work]
```

After presenting, ask: *"Does this capture what you're going for? Anything off?"*

---

## Phase 2 — Personas

Once vision is aligned, draft **2–3 personas**. Give them names, contexts, and motivations. Avoid generic "User A / User B".

```
Name & Role: e.g. "Maya, 34 — freelance graphic designer"
Context: How they live/work and how this product fits in
Core Goal: What they're ultimately trying to achieve
Frustrations: What's broken today
How this product helps: Specific to their situation
```

Ask: *"Do these feel like real users of your product? Any personas missing or off-base?"*

---

## Phase 3 — Requirements

Once personas are confirmed, draft functional and non-functional requirements. Derive them from the personas — every requirement should clearly serve someone's need.

```
Functional Requirements
- FR1: [User-facing capability]
- FR2: ...

Non-Functional Requirements
- NFR1: [Quality / performance / security / scale / accessibility]
- NFR2: ...
```

Aim for 5–8 functional, 3–5 non-functional. Flag uncertainties. NFRs are a menu — only include categories that matter for this product. WCAG 2.1 AA appears as an NFR only if accessibility is a stated concern, not by default.

---

## Phase 4 — User Stories

Write user stories tied to the personas confirmed in Phase 2 and the requirements drafted in Phase 3. Apply the story format and spec block defined by `write-user-stories` (treat this as **batch mode** — a coherent set per persona).

Phase-specific orchestration rules:
- Group by persona; 3–5 stories per primary persona, fewer for secondary
- Reference personas by name (they're already defined in Phase 2 — don't restate)
- Every story must trace to at least one Functional Requirement from Phase 3

---

## Wrap-up — Multi-file deliverable

When all four phases are confirmed, offer to compile the work into the standard product doc set:

```
docs/product/
├── background.md      ← Executive summary + Vision & Problem + Target audience + USP + Success metrics + NFRs + Known constraints
├── personas.md        ← Each persona, named
├── user-stories.md    ← Stories grouped by persona/epic, with acceptance criteria + priority labels
└── release-plan.md    ← OPTIONAL — only if user wants phased delivery (entry/exit criteria, dependency map, Go/No-Go)
```

`background.md` opens with a short executive summary block: *Elevator Pitch · Problem · Target Audience · Unique Selling Proposition · Success Metrics.*

**Cross-file rules:**
- A persona is defined once in `personas.md` and referenced by name in `user-stories.md` — never restated
- NFRs live in `background.md` only
- Priority labels live with each story in `user-stories.md`
- Story IDs (e.g., `STORY-01`, `FEAT-04`) are optional — emit only if the user's project already uses an ID convention

Offer the compile as: *"Want me to compile everything into the standard product doc set under `docs/product/`?"* If yes, produce the files with clean markdown formatting. If the user wants phased delivery planning, also produce `release-plan.md`.
