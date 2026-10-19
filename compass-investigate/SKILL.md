---
name: compass-investigate
description: Targeted fact-finding for a specific investigation requirement. Use when you need to reproduce a bug, search code, find prior art, clarify requirements, or map dependencies before deciding how to proceed.
---

# Compass Investigate

Investigate a specific question to produce structured findings that can be acted upon. This skill is the fact-finding layer: it answers "what are the facts?" so downstream skills can decide "what do we do?"

## Consumes / produces / hands off

- **Consumes:** an investigation requirement — a bug report ("X is broken"), a code question ("how does A interact with B?"), a prior-art query ("has this been tried before?"), a clarification need ("what does Y mean?"), or a dependency-mapping request ("what breaks if I change X?")
- **Produces:** an investigation record with typed findings, confidence level, artifacts, and a recommended next skill
- **Called from:** directly as an entry point; from `compass-navigate` when a decision interview needs facts; from `compass-map` for `investigate` waypoints
- **Hands off to:** `compass-navigate` (decisions needing interview), `compass-map` (large/unknown efforts), `compass-task` (ready-to-implement work), `compass-glossary` (decisions worth recording — not bugs), or terminal (findings stand alone)

## Investigation types

Every investigation is one of five types. The type determines the process and output shape.

### 1. Repro — "X is broken"

Confirm a bug by writing a failing test at the appropriate scope.

**Process:**
1. Infer test scope from the bug report:
   - Single module / function → unit test
   - Cross-module within one app → integration test
   - Cross-app / service boundary → contract test (both apps in harness)
2. Confirm inferred scope with the user: "This looks like a [unit/integration/contract] bug. Should I write a failing test at that level?"
3. Write the minimal failing test that reproduces the bug. Follow `compass-test` TDD standards: red before green, one slice at a time, test observable behaviour through public interfaces.
4. Run the test to confirm it fails (red). Capture output.
5. If the test cannot be written at the inferred scope (missing harness, no test infrastructure), escalate to the user with what's needed.

**Output:** failing test file, test output, confirmed scope.

### 2. Code search — "How does A interact with B?"

Find existing implementations, conventions, and call patterns.

**Process:**
1. Grep for the primary symbols (function names, types, routes, events).
2. Explore the matched files to understand the convention: naming, error handling, validation, test structure.
3. Trace callers/callees to map the interaction.
4. Identify the canonical pattern — the one the codebase uses consistently.

**Output:** file list with annotations, canonical pattern description, deviations noted.

### 3. Prior art — "Has this been tried before?"

Search history for previous attempts, decisions, and rejections.

**Process:**
1. Search git log for relevant commits (keywords, file paths, authors).
2. Search PRs/issues for related discussions (open, closed, merged, rejected).
3. Check ADRs (`docs/adr/`) and glossary (`compass-glossary`) for recorded decisions.
4. Check for `.out-of-scope` or equivalent rejection records.

**Output:** list of prior attempts with outcomes, links to PRs/commits/ADRs, any recorded decision with rationale.

### 4. Clarification — "What does Y mean?"

Structured questions to resolve ambiguity before acting.

**Process:**
1. Identify the specific ambiguities in the requirement.
2. Draft precise, answerable questions (not "please provide more info").
3. Present questions to the user, one round at a time.
4. Incorporate answers and re-run the investigation if needed.

**Output:** question list, answers received, updated understanding.

### 5. Dependency map — "What breaks if I change X?"

Map the impact radius of a proposed change.

**Process:**
1. Identify the symbol, module, interface, or data structure to change.
2. Find all callers, implementers, test files, and configuration references.
3. Classify each dependency: direct caller / implementer / test / config / indirect.
4. Note which dependencies have tests that would catch breakage.

**Output:** dependency list with classifications, test coverage notes, risk assessment.

## Process (general)

### 1. Clarify the investigation requirement

If the input is vague, ask: "What specific question should this investigation answer?" Do not begin until the question is precise.

### 2. Classify the type

Match the requirement to one of the five types above. If it spans multiple, run each as a separate investigation and aggregate.

### 3. Execute the type-specific process

Follow the process for the classified type. For repro, confirm scope with user before writing tests. For clarification, ask structured questions in rounds.

### 4. Assess confidence

Rate the investigation outcome:

- **high** — question fully answered, no open sub-questions, findings are actionable
- **medium** — partially answered, some gaps remain but core finding is solid
- **low** — inconclusive, significant gaps, more investigation needed

### 5. Produce the investigation record

Write the record using `INVESTIGATION-FORMAT.md`. Include:
- Type and original requirement
- Findings (structured per type)
- Confidence rating
- Artifacts (test files, file lists, links, question/answer pairs)
- Recommended next skill: `compass-navigate` / `compass-map` / `compass-task` / `compass-glossary` / none

### 6. Hand off

If confidence is high and a next skill is recommended, hand off explicitly. If confidence is medium/low or open questions remain, stay in investigate and iterate.

## Repro standards

See `REPRO-GUIDELINES.md` for detailed standards. Summary:
- TDD: write failing test first, confirm red, then (optionally) make green
- Scope: infer from bug report (unit/integration/contract), confirm with user
- Test observable behaviour through public interfaces (see `compass-test`)
- No implementation code — only the failing test
- Single command to run the test

## Clarification protocol

- Questions must be specific and actionable
- Present one round at a time; wait for answers before proceeding
- If clarification reveals a decision is needed (not just a fact), recommend `compass-navigate`
- If clarification reveals the effort is large/unknown, recommend `compass-map`

## Incremental investigation

When called from `compass-navigate` during an interview:
- Accept a focused follow-up question ("also check if X handles Y")
- Extend the existing investigation record rather than starting fresh
- Update confidence and recommended next if the new findings change the picture

## Boundary rules

- **vs. `compass-research`**: Investigate = targeted fact-finding for a specific question. Research = deep codebase understanding for design. If the goal is "understand this codebase," use research. If the goal is "answer this question," use investigate.
- **vs. `compass-prototype`**: Investigate = bug repro via failing test. Prototype = design questions via throwaway running code. If the question is "how should it feel/behave?", use prototype.
- **vs. `compass-navigate`**: Investigate produces facts. Navigate conducts the decision interview. Navigate may invoke investigate for facts; investigate never conducts the interview.
- **Persistence**: No persistent memory for investigations. Findings that become decisions go to `compass-glossary`. Bug repro tests become part of the implementation via `compass-task`/`compass-implement`.