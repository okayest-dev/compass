---
name: compass-map
description: Plan vague or large bodies of work (more than one agent session can hold) as a shared map of decision waypoints on your issue tracker (or local files), and resolve them one at a time until the way to the destination is clear.
---

# Compass Map

The user has come to you with an idea. It is unrefined and vague. There are still many unknowns. The destination is still uncharted: the way from here to the **destination** is not visible yet. This skill is about finding that way, not charging at the destination. It charts the way as a **shared map**, then works its **waypoints** (questions whose resolution is a decision, not slices of a build to execute) one at a time until the route is clear.

## Consumes / produces / hands off

- **Consumes:** a loose, oversized idea, a whole feature area, a migration, a strategic choice, too uncharted to spec directly.
- **Produces:** a map (destination, log, uncharted water, out-of-scope) and a set of resolved waypoints, each resolved via `compass-navigate`, `compass-prototype`, or direct investigation.
- **Called from:** used directly whenever an idea is too large or too uncharted for `compass-navigate` alone.
- **Hands off to:** `compass-spec`, once the map is clear enough that nothing is left to decide before someone writes the spec.

The destination varies per effort, and naming it is the first act of charting: it shapes every waypoint. It might be a spec to hand off and iterate on, a decision to lock before planning starts, or a change made in place, like a data migration. The map is domain-agnostic: engineering work, content planning, whatever fits the shape.

## Issue types

Decide the type before creating anything. Do not leave it untyped and deferred to a separate tracker-workflow doc.

