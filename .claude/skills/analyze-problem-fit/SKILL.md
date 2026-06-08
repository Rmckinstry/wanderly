---
name: analyze-problem-fit
description: Use this skill when the user asks to validate a product idea, sanity-check a problem framing, assess whether something is worth building, or evaluate problem-solution fit — without running a full phased discovery.
---

# Analyze Problem Fit

Quick standalone validation of a product idea. Produces a single short analysis, not a full doc set. If the user wants end-to-end discovery, use `run-product-discovery` instead.

## Step 1 — Problem analysis

State, in 2–3 sentences:
- The specific problem this idea addresses
- Who experiences this problem most acutely (be specific — role, context, frequency)
- How painful / costly the problem is for them today

If the problem is vague or the affected user isn't crisp, say so directly. A weak problem statement here is the most common reason ideas fail later.

## Step 2 — Solution validation

State:
- Why this is the right solution shape (not just *a* solution)
- The 2–3 most credible alternatives, including the status quo / "do nothing"
- The specific reason this idea beats the strongest alternative

If the alternative is roughly as good, surface that — it's a signal the idea may not be worth pursuing as-framed.

## Step 3 — Impact assessment

State:
- The success metric that would prove this worked (one metric, numeric where possible — activation rate, retention, time saved per user, revenue, etc.)
- The target value and time horizon (e.g., "30% of new users complete X within 14 days, measured 90 days post-launch")
- What changes for users if this succeeds (concrete behavior change, not abstract benefit)

Avoid vanity metrics. "Increased engagement" is not a metric.

## Step 4 — Verdict

Close with one of:
- **Worth building** — problem is real, solution beats alternatives, success is measurable. Recommend running `run-product-discovery` for full spec.
- **Worth building with caveats** — list the specific risks or assumptions that must hold.
- **Reframe first** — the problem or solution as stated has a gap. Name it. Suggest a sharper framing.
- **Skip** — alternatives are roughly as good, or the problem isn't painful enough to drive adoption. Say so honestly.

Keep the whole analysis under ~300 words. This is a sanity check, not a PRD.
