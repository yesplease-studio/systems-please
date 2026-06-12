---
name: build-learner
description: Capture implementation learnings during build phases and propose amendments to the Strategic PRD (upstream) and/or build documentation (lateral). Use this skill after completing a build phase, encountering repeated implementation issues, resolving non-trivial build problems, or at the end of a build sprint. Trigger when the user says things like "capture build learnings", "log what happened during the build", "this phase was harder than expected", "update docs based on what we learned", or "build retrospective". This skill closes the feedback loop between implementation and specification.
---

# Skill: build-learner

**Purpose:** Capture implementation learnings during build phases and codify them into permanent improvements — either upstream in the Strategic PRD or laterally in the build documentation. This skill ensures that implementation experience flows back into the specification layer so future builds benefit from past work.

---

## When to Use

- After completing a build phase (e.g., Al Dente phases 01-setup through 11-launch-audit).
- When a build step required significant deviation from the documented approach.
- When a PRD requirement turned out to be un-implementable as specified.
- When repeated issues occurred during a build session.
- At the end of a build sprint as a retrospective.
- When a human explicitly asks to capture build learnings.

## Core Principle

Build learnings operate at the **implementation level**, not the requirements level. The `prd-learner` skill captures "the requirement was wrong or missing." The `build-learner` captures "the requirement was fine but the translation to implementation hit problems" or "the build documentation was misleading."

A good build learning results in a concrete change to either the Strategic PRD (upstream) or the build documentation (lateral). If it doesn't change a document, it's an observation, not a learning.

---

## How This Skill Differs from prd-learner

| Dimension | prd-learner | build-learner |
|-----------|-------------|---------------|
| **Scope** | Requirements-level | Implementation-level |
| **Trigger** | After validation failure, sprint retrospective | After build phase, during build session |
| **Amendment targets** | Strategic PRD only | PRD (upstream) + build docs (lateral) |
| **Learning IDs** | `L-XXX` | `BL-XXX` |
| **Phase tagging** | None | Tagged with build phase (e.g., `phase: 05-db-flow`) |
| **Pattern detection** | By domain | By build phase |

The two skills complement each other. Use `prd-learner` when the issue is *what was specified*. Use `build-learner` when the issue is *how it was built or documented*.

---

## Workflow

### Step 1: Gather Context

Collect information about what happened during the build phase. Sources:

