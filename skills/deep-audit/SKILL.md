---
name: Deep Audit
description: Use for exhaustive audits and deep verification — full-repo sweeps, pre-release checks, "find everything" reviews, live-install and operations audits, and audits run by subagents. Adds coverage ledgers, flow tracing, hunch discipline, falsification passes, audit-safety rules, and repeat-audit continuity on top of the subject skill (code-review, security-audit, testing, documentation, design-audit). Every reported finding is evidence-backed. Not for a quick verification pass on a small diff — that is `code-review`.
---

# Deep Audit

Maximum coverage. Minimum noise. An audit earns trust by proving what it checked and never overstating what it found.

One unproven finding discredits every real one, so precision is not negotiable. The subject skill — `code-review`, `security-audit`, `testing`, `documentation`, `design-audit` — supplies the checklist. This skill supplies the coverage, the discipline, and the evidence bar.

## 1. Frame the audit

Before reading anything:

1. Restate the outcome in one sentence: what must be true when this audit ends.
2. Classify it: first audit or repeat (§9); code, UI, live-state, or mixed; and which type from the presets (§15).
3. Define the boundary. A diff review covers the diff plus its direct callers, callees, tests, config, and consumers. Everything outside that is adjacent, not out of bounds.
4. Order the surfaces by risk, not by interest: auth, permissions, money, data loss, concurrency, external calls, public contracts.
5. If the premise cannot be audited — empty diff, missing files, wrong branch, contradictory brief — stop and say so. Auditing the wrong thing is worse than not auditing.

## 2. Coverage map

Sweep every applicable surface at least once before going deep on any single one.

| Surface | What to inspect |
| --- | --- |
| Change itself | Every hunk. Read the final state, not the diff-in-flight. |
| Callers | Every place the changed code is used; do its assumptions still hold? |
| Callees | What the code relies on: errors, ordering, side effects, timing. |
| Boundaries | Empty, zero, one, null, negative, huge, unicode, duplicate. |
| Error paths | Every call that can fail, early returns, cleanup, rollback. |
| State | What is written where; consistency on every exit; retries; idempotency. |
| Contracts | Serialization, schemas, signatures, wire formats, migrations. |
| Tests | Whether the change is covered, the assertions actually assert, and the fixtures match reality. |
| Config | Flags, env vars, defaults, secrets, environment differences. |
| Docs | Claims that now contradict the code. |
| Dependencies | Shipped third-party code: lockfiles, vendored or bundled runtimes, pinned versions, install scripts. |
| Live state | The running installation and its data — method in §7. |
| Consumers | Anything outside the repo that reads the artifact. |

Record every surface as exactly one of: **finding** (with evidence), **clean** (with the reason you believe it), **not checked** (with why). That ledger is the audit. A surface without a verdict is a hole.

## 3. Entry points and flows

A file-by-file read finds style and local bugs. Serious bugs live between files: trace flows, not files.

**Inventory first.** List every entry point — routes, CLIs, jobs, webhooks, imports, file watchers:

| Entry | Inputs | Trust | Sinks | Auth |
| --- | --- | --- | --- | --- |

Trust is what the caller controls. Sinks are databases, files, the network, the UI, logs.

**Trace each flow end to end:** input → validation → transformation → logic → persistence → output → consumer. Follow the value, not the call graph. Where a flow crosses a boundary — user→server, server→DB, parser→store, store→document, document→user — check both sides:

- Validated, at the boundary, once, before use?
- Encoded for the sink it is heading into?
- Authorized per object, not per route?
- Consistent on partial failure — what state remains if step N fails?
- Observable — can a failure be seen in logs or is it silently lost?

**Run these questions on every flow:**

1. Where does untrusted input reach a sink, and is it validated, encoded, and authorized for that sink? (`security-audit` holds the per-check detail.)
2. What state remains if step N fails? Retries, idempotency, concurrency.
3. Boundary values: empty, zero, one, null, huge, duplicate, unicode.
4. Resource limits: size, depth, timeouts, unbounded growth.
5. Parsing and data quality: separators, encodings, truncation, round-trips.
6. Error paths: swallowed failures, misleading messages, missing rollback.
7. Doc-versus-behavior drift: what the spec claims against what runs. (`documentation`)
8. Data agreement: every copy of a fact — DB, document, export, UI, backup — agrees.

