# Investigation Format

Use this template for every investigation record. Write to the project's persistent memory tool if available; otherwise write to a local file under `investigations/<NN>-<slug>.md`.

---

## Investigation Record: `<NN>-<slug>`

**Type:** repro | code-search | prior-art | clarification | dependency-map
**Requirement:** <one-line summary of the investigation requirement>
**Date:** <YYYY-MM-DD>
**Status:** in-progress | complete
**Confidence:** high | medium | low

---

### Findings

#### For repro:
- **Scope:** unit | integration | contract
- **Test file:** <path to failing test>
- **Test command:** <single command to run>
- **Test output:** <captured red output>
- **Confirmed by user:** yes | no

#### For code-search:
- **Primary symbols searched:** <list>
- **Files found:** <list with one-line annotation each>
- **Canonical pattern:** <description>
- **Deviations noted:** <list>

#### For prior-art:
- **Search queries used:** <list>
- **Prior attempts:** <table: reference | outcome | link>
- **Recorded decisions:** <ADR/glossary references with rationale>
- **Rejections:** <.out-of-scope or equivalent links>

#### For clarification:
- **Ambiguities identified:** <list>
- **Questions asked:** <numbered list>
- **Answers received:** <numbered list matching questions>
- **Updated understanding:** <one paragraph>

#### For dependency-map:
- **Target of change:** <symbol/module/interface/data structure>
- **Dependencies:** <table: dependency | classification | test coverage>
- **Risk assessment:** <low/medium/high with rationale>

---

### Artifacts

- **Test files:** <list of paths>
- **File lists:** <list of paths>
- **Links:** <PRs, commits, ADRs, glossary entries>
- **Question/answer pairs:** <if clarification>

---

### Recommended next

- **Skill:** compass-navigate | compass-map | compass-task | compass-glossary | none
- **Rationale:** <one sentence why this skill follows from the findings>
- **Blocking questions:** <any questions that must be resolved before the next skill can proceed>

---

### Notes

<any additional context, caveats, or follow-up items>