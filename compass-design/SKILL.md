---
name: compass-design
description: Shared vocabulary for designing deep modules. Use when the user wants to design or improve a module's interface, find deepening opportunities, decide where a seam goes, make code more testable, or when another skill needs the deep-module vocabulary.
---

# Compass Design

Design **deep modules**: a lot of behaviour behind a small interface, placed at a clean seam, testable through that interface. Use this language and these principles wherever code is being designed or restructured. The aim is leverage for callers, locality for maintainers, and testability for everyone. This vocabulary is language-agnostic. It applies equally to a function, a class, a service, or a package, in any language.

## Consumes / produces / hands off

- **Consumes:** a module, cluster of modules, or interface under discussion.
- **Produces:** a shared vocabulary applied in conversation; no file output of its own.
- **Called from:** `compass-test` (seam placement for test doubles), `compass-implement` (interface shape before writing code), `compass-review` (naming smells against this vocabulary).
- **Hands off to:** [DEEPENING.md](./DEEPENING.md) for dependency-driven seam decisions; [DESIGN-IT-TWICE.md](./DESIGN-IT-TWICE.md) for comparing alternative interfaces; `compass-prototype` when the comparison needs running code, not just a description.

## Glossary

Use these terms exactly. Do not substitute "component," "service," "API," or "boundary." Consistent language is the whole point.

**Module**: anything with an interface and an implementation. Deliberately scale-agnostic: a function, class, package, or a slice spanning several deployable units. _Avoid_: unit, component, service.

**Interface**: everything a caller must know to use the module correctly. This includes the shape of its inputs and outputs, but also invariants, ordering constraints, error modes, required configuration, and performance characteristics. _Avoid_: API, signature (too narrow, since those refer only to the type-level surface).

**Implementation**: what is inside a module, its body of logic. Distinct from **Adapter**: a thing can be a small adapter with a large implementation (a database-backed store), or a large adapter with a small implementation (an in-memory fake). Reach for "adapter" when the seam is the topic; "implementation" otherwise.

**Depth**: leverage at the interface. The amount of behaviour a caller (or test) can exercise per unit of interface they have to learn. A module is **deep** when a large amount of behaviour sits behind a small interface, **shallow** when the interface is nearly as complex as the implementation.

**Seam** _(Michael Feathers)_: a place where you can alter behaviour without editing in that place. It is the *location* at which a module's interface lives. Where to put the seam is its own design decision, distinct from what goes behind it. _Avoid_: boundary (overloaded with domain-driven design's bounded context).

**Adapter**: a concrete thing that satisfies an interface at a seam. Describes *role* (what slot it fills), not substance (what is inside).

**Leverage**: what callers get from depth. More capability per unit of interface they have to learn. One implementation pays back across many call sites and many tests.

**Locality**: what maintainers get from depth. Change, bugs, knowledge, and verification concentrate in one place rather than spreading across callers. Fix once, fixed everywhere.

## Deep vs. shallow

**Deep module** = small interface + a lot of implementation:

```
┌─────────────────────┐
│   Small Interface   │  ← Few entry points, simple inputs
├─────────────────────┤
│                      │
│  Deep Implementation │  ← Complex logic hidden
│                      │
└─────────────────────┘
```

**Shallow module** = large interface + little implementation (avoid):

```
┌─────────────────────────────────┐
│       Large Interface           │  ← Many entry points, complex inputs
├─────────────────────────────────┤
│  Thin Implementation            │  ← Just passes through
└─────────────────────────────────┘
```

When designing an interface, ask:

- Can the number of entry points be reduced?
- Can the inputs be simplified?
- Can more complexity be hidden inside?

## Principles

- **Depth is a property of the interface, not the implementation.** A deep module can be internally composed of small, swappable parts; they are just not part of the interface. A module can have **internal seams** (private to its implementation, used by its own tests) as well as the **external seam** at its interface.
- **The deletion test.** Imagine deleting the module. If complexity vanishes, it was a pass-through. If complexity reappears across every caller, it was earning its keep.
- **The interface is the test surface.** Callers and tests cross the same seam. If a test needs to reach *past* the interface, the module is probably the wrong shape.
- **One adapter means a hypothetical seam. Two adapters means a real one.** Do not introduce a seam unless something actually varies across it.

## Designing for testability

Good interfaces make testing natural:

1. **Accept dependencies, do not create them.**

   ```
   // Testable: the caller supplies the dependency
   function processOrder(order, paymentGateway)

   // Hard to test: the module builds its own dependency internally
   function processOrder(order):
       gateway = newLiveGateway()
   ```

2. **Return results, do not produce side effects.**

   ```
   // Testable: the caller decides what to do with the result
   function calculateDiscount(cart) -> Discount

   // Hard to test: the effect is buried inside the call
   function applyDiscount(cart):
       cart.total -= discount
   ```

3. **Small surface area.** Fewer entry points means fewer tests are needed. Simpler inputs means simpler test setup.

## Relationships

- A **Module** has exactly one **Interface** (the surface it presents to callers and tests).
- **Depth** is a property of a **Module**, measured against its **Interface**.
- A **Seam** is where a **Module**'s **Interface** lives.
- An **Adapter** sits at a **Seam** and satisfies the **Interface**.
- **Depth** produces **Leverage** for callers and **Locality** for maintainers.

## Rejected framings

- **Depth as a ratio of implementation size to interface size**: rewards padding the implementation. Use depth-as-leverage instead.
- **"Interface" as a language keyword or a class's public methods**: too narrow, since interface here includes every fact a caller must know.
- **"Boundary"**: overloaded with domain-driven design's bounded context. Say **seam** or **interface**.

## Going deeper

- **Deepening a cluster given its dependencies**: see [DEEPENING.md](./DEEPENING.md) for dependency categories, seam discipline, and replace-don't-layer testing.
- **Exploring alternative interfaces**: see [DESIGN-IT-TWICE.md](./DESIGN-IT-TWICE.md) for comparing radically different interface designs on depth, locality, and seam placement.
