# Principles

The rules his decisions already follow. Apply them when judging an idea, and name the one that decided it.

1. **No bloat.** A new helper, cache, queue, flag, or layer must replace something or close a named gap. Proof of absence before adding: search for whatever already owns that concern.
2. **Measure before optimising.** Time the thing in the environment it was complained about. Name each cost separately, largest first. Re-measure after each change — one change, one measurement.
3. **Smallest correct change, at the level the problem lives.** Delete before repairing. If the cause is a layer down, fix it there.
4. **Reversible by construction.** Back up before every write, state the rollback, prefer additive to destructive.
5. **Evidence over assertion.** Never upgrade an assumption into an observation. Report unrun checks as unverified.
6. **Boring mechanisms.** Standard library, platform feature, the pattern already in the repo. Cheaper primitive first: a direct Win32 call beats a CIM query; reading a file beats spawning a process.
7. **One system, many compositions.** Tokens, type, voice, and components stay constant; layout and density follow the page's job.
8. **The agent's surface is the product.** An agent should never need the shell for something a tool can do. If it does, the tool is missing something.
9. **Fix the class, not the instance.** One symptom appearing twice is a structural problem, not two bugs.
10. **Never invent product behaviour.** Prices, limits, roles, permissions, guarantees: if it is unknown, say it is the user's call.
