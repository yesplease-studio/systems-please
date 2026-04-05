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
| `prd-discovery` | Run a structured pre-authoring interview to surface what is known, inferred, and unknown before writing a PRD |
| `prd-author` | Create and edit Strategic PRDs from human-provided context |
| `prd-taskmaster` | Derive executable tasks and client-facing views from a Strategic PRD |
| `prd-validator` | Validate implementation against PRD requirements and guardrails |
| `prd-learner` | Capture implementation learnings and propose PRD amendments |
| `prd-to-aldente` | Translate a Strategic PRD into Al Dente build documentation |
| `build-learner` | Capture build-phase learnings and propose amendments upstream or laterally |

### Workflows

Multi-step skill sequences defined in YAML. They chain skills together with conditional steps and human approval gates.

| Workflow | What it does |
|----------|-------------|
| `prd-new-engagement` | Author a PRD, get approval, generate tasks |
| `prd-post-build` | Validate, learn, amend, regenerate tasks |
| `prd-aldente-quickstart` | Author a PRD with Al Dente defaults, translate to build docs, generate tasks |

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
archgate create  Codify architectural decisions as ADRs (.archgate/adrs/)   [Archgate]
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
3. Initialize in your project directory: `archgate init`
4. When `prd-taskmaster` flags requirements as ADR candidates (see [PRD → ADR](#prd--adr-bridge) below), create companion ADRs in `.archgate/adrs/`.
5. Wire `archgate check` into CI to enforce decisions automatically.

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

1. Run `archgate create` to start a new ADR.
2. Fill in the Context (why the decision matters), Decision (what was decided), and Do's and Don'ts from the PRD requirement.
3. Optionally add a companion `.rules.ts` file to make the decision machine-checkable.

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

### 2. Set up your company profile

```bash
cp -R companies/_template companies/your-company
```

Edit `companies/your-company/COMPANY.md` with your product context. The template has detailed guidance for each section.

### 3. Configure your workspace

```bash
cp CLAUDE.md.template CLAUDE.md
```

Edit `CLAUDE.md` to set your active company:

```yaml
company: your-company
profile: companies/your-company/COMPANY.md
```

### 4. Start using skills

Open Claude Code in the repo directory and use skills directly:

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
  prd-discovery/     Pre-authoring interview — produces a Discovery Brief
  prd-author/        Create and edit Strategic PRDs
  prd-taskmaster/    Derive tasks from PRDs
  prd-validator/     Validate implementation against PRDs
  prd-learner/       Capture learnings and amend PRDs
  prd-to-aldente/    Translate PRDs into Al Dente build docs
  build-learner/     Capture build-phase learnings

workflows/         Multi-step YAML sequences
  product/           PRD workflows + Al Dente integration

companies/         Company profiles
  _template/         Scaffold for new companies

examples/          Fictional example engagement
  acme-analytics/    Complete PRD lifecycle demo

deploy/            Setup templates for new workspaces
docs/              Architecture and schema reference
```

## Contributing

See [CONTRIBUTING.md](CONTRIBUTING.md) for guidelines on issues, pull requests, and methodology changes.

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

Each tool works standalone. Fork it, open Claude Code, start working.

---

*PRD Please is created and maintained by [Eric Stein-Beldring](https://linkedin.com/in/ericsteinbeldring) at [Yes Please Studio](https://yesplease.studio).*
