---
name: prd-learner
description: Capture implementation learnings and codify them back into the Strategic PRD so future agents and team members inherit the knowledge. Use this skill whenever you need to close the feedback loop after building — after a validation failure, a bug is resolved, repeated mistakes in a session, or at the end of a sprint as a retrospective step. Trigger when the user says things like "capture learnings", "update the PRD based on what happened", "log what went wrong", "retrospective", "we keep hitting this issue", or "add this to the PRD". Always use this skill when the goal is turning implementation experience into permanent PRD rules.
---

# Skill: prd-learner

**Purpose:** Capture implementation learnings and codify them back into the Strategic PRD so that future agents, sessions, and team members inherit the knowledge. This skill closes the feedback loop — every mistake becomes a permanent rule.

---

## When to Use

- After a validation failure (something was built wrong and the root cause is worth capturing).
- After resolving a non-trivial bug or issue during implementation.
- After a session where repeated mistakes occurred.
- When a human explicitly asks to "capture learnings", "update the PRD based on what happened", or similar.
- Periodically, at the end of a development sprint or cycle, as a retrospective step.

## Core Principle

Learnings must be **actionable, specific, and permanent**. "We should be more careful" is not a learning. "All public API endpoints must implement rate limiting (added after brute-force incident on /auth/login)" is a learning.

A good learning results in a concrete change to the Strategic PRD: a new guardrail, a tightened requirement, an added acceptance criterion, a new "don't." The learning entry in Section 8 provides the *why*; the amendment in Sections 5-6 provides the *what*.

---

## Workflow

### Step 1: Gather Context

Collect information about what happened. Sources, in order of preference:

1. **Validation report** from `prd-validator` — if the learning was triggered by a validation failure, start here. The report identifies which requirements were unmet and why. In the build loop, this is `<project_scope>/validation/<batch_id>-REPORT.md`.

2. **Impl agent reports** — the `notes_for_learner` field from each implementation sub-agent's structured report. This is the designated pipe for patterns and workarounds observed during building; sub-agents record them there precisely so this skill can consume them.

3. **Workaround signatures** — the Workaround signatures table in `LOOP-STATE.md`, if the project runs the build loop. Recurring signatures are the strongest signal this skill receives (see Pattern Detection below).

4. **Session transcript / conversation history** — review the recent agent session for: errors encountered, workarounds applied, repeated attempts, corrections, and patterns.

5. **Human input** — if automated sources are insufficient, ask the human via `AskUserQuestion`:
   - "What was the most surprising or time-consuming issue in this session?"
   - "Were there mistakes that a clearer requirement or guardrail could have prevented?"
   - "Did you discover any patterns that should be standardized?"

### Step 2: Load Current PRD State

Read the full Strategic PRD to understand:

- Existing requirements (Section 5) — to check if the learning is already covered.
- Existing guardrails (Section 6) — to check for gaps.
- Existing learnings (Section 8) — to avoid duplicates and to see patterns.
- Dependencies (frontmatter `depends_on`) — the learning might apply to a dependent PRD instead.

### Step 3: Classify Each Learning

For each distinct learning identified, classify it:

| Category | Result | Criteria |
|----------|--------|----------|
| **New guardrail** | Add to Section 6 | The learning defines a cross-cutting rule that applies broadly (not just one requirement). |
| **Tightened requirement** | Amend in Section 5 | An existing requirement was too vague, too lenient, or missing an acceptance criterion. |
| **New requirement** | Add to Section 5 | The learning reveals a gap — something that should have been specified but wasn't. |
| **Amended acceptance criteria** | Update in Section 5 | The acceptance criteria for an existing requirement need to be more specific or additional criteria added. |
| **Already documented** | No PRD change | The learning is already captured in the PRD. Note this in the report. |
| **Not repeatable** | No PRD change | The issue was session-specific or environmental — not something the PRD should address. |
| **Applies to different PRD** | Flag for other PRD | The learning affects a dependent PRD, not this one. Recommend updating the other PRD. |

### Step 4: Draft Amendments

For each learning that requires a PRD change, draft:

1. **The learning entry** (for Section 8):

```markdown
### L-XXX: [Short descriptive title]
- **Date:** YYYY-MM-DD
- **Domain:** [domain]
- **Related requirements:** [requirement IDs, or "new" if adding]
- **What happened:** [Factual description of the incident or discovery]
- **Root cause:** [Why it happened — what was missing or wrong in the PRD]
- **Amendment made:** [Precise description of what changed in the PRD]
- **Status:** resolved
```

2. **The inline amendment** (in Sections 5 or 6):

- For a new guardrail: the exact DO or DON'T text to add to Section 6.
- For a tightened requirement: the specific change to the requirement row in Section 5.
- For a new requirement: the complete requirement row to add.
- For amended acceptance criteria: the specific criteria to add or modify.

**Every inline amendment should include a reference to its learning ID** so that future readers can trace the rule back to the incident. Format: add `[L-XXX]` at the end of the amended text.

### Step 5: Present for Approval

Show the human the proposed changes:

