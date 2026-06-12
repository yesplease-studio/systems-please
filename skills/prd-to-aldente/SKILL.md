---
name: prd-to-aldente
description: Translate a completed Strategic PRD into Al Dente documentation templates for AI-native SaaS building. Use this skill when a PRD is approved and ready to move into the build phase using the Al Dente boilerplate. Trigger when the user says things like "translate to Al Dente", "generate build docs", "prepare for build", "create Al Dente docs from PRD", "populate Al Dente templates", or "bridge PRD to implementation". This skill produces the 7 documentation files that Al Dente's build phases consume.
---

# Skill: prd-to-aldente

**Purpose:** Take a completed Strategic PRD and generate the 7 documentation templates that Al Dente's 11-phase build workflow consumes. This is the bridge between *what to build* (PRD Please) and *how to build it* (Al Dente).

---

## When to Use

- A Strategic PRD has been approved (`status: active`) and the team is ready to begin building with Al Dente.
- A major PRD amendment has been applied and the Al Dente docs need regeneration.
- The user explicitly requests translation from PRD to build documentation.

**Do not use** if the PRD is still in `draft` status — wait for approval first.

---

## Prerequisites

- A completed Strategic PRD (status: active).
- Optionally, the Al Dente repository cloned locally. The skill generates files regardless; Al Dente just needs to be present if writing directly into its `docs/` directory.
- Optionally, the company profile (`COMPANY.md`) for brand and product context that enriches the generated templates.

---

## Workflow

### Step 1: Load and Parse

Read the full Strategic PRD and the company profile (if available). Extract structured data organized by the mapping in Step 2. Parse:

- **Frontmatter** — product name, phase, labels, links.
- **Section 1** — Problem context, background, pain points.
- **Section 2** — Goals, non-goals, success metrics.
- **Section 3** — User roles, user stories, scenarios, buyer/stakeholder journey.
- **Section 4** — Solution overview, components, data flow.
- **Section 5** — All domain requirements (Product, Tech, Data, Design, AI, GTM, General).
- **Section 6** — Guardrails (do's and don'ts).
- **Company profile** — Product context, brand voice, differentiators, technical constraints.

### Step 2: Generate 7 Al Dente Templates

Map PRD content to Al Dente's documentation structure. Each template must be a complete, usable document — not a skeleton with TODOs.

#### Template 1: `content-pages.md`

**Sources:** Section 1 (Context & Problem), Section 2 (Goals & Success Metrics), Section 5 GTM domain, Company profile (product context, differentiators).

**Generate:**
- Product description and value proposition (derived from Section 1 problem statement + Section 2 goals).
- Feature descriptions for marketing pages (derived from Product domain requirements).
- Pricing page structure (derived from GTM requirements — tiers, limits, upgrade triggers).
- Legal pages inventory (privacy, terms — flag as needing legal review).
- Support resources inventory (docs, help center, contact).

**If GTM domain is absent:** Generate product description and feature list only. Mark pricing and positioning sections with `<!-- NEEDS INPUT: No GTM requirements in PRD. Define pricing tiers and messaging here. -->`.

#### Template 2: `journeys.md`

**Sources:** Section 3 (Users & Scenarios, Buyer Journey), Section 5 Product domain.

**Generate:**
- Signup/onboarding flow (derived from user stories tagged with onboarding actions + buyer journey onboarding stage).
- Core task flows (derived from key scenarios — translate each scenario into a step-by-step user flow).
- Upgrade/billing flow (derived from GTM requirements — what triggers an upgrade, what the user sees).
- Support flow (derived from any support-related requirements or user stories).
- Admin workflows (if applicable, derived from admin-related user stories).

**Map user stories to journey steps.** Reference `US-XX` IDs so traceability is maintained.

#### Template 3: `ui-structure.md`

**Sources:** Section 4 (Solution Overview — components, architecture), Section 5 Product domain (user-facing behavior), Section 5 Tech domain (routing, architecture constraints).

**Generate:**
- Route map — public routes (marketing, legal), app routes (authenticated pages), auth routes (login, signup, reset). Derive from Section 4 components + Product requirements.
- Layout definitions — marketing layout, app layout, settings layout. Derive from Section 4 architecture.
- Page inventory — one page per major user-facing requirement in the Product domain.
- Component inventory — identify reusable components from patterns across requirements (e.g., if multiple requirements reference filtering, define a filter component).
- Navigation structure — derive from route map and user flows.

#### Template 4: `data-models.md`

**Sources:** Section 5 Data domain, Section 4 (components and services), Section 5 Tech domain (storage constraints).

**Generate:**
- Entity definitions — one entity per data object referenced in Data or Product requirements. Include fields, types, and constraints.
- Relationships — derive from how entities reference each other in requirements and scenarios.
- Permissions and roles — derive from any access control requirements (Data or Tech domain).
- Events — derive from user actions in scenarios that would generate trackable events.
- API surface assumptions — derive from Tech domain requirements about API patterns.

#### Template 5: `schema-initial.sql`

**Sources:** Section 5 Data domain, Section 5 Tech domain, data-models.md output (from Template 4).

