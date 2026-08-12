---
name: worth-testing
description: Use when writing or reviewing tests, deciding whether a test is worth writing, necessary, helpful, or a useful addition, judging whether a test justifies its cost, or identifying and deleting unnecessary tests. Also use when a suite feels bloated, when tests fail on refactors, when a test was added for completeness or coverage, when extracting a function to make it testable, and whenever a diff under review adds test files.
---

# Worth Testing

## Stance

Most tests an agent proposes are not worth writing. The default is no.
A test earns its place only by clearing the gate below. The gate is
designed to be hard to pass.

When in doubt, skip it. If it is mostly not worth it, it is not worth it.

Write the test when the logic is non-obvious, silent failure is costly,
you are about to change code you do not understand, or the test is
cheaper than the manual check you would otherwise repeat.

Skip it when the code is a thin pass-through, the type system already
covers it, or the test just restates the implementation line by line.

This skill scopes test-driven development. It does not repeal it. When
this skill says a test is not worth writing, TDD does not override it.
When this skill says a test is worth writing, write it test-first.

## Costs

A test is code you must read, run, review, and maintain. Count these
against every test before you write it.

- More code to read and review.
- Slower CI. Higher compute cost.
- More context tokens every time an agent reads the suite.
- Maintenance. A test coupled to the code fails on every refactor.
- Flakiness. Intermittent failures waste triage time and erode trust.
- False confidence. A green test that guards the wrong five lines is
  worse than no test, because it ends the conversation about the risk.
- Noise. Failures that are not bugs teach people to ignore the red bar.
- Drag. Every behavior-preserving edit costs more to land.

A useful test pays these costs back. A useless test never does.

## Before the gate: subtract the compiler

You cannot ship a bug that does not compile. Before naming a bug, strike
everything the language already prevents.

- Changed function signatures, arity, or parameter types.
- Renamed, retyped, or removed struct fields.
- Missing cases in an exhaustive match.
- Nil, ownership, and lifetime errors in languages that track them.
- Anything a linter or static check fails in CI.

Dependency upgrades are where this matters most, because they feel risky
and the risk is mostly compile-time. "The library changed its API" is an
argument for recompiling, not for testing. What survives the compiler is
semantics: a boolean read backwards, a field consulted instead of its
sibling, a status code read wrong. Only those may be named in question 1.

If richer types would eliminate the bug outright, prefer that to a test.
A sum type instead of a boolean. A newtype instead of a bare string. The
type is checked everywhere. The test is checked in one place.

## The gate

Answer all four questions in writing before you write a test, and for
every test in a diff you review. Write the answers out. In the PR
comment, the commit message, or to the user. Reasoning you do not write
down is reasoning you will fake.

**1. What production change would make this test fail?**

Name the specific edit, not a category. "Someone breaks the validation"
is not an answer. "Someone inverts `!result.OK`" is.

**2. Would a competent person plausibly make that change?**

You can invent a bug for any line of code ever written. That is why
question 1 alone is not a gate. The bug must be one a real person would
ship: an inverted boolean, an off-by-one, a swapped argument of the same
type, a dropped error return, a subtle edge case in logic you had to
think about. If you had to strain to imagine the mistake, stop.

**3. Does this test actually sit on the path to that failure?**

This is the question most often skipped, and skipping it is how useless
tests get written with a straight face. Trace the failure you named in
question 1 from cause to consequence. Then check whether the test
intercepts that path.

A test that exercises a pure function while the named risk lives in the
wiring does not guard the named risk. The wiring is whether the function
is called at all, whether its error reaches the exit code, whether the
right dependency is passed. Such a test guards the segment that needed
it least, because a bug in five lines of straight-line logic is visible
on sight, while a bug in the wiring is not.

If the test does not sit on that path, write the test that does, or write
none.

**4. Is the failure silent on the common path?**

Loud failures need fewer tests. If the bug would blow up on the next
deploy, the next request, or the next local run, with a clear message, in
a place someone is already looking, the feedback loop already exists.
Tests earn the most where failure is quiet: wrong results that look
plausible, data that corrupts slowly, guards that stop guarding without
anyone noticing.

