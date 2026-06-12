---
name: prd-loop
description: Read, write, and update a project's orchestration loop state at `<project_scope>/LOOP-STATE.md` — current batch, decision queue, fan-out registry, debt counters, and workaround signatures. Use this skill at the start of any orchestrated build cycle, whenever another execution-loop skill needs the state file, or when the user asks where the build stands. Trigger when the user says things like "loop state", "where are we in the loop", "update loop state", "log a decision", "log a workaround", or "initialize the build loop". The loop state externalizes the impl → validate → learn → apply → commit cycle so it survives session boundaries.
---

# Skill: prd-loop

**Purpose:** Externalize the impl → validate → learn → apply → commit loop so the orchestrator does not have to hold it in conversation context. The state file survives session boundaries; other execution-loop skills (`prd-agent-brief`, `prd-validator`, `prd-learner`, `prd-context-pack`, `prd-decision-batch`) read and update it.

The format is small on purpose. Add a section only when a real signal requires it.

**Project scope:** `<project_scope>` is wherever the build's coordination files live — typically the root of the repository being built, or a project directory alongside the PRD. All loop artifacts live there: `LOOP-STATE.md`, `CONTEXT-PACK.md`, `validation/`, `learnings/`, `decisions/`.

---

## When to Use

| Trigger | What happens |
|---|---|
| Starting a new orchestration cycle in a project that has no `LOOP-STATE.md` | Step 1 — initialize |
| `prd-agent-brief`, `prd-validator`, `prd-learner`, `prd-context-pack`, or `prd-decision-batch` needs to read state | Step 2 — read and return the relevant section |
| A validated batch finished, a decision was made, a workaround was applied, or a fan-out was opened/closed | Step 3 — update the relevant section |
| The user asks "where are we" or "what's the loop state" | Step 2 — read and summarize |

---

## Step 1: Initialize (only if the file does not exist)

Confirm the project scope. If unclear, ask.

Create `LOOP-STATE.md` at that path with this skeleton:

```markdown
# Loop State — <project name>

**Project:** <project_scope>
**Contract source:** <path to the active PRD, ADR directory, and/or playbook>
**Started:** YYYY-MM-DD
**Last update:** YYYY-MM-DD

---

## Batch pointer

- **Current:** <batch id, e.g. T-001..T-006>
- **Phase:** <impl | validate | learn | apply | commit>
- **Started:** YYYY-MM-DD

## Debt counters

- **Learner debt:** 0 (validated batches not yet processed by the learner)
- **Apply debt:** 0 (learner proposals not yet applied)
- **Ceiling:** 2 (hard stop on fan-out if combined debt exceeds this)

## Fan-out registry

| Agent | Role | Phase | File ownership | Status | Started |
|---|---|---|---|---|---|

## Decision queue

| ID | Decision | Recommended | Unblocks | Logged |
|---|---|---|---|---|

## Workaround signatures

| Signature | Count | First seen | Last seen | Generalizing ADR proposed? |
|---|---|---|---|---|

## Recent batches

| Batch | Tasks | Validated | Committed | Learner applied |
|---|---|---|---|---|
```

The `Contract source` line is what makes the loop modular: it can point at a Strategic PRD, an ADR directory, a playbook, or any combination. A PRD is the richest contract source, but the loop runs against whatever contracts the project has.

Write the file. Report the path and that the loop is initialized.

---

## Step 2: Read

When another skill needs state, return only the section asked for, not the whole file.

Common queries:

- **Current batch + phase** → return the Batch pointer block.
- **Can I fan out a new sub-agent?** → check Debt counters. If `learner debt + apply debt >= ceiling`, return `BLOCKED — drain debt first` with the counts. Otherwise return `OK` with the current fan-out count from the registry.
- **Open decisions** → return the Decision queue rows.
- **Has this workaround signature appeared before?** → match against the Workaround signatures table. If found, return the current count and whether a generalizing ADR has been proposed. If `count >= 3` and no ADR proposed, return `ESCALATE — propose generalizing ADR`.

---

## Step 3: Update

Mutations are surgical edits to the relevant section. Always update `Last update:` at the top.

### 3a. Batch pointer

On phase transition (impl → validate → learn → apply → commit → next), update the Batch pointer block. Append the just-finished batch to Recent batches.

### 3b. Debt counters

- Increment `Learner debt` when a validated batch lands without a learner run.
- Decrement it when the learner runs on that batch. Increment `Apply debt` for proposals not yet applied.
- Decrement `Apply debt` when proposals are applied.

If either counter exceeds the ceiling, surface this to the orchestrator before the next fan-out. The ceiling exists because unprocessed learnings are exactly the mistakes the next batch will repeat.

### 3c. Fan-out registry

Append a row when a sub-agent is spawned, with its MAY-WRITE list (compressed). Update Status when it returns (`done` | `blocked` | `killed`). Drop rows after commit to keep the table scannable.

### 3d. Decision queue

Append a row when the orchestrator surfaces a decision that needs the human. Remove the row when decided. Format:

```
| OQ-N | <one-line decision> | <recommended option + one-line why> | <what it unblocks> | YYYY-MM-DD |
```

Use the PRD's open-question IDs (`OQ-XX`) when the decision originates from the PRD; continue the sequence for decisions that surface during the build. The `prd-decision-batch` skill consumes this queue.

### 3e. Workaround signatures

Append a row when the same shape of workaround is applied a second time (the first one looks like a one-off; the second is the signal). Format:

```
| <signature, e.g. "frozen-RLS + additive SECURITY DEFINER RPC"> | 2 | YYYY-MM-DD | YYYY-MM-DD | no |
```

Increment Count on each subsequent occurrence. When `Count >= 3` and no ADR has been proposed, add a row to the Decision queue: `OQ-N | Propose generalizing ADR for <signature> | <recommended scope> | unblocks future <pattern> work | YYYY-MM-DD`.

This is the runtime complement to `prd-taskmaster`'s `adr_candidate` flag: taskmaster predicts which decisions need enforcement before the build; signature tracking catches the ones that only reveal themselves during it.

---

## Step 4: Confirm

After any update, report a one-line summary to the orchestrator:

```
Loop state: batch <id> phase <phase>, debt L=<n> A=<n>, fan-out <n active>, decisions <n open>, signatures <n tracked>.
```

If a threshold was crossed (debt ceiling, signature count), include the surface line: `→ ESCALATE: <what>`.

---

## Edge Cases

- **No project scope identified.** The loop state is per-project. If invoked without a clear project scope, ask before creating files at the wrong path.
- **Stale state from an abandoned session.** If `Last update:` is more than 14 days old and the orchestrator is starting fresh, summarize the current state and ask whether to keep, archive, or reset.
- **Parallel cycles in two projects.** Each project has its own `LOOP-STATE.md`. Do not merge.
- **Debt counter looks wrong.** If counters disagree with what the orchestrator believes about the latest batch, trust the file and ask. The file is canonical; conversation memory is not.
