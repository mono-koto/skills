# Skills

A small collection of reusable skills for coding agents. Every skill follows
the open `SKILL.md` format and can be installed with
[`npx skills`](https://www.npmjs.com/package/skills).

## Included skills

| Skill | Use it for | Prerequisites and side effects |
| --- | --- | --- |
| `defuddle` | Extracting the useful content from a webpage or HTML file | Requires the `defuddle` CLI. Reads pages and local files. |
| `linear-tickets` | Reading a Linear issue and preparing a Git worktree | Requires an authenticated Linear MCP connection. Creates a local branch or worktree only after confirming the setup. |
| `worktrunk` | Managing and navigating git worktrees, and running parallel AI agents in worktrees | Requires the `wt` CLI. Creates, lists, removes, and merges worktrees. |
| `write-good-commit-messages` | Writing a Git commit message | Reads project conventions and Git history. |
| `write-naturally` | Revising prose for clarity, specificity, and an intended voice | Does not imitate a named person or try to evade AI detection. |
| `write-simply` | Cutting inflated or overly polished prose down to its point | Produces shorter, more literal writing without changing its meaning. |

## Install

Install one skill for selected agents at user scope:

```sh
npx skills add mono-koto/skills \
  --global \
  --agent codex claude-code gemini-cli \
  --skill write-naturally
```

Install every skill for selected agents:

```sh
npx skills add mono-koto/skills \
  --global \
  --agent codex claude-code gemini-cli \
  --skill '*' \
  --yes
```

By default, `npx skills` symlinks the installed skill into each agent's skill
directory. Use `--copy` to install independent copies instead.

## Develop locally

From a checkout of this repository, install a local skill with a live symlink:

```sh
npx skills add . \
  --global \
  --agent codex claude-code \
  --skill write-naturally
```

Inspect discovered skills:

```sh
npx skills add . --list
```

List installed global skills:

```sh
npx skills list --global
```

Remove a global skill:

```sh
npx skills remove --global \
  --agent codex claude-code \
  write-naturally \
  --yes
```

## License

MIT. See [LICENSE](./LICENSE).
