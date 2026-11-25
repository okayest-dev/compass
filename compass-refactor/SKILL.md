---
name: compass-refactor
description: "Behavior-preserving simplification of a module or seam per an agreed deepening decision. Writes characterization tests first, transforms in incremental verified steps, commits atomically, passes compass-review. Use when a compass-survey candidate has been through compass-navigate and compass-task and the resulting task is 'refactor X to deepen module Y per decision Z.'"
---

# Compass Refactor

Execute a behavior-preserving refactor that deepens a module or seam. The refactor is defined by a `compass-navigate`-settled decision (recorded in `compass-glossary`) and published as a Task by `compass-task`. This skill runs inside `compass-implement` for that task.

## Consumes / produces / hands off

- **Consumes:** a claimed Task whose "What to build" describes a refactor (e.g., "Collapse the Order intake pipeline into a single deep module per decision D-012"), with a reference to the `compass-navigate` decision and the agreed seam.
- **Produces:** working, tested code committed in atomic steps, with characterization tests at the agreed seam, passing `compass-review` with a Clear verdict.
- **Called from:** `compass-implement` when the task is a refactor.
- **Hands off to:** `compass-review` (verdict gates done); on Blocked, fixes and re-reviews until Clear.

## Preconditions (verified at start)

Before starting, confirm all are true:

- The task references a `compass-navigate` decision (stable ID `D-<NNN>`) that settled the deepened module's shape, seam, and what tests survive.
- The agreed seam is identified and testable (see `compass-design` for seam vocabulary).
- `compass-test` can drive tests at that seam.
- The task's blocking edges are satisfied (no concurrent changes to the same seam).

If any precondition fails, stop and hand back to `compass-navigate` or `compass-task`.

## Process

### 1. Read the decision and seam

Fetch the `compass-navigate` decision (`D-<NNN>`) from `compass-glossary` or the tracker. Extract:
- The deepened module's new interface (the **small interface**).
- What moves behind the seam (the **deep implementation**).
- The agreed seam(s) where behavior must not change.
- What existing tests cover the seam, and what new characterization tests are needed.

### 2. Write characterization tests at the seam

Use `compass-test` to write **characterization tests** at the agreed seam *before* changing any implementation code:

- Tests call the seam (the new small interface) with real inputs and assert current observable behavior.
- Do not test implementation details; only observable behavior at the seam.
- If the seam has no tests, write enough to cover the behavior the refactor must preserve.
- Run the tests; they must pass against current code.

Record the test file(s) and the seam they cover in the task.

### 3. Plan incremental steps

Break the refactor into a sequence of **safe, verified steps**. Each step:

- Changes only one side of the seam (either the caller side or the implementation side).
- Is small enough to verify in a single test run.
- Has a clear rollback point (the commit before the step).

Present the step plan in the conversation for confirmation. Example:

```
Step 1: Extract OrderValidator class behind the seam (implementation side only). Tests: existing characterization tests pass.
Step 2: Replace inline validation calls with OrderValidator calls in OrderService (caller side). Tests: pass.
Step 3: Inline and remove the old ValidationHelper module (implementation side). Tests: pass.
Step 4: Update the seam interface to the final small shape (both sides). Tests: pass.
```

### 4. Execute steps incrementally

For each step:

1. Make the change.
2. Run the project's build/verification step.
3. Run the characterization tests at the seam.
4. Run any existing tests that touch the changed files.
5. If all pass: commit using `compass-git atomic-commit` (one commit per step, message references the task and step).
6. If any fail: revert the step (`git checkout -- .`), analyze, adjust, retry. Do not proceed with failing tests.

Do not batch steps. Each commit must build and pass tests.

### 5. Prune tests and clean comments

Before the final review, do the two mandatory passes (see `compass-review`):

**Test pruning pass**: remove any characterization test that is redundant, tautological, trivial, or coupled to implementation details. Keep only tests that verify observable behavior at the seam.

**Comment cleanup pass**: remove narrating comments, commented-out code, change-history comments, obvious statements. Keep only non-obvious *why* comments (constraints, trade-offs, workarounds).

### 6. Review and done

Run `compass-review` against the full diff (all refactor commits). The review axes apply with these emphases:

- **Standards**: *Hard* finding if any test violates the test-pruning rules or any comment violates the comment-cleanup rules. Also: verify no behavior change — the characterization tests must still pass with the same inputs/outputs as before.
- **Spec**: *Missing* or *Wrong* if the refactor deviates from the `compass-navigate` decision (interface shape, what moved behind the seam, what tests survive).
- **Security**: *Critical/High* if the refactor introduces a new trust boundary, removes a validation, or changes authorization at the seam.

- **Clear**: all blocking findings resolved → commit final cleanup if any, close task, done.
- **Blocked**: fix, re-review, loop until Clear (same as `compass-implement`).

### 7. Record outcome

In the task and `compass-glossary` (if the refactor produced a hard-to-reverse decision):

- One-line summary of what changed.
- The decision ID (`D-<NNN>`) and how the refactor implemented it.
- The seam and characterization test files.
- Any new vocabulary or sharpened terms for the glossary.

## If the refactor grows

If during execution the work turns out to be larger than the task scope (new files beyond the seam, new contract, new dependency), stop. Hand off to `compass-navigate` with what has been gathered (the decision, the current diff, the characterization tests). The full pipeline will re-scope without losing ground.

## Notes

- This skill assumes the refactor is **behavior-preserving at the seam**. It does not handle "refactors" that change observable behavior — those are implementation tasks, not refactors.
- The characterization tests are the safety net. If the seam has no tests and cannot be tested, do not refactor; hand back to `compass-navigate` to decide on a testable seam or a different approach.
- All commits go through `compass-git atomic-commit` for consistent messages and ordering.

## Next step

**After the refactor is complete (Clear verdict, task closed):**

| Situation | Next action | Invocation |
|-----------|-------------|------------|
| More refactor tasks in the same Epic/Feature (each with its own navigate decision) | `compass-horizon` | Continue in **this session** — invoke `compass-horizon` to claim the next refactor task. |
| Refactor complete, implementation tasks remain in the same Feature | `compass-horizon` | Continue in **this session** — invoke `compass-horizon` to claim the next implementation task (will route to `compass-implement`). |
| All tasks in the Feature done | `compass-review` (optional, final) | **Fresh session recommended** — run a final `compass-review` against the full Feature diff for a holistic audit before merging. |
| No more unblocked tasks in context | — | **End of session** — start a **fresh session** when new work arrives. |
| Refactor grew beyond task scope (new files beyond seam, new contract) | `compass-navigate` | **Fresh session** — hand off to `compass-navigate` with the decision, diff, and characterization tests gathered. The full pipeline re-scopes from there. |