# research-bridge — local Rust MCP research server

- 2026-11-02: built at `/home/benkaesy/Projects/research-bridge` by the Penny Pincher subagent. Rust, single binary, MCP over stdio.
- Installed binary: `~/.local/bin/research-bridge` (4.0 MB). Source binary: `target/release/research-bridge` (4.0 MB). Idle RSS after MCP `initialize`: ~5.5 MB, 14 threads (tokio runtime).
- Eight tools: `research_start`, `research_search`, `research_read`, `research_source`, `research_note`, `research_status`, `research_report`, `research_list`.
- Storage: `~/.local/share/research-bridge/<sessionId>/` with `session.json`, `searches.jsonl`, `sources.jsonl`, `claims.jsonl`, `sources/SN.txt`, `report.md`.
- Trust gate: `research_note` accepts a claim only when its quote is an exact substring of the stored source after whitespace normalization; `research_report` re-verifies every quote and excludes unverified claims.
- Search: DuckDuckGo HTML by default; optional self-hosted SearXNG via `SEARXNG_URL`. No API key.
- Read: HTML/text/JSON/PDF; PDF via `pdftotext`. No browser, JavaScript, login, paywall, or OCR.
- Security: public http/https only; localhost/private/link-local/CGNAT blocked unless `RESEARCH_ALLOW_PRIVATE=1`; manual redirects; size caps; untrusted external content.
- Wired into `~/.config/opencode/opencode.json` as `mcp.servers.research-bridge`, command `~/.local/bin/research-bridge`, env `RESEARCH_HOME`. `opencode mcp list` reports **connected**; `research_list` returns `{count: 0, home: ...}`.
- Build/tests: `cargo fmt --check`, `cargo clippy --all-targets -- -D warnings`, `cargo test` (18 unit + 4 integration), `cargo build --release` all pass on `rustc 1.99.1`.
- `AGENTS.md` global rule: web research uses `research-bridge`; search snippets are leads, not evidence; exact quote via `research_note`; report only verified claims.
- Deferred: `spawn_blocking` for blocking handlers, extraction quality (`tl` vs `scraper`/readability), PDF title heuristics, gzip, multi-process locking, binary/RSS tuning. Do not `cargo clean` while the project is active.
- Not a git checkout; no commits or push. OpenCode config and this record are local changes.
