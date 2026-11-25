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

## Next step

**After the task is complete (Clear verdict):**

| Situation | Next action | Invocation |
|-----------|-------------|------------|
| Task closed, more tasks in the same Epic/Feature | `compass-horizon` | Continue in **this session** — invoke `compass-horizon` to claim the next unblocked task. |
| Task closed, no more unblocked tasks in context | — | **End of session** — work for this Epic/Feature is done. Start a **fresh session** when ready for the next Epic/Feature or when blockers clear. |
| All tasks in the Feature done | `compass-review` (optional, final) | **Fresh session recommended** — run a final `compass-review` against the full Feature diff (merge-base to HEAD) for a holistic Standards/Spec/Security audit before merging. |
| Blocked finding survived 3 rounds or requires spec change | `compass-navigate` / `compass-task` | **Fresh session** — take it to the task owner per `compass-review`'s exception process. If they direct a spec change, invoke `compass-navigate`; if re-scoping, invoke `compass-task`. |

**If the task was a refactor** (detected by `compass-implement`): it delegates to `compass-refactor` internally. The next-step logic above applies after `compass-refactor` returns.
