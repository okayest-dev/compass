# Compare prototype

Build **several radically different, slimmed-down implementations** of the same thing, presented side by side, so a choice between approaches is judged by looking at something concrete rather than argued over in the abstract. Whoever is choosing flips between candidates, picks one (or takes pieces from several), then the rest gets thrown away.

If the question is about driving one model through cases by hand rather than choosing between named alternatives, this is the wrong branch. Use [INTERACTIVE.md](./INTERACTIVE.md).

## When this is the right shape

- "What should this look like?" with more than one plausible direction.
- "I want to see a few options before committing to a layout, a data shape, or an interface."
- "Try a different approach to this and let's compare."
- `compass-design`'s "Design It Twice" comparison, when the comparison needs running code rather than a written description.
- Any time someone would otherwise spend a day weighing options in their head instead of looking at them side by side.

## Two sub-shapes: strongly prefer sub-shape A

A comparison is much easier to judge when it sits **next to the real context**: real surrounding data, real callers, real density. A standalone comparison in a vacuum makes every candidate look fine in isolation. Default to sub-shape A whenever there is a plausible existing place to host the candidates. Only reach for sub-shape B when the thing being compared genuinely has no home yet.

### Sub-shape A: adjustment to something existing (preferred)

The thing being compared already exists: a page, an interface, a module. Candidates are presented **in the same place**, gated by a simple switch (a query parameter for something UI-hosted, a flag or environment variable for something behind a command line, a selector argument for a function-level comparison). Whatever the thing already relies on (data, callers, context) stays the same. Only the candidate implementation swaps. This is the default; pick it unless there is a specific reason not to.

If the thing being compared does not exist yet but *would naturally live inside* something that does (a new section of an existing screen, a new case in an existing module), it is still sub-shape A. Mount the candidates inside the host.

### Sub-shape B: a new, standalone comparison (last resort)

Only use this when the thing being compared genuinely has no existing host (an entirely new surface, or something that cannot be embedded anywhere sensible).

Create a throwaway home for it, following whatever convention the project already uses for throwaway or scratch code. Name it so it is obviously a prototype (include the word "prototype" or "compare" in the path or filename). Same switch pattern as sub-shape A.

Before committing to sub-shape B, check: is there really no existing place this could live inside? A comparison built in a vacuum hides problems that one built in context would expose.

## Process

### 1. State the question and pick N

Default to **three candidates**. More than five stops being radically different and starts being noise, so cap there.

Write down the plan in one line, in the prototype's location or a comment at the top:

> "Three candidate layouts for the settings screen, switchable via a `variant` selector, hosted on the existing settings page."

This works whether someone is here to push back on it or not.

### 2. Generate radically different candidates

Draft each candidate. Hold each one to:

- The purpose of the thing being compared, and the data or context it has access to.
- Whatever conventions the project already has (styling system, interface style, naming). Do not invent new conventions just for the comparison.
- A clear, distinct name for each candidate: `A`, `B`, `C`, or short descriptive names.

Candidates must be **structurally different**: different shape, different structure, different primary approach, not just cosmetic differences. Three lightly-tweaked versions of the same idea is not a comparison, it is wallpaper. If two drafts come out too similar, redo one under an explicit constraint that rules out the shared structure.

### 3. Wire them together

Build a single switch at the comparison's entry point that selects which candidate is active, and pass it whatever context that candidate needs. Whatever the medium:

- For sub-shape A (something existing): keep everything the host already does, its data, its inputs, its surrounding context, above the switch. Only the part being compared changes per candidate.
- For sub-shape B (a new, standalone comparison): the throwaway home mounts the same switch.

### 4. Build the switcher

Something small that lets whoever is judging move between candidates without restarting from scratch:

- **For something UI-hosted**: a small, visually distinct control (a bar, a pill) that is obviously not part of the design being judged, with a way to step forward and back between candidates and see which one is active. Make the current candidate reflected in something shareable (a URL, a saved state) so it survives a reload. Gate it so it cannot ship into a real build, using whatever mechanism the project uses to distinguish throwaway code from production code.
- **For something behind a command line or a script**: a flag, an argument, or a simple prompt that selects the candidate, with the active one printed clearly each run.
- **For a function-level comparison**: a small harness that runs each candidate against the same inputs and prints the outputs next to each other.

Put the switcher in one shared place so every candidate reuses it, wherever this kind of shared support code already lives in the project.

### 5. Hand it over

Surface however the comparison is reached (a URL, a command, a script name) and the candidate identifiers. Whoever is judging flips through whenever they get to it. The interesting feedback is usually **"I want this part of A with that part of B"**, which is the actual design being asked for.

### 6. Capture the answer and clean up

Once a candidate has won, capture the answer (which one, and why), then capture the prototype the way [SKILL.md](./SKILL.md) describes. Fold the winner into the real code and move the rest onto the throwaway branch, not into the main line:

- **Sub-shape A**: fold the winner into the existing host; drop the losing candidates and the switcher from the main line.
- **Sub-shape B**: promote the winning candidate to its real place; drop the throwaway home and the switcher from the main line.

The full set of candidates is the primary source, so it lands on the throwaway branch, not the bin. Candidate code and switchers left in the main line rot fast and confuse the next reader.

## Anti-patterns

- **Candidates that differ only cosmetically.** That is a tweak, not a comparison. Real candidates disagree about structure.
- **Sharing too much between candidates.** A shared header or a shared entry signature is fine; sharing the whole structure defeats the point. Each candidate should be free to diverge.
- **Wiring candidates to real side effects.** Read-only or side-effect-free comparisons are fine. If a candidate needs to write somewhere real, point it at a stub. The question is "which shape is better," not "does the write path work."
- **Promoting a candidate directly into production as-is.** Candidate code was written under prototype constraints (no tests, minimal error handling). Rewrite it properly when folding it in.
