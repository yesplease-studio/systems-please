---
name: prd-taskmaster
description: Derive executable tasks and client-facing views from a Strategic PRD. Use this skill whenever you need to break a PRD down into buildable work items, generate the initial task backlog for a new engagement, map an incoming client request against the existing PRD, or regenerate task contexts after a PRD amendment. Trigger when the user says things like "generate tasks from the PRD", "what should we build first", "break this into tasks", "a client request just came in", "the PRD changed — update the tasks", or "start the sprint". Always use this skill when the goal is turning a Strategic PRD into actionable, scoped work items.
---

# Skill: prd-taskmaster

**Purpose:** Derive executable tasks and client-facing views from a Strategic PRD. This skill bridges the gap between long-term product truth and short-term buildable work.

---

## When to Use

- After a Strategic PRD is approved (`status: active`) — generate the initial task backlog.
- When a new client request arrives — map it against the PRD and produce a scoped task.
- When the PRD is amended — regenerate affected task contexts.
- When a new phase/release begins — generate the task backlog for that phase.

## Core Responsibilities

1. **Decompose** a Strategic PRD into discrete, executable tasks.
2. **Generate task context** — the minimal payload a coding agent needs to build each task.
3. **Generate the client-facing execution view** — what non-technical clients see on their kanban/platform.
4. **Map incoming requests** — take a client request and determine how it relates to the existing PRD.

---

## Workflow 1: Initial Task Decomposition

### Input

- A Strategic PRD with `status: active`.
- The target phase (e.g. `R1`).

### Step 1: Load and Parse the PRD

Read the full Strategic PRD. Extract:

- All requirements for the target phase from Section 5 (grouped by domain).
- All guardrails from Section 6.
- All user stories from Section 3 for the target phase.
- Any learnings from Section 8.
- Dependencies from frontmatter (`depends_on`).

### Step 2: Decompose into Tasks

Group requirements into **buildable units** — each task should represent work that can be completed and validated independently within 24-48 hours (aligned with InternalOS delivery cadence).

**Shape-check:** When a requirement cluster would produce a task exceeding 48h, surface the split with reasoning before finalizing -- do not just split silently. The reasoning is the teaching.

> "This cluster (TECH-01, TECH-02, TECH-03) maps to roughly 72h. I'd split it into: auth endpoints (T-001, ~24h) and token management (T-002, ~48h). Does that split make sense, or is there a reason to keep them together?"

In expert mode: split without surfacing the reasoning unless the human asks. In standard and teach modes: always name the estimated size and the proposed split before proceeding.

**Decomposition principles:**

- **One task = one deliverable.** A task produces a specific, testable output: an API endpoint, a UI component, a data migration, etc.
- **Respect domain boundaries where practical.** Don't mix frontend and backend work in a single task unless they're trivially coupled.
- **Order by dependency.** If task B requires task A's output, A comes first. Make dependencies explicit.
- **Cluster related requirements.** Multiple requirements that touch the same component can be one task. But don't create mega-tasks — if a task would take more than 48 hours, split it.
- **`must` before `should` before `may`.** Priority within a phase follows severity.

**Batch grouping (for orchestrated builds).** When the build will run as an orchestrated loop (see `systems/prd/SYSTEM.md` §7.4), additionally group tasks into **batches** — the unit the impl → validate → learn cycle operates on. A batch is 3-8 tasks that share a coherent slice of the system and, critically, **do not overlap in the files they touch with any batch that might run in parallel**. Note the expected file footprint per batch; the `prd-agent-brief` skill uses it to derive MAY-WRITE contracts. For non-orchestrated builds, skip batching — tasks go to the backlog individually.

### Step 3: Generate Task Definitions

For each task, produce:

```yaml
task_id: T-001                    # Sequential within the PRD
prd_id: PRD-001                   # Source PRD
title: "Implement user authentication API"
description: |
  Build the authentication endpoints (login, logout, token refresh)
  with JWT-based session management.
priority: must                     # Inherited from highest-severity requirement in the task
phase: R1
estimated_effort: small | medium | large
dependencies:
  - T-000                          # Task dependencies (if any)
requirements:                      # Specific requirement IDs this task fulfills
  - TECH-01
  - TECH-02
  - PR-03
user_stories:
  - US-01
  - US-04
acceptance_criteria:               # Pulled from referenced requirements
  - "POST /auth/login returns JWT token on valid credentials."
  - "POST /auth/login returns 401 on invalid credentials."
  - "Token expires after 24 hours."
  - "POST /auth/refresh extends token by 24 hours."
guardrails:                        # Only the guardrails relevant to this task
  dos:
    - "DO use bcrypt for password hashing with cost factor 12."
    - "DO validate all input with schema validation before processing."
  donts:
    - "DON'T store plain-text passwords."
    - "DON'T expose internal error details in API responses."
relevant_learnings:                # Any learning entries that affect this task
  - L-001: "Rate limiting must be applied to auth endpoints (added after brute force incident)."
adr_candidate: false               # true if this task contains requirements that warrant an ADR
adr_note: ""                       # if adr_candidate: true, briefly describe what the ADR should cover
```

