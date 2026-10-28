---
name: Live System
description: Use when changing how a running machine or an installed app behaves or looks — terminal themes, shell prompts, keyboard layouts, app settings, startup items, context menus, "make X look like Y", "make X start faster", "stop that banner appearing", "make this the default". Linux/Wayland first: XDG paths, the app's own schema, niri/noctalia IPC, D-Bus; Windows and Wine only as a branch. Covers finding where an app keeps its settings, grounding values in the app's own files instead of a screenshot, measuring start-up cost, verifying a change on a GUI you may not be able to see, and keeping every change reversible. Triggers on "configure my terminal", "theme my app", "customise my desktop", "why does X print that", "make startup faster", "change the default app", "add a shell prompt", "make it look like".
---

# Live System

A live system has no repository to read, no test suite to run, and no diff to review. The evidence is the running thing itself. This skill is the procedure for working on one without guessing.

## 1. Discover before configuring

- **Where does this app keep its settings?** On Linux, check in this order:
  1. the app's own docs and `--help`;
  2. `~/.config/<app>/` (user config), `/etc/<app>/` or `/etc/xdg/<app>/` (system defaults);
  3. `~/.local/state/<app>/` (runtime and GUI state), `~/.local/share/<app>/` (data);
  4. the app's schema or source (`--dump-config`, a published JSON schema, the theme file);
  5. the compositor or desktop service (`niri msg`, `noctalia msg config export`, `gsettings`, `dconf`, `busctl`);
  6. the running process (`/proc/<pid>/cmdline`, `lsof -p <pid>`) when it reads a path you cannot find.

  Windows apps fall back to `%APPDATA%`/`%LOCALAPPDATA%` and `HKCU`/`HKLM`; Wine prefixes keep the same shape under `$WINEPREFIX`.
- **Find the schema, not a blog post.** Prefer, in order: a machine-readable schema the app publishes or ships; a settings UI that can be inspected; the app's own source. A JSON Schema answers "which keys and value shapes are legal" — that is the contract, and guessing at it is the single most common way to break an app's config.
- **Record the app version and the exact path.** Config keys, defaults, and file locations are version-specific. The same key can mean different things across releases.
- **Find out what reads the setting.** A key that nothing reads is dead weight; a key read by a running process is live and may need a restart to take effect. On this desktop, `noctalia msg config export` is the merged-truth instrument: `config.toml` loads first, the GUI state wins, and `config-reload` applies edits live.

## 2. Ground truth in the app's own artifacts

- **Design tokens live in the app's source.** A colour palette is usually in a source file (`colorSchemes.ts`, a theme file, a stylesheet), not in a screenshot. Read it and use the real values; a screenshot is one rendering, not the spec, and colour profiles lie.
- **Prefer the tool that generated the values.** If the app ships a theme file, a colour scheme list, or a `--dump-config` command, use it rather than transcribing pixels. On Wayland, `kitty @ get-colors`, `noctalia msg config export`, and `niri msg --json` are better instruments than a screen grab.
- **Distinguish "the app's palette" from "the app's chrome".** Terminal background, tab bar, title bar, and window frame are often different systems with different sources.
- **Sample what is on screen when a value must be read from the screen.** Use `grim -g "$(slurp)"` or a region capture, one pixel at the exact region, with the surrounding context — an average across a window proves nothing.

## 3. Measure before optimizing

- **Time the thing the user complained about, in the environment they complained in.** An interactive shell start is not a piped `fish -c` run; a cold start is not a warm one. Use `hyperfine` for repeatable commands, `systemd-analyze` for boot, and the compositor's own timing when it exposes one.
- **Name each cost separately.** "It is slow" is not a cause. Split the total into its parts, largest first, and attack the largest.
- **Separate one-time cost from per-call cost.** First-call loading, cache builds, and theme compilation can dwarf everything else while looking like "the code is slow". Moving a cost off a measured path can be worth more than deleting it.
- **Re-measure after each change.** One fix, one measurement. Two changes at once and you cannot attribute the result.
- **Prefer the cheaper primitive.** Reading a file beats spawning a process; a direct platform call beats a wrapper — but only when the measurement says it matters.

## 4. Verify what you cannot see

Use the highest rung you can reach; state which rung you reached and what stayed unverified.

| Rung | Instrument | Proves |
| --- | --- | --- |
| 1 | Screenshot of the window (`grim`, the desktop's capture) | What the user sees |
| 2 | Pixel sample of the changed region | The exact value rendered |
| 3 | Protocol capture (`kitty @`, `niri msg`, D-Bus, OSC replies) | What the app emitted or read |
| 4 | Read back the file, D-Bus property, or registry value | What is stored |

- **A screenshot you did not take is not evidence.** Same rule as any other audit.
- **If the screen is locked, covered, or headless, drop a rung** — do not substitute a plausible claim for a measurement.
- **Probe the running system rather than inferring.** A terminal that answers an OSC query, a process that exposes a status field, a config file the app rewrites on change — each is a better instrument than a guess.

## 5. Reversible by construction

- **Back up before every write**, and put the backup outside any directory the app scans.
- **State the rollback in the handoff** — the exact command, not "undo the change".
- **Prefer additive to destructive.** Add a profile, a shortcut, a scheme; do not overwrite the user's existing set.
- **Targeted edits beat whole-file rewrites.** A rewritten file loses ordering, comments, and formatting the app or the user relies on.
- **Leave the workspace as you found it.** Close probe windows, remove temp scripts, undo test writes that were only for measurement.
- **Check the version's upgrade path.** A config a future update will rewrite is worth telling the user about.

## 6. Refuse to fake a capability

- **Before declaring it impossible, check whether a config key can do it.** Search the schema, the app's source, and its issue tracker.
- **If the app cannot do it, say why, with the upstream reason** — missing interface, unimplemented feature, tracked issue with a link. That is a far more useful answer than a workaround.
- **Never simulate a feature by hijacking a system mechanism.** Rewriting a system key to make an app appear to do something it cannot do is not a fix; it is a trap that fires later and breaks something else.
- **Name the honest approximation if one exists** — and be explicit that it is an approximation.

## 7. Handoff

```text
Changed:
- <app> — <what changed, and the exact file/D-Bus path>

Measured:
- <instrument> → <actual value>

Verified:
- <what you actually observed, and how>

Unverified:
- <what you could not observe, and the exact step to check it>

Rollback:
- <one command>

Next safe action:
- <what the user should do, if anything>
```

## Rules

- Read the app before touching the app's configuration.
- Measure before optimizing; re-measure after.
- Name the instrument behind every claim; never report a visual result you did not see.
- Every write ships with its rollback.
- Refuse to fake a capability; report the upstream reason instead.
