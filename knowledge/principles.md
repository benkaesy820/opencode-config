# Principles

The rules his decisions already follow. Apply them when judging an idea, and name the one that decided it.

1. **No bloat.** A new helper, cache, queue, flag, or layer must replace something or close a named gap. Proof of absence before adding: search for whatever already owns that concern.
2. **Measure before optimising.** Time the thing in the environment it was complained about. Name each cost separately, largest first. Re-measure after each change — one change, one measurement.
3. **Smallest correct change, at the level the problem lives.** Delete before repairing. If the cause is a layer down, fix it there.
4. **Reversible by construction.** Back up before every write, state the rollback, prefer additive to destructive.
5. **Evidence over assertion.** Never upgrade an assumption into an observation. Report unrun checks as unverified.
6. **Never consult the old TypeScript project.** `~/Projects/republic-mart` is not a source of truth and must not be read again — not for features, not for behaviour, not for comparison. When a product rule is needed, ask Ben and record his answer. The old project was a good product but not the efficiency target.
7. **Every surface names its users.** Before building anything, ask how every kind of user meets it at that level: anonymous, buyer, seller, courier, staff, admin, superadmin, suspended and closed. If it is not per-user, say it is transport; if it is, list every kind and what each may and may not do. A rule that is true for one kind and false for another is a decision, not a detail.
8. **Boring mechanisms.** Standard library, platform feature, the pattern already in the repo. Cheaper primitive first: a direct Win32 call beats a CIM query; reading a file beats spawning a process.
9. **One system, many compositions.** Tokens, type, voice, and components stay constant; layout and density follow the page's job.
10. **The agent's surface is the product.** An agent should never need the shell for something a tool can do. If it does, the tool is missing something.
11. **Fix the class, not the instance.** One symptom appearing twice is a structural problem, not two bugs.
12. **Never invent product behaviour.** Prices, limits, roles, permissions, guarantees: if it is unknown, say it is the user's call.
13. **Inherit, do not restate.** An agent's own config must not repeat what the global config already owns, and order matters: a blanket allow placed after the asks overrides them (2026-10-21, commit ade7fb7).
14. **The system Node stays on LTS.** JS tooling installs user-space (`npm` global prefix `~/.local`, set 2026-10-28; pnpm global was the route until the pnpm binary disappeared), not Arch packages that depend on `nodejs`, because that dependency can drag the system Node to 26 and break anything pinned to 22–24 (2026-10-23, browser-bridge; corrected 2026-10-28).
