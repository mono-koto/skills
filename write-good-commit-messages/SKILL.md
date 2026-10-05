---
name: write-good-commit-messages
description: Use to write a Git commit message. Defaults to a concise one-line subject and defers to documented or established project conventions.
---

A commit message should let someone reading `git log` understand what changed and why without opening the diff. 

Your goal is to optimize for reducing the time and thinking a human reader needs to understand the change. That's it.

## Check the project style first

Before writing a message, look for a documented convention in `CONTRIBUTING.md`, `COMMITS.md`, `.gitmessage`, the README, or repository instructions. Then inspect recent commit subjects:

```sh
git log -20 --pretty=format:'%s'
```

Project style wins. Match a consistent convention such as Conventional Commits or a ticket prefix when one exists.

## Default to one line

Most commits need only a subject. Use a body only when it adds a non-obvious
reason, constraint, or follow-up that the subject cannot carry.

Subject rules:

- Aim for about 50 characters and keep it under 72.
- Use imperative mood: `add foo`, not `added foo` or `adds foo`.
- Do not add a trailing period.
- Describe the change, not the files touched.
- Avoid marketing language such as `comprehensive`, `robust`, or `cleanly`.

Examples:

```
bump node to 22.11
```

```
drop unused legacy cookie path
```

Use a body when it carries real context:

```
switch session store to redis

- in-memory store dropped sessions on every deploy
- reuse the existing cache connection pool
```

## Common mistakes

- Long single paragraph body that no one will read.
- Adding a body that only repeats the subject.
- Narrating the diff one file at a time.
- Mixing moods such as `add`, `added`, and `adds`.
- Ignoring an obvious repository convention.
- Adding co-author or attribution lines unless the project explicitly requires them.
