# PRD Please — Architecture

## What this document is

A technical overview of how PRD Please is structured, why it's structured this way, and how the layers compose.

---

## The layered model

Everything in PRD Please follows a pattern: there's a **methodology** (how to think about the problem), **skills** (how to execute the work), **company context** (who you're doing it for), and **workflows** (in what sequence). For orchestrated builds, a fifth element — **agents** (who does the work) — runs the execution loop.

### Layer 1: Systems (the "how to think")

A system is a methodology document that defines the problem space, core principles, how its skills compose, and quality standards. Skills reference their parent system when methodology reasoning is needed.

| System | File | What it codifies |
|--------|------|-----------------|
| PRD | `systems/prd/SYSTEM.md` | Three-layer PRD architecture, domain forcing, learning loops |

Systems are loaded on demand — a skill doesn't need the full system definition for routine execution, but designing a new engagement or resolving an ambiguity does.

### Layer 2: Skills (the "how to do")

Skills are `SKILL.md` files — self-contained executable specifications that Claude reads before performing a task. Each skill has a single responsibility and produces a defined output.

| Skill | Purpose |
|-------|---------|
| `prd-author` | Create and edit Strategic PRDs from human-provided context |
| `prd-taskmaster` | Derive executable tasks, batches, and client-facing views from a Strategic PRD |
| `prd-validator` | Adversarially validate implementation against PRD requirements and guardrails — executes typechecks, builds, and tests |
| `prd-learner` | Capture implementation learnings and propose PRD amendments |
| `prd-loop` | Manage the build loop's externalized state (`LOOP-STATE.md`) |
| `prd-context-pack` | Compile frozen contracts into the `CONTEXT-PACK.md` sub-agents read at spawn |
| `prd-agent-brief` | Compose sub-agent invocation contracts (file ownership, token budgets, report formats) |
| `prd-decision-batch` | Emit ranked decision packages from the loop's decision queue |
| `prd-to-aldente` | Translate a Strategic PRD into Al Dente build documentation |
| `build-learner` | Capture build-phase learnings and propose amendments upstream or laterally |

**What stays in a skill:** format, structure, process steps, quality standards, self-check.

**What lives in the company profile:** product context, users, constraints, domain-specific details.

### Layer 3: Company profiles (the "who for")

A company profile (`COMPANY.md`) contains everything skills need to produce company-specific output. For PRD skills, the key sections are: product context, user definitions, technical constraints, and competitive landscape.

The complete schema is documented in `companies/_template/COMPANY.md`.

**How skills load context at runtime:**

```markdown
## Step 0: Load context

1. Read CLAUDE.md to identify the active company.
2. Read `companies/<active-company>/COMPANY.md` for company context.
3. If no company profile is found, ask the user which company they're working
   for before proceeding. Do not guess.
```

### Layer 4: Workflows (the "in what sequence")

A workflow is a composed skill sequence defined in YAML. Workflows chain skills together, passing outputs between steps, with conditional execution and human approval gates.

```yaml
name: prd-new-engagement
description: Author a PRD, get approval, generate tasks

steps:
  - id: author
    skill: prd-author
    approval: true

  - id: tasks
    skill: prd-taskmaster
    inputs:
      prd: "{{steps.author.output}}"
```

### Layer 5: Agents (the "who does it")

For orchestrated builds, three role-typed agent definitions ship in `.claude/agents/` and are discovered automatically by Claude Code:

| Agent | Permission envelope | Writes |
|-------|--------------------|--------|
| `impl` | Bash, Read, Write, Edit, Grep, Glob | Only files in its brief's MAY-WRITE list |
| `validator` | Bash (build/typecheck/test execution), Read, Grep, Glob, Write | Only its validation report |
| `learner` | Read, Grep, Glob, Write — no Bash | Only its LEARNINGS proposal file |

The permission envelope is part of the role, not negotiated per spawn. The `prd-agent-brief` skill composes the per-spawn contract (task, file ownership, token budget, report format); the agent definition supplies the standing rules (honest degradation, adversarial verification, proposal-only synthesis).

The execution loop's coordination state lives in derived files at the project scope — `LOOP-STATE.md` (batch pointer, debt counters, decision queue, workaround signatures) and `CONTEXT-PACK.md` (the curated contract set agents read instead of the full corpus). Both survive session boundaries; when conversation memory and file disagree, the file wins. See `systems/prd/SYSTEM.md` §7.4 for the full loop methodology.

---

## How it connects

```
CLAUDE.md (per-workspace)
    ↓ sets active company
companies/<company>/COMPANY.md
    ↓ loaded before every skill
skills/<skill>/SKILL.md
    ↓ follows methodology from
systems/prd/SYSTEM.md
    ↓ composed into
workflows/<workflow>.yaml
```

---

## Progressive context loading

Not everything loads at once. Context is loaded progressively as needed:

1. **CLAUDE.md** — always in context. Sets active company, points to systems.
2. **SKILL.md** — loads when triggered.
3. **COMPANY.md** — loads once on first company-facing skill.
4. **SYSTEM.md** — loads only for methodology reasoning.
5. **Workflow YAML** — loads only for orchestrated execution.

This keeps token usage efficient. Updates to company context propagate instantly to all skills without editing each one.

---

## Design decisions

### Why a flat skills directory

Claude Code discovers skills by scanning a directory for `SKILL.md` files. The `skills/` directory stays flat for compatibility. Company-specific skill overrides live in `companies/<company>/skills/`.

### Why companies are gitignored

Company profiles contain proprietary context — product details, competitive intelligence, internal constraints. They should never be pushed to a shared remote. The `.gitignore` excludes `companies/` by default (except `_template/` and `examples/`).

### Why CLAUDE.md is generated, not committed

CLAUDE.md sets the active company — it's workspace-specific. Committing it would force one active company on everyone. Generated-per-workspace means it's always correct for the environment you're in.

### Why workflows are YAML, not code

YAML is human-readable, version-controllable, and Claude interprets it directly. No runtime, no dependencies, no build step.

---

## Integration with build systems

PRD Please owns *what to build*. Build systems (like [Al Dente](https://github.com/aline-no/aldente)) own *how to build it*. The integration boundary is clean: a translation skill maps PRD artifacts into the build system's expected format, and a learning skill feeds implementation experience back upstream. When the build phases are executed by AI sub-agents, the execution loop (Layer 5) orchestrates them batch by batch.

### The integration flow

```
Strategic PRD (active)
        ↓
   prd-to-aldente skill
   Translates PRD sections → 7 Al Dente doc templates
        ↓
Al Dente docs/ (content-pages, journeys, ui-structure,
   data-models, schema, design-guidelines, assets)
        ↓
Al Dente build phases (01-setup → 11-launch-audit)
        ↓
   build-learner skill
   Captures implementation learnings
        ↓
   Amendments flow to:
   ├── Upstream → Strategic PRD (via prd-author)
   ├── Lateral → Al Dente docs (direct edit)
   └── Flagged → Al Dente build guides (for maintainer)
```

### Design principles for integration

- **Translation, not coupling.** The `prd-to-aldente` skill generates self-contained documents. Al Dente never reads the PRD directly; the translation skill is the only point of contact.
- **Bidirectional learning.** The `build-learner` skill feeds learnings both upstream (improving the PRD) and laterally (improving the build docs), closing the loop in both directions.
- **Optional integration.** The PRD system works standalone without any build system. The Al Dente integration is an additive layer, not a dependency.
- **Overridable defaults.** When using the quick-start workflow, Al Dente's default stack (React, Supabase, Stripe, etc.) is presented as suggestions during the PRD authoring phase. The user can override any default based on their project's needs.
