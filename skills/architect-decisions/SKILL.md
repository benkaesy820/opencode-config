---
name: Architect Decisions
description: Use when a load-bearing choice is missing or contested and reversing it would be expensive — ownership or permission rules, tenant or data visibility, schema or public API direction, an irreversible migration, versioning or deprecation, money, or legal exposure. Triggers on "should this be", "which approach", "what's the right way", "how should I model", "where should this live", "is X or Y better", a design with several defensible answers, or a missing requirement blocking implementation. Produces a decision record, not code, and loads before implementation.
---

# Architect Decisions

For decisions that are expensive to reverse and load-bearing for correctness. Not for ordinary design, which should follow existing patterns in the repository.

## When this applies

Escalate here when any of these is true:

- Ownership, authorization, tenancy, or data visibility rules are unclear or disputed.
- A public API, event schema, or database constraint must change in a way callers notice.
- Data must be migrated, backfilled, or deleted, and the rollback is unclear.
- The choice affects money, legal exposure, or compliance.
- Two designs are both defensible and the choice affects more than one module.
- The user asked for "the right way" rather than "a way".

Do not use this skill for naming, formatting, library choice within an existing pattern, or anything a code review would catch.

## Method

### 1. Frame the decision

State it as a single question with a test for a correct answer. If you cannot, the request is not clear enough to decide yet.

Write down:

- The requirement being satisfied, in one sentence.
- The constraint that makes this hard.
- What is genuinely fixed (existing callers, data on disk, law, provider behavior).
- What is actually free.

### 2. Survey the real constraints

Read before proposing. Cite the evidence.

- Existing implementations and their callers.
- Schemas, migrations, and indexes already in place.
- Tests that would break.
- Public docs or changelogs that describe current behavior.
- Provider and platform behavior you are depending on.

Mark anything you could not verify as an assumption, not a finding.

### 3. Generate real options

Three is usually enough. For each: how it works, what it costs, and how it fails.

Include the do-nothing option. It is often correct and it is always worth naming.

Prefer options that are additive and reversible. Prefer the one that is cheapest to reverse, not the one that is fastest to build.

### 4. Decide, and say why

Pick one. Give the reason in terms of the requirement and the constraints, not in terms of personal taste or trend.

Then state:

- What would invalidate this decision.
- What it costs to reverse later.
- The smallest step that commits to it, so it stays reversible as long as possible.

### 5. Hand back

Return the decision in this shape:

```text
DECISION: <one sentence>

Requirement: <what this satisfies>
Key constraint: <what makes it hard>

Options:
1. <name> — <one line> | cost: <n> | reverses: <easy|hard>
2. <name> — <one line> | cost: <n> | reverses: <easy|hard>
3. Do nothing — <one line> | reverses: n/a

Chosen: <option> because <reason tied to requirement/constraint>.

Assumptions: <each one, or "none">
Invalidated if: <the observable condition>
Reversal cost: <what reversing later costs>
First step: <smallest reversible commitment>
```

## Rules

- Recommend, do not survey. An options list with no recommendation pushes the decision back to the caller.
- Never invent product or privileged behavior to make a decision resolve: prices, fees, limits, roles, permissions, ownership, delivery zones, compliance rules, provider guarantees. If one is missing, say it is missing and it is the user's call.
- Prefer the boring, reversible option over the elegant, irreversible one.
- Do not design for scale that has not been requested.
- State the cost of the decision, including the option you rejected and why.
- If the answer is that the question is not yet decidable, say that and name the missing input. That is a valid outcome.