```markdown
## Learning Report — PRD-001

### Session Summary
[Brief description of what was being built and what triggered the learning capture]

### Proposed Amendments

**1. New guardrail (Section 6):**
- DON'T expose stack traces or internal error details in API responses. Return generic error messages with correlation IDs for debugging. [L-003]

**2. Tightened requirement (Section 5.2, TECH-01):**
- Added acceptance criterion: "Rate limiting applied: max 10 requests per minute per IP on authentication endpoints." [L-003]

**3. New requirement (Section 5.2):**
| ID | Severity | Requirement | Acceptance Criteria | Phase |
|----|----------|-------------|-------------------|-------|
| TECH-08 | must | All public endpoints implement rate limiting | Configurable per-endpoint; default 60 req/min; returns 429 with Retry-After header | R1 |
[L-003]

### Watchlist (below threshold — tracking)
- Supabase client timeout on cold start: seen once this sprint. Not proposing yet; will promote if it recurs.

### No Action Needed
- Agent timeout during build was environmental (CI runner resource limits), not a PRD gap.

### Patterns Observed
- This is the second learning related to API security (see also L-001). Consider whether a dedicated security review step should be added to the build workflow.
```

**Threshold discipline:** below-threshold signals (seen once, plausibly one-off) go to the Watchlist, never to Proposals. The Proposals section is for things ready to commit to as permanent rules. A learning report with empty Proposals and a populated Watchlist is a valid outcome — do not invent amendments to justify the run.

**Wait for human approval before any changes are applied.**

### Step 6: Apply (via prd-author)

Once approved, invoke the `prd-author` skill in **edit mode** to apply the amendments. The `prd-learner` does not edit the PRD directly — `prd-author` is the single authoritative writing layer.

Provide `prd-author` with:

- The specific amendments to make (from Step 4).
- The learning entries to add to Section 8.
- The change summary format.

After `prd-author` applies the changes:

- Notify `prd-taskmaster` that affected task contexts may need regeneration.
- If the project has a `CONTEXT-PACK.md`, regenerate it via the `prd-context-pack` skill — amendments invalidate the pack.
- If the project runs the build loop, update `LOOP-STATE.md` via the `prd-loop` skill: decrement Learner debt and Apply debt, increment workaround signature counts for any signatures this run identified, and mark `Generalizing ADR proposed? = yes` on any signature whose ADR proposal was approved.

---

## Quality Criteria for Learnings

Every learning must be:

- **Actionable** — It results in a concrete PRD change (or a clear reason why no change is needed).
- **Specific** — It names exact patterns, endpoints, behaviors, thresholds — not generalities.
- **Non-obvious** — It captures something that wasn't apparent from the original PRD. Obvious requirements that were simply missed belong in `prd-author` edit, not here.
- **Repeatable** — The issue could happen again if not addressed. One-off environmental issues are not learnings.

**Good learning:** "Authentication endpoints must implement rate limiting — brute-force attempt on /auth/login exposed the lack of throttling."

**Bad learning:** "We should write more secure code." (Not specific, not actionable, not tied to a PRD change.)

---

## Patterns and Escalation

### Pattern Detection

When adding a new learning, check Section 8 for related entries. If this is the **third or more learning in the same domain or area**, flag it as a pattern:

> "This is the third learning related to API security (L-001, L-003, L-005). This pattern suggests the PRD may need a dedicated security requirements subsection or that security should be elevated as a cross-cutting concern."

Pattern detection helps the human decide whether a more fundamental PRD restructuring is needed, rather than just incremental patches.

### Workaround Signatures → Generalizing ADRs

When the project runs the build loop, `LOOP-STATE.md` tracks **workaround signatures** — the shape of a workaround, recorded the second time it appears (the first looks like a one-off; the second is the signal). This skill is the consumer of that table:

- **Count = 2:** note the recurrence in the learning report's Watchlist. Not yet a proposal.
- **Count >= 3:** propose a **generalizing ADR** — a rule that would obviate the workaround class entirely. Include: the signature, occurrences across batches, the recommended scope of the rule, a one-paragraph sketch, and what future work it unblocks. Route the proposal through the standard approval path; once approved, it becomes an ADR via `archgate adr create` (or an import, if a registry pack covers it — see `systems/prd/SYSTEM.md` §8.2).

This is the runtime complement to `prd-taskmaster`'s `adr_candidate` flag: taskmaster predicts which decisions need enforcement before building starts; signature tracking catches the ones that only reveal themselves during the build.

### Escalation

Escalate to a human without proposing amendments when:

- The learning suggests the solution approach (Section 4) is fundamentally wrong.
- The learning reveals a conflict between two `must` requirements.
- The learning applies to a PRD the learner doesn't have access to.
- The pattern detection suggests systemic issues beyond individual amendments.

---

## What This Skill Does NOT Do

- **Does not edit the PRD directly.** All changes go through `prd-author`.
- **Does not write or modify code.** It captures knowledge, not implementations.
- **Does not validate.** Validation is `prd-validator`'s job. This skill runs after validation has already identified issues.
- **Does not create new PRDs.** If a learning suggests entirely new scope, it flags this for the human to decide.

---

## Token Optimization

- **Selective PRD loading.** When triggered by a validation report, load only the sections referenced in the report (plus Section 8 for existing learnings). Only load the full PRD if the learning seems to span multiple domains.
- **Batch learnings.** When multiple issues arise in a single session, batch them into one learning report rather than running the skill multiple times. Each distinct learning gets its own `L-XXX` entry, but the human approval happens once for the whole batch.
- **Concise proposals.** The amendment drafts should be minimal — the exact text to add or change, not a lengthy analysis. The learning entry provides context; the amendment is the action.
