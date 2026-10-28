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

## A from-scratch Rust "cognitive" model on the laptop

- 2026-10-25 — **parked.** Built, measured, and then stopped by Ben's decision; the project directory is deleted. The idea, the architecture, every measurement and the resume list live in `cognitive.md` in this directory; the code is archived at `~/cognitive-2026-10-25.tar.gz`. Nothing here is running — do not treat it as live work.

## Rebuild the marketplace backend in Rust

- 2026-10-26 — **decided: build it.** Ben chose a greenfield Rust rebuild of the republic-mart backend, the TypeScript one treated as nonexistent, with **footprint as the dominant goal** (it must fit the Northflank box: 0.1 shared vCPU, 256 MB). The database stays Turso; the Rust `libsql` client path is already proven working locally.
- The reason that survives scrutiny is not "Rust is fast" — it is that the current production image is a Node runtime plus a 345 MB dependency tree plus sharp plus ffmpeg, which is the real waste. A musl-static Rust binary with no bundled SQLite lands around 5–12 MB and tens of MB RSS.
- The reason it is *not* a port: the TypeScript backend is 58,665 lines over 106 files, 60 tables, 773 route registrations and 59 test files, and it was hardened on 2026-10-24/25 (email builders, order/delivery/dispute state machines, idempotency, money paths). Porting it would re-derive behaviour that is already right and risks drift exactly where the money is. A fresh build with the domain written down first is cheaper than a port and safer than a rewrite-in-place.
- The speed claim has to be kept honest: catalogue p99 is SQLite plus R2 plus the Moolre and Brevo round trips, so a faster CPU moves it 20–30%. The wins that move it are an in-process catalogue read model, one round trip per transaction, and instance sizing.
- Revisit if: the first vertical slice fails to hit the footprint budget, or the Turso round trip from the box measures worse than expected.

## Standing answers

- "Do I need a plugin for this?" — No, until there is a hook the skill system cannot express.
- "Should the config name a model?" — No. The operator picks the model; the config names the procedure.
