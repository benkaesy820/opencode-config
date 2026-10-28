---
name: Testing
description: Use when writing, repairing, or reviewing automated tests, and before adding a test suite or raising coverage. Triggers on "write a test", "add tests", "update the tests", "fix the test suite", "increase coverage", "mock this", "stub that", a test that fails intermittently or in CI but not locally, or deciding what is worth testing. Also use when a fix needs a regression test, and when a test asserts something weak or no longer matches the code.
---

# Testing

A test earns its place by failing when the behavior is wrong and passing when it is right. Anything else is cost.

## The bar for a good test

1. It asserts real behavior, not just that code runs.
2. It fails if you revert the change it covers.
3. It fails for a specific, readable reason.
4. It is deterministic.
5. It is fast enough that people run it.

Check #2 explicitly: revert the behavior (or the whole change) and watch the test go red, then restore it. If you only reasoned about it, say so. A test that passes either way is decoration.

## What to test

**Test:**

- Behavior a caller depends on, especially edge cases: empty, zero, one, null, missing, duplicate, very large, negative, unicode.
- Bug regressions. Every confirmed bug gets a test that reproduces it. This is the highest-value category.
- Error paths, not just the happy path.
- Boundary conditions around a threshold or limit.
- Contracts at the edge of your system: serialized shapes, migrations, external provider payloads.
- Invariants the code must never break.

**Do not test:**

- Private implementation details. They change without changing behavior, so the test breaks for no reason.
- Framework or library behavior.
- Trivial getters and setters.
- Every branch for coverage's sake.
- Exact log strings, timestamps, or generated IDs, unless the string is itself a contract.

## Structure

Follow the project's existing conventions. Where none exist:

- **Arrange, act, assert** — keep the three visibly separate.
- **One behavior per test.** Multiple assertions are fine when they assert one outcome. Two unrelated behaviors means two tests.
- **Name the behavior, not the function.** `rejects expired token` beats `test validate 3`.
- **Use the real thing where cheap.** Prefer calling the public function over mocking its internals. Mock at the boundary you do not control: network, clock, filesystem, third-party service.
- **Assert on values, not on the absence of a crash.** `expect(result).toEqual([1, 2])` proves something; `expect(fn).not.toThrow()` proves little.

## Fixing a failing test

Determine which is wrong before changing anything.

| The test is wrong | The code is wrong |
| --- | --- |
| Asserts an incorrect expectation | Behavior regressed |
| Depends on order or timing | Logic is defective |
| Asserts implementation detail | Missing error handling |
| Duplicates what another test covers | Wrong branch taken |

Never change an assertion just to reach green. If the expectation was right, the code is wrong. If the expectation was wrong, say so in the handoff and explain why the original expectation was incorrect.

If a test must be weakened, record the reason. A silently weakened test is a hidden regression.

## Flaky tests

A flaky test is a defect in the test, and it destroys trust in the suite.

- Remove reliance on execution order, shared mutable fixtures, and real time.
- Inject the clock and randomness rather than stubbing them inconsistently.
- Wait on a condition, not a fixed duration.
- Clean up global state and temp files in teardown.
- Never fix flakiness by adding retries or increasing timeouts. That hides the race.
- If a suite is broadly flaky, treat it as a `STANDARD` or `CRITICAL` task depending on what it covers, and say so.

## Output shape

Report:

```text
Added:
- <path> — <behavior covered>, and the bug or risk it guards

Verified:
- <command> → <result>
- Confirmed each new test fails when the behavior is reverted: <yes/no/how>

Not covered: <what you deliberately did not test, and why>
```

## Rules

- Prefer a small number of high-signal tests over broad, shallow coverage.
- Match the existing test framework, location, and naming. Do not introduce a second one.
- Do not add a test for behavior that is unverified or speculative.
- Do not change production behavior to make a test pass unless the production behavior was the actual defect, and say so explicitly.
- Test data should be readable and minimal. A 200-line fixture obscures the case.