**ADR candidate detection:** when generating task definitions, set `adr_candidate: true` and populate `adr_note` if the task contains TECH-domain requirements that meet one or more of these criteria:
- Constrains a structural or cross-cutting concern (auth pattern, data model shape, API conventions, dependency policy, error handling strategy)
- Would be expensive to reverse mid-build
- Is likely to be violated accidentally by an AI agent without explicit enforcement

Typical ADR candidates: auth mechanism choice, ORM/query pattern, API versioning strategy, session management approach, file structure conventions, third-party service selection with lock-in implications.

Typical non-candidates: implementation details within an already-decided framework, UI component choices, copy or styling decisions.

When `adr_candidate: true` tasks are present in the backlog, note them in the Step 5 summary: "X task(s) flagged as ADR candidates — before building begins, check `archgate adr import --list` (or the [awesome-adrs registry](https://github.com/archgate/awesome-adrs)) for a covering pack, then `archgate adr create` for anything left uncovered." This is advisory — the human decides whether to create or import ADRs.

### Step 4: Generate Execution View (Client-Facing)

For each task, generate a **plain-language summary** suitable for non-technical clients:

```markdown
**Build login system**
We're setting up secure user login — users will be able to sign in with email and password, stay signed in for 24 hours, and extend their session. This covers the foundation that all other user-facing features will build on.

Status: Ready to build
Estimated delivery: 24-48 hours
```

**Execution view rules:**

- No jargon. No technical terms unless the client uses them.
- Explain **what they'll see**, not how it works internally.
- Tie back to a goal or user scenario from the PRD when helpful.
- Keep each summary to 2-4 sentences.

### Step 5: Present for Approval

Show the human the complete task backlog:

- Task list with priorities and dependencies.
- Execution view summaries.
- Any requirements that couldn't be cleanly decomposed (flag for discussion).
- Any open questions (`OQ-XX`) that block specific tasks.
- If any tasks have `adr_candidate: true`: note them with a brief summary — "Before building begins, consider creating ADRs for: [list with adr_note for each]."

Wait for approval before tasks go to the kanban.

### Step 6: Companion Doc and Playbook

After the backlog is approved, generate two artifacts. Do not wait for the human to ask.

**Companion doc:** Create `PRD-XXX-taskmaster-companion.md` in the same directory as the task backlog, using `templates/taskmaster-companion.md` as the base. Fill each section with content specific to this decomposition: name the actual grouping decisions, the actual dependency chains, the actual ADR candidates flagged. A companion doc that could apply to any backlog is not useful.

**Playbook entry:** Append one entry to `playbook.md` at the project root. Create the file from `templates/playbook.md` if it does not exist.

```
## [date] PRD-XXX tasks (Phase [R1/R2/...])
Decomposition pattern: [how requirements were grouped -- by component, by domain, by dependency order]
ADR candidates flagged: [yes/no; if yes, what]
Scope boundary: [one clean line about where the task edges reveal the PRD is thin or strong]
```

**Mode behavior:**
- Teach mode: surface the companion doc, mention where it lives, note the decomposition move that will transfer best to future work.
- Standard mode: mention that the companion doc was generated and name its path.
- Expert mode: generate both silently.

---

## Workflow 2: Mapping an Incoming Client Request

### Input

- A client request (free-form text from kanban, message, email, etc.).
- The existing Strategic PRD(s) for the product.

### Step 1: Interpret the Request

Parse the client's request and identify:

- What they want (feature, fix, change, enhancement).
- Which part of the product it relates to.
- Any constraints or preferences they've mentioned.

### Step 2: Map Against the PRD

Compare the request to the existing Strategic PRD:

