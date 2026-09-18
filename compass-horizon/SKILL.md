---
name: compass-horizon
description: Fetch and claim the next available task from the project's task backlog (beads, GitHub, GitLab, Linear, Jira, local files, or other, detected from however this project documents its tracker workflow). If an epic or feature is already in context, prefers tasks under it. Use at session start to pick up work, when asked what to work on next, or to grab the next task after finishing one.
---

# Compass Horizon

When a session needs work, grab the next available task. Fetch the horizon, claim the first task on it, report it. If an epic or feature is already in context, prefer tasks under it over the rest of the backlog.

## Consumes / produces / hands off

- **Consumes:** the task backlog `compass-task` produced (Task-type issues or local task files); however this project's tracker exposes an open/unblocked/unclaimed query; and, when one is in context, the Epic or Feature to prefer.
- **Produces:** one claimed task, reported with enough detail to start work from.
- **Called from:** directly, at the start of a session, when asked what to work on next, or right after finishing a task to grab the next one.
- **Hands off to:** `compass-implement`, to do the work.

The **horizon** is the set of open, unblocked, unclaimed Task-type issues in the backlog: the next work an agent can pick up. This skill runs whichever query and claim operation this project's tracker supports, so it works for any manager.

## Steps

### 1. Find the backlog

If this project documents an issue-tracker workflow (in `AGENTS.md`, a contributing guide, or similar), read it. It names the manager and the operations for it.

No such doc: the backlog is wherever `compass-task` already wrote its tasks, a tracker if one is in use, otherwise local files under `tasks/`. Don't guess at commands you haven't confirmed; ask what manager is in use if it isn't obvious from the repo.

Done when: the manager is known and its horizon query can be named.

### 2. Fetch the horizon

Run the horizon query: open, unblocked, unclaimed Task-type issues. For example `bd ready` for beads, a saved search or filter on GitHub, GitLab, Linear, or Jira, or scanning `tasks/*.md` for files whose `Blocked by` is empty or fully closed and whose `Status` isn't yet claimed.

No documented query: ask how to list available tasks, run it, then note the query for next time.

Done when: there is a candidate list of tasks, or a clear answer that the horizon is empty.

### 3. Pick the next one

Check whether an Epic or Feature is already in context. If one is, narrow the candidates to tasks linked to it first, however this project's tracker exposes that link (a direct parent field, a shared label, or the task's own Feature tracing back to an Epic). Widen back to the full horizon only if none of its tasks are unblocked and unclaimed.

Order the remaining candidates as the query defines (dependency order, file number, the tracker's own default order). Prefer tasks carrying whatever label or tag this project uses to mark work as ready for an implementer (see `compass-task`'s issue type notes); fall back to any horizon task.

Done when: exactly one task is chosen.

### 4. Claim it

Run the claim operation: `bd update <id> --claim`, assigning the issue to yourself on GitHub, GitLab, Linear, or Jira, or setting `Status: claimed` on a local task file. This is the session's first write, so no other session double-picks the task.

Done when: the claim write has succeeded and the task shows as claimed.

### 5. Report

Read the task's full body if the summary needs more detail. Then report one tight summary: the task's id or path, its title, and what it asks. State that it is unblocked and now claimed. A task that reads as unclear or under-specified: say so rather than papering over it, and consider whether it needs a trip back through `compass-navigate` before implementation starts. Then hand off to `compass-implement` for the build.

Done when: an agent could start the task from the summary alone.

When the horizon is empty, say so plainly and stop. Never invent a task.
