# Contributing to PRD Please

Thank you for your interest in improving PRD Please. This project is maintained by [Yes Please Studio](https://yesplease.studio) and welcomes contributions from the community.

## What's welcome

- **Bug reports** — A skill doesn't work as described, a workflow has a broken step, a template is missing a field.
- **Methodology improvements** — You've used a system and found a gap, an ambiguity, or a better way to structure something. Open an issue describing the problem and your proposed change.
- **New skill proposals** — Open an issue first. Describe what the skill does, when it's used, and how it fits into the existing systems. Skills should be single-responsibility and composable.
- **Example improvements** — Better example company data, more realistic scenarios, clearer walkthroughs.
- **Documentation fixes** — Typos, broken links, unclear instructions.

## What needs discussion first

- **Changes to SYSTEM.md files** — These define core methodology. Open an issue before submitting a PR so we can discuss the implications.
- **New systems** — A new top-level system (beyond prd) is a significant addition. Start with an issue.
- **Structural changes** — Renaming directories, changing the company profile schema, modifying frontmatter conventions.

## How to submit

1. Fork the repository.
2. Create a branch from `main` (`git checkout -b your-branch-name`).
3. Make your changes.
4. Test your changes by using them in a real workflow if possible.
5. Submit a pull request with a clear description of what changed and why.

## Conventions

- **Commit messages:** Lead with the area (`prd: ...`, `skills/prd-author: ...`).
- **File format:** All methodology files are Markdown. Workflows are YAML. Follow existing patterns.
- **Tone:** Prescriptive and specific. "The skill must X" not "the skill should consider X." See any existing SYSTEM.md for calibration.

## Code of conduct

Be respectful, constructive, and specific. We're here to build useful tools, not to argue about abstractions.