| Outcome | Action |
|---------|--------|
| **Fits existing requirements** | Generate a task that references the relevant requirement IDs. Proceed normally. |
| **Extends existing scope** | Flag as a scope addition. Draft a proposed PRD amendment (new requirements) for the human to review. If approved, invoke `prd-author` in edit mode, then generate the task. |
| **Conflicts with existing requirements** | Flag the conflict. Show the client request alongside the conflicting requirement. Escalate to human for decision. Human may: (a) invoke `prd-author` to amend the PRD, after which `prd-taskmaster` re-maps the request; (b) reject the request; or (c) override the conflict with documented reasoning. |
| **Entirely new scope** | Flag as out-of-scope for the current PRD. Recommend whether it should be a PRD amendment or a new PRD. Escalate to human. |
| **Blocked by open question** | Identify which `OQ-XX` blocks this. Notify the human that the open question needs resolution before the task can be created. |

### Step 3: Generate Task (if applicable)

If the request maps to existing scope, generate the task definition and execution view following the same format as Workflow 1.

If the request needs clarification, use `AskUserQuestion` with specific questions — don't generate a task from ambiguous input.

**Escalation routing:** when the project runs the build loop (`LOOP-STATE.md` exists), append scope conflicts, out-of-scope flags, and open-question blockers to the loop's decision queue (via the `prd-loop` skill) instead of interrupting one at a time. The `prd-decision-batch` skill packages them for review. Decisions that block the *current* batch still surface immediately.

---

## Workflow 3: Regenerating Task Contexts After PRD Amendment

### Input

- A change summary from `prd-author` (listing which sections/requirements changed).

### Step 1: Identify Affected Tasks

Map the changed requirements and guardrails to existing tasks. A task is affected if:

- Any of its referenced requirement IDs were modified.
- Any of its applicable guardrails were added, removed, or changed.
- A new learning entry affects its domain.

### Step 2: Regenerate

For each affected task, regenerate the task definition and execution view from the updated PRD. Highlight what changed.

### Step 3: Notify

Produce a brief summary of regenerated tasks:

```
Tasks updated after PRD-001 amendment:
- T-003: Updated acceptance criteria for TECH-01 (rate limiting added).
- T-005: New guardrail applied (DON'T expose stack traces).
- T-008: NEW task created for FR-12 (new requirement added to PRD).
```

---

## External Backlog Integration

Task definitions produced by this skill are **deliberately tool-agnostic** — they are structured YAML/Markdown designed to be readable by humans and machines alike. This makes them straightforward to push into any project management system.

### Recommended integration patterns

**Via MCP (recommended):** When working in Claude Code or another MCP-capable agent environment, use the relevant MCP server (Linear, JIRA, GitHub Issues, etc.) to send task definitions directly to your team's backlog after this skill produces them. The task definition fields map naturally:

| Task context field | Linear | JIRA | GitHub Issues |
|---|---|---|---|
| `title` | Issue title | Summary | Title |
| `description` + `acceptance_criteria` | Description | Description | Body |
| `priority` (`must`/`should`/`may`) | Priority | Priority | Labels |
| `estimated_effort` | Estimate | Story points | Labels |
| `dependencies` | Blocked by | Linked issues | Linked PRs |
| `requirements` | References | References | Body reference |

**Via agent instruction:** After this skill produces the task backlog, instruct your agent: *"Take these task definitions and create them in [Linear/JIRA/GitHub] using the MCP server."* The structured output is designed for this — no reformatting needed.

**Manual copy:** For teams without MCP integration, the task definitions and execution view are copy-paste ready for any kanban tool.

### What this skill does NOT do

This skill does not push tasks to any external system directly. It produces the artifact; the agent or human decides where it goes. This keeps the PRD system platform-agnostic — the same output works for Linear, JIRA, Notion, a spreadsheet, or a physical board.

---

## Token Optimization Notes

This skill is called frequently — every new task, every client request, every PRD amendment. Token efficiency matters here more than anywhere else in the system.

**Optimization strategies:**

1. **Load selectively.** When mapping a client request, load only the PRD frontmatter and Section 5 (requirements) first. Only load other sections if needed for disambiguation.

2. **Use sub-agents for formatting.** The decomposition logic (grouping requirements into tasks, determining dependencies) requires judgment and belongs in the orchestrating context. Generating the execution view summaries and YAML task definitions from an already-determined decomposition is mechanical — delegate it to the smallest model that formats reliably.

3. **Cache task contexts.** Task contexts only need regeneration when the PRD changes. Between amendments, they're stable. For orchestrated builds, the project-level `CONTEXT-PACK.md` (see `prd-context-pack`) serves the same role one level up: agents read the pack plus their task context, never the full corpus.

4. **Minimal task context payloads.** The whole point of task contexts is to avoid loading the full PRD per-task. A task context should typically be 30-60 lines of YAML — enough to build from, small enough to fit in any context window alongside the codebase.
