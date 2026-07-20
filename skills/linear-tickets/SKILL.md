---
name: linear-tickets
description: Use when the user references a Linear issue or asks to start work on one. Fetches the issue through an authenticated Linear MCP connection, reviews its context, prepares a Git workspace, and stops for scope confirmation.
---

# Start a Linear Ticket

Use this skill for a Linear issue identifier such as `ABC-1234`, a Linear issue
URL, or a request to pick up a Linear ticket. The goal is to establish context
and prepare a workspace, not to begin implementation.

## Prerequisite

Use an authenticated Linear MCP connection. If one is unavailable, explain that
it must be configured before private issue data can be read. The standard remote
endpoint is `https://mcp.linear.app/mcp`; use the agent's MCP configuration and
authentication flow rather than guessing issue details from a URL.

For Codex, the documented setup is:

```sh
codex mcp add linear --url https://mcp.linear.app/mcp
codex mcp login linear
```

## Gather context

1. Discover the available Linear MCP tools. Do not assume a workspace-specific
   tool namespace or tool name.
2. Fetch the issue and capture its identifier, title, description, state,
   assignee, priority, relations, attachments, and supplied `gitBranchName`.
3. Fetch relevant comments. Review attached or embedded images when the client
   can access them. If an image is unavailable, say so rather than pretending its
   contents are known.
4. Call out blockers, duplicates, related issues, and open questions before
   selecting a branch or starting work.

## Prepare the Git workspace

Use the issue's `gitBranchName` verbatim when it is supplied. If Linear does not
supply one, inspect the repository's documented branch convention and ask the
user before choosing a name when the convention is unclear.

Default to an isolated Git worktree unless the user asks to work in the current
checkout:

```sh
git worktree add <path> <branch>
```

If the branch already exists locally or remotely, attach the worktree to that
branch. If it does not exist, create it from the repository's intended base
branch. Use only native Git commands; do not depend on another skill to set up
the worktree.

## Report and stop

Report the issue title and state, branch and workspace path, a short summary of
the request, image findings, blockers, and open questions. Stop there until the
user confirms the scope or asks to continue.
