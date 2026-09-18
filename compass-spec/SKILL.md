---
name: compass-spec
description: Turn the current conversation into a spec and publish it. No interview, just synthesis of what has already been discussed.
---

# Compass Spec

Take the current conversation and codebase understanding and produce a spec. Do **not** interview. Just synthesise what is already known. If a decision is still open, hand back to `compass-navigate` (or `compass-map`, for a large effort) before writing the spec, rather than guessing.

## Consumes / produces / hands off

- **Consumes:** a conversation (and its resolved decisions), typically the output of `compass-navigate` directly, or one feature grouping from a cleared `compass-map` (see that skill's "Splitting into features"). A single call covering the whole map is normal when the map turns out to be one feature.
- **Produces:** a spec document, published as a **Feature**-type issue wherever this project tracks work.
- **Called from:** directly, once enough decisions exist to describe a feature; once per feature identified when a `compass-map` clears.
- **Hands off to:** `compass-navigate`, to sanity-check the finished spec and confirm its Feature registration, before `compass-task` breaks it into implementation-ready tasks. For a spec just authored here, that pass is typically fast, since the decisions behind it were already interrogated on the way in, but it still runs. Every spec reaches `compass-task` through the same gate regardless of how it was produced.

## Issue type

Publish the spec as a **Feature** (use the tracker's closest equivalent: a Jira Story, a GitHub or Linear issue labelled `feature`, a `beads` issue created with `-t feature`). If this spec came from a `compass-map`, link the Feature as a child of the map's Epic. Do not publish the spec as a generic, untyped issue. The type is what lets `compass-task` and `compass-review` find it later without guessing. (When a spec arrives without going through this skill at all, handed in from outside the conversation, `compass-navigate` performs this same registration; see its "Sanity-checking a provided spec" section.)

## Process

1. Explore the repository to understand the current state of the code, if that has not already been done. Use the project's domain glossary vocabulary throughout the spec (see `compass-glossary`), and respect any recorded decisions in the area being touched.

2. Sketch out the seams at which the feature will be tested (see `compass-design` for the vocabulary). Existing seams should be preferred to new ones. Use the highest seam possible. If new seams are needed, propose them at the highest point possible. The fewer seams across the codebase, the better. The ideal number is one.

   Check that these seams match expectations before moving on.

3. Write the spec using the template below, then publish it as a Feature (see [Issue type](#issue-type) above): as an issue on this project's tracker if one is documented, otherwise a file under a local `specs/` directory, with `Type: feature` noted at the top. Apply whatever label or tag this project uses to mark work as ready for an implementer. If no such convention exists yet, say so explicitly at the top of the spec rather than inventing one.

<spec-template>

## Problem Statement

The problem being faced, from the user's perspective.

## Solution

The solution to the problem, from the user's perspective.

## User Stories

A long, numbered list of user stories. Each user story should be in the format:

1. As a `<actor>`, I want `<feature>`, so that `<benefit>`.

Example:

> As a mobile bank customer, I want to see the balance on my accounts, so that I can make better informed decisions about my spending.

This list should be extensive and cover all aspects of the feature.

## Implementation Decisions

A list of implementation decisions that were made. This can include:

- The modules that will be built or modified.
- The interfaces of those modules that will change.
- Technical clarifications gathered along the way.
- Structural decisions.
- Schema changes.
- Interface contracts.
- Specific interactions.

Do **not** include specific file paths or code snippets. They tend to go out of date quickly.

Exception: if `compass-prototype` produced a snippet that encodes a decision more precisely than prose can (a state machine, a schema, a shape of data), inline it within the relevant decision and note briefly that it came from a prototype. Trim it to the decision-rich parts, not a working demo, just the important bits.

## Testing Decisions

A list of testing decisions that were made. Include:

- A description of what makes a good test here (only test observable behaviour, not implementation details, see `compass-test`).
- Which modules will be tested.
- Prior art for the tests: similar tests already in the codebase.

## Out of Scope

A description of what is out of scope for this spec.

## Further Notes

Any further notes about the feature.

</spec-template>
