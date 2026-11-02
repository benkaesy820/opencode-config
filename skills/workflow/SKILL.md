---
name: Workflow
description: Load this first for any non-trivial task — anything spanning more than one or two files, touching a contract, or risky. Classifies risk (LIGHT/STANDARD/CRITICAL), routes to the right skill, sets verification gates, and defines what to do when code cannot be run. Triggers on "how should I approach this", "what's the plan", planning or scoping work, starting an unfamiliar area, or before any multi-file, irreversible, or security-sensitive change. Load it early when unsure which skill applies.
---

# Workflow

Lifecycle, risk model, routing, gates, and where decisions live. Load this before editing in unfamiliar territory or for any task touching more than a couple of files.

## 1. Lifecycle

| Phase | Action |
| --- | --- |
| Orient | Restate the outcome and the constraints. Identify the smallest relevant surface. |
| Survey | Read the manifest, lockfile, task runner, CI config, and nearby tests to learn project facts. |
| Assess | Check working-tree state (`git status`) and classify risk using section 3. |
| Route | Pick one primary skill, plus supporting skills only if genuinely involved (section 2). |
| Decide | If a load-bearing choice is missing, load the `architect-decisions` skill. |
| Build | Smallest correct change, made at the level the problem actually lives. Reuse existing patterns. Keep the diff scoped. |
| Verify | Run the gates for the risk level (section 4). Record actual results. |
| Hand off | Report per section 6 of `AGENTS.md`. |

## 2. Routing table

Load one primary skill. Add a supporting skill only when that area is actually touched.

| Task shape | Primary | Supporting |
| --- | --- | --- |
| Load-bearing contract choice, unclear ownership or permission model, irreversible schema or API direction | `architect-decisions` | none |
| Reviewing a diff, PR, or branch for correctness | `code-review` | `rust-review` for Rust; `security-audit` if auth, input handling, or secrets are in scope |
| Exhaustive audit or full sweep: "find everything", pre-release verification, audits run by subagents | `deep-audit` | `code-review` for severity and output; `security-audit` when auth, input, or secrets are involved |
| Removing dead code, unused dependencies, or reducing bloat | `code-review` | `git-workflow` for the removal commit |
| Something is broken or behaving unexpectedly | `debugging` | `testing` once the root cause is found |
| A symptom that points at something bigger: recurring bug, one failure in several places, a fix that became a workaround, a performance ceiling | `root-cause` | `architect-decisions` when the change is load-bearing |
| Writing or repairing automated tests | `testing` | `debugging` if the failure is not understood |
| A "keep going until X" objective: make tests pass, fix until green, run-until conditions | goal plugin (`/goal`) | `debugging` for the failures it hits; `testing` for the check itself |
| Auth, secrets, input validation, dependency risk, exposure review | `security-audit` | `code-review` |
| READMEs, ADRs, API docs, comments, changelogs | `documentation` | none |
| Branching, worktrees, commits, rebase, tags, release prep | `git-workflow` | `code-review` before proposing commits |
| Endpoint or interface design: REST, GraphQL, RPC, schemas | `api-design` | `architect-decisions` for versioning or breaking changes |
| Building new UI or reshaping an existing one: aesthetic direction, typography, palette, layout | `frontend-design` | `api-design` for forms backed by an endpoint |
| Reviewing UI, UX, or design quality: usability, accessibility, states, responsive, copy | `design-audit` | `browser-bridge` for evidence; `frontend-design` for the quality bar |
| Operate a website or test a local app in a real browser | `browser-bridge` | none |
| Changing how a running machine or an installed app behaves or looks: theming, shell prompts, app settings, compositor or D-Bus config, startup cost, keyboard layout, default apps | `live-system` | `frontend-design` when the look itself is the deliverable |
| Changing the agent's own configuration: AGENTS.md, skills, plugins, MCP servers, model routing | `agent-config` | none |
| How OpenCode itself works: config schema, CLI, API, SDK, plugin API | `opencode` | `agent-config` only when changing Ben's configuration |
| Unclear category | this skill alone | none |

