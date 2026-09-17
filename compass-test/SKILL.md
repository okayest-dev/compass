---
name: compass-test
description: Test-driven development, language-agnostic. Use when building a feature or fixing a bug test-first, following a red-green-refactor loop, or writing integration-style tests.
---

# Compass Test

TDD is the red → green loop. This skill is the reference that makes that loop produce tests worth keeping: what a good test is, where tests go, the anti-patterns, and the rules of the loop. Every section applies on every cycle. Consult them before and during the loop, not after.

When exploring the codebase, read `CONTEXT.md` (if it exists) so test names and interface vocabulary match the project's domain language, and respect any recorded decisions in the area being touched. Use `compass-glossary` if the glossary needs updating along the way.

## Consumes / produces / hands off

- **Consumes:** a task slice from `compass-task` (or a bug description), and the seam under test.
- **Produces:** passing tests plus the minimal implementation that makes them pass.
- **Called from:** `compass-implement`, once per vertical slice.
- **Hands off to:** nothing directly. Refactoring and structural cleanup belong to `compass-review`, not this loop.

## What a good test is

Tests verify behaviour through public interfaces, not implementation details. The implementation can change entirely; the tests should not. A good test reads like a specification: "user can complete checkout with a valid cart" states exactly what capability exists, and it survives refactors because it does not care about internal structure.

See [TESTS.md](./TESTS.md) for worked examples and [MOCKING.md](./MOCKING.md) for mocking guidelines.

## Seams: where tests go

A **seam** is the public boundary a test exercises: the interface where behaviour is observed without reaching inside. Tests live at seams, never against internals.

**Test only at pre-agreed seams.** Before writing any test, write down the seams under test and confirm them with whoever owns the work. No test is written at an unconfirmed seam. Testing effort cannot cover everything, so agreeing the seams up front is how effort lands on the critical paths and complex logic instead of every edge case.

Ask: "What is the public interface here, and which seams should be tested?"

When the shape of that interface is itself in question, meaning how deep the module is, where the seam belongs, what the interface should expose, use `compass-design` for the vocabulary. It is the shared source of the module, interface, depth, seam, adapter, leverage, and locality terms, and it is a reference to consult, not a session to run.

## Anti-patterns

- **Implementation-coupled**: mocks internal collaborators, tests private methods, or verifies through a side channel (querying storage directly instead of using the interface). The tell: the test breaks on refactor even though behaviour has not changed.
- **Tautological**: the assertion recomputes the expected value the same way the code does, so it passes by construction and can never disagree with the code. Expected values must come from an independent source of truth: a known-good literal, a worked example, the spec.
- **Horizontal slicing**: writing all tests first, then all implementation. Bulk tests verify *imagined* behaviour. They test the *shape* of things rather than observable behaviour, they go insensitive to real changes, and the test structure gets locked in before the implementation is understood. Work in **vertical slices** instead: one test → one implementation → repeat, each test a **tracer bullet** that responds to what the last cycle taught you.

## Rules of the loop

- **Red before green.** Write the failing test first, then only enough logic to pass it. Do not anticipate future tests or add speculative behaviour.
- **One slice at a time.** One seam, one test, one minimal implementation per cycle.
- **Refactoring is not part of the loop.** It belongs to the review stage (`compass-review`), not the red → green cycle.
