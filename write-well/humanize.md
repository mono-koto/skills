# Humanize

If subagents are available, always perform this task in a subagent so that the long document of AI writing signs is not in the primary agent context.

This is a multi-step procedure that should be performed on target content.

1. Read the [Signs of AI Writing](./resources/wikipedia-signs-of-ai-writing.md) document. 
2. Identify and list all signs of AI writing in the target content.
3. For each of the issues identified, recommend an edit that would preserve quality and accuracy while removing the issue.
4. Update the target content with the recommended edits.

## Output

Return two things:

- The list of issues identified and recommended fixes
- The actual updated content