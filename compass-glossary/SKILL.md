---
name: compass-glossary
description: Build and sharpen a project's shared vocabulary and decision history, using the project's persistent memory tool as the primary store and CONTEXT.md/decision records only as a fallback. Use when discussing codebase terminology or recording a decision that is hard to reverse.
---

# Compass Glossary

Build and sharpen the project's shared vocabulary and decision history as you design. This is the *active* discipline: challenge terms, invent edge-case scenarios, and write facts and decisions down the moment they crystallise. Passively reading existing vocabulary is not this skill. Any skill can do that in passing. This skill is for when the model itself is changing.

## Consumes / produces / hands off

- **Consumes:** a live conversation where terminology is unclear, contested, or about to be locked in, or a decision that is hard to reverse.
- **Produces:** structured facts and decisions in this project's persistent memory tool, when one is configured. Falls back to `CONTEXT.md` (glossary) and `docs/adr/NNNN-slug.md` (decision records) only when no such tool exists.
- **Called from:** `compass-decide`, `compass-map`, `compass-spec`, `compass-implement`, `compass-review`, or any skill that needs shared vocabulary or needs to record a hard-to-reverse choice.
- **Hands off to:** nothing. This skill only writes memory or files; it does not trigger the next phase.

## Pick a store, once, before writing anything

Check whether this project has a persistent memory or knowledge tool configured. Signs to look for: an available tool exposing a `remember` / `recall` / `relate`-style interface (for example a knowledge tool such as lean-ctx's `ctx_knowledge`), a graph-based issue tracker with its own memory command (for example `beads`, via `bd remember` or `bd note`), or an equivalent named in `AGENTS.md`.

- **A persistent memory tool is available** → use it as the source of truth. Go to [Using a persistent memory tool](#using-a-persistent-memory-tool). This is the default; reach for it first.
- **Nothing is configured** → fall back to flat files. Go to [Fallback: flat files](#fallback-flat-files).

Pick one store per project and stay in it for the life of the project. Do not maintain both for the same fact. Half the model living in a file nobody thinks to check, and half in memory nobody thinks to query, is worse than either alone.

## Using a persistent memory tool

Treat the tool as the glossary and the decision log combined; there is no separate `CONTEXT.md` to keep in sync alongside it.

- **Resolved terms** become a remembered fact: the term, its definition, and what to avoid saying instead, tagged with a category like `domain-term` or `glossary` so they are easy to recall as a set later. If the tool supports relating facts to each other, relate related terms so the glossary reads as a connected model, not a flat list.
- **Hard-to-reverse decisions** (the same three-part test as below) go in whichever construct the tool gives decisions specifically. Some tools have a distinct type for this, for example `beads` has a native `decision` issue type (`bd create "..." -t decision`), separate from its `task`/`feature`/`epic`/`bug` types. If the tool has no distinct decision type, remember it as a fact tagged `decision` with high confidence, still following the [decision criteria](#offer-to-record-a-decision-sparingly) below.
- **Before defining a term**, recall or search the store for it first, the same way you would grep `CONTEXT.md`. The challenge-and-sharpen behaviour below only works if existing terms are actually checked, not just added to.
- **Multiple contexts** (separate glossaries for genuinely separate domains within one project) map to whatever grouping mechanism the tool has, a category, a namespace, a tag, rather than the `CONTEXT-MAP.md` convention described in the fallback below.

## During the session (either store)

### Challenge against the glossary

When a term conflicts with an existing definition, call it out immediately. "The glossary defines 'cancellation' as X. You seem to mean Y. Which is it?"

### Sharpen fuzzy language

When a term is vague or overloaded, propose a precise canonical term. "You said 'account.' Do you mean the Customer or the User? Those are different things."

### Discuss concrete scenarios

When domain relationships come up, stress-test them with specific scenarios. Invent edge cases that force precision about the boundaries between concepts.

### Cross-reference with the implementation

When someone states how something works, check whether the implementation agrees. If there is a contradiction, surface it: "The code cancels an entire Order. You just said partial cancellation is possible. Which is right?"

### Record a resolved term immediately

Do not batch these up. The moment a term is resolved, write it to whichever store is in use, a remembered fact or a `CONTEXT.md` edit, right there in the session.

### Offer to record a decision sparingly

Only offer to record a decision when all three are true:

1. **Hard to reverse**: the cost of changing course later is meaningful.
2. **Surprising without context**: a future reader will wonder "why did they do it this way?"
3. **The result of a real trade-off**: genuine alternatives existed and one was picked for specific reasons.

If any of the three is missing, skip the record.

## Fallback: flat files

Use this section only when no persistent memory tool is configured for the project.

### File structure

Most projects have a single context:

```
/
├── CONTEXT.md
├── docs/
│   └── adr/
│       ├── 0001-slug.md
│       └── 0002-slug.md
└── ...
```

If a `CONTEXT-MAP.md` exists at the root, the project has multiple contexts. The map points to where each one lives:

```
/
├── CONTEXT-MAP.md
├── docs/
│   └── adr/                 ← system-wide decisions
├── ordering/
│   ├── CONTEXT.md
│   └── docs/adr/            ← context-specific decisions
└── billing/
    ├── CONTEXT.md
    └── docs/adr/
```

Create files lazily: only when there is something to write. If no `CONTEXT.md` exists, create one when the first term is resolved. If no `docs/adr/` exists, create it when the first decision needs recording.

### Update CONTEXT.md inline

When a term is resolved, update `CONTEXT.md` right away, using the format in [GLOSSARY-FORMAT.md](./GLOSSARY-FORMAT.md).

`CONTEXT.md` must contain only vocabulary. Do not treat it as a spec, a scratch pad, or a place for implementation decisions.

### Record a decision as an ADR

Use the format in [ADR-FORMAT.md](./ADR-FORMAT.md), applying the same three-part test above for whether a decision is worth recording at all.
