# Decision record format

Fallback format only. Use this when no persistent memory tool is configured for the project. See [SKILL.md](./SKILL.md#pick-a-store-once-before-writing-anything). When a memory tool is available, record decisions there instead (using its native decision type if it has one).

Decision records live in `docs/adr/` and use sequential numbering: `0001-slug.md`, `0002-slug.md`, and so on.

Create the `docs/adr/` directory lazily: only when the first record is needed.

## Template

```md
# {Short title of the decision}

{One to three sentences: what was the context, what did we decide, and why.}
```

That is the whole template. A record can be a single paragraph. The value is in recording *that* a decision was made and *why*, not in filling out sections.

## Optional sections

Include these only when they add genuine value. Most records will not need them.

- **Status** frontmatter (`proposed | accepted | deprecated | superseded by ADR-NNNN`): useful when decisions get revisited.
- **Considered options**: only when the rejected alternatives are worth remembering.
- **Consequences**: only when non-obvious downstream effects need calling out.

## Numbering

Scan `docs/adr/` for the highest existing number and increment by one.

## When to record a decision

All three of these must be true:

1. **Hard to reverse**: the cost of changing course later is meaningful.
2. **Surprising without context**: a future reader will look at the result and wonder "why on earth did they do it this way?"
3. **The result of a real trade-off**: genuine alternatives existed and one was picked for specific reasons.

If a decision is easy to reverse, skip it. You will just reverse it. If it is not surprising, nobody will wonder why. If there was no real alternative, there is nothing to record beyond "we did the obvious thing."

### What qualifies

- **Structural shape.** "The system is split into three deployable units." "The write path is event-sourced; the read path is a derived projection."
- **Integration patterns between contexts.** "Ordering and Billing communicate through published events, not direct calls."
- **Dependency choices that carry lock-in.** Datastore, message bus, identity provider, deployment target. Not every library, only the ones that would take real effort to replace.
- **Boundary and scope decisions.** "Customer data is owned by the Customer context; other contexts reference it by identifier only." The explicit no's are as valuable as the yes's.
- **Deliberate deviations from the obvious path.** "We built this by hand instead of using an existing library, because X." Anything where a reasonable reader would assume the opposite. These stop the next person from "fixing" something that was deliberate.
- **Constraints not visible in the result itself.** "We cannot use a particular provider because of a compliance requirement." "Response times must stay under a fixed threshold because of a partner agreement."
- **Rejected alternatives when the rejection is non-obvious.** If an alternative approach was seriously considered and set aside for subtle reasons, record it. Otherwise someone will propose it again later.
