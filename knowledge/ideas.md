# Idea log

One entry per idea. Verdict first, then the reason, then what to do first. Update an entry rather than appending a contradiction; mark a changed verdict superseded with its date.

## Contract tests for the read/eval paths

- 2026-10-21 — proposed, not decided. The newest code in browser-bridge has no tests; the markdown walker stack-overflowed on the first real page and was caught only by running it. Build now: a fixture with a big DOM, a missing body, a throwing expression, a CSP page.
- 2026-10-21 — done. Landed in 9580e5e as `tests/contract/read-eval.test.ts`: markdown, text, links, html, a huge document, a throwing expression, an empty document.
- Revisit if: the read path is replaced rather than extended.

## A node budget in the read walker

- 2026-10-21 — proposed, not decided. Output is capped at 40k characters, but the walk visits every element, so the cost is unbounded on a Docs-scale DOM. Same change as the contract tests; build them together.
- 2026-10-21 — done in 9580e5e: the walk stops at 40k nodes and reports the read as truncated.

## Run the benchmark in CI

- 2026-10-21 — proposed, not decided. The harness already exists and measures every operation; nothing runs it on change, so "fast" is a claim rather than a fact.
- 2026-10-21 — done: `.github/workflows/ci.yml` runs `pnpm benchmark` on every matrix job and uploads the results.

## Decouple the MCP frontend from the daemon

- 2026-10-21 — proposed, not decided. Stopping the daemon takes the relay with it and the host then keeps the MCP marked failed. Small change; removes a failure mode agents hit.
- 2026-10-21 — done: `bridge mcp --stdio` frontends are byte relays to the daemon's MCP pipe; the daemon owns the engine and browser, and a frontend can exit without taking sessions down.

## Viewer dashboard and screencast

- 2026-10-21 — deferred. Needs the daemon to serve static files and a screencast feature the Edge tree does not have. A port, not a copy of upstream. Revisit when the agent surface is solid.
- 2026-10-21 — verdict: not now, and not as a live viewer. The viewer and continuous screencast were dropped deliberately in the 0.1.0 scaffold (CHANGELOG.md:6-7); README, docs/one-flow.md:19-23 and docs/advisor-brief.md:27 all state the product direction: one visible window, no second UI, no screencast. A live screencast dashboard contradicts the ownership trust model (advisor-brief priority 5) and one-flow.md:82 explicitly excludes preview frames.
- 2026-10-21 — correction to the first entry: the daemon needs no new static-serving machinery. It is already Fastify (packages/cli/src/daemon.ts) with /v1/events SSE and /v1/artifacts/:session/:artifact. The real gap is the frame source — grep finds zero screencast code in packages/, so it is CDP Page.startScreencast in the engine, plus a frame channel, plus a UI.
- 2026-10-21 — the live viewer's real prerequisite is decoupling the MCP frontend from the daemon lifecycle, not screencast: a viewer means the daemon is a long-lived server the user starts deliberately.
- 2026-10-21 — better shape if it returns: a replay/post-mortem view over the existing journal, event stream and screenshot artifacts. No live frames, no ownership conflict, no new capture technology.
- 2026-10-21 — the "agent surface is solid" gate is now met (read/eval contract tests, walk budget, CI benchmark and the MCP-to-daemon relay all landed) and the answer is still no: the gate was wrong, the product direction decides.
- 2026-10-21 — the one dashboard-shaped item still worth building is the M3 fallback control page (`docs/mcp-plan.md` M3 remainder, `docs/one-flow.md` "Fallback page"): owner state, hold/resume, last N events, no preview frames, explicitly "not the viewer". It closes a real gap — on PDF, canvas-by-design and browser-internal targets the human loses the owner chip, the hold/resume button and the last-action summary, and the only resume path today is a raw `bridge call` from the terminal.
- 2026-10-21 — that page's open decision is auth, not layout: this daemon is deliberately closed to webpages (bearer token in a header, CORS never enabled). Upstream served an unauthenticated viewer shell protected only by loopback binding, and audit #16 says the Windows token is effectively unprotected anyway. Give the page a narrow, session-scoped token that can read state and resume; no CORS; the decision goes in an ADR and a security pass.
- 2026-10-21 — before building it, count overlay-injection failures in real use. Zero means build nothing and document the CLI path instead.
- Revisit if: the agent surface is solid AND the product direction changes — remote or headless use returns, or the human genuinely cannot watch the shared window.

## Standing answers

- "Do I need a plugin for this?" — No, until there is a hook the skill system cannot express.
- "Should the config name a model?" — No. The operator picks the model; the config names the procedure.
