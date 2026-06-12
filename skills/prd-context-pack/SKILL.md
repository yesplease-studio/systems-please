---
name: prd-context-pack
description: Generate or regenerate a project's curated build context pack — the single `CONTEXT-PACK.md` that sub-agents read instead of the full PRD + ADR + docs corpus. Use this skill at the start of an orchestration cycle, after a PRD or ADR amendment is applied, or whenever the pack predates the latest contract change. Trigger when the user says things like "regenerate the context pack", "refresh context", "build the context pack", or "the pack is stale". The pack is a derived artifact: every line comes from a tracked file, and regenerating it changes freshness, never meaning.
---

# Skill: prd-context-pack

**Purpose:** Compile a project's current frozen contracts into a single file that sub-agents read at spawn. This replaces tens of thousands of tokens of redundant context loading per agent spawn with one curated read.

The pack is **derived**, not authored. Every line in it comes from a tracked file. Regenerating the pack should not change meaning, only freshness. It is the fourth derived artifact in the PRD system, alongside the rules summary, execution view, and task contexts (see `systems/prd/SYSTEM.md` §3).

---

## When to Use

| Trigger | What happens |
|---|---|
| New orchestration cycle in a project that has no `CONTEXT-PACK.md` | Step 1 — first build |
| `prd-learner` or `build-learner` applied a PRD or ADR amendment | Step 1 — regenerate |
| The most recent ADR or PRD edit timestamp is newer than the pack | Step 1 — regenerate |
| The user says "refresh context" or "the pack is stale" | Step 1 — regenerate |
| A sub-agent reports the pack contradicts current source | Step 1 — regenerate, then re-brief |

---

## Step 1: Gather Sources

Identify the project's contract sources. The exact paths vary by project but the categories are fixed:

| Category | Typical location | What to extract |
|---|---|---|
| PRD contracts | `companies/<company>/prds/PRD-*.md` or the project's PRD path | The requirements and guardrails in scope for the current phase, the AC/scope summary, the current open-questions list |
| Active ADR rules | `.archgate/adrs/*.md` or the project's ADR directory | Decision + rule of each accepted ADR; skip `status: deprecated` |
| Schema digest | `migrations/`, `prisma/schema.prisma`, or equivalent | Table names + key constraints (foreign keys, NOT NULL, RLS policies); not full DDL |
| Phase map | Build-system phase docs (e.g. Al Dente `phases/`) or the PRD's phase plan | The phase index and current cursor |
| Playbook standing rules | `playbook.md` at the project root | Entries that function as standing rules |
| Voice / conventions | Project or company convention files | Only if the build produces text content (skip for pure infra work) |

If a category does not apply to the project (e.g., no ADRs yet), omit the section. Do not invent content to fill it.

---

## Step 2: Compose the Pack

Write to `<project_scope>/CONTEXT-PACK.md`:

```markdown
# Context Pack — <project name>

**Project:** <project_scope>
**Generated:** YYYY-MM-DD HH:MM
**Source revisions:** <git short-sha of HEAD at compile time>

This pack is the curated, current contract set for sub-agents. Read this first. Do not re-read the full PRD or ADR corpus; if you find a contradiction between this pack and source, halt and report — the pack may be stale and needs regenerating.

---

## Frozen contracts (from PRD)

<excerpt: the in-scope requirements and guardrails of the active PRD, verbatim>

## Active ADR rules

| ID | Title | Rule | Source |
|---|---|---|---|
| ARCH-001 | <title> | <one-line rule> | <path> |

(Only `status: accepted` ADRs. Skip deprecated.)

## Schema digest

<table-and-key-constraint summary, not full DDL>

## Phase map

- **Phases:** <list>
- **Current cursor:** <phase id>, <progress>

## Standing rules (from playbook)

- <rule 1, one line>
- <rule 2, one line>

## Voice / conventions

(Include only if the build produces text content.)

<excerpt of relevant rules>

---

## Open questions

<the current Open Questions section of the PRD, verbatim>

---

## What this pack is not

- This is not the full PRD. For full PRD context on a specific feature, the sub-agent should be given the relevant PRD section path in its brief.
- This is not the full ADR set. For the full reasoning behind a rule, the sub-agent reads the cited ADR file.
- This pack does not include in-flight or proposed amendments — only currently-applied state.
```

---

## Step 3: Validate the Pack

Before writing, run these checks:

- Every ADR row in the table has a real `Source` path that exists.
- The Schema digest references real tables.
- The Phase map cursor matches the current phase per `LOOP-STATE.md` (or the project's status file).
- No section was filled with placeholder text. Empty sections are omitted, not stubbed.

If any check fails, fix the input or omit the section. Do not write a pack that cites contracts that do not exist.

---

## Step 4: Write and Confirm

Write to `<project_scope>/CONTEXT-PACK.md`. Report:

```
Context pack regenerated for <project>.
Sections: <list>
Active ADRs: <count>
Schema tables: <count>
Phase cursor: <phase id>
Generated at: <timestamp>
```

If the pack was regenerated mid-cycle (e.g., after a learner apply), notify the orchestrator that any agent currently running with a stale pack should be re-briefed after its current task completes. Running agents are not interrupted; the next spawn uses the fresh pack.

---

## Edge Cases

- **Project has no PRD yet (early phase).** Compose the pack with only the sections that exist. A pack with just standing rules + phase map is still valuable; a one-line "no PRD yet" note documents the gap. This is the modularity guarantee: the loop runs against whatever contracts exist.
- **Two active PRDs in the same project.** The pack is per-active-PRD. If both are active, ask the human which is the current build's contract source.
- **ADR set is large (>30 ADRs).** Include only the rules that currently apply to the active phase. Cite the ADR file path; the pack is a digest, not an archive.
- **Schema digest exceeds 2KB.** Compress further: list only tables touched in the current phase plus foundational tables (auth/users, billing). Cross-reference the full schema file path.
- **Pack would exceed 8KB.** This is a signal the project's frozen contracts are too sprawling. Surface to the human — likely the PRD or ADR set needs consolidation, not the pack.
- **Pack and source disagree (sub-agent flagged it).** Always regenerate. The pack is derived; source wins.
