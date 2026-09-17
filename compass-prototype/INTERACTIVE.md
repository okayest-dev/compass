# Interactive prototype

A single, self-contained, driveable artifact that lets anyone push a state model, workflow, or interface through cases by hand. Use this when the question is about **behaviour, state transitions, or a shape of interaction**: the kind of thing that looks reasonable on paper but only feels wrong once it is pushed through real cases.

Because it is one artifact with nothing to install, it can be handed to anyone, a designer, a domain expert, a teammate unfamiliar with the code, and they can feel the model for themselves. It should speak their language, not the code's.

If the question is "which approach should we pick between named alternatives," this is the wrong branch. Use [COMPARE.md](./COMPARE.md).

## When this is the right shape

- "I'm not sure this state machine handles the edge case where X then Y."
- "Does this data model actually let me represent the case where…"
- "I want to feel out what this interface should look like before writing it for real."
- "What should this screen or flow look like when driven end to end?"
- Anything where someone wants to **take an action and watch what happens**.

## Process

### 1. State the question

Before writing anything, write down what model and what question is being prototyped. One paragraph, at the top of the artifact, visible to whoever runs it, not just a comment. An interactive prototype that answers the wrong question is pure waste, so make the question explicit enough to check later, whether the person is watching now or comes back to it afterwards.

### 2. Isolate the logic in a portable piece

Put the actual logic that is answering the question, the model, the transitions, the rules, in a single, small, self-contained piece that could be lifted out and dropped into the real codebase later. The shell around it is throwaway; this piece is not.

The right shape depends on the question:

- **A pure reducer**: `(state, action) -> state`. Good when actions are discrete events and state is a single value.
- **A state machine**: explicit states and transitions. Good when "which actions are even legal right now" is part of the question.
- **A small set of pure functions** over a plain data shape. Good when there is no implicit current state, just transformations.
- **A module with a clear, small method surface** when the logic genuinely owns ongoing internal state.

Pick whichever shape best fits the question, *not* whichever is easiest to wire to a shell. Keep it pure: nothing in it should reach out to a display, a network call, or an input handler. The shell calls into it; nothing flows the other way. This is what makes the prototype useful past its own lifetime: once the question is answered, the validated piece lifts into the real module on its own.

### 3. Build the driveable shell

Choose the simplest medium that fits the intended audience and the project:

- **A single self-contained page or document** (plain markup, no build step, no server) when the audience includes non-developers, or when the question is inherently about a visual or interactive surface. It should open directly with nothing to install and survive being shared as a single file.
- **A small command-line or REPL harness** when the audience is technical and the question is purely about logic or data shape, with no visual component.
- **A minimal screen inside the project's existing shell** (following whatever the project already uses to host a throwaway view) when the question is about how something feels alongside real surrounding context, and a fully standalone artifact would lose that context.

Whichever medium is chosen, write it for the intended audience. Every label is in **domain language**, not implementation language: actions and state read like the situation being modelled, not the internals. Explain in plain words what is happening.

Lay it out with a clear hierarchy:

1. **Title and a one-line explanation** of what this lets someone explore (the question from step 1).
2. **Current state**: the full relevant state, shown as something readable (labelled fields, not a raw dump), refreshed after every action so the change is visible. Where it helps, call out what just changed.
3. **Free-play actions**: one action per available operation, always available, so anyone can poke at the model in any order. Each action updates and re-shows the state.
4. **Guided walkthroughs**: a set of **scenarios**. Each scenario holds a short plain-language description (the situation it sets up and what to watch for) and, underneath it, the ordered actions to take for that scenario. Each step is a real action: taking it performs that step and moves to the next one. Starting a walkthrough resets to a known initial state so the scenario runs the same way every time.

Choose scenarios that demonstrate the awkward cases, the ones hard to reason about on paper: the expected path, a tricky edge case, an attempt at something that should not be possible.

Keep it plain and restrained: clear structure, generous spacing, nothing that competes with the state and the actions for attention.

### 4. Hand it over

Send it, or run it together. Whoever is judging will work through the walkthroughs and free-play whenever they get to it; the interesting moments are when they say "wait, that shouldn't be possible" or "huh, I assumed X would be different." Those are bugs in the *idea*, which is the whole point. If they want a new action or a new scenario, add it. Prototypes evolve.

### 5. Capture the answer and the prototype

Once the prototype has answered its question, capture the answer, then capture the prototype the way [SKILL.md](./SKILL.md) describes. The specific mapping here: the validated piece (the reducer, machine, or function set) lifts into the real module as the decision, absorbed; the shell around it rides along to the throwaway branch that keeps the prototype as a primary source, and being small and self-contained, it stays easy to re-run there.

## Anti-patterns

- **Don't add tests.** A prototype that needs tests is no longer a prototype.
- **Don't wire it to the real datastore.** Use in-memory state unless the question is specifically about persistence.
- **Don't generalise.** No "what if this needs to support X later." The prototype answers one question.
- **Don't blur the logic and the shell together.** If the pure piece reaches into a display, a network call, or an input handler, it is no longer liftable. Keep the shell as a thin layer over a pure piece.
- **Don't reach for a full framework, bundler, or server** unless the project's own conventions make that the simplest option. The simplest thing that runs and can be shared is the point.
- **Don't ship the shell into production.** The shell is optimised for being driven by hand. The logic behind it is the part worth keeping.
