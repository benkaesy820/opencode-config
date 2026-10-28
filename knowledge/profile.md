# Profile

Last updated: 2026-10-28.

## The person

- Ben. US keyboard layout. Lives in a clipped lowercase register; reads fast; wants the substance, not the scaffolding.
- Superseded 2026-10-23: Windows 11 on a desktop, OpenCode run as administrator.
- 2026-10-23: moved to CachyOS Linux — niri Wayland + noctalia-shell on an HP EliteBook 845 G7 (AMD Renoir, 7 GB RAM). Windows desktop retired.
- Builds tools for AI agents, and tunes the machine those agents run on.
- Writes in English.

## What he is working on

- `browser-bridge` — an agentic browser driven over MCP: one visible browser shared by a human and an agent, typed tools, human takeover. See `projects.md`.
- The agent's own configuration — skills, `AGENTS.md`, agent definitions. `opencode-config` is a checkout again since 2026-10-28; commits `fa41317`, `9841a8f`, `8e31e4d`, `dc24c52` are local, not pushed.
- Superseded 2026-10-23: Windows environment work — Windows Terminal theme, PowerShell prompt, keyboard layout.
- 2026-10-23: Linux environment work — niri keybinds, Foot terminal, display scale, package hygiene.

## Taste

- Wants the look of Tabby: dark and light schemes that follow the OS theme automatically, no transparency, a small precise cursor, a quiet tab strip.
- 2026-10-26: the terminal is **kitty** (foot removed). `~/.config/kitty/kitty.conf` is Tabby-like on purpose — opaque, a beam cursor that does not blink, a quiet separator tab strip at the bottom — and Noctalia drives its colours live, so the terminal follows the desktop theme.
- Speed is a feature. Measure before optimising, re-measure after, name the instrument.
- Hates bloat — "love efficient, hate bloat". A new mechanism must replace something or close a named gap.
- 2026-10-27: portability is a hard requirement for the apps he ships — one codebase that runs on Linux now and moves to other desktops later without a rewrite, and that he can keep maintaining himself. Weight, reliability and efficiency rank above ecosystem size.
- Likes the boring mechanism: standard library, platform feature, the pattern already used in the repo.
- 2026-10-26: one downloader — the browser's built-in; an external one only earns a place if a measurement shows it beating native (FDM removed on exactly that evidence).
- 2026-10-27: one browser — Helium (`helium-browser-bin`, Chromium fork); Firefox uninstalled entirely at his request and every browser default verified pointing at Helium.
- Changes must be reversible, with the rollback stated.
- Evidence over assertion: executed beats reproduced beats traced beats read. "Unverified" is an honest answer.
- 2026-10-28: trusts the model to follow the written rules; the permission list is a backstop for the catastrophic class, not a sandbox.
- Superseded 2026-10-23: prefers Edge over Playwright's bundled Chromium — the installed browser, used as-is. On Linux the bridge only ships bundled Chromium, and that is accepted.
- Superseded 2026-10-23: prefers Windows Terminal to be the default terminal where possible.

## How he wants to be talked to

- Give the recommendation you actually hold, with the reason. Do not present a menu.
- Uses session forks as parallel branches: the main thread keeps working while a fork takes a different job (stated 2026-10-21).
- Be willing to propose something he did not ask for when it solves the problem better — say why, and what it costs to switch.
- Concise. Speak only when there is something to say.
- Plain words, not simplified thinking: keep the terms that carry meaning, explain unfamiliar ones once, never soften a real constraint.

- 2026-10-27: wants an explicit done/deferred ledger in every handoff — what was built, what was deliberately deferred, and the trigger for each deferred item. Do not bury a deferral in reasoning; say "not now, because X; it lands when Y".
- 2026-10-27: **MFA is not used in republic-mart v2.** Do not plan TOTP, authenticators or recovery codes. The frozen schema's authenticator tables stay unused.
- 2026-10-27: **the old TypeScript project is off limits.** Do not read `~/Projects/republic-mart` again; ask Ben when a product rule is needed.
