# CONTEXT.md format

Fallback format only. Use this when no persistent memory tool is configured for the project. See [SKILL.md](./SKILL.md#pick-a-store-once-before-writing-anything).

## Structure

```md
# {Context name}

{One or two sentences: what this context is and why it exists.}

## Language

**Order**:
{A one or two sentence description of the term.}
_Avoid_: Purchase, transaction

**Invoice**:
A request for payment sent to a customer after delivery.
_Avoid_: Bill, payment request

**Customer**:
A person or organization that places orders.
_Avoid_: Client, buyer, account
```

## Rules

- **Be opinionated.** When several words exist for the same concept, pick the best one and list the others under `_Avoid_`.
- **Keep definitions tight.** One or two sentences at most. State what the term IS, not what it does.
- **Only include terms specific to this project's domain.** General programming concepts (timeouts, error types, retry policies) do not belong here even if the project uses them heavily. Before adding a term, ask: is this concept unique to this domain, or a general engineering concept? Only the former belongs.
- **Group terms under subheadings** when natural clusters emerge. A flat list is fine when everything belongs to one cohesive area.

## Single vs. multi-context projects

**Single context (most projects):** one `CONTEXT.md` at the project root.

**Multiple contexts:** a `CONTEXT-MAP.md` at the project root lists the contexts, where each lives, and how they relate:

```md
# Context Map

## Contexts

- [Ordering](./ordering/CONTEXT.md): receives and tracks customer orders
- [Billing](./billing/CONTEXT.md): generates invoices and processes payments
- [Fulfillment](./fulfillment/CONTEXT.md): manages warehouse picking and shipping

## Relationships

- **Ordering → Fulfillment**: Ordering emits an OrderPlaced signal; Fulfillment consumes it to start picking.
- **Fulfillment → Billing**: Fulfillment emits a ShipmentDispatched signal; Billing consumes it to generate an invoice.
- **Ordering ↔ Billing**: shared identifiers for Customer and Money.
```

Infer which structure applies:

- If `CONTEXT-MAP.md` exists, read it to find the contexts.
- If only a root `CONTEXT.md` exists, treat the project as a single context.
- If neither exists, create a root `CONTEXT.md` lazily when the first term is resolved.

When multiple contexts exist, infer which one the current topic relates to. If it is unclear, ask.
