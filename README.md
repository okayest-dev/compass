# Compass

A tool-agnostic set of agent skills for planning and delivering work through a decisions, spec, tasks, implement pipeline. Compass takes its shape from Matt Pocock's skills package, replacing the JavaScript-specific examples with language-agnostic prose and preferring a persistent memory tool over markdown files for project knowledge.

## Install

OpenCode (and Claude-compatible agents) discover skills from a handful of fixed paths. Clone this repo, then symlink each skill folder into one of them:

- Global: `~/.config/opencode/skills/`, `~/.claude/skills/`, or `~/.agents/skills/`
- Project-local: `.opencode/skills/`, `.claude/skills/`, or `.agents/skills/`

```bash
git clone https://github.com/okayest-dev/compass.git ~/skills/compass
for skill in ~/skills/compass/compass-*; do
  ln -s "$skill" ~/.agents/skills/"$(basename "$skill")"
done
```

## The pipeline

```
compass-map ──▶ compass-spec ──▶ compass-navigate ──▶ compass-task ──▶ compass-horizon ──▶ compass-implement ──▶ compass-review
   (Epic)          (Feature)      (sanity check)         (Tasks)        (pick one up)        (build it)         (check it)
```

Most work does not start at `compass-map`. That skill exists for ideas too big or too vague for one session. Skip straight to `compass-navigate` or `compass-spec` for anything that fits in a normal working session.

| Skill | Role |
|---|---|
| `compass-map` | Decomposes an oversized or vague effort into waypoints (an Epic and its children) before a spec can be written. |
| `compass-spec` | Synthesises a spec from the current conversation and publishes it as a Feature. |
| `compass-navigate` | The gate every spec passes through: sanity-checks it against the codebase and glossary, resolves anything missing, and registers it as a Feature if nobody has yet. Also the general-purpose decision interview, used on its own for anything smaller than a full spec. |
| `compass-task` | Breaks a spec, a plan, or the current conversation into vertical-slice Tasks with explicit blocking edges. |
| `compass-horizon` | Picks the next unblocked, unclaimed Task off the backlog and claims it. Prefers the current Epic or Feature when one is in context. |
| `compass-implement` | Builds one Task: drives `compass-test`, commits, hands off for review. |
| `compass-review` | Two-axis review of a diff: does it follow this project's standards, and does it match the spec it claims to implement. |

Cross-cutting skills, called from wherever they're needed rather than as fixed pipeline steps:

| Skill | Role |
|---|---|
| `compass-glossary` | The project's shared vocabulary and decision history. A persistent memory tool first, `CONTEXT.md`/`docs/adr/` only as a fallback. |
| `compass-design` | Shared vocabulary for module depth, seams, and interfaces. Reference material, not a session to run. |
| `compass-test` | The red/green loop and its anti-patterns. Called by `compass-implement` per vertical slice. |
| `compass-prototype` | Throwaway code to answer a question that's cheaper to check by running than by reasoning about. |
| `compass-research` | Reads and understands a codebase, committing durable facts to whatever memory store the project uses. |

## Issue types

Every skill that creates tracked work names its type explicitly, rather than leaving it generic:

- **Epic**: `compass-map`'s map.
- **Feature**: `compass-spec`'s output, or whatever `compass-navigate` registers when a spec arrives from outside the conversation.
- **Task**: `compass-task`'s output, and `investigate`/`groundwork` waypoints on a map.
- **Decision**: a hard-to-reverse call recorded via `compass-glossary`, and `decide`/`prototype` waypoints on a map.

These map onto whatever the project's own tracker calls them (a Jira Story or Task, a GitHub or Linear label, a `beads -t` type, a local file's `Type:` field). No skill hardcodes one tracker's API. Each one lists two or three concrete examples and falls back to local files when nothing is configured.

## Best practices

**Let `compass-navigate` gate every spec, even one that looks complete.** An extensive spec handed in from outside the conversation still gets sanity-checked before `compass-task` touches it. Skipping this because the spec looks thorough is exactly the case it exists to catch: a gap that reads as obvious once you're implementing it and invisible on a first read.

**Pick one memory store per project and stay in it.** `compass-glossary` prefers a persistent memory tool over `CONTEXT.md` when one exists. Don't run both for the same fact. Half the model in a file nobody checks and half in memory nobody queries is worse than either alone.

**Use `compass-horizon` to pick up work, don't grab a task by hand.** It applies blocking order, epic preference, and the ready-for-implementer label consistently. Reaching around it to start on a task means those checks don't happen.

**Claim before you start, close when you're done.** Every task-touching skill treats the claim as the first write of a session, specifically so two sessions can't pick up the same task. If `compass-horizon` already claimed it, `compass-implement` doesn't re-claim it. Check before assuming you need to.

**Don't merge the two review axes.** `compass-review`'s Standards and Spec findings are reported side by side on purpose. A change can pass one and fail the other. Collapsing them into one ranked list hides that.

**There's no router skill.** Unlike Matt Pocock's `ask-matt`, nothing here decides which skill to use for you. Each `SKILL.md` states its own consumes, produces, and hands off at the top, so the whole pipeline is readable from any single file without a separate map of the system.

**Keep cross-references in sync when renaming a skill.** Renaming one skill (`compass-decide` became `compass-navigate`) means grepping every other `SKILL.md` for the old name, not just the ones you remember touching it. A stale reference points at a skill that no longer exists, and nobody notices until someone reads that exact line.

**No tool-specific language, no em dashes.** Examples name concrete tools (`beads`, Jira, GitHub, Linear) but the mechanism is always described generically first. Style follows the same rule throughout: periods and commas, not em dashes; sentence-case headings; pseudocode instead of language-specific snippets.

## Vocabulary

Compass uses a consistent nautical vocabulary rather than reusing terms from the packages it draws on, partly to avoid colliding with them:

- **Waypoint**: a single open question on a `compass-map` map. Not the same as a Task.
- **Horizon**: the set of open, unblocked, unclaimed items ready to pick up, whether waypoints on a map or tasks in the backlog.
- **Log**: the running record of resolved waypoints on a map.
- **Uncharted water**: work that's in scope but not yet sharp enough to turn into a waypoint.
- **Seam**, **interface**, **depth**, **leverage**, **locality**: `compass-design`'s vocabulary for module shape. Used consistently across `compass-spec`, `compass-task`, and `compass-test` rather than substituted with "component" or "boundary."
