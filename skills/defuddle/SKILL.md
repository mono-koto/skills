---
name: defuddle
description: Use when a webpage's content should be retrieved and extracted into clean markdown, such as "grab the page", "extract this URL", "read this webpage", or "pull the text from". Uses the defuddle CLI and also handles local HTML files and HTML on stdin.
allowed-tools: Bash(defuddle *)
---

# Defuddle

Extract the main content from web pages or local HTML as clean markdown, HTML,
or JSON. Navigation, sidebars, comments, and footers are stripped automatically.

This skill assumes that `defuddle` is installed. Check with `command -v
defuddle`; if it is unavailable, install it with `npm install -g defuddle`.

The source can be a URL, a local HTML file path, or `-` to read HTML from stdin.

## Quick start

```sh
# URL to markdown on stdout
defuddle parse "https://example.com/article" -m

# Markdown with YAML frontmatter
defuddle parse "https://example.com/article" -m -f

# Save markdown to a file
defuddle parse "https://example.com/article" -m -o article.md

# JSON with metadata, HTML, and markdown
defuddle parse "https://example.com/article" -j

# One metadata property
defuddle parse "https://example.com/article" -p title

# Local HTML or stdin
defuddle parse page.html -m
curl -s https://example.com/article | defuddle parse - -m
```

## Common options

| Option | Purpose |
| --- | --- |
| `-m`, `--markdown` | Return markdown. |
| `-f`, `--frontmatter` | Add title, author, source, domain, language, published date, and word count. |
| `-j`, `--json` | Return metadata plus HTML and markdown in JSON. |
| `-p`, `--property <name>` | Return a single property, such as `title`, `author`, or `published`. |
| `-o`, `--output <file>` | Write output to a file. |
| `-l`, `--lang <code>` | Set the preferred language. |
| `-u`, `--user-agent <string>` | Supply a user agent for sites that reject the default request. |
| `--debug` | Emit verbose output and preserve more HTML structure. |

`-m` returns only markdown. Use `-m -f` for markdown with metadata, or `-j`
when a program needs both metadata and content.

## Limits and recovery

- Defuddle fetches server-rendered HTML. For JavaScript-rendered pages whose
  content is absent from the response, use a browser-based tool instead.
- Do not combine `-m` and `-j`; use one output format at a time.
- For a 403 response, retry with an ordinary browser user agent:

```sh
defuddle parse "https://example.com/article" -m \
  -u "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/124.0 Safari/537.36"
```

- Confirm the output contains the expected title and body before treating an
  extraction as complete.
