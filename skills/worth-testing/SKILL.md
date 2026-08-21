---
name: worth-testing
description: Use when writing or reviewing tests, deciding whether a test is worth writing, necessary, helpful, or a useful addition, judging whether a test justifies its cost, or identifying and deleting unnecessary tests. Also use when a suite feels bloated, when tests fail on refactors, when a test was added for completeness or coverage, when extracting a function to make it testable, and whenever a diff under review adds test files.
---

# Worth Testing

## Stance

Most tests an agent proposes are not worth writing. The default is no.

A test earns its place only by clearing the gate below. The gate is deliberately hard to pass. When in doubt, skip the test. If it is mostly not worth it, it is not worth it.

Write a test when the logic is non-obvious, a silent failure would be costly, you are changing code you do not understand, or the test costs less than repeating a manual check.

Skip it when the code is a thin pass-through, the type system already covers it, or the test merely repeats the implementation.

This skill scopes test-driven development. It does not repeal it. TDD does not override a decision that a test is not worth writing. When a test clears the gate, write it test-first.

## Costs

A test is code you must read, run, review, and maintain. Count these costs before writing it.

- More code to read and review.
- Slower CI and higher compute cost.
- More context tokens whenever an agent reads the suite.
- Maintenance when the test is coupled to implementation.
- Flakiness that wastes triage time and erodes trust.
- False confidence from a green test that guards the wrong code.
- Noise that teaches people to ignore failures.
- Extra work for every behavior-preserving edit.

A useful test pays these costs back. A useless test does not.

## Before the gate: subtract the compiler

Do not call a compile-time error a bug that needs a test. First remove everything the language, type checker, linter, or static checks already prevent.

Examples include:

- Changed function signatures, arity, or parameter types.
- Renamed, retyped, or removed struct fields.
- Missing cases in an exhaustive match.
- Nil, ownership, and lifetime errors in languages that track them.
- Anything a linter or static check fails in CI.

This matters most during dependency upgrades. “The library changed its API” is an argument for recompiling, not for adding a test. Ask what survives the compiler. That may include a reversed boolean, a field read from the wrong sibling, or a misread status code. Only those semantic failures belong in question 1.

If richer types would eliminate the bug, prefer them to a test. Use a sum type instead of a boolean or a newtype instead of a bare string. The type is checked everywhere. The test is checked in one place.

## The gate

Answer all four questions in writing before writing a test. Do the same for every test in a diff you review. Write the answers in the PR comment, commit message, or response to the user. Unwritten reasoning is easy to fake.

**1. What production change would make this test fail?**

Name the specific edit, not a category. “Someone breaks validation” is not enough. “Someone inverts `!result.OK`” is specific.

**2. Would a competent person plausibly make that change?**

You can invent a bug for any line of code. That is why question 1 is not enough. The bug must be one a real person might ship, such as an inverted boolean, an off-by-one error, swapped arguments of the same type, a dropped error return, or a subtle edge case in logic that required thought. If you must strain to imagine the mistake, stop.

**3. Does this test sit on the path to that failure?**

Trace the named failure from cause to consequence. Then check whether the test intercepts that path.

A test of a pure function does not guard wiring around that function. Wiring includes whether the function is called, whether its error reaches the exit code, and whether the correct dependency is passed. A test that misses the wiring guards the part that was easiest to inspect.

If the test does not sit on the path, write the test that does or write none.

**4. Is the failure silent on the common path?**

Loud failures need fewer tests. If the bug would fail on the next deploy, request, or local run with a clear message where someone is already looking, the feedback loop already exists. Tests matter most when failure is quiet, such as plausible wrong results, slowly corrupted data, or a guard that stops guarding unnoticed.

Ask about the common case, not the rare case. Would the bug announce itself on the next normal run?

Inverted guards often fail loudly. A guard exists for the exception, so the common path passes. Inverting it makes ordinary runs error immediately. The rare branch remains dangerous, but the loud failure exposes the mistake first.

A test whose named failure would break the build immediately and noisily is a test for a bug that already reports itself.

If a test misses any question, it does not earn its place. Delete it during review.

Run the gate once per bug and once per test case. Do not name several candidate bugs and let the strongest one justify all of them. Each bug must clear all four questions. A test case survives only if at least one bug clears the gate through that case. Judge the file as independent claims, because that is what it contains.

## Assertions

A worthwhile test can still assert the wrong things. Remove every assertion that would fail under a behavior-preserving edit.

- **Exact error strings.** `ErrorContains(err, "validation failed: [migration 7]")` couples the test to formatting and default-value rendering. Assert the condition, error type, or offending item instead.
- **Stub arguments** that do not affect behavior.
- **Call counts** that are not part of the contract.
- **Private state**, internal structure, and field values the caller cannot observe.

Ask this diagnostic question. Would the assertion fail if you rewrote the internals but kept the behavior? If yes, it detects an implementation change. Remove it.

## Unnecessary tests

Delete or refuse to write these tests.

- **Change detector.** Fails on any refactor.
- **Mirror.** Derives the expected value from the code under test.
- **Mock assertion.** Checks that a double was called, not that the outcome occurred.
- **Duplication.** A cheaper test at another layer already covers the behavior.
- **Trivial target.** Tests a getter, setter, constant, or pure forwarder.
- **Glue test.** Exercises straight-line orchestration while the real risk is the surrounding wiring.
- **Coverage test.** Exists only to raise a number.
- **Framework, language, or third-party test.** Verifies behavior someone else owns.

## Extraction is not justification

“I extracted this function so it could be tested” proves only that testing became possible. It does not prove that testing became worthwhile.

Run the gate on the extracted function as if it had always existed. If extraction moved the logic out of the risky path and left the risk in the caller, it made the suite worse.

## Excuses to reject

- **“It raises coverage.”** Coverage measures whether code ran.
- **“It documents the behavior.”** A test that cannot fail documents nothing. Write a comment.
- **“Tests are cheap.”** A suite of cheap, useless tests is expensive.
- **“Simple code still needs a test.”** Only when it has a real, silent failure mode.
- **“The blast radius is huge.”** A test that misses the risk does not shrink it.
- **“It is cheap enough not to fuss over.”** Three lines of noise still fail on every refactor.
- **“A dependency just changed this code.”** Recompile, then test only what survived the compiler.
- **“I already wrote it.”** Sunk cost does not justify maintenance cost.
- **“It is TDD, so I have to.”** TDD rewards tests that catch bugs.

## Final rule

Name the bug. Show that a competent person could make it. Show that the test intercepts it. Show that it would otherwise fail quietly. Four answers, in writing, or no test.

## Sources

- Kent Beck: pay for code that works, not tests.
- Sandi Metz: test the interface, not the implementation.
- Ian Cooper: test requirements, not methods.
- Martin Fowler, *Mocks Aren't Stubs*: mockist tests couple to implementation and break on refactor. Fowler presents both schools fairly and calls himself a classicist by habit.
- Kent C. Dodds: the Testing Trophy, confidence per cost.
- JB Rainsberger: integrated tests as duplication.
- Vladimir Khorikov, *Unit Testing*: maximize sensitivity and minimize brittleness.
