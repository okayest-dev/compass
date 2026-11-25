---
name: compass-quick-implement
description: Implement a small, self-contained change in one session: short spec in conversation, test-first implementation, single commit, done. Use when the work is one behaviour, a few files, no new contract or dependency.
---

# Compass Quick Implement

A fast path for small changes that don't need the full pipeline. One session: confirm behaviour, write a short spec in the conversation, implement test-first, prune tests, clean comments, commit, done.

## Consumes / produces / hands off

- **Consumes:** a rough issue description or a claimed Task that reads as small (one behaviour, few files, no new contract/dependency).
- **Produces:** a short spec captured in the conversation, working tested code committed to the current branch, the task closed.
- **Called from:** `compass-horizon` when the claimed task passes the size gate (see below); or directly when a user describes a small change.
- **Hands off to:** `compass-review` (single pass, not a loop); if review is Blocked, fix and re-review once; if still Blocked, escalate to full `compass-implement` pipeline.

## Size gate

Before using this skill, confirm the work fits:

- One user-visible behaviour (one user story or one bug fix).
- Touches at most 3–5 files.
- No new external contract (API, schema, message format, auth change).
- No new dependency.
- No cross-repo impact.
- Estimated to complete in one fresh working session.

If any answer is "no" or "unsure," use the full pipeline: `compass-navigate` → `compass-spec` → `compass-task` → `compass-horizon` → `compass-implement`.

## Process

### 1. Confirm and claim

Confirm the task is claimed (see `compass-implement` step 1). If not claimed, claim it now.

### 2. Confirm current behaviour

Find the code that implements the current behaviour. Read it. Summarise what it does now in one or two sentences. Ask the user to confirm or correct.

### 3. Short spec in conversation

Ask the minimal questions to nail the change:

- What changes? (one sentence)
- What is the new behaviour? (one sentence)
- Any non-goals? (explicitly list what is not changing)

Write the short spec in the conversation using this template:

```
## Quick Spec: <title>

**Current behaviour:** <one sentence>
**New behaviour:** <one sentence>
**Non-goals:** <list, or "none">
**Files to change:** <list, or "tbd — will find during implementation">
**Test approach:** <one sentence: what new test, or what existing test to extend>
```

Get explicit user approval: "Does this capture it? Approve to implement."

### 4. Implement test-first

Use `compass-test` to drive the implementation:

- Write the failing test first (new test or extended existing test).
- Make it pass with the minimal change.
- Run the project's build/verification step.
- Run the full test suite once at the end.

### 5. Prune tests and clean comments

Before committing, do two mandatory passes:

**Test pruning pass** (see `compass-test`): remove any test that is
- redundant (same behaviour tested at multiple levels),
- tautological (tests the test, not the behaviour),
- trivial (tests a getter, a constructor, a constant),
- coupled to implementation details (tests private methods, internal structure).

Keep only tests that verify observable behaviour at the agreed seam.

**Comment cleanup pass**: remove any comment that
- narrates the code ("this function does X"),
- is commented-out code,
- is a change history or TODO that belongs in the tracker,
- states the obvious.

Keep only comments that explain a non-obvious *why* (a constraint, a trade-off, a workaround) in one line.

### 6. Single review pass

Run `compass-review` once against the diff. 

- **Clear**: commit, close the task, done.
- **Blocked**: fix every blocking finding, re-review once. If still Blocked, stop and hand off to the full `compass-implement` pipeline (which loops until Clear). Do not loop here.

### 7. Commit and close

Commit with a message that references the task and summarises the change. Close the task (or move to the project's "done" status).

Record the outcome in the task: a one-line summary of what changed, the short spec from step 3, and any decisions made during implementation.

## If the change grows

If during implementation the work turns out to be larger than the size gate (new files, new contract, new dependency, cross-repo), stop. Hand off to `compass-navigate --quick` with what has been gathered so far (the short spec, the current diff, the decisions). The full pipeline will pick up from there without losing ground.

## Notes

This skill does not publish a formal spec artifact. The short spec lives in the conversation and the task record. If the project requires a durable spec for every change, use the full pipeline instead. This skill is for changes small enough that the conversation + commit message + task record are sufficient traceability.