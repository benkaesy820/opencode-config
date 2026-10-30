---
name: Code Review
description: Use when reviewing changes, a diff, a branch, or a pull request, before committing or opening a PR, and when asked to check, verify, sanity-check, or critique code someone wrote. Triggers on "review my changes", "review this diff", "is this ok", "check this PR", "what did I break", "any problems with this", or a request to look over work before it ships. Also use before committing a change that alters existing behavior or a public surface, and when removing dead code, unused dependencies, or reducing bloat — reporting findings with evidence before deleting anything.
---

# Code Review

Review for defects that matter. Not a style pass.

## Scope first

Establish what you are reviewing before reading anything:

```sh
git status                 # working tree state
git diff                   # unstaged
git diff --staged          # staged
git diff <base>...HEAD     # branch vs base
```

If the user asked for a specific set of files, review only those. If the working tree has unrelated changes, say so and exclude them rather than reviewing everything.

## What to look for

Work outward from highest consequence.

**Correctness**
- Does the code do what the change claims, including at the edges: empty, zero, one, null, missing, very large, negative?
- Are off-by-one and boundary conditions right in both directions?
- Are error paths handled, or swallowed?
- Is state left consistent on every exit, including early return and throw?

**Regressions**
- What existing caller now behaves differently?
- Which existing test covers the old behavior, and was it updated or deleted?
- Does a removed guard or check have a replacement?
- Is a synchronous path now async, or the reverse?

**Contracts**
- Changed function signature, return type, or error type with existing consumers.
- Schema, migration, or serialized shape changes.
- Anything persisted or sent over the wire.
- Public API changes without versioning or deprecation.

**Resource handling**
- Files, sockets, locks, transactions, and subscriptions opened but not closed.
- Loops that can grow unbounded or recurse without a base case.
- Blocking work inside a request or event handler.

**Concurrency**
- Shared mutable state without a stated guarantee.
- Missing idempotency where retries are possible.
- Check-then-act races.

**Recency**
- Is this pattern, API, or dependency still the current one? Deprecated, superseded, or unmaintained code is a liability with a running maintenance clock.
- Does the code work around a bug that upstream has since fixed? The workaround is now weight, and deleting it is often smaller than keeping it.
- Look it up, never recall it. Repo-local: `git log -S<symbol>` for how long a workaround has stood, the project's own outdated/audit command (`pnpm outdated`, `npm audit`, `cargo outdated` — discover it, do not invent it), and the versions the lockfile actually resolved. External: fetch the release notes, changelog, or upstream issue. A finding with no lookup behind it goes under `Unverified`.

**Stability**
- Blast radius: how many callers, modules, and users does this reach? Say the number.
- Partial failure: if this breaks halfway, what state is left behind, and is it observable?
- Rollback: if this must be undone at 3am, what exactly is undone, and is it safe?
- Faster but flakier is a net loss. Name the trade when a latency win is bought with a race, a retry, or a cached assumption.
- Does this make the platform less reliable anywhere? That question deserves an answer even when the change is good.

**Fit**
- Does this match how the rest of the codebase solves the same problem?
- New abstraction, dependency, or flag that closes no named requirement.
- An abstraction that only forwards to one implementation without naming a concept.
- Duplicated logic that already exists elsewhere.
- New file per trivial function, or unrelated concerns fused into one file to hit a count.
- Robustness and efficiency traded off without a reason tied to path or measurement.
- Comments that narrate a change, instruct a future agent without an owner, or are now wrong are defects, not style. `documentation` owns that judgment and the historical-exception rule; judge against it here.

**Dead code**
- A dependency declared in the manifest but absent from the source tree.
- A flag, config key, or environment variable with only one value ever used.
- A dependency superseded by something else in the tree.

Dead code is a claim that must be demonstrated, not inferred from a failed search. Dynamic resolution defeats grep: `getattr`, `import()`, string-keyed registries, plugin discovery, and framework conventions all reference code without a literal symbol match. If you cannot rule those out, the item goes under `Unverified`, never under confirmed dead. The exhaustive search is in *Dead code and bloat removal* below.

**Tests**
- Does the test assert the behavior, or only that it ran? Would it fail if the change were reverted? Was a valid assertion weakened to get green? `testing` owns the bar.

