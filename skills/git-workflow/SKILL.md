---
name: Git Workflow
description: Use before committing, pushing, branching, stashing, rebasing, amending, cherry-picking, resetting, tagging, creating, parking or removing a worktree, or cutting a release, and whenever history could be rewritten or a remote touched. Triggers on "commit this", "commit and push", "open a PR", "make a branch", "create a worktree", "remove a worktree", "rebase onto main", "squash", "undo my last commit", "bump the version", "cut a release", or a request to prepare a commit message. Use it to keep diffs scoped and to know which git commands need explicit authorization first.
---

# Git Workflow

Preserve the user's work. Git operations that lose changes require explicit authorization, every time.

## Never run without explicit authorization

State the exact command and its effect first, then wait for a yes.

- `push`, `push --force`, `push --force-with-lease`, `push --delete`
- `reset --hard`, `clean`, `revert` on shared history
- `rebase`, `cherry-pick`, `filter-branch`, `filter-repo`
- `commit --amend`, `commit` on someone else's behalf
- `stash`, `stash pop`, `stash drop`
- `checkout`/`switch` with local changes that would be discarded
- `branch -D`, `tag -d`, `restore`, `checkout -- <path>`
- Anything touching a protected branch, tag, or default branch

Never run these "to clean things up" or "to start fresh". If the tree is messy, report it and let the user decide.

## Always safe

Read-only inspection:

```sh
git status
git diff
git diff --staged
git log --oneline -20
git show <sha>
git branch -vv
git stash list
```

Read the state before proposing anything.

## Before staging

```sh
git status                     # what changed, what is untracked
git diff                       # review the unstaged changes
git diff --staged              # review what is already staged
```

Then:

1. Confirm the change is scoped. Unrelated edits from other work get excluded, never swept in.
2. Look for secrets, credentials, `.env` files, and large binaries before staging.
3. Check for debug leftovers: `console.log`, commented-out code, focused/skipped tests, temporary files.
4. Check the diff will not break the build.

## Committing

- One logical change per commit. A commit should be independently revertible.
- Write the message in the imperative: `fix null deref in token refresh`, not `fixed` or `Fixed token bug`.
- Subject under about 72 characters, one line, no trailing period.
- Body explains **why**. The diff already shows **what**.
- Reference issues and PRs.

```
<area>: <imperative summary>

<why this change is needed and what approach was taken>
<relevant trade-off or constraint>

Refs #<issue>
```

Match the repository's existing commit style. Run `git log --oneline -20` and follow it rather than imposing a convention.

## Pushing

A push means the configured remote. A local commit is not a push.

- Push only when authorized. Then push the current branch to its upstream: `git push`, or `git push -u <remote> <branch>` when it has none.
- Verify the result: `git status -sb` shows the upstream and ahead/behind; `git log -1 --format='%H %s'` gives the SHA.
- If there is no remote or upstream, stop and report it. Do not invent one.
- Report the remote, branch and SHA. Never leave project work local-only when the task said push.

## Branching

```sh
git switch -c <type>/<short-description>
```

Common shapes: `feature/`, `fix/`, `chore/`, `docs/`. Follow the repo's existing pattern.

Never create or push a branch unless asked.

## Worktrees

A worktree is scratch space, not storage. Its source is cheap; its build cache is what makes iteration fast.

- Keep one worktree per active task. Remove it once its branch is merged: `git worktree remove <path>`, then `git worktree prune`.
- Do not clean build output while the worktree is active. Keep the incremental target so rebuilds stay fast. After the branch is merged, abandoned, or otherwise done, remove the worktree; if it must stay, clean its dev profile then.
- Never delete an unmerged worktree or uncommitted changes. Report it instead.
- Do not share `CARGO_TARGET_DIR` across parallel worktrees; the build lock and feature differences cause thrash. Use `sccache` if rebuild time matters.
- `cargo clean` is a completion or disk-pressure action, never routine housekeeping. Do not wipe an active target just to save space.

## Inspecting history

```sh
git log --oneline -20
git log -p <path>                 # how a file evolved
git log --author=<name>
git blame <path>                   # last change per line
git show <sha> --stat
git diff main...HEAD               # branch vs base
```

## Rebase and merge, when authorized

- Rebase your own unpushed branch; do not rebase shared history.
- Before rewriting, confirm the commits are unpushed and not someone else's.
- After rewriting, force-push only with `--force-with-lease`, never plain `--force`.
- Say clearly that history was rewritten, and give the recovery ref.

## Releases

1. Confirm the working tree is clean.
2. Determine the version from actual changes since the last tag. Use SemVer: breaking → major, behavior change → minor, fix → patch. Do not guess the version; state your reasoning.
3. Update the changelog and version file.
4. Run the full test, lint, and typecheck suite.
5. Create the tag and annotate it.
6. Push the tag only with authorization.

Never publish to a package registry, container registry, or any production target without explicit authorization and an unambiguous target.

## Recovering from a mistake

If something was lost:

```sh
git reflog                  # find the previous HEAD
git fsck --lost-found        # find dangling objects
```

Report what was recoverable and let the user restore it. Do not attempt an aggressive recovery yourself.

## Output shape

```text
<command you ran> → <actual result>
Files: <what is staged or changed>
Not included: <unrelated changes you deliberately left alone>
Needs authorization: <any destructive command proposed, with its exact effect>
```

## Rules

- Report the real result of every command. Never claim a commit or push succeeded without its output.
- Keep diffs scoped. Never sweep in unrelated work.
- Do not hand-edit `.git` internals.
- Do not create commits, branches, or tags unless asked.
- If the user asks for something destructive without being explicit about the target, ask which target rather than assuming.