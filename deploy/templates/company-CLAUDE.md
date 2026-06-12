# {{COMPANY_NAME}} — PRD Please

Always use and write in American English.

## Company profile

Your company context (product, users, constraints) is in `COMPANY.md` at the root of this repository. All skills reference this file automatically.

## Before executing any skill

1. Read `COMPANY.md` for company context.
2. Read `systems/prd/SYSTEM.md` for the PRD methodology when authoring or reasoning about requirements.

## Available skills

### PRD System
- **prd-author** — Create and edit Strategic PRDs from human-provided context
- **prd-taskmaster** — Derive executable tasks and client-facing views from a Strategic PRD
- **prd-validator** — Validate implementation against PRD requirements and guardrails
- **prd-learner** — Capture implementation learnings and propose PRD amendments

### Integration (optional)
- **prd-to-aldente** — Translate a Strategic PRD into Al Dente build documentation
- **build-learner** — Capture build-phase learnings and propose amendments upstream or laterally

## Workflows

Check `workflows/product/` for multi-step PRD sequences (new engagement, post-build validation, Al Dente quick-start).

## Improving skills

When a skill produces suboptimal output:
1. Give feedback on what should change.
2. Claude will identify whether the issue is in the skill, the system definition, or the company profile.
3. Approve the proposed edit.
4. Commit with a clear message.

Your Git history tells the story of how your system improves over time.
