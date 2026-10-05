---
name: worth-testing
description: Use when writing or reviewing tests, deciding whether a test is worth writing, necessary, helpful, or a useful addition, judging whether a test justifies its cost, or identifying and deleting unnecessary tests. Also use when a suite feels bloated, when tests fail on refactors, when a test was added for completeness or coverage, when extracting a function to make it testable, and whenever a diff under review adds test files.
---

# Worth Testing

The default answer is no. Add a test only when it protects important behavior from a plausible production mistake that could otherwise pass unnoticed.

Tests cost time to read, run, review, maintain, and debug. If a compiler, static check, existing test, or cheap manual check catches the same mistake, skip the test.

## Remove what tools already catch

Before writing a test, exclude bugs already prevented by the compiler, type system, linter, or other static checks.

Do not test:

- Changed signatures, arity, or parameter types.
- Renamed, removed, or mistyped fields.
- Missing exhaustive-match cases.
- Ownership, lifetime, or nullability errors already checked.
- Anything that already fails a static check.

A dependency API change usually calls for recompiling, not adding a test. Test semantic mistakes that survive compilation, such as a reversed boolean, swapped same-typed arguments, wrong status-code interpretation, or incorrect edge-case logic.

If a richer type can prevent the mistake everywhere, prefer the type to a test.

## The four-question gate

Before writing a test, answer all four questions for that test case:

1. **What specific production change would make it fail?** Name the edit, not a category. “Invert `result.OK`” is specific; “break validation” is not.

2. **Could a contributor plausibly make that mistake?** Consider inverted conditions, off-by-one errors, swapped same-typed arguments, dropped errors, and subtle logic errors. If you have to invent an unlikely mistake, stop.

3. **Does the test intercept that failure path?** Trace the mistake from cause to observable consequence. Test the layer where the risk occurs. A pure-function test does not protect whether the caller invokes it, passes the right dependency, or propagates its error.

4. **Would this bug be obvious without the test?** Ask whether a normal run would reveal the named failure clearly. Loud failures need less testing. Focus on wrong results, quietly corrupted data, missing side effects, and work that silently does not happen.

An inverted guard can make normal input fail immediately, so a normal-path test adds little for that mistake. The same inversion can let invalid input through. Test that rare path when accepting it could stay hidden or cause harm.

If any answer is no, skip the test. Apply the gate independently to each test case. If it passes, write the test first.

## Assert behavior

Assert the contract that users or callers can observe, not the implementation.

Remove assertions that would fail after an internal rewrite that preserves behavior. Be cautious with:

- Exact error strings when the contract is the error condition or type.
- Stub arguments that do not affect the outcome.
- Call counts that are not part of the contract.
- Private state, implementation details, and incidental field values.
- Mocks that prove only that a method was called.

Ask: **Would this assertion fail if the implementation changed but the behavior stayed correct?** If so, remove it. Prefer returned results, persisted outcomes, emitted effects, error conditions, and other externally visible state.

## Usually skip these

- **Change detectors:** fail on ordinary refactoring.
- **Mirrors:** derive expected values from the implementation.
- **Duplication:** repeat behavior already covered more cheaply elsewhere.
- **Trivial targets:** getters, setters, constants, and pure forwarders.
- **Glue tests:** exercise straight-line orchestration without protecting a real risk.
- **Coverage tests:** exist only to raise a percentage.
- **Framework or dependency tests:** verify behavior owned by someone else.

Extracting a function makes testing possible, not worthwhile. Run the gate against the extracted logic and its caller. If extraction leaves the risky wiring untested, it made the suite worse.
