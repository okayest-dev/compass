---
name: compass-survey
description: Survey a codebase for deepening opportunities, present them for discussion, then hand the agreed candidate to compass-navigate and compass-task to land it in the tracker. Use when auditing architecture, hunting for shallow modules, or asked to find refactor candidates.
---

# Compass Survey

Survey the codebase for **deepening opportunities**: places where a shallow module could become a deep one. The aim is leverage for callers, locality for maintainers, and testability for everyone, the same three payoffs `compass-design` names. `compass-design`'s vocabulary is scale- and language-agnostic: it applies equally to a function, a class, a service, or a package, whatever language the codebase is written in. Nothing in this skill assumes a particular language, runtime, or toolchain.

## Consumes / produces / hands off

- **Consumes:** a codebase, or a named direction within it (a module, a subsystem, a pain point), in any language.
- **Produces:** a short list of deepening candidates, discussed directly in conversation. Nothing is written down anywhere durable until the user agrees to pursue one.
- **Called from:** used directly whenever architecture needs auditing; can also seed a `compass-map` `investigate` waypoint when the survey itself is the open question.
- **Hands off to:** `compass-navigate` once a candidate is agreed, to work the decision tree (constraints, dependencies, the deepened module's shape, what sits behind the seam, what tests survive); `compass-glossary` to record new vocabulary or a hard-to-reverse decision as it crystallises during that interview; `compass-task`, once `compass-navigate` has settled the shape, to publish the agreed candidate as one or more Task-type issues wherever this project tracks work, the same terminal step every other path through the pipeline ends on; `compass-design` for the vocabulary itself, and its [DEEPENING.md](../compass-design/DEEPENING.md) for dependency-driven seam decisions; `compass-prototype` when comparing alternative interfaces for the deepened module needs running code, not just a description.

Use `compass-design`'s terms exactly (**module**, **interface**, **depth**, **seam**, **adapter**, **leverage**, **locality**), in every candidate, and don't drift into "component," "service," "API," or "boundary."

Read the project's glossary before surveying: recall the persistent memory store if one is configured, or `CONTEXT.md` and `docs/adr/` otherwise (see `compass-glossary`). The glossary gives names to good seams; recorded decisions there mark ground this survey should not re-litigate.

## Process

### 1. Survey

**Scope before you scan: YAGNI.** Deepening a module pays off by making future changes to it easier, so put extra weight on the parts of the codebase that have recently changed. Decide *where* to look before you look:

- If a direction was named (a module, a subsystem, a pain point), take it, and skip the inference below.
- Otherwise, walk back a good stretch of the commit history (`git log --oneline`) to find the codebase's hot spots, the files and areas that keep coming up, and let those paths pull attention first. If the changes are scattered with no clear hot spot, widen the net.

Then spawn a sub-agent to walk the codebase. Don't follow rigid heuristics; explore organically and note where friction shows up:

- Where does understanding one concept require bouncing between many small modules?
- Where are modules **shallow**, with an interface nearly as complex as the implementation?
- Where have pure functions been extracted just for testability, but the real bugs hide in how they're called (no **locality**)?
- Where do tightly-coupled modules leak across their seams?
- Which parts of the codebase are untested, or hard to test through their current interface?

Apply the **deletion test** to anything suspected of being shallow: would deleting it concentrate complexity, or just move it? A "yes, concentrates" is the signal worth surfacing.

### 2. Present the candidates

Present candidates directly in the conversation. No file, no report, no diagramming tool: everything here renders as plain text so it fits any terminal, any language, any project, without assuming a browser or a particular ecosystem.

For each candidate, give:

- **Title**: short, names the deepening (e.g. "Collapse the Order intake pipeline").
- **Recommendation strength**: one of `Strong`, `Worth exploring`, `Speculative`.
- **Dependency category**: one of `in-process`, `local-substitutable`, `ports & adapters`, `mock` (see `compass-design`'s [DEEPENING.md](../compass-design/DEEPENING.md)), once it's clear enough to name.
- **Files**: the files or modules involved.
- **Problem**: one sentence. What hurts.
- **Solution**: one sentence. What changes.
- **Wins**: bullets, six words or fewer each, named in glossary terms (*"locality: bugs concentrate in one module"*, *"leverage: one interface, many call sites"*). Don't write *"easier to maintain"* or *"cleaner code"*; those terms aren't in the glossary and don't earn their place.
- **Before / after**: two small boxes, plain characters, reusing `compass-design`'s own notation for shallow and deep:

  ```
  Before (shallow)                After (deep)
  ┌─────────────────────┐         ┌─────────────────────┐
  │   Large Interface    │         │   Small Interface    │
  ├─────────────────────┤         ├─────────────────────┤
  │  Thin Implementation │         │  Deep Implementation │
  └─────────────────────┘         └─────────────────────┘
  ```

  Adjust the labels to name what's actually shrinking or growing; the box shapes are the point, not decoration.
- **Decision-record callout** (if applicable): one line, flagged clearly, e.g. _"Contradicts the recorded decision on X, worth reopening because…"_. Only surface a candidate that contradicts a recorded decision when the friction is real enough to warrant reopening it; don't list every theoretical refactor a recorded decision forbids.

End with a **top recommendation**: which candidate to tackle first and why, one line.

Do not propose interfaces yet, and do not write anything to a file or the tracker at this point. Ask which candidate to pursue, and wait for an answer.

### 3. Once agreed, hand off

Nothing here is durable until it is in the tracker. Once a candidate is agreed:

1. Hand off to `compass-navigate` to work the decision tree: constraints, dependencies, the shape of the deepened module, what sits behind the seam, what tests survive.
2. Once `compass-navigate` settles that shape, hand off to `compass-task` to publish the agreed candidate as one or more Task-type issues wherever this project tracks work, exactly as every other terminal step in the pipeline does. A rejected or unpursued candidate is never published; only what was agreed reaches the tracker.

Side effects happen inline as decisions crystallise during the `compass-navigate` interview; use `compass-glossary` to keep the model current:

- **Naming a deepened module after a concept not yet in the glossary?** Record the term.
- **Sharpening a fuzzy term during the conversation?** Update the glossary right there.
- **The candidate gets rejected for a load-bearing reason?** Offer to record the decision, following `compass-glossary`'s three-part test (hard to reverse, surprising without context, the result of a real trade-off). Skip ephemeral reasons ("not worth it right now") and self-evident ones.
- **Exploring alternative interfaces for the deepened module?** Use `compass-prototype`'s comparison branch.
