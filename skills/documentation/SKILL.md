---
name: Documentation
description: Use when writing or correcting a README, ADR, API doc, code comment, changelog, runbook, or contributing guide, when existing documentation has drifted from what the code actually does, and when deciding whether a comment should stay or go. Triggers on "document this", "write a README", "update the docs", "add an ADR", "changelog", "write a comment explaining", "write a runbook", "this comment is stale", "should I keep this comment", "remove these comments", "clean up the TODOs", or a claim in a doc that no longer matches the implementation. Also use before publishing a command or code sample that a reader will copy.
---

# Documentation

Documentation that drifts is worse than none, because it is trusted. Every claim must be checkable against the code.

## Before writing

1. **Identify the audience.** A newcomer, a caller integrating the API, a future maintainer, or an on-call engineer. Different audience, different document.
2. **Find the code that owns the behavior.** Documentation follows implementation, never the reverse.
3. **Decide whether docs are needed at all.** If the code is self-evident and the audience is only developers in-repo, a comment may be the whole answer.

## Workflow

1. Read the implementation and the existing docs.
2. List every claim in the existing docs and check each against the code. Note which are wrong.
3. Fix the wrong ones first. A stale sentence is a defect.
4. Write the smallest document that answers the actual questions.
5. Verify every command, path, flag, and code sample you wrote.
6. Leave `WORKFLOW.md`-style process docs to the project's own `AGENTS.md`.

## Rules by type

**README**

- What it is, in one sentence, before anything else.
- Prerequisite versions, exact.
- Install, run, test commands, copy-pasteable and verified to work.
- Point to real documentation instead of restating it.
- No roadmap, no aspiration, no marketing.

**ADR**

Use when a decision was non-obvious and someone will eventually ask why.

```text
# <decision, as a statement>

Status: Proposed | Accepted | Superseded by <link>
Context: <the forces and constraints>
Decision: <what was chosen>
Alternatives considered: <each, and why rejected>
Consequences: <what this makes easy, and what it makes hard>
Revisit if: <the observable condition that changes this>
```

Record the reason. A decision without its rejected alternatives gets re-litigated.

**API docs**

Document the contract a caller must know: parameters, types, required or optional, return shape, every error the caller can receive, rate limits, and authentication. Show one realistic request and response each. Document failure modes; they are what callers actually hit.

**Changelog**

Group by Added, Changed, Deprecated, Fixed, Removed, Security. Write for someone deciding whether to upgrade. Reference issues. Do not describe the diff; describe the effect on the caller. Follow Keep a Changelog and SemVer if the project already does.

**Runbook**

Trigger, prerequisites, exact steps, how to confirm success, how to roll back. Written for someone under pressure. Assume no context.

## Comments

Comments explain **why**, not **what**. The code already says what.

Write a comment when:

- The reason is invisible from the code: a constraint, a workaround, a rejected alternative, a performance requirement.
- Something is genuinely surprising and a reader would otherwise "fix" it.
- A reference like an issue, RFC, or spec URL keeps the reasoning traceable.

Do not write a comment when:

- It restates the code. `// increment i` above `i++` is noise.
- It narrates the diff. Comments describing a change get stale the moment the code moves again.
- It explains a language or library feature the reader already knows.

For a function or module, a short header is worth it: purpose, non-obvious constraints, invariants, and what it must not do. Keep it short.

Delete comments that are now wrong. Leaving them is worse than having none.

### Why now, versus what changed

The distinction that matters: a comment may explain **why the current code looks this way**, or it may record **what changed to get here**. Only the first earns its place.

| Keep | Delete |
| --- | --- |
| `// liveness failed on low-end Android; PIN replaced face unlock` — the decision and its reason | `// changed from face unlock to PIN` — pure narration, git already holds it |
| `// 5ms is deliberate, the retry budget assumes it` | `// TODO: reinstate face unlock when the API is ready` |
| `// must stay last: the writer depends on ordering` | `// migrated from the v1 schema` when a migration records it |

Why the difference is not cosmetic. A comment narrating a change goes stale the moment the code moves again, and a stale comment is a claim a reader must verify. Once readers learn that comments here drift, they discount all of them, including the ones that prevent a real mistake. One bad comment costs you every good one.

The cost is not tokens. A comment that says why is invisible from the code and cannot be derived from it — that is why it exists. A comment that says what changed duplicates `git log`, which never drifts.

**The test:** would this comment still make sense if the reader already knew the history? If not, it is narration.

**The exception:** keep the historical note when it records a decision that could plausibly be re-litigated — a removed feature, a rejected approach, a constraint someone will hit again. Then say *why*, and say what happens if someone reverses it. That is institutional knowledge, and deleting it loses it for real.

### A comment that instructs a future agent

Worst case, and distinct from both above:

```js
// TODO: reinstate face unlock once the API is ready
```

That is not documentation, it is a pending requirement with nothing marking it historical. It survives the refactor, sits in the auth module, and reads as live intent. A TODO with no owner or condition. Either give it a tracking issue and a condition, or delete it.

## Anti-patterns

| Pattern | Why it fails |
| --- | --- |
| Commented-out code | It rots and gets read as meaningful. Delete it; git remembers. |
| `TODO` with no owner or condition | Accumulates forever. Add an issue reference or remove it. |
| Restating the spec in a comment | The spec changes; the comment does not. |
| Doc that describes intent, not behavior | Readers check behavior. |
| Example that was never run | Always wrong, often subtly. |

## Output shape

Report which claims you verified against the code, which you corrected, and which you could not confirm.

```text
Updated: <path> — <what changed>
Verified: <commands, paths, flags, code samples actually executed or read>
Corrected stale claims: <doc said X, code does Y>
Unverified: <what you could not check>
```

## Rules

- Every command and code sample must be executed or read before it is published.
- Never document behavior you did not verify. Write the verification first.
- Match the existing documentation's tone and format.
- Prefer deleting a wrong doc over rewriting it into another wrong doc.
- Do not invent product behavior to fill a documentation gap. Mark it as an open question.