## 4. Hunches are leads, not findings

Every gut feeling is written down and resolved. None is reported unproven.

```text
LEAD: <the suspicion, one line>
Check: <the cheapest observation that confirms or kills it>
Status: confirmed | refuted | cannot-verify — <why>
```

- Run cheap checks immediately. Queue expensive ones and timebox them.
- A lead ends **confirmed** (becomes a finding, with evidence), **refuted** (record what killed it — refuted leads are proof of coverage), or **cannot-verify** (goes to `Unverified` with the exact next step).
- Never let a hunch silently vanish. An unrecorded hunch is a coverage hole.
- Promote a lead to immediate deepest investigation when it touches: auth or permissions, money, data deletion or migration, concurrency or retries, serialization, a public contract, or code with no tests.

## 5. Evidence

Prefer the strongest evidence available, in this order:

1. **Executed** — ran it, with the command and output captured.
2. **Reproduced** — the failing case replayed on a copy, with output.
3. **Traced** — the code path walked end to end, cited `file:line`.
4. **Read** — the surrounding code, not just the line.

Anything weaker stays `Unverified` with the exact command or observation that would confirm it. Never upgrade an assumption into an observation. Keep the raw evidence — hashes, command output, timestamps — for every finding you report.

When you cannot run code: trace the values, cite lines, show the path end to end. Execution-dependent claims — races, timing, flaky behavior — remain `Unverified`. For each one, state the exact command that would confirm it so the caller can run it.

## 6. Audit safety

An audit must not change what it measures.

- Read-only by default. Open live databases read-only; never write to live state.
- Anything that must execute runs on a **copy**, never the original.
- Fingerprint the live artifacts before and after (hashes). When the audit ends, the report states exactly what was and was not modified.
- No fixing, no cleanup, no "while I was in there" changes. Report, do not repair.

## 7. Live state and operations

Code audits miss the install. When a live system exists:

- **Sources of truth.** List every representation of the data — database, source documents, baselines, exports, UI, backups — and check they agree. Say which one wins when they disagree.
- **Data integrity.** Orphan rows, nulls, duplicates, drift counts, impossible values.
- **History and forensics.** Logs, file mtimes, backup timestamps, request patterns; reconstruct what actually happened, with timestamps.
- **Backups.** Existence, freshness, and one restore rehearsal on a copy.
- **Environment.** Processes, stale PID files, ports, log growth, config drift between what is installed and what the repository says.

## 8. Sweep, then falsify

Three passes, always in this order:

1. **Sweep.** Walk the coverage map and the flows. Collect findings with evidence.
2. **Falsify.** Attack your own findings. For each: what upstream validation, downstream guard, constraint, or convention would make this wrong? Check it. A finding must survive its falsification attempt to be reported; one that does not was never real.
3. **Recheck.** Reread the brief. Ask what a second auditor would find that you did not. Force five candidate gaps, check the cheap ones, and list the rest as `not checked`.

Correlation is not causation. "It looks wrong" is not a finding. The short list that survives all three passes is the deliverable.

## 9. Repeat audits

If a previous audit ledger exists:

1. Re-verify every prior finding first — **fixed / open / regressed** — with evidence.
2. Sweep the delta since the last audit, plus everything the prior audit left `not checked`.
3. Carry the ledger forward. Do not re-audit unchanged code from scratch for sport.

The calling agent persists `audit-ledger.md` in the project — findings, status, evidence pointers, and the scripts worth keeping. The auditor is read-only.

## 10. Large targets

One context has limits. When the target exceeds a single reviewer's context, slice it:

- Slice by entry-point group or module domain — never mid-flow.
- One reviewer per slice, same ledger format, each read-only.
- The orchestrator merges: dedupe findings by mechanism; overlaps belong to the slice owning the entry point; concatenate ledgers; run the merged `Unverified` list last.
- A slice that returned cut surfaces is a gap, not a pass.

## 11. Budget and stopping

