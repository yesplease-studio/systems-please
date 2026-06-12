<img width="1584" height="396" alt="Github-prd-please" src="https://github.com/user-attachments/assets/130bfce9-a0c2-42ee-82e3-24de7287f7ea" />

# PRD Please

A structured methodology for AI-native product requirements. Write, decompose, validate, and learn from Strategic PRDs — designed for [Claude Code](https://docs.anthropic.com/en/docs/claude-code).

Built and maintained by [Yes Please Studio](https://yesplease.studio).

---

## The problem

AI coding agents are powerful, but they produce inconsistent results without structured requirements. "Build me an app" gets you an app — just not the one you needed. The gap isn't intelligence, it's specification.

PRD Please fills that gap. It gives you a repeatable methodology for translating human intent into artifacts that AI agents can execute reliably — and a learning loop that makes each engagement better than the last.

## What's included

### PRD system

A structured approach to writing, evolving, and acting on product requirements. It solves three problems:

1. **Distillation.** Converts unstructured human context into a precise, structured source of truth (the Strategic PRD).
2. **Operationalization.** Derives executable tasks from the source of truth so building can begin within hours, not days.
3. **Continuous learning.** Feeds implementation experience back into the source of truth so the same mistakes are never repeated.

### Skills

Executable single-step workflows. Each has a `SKILL.md` that Claude reads before executing.

| Skill | What it does |
|-------|-------------|
| `prd-onboard` | Set up prd-please for your company. Walks the COMPANY.md schema interactively, supports an in-repo profile or an external pointer, refers you to sibling please-family tools when a question is upstream of PRD scope |
| `prd-discovery` | Run a structured pre-authoring interview to surface what is known, inferred, and unknown before writing a PRD |
| `prd-author` | Create and edit Strategic PRDs from human-provided context |
| `prd-taskmaster` | Derive executable tasks, batches, and client-facing views from a Strategic PRD |
| `prd-validator` | Adversarially validate implementation against PRD requirements and guardrails — executes typechecks, builds, and tests rather than trusting self-reports |
| `prd-learner` | Capture implementation learnings and propose PRD amendments |
| `prd-loop` | Manage the build loop's externalized state (`LOOP-STATE.md`): batch pointer, debt counters, decision queue, workaround signatures |
| `prd-context-pack` | Compile the project's frozen contracts into the single `CONTEXT-PACK.md` sub-agents read instead of the full corpus |
| `prd-agent-brief` | Compose the invocation contract for impl/validator/learner sub-agents: file ownership, token budgets, report format |
| `prd-decision-batch` | Replace one-at-a-time interruptions with a single ranked decision package |
| `prd-to-aldente` | Translate a Strategic PRD into Al Dente build documentation |
| `build-learner` | Capture build-phase learnings and propose amendments upstream or laterally |

### Workflows

Multi-step skill sequences defined in YAML. They chain skills together with conditional steps and human approval gates.

| Workflow | What it does |
|----------|-------------|
| `prd-new-engagement` | Author a PRD, get approval, generate tasks |
| `prd-post-build` | Validate, learn, amend, regenerate tasks |
| `prd-build-loop` | One batch cycle of the orchestrated build loop: impl → validate → learn → apply → commit |
| `prd-aldente-quickstart` | Author a PRD with Al Dente defaults, translate to build docs, generate tasks |

### Execution loop

For builds executed by AI sub-agents, prd-please ships an orchestration layer that runs the **impl → validate → learn → apply → commit** cycle batch by batch. The specification skills define *what* must be true; the loop is *how* agents build against it without drifting, colliding, or silently failing.

The mechanics that make it safe:

- **Externalized state.** `LOOP-STATE.md` holds the batch pointer, debt counters, fan-out registry, decision queue, and workaround signatures — the loop survives session boundaries because none of it lives in conversation context.
- **Role-typed agents.** Three agent definitions ship in `.claude/agents/`: `impl` (writes within its declared contract), `validator` (read-only against source, executes build/typecheck/tests, adversarial by design), and `learner` (read-only, proposal-only). Claude Code picks them up automatically on clone.
- **File-ownership contracts.** Every spawn declares MAY-WRITE / MAY-NOT-TOUCH lists, cross-checked against parallel agents before briefing — this is what makes fan-out safe.
- **Honest degradation.** Agents that can't cross a seam degrade visibly (partial-by-design with a documented contract). Faked success is a hard fail, caught by the validator.
- **Batched decisions.** Escalations queue in loop state and surface as one ranked package instead of a stream of interruptions.
- **Workaround signatures.** The same workaround appearing three times triggers a generalizing-ADR proposal — the runtime complement to `prd-taskmaster`'s `adr_candidate` flag.

The loop is modular: its contract source can be a Strategic PRD, an ADR set, a playbook, or any combination. Pairing it with a PRD closes the full learning circuit (impl notes → `prd-learner` → PRD amendment → regenerated context pack), but it runs without one. Full methodology: `systems/prd/SYSTEM.md` §7.4.

### Learning layer

prd-please is built to make you better at product specification, not just faster at producing documents. Every skill run does three things beyond generating the artifact:

**Teach-through-choice questions.** Every interview question includes a one-line rationale so you understand why it matters -- not just what to answer. Representative examples show what good looks like by contrast. Over time, you start asking these questions yourself before the skill does.

**Shape-check tutoring.** When you write something the skill can assess -- an outline, a task cluster, a problem statement -- it responds with a brief evaluation before proceeding. "This goal has no metric. Add one or flag it as qualitative -- either is valid, but 'improve the experience' won't tell you when you're done." Tutoring, not linting.

**Companion doc.** Alongside every artifact, the skill generates a `<artifact>-companion.md` with four sections: why the output sounds the way it does, why it is structured this way, which moves are worth reusing, and how to do it yourself next time without the skill.

**Playbook accretion.** Every companion doc contributes one entry to `playbook.md` at your project root. After ten sessions, you have a dozen reusable frames specific to your product and team -- decomposition patterns that worked, scope boundaries you had to learn the hard way, ICP distinctions that made user stories sharper.

The mode selector adjusts verbosity: **teach** (first-time users, full rationale inline), **standard** (returning users, rationale on request), or **expert** (declare it in conversation, companion docs generate silently). Mode is detected automatically from playbook state; no configuration needed.

### Company profiles

Structured context files (product, users, constraints) that skills load before executing. Your company profile is what makes generic skills produce project-specific output.

## How it connects

### Within PRD Please

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

### Full toolchain (PRD → governance → build)

```
prd-discovery    Surface what's known, inferred, and unknown before authoring
    ↓
prd-author       Translate human intent into a structured Strategic PRD
    ↓
prd-taskmaster   Derive executable tasks — flags TECH requirements as ADR candidates
    ↓
archgate adr     Codify architectural decisions as ADRs (.archgate/adrs/)   [Archgate]
   create/import   — author from scratch, or import a curated pack from the registry
    ↓
Al Dente         Phased SaaS build — tasks from prd-taskmaster drive each phase
    ↓
archgate check   Enforce ADRs in CI — blocks merges on architectural violations [Archgate]
    ↓
prd-validator    Validate implementation against PRD requirements
    ↓
prd-learner      Capture learnings — propose amendments back to the Strategic PRD
```

PRD Please handles the *specification* layer (authoring, task derivation, validation, learning). [Archgate](https://github.com/archgate/cli) handles the *governance* layer (architectural decisions as enforceable rules). [Al Dente](https://github.com/aline-no/aldente) handles the *build* layer (phased SaaS implementation).

## Setup paths

### Path A: Standalone (PRD only)

Use the PRD methodology on its own for any product or project. No build system dependency. Follow the quick start below.

### Path B: PRD + Al Dente (SaaS build integration)

Combine PRD Please with [Al Dente](https://github.com/aline-no/aldente) for a full pipeline from requirements to SaaS implementation. PRD Please handles *what to build*; Al Dente handles *how to build it*.

1. Follow the quick start below to set up PRD Please.
2. Clone Al Dente alongside: `git clone https://github.com/aline-no/aldente.git`
3. Use the combined workflow: run `/prd-aldente-quickstart` to author a PRD with Al Dente's default stack (React + Vite + Tailwind + Supabase + Stripe), translate it into Al Dente's build docs, and generate your task backlog — all in one flow.

You can start with Path A and add Al Dente later. The `prd-to-aldente` skill can translate any existing PRD into Al Dente docs at any time.

### Path C: PRD + Archgate (architecture governance)

Add [Archgate](https://github.com/archgate/cli) to enforce architectural decisions made during PRD work throughout implementation.

PRD Please defines *what to build* and *what constraints apply*. Archgate turns technical constraints and architectural decisions into ADRs — machine-checkable rules that run in CI and feed live context to AI coding agents.

1. Follow the quick start below to set up PRD Please.
2. Install Archgate: `npm install -g archgate` (or `bun install -g archgate`)
3. Initialize in your project directory: `archgate init`. The greenfield wizard detects your stack and offers to import recommended starter packs from the [awesome-adrs registry](https://github.com/archgate/awesome-adrs) — accept the ones that match your stack to skip writing baseline ADRs from scratch.
4. When `prd-taskmaster` flags requirements as ADR candidates (see [PRD → ADR](#prd--adr-bridge) below), either import a covering pack with `archgate adr import` or author a new ADR in `.archgate/adrs/` with `archgate adr create`.
5. Wire `archgate check` into CI to enforce decisions automatically. Run `archgate adr sync` periodically to pull updates for any imported packs.

Paths B and C compose: use PRD Please + Archgate + Al Dente together for the full pipeline from requirements to governed SaaS implementation.

```
PRD authoring (PRD Please)
    ↓
Architecture governance (Archgate)   ← codify TECH-domain decisions as ADRs
    ↓
Build (Al Dente)                      ← ADR rules enforce decisions in CI
```

### PRD → ADR bridge

Not every TECH-domain requirement needs an ADR. The signal is whether a decision is durable, has downstream consequences, and needs to be enforced — not just documented.

`prd-taskmaster` flags requirements that meet this threshold when deriving tasks. Look for the `adr_candidate: true` flag in task definitions. When you see it:

1. Check the [awesome-adrs registry](https://github.com/archgate/awesome-adrs) for a pack that already covers the decision. If one exists, `archgate adr import <pack>` brings it in (IDs are remapped to your project's numbering automatically).
2. Otherwise, run `archgate adr create` to start a new ADR.
3. Fill in the Context (why the decision matters), Decision (what was decided), and Do's and Don'ts from the PRD requirement.
4. Optionally add a companion `.rules.ts` file to make the decision machine-checkable.

A requirement worth an ADR typically has one or more of these properties:
- Constrains a structural or cross-cutting concern (auth pattern, data model, API conventions, dependency choices)
- Would be expensive to reverse mid-build
- Is likely to be violated accidentally by an AI agent without explicit enforcement

---

## Quick start

### 1. Clone the repo

```bash
git clone https://github.com/yesplease-studio/prd-please.git
cd prd-please
```

### 2. Run onboarding

Open Claude Code in the repo directory and run:

```
/prd-onboard
```

`prd-onboard` walks you through the COMPANY.md schema section by section, sets up `CLAUDE.md`, and refers you to sibling please-family tools when a question is upstream of PRD scope (e.g. ICP definition, voice rules). It supports two patterns:

- **In-repo COMPANY.md** — the default. Creates `companies/<your-company>/COMPANY.md` from the template and walks you through populating it.
- **External pointer** — for users who already maintain company context in a private internal repo or vault. Points `CLAUDE.md` at that file instead of duplicating it.

You can also set up manually if you prefer: copy `companies/_template/` to `companies/<your-company>/`, edit the COMPANY.md, then copy `CLAUDE.md.template` to `CLAUDE.md` and set the `company:` and `profile:` fields. The schema is documented in [docs/company-schema.md](docs/company-schema.md).

### 3. Start using skills

Once onboarding is complete, use skills directly:

```
/prd-discovery       Run a pre-authoring interview before writing a new PRD
/prd-author          Create a Strategic PRD for your product
/prd-taskmaster      Break a PRD into executable tasks
/prd-validator       Validate implementation against PRD requirements
/prd-learner         Capture learnings and propose amendments
```

Or use natural language:

- "I have a rough idea — run discovery before we write the PRD"
- "Write a PRD for the new onboarding flow"
- "Break PRD-001 into tasks for R1"
- "Validate this code against the PRD"
- "Capture what we learned from this sprint"

### 5. See it in action

Check `examples/acme-analytics/` for a complete fictional engagement showing the PRD lifecycle: company profile, authored PRD, and learnings.

## Directory layout

```
systems/           Methodology definitions
  prd/               PRD system

skills/            Executable workflows (one SKILL.md each)
  prd-onboard/       Interactive setup — populates COMPANY.md and wires CLAUDE.md
  prd-discovery/     Pre-authoring interview — produces a Discovery Brief
  prd-author/        Create and edit Strategic PRDs
  prd-taskmaster/    Derive tasks and batches from PRDs
  prd-validator/     Adversarially validate implementation against PRDs
  prd-learner/       Capture learnings and amend PRDs
  prd-loop/          Manage build-loop state (LOOP-STATE.md)
  prd-context-pack/  Compile frozen contracts into CONTEXT-PACK.md
  prd-agent-brief/   Compose sub-agent invocation contracts
  prd-decision-batch/ Emit ranked decision packages
  prd-to-aldente/    Translate PRDs into Al Dente build docs
  build-learner/     Capture build-phase learnings

.claude/agents/    Role-typed sub-agent definitions for the build loop
  impl.md            Implementation agent — writes within its file-ownership contract
  validator.md       Validator agent — read-only, executes tests, adversarial
  learner.md         Learner agent — read-only, proposal-only

workflows/         Multi-step YAML sequences
  product/           PRD workflows + Al Dente integration

companies/         Company profiles
  _template/         Scaffold for new companies

examples/          Fictional example engagement
  acme-analytics/    Complete PRD lifecycle demo

templates/         Output templates for companion docs and playbook
  prd-companion.md       Companion doc for PRD artifacts
  taskmaster-companion.md  Companion doc for task backlog artifacts
  company-companion.md   Companion doc for COMPANY.md
  playbook.md            Playbook entry format

deploy/            Setup templates for new workspaces
docs/              Architecture and schema reference
```

## Contributing

See [CONTRIBUTING.md](open-source/please-family/prd-please/CONTRIBUTING.md) for guidelines on issues, pull requests, and methodology changes.

## License

Apache 2.0 — see [LICENSE](LICENSE). Attribution required.

---

## The "-please" family

Open-source, AI-native tools for strategic and product work by [Yes Please Studio](https://yesplease.studio):

| Tool | What it does |
|------|-------------|
| **[strategy-please](https://github.com/yesplease-studio/strategy-please)** | Maps challenges to the right workshop using a strategic maturity framework |
| **prd-please** (this repo) | Structured methodology for AI-native product requirements |
| **[sales-please](https://github.com/yesplease-studio/sales-please)** | Lightweight deal qualification framework built on WORTH |
| **[voice-please](https://github.com/yesplease-studio/voice-please)** | Defines, encodes, and maintains a company's voice and language system |
| **[design-please](https://github.com/yesplease-studio/design-please)** | Scopes design direction and encodes it for Claude Design or designer handoff |
| **[name-please](https://github.com/yesplease-studio/name-please)** | Validates and generates company and product names grounded in ICP and voice context |
| **[messaging-please](https://github.com/yesplease-studio/messaging-please)** | Builds messaging architecture -- purpose, positioning, and per-audience message matrix |

Each tool works standalone. Fork it, open Claude Code, start working.

---

*PRD Please is created and maintained by [Eric Stein-Beldring](https://linkedin.com/in/ericsteinbeldring) at [Yes Please Studio](https://yesplease.studio).*
