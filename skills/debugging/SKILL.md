---
name: Debugging
description: Use when something is broken, failing, crashing, flaky, or behaving unexpectedly, and before guessing at a fix. Triggers on "this is broken", "it doesn't work", "why is X failing", a stack trace, a failing test, a bug report, a regression, intermittent or flaky behavior, wrong output, or an exception. Also use when a test fails and the cause is not yet known, and when a fix did not work on the first attempt.
---

# Debugging

One falsifiable hypothesis at a time. No shotgun edits.

## 1. Reproduce

Do not start editing until you can make the failure happen on demand.

Get:

- The exact error, full text, not a summary.
- The full stack trace.
- The exact input, state, and environment.
- The command or action that triggers it.
- Whether it is deterministic or intermittent.

```sh
<test or command that reproduces>
```

If you cannot reproduce it, say so and gather evidence instead of editing. "I cannot reproduce this, and here is what I would need" is a legitimate outcome. Guessing at an unreproducible fault produces changes that cannot be validated.

## 2. Narrow

Bisect deliberately. Each step must eliminate a half.

1. **Confirm the failure is where you think.** Read the trace. Do not assume the reported line is the cause; it is frequently a symptom.
2. **Bisect the change.** If a commit is suspected, test at the midpoint commit. `git bisect` is the tool.
3. **Bisect the input.** Reduce the failing case to the smallest input that still fails. Remove one variable at a time.
4. **Bisect the layer.** UI, API, service, database, or external provider. Which layer first produces the wrong value?
5. **Check recent changes.** What moved? Migrations, dependency bumps, config, timeouts, clock skew, locale.

State a hypothesis in falsifiable form:

> I believe `<cause>` because `<evidence>`. This is wrong if `<observation>`.

Then test exactly that one thing.

## 3. Confirm the cause

You are done narrowing when you can explain the whole chain:

- Why the observed value appears at the reported location.
- Why the fix changes that value.
- Why the failure did not happen before.

If you cannot explain all three, you have a correlation, not a cause.

## 4. Fix

- Smallest change that addresses the confirmed cause.
- No defensive guards for conditions you could not reproduce. They hide the real bug.
- No retries, sleeps, or swallowed exceptions to make the symptom disappear.
- Match the surrounding code's existing approach.

## 5. When the obvious fix does not exist

When the cause is confirmed but the fix is not obvious, or two fixes have failed, stop widening the search and reframe:

1. Question the premise. The reported line is often the messenger, not the cause.
2. Try the layer below. The fix usually lives one level down from the symptom — in the data, the contract, the caller — not in the code that noticed it.
3. Delete before you repair. A capability nobody uses can be removed; that is often smaller, faster, and more reliable than fixing it.
4. Prefer the boring mechanism. The standard library, the platform feature, the pattern already used elsewhere in this repository.
5. Change the requirement, not the code. If the only way to satisfy the spec is to make it fragile, the spec is the defect — say so.
6. Timebox it. Two failed fixes mean the model of the problem is wrong. Bring the user three reframings with evidence, not three patches.

If the confirmed cause lives outside the code in front of you, or the same symptom appears in more than one place, load `root-cause`: a local fix there is a workaround, and the platform needs the structural one.

## 6. Verify

Run the original reproduction first. Then:

- The tests covering the affected area.
- The project's lint and typecheck.
- The full suite if the blast radius is unclear.

For intermittent failures, a single green run proves nothing. Either run it many times, or add a test that would have caught it and confirm that test fails before the fix and passes after.

## 7. Common causes worth checking

- Unhandled null, undefined, or empty collection.
- Off-by-one at a boundary.
- Async ordering: unawaited promise, missing barrier, race.
- Stale state: cache, memo, global, connection reuse.
- Floating-point rounding in money or percentages.
- Timezone, DST, clock skew, or locale.
- Implicit string-to-number or string-to-date coercion.
- Encodings and line endings.
- Resource exhaustion: connection pool, file handles, unbounded growth.
- Environment difference: local vs CI, timezone, locale, dependency version.

## 8. Output shape

```text
Root cause: <one sentence — mechanism, not location>

Evidence:
- <observation that confirmed it>
- <observation that eliminated alternatives>

Fix: <path>:<line> — <what changed>

Verified:
- <command> → <actual result>

Not verified: <anything not actually run>
For intermittent cases: <N runs, M failures, before vs after>
```

## Rules

- Never change code to test a hypothesis unless the hypothesis is about the code's logic.
- Never leave a change in place that you could not justify with an observation.
- If the fix is a workaround for an upstream or platform bug, say so explicitly and name the issue — and load `root-cause` when the same mechanism shows up in more than one place.
- Do not delete a failing test. Determine whether the test or the code is wrong, and say which.