## Dead code and bloat removal

Applies when the request is to remove dead code, reduce bloat, or clean up a codebase.

**Default to reporting.** List findings with evidence. Remove only what the user asked to remove, or what they approve after seeing the list. Deletion is destructive and is almost never wanted unprompted.

**Evidence per item.** State the symbol or path, the search performed, and the count found. "No references found" without naming the search is not evidence.

**Verify before removing:**

- Search the whole repository, not the file or directory you happen to be reading.
- Check tests. A test referencing a function is a live reference even if production code does not use it.
- Check re-export barrels and `__init__.py`. A symbol can be public API with zero internal callers.
- Check manifests and config: `package.json` scripts and `bin`, `entry_points`, `exports`, `main`, `sideEffects`.
- Check framework and platform registration: routes, migrations, serializers, providers, service workers, decorators, DI containers.
- Check strings, including template and config files. Configuration often references code by string.
- Check the git history if the item looks load-bearing and surprising. `git log -S<symbol>` shows who added it and why.

**Never remove:**

- Anything you cannot demonstrate is unreferenced.
- Public API surface of a library or package, even with no internal callers.
- Test fixtures, seed data, migrations, or anything a human reads.
- Code whose only caller is a human: a CLI entry point, a scheduled job, a script run by hand.
- Anything the user did not ask you to remove and has not approved.

**Large cleanups get reviewed, not merged.** Above roughly ten files or a few hundred changed lines, the diff will be skimmed rather than read. Break it into reviewable commits, or ask whether to proceed at that size. A cleanup diff is also the easiest place to hide an unrelated behavior change, so check that each hunk is purely a removal.

**Bloat is not dead code.** A long function, deep nesting, or three layers of wrapping may be worth simplifying, but that is a judgment, not a demonstration. Report it as a `Consider` finding with the specific improvement and what it costs to be wrong. Do not remove it unprompted.

## Review scope and rounds

- Review the change's intended case first. Another path, a rare input, or a cheap extra guard is a `Consider` or a follow-up, not a `Blocker`, whatever label it came with.
- Fix the case that happened, not every case that could. A review loop grows a change one cheap-looking fix at a time; keep the diff at the size the goal needs.
- Default to three rounds. Extend to five only for security, permissions, auth, data loss, or concurrency while a `Blocker` or `Should fix` stays open with evidence.
- Across rounds keep a ledger: id, severity, outcome (`fixed` / `disagree` / `declined`), and the evidence. Later rounds check the changed code and the open ids only.
- Budget end is not approval. If an item is still disputed after two replies, stop and report both sides; the user decides.
- Do not push during the loop. Push after agreement, or when the user asked.

## Severity order

Report in this order, and be explicit when a level is empty.

| Level | Meaning |
| --- | --- |
| **Blocker** | Wrong results, data loss, security hole, or breaks existing callers. |
| **Should fix** | Real defect in a less common path, missing error handling, weak test coverage. |
| **Consider** | Fit, clarity, or maintainability. Optional. |
| **Nit** | Cosmetic. Mention at most a few, never as the headline. |

A review with no blockers should say so plainly. Do not inflate a nit into a blocker to seem thorough.

## Output shape

```text
<verdict>: <one line — approved / approved with nits / changes requested>

Blockers
- <path>:<line> — <the defect in one sentence>
  Why: <the input or sequence that triggers it>
  Fix: <the smallest correction>

Should fix
- <path>:<line> — <defect> | Fix: <correction>

Consider
- <path>:<line> — <observation> | Suggestion: <what and why>

Nits
- <path>:<line> — <cosmetic>

Verified
- <what you actually confirmed by reading or running>
Unverified
- <what you could not check, and why>
```

## Rules

- Every finding needs a file and line, a trigger, and a concrete fix.
- Do not report findings you cannot point at. Speculation belongs under `Unverified`.
- Read the surrounding code before judging a line. A correct fix in a wrong place is a blocker.
- Prefer fewer, real findings over a long list of theoretical ones.
- Do not rewrite the change yourself unless asked. Review only.
- Distinguish "this is wrong" from "I would have done it differently."
- If the change is correct but larger than it needed to be, that is a `Consider`, not a blocker.