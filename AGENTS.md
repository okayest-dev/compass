# AGENTS.md

This file tells agents how to work with the Compass repository.

## What this repo is

Compass is a collection of agent skills for planning and delivering software work. Each skill lives in its own folder under `compass-*` with a `SKILL.md` file that defines its behavior. Skills are discovered by OpenCode/Claude from symlinked locations.

## Project structure

```
compass/
├── AGENTS.md              # This file
├── README.md              # User-facing documentation
├── compass-*/             # Individual skills
│   ├── SKILL.md           # Skill definition (required)
│   └── *.md               # Supporting docs (DEEPENING.md, TESTS.md, etc.)
```

## Conventions for skill files

**SKILL.md front matter** — Every skill must have:
```yaml
---
name: compass-<name>
description: "One-line description used in skill discovery"
---
```

**Consumes/Produces/Hands off** — Every SKILL.md must document these at the top of the body (after front matter). This is the contract between skills.

**Vocabulary** — Use the terms from `compass-design` (module, interface, depth, seam, adapter, leverage, locality) and `compass-index` (waypoint, horizon, log, features, uncharted water) consistently. Do not substitute "component," "service," "API," or "boundary."

**Style** — Periods and commas, not em dashes. Sentence-case headings. Pseudocode, not language-specific snippets. Name concrete tools (beads, Jira, GitHub, Linear) but describe mechanisms generically first.

**Cross-references** — When renaming a skill, grep every `SKILL.md` for the old name. A stale reference points at a skill that no longer exists.

## Adding a new skill

1. Create `compass-<name>/SKILL.md` with front matter and consumes/produces/hands-off
2. Add any supporting docs (e.g., `DEEPENING.md`, `TESTS.md`) in the same folder
3. Update `compass-index/SKILL.md` to include the new skill in the catalogue and pipeline diagram
4. Update `README.md` skill tables if the skill is user-facing

## Modifying an existing skill

1. Edit the skill's `SKILL.md` and any supporting docs
2. Check `compass-index/SKILL.md` for cross-references that need updating
3. Check `README.md` for user-facing descriptions that need updating
4. Ensure consumes/produces/hands-off still aligns with actual behavior

## Testing

There is no automated test suite for skill markdown. Validation is manual:

- Read the skill end-to-end: does the process make sense?
- Check consumes/produces/hands-off against the skills it connects to
- Verify vocabulary consistency with `compass-design` and `compass-index`
- Run `compass-onboarding` in a test repo to verify tracker/memory detection works

## Linting

No formal linter. Before committing, verify:
- Front matter `name` matches folder name (`compass-<name>`)
- Description is one line, no trailing period
- All cross-references to other skills use backticks and correct names
- No em dashes, no tool-specific language in mechanism descriptions

## Git workflow

- Commit directly to main for documentation fixes
- For skill behavior changes, consider whether existing users' workflows break
- Keep commits focused; one skill per commit when possible
- No commit message format enforced; be descriptive