1. **Build session history** — Review the recent agent session for: errors encountered, workarounds applied, deviations from the build docs, repeated attempts, and corrections. If the build ran as an orchestrated loop, also read the `notes_for_learner` fields from impl agent reports and the Workaround signatures table in `LOOP-STATE.md` — a signature with count >= 3 is a generalizing-ADR candidate (see the `prd-learner` skill's "Workaround Signatures → Generalizing ADRs" section; the same mechanism applies here for implementation-level signatures).

2. **Build phase documentation** — The build phase prompt or guide that was being followed (e.g., Al Dente's `prompt/05-db-flow.md`).

3. **Build documentation** — The generated docs that informed the build (e.g., `data-models.md`, `ui-structure.md`). Compare what was documented against what was actually built.

4. **Human input** — If automated sources are insufficient, ask the human via `AskUserQuestion`:
   - "What was the most surprising or time-consuming issue during this build phase?"
   - "Did the build docs accurately describe what needed to be built, or did you have to deviate?"
   - "Were there decisions you had to make that should have been specified upfront?"

5. **Diff of built vs. specified** — If available, compare the implementation output against what the build docs specified. Look for structural deviations, missing features, or added features not in the docs.

### Step 2: Load Current State

Read the relevant documents to check what's already captured:

- **Strategic PRD** (Sections 5, 6, 8) — To check if the learning reveals a PRD gap vs. a translation/build gap.
- **Build documentation** — The specific templates or guides that were used during the build phase.
- **Existing build learnings** — Check for previous `BL-XXX` entries to detect patterns and avoid duplicates.

### Step 3: Classify Each Learning

For each distinct learning identified, classify it by where the fix belongs:

| Category | Target | Criteria | Action |
|----------|--------|----------|--------|
| **PRD gap** | Upstream (Strategic PRD) | The requirement was wrong, vague, or missing. The build docs faithfully translated a flawed requirement. | Route through `prd-author` (edit mode) — same as `prd-learner`. |
| **Translation gap** | Build documentation | The PRD was correct but the translation skill (`prd-to-aldente` or equivalent) didn't map it well to the build docs. | Re-run the translation skill or apply a direct edit to the build docs. |
| **Build-guide gap** | Build phase documentation | The build phase guide itself was misleading, incomplete, or assumed something that wasn't true. | Flag for the build system maintainer (e.g., Al Dente repo). The `build-learner` does not own the build phase guides. |
| **Implementation-only** | Learning log only | The issue was code-level (a bug, a library quirk, an environment issue) and doesn't need doc changes. | Log it for reference but propose no amendments. |
| **Already documented** | None | The learning is already captured in the PRD or build docs. | Note in the report; no action needed. |

### Step 4: Draft Amendments

For each learning that requires a document change, draft:

**1. The learning entry:**

```markdown
### BL-XXX: [Short descriptive title]
- **Date:** YYYY-MM-DD
- **Phase:** [build phase, e.g., 05-db-flow]
- **Domain:** [PRD domain this relates to, e.g., Data, Tech, Product]
- **Target:** prd | build-docs | build-guide | none
- **Related requirements:** [requirement IDs from the PRD, or "new" if adding]
- **What happened:** [Factual description of the issue]
- **Root cause:** [Why it happened — what was wrong or missing in which document]
- **Proposed amendment:** [Precise description of what should change and where]
- **Status:** proposed
```

**2. The amendment itself:**

- **For PRD amendments:** The exact change to make in the Strategic PRD (same format as `prd-learner` — new guardrail text, tightened requirement, new acceptance criterion). Reference the `BL-XXX` ID.
- **For build doc amendments:** The exact change to make in the build documentation template. Specify which file and section.
- **For build-guide flags:** A clear description of what's wrong in the build phase guide and what the fix should be, formatted for submission to the build system maintainer.

### Step 5: Present for Approval

Show the human the proposed changes, **separated by target**:

```markdown
## Build Learning Report — [Phase Name]

### Session Summary
[Brief description of what was built and what triggered the learning capture]

### Upstream Amendments (Strategic PRD)

**1. [Amendment title]:**
- Target: PRD-XXX, Section [X]
- Change: [Precise change description]
- Reason: [BL-XXX reference]

### Lateral Amendments (Build Documentation)

**1. [Amendment title]:**
- Target: [filename], Section [X]
- Change: [Precise change description]
- Reason: [BL-XXX reference]

### Build-Guide Flags (for build system maintainer)

**1. [Flag title]:**
- Phase: [phase number and name]
- Issue: [What's wrong or misleading]
- Suggested fix: [What should change]

### No Action Needed
- [Issue]: [Why no document change is required]

### Patterns Observed
- [Pattern detection: "This is the Nth learning related to [area]. Consider..."]
```

**Approval is per-target.** The human may approve PRD amendments but reject build doc changes, or vice versa. Do not batch approval across targets.

### Step 6: Apply

- **PRD amendments:** Invoke `prd-author` in edit mode (same mechanism as `prd-learner`). Provide the specific amendments and learning entries for Section 8.
- **Build doc amendments:** Apply directly to the build documentation files. If the docs were generated by `prd-to-aldente`, note in the file that the change was a manual post-translation amendment: `<!-- Amended by build-learner: BL-XXX -->`.
- **Build-guide flags:** Do not apply. Present them to the user for submission to the build system maintainer (e.g., as a GitHub issue on the Al Dente repo).

After applying PRD amendments, notify `prd-taskmaster` that affected task contexts may need regeneration.

---

## Pattern Detection

When adding a new learning, check existing `BL-XXX` entries for patterns:

- **By build phase:** If this is the third or more learning in the same build phase, flag: "Phase [X] has generated [N] learnings. This suggests the build phase documentation or the PRD-to-build-doc translation for this area needs structural improvement, not just patches."
- **By PRD domain:** If multiple build learnings trace back to the same PRD domain, flag for `prd-learner` to review whether the domain requirements are systematically under-specified.
- **By target:** If most learnings are "translation gaps," the `prd-to-aldente` mapping for that area may need refinement.

---

## Standalone Mode (Without PRD Please)

This skill can operate independently within a build system (e.g., the Al Dente repository) without a PRD Please PRD upstream. In standalone mode:

- **Skip Step 2's PRD loading.** There is no upstream PRD to check against.
- **All learnings are lateral.** The only amendment targets are the build documentation and build-guide flags.
- **Classification simplifies:** "PRD gap" becomes "specification gap" — flag it for whoever owns the product specification, in whatever format they use.
- **Learning entries use the same `BL-XXX` format** but omit the "Related requirements" field (no PRD requirement IDs to reference).
- **The workflow is otherwise identical.** Gather context, classify, draft, present, apply.

This mode allows the skill to be adopted by build system maintainers (e.g., dropped into the Al Dente repository) without requiring the full PRD Please methodology.

---

## What This Skill Does NOT Do

- **Does not run build phases.** It captures learnings after building, not during.
- **Does not edit the Strategic PRD directly.** PRD amendments route through `prd-author`, maintaining the single-writer principle.
- **Does not push changes to external repositories.** Build-guide flags are presented to the user for manual submission.
- **Does not replace `prd-learner`.** The two skills have different scopes: `prd-learner` for requirements-level issues, `build-learner` for implementation-level issues. If a build learning reveals a pure requirements gap, it routes upstream through the same `prd-author` mechanism that `prd-learner` uses.

---

## Quality Criteria for Build Learnings

Every learning must be:

- **Actionable** — It results in a concrete document change (or a clear reason why no change is needed).
- **Specific** — It names exact files, sections, patterns, or behaviors — not generalities.
- **Traceable** — It connects to a specific build phase and, when applicable, specific PRD requirements.
- **Repeatable** — The issue would happen again on the next build if not addressed. One-off environment issues are not learnings.

**Good learning:** "The `data-models.md` template defined a `projects` entity with a `status` field as a string, but the PRD's PROD-04 requirement for workflow transitions implies an enum with specific allowed values. The translation skill should derive enum values from acceptance criteria when they describe state transitions." [BL-003]

**Bad learning:** "The database setup was confusing." (Not specific, not traceable, not actionable.)

---

## Token Optimization

- **Selective loading.** Load only the PRD sections and build docs relevant to the build phase being reviewed. Only load the full PRD if the learning spans multiple domains.
- **Batch learnings.** Multiple issues from a single build phase should be batched into one learning report. Each distinct learning gets its own `BL-XXX` entry, but human approval happens once for the whole batch.
- **Concise amendments.** The amendment drafts should be minimal — the exact text to add or change. The learning entry provides context; the amendment is the action.
