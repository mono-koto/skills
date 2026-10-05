---
name: linear-issues
description: Use when the user references a Linear issue or asks to start work on one. Fetches the issue through an authenticated Linear MCP connection, reviews its context, prepares a Git workspace, and stops for scope confirmation.
---

Use this skill for a Linear issue identifier such as `ABC-1234`, a Linear issue URL, or a request to pick up a Linear issue. The goal is to establish context and prepare a workspace, not to begin implementation.

## Prerequisite

Use an authenticated Linear MCP connection. If one is unavailable, explain that it must be configured before private issue data can be read.

## Gather context

1. Discover available Linear MCP tools. Do not assume a workspace-specific tool namespace or tool name.
2. Fetch issue and capture its identifier, title, description, state, assignee, priority, relations, attachments, and supplied `gitBranchName`.
3. If there is not a clear single issue that matches the provided input, but if there is sufficient context to warrant a quick search, you may use the available tooling to try to find the relevant issue.
4. If multiple issues match, this is important and should be noted.
5. Fetch relevant comments. Review attached or embedded images when the client can access them. If an image is unavailable, say so rather than pretending its contents are known.

## Prepare the Git workspace

Use the issue's `gitBranchName` verbatim when it is supplied. If Linear does not supply one, inspect the repository's or user's branch conventions.

Use an isolated worktree unless the user asks to work in the current checkout. A worktree may already exist.

## Report

Report the issue title and state, branch and workspace path, a summary of the request, image findings, blockers, and open questions.

## Recommend

Recommend the next actions needed to address the issue, if any.

## Next steps

If explicitly running interactively, pause for user confirmation and discussion. Otherwise, proceed with addressing the linear issue.
