# md2mindmap

A [Claude Code Skill](https://docs.claude.com/claude-code) that converts a **Markdown document** — local file or online URL — into an interactive mindmap.

It can output:

- **HTML Markmap** — a local interactive HTML file with zoom/pan/fold and PDF export
- **Lark/Feishu document** — a freshly-created Lark doc containing a Mermaid mindmap whiteboard
- **Both**

## What it does

Two input modes:

- **Mode A — Local MD file**: Path like `~/path/to/README.md` or `./docs/intro.md`. Read with the `Read` tool.
- **Mode B — Online MD URL**: A URL pointing to raw markdown — GitHub raw links (`https://raw.githubusercontent.com/...`), GitHub blob pages (auto-converted to raw), or any HTTP(S) link returning markdown text. Fetched with `WebFetch`.

Before generating, the skill resolves three preferences:

1. **Language**: keep the source language or translate to Chinese
2. **Depth**: title skeleton only, or titles + short summaries
3. **Format**: HTML file, Lark/Feishu document, or both

If the user already specified any of these in the prompt (for example "生成 HTML", "仅标题", "保持原文"), the skill respects that and only asks about missing choices.

## HTML output

The HTML output uses [Markmap](https://markmap.js.org/) via `markmap-autoloader`.

Features:

- Interactive zoom, pan, fold/unfold
- Dark theme with bright readable text
- Default first-level expansion (`initialExpandLevel: 1`) for a clean overview
- PDF export button
  - Uses `html2canvas` to snapshot the rendered mindmap into a PNG
  - Opens the browser print dialog; choose **Save as PDF**
  - This avoids Chrome's SVG `foreignObject` / Chinese font issues when printing

## Lark/Feishu output

The Lark/Feishu output creates a new Lark document, inserts a blank whiteboard, then writes a Mermaid mindmap into that whiteboard.

This output requires `lark-cli` authentication.

## Not in scope

This skill handles **plain Markdown text only**. It does not parse Lark / Notion / Confluence / other rich-text systems directly.

## Installation

Clone into your Claude Code skills directory:

```bash
git clone https://github.com/chrisjianghp/md2mindmap.git ~/.claude/skills/md2mindmap
```

That's it. Claude Code will pick up the skill on the next session.

## Requirements

- [Claude Code](https://docs.claude.com/claude-code) with skills support
- For HTML output: internet access to load CDN resources (`markmap-autoloader`, `html2canvas`)
- For Lark/Feishu output:
  - [`lark-cli`](https://github.com/larksuite/lark-cli) — authenticated (`lark-cli config init`)
  - Companion skills (usually bundled together): `lark-shared`, `lark-doc`, `lark-whiteboard`

## Usage

Just ask Claude things like:

- "把这个 md 文件变成思维导图：~/path/to/README.md"
- "帮我看下这个 md 的结构：~/Documents/foo.md"
- "可视化一下这个 GitHub 文档：https://raw.githubusercontent.com/owner/repo/main/README.md"
- "解析 https://github.com/owner/repo/blob/main/SKILL.md 到 HTML"
- "保持原文，仅标题，生成 HTML 思维导图：~/docs/architecture.md"
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