The audit ends when every surface has a verdict and every lead has an outcome — not when steps run out.

- Phase the budget roughly: 45% sweep, 25% verification, 15% falsification, 15% report. Adjust, but know the split.
- Triage: leads touching auth, permissions, money, data deletion, concurrency, or public contracts get uncapped investigation; everything else is timeboxed.
- Batch independent reads and searches into one step where possible: a step can carry many tool calls, and steps are the limited resource.
- If the budget ends with surfaces still open, stop and report the cut: what was not checked and which leads are unresolved. The ledger is the handoff; the caller continues in a fresh session from it.
- Never repeat work or hunt for new surfaces to fill the budget. A complete ledger with an empty findings list is a finished audit.

## 12. Beyond scope

You may follow a signal outside the asked scope — that is how some of the worst issues get caught. The discipline:

- It becomes an **adjacent observation**: same evidence bar, its own report section, never mixed into the findings.
- Investigate while it is cheap or plausibly high-severity. Timebox everything else and name the cut.
- Never expand a read-only mandate into fixing. Never expand unboundedly — if the audit is becoming a different audit, finish the original first and say what is left.
- If the premise itself is broken, that outranks everything else. Report it first.

## 13. Severity rubric

Use the subject skill's scale. Only when the subject has none, use this:

| Level | Anchor |
| --- | --- |
| **High** | Live-blocking, silent data loss, or integrity risk in official records; exploitable with a reachable path. |
| **Medium** | Real but bounded: security gap with limited reach, broken checks, fixture drift, bounded performance cost. |
| **Low** | Hygiene, operations, documentation drift, degraded-but-working behavior. |
| **Info** | Policy questions and decisions with no defect. |

Severity is impact × reachability, never alarm. Record the trigger, not just the level.

## 14. Report

Use the subject skill's severity order. Add these sections:

```text
<verdict>

Findings — <severity order; each: file:line, mechanism, trigger, impact, smallest fix, evidence>

Adjacent observations — <out-of-scope findings, same evidence bar>

Leads — <each hunch: confirmed | refuted (how) | cannot-verify (why)>

Coverage — <each surface and flow: finding | clean | not checked (why)>

Unverified — <what, why, and the exact next step for each>

Decisions needed — <policy questions the operator must answer, not defects>

Residual risk — <what could still be hiding, and why you believe it is unlikely>
```

An empty findings list is a valid and useful result. Padding it to look thorough is the one unforgivable failure.

## 15. Audit-type presets

| Type | Mandatory additions |
| --- | --- |
| Security review | Auth, per-object authorization, injection, secrets, dependency advisories (`security-audit`) |
| Dead-code sweep | Callers and consumers exhaustively enumerated; dynamic resolution ruled out (`code-review`) |
| Contract drift | Contracts, migrations, consumers, doc claims |
| Production-readiness | Live state and operations (§7), backups, runbooks, rollback |
| Pre-release | Test hermeticity, config diffs, rollout and rollback path |
| UI/UX | `design-audit`: screens × states × breakpoints × input modes |

## 16. As a subagent

Your report is the deliverable: the parent sees the coverage ledger, the leads, and the cuts, not your process. Every finding pointable, every claim checkable, no reliance on unstated context. Read-only unless the task says otherwise — report, do not repair. If an instrument is unavailable — browser, runtime, database — do the applicable half and list the rest as `Unverified` with exact steps for the caller. Spend steps on coverage and falsification, not on prose.

## Rules

- Flows, not files. A finding that crosses files traces end to end.
- Every finding survives its falsification pass and cites `file:line` and a trigger.
- Never report a hunch, a preference, or a pattern you did not verify.
- Prefer the strongest available evidence; never upgrade an assumption into an observation.
- Never mutate live state; prove non-mutation with fingerprints.
- Severity is impact × reachability, never alarm.
- Record refuted leads — they are evidence, not waste.
- Read the code around a line before judging it. A correct observation in the wrong context is a false positive.
- Do not grade your own accuracy and do not claim a percentage you did not measure; show the ledger instead.
- If the standard cannot be met within the budget, say exactly which surfaces were cut.
