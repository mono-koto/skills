---
name: worktrunk
description: Use when managing git worktrees with Worktrunk (`wt`).
---

# Worktrunk

Use worktrunk to manage git worktrees, including templatized .env generation, worktree management, and llm summaries. see `wt --help` and `wt <command> --help` for more info.

## Commands

```bash
wt switch <branch>              # switch to or create a worktree
wt switch --create <branch>     # create a worktree for a new branch
wt list                         # list worktrees
wt list --format=json           # list worktrees as JSON
wt remove <branch>              # remove a worktree
```

## Non-interactive shells

```bash
worktree="$(wt switch foo/bar-baz --no-cd --format=json | jq -r .path)"
cd "$worktree"
# run commands here
```

For more information, run `wt --help` or `wt <command> --help`.
