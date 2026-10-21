# Projects

## browser-bridge — agentic browser over MCP

- Repo `github.com/benkaesy820/browser-bridge` (private). Tool checkout `C:\BrowserX-AgenticBrowser` — what the MCP server runs from. Dev tree `D:\Project\BrowserX-AgenticBrowser` — develop and push from here.
- One visible browser shared by the human and the agent, driven by typed tools. The MCP frontend is a thin relay to a background daemon; Edge on Windows.
- Agent surface: observe, act, batch, search, tabs, diagnostics, takeover, storage, plus `browser_read` (the page as text, markdown, links, or html) and `browser_js` (expression escape hatch).
- Speed, MCP tool p50: observe 13.9ms, storage 5.6ms, search 18.5ms, fill 69.9ms, click 81.5ms, batch 92.3ms, screenshot 102.8ms, start 410.7ms. Session stress 300ms per open/close cycle, no process leaks.
- Open (2026-10-21, verified in the dev tree): the M3 fallback control page — one-flow's last residue: owner state, hold/resume, last N events, no preview frames (`docs/mcp-plan.md`, `docs/one-flow.md`). How often the in-page ownership bar actually fails to install is still unmeasured.
- Closed 2026-10-21: read/eval contract tests and the 40k-node walk budget (commit 9580e5e, `tests/contract/read-eval.test.ts`); the benchmark runs in CI (`.github/workflows/ci.yml`); MCP frontends are byte relays to the daemon's MCP pipe, so the relay no longer dies with the daemon.
- Upstream 0.5.0 (React viewer + continuous screencast) stays reachable at `origin/backup/main-0.5.0-2026-10-21`; it was removed on purpose in 6b6e3f8.
- Care: `D:` has lost files once and pnpm's store links broke. Keep the working copy on `C:`; treat `D:` as a mirror.

## opencode-config — the agent's own configuration

- Private repo `github.com/benkaesy820/opencode-config`: `AGENTS.md`, the skill set, agent definitions, host config. `service.json` holds a live service credential and stays untracked.
- Versioned so that a change to a skill is reviewable before it is used.
- Model routing: the operator picks the model. The config names no model at all, and a subagent inherits the session model.

## Windows environment

- Windows Terminal in a Tabby-like theme: Tabby dark and light schemes, auto-switching with the OS theme, Cascadia Code, ligatures off, opaque, no acrylic.
- PowerShell prompt: lean powerline, light and dark aware, with git state and an ownership bar.
- US keyboard layout.

## Open threads

- The OpenCode TUI re-probes the terminal background only on focus-in, so switching the OS theme needs a click back into the window.
- Superseded 2026-10-21: the read/eval paths in browser-bridge now have contract tests and a node budget (commit 9580e5e).
