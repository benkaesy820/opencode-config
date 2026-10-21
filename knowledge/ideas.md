# Idea log

One entry per idea. Verdict first, then the reason, then what to do first. Update an entry rather than appending a contradiction; mark a changed verdict superseded with its date.

## Contract tests for the read/eval paths

- 2026-10-21 — proposed, not decided. The newest code in browser-bridge has no tests; the markdown walker stack-overflowed on the first real page and was caught only by running it. Build now: a fixture with a big DOM, a missing body, a throwing expression, a CSP page.
- Revisit if: the read path is replaced rather than extended.

## A node budget in the read walker

- 2026-10-21 — proposed, not decided. Output is capped at 40k characters, but the walk visits every element, so the cost is unbounded on a Docs-scale DOM. Same change as the contract tests; build them together.

## Run the benchmark in CI

- 2026-10-21 — proposed, not decided. The harness already exists and measures every operation; nothing runs it on change, so "fast" is a claim rather than a fact.

## Decouple the MCP frontend from the daemon

- 2026-10-21 — proposed, not decided. Stopping the daemon takes the relay with it and the host then keeps the MCP marked failed. Small change; removes a failure mode agents hit.

## Viewer dashboard and screencast

- 2026-10-21 — deferred. Needs the daemon to serve static files and a screencast feature the Edge tree does not have. A port, not a copy of upstream. Revisit when the agent surface is solid.

## Standing answers

- "Do I need a plugin for this?" — No, until there is a hook the skill system cannot express.
- "Should the config name a model?" — No. The operator picks the model; the config names the procedure.
