---
name: compass-review
description: "Review the changes since a fixed point (commit, branch, tag, or merge-base) along two axes: Standards (does the code follow this project's documented coding standards?) and Spec (does the code match what the originating spec asked for?). Runs both reviews in parallel sub-agents and reports them side by side. Use when reviewing a branch, work-in-progress changes, or asked to review since a given point."
---

# Compass Review

Two-axis review of the diff between `HEAD` and a fixed point:

- **Standards**: does the code conform to this project's documented coding standards?
- **Spec**: does the code faithfully implement the originating spec?

## Consumes / produces / hands off

- **Consumes:** a diff (a branch, a set of commits, work-in-progress changes) and, where one exists, the spec it implements.
- **Produces:** a two-axis findings report.
- **Called from:** `compass-implement`, once a piece of work is done and before it is considered finished. Can also be run standalone against any diff.
- **Hands off to:** back to `compass-implement` for fixes, if findings require them.

Both axes run as **parallel sub-agents** so they do not pollute each other's context, then this skill aggregates their findings.

## Process

### 1. Pin the fixed point

Use whatever fixed point was given (a commit, a branch name, a tag, `main`, `HEAD~5`). If none was given, ask for one.

Capture the diff command once: `git diff <fixed-point>...HEAD` (three-dot, so the comparison is against the merge-base). Also note the commit list via `git log <fixed-point>..HEAD --oneline`.

Before going further, confirm the fixed point resolves (`git rev-parse <fixed-point>`) and the diff is non-empty. A bad reference or an empty diff should fail here, not inside two parallel sub-agents.

### 2. Identify the spec source

Look for the originating spec, a **Feature**-type issue if this work came through `compass-spec`, in this order:

1. Check however this project's commit messages reference tracked work (an issue number, a ticket key, a link). Follow that reference to fetch the item.
2. A path given directly as an argument.
3. A spec file under `docs/`, `specs/`, or `tasks/` matching the branch name or feature.
4. If nothing is found, ask where the spec is. If there is not one, the **Spec** sub-agent skips and reports "no spec available."

### 3. Identify the standards sources

Anything in the project that documents how code should be written: a coding-standards document, a contributing guide, house-style notes.

On top of whatever the project documents, the Standards axis always carries the **smell baseline** below: a fixed set of Fowler code smells (*Refactoring*, ch. 3) that applies even when a project documents nothing. Two rules bind it:

- **The project overrides.** A documented project standard always wins; where it endorses something the baseline would flag, suppress the smell.
- **Always a judgement call.** Each smell is a labelled heuristic ("possible Feature Envy"), never a hard violation. Like any standard here, skip anything tooling already enforces.

Each smell reads *what it is* → *how to fix*; match it against the diff:

- **Mysterious Name**: a function, variable, or type whose name does not reveal what it does or holds. → rename it; if no honest name comes, the design is murky.
- **Duplicated Code**: the same logic shape appears in more than one hunk or file in the change. → extract the shared shape, call it from both.
- **Feature Envy**: a piece of logic that reaches into another module's data more than its own. → move the logic onto the data it envies.
- **Data Clumps**: the same few values keep travelling together (a type wanting to be born). → bundle them into one type, pass that.
- **Primitive Obsession**: a primitive or string standing in for a domain concept that deserves its own type. → give the concept its own small type.
- **Repeated Switches**: the same branch/cascade on the same distinction recurs across the change. → replace with a single shared dispatch both sites use.
- **Shotgun Surgery**: one logical change forces scattered edits across many files in the diff. → gather what changes together into one module.
- **Divergent Change**: one file or module is edited for several unrelated reasons. → split so each module changes for one reason.
- **Speculative Generality**: abstraction, parameters, or hooks added for needs the spec does not have. → delete it; inline back until a real need shows.
- **Message Chains**: long chained navigation through several objects that the caller should not depend on. → hide the walk behind one call on the first object.
- **Middle Man**: a module that mostly just delegates onward. → cut it, call the real target directly.
- **Refused Bequest**: something that inherits or implements an interface but ignores or overrides most of what it inherits. → drop the inheritance, use composition.

### 4. Spawn both sub-agents in parallel

**Standards sub-agent prompt** should include:

- The full diff command and commit list.
- The list of standards-source files found in step 3, **plus the smell baseline from step 3** pasted in full (the sub-agent has no other access to it).
- The brief: "Report, per file or hunk where relevant, (a) every place the diff violates a documented standard: cite the standard (file and rule); and (b) any baseline smell spotted: name it and quote the hunk. Distinguish hard violations from judgement calls: documented-standard breaches can be hard, but baseline smells are always judgement calls, and a documented project standard overrides the baseline. Skip anything tooling enforces. Under 400 words."

**Spec sub-agent prompt** should include:

- The diff command and commit list.
- The path or fetched contents of the spec.
- The brief: "Report: (a) requirements the spec asked for that are missing or partial; (b) behaviour in the diff that was not asked for (scope creep); (c) requirements that look implemented but where the implementation looks wrong. Quote the spec line for each finding. Under 400 words."

If the spec is missing, skip the Spec sub-agent and note this in the final report.

### 5. Aggregate

Present the two reports under `## Standards` and `## Spec` headings, verbatim or lightly cleaned. Do **not** merge or rerank findings. The two axes are deliberately separate (see *Why two axes*).

End with a one-line summary: total findings per axis, and the worst issue *within each axis* (if any). Do not pick a single winner across axes. That is the reranking the separation exists to prevent.

## Why two axes

A change can pass one axis and fail the other:

- Code that follows every standard but implements the wrong thing → **Standards pass, Spec fail.**
- Code that does exactly what the spec asked but breaks the project's conventions → **Spec pass, Standards fail.**

Reporting them separately stops one axis from masking the other.
