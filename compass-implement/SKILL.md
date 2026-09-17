---
name: compass-implement
description: Implement a piece of work based on a spec or a set of tasks.
---

# Compass Implement

Implement the work described by a spec or a set of tasks.

## Consumes / produces / hands off

- **Consumes:** a spec (`compass-spec`) or a set of tasks (`compass-task`), one task at a time.
- **Produces:** working, tested code committed to the current branch.
- **Called from:** `compass-horizon`, once it has claimed a task; or directly, if a task has already been picked and claimed.
- **Hands off to:** `compass-review`, once the work is done and before it is considered finished.

## Process

Confirm the task is claimed before starting. If `compass-horizon` already claimed it, nothing to do. Otherwise claim it now: assign it to yourself, or use its tracker's equivalent of an atomic claim (for example `bd update <id> --claim`), so a concurrent session skips it.

Use `compass-test` where possible, working at pre-agreed seams (see `compass-design` if the seam itself is in question).

Run whatever this project's build/verification step is (a compiler check, a linter, a static-analysis pass, whatever catches structural errors before tests do) regularly, run the tests relevant to the current slice regularly, and run the full test suite once at the end.

Once done, use `compass-review` to review the work.

Commit the work to the current branch, then close the task (or move it to whatever status this project uses for "done").