- **The map itself is an Epic.** Use the tracker's closest equivalent: a Jira Epic, a GitHub or Linear issue labelled `epic`, a `beads` issue created with `-t epic`. If the tracker has no Epic concept at all, a plain top-level issue labelled `compass:map` stands in for it.
- **Every waypoint is a Task or a Decision**, never left generic. The [waypoint type](#waypoint-types) fixes which:
  - `investigate` and `groundwork` waypoints are **Tasks** (work items such as reading, provisioning, or moving data). Use `beads -t task`, a Jira Task, or a plain issue labelled `task`.
  - `prototype` and `decide` waypoints are **Decisions**. Use `beads -t decision` where the tracker has that type natively. Where it does not, use a Task and say so explicitly in the body ("this task resolves a decision, not a build step") so it is not mistaken for implementation work later.

## Plan, don't do

This skill is **planning** by default: each waypoint resolves a decision, and the map is done when the way is clear, with nothing left to decide before someone goes and does the thing. The pull to just do the work is usually the signal that the edge of the map has been reached and it is time to hand off. An effort can override this in its **Notes**, carrying execution into the map itself, but absent that, produce decisions, not deliverables.

## Refer by name

Every map and waypoint has a **name**: its title. In everything a human reads (narration, the map's Log), refer to it by that name, never by a bare id, number, or slug. A wall of numeric references is illegible; names read at a glance. The id and link do not vanish, since a name wraps its link, but they ride *inside* the name, never stand in for it.

## The map

The map is a single item, labelled `compass:map`, the canonical artifact. Its waypoints are its children.

The map is an **index**, not a store. It lists the decisions made and points at the waypoints that hold their detail; a decision lives in exactly one place, its waypoint, so the map never restates it, only gists it and links.

**Where the map, its waypoints, blocking, and horizon queries physically live is project-specific.** If this project documents an issue-tracker workflow (in `AGENTS.md`, a contributing guide, or similar), follow it for *mechanics*, meaning labels, fields, and API calls. The [issue types](#issue-types) above are fixed by this skill regardless of what a workflow doc says, since which type each item is affects how it is scheduled and reported on. If nothing is documented, default to plain files: a `decisions/map.md` for the map body, one file per waypoint under `decisions/waypoints/<slug>.md`, and a checklist in the map body to express blocking.

### The map body

The whole map at low resolution, loaded once per session. Open waypoints are **not** listed here: they are found by query (or by scanning `decisions/waypoints/` for files not yet closed).

```markdown
## Destination

<what reaching the end of this map looks like: the spec, decision, or change this effort is finding its way to. One or two lines; every session orients to it before choosing a waypoint.>

## Notes

<domain notes; skills every session should consult; standing preferences for this effort>

## Log

<!-- the index: one line per closed waypoint, enough to judge relevance, then zoom the link for the detail the waypoint holds -->

- [<closed waypoint title>](link): <one-line gist of the answer>

## Not yet specified

<!-- see "Uncharted water": in-scope territory you can't waypoint yet; graduates as the horizon advances -->

## Out of scope

<!-- see "Out of scope": work ruled beyond the destination; closed, never graduates -->
```

### Waypoints

Each waypoint is a **child item** of the map; its id (or filename) is its identity. Its body is the question, sized to one session:

```markdown
## Question

<the decision or investigation this waypoint resolves>
```

Each waypoint carries a `compass:<type>` label (or a `Type:` line in the local-file form), one of `investigate`, `prototype`, `decide`, `groundwork` (see [Waypoint Types](#waypoint-types)).

A session **claims** a waypoint by assigning it to the person or agent driving the map, **first**, before any work, so concurrent sessions skip it. That assignee *is* the claim: an open, unassigned waypoint is unclaimed.

Blocking uses the tracker's **native** dependency relationship when one exists: essential because it renders the horizon *visually* in the tracker's own UI, so anyone can see what's takeable without opening the map. Only fall back to a body/checklist convention when the tracker (or the local-file form) lacks native blocking. A waypoint is **unblocked** when every waypoint blocking it is closed; the **horizon** is the open, unblocked, unclaimed children, the edge of the known.

The answer is not part of the body; it is recorded on resolution (see [Work through the map](#work-through-the-map)). Assets created while resolving a waypoint are linked from it, not pasted in.

## Waypoint types

Every waypoint is either **live** (worked *with* a person who speaks for themselves) or **unattended**, driven by the agent alone. A live waypoint only resolves through that real exchange; the agent never stands in for the other side of it (an agent that answers its own interview questions has broken this). Each type also fixes the waypoint's tracker issue type, see [Issue types](#issue-types).

- **Investigate** (unattended, issue type: Task): reading documentation, third-party references, or local resources to surface a fact a decision waits on. Resolved by dispatching a sub-agent directly to find the fact. No separate skill needed. Use when knowledge outside the current working context is required.
- **Prototype** (live, issue type: Decision): raise the fidelity of the discussion by making a cheap, rough, concrete artifact to react to, using `compass-prototype`. Link the prototype as an asset. Use when "how should it look" or "how should it behave" is the key question.
- **Decide** (live, issue type: Decision): conversation. The default case. Use `compass-navigate`, and `compass-glossary` if anything resolved is worth recording.
- **Groundwork** (live or unattended, issue type: Task): manual work that must happen before a *decision* can be made. There is nothing to decide, prototype, or investigate, but the discussion is blocked until it is done. Signing up for a service so its capabilities can be judged, provisioning access, moving data so its shape can be seen. This is the one type that *does* rather than decides, and it earns its place by unblocking a decision, not by delivering the destination. The agent drives it alone where it can; otherwise it hands over a precise checklist. Resolved when the work is done; the answer records what was done and any resulting facts (where credentials live, new locations, row counts) later waypoints depend on.

## Uncharted water

The map is *deliberately* incomplete: do not chart what cannot yet be seen. Beyond the live waypoints lies **uncharted water**: the dim view of decisions and investigations that are clearly coming but cannot yet be pinned down, because they hang on questions still open. Resolving a waypoint charts the water ahead of it, graduating whatever is now specifiable into fresh waypoints, one at a time, until the way to the destination is clear and no waypoints remain.

The map's **Not yet specified** section is where that dim view is written down: the suspected question, the area to revisit later. It is the undiscovered horizon *toward* the destination: everything here is in scope, just not sharp enough to waypoint yet. Write as loosely or as fully as the view allows; it doubles as a signpost for collaborators reading where the effort is headed.

**Uncharted or waypoint?** The test is whether the question can be stated precisely now, *not* whether it can be answered now.

- **Waypoint when** the question is already sharp, even if it is blocked and cannot be acted on yet.
- **Not yet specified when** it cannot yet be phrased that sharply. Do not pre-slice uncharted water into waypoint-sized pieces: it is coarser than a waypoint, and one patch may graduate into several waypoints, or none, once the horizon reaches it.

**Not yet specified** excludes what is already decided (the Log), what is already a live waypoint, and what is out of scope (the next section).

## Out of scope

Uncharted water only ever gathers *toward* the destination. The destination fixes the scope, so work beyond it is **out of scope**: it is not uncharted water, and it does not belong in **Not yet specified**. It gets its own **Out of scope** section on the map: work consciously ruled out of *this* effort. Scope, not sharpness, lands it here.

Out-of-scope work never graduates (the horizon stops at the destination), so it returns only if the destination is redrawn, and then as a fresh effort, not a resumption.

Ruling something out of scope is a scoping act, not a step on the route. When a waypoint that already exists turns out to sit past the destination (mis-scoped in while charting, or exposed by a resolution), **close it** (a closed waypoint is unambiguously off the horizon) and leave one line in the **Out of scope** section: the gist plus why it is out of scope, linking the closed waypoint. It stays out of the **Log**, which records the route actually walked; a scope boundary is not a step on it.

## Invocation

Two modes. Either way, **never resolve more than one waypoint per session**, with the exception of investigate waypoints.

### Chart the map

Invoked with a loose idea.

1. **Name the destination.** Use `compass-navigate` (and `compass-glossary` if terms need pinning down) to settle what this map is finding its way to: the spec, decision, or change. The destination fixes the scope, so it is settled first.
2. **Map the horizon.** Interview again, **breadth-first** this time: fan out across the whole space rather than deep on any one thread, surfacing the open decisions and the first steps takeable now. **If this surfaces no uncharted water** (the way to the destination is already clear, the whole journey small enough for one session), a map is not needed. Stop and ask how to proceed instead.
3. **Create the map** (labelled `compass:map`): Destination and Notes filled in, Log empty, the uncharted water sketched into **Not yet specified**.
4. **Create the waypoints that can be specified now** as children of the map, then wire blocking edges in a **second pass** (items need ids before they can reference each other). Wiring sorts them into the horizon and the blocked; everything not yet specifiable stays uncharted: the **Not yet specified** section.
5. **Fire the investigate sub-agents.** For each `investigate` waypoint just created, dispatch a sub-agent to resolve it in parallel, capturing its findings with a pointer from the waypoint back to where they were written.
6. Stop: charting is one session's work; it hand-resolves nothing.

### Work through the map

Invoked with a map (a link or id). A waypoint is **optional**: without one, the next decision is chosen deliberately, not left to whoever asks.

1. Load the **map**: the low-res view, not every waypoint's full body.
2. Choose the waypoint. If one was named, use it. Otherwise take the first horizon waypoint in order. **Claim it**: assign it before any work.
3. Resolve it. **Zoom as needed**: fetch the full body of any related or closed waypoint on demand; use whichever skills the `## Notes` block names. If in doubt, use `compass-navigate` and `compass-glossary`.
4. Record the resolution: post the answer as a **resolution comment** (or write it into the waypoint file), **close** the waypoint, and **append a pointer** to the map's Log.
5. Add newly-surfaced waypoints (create, then wire); graduate any uncharted water the answer has made specifiable, clearing each graduated patch from **Not yet specified** so it lives only as its new waypoint. If the answer reveals that a waypoint (this one or another) sits beyond the destination, **rule it out of scope** rather than resolving it on the route. If the decision invalidates other parts of the map, update or delete those waypoints.

Waypoints without a blocking relationship between them may be worked in parallel, so expect other sessions to be editing the map concurrently.
