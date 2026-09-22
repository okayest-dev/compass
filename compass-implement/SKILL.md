---
name: compass-implement
description: Implement a piece of work based on a spec or a set of tasks.
---

# Compass Implement

Implement the work described by a spec or a set of tasks.

## Consumes / produces / hands off

- **Consumes:** a spec (`compass-spec`) or a set of tasks (`compass-task`), one task at a time.
- **Produces:** working, tested code committed to the current branch, only once `compass-review` returns a Clear verdict or every remaining blocking finding has a recorded, owner-accepted exception.
- **Called from:** `compass-horizon`, once it has claimed a task; or directly, if a task has already been picked and claimed.
- **Hands off to:** `compass-review`, in a loop, once the work is done and before it is considered finished.

## Process

Confirm the task is claimed before starting. If `compass-horizon` already claimed it, nothing to do. Otherwise claim it now: assign it to yourself, or use its tracker's equivalent of an atomic claim (for example `bd update <id> --claim`), so a concurrent session skips it.

Use `compass-test` where possible, working at pre-agreed seams (see `compass-design` if the seam itself is in question).

Where the spec names Security Decisions (trust boundaries, sensitive data, authorisation changes, restricted outbound calls), implement them as fixed requirements, not options to revisit at review time. If the seam or approach shifts during the build in a way that changes a Security Decision (a new endpoint appears that was not in the spec, a value turns out to be sensitive that was not flagged as such), update the spec in place and say so, rather than letting the code diverge from what it claims to guard against.

Run whatever this project's build/verification step is (a compiler check, a linter, a static-analysis pass, whatever catches structural errors before tests do) regularly, run the tests relevant to the current slice regularly, and run the full test suite once at the end.

### Clear the review gate before calling it done

Once the slice (or the whole task) is built, run `compass-review` against the work. Its verdict, not the passage of a review, decides whether the task is finished:

1. **Verdict is Clear.** Nothing further from this loop. Commit the work to the current branch, then close the task (or move it to whatever status this project uses for "done").
2. **Verdict is Blocked.** Fix every blocking finding it lists, then run `compass-review` again on the updated diff. This is a loop, not a single pass: fix, re-review, fix, re-review, until Clear.
3. **A blocking finding survives three rounds, or the fix would change what the spec asked for rather than how it's built.** Stop looping. Take it to whoever owns the task, following `compass-review`'s "Accepting a blocking finding": either they accept it as a recorded exception, or the task goes back to `compass-navigate` (if the spec itself needs to change) or `compass-task` (if the work needs re-scoping). Never accept a blocking finding yourself, and never keep re-running the same fix past three rounds hoping it lands differently.

Only step 1, or step 3 ending in a recorded exception, counts as done. A Blocked verdict with unresolved findings and no recorded exception is not a stopping point, no matter how much of the task is otherwise built.
