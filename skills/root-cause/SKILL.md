---
name: Root Cause
description: Use when a symptom points at something bigger than the code in front of you — a bug that keeps returning, the same failure in several places, a fix that turned out to be a workaround, a performance ceiling, or a request to "find the root cause", "why does this keep breaking", "this feels like a symptom", "we have patched this three times". Triggers on recurring bugs, one error appearing in multiple modules, "workaround", "this is slow and nobody knows why", or a symptom whose local fix cannot remove it. Produces an escalation that names the systemic cause, proves the platform has no existing mechanism for it, and proposes the smallest structural change that removes the whole class of defect without adding a parallel path. Not for a single local defect — that is `debugging`.
---

# Root Cause

A local fix removes one instance. A structural fix removes the class. This skill decides which one you are looking at, and when it is the structural one, says what to change and what it replaces.

## 1. Gate: class or instance?

Count before theorising.

| Instances of the same symptom | Conclusion |
| --- | --- |
| One | Local defect. Fix it (`debugging`). Mention it as a candidate only if it was expensive. |
| Two or more, same mechanism | A class. This skill applies. |
| One, but the mechanism lives outside this layer or this repository | An upstream or platform cause. This skill applies. |
| One, expensive to fix, recurring in the same shape | Worth the structural route. Say why. |

Evidence for the count: `file:line` for each instance, or the search that found them. If you cannot count them, say so — an unproven class is a hypothesis, not an escalation.

## 2. Chain: symptom → mechanism → layer → class

Reuse `debugging`'s confirmed-cause chain (its section 3) rather than re-deriving it, then extend it:

- **Symptom** — what was observed, and the trigger.
- **Mechanism** — why the symptom appears there. Not the location.
- **Layer** — the layer that owns the cause: data model, contract/schema, cache, scheduler or queue, transport, session and state, permissions, build or packaging, UI. Name one; if two fit, the lower one is the owner.
- **Class** — the set of defects this mechanism produces, written as a rule: "every X that is not Y does Z". If you cannot write it as a rule, you have a bug, not a class.

Fixing below the symptom is what makes a change structural. Fixing at the symptom's own layer produces the next instance later.

## 3. Redundancy check — before any proposal

Prove the platform does not already have a mechanism for this class before proposing one. This step is what stops a structural change from becoming bloat.

1. Search for the mechanism, not the symptom: the existing helper, cache, queue, retry layer, validator, config key, or flag that already owns this concern.
2. Check the layers above as well — the mechanism may exist but be wired for only one caller, or exist under a different name.
3. Record the search: what you searched, and what you found.

| Existing mechanism | The proposal becomes |
| --- | --- |
| Found and works | Use it. No new mechanism. |
| Found but broken or too narrow | Fix or widen it; the proposal names it as the target. |
| Not found after a real search | Propose one; name the search that proved it absent. |

A proposal that adds a parallel path next to an existing one is a finding against itself. Say so before proposing it.

## 4. Propose: the smallest structural change

The change must remove the class, not move it.

- **Retires** — name every local fix, special case, comment, and test the change makes unnecessary. A structural change that leaves the old workarounds in place is a second mechanism, which is what this skill forbids.
- **Measurable effect** — name the metric and the direction: which operation gets faster, which failure stops, which call disappears. If there is no metric, say "unmeasured" and say what would measure it.
- **Blast radius** — the modules, callers, and data the change touches. Say it even when it is one file.
- **Cost of doing nothing** — what degrades, and how fast. This is what earns the structural option its price.
- **First reversible step** — the smallest slice that commits to the direction and can still be walked back. A structural change is expensive to reverse; do not make the first step more expensive than it has to be.

If the change is expensive to reverse, load `architect-decisions` before proposing it.

## 5. Hand back

```text
ESCALATION: <one line — the class, not the symptom>

Symptom: <what was observed, and the trigger>
Mechanism: <the causal chain, one line per link>
Instances: <n places, file:line each>
Layer: <the layer that owns it>
Class: <the rule this mechanism produces>

Existing mechanism check: <found X — use or widen it | none — searched Y>
Retires: <the local fixes, special cases, and tests this replaces>
Proposed structural change: <one sentence>
Measurable effect: <metric, before → after, or "unmeasured">
Blast radius: <modules, callers, data>
Cost of doing nothing: <what degrades, how fast>
First reversible step: <the smallest slice>
Not verified: <anything unproven>
```

## Rules

- Count instances before theorising. One instance is a bug, not a class.
- Prove the absence of an existing mechanism by search, never by assumption.
- Never propose a parallel mechanism next to an existing one.
- Every structural proposal names what it retires.
- Name the metric, or say "unmeasured". Never claim a speedup without one.
- Escalate the cause, not the symptom: report the mechanism and the class, then hand back.
- Do not implement the structural change from this skill. It produces the proposal; the change itself is normal work (`workflow`).
