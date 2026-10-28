---
name: Agent Config
description: Use when changing Ben's own OpenCode configuration — his AGENTS.md files, skills, agents, plugins, MCP servers, model routing, keybindings, or the tools they load. Triggers on "write a skill", "edit my AGENTS.md", "add an MCP server", "change the model routing", "why isn't this skill loading", "remove a skill", "the agent ignores my instructions", "add a command", "wire up a plugin". For how OpenCode itself works, the built-in `opencode` skill is the reference.
---

# Agent Config

The agent's configuration is executable documentation: it decides what the agent does before it does anything. A one-line change here alters every future session, so it needs the same rigour as a code change.

## 1. Find the effective config

- **There is usually more than one file, and they merge.** Global policy, project policy, and tool config live in different places and different formats. Read all of them before editing one.
- **The effective file is the one the tool actually reads.** Confirm the path from the tool's own documentation or a listing command, not from memory of an earlier session.
- **Check what is actually installed.** A plugin listed in a config file may not be installed; an MCP server may point at a path that no longer exists. Verify against the tool's own inventory, not the config text.

## 2. The description is the routing

- **A skill loads because its description matched the request.** The description is a trigger contract, not a summary — it should contain the situations and the words a user would actually use.
- **One skill per domain, with a crisp boundary.** Two skills that overlap are two chances to load the wrong one. If the boundaries blur, merge them.
- **Do not load skills speculatively.** A skill that is "not speculative" for its own domain becomes noise everywhere else.

## 3. Precedence

`AGENTS.md` §1 is the single source: the user's current request → project or nested `AGENTS.md` → global `AGENTS.md` → the loaded skill → repository conventions. When two layers disagree, follow the higher one and say so, briefly.

- A global policy file is for rules that hold in every project. Project-specific detail belongs in the project's own file, or it goes stale everywhere.

## 4. Change procedure

1. **Back up the file** outside the directory the tool scans.
2. **Make the smallest edit that achieves the intent** — a routing row, one bullet, one reference.
3. **Keep the file's voice and formatting.** These files are read by a model; consistency is part of their function.
4. **Say what is now different**, in one line.

## 5. Anatomy of a good skill

- **Frontmatter** with `name` and `description` — the description decides when it loads, so write it as a trigger list.
- **A body that is a procedure**, not an essay: the steps, the order, the decision points.
- **An output shape**, so results are consistent and reviewable.
- **Rules** that encode what has gone wrong before.
- **One file** unless the skill genuinely needs scripts or references.
- **A skill that cannot be applied is a liability.** If a skill's guidance has no reachable trigger, delete it rather than leaving it to compete for attention.

## 6. Removing something

- **Check what references it first.** A skill named in a routing table, a policy file, a plugin config, or another skill is a live reference; removing the target leaves a dangling instruction that the agent will try to follow.
- **Fix or remove every reference in the same change**, then grep to prove none remain.
- **Keep one copy outside the load path** so it can be restored without network access.
- **Never remove what you cannot show is unreferenced.**

## 7. Verify the change took effect

- A config edit that does not reload is invisible until you check it. Read the file back, or ask the tool to list what it loaded.
- If a plugin, MCP server, or skill fails to start, find out why before assuming the config is fine — the inventory command is the instrument.

## 8. Handoff

```text
Changed:
- <file> — <what changed, one line>

Verified:
- <listing or read-back that proves it loaded>

Broken references:
- <any reference updated or removed alongside it>

Rollback:
- <one command>
```

## Rules

- The description decides when the skill loads; write it as a trigger contract.
- Back up before editing; prove the change loaded.
- Remove a thing and its references together.
- Keep global policy general and project policy local.
