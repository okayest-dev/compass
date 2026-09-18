---
name: compass-navigate
description: Interview relentlessly to sharpen a plan, decision, or idea until there's a shared understanding, or sanity-check an existing spec and register it as a Feature before compass-task runs. Use when stress-testing thinking, resolving an open decision, or when a spec has been provided and needs vetting before implementation starts.
---

# Compass Navigate

Interview whoever owns the decision relentlessly until there is a shared understanding. Map this as a **decision tree**: every decision branches into the decisions that hang off it.

## Consumes / produces / hands off

- **Consumes:** an idea, plan, or open decision that is not yet sharp enough to act on, or an existing spec, self-authored or handed in from outside this conversation, that needs sanity-checking before implementation starts.
- **Produces:** a resolved decision, recorded via `compass-glossary` when it is hard to reverse. For a spec, a confirmed (or corrected) spec registered as a Feature-type issue.
- **Called from:** `compass-map` (one waypoint at a time); directly whenever a decision needs sharpening before `compass-spec` can be written; directly as the first port of call whenever a spec is provided, before `compass-task` touches it.
- **Hands off to:** `compass-glossary` to record anything hard to reverse; `compass-spec` once enough decisions exist to write a fresh spec; `compass-task` once a provided spec has been sanity-checked and registered as a Feature; `compass-prototype` when a question is better answered by running code than by more questions.

## Run the interview

Work the tree in **rounds**. The **horizon** is every decision whose prerequisites are already settled. These are the questions that can be asked *now* without guessing at answers not yet heard. Ask the whole horizon in one round. Number each question and give a recommended answer. Then wait for the answers before the next round.

Format a round like this:

```
Q1 - <question title>: <question body, may be multiple paragraphs, including multiple choices>

Recommended: <recommended answer>

---

Q2 - <question title>: <question body, may be multiple paragraphs, including multiple choices>

Recommended: <recommended answer>
```

Each round's answers reshape the tree. Settled decisions push the horizon outward and unblock questions that depended on them. Recompute the horizon and ask the next round. A question whose answer depends on another question still open this round belongs to a *later* round, not this one.

Finding **facts** is always this skill's job, never the other party's. When a horizon question needs a fact from the environment (files, tools, existing code, prior records), dispatch a sub-agent to find it. Do not ask for anything that can be looked up directly. Do not block on it either: a running lookup is an unsettled prerequisite, so only the questions downstream of it wait. Ask the rest of the horizon now. The **decisions** themselves belong to the person being interviewed. Put each to them and wait.

If a question is better answered by comparing running code than by more rounds of questions, pause and use `compass-prototype` to build a slimmed-down comparison, then resume the interview with the result in hand.

The session is done when the horizon is empty: every branch of the decision tree visited, nothing left silently assumed. Do not act on the outcome until it is confirmed that a shared understanding has been reached.

## Sanity-checking a provided spec

When a spec already exists, whether just written by `compass-spec` in this conversation or handed in from outside it, this is the first port of call, before `compass-task` touches it. Do not re-litigate decisions the spec already states clearly. Only interrogate what is missing, contradictory, or assumed.

### 1. Read the whole spec first

Read it fully before forming a single question. A partial read produces sanity-check questions the rest of the document already answers.

### 2. Build the horizon from gaps, not from scratch

Walk the spec's sections (User Stories, Implementation Decisions, Testing Decisions, or their equivalents) against:

- **The codebase**: does anything the spec assumes about existing modules, seams, or data actually hold? Use `compass-design` if a claimed seam doesn't match what is there.
- **The glossary**: does the spec's vocabulary match the recorded model (see `compass-glossary`)? Flag terms used inconsistently with it.
- **Internal consistency**: do two sections of the spec contradict each other, for example a User Story implying behaviour the Implementation Decisions rule out, or the reverse?
- **Completeness**: is there a decision a reasonable implementer would need that the spec is silent on?

Only what fails one of these checks becomes a horizon question. A spec with nothing wrong produces an empty horizon. Say so, and go straight to registration (step 4). Do not run an interview to fill time.

### 3. Run the interview on the horizon, if one exists

Use the same rounds/horizon mechanics as [Run the interview](#run-the-interview) above, scoped only to what step 2 surfaced. Resolve each round the same way: recommend, wait, recompute.

If an answer contradicts something the spec states outright, update the spec in place rather than letting the interview's answer live only in this conversation. A sanity check that corrects the spec but never writes the correction back has not actually fixed anything.

### 4. Register the spec as a Feature

Once the horizon is empty, meaning nothing was wrong or everything found is now resolved, confirm the spec has a durable, typed home before handing off to `compass-task`:

- **Already registered** (`compass-spec` wrote it in this conversation, or it already exists as a typed issue): nothing to do.
- **No durable home yet** (pasted text, a linked external document, anything not tracked by this project's own tools): create one now. Publish it as a **Feature**, using the tracker's closest equivalent: a Jira Story, a GitHub or Linear issue labelled `feature`, or a `beads` issue created with `-t feature`. Use the spec's own content as the body. This is the same issue type `compass-spec` would have used had it authored the spec itself; `compass-decide` registers it when nobody else did.
- **A durable home exists but is untyped** (an issue with no type set, a wiki page): retype or link it so it reads as a Feature going forward, rather than leaving `compass-task` to guess at a parent reference.

Hand off to `compass-task` only once the spec is sane and registered.

## Recording decisions as you go

Not every decision needs a paper trail. Use `compass-glossary` alongside this skill when:

- a term surfaces that conflicts with or sharpens the existing glossary, or
- a decision, from either an open-ended interview or a spec sanity check, is hard to reverse, surprising without context, and the result of a real trade-off (see `compass-glossary`'s recording criteria).

For a quick, disposable interview where nothing needs to persist, thinking out loud about a small choice, skip `compass-glossary` entirely. Whether to record is a judgment call each round, not an upfront mode switch.
