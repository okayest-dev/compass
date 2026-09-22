---
name: compass-review
description: "Review the changes since a fixed point (commit, branch, tag, or merge-base) along three axes: Standards (does the code follow this project's documented coding standards?), Spec (does the code match what the originating spec asked for?), and Security (does the change introduce or fail to mitigate a security weakness?). Runs all three as parallel sub-agents, reports them side by side, and ends in a Clear or Blocked verdict that gates whether the work is done. Use when reviewing a branch, work-in-progress changes, or asked to review since a given point, or whenever a security review is specifically requested."
---

# Compass Review

Three-axis review of the diff between `HEAD` and a fixed point:

- **Standards**: does the code conform to this project's documented coding standards?
- **Spec**: does the code faithfully implement the originating spec?
- **Security**: does the change introduce a security weakness, or fail to guard a trust boundary, sensitive value, or authorisation check the spec named?

## Consumes / produces / hands off

- **Consumes:** a diff (a branch, a set of commits, work-in-progress changes) and, where one exists, the spec it implements, including its Security Decisions section (see `compass-spec`).
- **Produces:** a three-axis findings report ending in a **Clear** or **Blocked** verdict (see [Verdict](#6-aggregate-and-verdict)).
- **Called from:** `compass-implement`, once a piece of work is done, in a loop until the verdict is Clear (see that skill's process). Can also be run standalone against any diff.
- **Hands off to:** back to `compass-implement` for fixes when Blocked; to whoever owns the task when a blocking finding needs an accepted exception rather than a fix (see [Accepting a blocking finding](#accepting-a-blocking-finding)).

All three axes run as **parallel sub-agents** so they do not pollute each other's context, then this skill aggregates their findings and computes the verdict.

## Process

### 1. Pin the fixed point

Use whatever fixed point was given (a commit, a branch name, a tag, `main`, `HEAD~5`). If none was given, ask for one.

Capture the diff command once: `git diff <fixed-point>...HEAD` (three-dot, so the comparison is against the merge-base). Also note the commit list via `git log <fixed-point>..HEAD --oneline`.

Before going further, confirm the fixed point resolves (`git rev-parse <fixed-point>`) and the diff is non-empty. A bad reference or an empty diff should fail here, not inside three parallel sub-agents.

### 2. Identify the spec source

Look for the originating spec, a **Feature**-type issue if this work came through `compass-spec`, in this order:

1. Check however this project's commit messages reference tracked work (an issue number, a ticket key, a link). Follow that reference to fetch the item.
2. A path given directly as an argument.
3. A spec file under `docs/`, `specs/`, or `tasks/` matching the branch name or feature.
4. If nothing is found, ask where the spec is. If there is not one, the **Spec** sub-agent skips and reports "no spec available." The **Security** sub-agent still runs, since the baseline in step 4 applies with or without a spec; note in its prompt that no spec-level Security Decisions exist to check against.

### 3. Identify the standards sources

Anything in the project that documents how code should be written: a coding-standards document, a contributing guide, house-style notes.

On top of whatever the project documents, the Standards axis always carries the **smell baseline** below: a fixed set of Fowler code smells (*Refactoring*, ch. 3) that applies even when a project documents nothing. Two rules bind it:

- **The project overrides.** A documented project standard always wins; where it endorses something the baseline would flag, suppress the smell.
- **Always a judgement call.** Each smell is a labelled heuristic ("possible Feature Envy"), never a hard violation. Like any standard here, skip anything tooling already enforces.
- **Blocking rule.** A documented-standard breach is **Hard** and blocks the verdict. A baseline smell is a **Judgement call** and is advisory only: it is reported, but does not by itself block.

Each smell reads *what it is* → *how to fix*; match it against the diff:

- **Mysterious Name**: a function, variable, or type whose name does not reveal what it does or holds. → rename it; if no honest name comes, the design is murky.
- **Duplicated Code**: the same logic shape appears in more than one hunk or file in the change. → extract the shared shape, call it from both.
- **Feature Envy**: a piece of logic that reaches into another module's data more than its own. → move the logic onto the data it envies.
- **Data Clumps**: the same few values keep travelling together (a type wanting to be born). → bundle them into one type, pass that.
- **Primitive Obsession**: a primitive or string standing in for a domain concept that deserves its own type. → give the concept its own small type.
- **Repeated Switches**: the same branch/cascade on the same distinction recurs across the change. → replace with a single shared dispatch both sites use.
- **Shotgun Surgery**: one logical change forces scattered edits across many files in the diff. → gather what changes together into one module.
- **Divergent Change**: one file or module is edited for several unrelated reasons. → split so each module changes for one reason.
- **Speculative Generality**: abstraction, parameters, or hooks added for needs the spec does not have. → delete it; inline back until a real need shows.
- **Message Chains**: long chained navigation through several objects that the caller should not depend on. → hide the walk behind one call on the first object.
- **Middle Man**: a module that mostly just delegates onward. → cut it, call the real target directly.
- **Refused Bequest**: something that inherits or implements an interface but ignores or overrides most of what it inherits. → drop the inheritance, use composition.

### 4. Identify the security sources

Gather, in this order:

1. Anything the project documents on security: a `SECURITY.md`, a threat-model doc, a security section of the house-style guide.
2. The spec's own **Security Decisions** section, if one exists (fetched in step 2): the trust boundaries, sensitive data, and authorisation changes it named. The diff is checked against these directly, the same way the Spec axis checks the diff against User Stories.

On top of both, the Security axis always carries the **weakness baseline** below: a fixed set of categories anchored on the OWASP Top 10 (2021) that applies even when a project documents nothing and no spec is available. Three rules bind it, distinct from the Standards axis's rules because a security weakness is not the same kind of thing as a code smell:

- **The project overrides, but only when the acceptance is documented.** A silent absence of a control is not an override; a project's security doc explicitly accepting a named risk is.
- **Severity, not hard-versus-judgement.** Rate each finding Critical, High, Medium, or Low. Unlike a smell, a matched pattern here is usually a real weakness, not a style preference, so this axis does not carry the Standards axis's "always a judgement call" framing.
- **Skip anything a security tool already enforces** in this project's CI (a SAST scanner, a secret scanner, a dependency-audit gate), the same tooling-deference rule the Standards axis uses.
- **Blocking rule.** **Critical** and **High** findings block the verdict. **Medium** and **Low** findings are advisory only: reported, but do not by themselves block.

Each category reads *what it is* → *what to check*; match it against the diff:

- **Injection**: untrusted input reaches a query, command, template, or interpreter without parameterisation or escaping. → check for a query, shell call, or template built by concatenating request data instead of using a parameterised or escaped form.
- **Broken access control**: an operation that should check who is calling it, or that they own the resource, and does not, or checks the wrong thing (authenticated but not authorised). → verify the check exists and checks ownership or role, not just that a caller is logged in.
- **Cryptographic failure**: a secret or sensitive value stored or sent unencrypted, or protected with a broken or outdated algorithm (MD5, SHA-1, DES, ECB mode), or compared in non-constant time. → flag the algorithm, the missing encryption, or the comparison.
- **Hardcoded secret**: a credential, key, or token committed as a literal in code or config rather than sourced from a secret store or environment. → flag the literal, point at wherever the project keeps secrets.
- **Unsafe parsing of untrusted input**: data crossing a trust boundary handled by a parser that can execute code or exhaust resources (arbitrary deserialisation, XML entity expansion, unbounded recursion, an unsafe eval). → flag the parser call, check for a safe-mode or schema-constrained equivalent.
- **Missing input validation at a trust boundary**: data from an untrusted caller used with no shape, range, or type check immediately after it crosses in. → flag the boundary named in the spec's Security Decisions (or the obvious one if there is no spec), check for validation right after it.
- **Sensitive data exposure**: a secret, credential, or personal data value logged, returned in an error message, or included in a response that does not need it. → flag the log call or response, check what it actually needs to contain.
- **Server-side request forgery**: the server fetches a URL built from untrusted input with no allow-list or destination check. → flag the fetch, check for a restriction on where it can point.
- **Insecure direct object reference**: an identifier supplied by the caller used to fetch a resource with no check that this caller may see that specific resource. → flag the lookup, check for an authorisation check alongside it, not just that the identifier parses.
- **Security misconfiguration**: a permissive or debug default (wildcard CORS with credentials, verbose error pages, disabled certificate verification) left reachable outside a development environment. → flag the setting, check whether it is gated by environment.
- **Vulnerable or outdated dependency introduced by this diff**: a new dependency or version bump added with no check for known vulnerabilities in it. → flag the change; note it needs a dependency-audit pass if the project has one, otherwise say so explicitly rather than silently skipping it.
- **Missing audit trail for a sensitive operation**: an action that changes access, deletes data, or moves money or credentials with nothing recording who did it and when. → flag the operation, check for a log or event naming the actor.

### 5. Spawn all three sub-agents in parallel

**Standards sub-agent prompt** should include:

- The full diff command and commit list.
- The list of standards-source files found in step 3, **plus the smell baseline from step 3** pasted in full (the sub-agent has no other access to it).
- The brief: "Report, per file or hunk where relevant, (a) every place the diff violates a documented standard: cite the standard (file and rule); and (b) any baseline smell spotted: name it and quote the hunk. Label every finding **Hard** (a documented-standard breach) or **Judgement call** (a baseline smell); a documented project standard overrides the baseline. Skip anything tooling enforces. Under 400 words."

**Spec sub-agent prompt** should include:

- The diff command and commit list.
- The path or fetched contents of the spec.
- The brief: "Report: (a) requirements the spec asked for that are missing or partial; (b) behaviour in the diff that was not asked for (scope creep); (c) requirements that look implemented but where the implementation looks wrong. Quote the spec line for each finding. Label every finding **Missing**, **Wrong**, or **Scope creep**. Under 400 words."

If the spec is missing, skip the Spec sub-agent and note this in the final report.

**Security sub-agent prompt** should include:

- The full diff command and commit list.
- The security-source files found in step 4, the spec's Security Decisions section if one exists, **plus the weakness baseline from step 4** pasted in full (the sub-agent has no other access to it).
- The brief: "Report, per file or hunk where relevant, (a) every place the diff violates a documented project security requirement or contradicts the spec's Security Decisions section: cite the source; (b) any baseline weakness spotted: name the category, quote the hunk. Label every finding **Critical**, **High**, **Medium**, or **Low**. A documented, considered risk acceptance in the project's own security docs overrides the baseline; note where that applies. Skip anything a security tool already enforces in this project's CI. Under 400 words."

If neither a spec's Security Decisions section nor a project security doc exists, the sub-agent still runs against the baseline alone; say so in the final report rather than treating it as a skip.

### 6. Aggregate and verdict

Present the three reports under `## Standards`, `## Spec`, and `## Security` headings, verbatim or lightly cleaned. Do **not** merge or rerank findings. The three axes are deliberately separate (see *Why three axes*). Within `## Security`, order findings Critical first, then High, Medium, Low, so the most consequential finding is never buried under lower-severity noise.

**Blocking findings**, pulled straight from the labels each sub-agent applied, no re-judging required:

- Standards: **Hard**.
- Spec: **Missing** or **Wrong**.
- Security: **Critical** or **High**.

Everything else (Judgement call, Scope creep, Medium, Low) is advisory: reported, tracked, never blocking on its own.

Compute the **Verdict**:

- **Clear**: no report contains a blocking finding, or every blocking finding present has a recorded, owner-accepted exception (see [Accepting a blocking finding](#accepting-a-blocking-finding)). List any accepted exceptions under an `### Exceptions` sub-heading: the finding, who accepted it, and why.
- **Blocked**: at least one blocking finding remains with no accepted exception. List each one, one line per finding, under a `### Blocking` sub-heading: axis, label, and a one-line quote of the finding, so the verdict is actionable without re-reading the full sections above.

End with a one-line summary: total findings per axis, and the worst issue *within each axis* (for Security, its highest-severity finding; for Standards and Spec, the most consequential one). Do not pick a single winner across axes. That is the reranking the separation exists to prevent.

## Accepting a blocking finding

A blocking finding is not something this skill, or whoever is implementing the work, gets to wave off alone. It is a decision, and like any decision in this pipeline it belongs to whoever owns the work, not to the agent running the review or the fix.

When a blocking finding cannot or should not be fixed before shipping (the fix would require re-scoping what the spec asked for, the finding is disputed as a false positive, or the risk is judged acceptable for this change), stop and put it to whoever owns the task rather than silently proceeding or silently blocking forever:

1. State the finding, its axis and label, and why a fix is not happening now.
2. Ask for an explicit accept-or-fix decision. Do not infer consent from silence.
3. On acceptance, record it: as the exception entry in this report (see above), and, if it meets `compass-glossary`'s three-part test (hard to reverse, surprising without context, the result of a real trade-off), also as a recorded decision there. A verbal "that's fine" that never reaches either place is indistinguishable, to the next reader, from a finding nobody looked at.
4. On refusal, the finding stays blocking. If the disagreement is about whether the finding is even correct, that is itself a decision worth a trip through `compass-navigate` rather than a re-argued comment thread.

## Why three axes

A change can pass some axes and fail others, because each one checks something structurally invisible to the other two:

- Code that follows every standard but implements the wrong thing → **Standards pass, Spec fail.**
- Code that does exactly what the spec asked but breaks the project's conventions → **Spec pass, Standards fail.**
- Code that follows every standard and matches the spec exactly, but builds a query by string-concatenating a caller-supplied value → **Standards pass, Spec pass, Security fail.** Neither style conformance nor requirement conformance says anything about exploitability; only the Security axis is looking for it.

Reporting them separately stops any one axis from masking the other two.
