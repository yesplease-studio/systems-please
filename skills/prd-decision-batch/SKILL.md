---
name: prd-decision-batch
description: Emit a single, scannable, ranked decision package from a project's `LOOP-STATE.md` decision queue, replacing one-at-a-time interruptions. Use this skill when 3+ decisions are queued, when learner/apply debt is near the ceiling, when the next batches all depend on the same open decision, or at the end of a session with open decisions. Trigger when the user says things like "decisions", "what do you need from me", "batch the decisions", or "decision package". Humans review batched, ranked decisions far better than a steady stream of interruptions — the orchestrator should forecast the bottleneck and emit the package before velocity hits the wall.
---

# Skill: prd-decision-batch

**Purpose:** Convert the running decision queue into one scannable package the human reviews in a single pass. Replaces stream-of-interruption decision asks with one consolidated batch, ranked by leverage and irreversibility.

---

## When to Use

| Trigger | What happens |
|---|---|
| 3+ open decisions in the `LOOP-STATE.md` Decision queue | Step 1 — emit the package |
| Learner debt or Apply debt near ceiling | Step 1 — many remaining batches are likely gated on decisions; surface them now |
| The user says "what do you need from me" or "decisions" | Step 1 — emit even if the queue is small |
| The orchestrator detects the next 2 batches all depend on the same open decision | Step 1 — escalate that one decision first |
| End of a session with any decisions open | Step 1 — emit before closing |

---

## Step 1: Read the Decision Queue

Use the `prd-loop` skill to read the Decision queue section of `LOOP-STATE.md`. Each row has: ID, Decision, Recommended, Unblocks, Logged.

If the queue is empty, report `No open decisions.` and stop. Do not invent decisions to justify a run.

---

## Step 2: Enrich Each Row

For every row, gather:

| Field | How to derive |
|---|---|
| `leverage` | What % of remaining work is gated on this decision. Approximate from the Unblocks field plus the orchestrator's view of the next 2-3 batches |
| `irreversibility` | `low` (can revisit later) / `medium` (mild rework cost) / `high` (frozen contract; hard to undo) |
| `recommended` | Already in the row from when the decision was logged. Confirm it still applies; update if the loop has shifted |
| `recommended_why` | One line. From the logging context or the learner notes that surfaced the decision |
| `risk_if_default` | What happens if the human chooses the recommended option |
| `risk_if_alternative` | What happens if the human overrides |

If any field is genuinely unknown, write `unknown` rather than guess. The package's value comes from honesty.

---

## Step 3: Rank

Sort by leverage descending, then by irreversibility descending. The first decision in the package is the one with the most downstream impact that is also hardest to revisit later.

Cap the package at 7 decisions. If there are more, include the top 7 and note the remaining count at the bottom (`N more decisions queued — will batch after these are resolved`). Long packages do not get read.

---

## Step 4: Compose the Package

Write to `<project_scope>/decisions/<YYYY-MM-DD>-DECISION-PACKAGE.md`:

```markdown
# Decision Package — <project>

**Date:** YYYY-MM-DD
**Open decisions:** <total queue size>
**In this package:** <up to 7>
**Loop state:** batch <id>, phase <phase>, debt L=<n> A=<n>

---

## How to use this

Each decision below has a recommended option. To accept all recommendations, say "accept all". To override or modify, reference the decision ID (`OQ-3: take option B because <reason>`). Decisions not in this package are queued; the next package will pick them up.

---

## 1. <OQ-ID>: <one-line decision>

**Recommended:** <option> — <one line why>
**Unblocks:** <what becomes possible after this>
**Leverage:** <approx % of remaining work gated on this>
**Irreversibility:** <low | medium | high>
**Risk if recommended:** <one line>
**Risk if alternative:** <one line>

(Repeat for each decision, in ranked order.)

---

## Not in this package (queued)

- <OQ-N>: <one-line decision>

(Only present if the queue exceeds 7.)
```

---

## Step 5: Surface to the Human

Present the package path and a one-line preview of the top 3:

```
Decision package ready — <path>

Top 3:
1. <OQ-ID>: <decision> → <recommended>
2. <OQ-ID>: <decision> → <recommended>
3. <OQ-ID>: <decision> → <recommended>

<N> more in the package. Say "accept all" or reference an OQ-ID to override.
```

Then wait. Do not advance the loop while decisions are open.

---

## Step 6: Apply the Decisions

For each decision the human resolves:

1. Remove the row from the `LOOP-STATE.md` Decision queue (via the `prd-loop` skill).
2. If the decision triggers an immediate action (start a new batch, freeze a contract, kick off a learner re-run), do that action.
3. If the decision results in a PRD or ADR amendment, route through the standard learn → apply path (`prd-learner` → `prd-author` in edit mode); do not edit those files inline from this skill.
4. If the decision resolves a PRD open question (`OQ-XX`), the resolution belongs in the PRD too — flag it for `prd-author`.

Update the package file to record the resolutions, in place. The file becomes a permanent record of the batch's decisions for future reference.

---

## Edge Cases

- **Two decisions are mutually exclusive.** Group them as `1a / 1b` with a shared lead-in: "These trade off against each other. Choose one." The orchestrator does not pick for the human.
- **A decision depends on another decision in the same package.** Order them sequentially and note `Depends on resolution of OQ-N` on the second.
- **Recommended option no longer applies.** Update the Recommended field before emitting. Stale recommendations erode trust in the package.
- **The human overrides every recommendation.** The orchestrator's defaults may be miscalibrated — note this and recalibrate future recommendations against the override rationale.
- **A decision in the queue has been there for >14 days.** Either it is genuinely waiting on external input (note that on the row) or it is dead. Surface as a separate question: "OQ-N has been open 14 days — still live or drop?"
- **No project scope context.** Decision packages are per-project. If invoked outside a project scope without one specified, ask before composing.
