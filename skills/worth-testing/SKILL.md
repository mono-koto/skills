---
name: worth-testing
description: Use when writing or reviewing tests, deciding if a test is worth writing, necessary, helpful, or a useful addition, judging whether a test justifies its cost, or identifying and deleting unnecessary tests. Also use when a suite feels bloated, tests fail on refactors, or a test was added for completeness or coverage.
---

# Worth Testing

## Stance

Write the test when the logic is non-obvious. Write it when silent
failure is costly. Write it when you are about to change code you do not
understand. Write it when the test is cheaper than the manual check you
would otherwise repeat.

Skip it when the code is a thin pass-through. Skip it when the type
system already covers it. Skip it when the test just restates the
implementation line by line.

This skill scopes the test-driven-development skill. It does not repeal
it. When this skill says a test is not worth writing, TDD does not
override it. When this skill says a test is worth writing, write it
test-first.

## Costs

A test is code you must read, run, review, and maintain. Count these
against every test before you write it.

- More code to read and review.
- Slower CI. Higher compute cost.
- More context tokens every time an agent reads the suite.
- Maintenance. A test coupled to the code fails on every refactor.
- Flakiness. Intermittent failures waste triage time and erode trust.
- False confidence. A green test that catches nothing hides the gap.
- Noise. Failures that are not bugs teach people to ignore the red bar.
- Drag. Every behavior-preserving edit costs more to land.

A useful test pays these costs back. A useless test never does.

## The gate

Before writing a test, and for every test in a diff you review, answer
one question.

What production change would make this test fail?

Name it. If you cannot name one, do not write the test. If you are
reviewing, delete it.

- A behavior-breaking change, caught nowhere cheaper. Write it, or keep it.
- Only a behavior-preserving refactor. It is a change detector. Delete it.
- A bug already caught cheaper at another layer. Delete this one. The
  other test owns it.
- A bug no one would write, with a tiny blast radius. Do not write it.

## A worthwhile test meets all of these

1. It guards a real behavior someone wants from the system.
2. It fails on a bug, not on a refactor.
3. Nothing cheaper gives the same confidence.
4. You make this kind of mistake, or the cost of being wrong is high.

Miss any one and the test does not earn its place.

## Unnecessary tests

Delete or refuse to write these.

- Change detector. Fails on any refactor. Couples to private state,
  internal structure, or call counts not in the contract.
- Mirror. The expected value comes from the code under test. Passes no
  matter what the code does.
- Mock assertion. Asserts a double was called, not that the outcome
  happened.
- Duplication. A cheaper test at another layer already covers this
  behavior.
- Trivial target. Getter, setter, constant, pure forwarder. No failure
  mode.
- Coverage test. Exists to move a number.
- Testing the framework, the language, or someone else's code.
- No failure mode in practice, and maintenance costs more than the bugs
  it would catch.

## Push back on three reflexes

Before writing. Run the gate. Can you name the bug? If not, stop.

During review. Run the gate on every test in the diff. Delete the ones
that fail it.

On "I should add a test for completeness." Stop. Completeness is not a
reason. A test that catches no bug has a cost and no return. Run the
gate.

## Excuses to reject

- "It raises coverage." Coverage measures whether code ran. It does not
  prove the code works.
- "It documents the behavior." A test that cannot fail documents
  nothing. Write a comment.
- "Tests are cheap." Cheap things still cost. A suite of cheap, useless
  tests is expensive.
- "Simple code still needs a test." Simple code with a real failure
  mode does. Simple code with no failure mode does not.
- "I already wrote it, deleting is wasteful." Sunk cost. Keeping a
  useless test costs every refactor.
- "It is TDD, I have to." TDD rewards tests that catch bugs. It does
  not reward tests that cannot.

## Final rule

If you cannot name the bug this test catches, and show nothing cheaper
already catches it, do not write it. If you are reviewing a test that
cannot answer that, delete it.

## Sources

Kent Beck. "I get paid for code that works, not for tests." Sandi Metz.
Test the interface, not the implementation. Ian Cooper. Test
requirements, not methods. Martin Fowler. Mockist tests are change
detectors. Kent C. Dodds. The Testing Trophy. Confidence per cost. JB
Rainsberger. Integrated tests as duplication.
