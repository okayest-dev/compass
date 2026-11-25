---
name: compass-onboarding
description: "Configure this repo for Compass: the issue tracker's mechanics for each Compass issue type, the horizon query and claim operation, the ready-for-implementer label, and the memory store compass-glossary writes to. Run once before first use of the other Compass skills."
disable-model-invocation: true
---

# Compass Onboarding

Scaffold the per-repo configuration the rest of Compass assumes is already documented:

- **Issue tracker mechanics**: where issues live, how each Compass issue type maps onto the tracker's native types, the horizon query, the claim operation, and the ready-for-implementer label.
- **Memory store**: what `compass-glossary` writes to, a persistent memory tool if one is available, flat files otherwise.

This is a prompt-driven skill, not a deterministic script. Explore, present what was found with a recommended default, confirm with the user, then write.

## Consumes / produces / hands off

- **Consumes:** the repo's current state: its git remote, any existing `AGENTS.md`/`CLAUDE.md`, `CONTEXT.md`/`CONTEXT-MAP.md`/`docs/adr/`, and whether a memory tool is available.
- **Produces:** a `## Compass setup` block in `AGENTS.md` or `CLAUDE.md`, naming the tracker, the issue-type mapping, the horizon query, the claim operation, the ready-for-implementer label, and the memory store.
- **Called from:** directly, once, before any other Compass skill is used on this repo for the first time.
- **Hands off to:** nothing. Every pipeline skill reads what this one wrote (`compass-horizon` and `compass-map` for tracker mechanics, `compass-glossary` for the memory store); this skill does not trigger the next phase itself.

Only these two sections are in scope. `compass-review` and `compass-implement` re-explore for standards docs, security docs, and the build/verification command fresh on every run; neither claims to read them from `AGENTS.md`, so pre-recording them here would be dead weight.

## Process

### 1. Explore

Look at the current repo to understand its starting state. Read whatever exists; don't assume:

- `git remote -v`: is this a GitHub repo, a GitLab repo, something else?
- `AGENTS.md` and `CLAUDE.md` at the repo root: does either exist? Is there already a `## Compass setup` block in either?
- `CONTEXT.md`, `CONTEXT-MAP.md`, and `docs/adr/` at the repo root.
- A persistent memory tool: an available tool exposing a `remember`/`recall`/`relate`-style interface, or a tracker with its own memory command (for example `beads` via `bd remember` or `bd note`).
- Monorepo signals: a `pnpm-workspace.yaml`, a `workspaces` field in `package.json`, or a populated `packages/*` with its own `src/`. Their absence means single-context, which is almost every repo.

### 2. Section A: Issue tracker mechanics

> Explainer: `compass-horizon` and `compass-map` both read this project's issue-tracker workflow from `AGENTS.md` to know which query and claim commands to run. `compass-task` and `compass-spec` need the same label to mark a Task or Feature ready for an implementer. Without this section, every one of those skills has to ask from scratch, every session.

Lead with the recommended default from what step 1 found: GitHub if the remote points there, GitLab if it points at `gitlab.com` or a self-hosted host. Otherwise offer:

- **GitHub**: issues live in the repo's GitHub Issues (`gh` CLI).
- **GitLab**: issues live in the repo's GitLab Issues (`glab` CLI).
- **Jira** / **Linear**: ask which project or team, and how issue types map there.
- **`beads`**: issues live in the repo's `beads` database (`bd` CLI).
- **Local markdown**: issues live as files under `tasks/` and `decisions/` in this repo, per `compass-task` and `compass-map`'s own local-file conventions.
- **Other**: ask the user to describe the workflow in one paragraph; record it as freeform prose.

Once the tracker is picked, confirm each of the following, proposing the obvious default for that tracker rather than asking from scratch:

- **Issue-type mapping**: Epic, Feature, Task, Decision → the tracker's native types (a Jira Epic/Story/Task/a Decision-labelled issue; a `beads -t epic`/`-t feature`/`-t task`/`-t decision`; a GitHub or Linear label per type; a local file's `Type:` field).
- **Horizon query**: how to list open, unblocked, unclaimed Task-type issues. Default: `bd ready` for `beads`; an assigned-to-nobody, label-filtered saved search for GitHub/GitLab/Linear/Jira; a scan of `tasks/*.md` for files whose `Blocked by` is empty or fully closed and whose `Status` isn't yet claimed, for local files.
- **Claim operation**: how a task gets marked as picked up. Default: `bd update <id> --claim` for `beads`; assigning the issue to whoever's claiming it for GitHub/GitLab/Linear/Jira; setting `Status: claimed` on the local file.
- **Ready-for-implementer label**: the tag `compass-task` and `compass-spec` apply so `compass-horizon` can filter on it. Default to a literal `ready-for-implementer` tag if nothing already exists in this project's label vocabulary.

### 3. Section B: Memory store

> Explainer: `compass-glossary` prefers a persistent memory tool over flat files whenever one is configured, and reads `AGENTS.md` as one of the signals it checks for that. Recording the choice here means it doesn't have to re-detect it every session.

If step 1 found a memory tool, name it and move on; there is nothing else to ask. Otherwise default to flat files, single-context (`CONTEXT.md` + `docs/adr/`), which fits almost every repo. Only raise multi-context (`CONTEXT-MAP.md` + a per-context `CONTEXT.md`/`docs/adr/` under each) if step 1 found monorepo signals; then confirm which layout is wanted.

### 4. Confirm

Show the user a draft of the `## Compass setup` block (see [Write](#5-write) below). Let them edit before writing.

### 5. Write

**Pick the file to edit:**

- If `CLAUDE.md` exists, edit it.
- Else if `AGENTS.md` exists, edit it.
- If neither exists, ask the user which one to create; don't pick for them.

Never create `AGENTS.md` when `CLAUDE.md` already exists, or vice versa; always edit the one that's already there.

If a `## Compass setup` block already exists, update its contents in place rather than appending a duplicate. Don't overwrite user edits to the surrounding sections.

<compass-setup-block>

```markdown
## Compass setup

### Issue tracker

[Tracker name]. Epic → [x], Feature → [x], Task → [x], Decision → [x].
Horizon query: [command or filter]. Claim: [command or action].
Ready-for-implementer: [label or tag].

### Memory store

[Tool name, or "flat files: single-context", or "flat files: multi-context"].
```

</compass-setup-block>

### 6. Done

Tell the user the setup is complete, and name which Compass skills will now read from this block (`compass-horizon`, `compass-map`, `compass-task`, `compass-spec`, `compass-glossary`). Mention they can edit the block directly later; re-running this skill is only necessary to switch tracker or memory store, or to start over.
