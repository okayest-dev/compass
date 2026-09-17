# Design It Twice

When someone wants to explore alternative interfaces for a chosen deepening candidate, use this parallel sub-agent pattern. Based on "Design It Twice" (Ousterhout): the first idea is unlikely to be the best.

Uses the vocabulary in [SKILL.md](./SKILL.md): **module**, **interface**, **seam**, **adapter**, **leverage**.

## Process

### 1. Frame the problem space

Before spawning sub-agents, write a user-facing explanation of the problem space for the chosen candidate:

- The constraints any new interface would need to satisfy.
- The dependencies it would rely on, and which category they fall into (see [DEEPENING.md](./DEEPENING.md)).
- A rough illustrative sketch of inputs and outputs to ground the constraints, not a proposal, just a way to make the constraints concrete.

Show this to the user, then proceed to step 2 immediately. The user reads and thinks while the sub-agents work in parallel.

### 2. Spawn sub-agents

Spawn three or more sub-agents in parallel. Each must produce a **radically different** interface for the deepened module.

Prompt each sub-agent with a separate technical brief (file paths, coupling details, dependency category from [DEEPENING.md](./DEEPENING.md), what sits behind the seam). The brief is independent of the user-facing problem-space explanation in step 1. Give each agent a different design constraint:

- Agent 1: "Minimise the interface: aim for one to three entry points at most. Maximise leverage per entry point."
- Agent 2: "Maximise flexibility: support many use cases and extension points."
- Agent 3: "Optimise for the most common caller: make the default case trivial."
- Agent 4 (if applicable): "Design around ports and adapters for every cross-seam dependency."

Include both the [SKILL.md](./SKILL.md) vocabulary and the project's `CONTEXT.md` vocabulary (via `compass-glossary`) in the brief, so each sub-agent names things consistently with both the architecture language and the project's domain language.

Each sub-agent outputs:

1. Interface (entry points, inputs, outputs, plus invariants, ordering, error modes).
2. A usage example showing how callers use it.
3. What the implementation hides behind the seam.
4. Dependency strategy and adapters (see [DEEPENING.md](./DEEPENING.md)).
5. Trade-offs: where leverage is high, where it is thin.

If the comparison would benefit from running code rather than a written description, use `compass-prototype` to build a slimmed-down comparison implementation for the strongest two or three candidates before presenting.

### 3. Present and compare

Present designs sequentially so the user can absorb each one, then compare them in prose. Contrast by **depth** (leverage at the interface), **locality** (where change concentrates), and **seam placement**.

After comparing, give a recommendation: which design is strongest and why. If elements from different designs would combine well, propose a hybrid. Be opinionated. The user wants a strong read, not a menu.
