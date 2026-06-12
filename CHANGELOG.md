# Changelog

All notable changes to PRD Please will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/), and this project adheres to [Semantic Versioning](https://semver.org/).

## [1.1.0] - 2026-06-12

### Added

- **Execution loop (orchestrated builds)** — A new execution layer that runs the impl → validate → learn → apply → commit cycle batch by batch, with externalized state that survives session boundaries. Four new skills: `prd-loop` (loop state at `LOOP-STATE.md`: batch pointer, debt counters, decision queue, workaround signatures), `prd-context-pack` (curated `CONTEXT-PACK.md` sub-agents read instead of the full PRD + ADR corpus), `prd-agent-brief` (sub-agent invocation contracts: MAY-WRITE/MAY-NOT-TOUCH file ownership, token budgets, structured report format, honest degradation), and `prd-decision-batch` (ranked decision packages replacing one-at-a-time interruptions). Methodology in `systems/prd/SYSTEM.md` §7.4; one-batch workflow at `workflows/product/prd-build-loop.yaml`.
- **Role-typed agent definitions** — `.claude/agents/impl.md`, `validator.md`, and `learner.md` ship with the repo and are discovered automatically by Claude Code. Permission envelopes are part of the role: impl writes within its contract, validator is read-only with execution rights, learner is read-only and proposal-only.
- **Workaround signatures** — Recurring workarounds (3+ occurrences) trigger generalizing-ADR proposals via `prd-learner`/`build-learner`: the runtime complement to `prd-taskmaster`'s pre-build `adr_candidate` flag.
- **Batch grouping in `prd-taskmaster`** — Tasks group into batches (the loop's unit of work) with declared file footprints that drive MAY-WRITE contracts.

### Changed

- **`prd-validator` is now adversarial and executes.** The validator runs typechecks, builds, and tests for the touched scope instead of trusting self-reports (a validator that cannot execute is not validating). Per-AC verdicts use the `pass | fail | partial-by-design | not-applicable` taxonomy; dishonest fallbacks are `HARD-FAIL`; out-of-contract file changes are `CONTRACT-VIOLATION`. Cross-cutting checks run on every validation.
- **`prd-learner` consumes loop signals** — impl agents' `notes_for_learner` reports and the workaround signature table join validation reports as primary sources; reports gain a Watchlist and threshold discipline (empty proposals with a populated watchlist is a valid outcome).
- **Model guidance modernized** — Skills no longer hardcode model names. The durable rule lives in `systems/prd/SYSTEM.md` §1.3: judgment roles on the strongest available model, checklist/formatting roles on the smallest model that does the job reliably.
- **Design principles extended** — Principle 5 reframed from token efficiency to context discipline; new principles 7 ("State lives in files, not conversations") and 8 ("Trust is verified, not reported").
- **Naming cleanup** — Remaining "Systems Please" references updated to "PRD Please" across docs, templates, and skills.

## [1.0.0] - 2026-03-20

### Added

- **PRD System** — Full methodology for authoring, decomposing, validating, and learning from product requirements. Includes four skills: `prd-author`, `prd-taskmaster`, `prd-validator`, `prd-learner`.
- **Workflow orchestration** — YAML-based multi-step skill sequences for PRD workflows (new engagement, post-build validation).
- **Company profile template** — Scaffold for setting up new company contexts.
- **Deploy templates** — Setup guides and configuration scaffolds for new workspaces.
- **Example company** — Fictional "Acme Analytics" demonstrating the full PRD lifecycle.
- **Documentation** — Architecture overview and company schema reference.
