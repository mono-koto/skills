---
name: write-well
description: Write clearly, concisely, and effectively. Use when the user explicitly asks for help with writing or editing, or when producing text that other people will read — commit messages, pull requests, code comments, committed docs, READMEs, release notes, issue replies. Do not use for ordinary replies to the user, or internal agent artifacts like notes, handoff plans, and specs.
---

# Write Well

## Scope

This skill is for text that other people will read (commits, PRs, comments, docs, release notes, issue replies) or for explicit user requests. Ordinary replies to the user and internal agent artifacts (handoff plans, specs, notes) do not need it.

## Subareas

This skill is a wrapper around multiple sub-areas.

[simplify](simplify.md) — revising existing content
[humanize](humanize.md) — remove LLM style. also includes a long, heavyweight guide for identifying AI writing signs.

## How to perform this skill

When asked to write from scratch, write clearly, naturally, and concisely to the best of your ability. Then run the output through the subskills to improve clarity, conciseness, and effectiveness.

When asked to edit existing content, run it through the subskills.

If the content is clearly human-written, you may skip the humanize subskill. If you are not sure and if in an interactive session, you can ask. If it is non-interactive, run the humanize subskill to test for AI writing signs.

Always run these tasks in subagents so that the long inputs and multiple iterations do not clutter primary agent context.

## Invocation

If invoked directly with arguments, treat the arguments as a hint for what to focus on for the writing or editing task. If no arguments are provided, consider all context to identify the target.

## Output

For ordinary editing requests, return the revised prose without explaining each change. For substantial rewrites or explicit feedback requests, briefly note the important changes and any unresolved factual or voice questions.
