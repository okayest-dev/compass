---
name: compass-prototype
description: Build a throwaway prototype to answer a design question. Use when checking whether a state model, workflow, or interface feels right by driving it directly, or when comparing two or more candidate approaches side by side before committing to one.
---

# Compass Prototype

A prototype is **throwaway code that answers a question**. The question decides the shape.

## Consumes / produces / hands off

- **Consumes:** an open question that is easier to answer by looking at something running than by reasoning about it in the abstract, from `compass-navigate`, `compass-map`, or `compass-design`'s "Design It Twice" comparison.
- **Produces:** a throwaway artifact, a captured answer, and (where the answer encodes a decision precisely) a trimmed snippet worth quoting in a spec.
- **Called from:** `compass-navigate` (when a question is better answered by running code than by more questions), `compass-map` (for a `prototype` waypoint), `compass-design` (when comparing alternative interfaces).
- **Hands off to:** whichever skill asked for the prototype resumes with the answer in hand; the validated decision (not the prototype code) is what carries forward into `compass-spec`.

## Pick a branch

Identify which question is being answered, using the context, the surrounding code, or by asking if someone is around to answer:

- **"Does this feel right when driven through?"** → [INTERACTIVE.md](./INTERACTIVE.md). Build a single driveable artifact, free-play actions plus guided walkthroughs, that pushes a state model, workflow, or interface through cases that are hard to reason about on paper, and that lets someone step through it by hand.
- **"Which of these approaches is better?"** → [COMPARE.md](./COMPARE.md). Build several radically different, slimmed-down implementations of the same thing, presented side by side, so the comparison is concrete rather than argued in prose.

The two branches produce different artifacts, so getting this wrong wastes the whole prototype. If the question is genuinely ambiguous and nobody is reachable, default to whichever branch matches the shape of the question (one thing to feel out → interactive; a choice between named alternatives → compare) and state the assumption at the top of the prototype.

## Rules that apply to both

1. **Throwaway from day one, and clearly marked as such.** Locate the prototype close to where it will actually be used (next to the module or area it is prototyping for) so context is obvious, but name it so a casual reader can see it is a prototype, not production code. For anything routed or served, follow whatever routing or entry-point convention the project already uses; do not invent a new top-level structure.
2. **Trivial to run.** Whatever the project's normal way of running something is, a single command, a script, a file that opens directly, the prototype should start the same way, with no extra setup. If the intended audience is a non-developer, prefer a form that opens directly with nothing to install (a single self-contained page or document) over one that needs a toolchain running.
3. **No persistence by default.** State lives in memory. Persistence is the thing the prototype may be *checking*, not something it should depend on incidentally. If the question explicitly involves a datastore, use a scratch instance or a local file with a clear "PROTOTYPE, wipe me" name.
4. **Skip the polish.** No tests, no error handling beyond what makes the prototype *runnable*, no abstractions built for a future the prototype is not testing. The point is to learn something fast.
5. **Surface the state.** After every action (interactive) or for every candidate (compare), show the full relevant state or output so whoever is judging can see what changed or differs.
6. **Capture it when done.** Fold any validated decision into the real code, then capture the prototype itself as a **primary source**: commit it to a throwaway branch, out of the main line, and leave a pointer to that branch on whatever tracks the work. Capture the answer too (the verdict and the question it settled) alongside that pointer. The main line keeps only the validated decision, never the prototype code itself.
