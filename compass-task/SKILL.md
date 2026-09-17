---
name: compass-task
description: Break a plan, spec, or the current conversation into a set of tracer-bullet tasks, each declaring its blocking edges, published wherever this project tracks work.
---

# Compass Task

Break a plan, spec, or conversation into a set of **tasks**: tracer-bullet vertical slices, each declaring the tasks that **block** it.

## Consumes / produces / hands off

- **Consumes:** a spec that has already been through `compass-decide`'s sanity check and Feature registration; a plan; or the current conversation, for lighter-weight work with no formal spec.
- **Produces:** a set of vertical-slice **Task**-type issues with explicit blocking edges, published wherever this project tracks work.
- **Called from:** `compass-decide`, once a spec is confirmed sane and registered; or directly, once a plan or the current conversation contains enough decisions to break into tasks without a formal spec.
- **Hands off to:** `compass-horizon`, to pick up the first unblocked task and hand it to `compass-implement`. A task can also go straight to `compass-implement` if it has already been chosen and claimed.

## Issue type

Every item this skill produces is a **Task** (use the tracker's closest equivalent: a Jira Task or Sub-task, a GitHub or Linear issue labelled `task`, a `beads` issue created with `-t task`, linked as a child of the spec's Feature where one exists). This applies to the expand-contract batches in [wide refactors](#3-draft-vertical-slices) too. Each batch is still a Task, just one with a narrower, more mechanical "what to build." Do not leave a task untyped or generic; the type is what makes it pick-up-able as work rather than mistaken for an open decision.

## Process

### 0. Confirm the spec has cleared compass-decide

Skip this step when working from a plan or the current conversation with no formal spec behind it. Go straight to step 1.

When working from a spec, do not draft a single task until it has been through `compass-decide`'s sanity check and Feature registration (see that skill's "Sanity-checking a provided spec" section). This applies whether the spec just came from `compass-spec` or was handed in directly, including a spec pasted straight into this conversation. If that has not happened yet, stop and hand off to `compass-decide` now; resume here only once it confirms the spec is sane and registered.

### 1. Gather context

Work from whatever is already in the conversation. If a reference was given, fetch and read it fully first.

### 2. Explore the codebase (optional)

If the codebase has not already been explored, do so to understand its current state. Task titles and descriptions should use the project's domain glossary vocabulary (see `compass-glossary`), and respect any recorded decisions in the area being touched.

Look for opportunities to prefactor the code to make the implementation easier. "Make the change easy, then make the easy change."

### 3. Draft vertical slices

Break the work into **tracer bullet** tasks.

<vertical-slice-rules>

- Each slice cuts a narrow but COMPLETE path through every layer it touches (data, logic, interface, tests): vertical, NOT a horizontal slice of one layer.
- A completed slice is demoable or verifiable on its own.
- Each slice is sized to fit in a single fresh working session.
- Any prefactoring should be done first.

</vertical-slice-rules>

Give each task its **blocking edges**: the other tasks that must complete before it can start. A task with no blockers can start immediately.

**Wide refactors are the exception to vertical slicing.** A **wide refactor** is one mechanical change (rename a shared field, retype a shared symbol) whose **wake** fans across the whole codebase, so a single edit breaks many call sites at once and no vertical slice can land cleanly. Do not force it into a tracer bullet; sequence it as **expand-contract**. First expand: add the new form beside the old so nothing breaks. Then migrate the call sites over in batches sized by wake (per package, per directory), each batch its own task blocked by the expand, keeping the build green batch to batch because the old form still exists. Finally contract: remove the old form once no caller remains, in a task blocked by every migration batch. When even the batches cannot stay green alone, keep the sequence but let them share an integration branch that all block a final integrate-and-verify task; green is promised only there.

### 4. Quiz whoever is approving the breakdown

Present the proposed breakdown as a numbered list. For each task, show:

- **Title**: short descriptive name.
- **Blocked by**: which other tasks (if any) must complete first.
- **What it delivers**: the end-to-end behaviour this task makes work.

Ask:

- Does the granularity feel right? (too coarse or too fine)
- Are the blocking edges correct: does each task only depend on tasks that genuinely gate it?
- Should any tasks be merged or split further?

Iterate until the breakdown is approved.

### 5. Publish the tasks

Publish the approved tasks as Task-type issues (see [Issue type](#issue-type) above) wherever this project tracks work. The tasks are the same either way; only the shape of the blocking edges changes:

- **No tracker documented for this project** → write one file per task under `tasks/<NN>-<slug>.md`, numbered from `01` in dependency order (blockers first), with `Type: task` noted at the top of each. Each file's "Blocked by" lists the numbers or titles it depends on. Use the local task template below: one task per file, never a single combined file.
- **A documented issue tracker** → publish one Task-type issue per task in dependency order (blockers first) so each task's blocking edges can reference real identifiers. Use the tracker's native blocking or sub-issue relationship where it has one; otherwise set each task's "Blocked by" field to the blocking issues. Apply whatever label or tag this project uses to mark work as ready for an implementer, unless told otherwise; the tasks are implementer-ready by construction.

Work the **horizon**: any task whose blockers are all done. For a purely linear chain that means top to bottom.

Do not close or modify any parent spec or issue.

<local-task-template>

# `<NN>`: `<Task title>`

**Type:** task

**What to build:** the end-to-end behaviour this task makes work, from the user's perspective, not a layer-by-layer implementation list.

**Blocked by:** the numbers or titles of the tasks that gate this one, or "None (can start immediately)".

**Status:** ready

- [ ] Acceptance criterion 1
- [ ] Acceptance criterion 2

</local-task-template>

<issue-template>

## Parent

A reference to the parent spec or issue (if the source was an existing one, otherwise omit this section).

## Type

Task

## What to build

The end-to-end behaviour this task makes work, from the user's perspective, not layer-by-layer implementation.

## Acceptance criteria

- [ ] Criterion 1
- [ ] Criterion 2

## Blocked by

- A reference to each blocking task, or "None (can start immediately)".

</issue-template>

In either form, avoid specific file paths or code snippets: they go stale fast. Exception: if `compass-prototype` produced a snippet that encodes a decision more precisely than prose can (a state machine, a schema, a shape of data), inline it and note briefly that it came from a prototype. Trim it to the decision-rich parts, not a working demo, just the important bits.
