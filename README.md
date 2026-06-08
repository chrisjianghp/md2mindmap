# md2mindmap

A [Claude Code Skill](https://docs.claude.com/claude-code) that converts a **Markdown document** — local file or online URL — into a Mermaid mindmap rendered on a Lark/Feishu whiteboard inside a freshly-created Lark doc.

## What it does

Two input modes, same output (a new Lark doc with the mindmap):

- **Mode A — Local MD file**: Path like `~/path/to/README.md` or `./docs/intro.md`. Read with the `Read` tool.
- **Mode B — Online MD URL**: A URL pointing to raw markdown — GitHub raw links (`https://raw.githubusercontent.com/...`), GitHub blob pages (auto-converted to raw), or any HTTP(S) link returning markdown text. Fetched with `WebFetch`.

Before generating, it asks the user two questions:

1. **Language**: keep the source language (recommended for technical docs) or translate to Chinese
2. **Depth**: title skeleton only, or titles + short summaries

Then it parses the heading hierarchy (h1–h6), generates a Mermaid mindmap, creates a new Lark doc with a blank whiteboard, and writes the mindmap into the whiteboard.

> **Not in scope**: parsing Lark / Notion / Confluence / other rich-text systems. This skill handles plain Markdown text only.

## Installation

Clone into your Claude Code skills directory:

```bash
git clone https://github.com/chrisjianghp/md2mindmap.git ~/.claude/skills/md2mindmap
```

That's it. Claude Code will pick up the skill on the next session.

## Requirements

- [Claude Code](https://docs.claude.com/claude-code) with skills support
- [`lark-cli`](https://github.com/larksuite/lark-cli) — authenticated (`lark-cli config init`)
- Companion skills (usually bundled together): `lark-shared`, `lark-doc`, `lark-whiteboard`

## Usage

Just ask Claude things like:

- "把这个 md 文件变成思维导图：~/path/to/README.md"
- "帮我看下这个 md 的结构：~/Documents/foo.md"
- "可视化一下这个 GitHub 文档：https://raw.githubusercontent.com/owner/repo/main/README.md"
- "把这个 README 用飞书画板画出来"
- "Turn this markdown into a Lark whiteboard mindmap: <url-or-path>"

The skill triggers automatically based on your phrasing.

## Repository layout

```
md2mindmap/
├── SKILL.md            # The skill definition (what Claude reads)
├── .gitignore
├── LICENSE
└── README.md
```

## License

MIT