**Generate:**
- `CREATE TABLE` statements for each entity defined in data-models.md.
- Default to PostgreSQL syntax (Supabase default). If Tech requirements specify a different database, use that dialect.
- Include primary keys (UUID), foreign key constraints, indexes on frequently queried fields.
- Include RLS policies if access control requirements exist in the Data domain.
- Include `created_at` and `updated_at` timestamps on all tables.

**This is a starting point, not a production schema.** For entities mentioned in the PRD but not detailed enough to define columns, add: `-- TODO: Define schema for [entity] — see [requirement ID] in PRD`.

#### Template 6: `design-guidelines.md`

**Sources:** Section 5 Design domain, Section 6 Guardrails (design-related), Company profile (brand voice, visual identity).

**Generate:**
- Brand tone and voice (from Company profile or inferred from PRD language).
- Design tokens — colors, typography, spacing. If the Company profile specifies brand colors, use them. Otherwise, provide sensible defaults with `<!-- CUSTOMIZE: Replace with your brand colors -->` markers.
- Component patterns — derive from Design domain requirements (accessibility standards, responsive breakpoints, interaction patterns).
- Layout standards — derive from Design requirements about viewport support, grid systems.
- Accessibility requirements — extract from Design domain (WCAG level, keyboard navigation, screen reader support).

#### Template 7: `assets.md`

**Sources:** Company profile (product name, brand assets), Section 5 Design domain, Section 5 GTM domain.

**Generate:**
- Logo and favicon requirements (from Company profile or flag as needed).
- OG image and social sharing assets (derived from product name and value proposition).
- Empty state illustrations inventory (derived from UI pages that could have empty states).
- Icon library recommendation (based on design system choice).
- Legal content inventory (derived from compliance requirements in Data or General domain).

### Step 3: Apply Defaults (Quick-Start Path)

If the user is using the Al Dente quick-start workflow (or explicitly requests defaults), pre-populate technology decisions in the generated templates:

- **`ui-structure.md`:** React + Vite project structure, Tailwind CSS utility classes, shadcn/ui component library.
- **`schema-initial.sql`:** PostgreSQL with Supabase conventions (RLS policies, `auth.uid()` references).
- **`data-models.md`:** Supabase client patterns for data access.
- **`design-guidelines.md`:** Tailwind design tokens, shadcn/ui component references.

Mark all defaults with: `<!-- Al Dente default — override if your stack differs -->`.

If the user is NOT on the quick-start path, generate stack-agnostic templates and let the user fill in technology choices.

### Step 4: Present for Review

Before writing files, show the user a summary:

```
## Translation Summary — PRD-XXX → Al Dente Docs

Templates generated:
  ✓ content-pages.md — [X items: product description, pricing, legal, support]
  ✓ journeys.md — [X flows: signup, core task, upgrade, ...]
  ✓ ui-structure.md — [X routes, X pages, X components identified]
  ✓ data-models.md — [X entities, X relationships]
  ✓ schema-initial.sql — [X tables, X indexes, X RLS policies]
  ✓ design-guidelines.md — [tokens, patterns, accessibility rules]
  ✓ assets.md — [logo, icons, empty states, legal content]

Thin sections (PRD data was limited):
  - [template]: [what's thin and why]

Assumptions made:
  - [assumption]: [what was assumed and why]

Needs input:
  - [template]: [what the user needs to provide]
```

Wait for the user to review before writing.

### Step 5: Write Output

**Default location:** `outputs/aldente-docs/` within the PRD Please workspace.

**If an Al Dente project path is specified:** Write directly into its `docs/` directory, overwriting existing templates.

**If re-running after a PRD amendment:** Show a diff summary of what changed in each template compared to the previous generation.

---

## What This Skill Does NOT Do

- **Does not execute Al Dente build phases.** It generates documentation; the user (or AI agent) runs the build phases separately.
- **Does not modify the Strategic PRD.** The PRD is the input, never the output. If the translation reveals PRD gaps, flag them for the user to address via `prd-author`.
- **Does not make architectural decisions.** It maps PRD requirements to Al Dente's template structure. Technology choices come from the PRD's Tech requirements or Al Dente's defaults — the skill doesn't opine.
- **Does not install or configure Al Dente.** The user is responsible for cloning and setting up the Al Dente repository.

---

## Quality Standards

- Every generated template must be a **complete, usable document** — a builder reading it should be able to start the corresponding Al Dente phase without needing to reference the original PRD.
- Where PRD data is insufficient for a complete section, generate what's possible and add a clearly marked `<!-- NEEDS INPUT: [specific guidance on what to provide] -->` comment.
- `schema-initial.sql` must be **valid, executable SQL** against the target database.
- `journeys.md` flows must **trace back to user stories** — reference `US-XX` IDs to maintain traceability.
- `ui-structure.md` routes must be **consistent with journeys.md** — every flow should have corresponding routes and pages.
- Templates should be **self-contained** — a reader shouldn't need the PRD to understand what to build. The PRD is the *why*; the templates are the *what to build and how it's structured*.

---

## Token Optimization

Parse the PRD once in Step 1 and map to all 7 templates in a single pass. Don't reload the PRD per template. When re-running after a PRD amendment, load only the changed sections (from the `prd-author` change summary) and regenerate affected templates.