Ask specifically: on the next trip through the common case, does this bug
announce itself? Not the rare case. The one that runs every time.

Inverted guards usually fail this. A guard exists for the exception, so
its common path is pass. Invert it and every ordinary run starts erroring
immediately and visibly. The dangerous branch is the rare one, but it
never gets reached, because the loud branch fired first and someone
already fixed it. Naming only the rare branch and calling the bug silent
is the standard way this question gets answered wrong.

A test whose named failure would break the build noisily and immediately
is a test for a bug that reports itself.

Miss any one of the four and the test does not earn its place. If you are
reviewing, delete it.

**Run the gate once per bug, and once per test case.** Do not name three
candidate bugs and let the strongest-sounding one carry a verdict for all
of them. Each named bug must clear all four questions on its own. A test
case survives only if at least one bug clears the gate through that case.
Bundling is how weak tests ride along with strong ones. Judge the file as
a set of independent claims, because that is what it is.

## Assertions

Even a worthwhile test can assert the wrong things. Cut every assertion
that would fail under a behavior-preserving edit.

- **Exact error strings.** `ErrorContains(err, "validation failed: [migration 7]")`
  pins message formatting and default value rendering. Assert the
  condition, not the sentence. Assert that an error occurred, that it
  is the right error type, that it mentions the offending item.
- **Arguments passed to a stub** that nobody's behavior depends on.
- **Call counts** not in the contract.
- **Private state**, internal structure, field values the caller never
  sees.

The diagnostic: would this assertion fail if you rewrote the internals
but kept the behavior? If yes, it is a change detector. Cut it.

## Unnecessary tests

Delete or refuse to write these.

- **Change detector.** Fails on any refactor.
- **Mirror.** The expected value is derived from the code under test.
- **Mock assertion.** Asserts a double was called, not that the outcome
  happened.
- **Duplication.** A cheaper test at another layer already covers this.
- **Trivial target.** Getter, setter, constant, pure forwarder.
- **Glue test.** Exercises a few lines of straight-line orchestration
  where the real risk is the wiring around them.
- **Coverage test.** Exists to move a number.
- **Testing the framework, the language, or someone else's code.**

## Extraction is not justification

"I extracted this function so it could be tested" proves nothing. The
extraction made a test possible. It did not make it worth writing. Run
the gate on the extracted function as if it had always been there. If
the extraction moved the logic out of the risky path and left the risk
behind in the caller, the extraction made the suite worse, not better.

## Excuses to reject

- "It raises coverage." Coverage measures whether code ran.
- "It documents the behavior." A test that cannot fail documents nothing.
  Write a comment.
- "Tests are cheap." A suite of cheap, useless tests is expensive.
- "Simple code still needs a test." Simple code with a real, silent
  failure mode does. Simple code without one does not.
- "The blast radius is huge." Blast radius only matters on the path. A
  test that misses the risk does not shrink it, and makes it likelier
  nobody looks again.
- "It is cheap enough not to fuss over." That is the sunk-cost excuse in
  a smaller hat. A test that fails the gate is deleted at any size. Three
  lines of noise still fire on every refactor.
- "A dependency just changed this code." Recompile. Then ask what survived
  the compiler.
- "I already wrote it." Sunk cost. Keeping it costs every refactor.
- "It is TDD, I have to." TDD rewards tests that catch bugs.

## Final rule

Name the bug. Show a competent person would write it. Show this test
intercepts it. Show it would otherwise fail quietly. Four answers, in
writing, or no test.

## Sources

Kent Beck: paid for code that works, not for tests. Sandi Metz: test the
interface, not the implementation. Ian Cooper: test requirements, not
methods. Martin Fowler, *Mocks Aren't Stubs*: mockist tests couple to
implementation and break on refactor. Fowler presents both schools fairly
and calls himself a classicist by habit. Kent C. Dodds: the Testing
Trophy, confidence per cost. JB Rainsberger: integrated tests as
duplication. Vladimir Khorikov, *Unit Testing*: maximize sensitivity,
minimize brittleness.
