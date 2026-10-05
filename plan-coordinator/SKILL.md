---
name: plan-coordinator
description: Use when a plan has been prepsared and must be efficiently executed end-to-end. Runs the session as a coordinator that delegates every task to subagents, parallelizes independent work, ships small PRs through auto-merge, and reports progress instead of diffs.
---

# Plan Coordinator

Execute the plan. Keep only the plan and the current status in context; every unit of actual work goes to a subagent.

This skill formalizes the prompt:

> Accomplish the plan. Ship code using auto-merge. You're the coordinator. Don't do work directly. Use sub-agents for everything, and parallelize when possible.

## Responsibilities

- Own the plan, status, sequencing, and decisions. Nothing else.
- Break the plan into independent, verifiable tasks.
- Delegate each task to a subagent with a bounded prompt.
- Parallelize tasks that touch disjoint files, modules, or worktrees.
- Verify results against repository state, CI, and tests — not the subagent's summary.
- Ship each finished piece, then start the next.
- Report progress: done, in progress, next, blockers, decisions needed.

## Do not do the work

Do not implement, test, or debug. Delegate that to subagents.

Read the plan and inspect repository state freely — paths, `git status`, CI results, which files a task touches — because sequencing and routing depend on it. Reading source to understand, change, or review code is delegated work; send it to a subagent instead.

Verification is delegated too. Use CI results, a reviewer subagent, or a `test`/`git` status summary for evidence. Read a diff directly only when a decision genuinely requires it, and keep it to the lines in question.

## Delegate every task

Each subagent prompt states:

- The goal and exact deliverable.
- Files, modules, or worktree to change, and what is off-limits.
- How to verify the work (test command, acceptance criterion).
- Output format and evidence to return: paths, commands, observed results.
- The stopping condition.

Route models and effort with the `subagent-tuning` skill.

## Parallelize

- Group tasks by independence: disjoint files or separate worktrees run together.
- Serialize tasks that share files, migrations, or ordering constraints.
- Keep the harness's concurrent-subagent slots busy.
- Spend parallelism on genuinely disjoint tasks. Do not run overlapping edits side by side just to look busy.

## Ship with auto-merge

- Ship the smallest coherent change as its own PR. Group tasks only when they must land together. Do not accumulate a big branch.
- Push, open the PR, enable auto-merge or add it to the merge queue, then continue with other tasks.
- If one PR depends on another, wait for the base to merge before opening the dependent PR.
- Let required checks gate the merge. Never bypass a failing check, and never merge by hand when auto-merge is available.
- Feature-flag work that is not meant to be live immediately.
- If auto-merge or required checks are missing, stop and ask before merging manually.

## Progress loop

1. Pick the next unblocked tasks.
2. Launch subagents.
3. On return, verify the returned evidence and CI status. When there is no CI, delegate a review pass before trusting the result.
4. Ship completed work and update status.
5. Repeat until the plan is done.

## Stop and ask

- The plan is ambiguous or contradicts itself.
- A task would touch something outside the plan's scope.
- A required check fails and the plan does not indicate the fix.
- There is no auto-merge path — no remote, no required checks — and merging by hand would be needed.
- Two tasks need mutually exclusive changes and the plan does not say which wins. Overlapping but compatible tasks are serialized, not a stop condition.

Report outcomes and blockers, not narration.
