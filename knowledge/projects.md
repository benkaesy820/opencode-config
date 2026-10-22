# Projects

## browser-bridge — agentic browser over MCP

- Repo `github.com/benkaesy820/browser-bridge` (private). Tool checkout `C:\BrowserX-AgenticBrowser` — what the MCP server runs from. Dev tree `D:\Project\BrowserX-AgenticBrowser` — develop and push from here.
- One visible browser shared by the human and the agent, driven by typed tools. The MCP frontend is a thin relay to a background daemon; Edge on Windows.
- Agent surface: observe, act, batch, search, tabs, diagnostics, takeover, storage, plus `browser_read` (the page as text, markdown, links, or html) and `browser_js` (expression escape hatch).
- Speed, MCP tool p50: observe 13.9ms, storage 5.6ms, search 18.5ms, fill 69.9ms, click 81.5ms, batch 92.3ms, screenshot 102.8ms, start 410.7ms. Session stress 300ms per open/close cycle, no process leaks.
- Open (2026-10-21, verified in the dev tree): the M3 fallback control page — one-flow's last residue: owner state, hold/resume, last N events, no preview frames (`docs/mcp-plan.md`, `docs/one-flow.md`). How often the in-page ownership bar actually fails to install is still unmeasured.
- Closed 2026-10-21: read/eval contract tests and the 40k-node walk budget (commit 9580e5e, `tests/contract/read-eval.test.ts`); the benchmark runs in CI (`.github/workflows/ci.yml`); MCP frontends are byte relays to the daemon's MCP pipe, so the relay no longer dies with the daemon.
- Closed 2026-10-21 (main at a44a59d, pushed, tree clean): the relay recovers a dead daemon — it restarts it, reconnects, and ignores events from the socket it replaced; swallowed failures surfaced (stop failure, MCP connect failure, dead catch); the markdown renderer now emits text nodes and keeps anchor hrefs; `browser_js` reports itself as a mutation.
- 2026-10-21 (commit c4f55c4): the ownership bar is measured. `browser_diagnostics` reports `ownershipBar` per tab and per session — attempts, installs, failures, removals, deliberate skips (challenge, headless) and failure reasons (inject-error, not-installed). An install is verified in the page, not assumed; a deliberate skip is never a failure. This is the instrument the M3 fallback page was gated on; the number itself still has to be collected from real use.
- The headed suite (`BRIDGE_TEST_HEADED=1`) had been skipped since the pointer and the bar were restyled, so two assertions still described the old 18x24 arrow and the old chip copy. Proven stale by running the suite at HEAD with the change reverted, then repaired. Without that flag `test:integration` silently skips 5 tests, so a green run says nothing about them.
- 2026-10-21 (commits 6d173a3, 56d34ab): a control the agent NAMED survives the page moving under it. A ref is a position and rots; an identity (role + name) survives. So a named action observes once more and acts on the control with that identity, in the same call, and a name that matches nothing in the registry is looked up again before it is reported missing. A ref on its own still says "this position" and stays dead across a human interaction — the takeover contract depends on that. Measured: typing into a search box went from six tool calls and one failure to one call.
- Upstream 0.5.0 (React viewer + continuous screencast) stays reachable at `origin/backup/main-0.5.0-2026-10-21`; it was removed on purpose in 6b6e3f8.
- Care: `D:` has lost files once and pnpm's store links broke. Keep the working copy on `C:`; treat `D:` as a mirror.

## opencode-config — the agent's own configuration

- Private repo `github.com/benkaesy820/opencode-config`: `AGENTS.md`, the skill set, agent definitions, host config. `service.json` holds a live service credential and stays untracked.
- The checkout is the live config directory itself, `C:\Users\BenKaesy\.config\opencode` (origin = that repo). There is no second clone; the working tree is the config OpenCode reads. The knowledge base lives in the same repo under `knowledge/`, so a KB edit and an agent-config edit land in one commit.
- Versioned so that a change to a skill is reviewable before it is used.
- 2026-10-21: `@kerneloops/opencode-session-tools` (v0.1.1, MIT, no dependencies) is installed. It adds `opencode_meta` (session metadata, tree, stats, tool usage), `recall` and `recall_get` (search and read other sessions' transcripts), and `send_message` (ask another session a question, Claude Code style). Installed with `opencode plugin add`.
- Its search index only covers sessions seen since the plugin started; older sessions are still reachable by session ID, and the direct read works on any session.
- `send_message` wakes the target session's agent, which then runs tools and can write files. It is not a read-only query. The read-only primitive, `ctx.session.generate`, is not exposed as a tool by the plugin.
- Model routing: the operator picks the model. The config names no model at all, and a subagent inherits the session model.
- 2026-10-21 (commit ade7fb7, HEAD): `default_agent` is `advisor`; `agents/advisor.md` is `mode: all` with no permission block of its own, so it inherits the blanket allow while the global destructive-command asks still apply (they sit after the allow — an allow placed after them would override them). The advisor arrived read-only in f445d9c and gained full capability here.
- 2026-10-21 (same commit): bloat sweep of `AGENTS.md` and five skills (agent-config, code-review, deep-audit, design-audit, workflow); `browser-bridge` left untouched. `workflow` lost the optional Project Brain for "Memory across sessions" (project `AGENTS.md` + the knowledge base).

## Windows environment

- 2026-10-21: telemetry blocked machine-wide. Office: connected experiences off (privacy = 2), diagnostic data off (sendtelemetry = 3, DisableTelemetry = 1), customer data and feedback off, first-run movie and opt-in suppressed, set at both HKLM and HKCU.
- Windows: AllowTelemetry = 0, WER disabled, CEIP policy 0, DiagTrack and dmwappushservice disabled, and five telemetry scheduled tasks disabled (Compatibility Appraiser Exp, CEIP Consolidator, UsbCeip, SustainabilityTelemetry, Office Performance Monitor).
- Edge: AllowTelemetry, DiagnosticData and Edge3PSerpTelemetryEnabled = 0, BackgroundModeEnabled = 0. An earlier debloat pass had already set SmartScreenEnabled = 0 and a wide feature-off set; that was pre-existing and left untouched.
- Rollback and the pre-change registry exports live in `C:\Users\BenKaesy\Documents\telemetry-rollback-2026-10-21\` (run `rollback.ps1` elevated).
- Windows Terminal in a Tabby-like theme: Tabby dark and light schemes, auto-switching with the OS theme, Cascadia Code, ligatures off, opaque, no acrylic.
- PowerShell prompt: lean powerline, light and dark aware, with git state and an ownership bar.
- US keyboard layout.

## Open threads

- The OpenCode TUI re-probes the terminal background only on focus-in, so switching the OS theme needs a click back into the window.
- Superseded 2026-10-21: the read/eval paths in browser-bridge now have contract tests and a node budget (commit 9580e5e).
