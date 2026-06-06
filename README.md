# md2mindmapinlarkdoc

A [Claude Code Skill](https://docs.claude.com/claude-code) that converts Markdown files or Lark/Feishu documents into Mermaid mindmaps rendered on a Lark whiteboard.

## What it does

Two modes:

- **Mode A — Local MD file → New Lark doc**: Read a local `.md` file, parse its heading structure, create a new Lark document containing a whiteboard mindmap.
- **Mode B — Lark doc URL → Update existing whiteboard**: Read an existing Lark document, parse its structure (headings, callouts, checkboxes, tables, etc.), and replace its whiteboard content with a mindmap.

Before generating, it asks the user two questions:
1. **Language**: keep the source language (recommended for technical docs) or translate to Chinese
2. **Depth**: title skeleton only, or titles + short summaries

## Installation

Clone into your Claude Code skills directory:

```bash
git clone https://github.com/<your-username>/md2mindmapinlarkdoc.git ~/.claude/skills/md2mindmapinlarkdoc
```

That's it. Claude Code will pick up the skill on the next session.

## Requirements

- [Claude Code](https://docs.claude.com/claude-code) with skills support
- [`lark-cli`](https://github.com/larksuite/lark-cli) — authenticated (`lark-cli config init`)
- Companion skills (usually bundled together): `lark-shared`, `lark-doc`, `lark-whiteboard`

## Usage

Just ask Claude things like:

- "把这个 md 文件变成思维导图：~/path/to/file.md"
- "Read this Lark doc and turn the whiteboard into a mindmap: https://xxx.feishu.cn/docx/..."
- "可视化一下这个文档的结构"
- "帮我看下这个 md 的结构"

The skill triggers automatically based on your phrasing.

## Repository layout

```
md2mindmapinlarkdoc/
├── SKILL.md            # The skill definition (what Claude reads)
├── .gitignore
├── LICENSE
└── README.md
```

## License

MIT