Rules:

- Do not load skills speculatively. Two skills is normal; three is the ceiling for a single-domain task; a composite audit may need four.
- `frontend-design` is not speculative when new UI is being built or an existing one reshaped. It does not apply to a one-line bug fix, a copy change, or a purely behavioral edit to existing markup.
- If the task turns out to be a different shape mid-flight, reload rather than accumulating skills.
- Skills are advisory procedure, not permission. They never widen what the user authorized.

## 3. Risk model

Classify before editing. The level sets the verification gates.

**`LIGHT`** — one file, no schema, no contract, no public surface, trivially reversible.

Examples: a typo fix, a comment, a local variable rename, a single test correction.

**`STANDARD`** — multiple files or modules, but no irreversible or externally visible contract change.

Examples: a bug fix in internal logic, a new internal helper, a feature behind an existing pattern.

**`CRITICAL`** — any of the following. If you are unsure which bucket applies, use the higher one.

- Data destruction, migration, or backfill.
- Authentication, authorization, sessions, secrets, or payment handling.
- Public API or schema change, deprecation, or versioning.
- Concurrency, transactions, retries, idempotency, or distributed coordination.
- Delivering something to a customer: sending mail or messages, charging a card, pushing to a registry, deploying, mutating production.
- Deleting or moving files others depend on.
- Anything the user described as critical, irreversible, or production.

State the level and the reason in one line before starting a `CRITICAL` task.

## 4. Verification gates

**`LIGHT`** — reread the changed lines. Confirm nothing else referenced what you touched.

**`STANDARD`** — the project's lint/typecheck and the tests covering the changed area, both actually executed. Re-read the full diff. No unexplained new warnings.

**`CRITICAL`** — everything in `STANDARD`, plus:

- Re-read the diff against the original requirement, line by line.
- Confirm every assumption from the summary is actually satisfied by the diff.
- Name the rollback path, or state that there is none.
- Confirm no secrets, credentials, or production identifiers appear in the diff.
- State explicitly what was *not* verified.

**Proof for behavior changes** — checks are not the whole gate. Observe the behavior yourself: reproduce the failure and watch it pass, call the endpoint, run the app and look at the flow. Green tests through mocks do not prove the user-facing result. If you cannot observe it, say `Unverified` and name the observation that would confirm it.

Report unrun checks as `UNVERIFIED`. Never upgrade an assumption into a result.

## 5. Memory across sessions

Decisions survive in two places, and nowhere else:

- **Project decisions** go in that project's own `AGENTS.md` (`AGENTS.md` section 2). The repository and its git history already hold the rest.
- **Standing personal decisions** go in the knowledge base, `~/.config/opencode/knowledge/`, which the advisor maintains.

Do not start a third copy. An entry that duplicates the code, the tests, or git history is waste. A signal names the decision behind a defect; if no decision exists, the missing record is the cause, so write it before fixing the class.

## 6. Blocking and asking

Ask the user directly when:

- Two reasonable interpretations would produce materially different software.
- The change is destructive and the target is not unambiguous.
- A requirement contradicts itself.
- Authorization is missing for anything irreversible or customer-visible.

Everything else: make the smallest defensible inference, state it in one line, and proceed (`AGENTS.md` section 3). A contract or design decision with several defensible answers goes to `architect-decisions`, not to the user.

## 7. When code cannot be run

If there is no shell, no runtime, or no dependency install available:

1. Do the full analysis and write the complete change anyway.
2. Give the exact commands the user should run, in order, with the expected result for each.
3. List the assumptions the change depends on.
4. Mark the work `UNVERIFIED` in the handoff.
5. Wait for the user's results before making any claim of success.

Never phrase inferred behavior as observed behavior. "This should return 404" is honest; "this returns 404" is not, until someone ran it.

Suggested handoff shape:

```text
UNVERIFIED. Cannot execute <reason>.

Changed:
- <path> — <one line>

Run:
1. <command>   → expect <result>
2. <command>   → expect <result>

Assumes: <assumption>
```