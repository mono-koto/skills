---
name: worktrunk
description: Use when managing or navigating git worktrees with the Worktrunk CLI (`wt`), or running multiple AI agents in parallel worktrees. Covers `wt switch`, `wt list`, `wt remove`, `wt merge`, and agent handoffs. Requires the `wt` CLI.
---

# Worktrunk

Automate the git-worktree lifecycle and run AI agents in parallel worktrees.
Worktrunk's three core commands make worktrees as easy as branches; a fourth,
`wt merge`, collapses a finished branch back into the default branch without
leaving the feature worktree.

## When to use

- Creating, listing, switching between, or removing git worktrees
- Merging a feature branch back to the default branch and cleaning up
- Running several AI agents in parallel without them clobbering each other's
  working directory or index

## When not to use

- Plain branch switching in a single directory — branch switching mixes
  uncommitted changes between agents or blocks switching entirely
- Stacked-branch workflows — use git-machete or git-town instead (they can run
  inside individual worktrees)

## Quick reference

| Command | Purpose | Example |
| --- | --- | --- |
| `wt switch` | Switch to a worktree; create if needed | `wt switch --create feature` |
| `wt list` | List worktrees and their status | `wt list --full` |
| `wt remove` | Remove a worktree; delete branch if merged | `wt remove` |
| `wt merge` | Merge current branch into the target; clean up | `wt merge` |

## Core lifecycle

### `wt switch` — switch to a worktree; create if needed

```sh
wt switch main                       # switch to (or create) main's worktree
wt switch --create feature-auth      # new branch off the default branch
wt switch --create feature --base=@  # branch from current HEAD (stacked)
wt switch -                          # previous worktree
```

Special arguments work across commands: `-` is the previous worktree, `@` is
the current one. Run `wt switch --help` for the full list.

### `wt list` — list worktrees and their status

Shows each worktree's branch, path, and how it relates to the default branch:
`_` means same commit, `⊂` means content integrated. `--full` adds CI status,
PR links, and (with `[list] summary = true` and an LLM configured) a one-line
summary per branch.

```sh
wt list --full          # full status across worktrees
wt list --format=json   # structured output for scripts and statuslines
```

### `wt remove` — remove a worktree; delete branch if merged

Defaults to the current worktree. Refuses worktrees with uncommitted changes
mirroring `git worktree remove`; `--force` discards them. Deletes the branch
only when its content is already in the default branch; use `-D` to force-delete
an unmerged branch, or `--no-delete-branch` to keep the branch.

```sh
wt remove feature       # remove a specific worktree
wt remove @              # remove the current worktree
```

### `wt merge` — merge current branch into the target; clean up

Run from the feature worktree. Squashes the branch's changes (uncommitted plus
existing commits) into one commit with an LLM-generated message, rebases onto
the target, fast-forwards the target, then removes the worktree and the branch
if merged — like GitHub's merge button. No `cd` back to main.

```sh
wt merge                # squash + rebase + fast-forward default branch + cleanup
wt merge main           # merge into a specific target
```

## Agent-parallel patterns

Worktrees give each agent its own directory, index, and files, so parallel
agents never collide. Worktrunk adds handoffs and unified status on top.

### Agent handoffs

Spawn a worktree with an agent CLI running in the background, handing it a
prompt. The agent works in its own worktree while you continue elsewhere.

```sh
# tmux: new detached session
tmux new-session -d -s fix-auth-bug "wt switch --create fix-auth-bug -x claude -- \
  'The login session expires after 5 minutes. Find the session timeout config and extend it to 24 hours.'"

# Zellij: new pane in the current session
zellij run -- wt switch --create fix-auth-bug -x claude -- \
  'The login session expires after 5 minutes. Find the session timeout config and extend it to 24 hours.'
```

For OpenCode, replace `claude` with `'opencode run'`. Hooks run inside the
multiplexer session or pane.

### Activity tracking

`wt list` shows the status of every worktree at once — commits ahead/behind,
CI, PR links — so you can watch several parallel agents from one place.

### Why isolation beats branch switching

Branch switching uses one directory, so uncommitted changes from one agent mix
with the next agent's work or block switching. Worktrees sidestep this
entirely: each agent gets an independent working tree sharing the same `.git`.

## Common mistakes

- **Manual `git worktree add`/`remove`** — skip the manual lifecycle;
  `wt switch`/`wt remove` handle naming, setup hooks, and cleanup validation.
- **`cd`ing back to main before `wt merge`** — `wt merge` runs from the feature
  worktree and merges into the target; there's no need to switch directories.
- **Expecting `wt remove` to discard uncommitted work silently** — it refuses
  worktrees with uncommitted changes by default; pass `--force` to discard, or
  `git worktree lock` worktrees holding precious ignored data.
- **Forcing branch deletion by default** — branches are deleted only when
  already integrated into the default branch; use `-D` to force an unmerged
  branch, or `--no-delete-branch` to keep it.
- **Surprised by project-hook approval** — project hooks (`.config/wt.toml`)
  require approval on first run; user hooks don't. Approvals are saved and a
  changed command re-prompts. Pass `--yes` to skip prompts (CI, automation).

## Reference

Deeper automation — `wt config` (hooks, shell integration, aliases, approvals),
`wt step` (commit, squash, rebase, diff, copy-ignored, tether, for-each), and
`wt hook` (the worktree lifecycle) — is documented at `wt <command> --help`
and <https://worktrunk.dev>.
