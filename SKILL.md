---
name: md2mindmap
description: >
  解析 Markdown 文档的结构（标题层级）并生成飞书思维导图画板。支持两种输入：
  (1) 本地 .md 文件路径；
  (2) 在线 Markdown URL（如 GitHub raw、原始 .md 链接、HTTP(S) 上托管的 markdown）。
  无论哪种来源，都支持三种输出格式（由用户选择）：
  - 飞书文档（含 Mermaid mindmap 画板）
  - 本地 HTML 文件（用 Markmap 渲染的交互式思维导图，支持缩放/折叠/搜索）
  - 同时输出两者
  当用户想要快速理解一份 Markdown 文档的结构、看骨架、生成思维导图、可视化层级时使用此 skill。
  即使用户没有明确说"思维导图"或"画板"，只要意图是理解 MD 文档的整体结构或层级，都应该触发此 skill。
  常见触发语句："帮我看下这个 md 的结构"、"这个文档讲了什么"、"把 md 变成思维导图"、
  "可视化这个 README"、"用画板展示文档结构"、"github 上这个 md 文件的思维导图"、
  "生成 HTML 思维导图"、"本地打开看看"。
---

# Markdown → 思维导图

将 Markdown 文档（本地或在线）的结构解析出来，生成思维导图，帮助用户一眼看清文档骨架。
支持两种输出格式（由用户选择）：飞书文档（含 Mermaid mindmap 画板）、本地 HTML 文件（Markmap 交互式）、或两者都要。

## 两种输入模式

两种模式的产出完全一致。区别只在于 MD 内容从哪里读：

### 模式 A：本地 MD 文件

用户提供本地 `.md` 文件路径（如 `~/Documents/foo.md`、`./README.md`），用 `Read` 工具读取后解析。

### 模式 B：在线 MD URL

用户提供一个指向 Markdown 文本的 URL，常见来源：
- GitHub raw：`https://raw.githubusercontent.com/owner/repo/branch/path/to/file.md`
- GitHub 普通文件页（`https://github.com/owner/repo/blob/...`）→ 自动转成 raw 后再下载
- 其他直接返回 markdown 文本的 HTTP(S) 链接

用 `WebFetch` 或 `curl` 拉取原始 markdown 文本后解析。

> **不在范围内**：解析飞书文档 / Notion / Confluence / 其他在线富文本系统的结构——这个 skill 只处理"纯 Markdown 文本"。

---

## 用户决策：只补问缺失的信息

读完 MD 内容、识别出标题结构之后，检查用户原始请求中是否已经明确指定了以下三个决策：Language、Depth、Format。**只对缺失的决策使用 `AskUserQuestion` 询问；用户已经明确指定的，不要重复询问。**

如果有多个缺失决策，把它们合并到一次 `AskUserQuestion` 调用中一起问，不要分多次问。这样既保留必要的确认，又避免用户已经说清楚时被重复打断。

### 决策 1：Language（语言选择）

文档源文件可能是任何语言（英文、中文、日文等）。模型有把英文文档自动翻译成中文的倾向，但用户的实际需求可能是保留原文（例如英文 README 保留英文术语更准确）。如果用户没有明确指定语言，才询问。

**视为已明确指定的表达示例**：
- `保持原文`、`不要翻译`、`用原语言`、`保留英文`、`英文原文` → 选择 `保持原文`
- `翻译成中文`、`中文输出`、`用中文`、`节点用中文` → 选择 `翻译为中文`

**缺失时询问参数**：
- header：`Language`（≤ 12 字）
- question：`思维导图节点的文字使用哪种语言？`
- 选项（固定两个，不要自行扩展）：
  1. `保持原文` — 描述："直接使用源文档中的文字，保留原始术语和措辞（推荐用于技术文档、专有名词、英文资料）"
  2. `翻译为中文` — 描述："将节点文字翻译为简体中文，便于中文读者快速理解"

用户选择或请求中指定"保持原文"时，节点文字必须严格使用源文档中出现的字符串（包括大小写、标点），不做任何翻译、改写或归纳；摘要可以截断但不能改写。

### 决策 2：Depth（内容深度）

思维导图可以只展示骨架（仅标题层级），也可以在每个标题节点下附带简短摘要。骨架更紧凑、一眼看清结构；带摘要信息更丰富，但节点更多、布局更密。如果用户没有明确指定内容深度，才询问。

**视为已明确指定的表达示例**：
- `仅标题`、`只要标题`、`标题骨架`、`只展示结构`、`不要摘要`、`不要总结` → 选择 `仅标题骨架`
- `加摘要`、`带总结`、`带简短说明`、`包含内容总结`、`标题加总结` → 选择 `标题加简短摘要`

**缺失时询问参数**：
- header：`Depth`（≤ 12 字）
- question：`思维导图包含哪些内容？`
- 选项（固定两个，不要自行扩展）：
  1. `仅标题骨架` — 描述："只展示标题层级（h1-h6），节点紧凑，一眼看清文档结构"
  2. `标题加简短摘要` — 描述："每个标题节点下附带一句话摘要（≤ 30 字），信息更丰富但节点更密集（推荐用于内容较少的文档）"

用户选择或请求中指定"仅标题骨架"时：**不要生成任何摘要节点**，只输出标题节点。

用户选择或请求中指定"标题加简短摘要"时：按下方「Step 1: 解析 MD 文档结构」中的摘要规则提取（首句、≤ 30 字）。

### 决策 3：Format（输出格式）

如果用户没有明确指定输出格式，才询问。

**视为已明确指定的表达示例**：
- `生成 HTML`、`到 html`、`本地网页`、`浏览器打开`、`Markmap`、`交互式网页` → 选择 `HTML 文件`
- `生成飞书文档`、`飞书画板`、`写到飞书`、`Lark doc`、`Feishu doc` → 选择 `飞书文档`
- `两者都要`、`同时生成 HTML 和飞书`、`html 和飞书都生成` → 选择 `两者都要`

**缺失时询问参数**：
- header：`Format`（≤ 12 字）
- question：`思维导图以什么格式输出？`
- 选项（固定三个，不要自行扩展）：
  1. `飞书文档` — 描述："创建飞书文档，思维导图嵌入文档中的 Mermaid 画板"
  2. `HTML 文件` — 描述："生成本地 HTML 文件，用 Markmap 渲染交互式思维导图，支持缩放/折叠/搜索，浏览器直接打开"
  3. `两者都要` — 描述："同时生成飞书文档和 HTML 文件"

### 默认值（仅当用户要求不要提问或上下文强烈暗示时使用）

常规情况下，缺失决策应询问。只有当用户明确说"不用问我/按默认来/直接生成"，或前文同一任务已经刚刚选择过相同偏好时，才使用默认值：
- Language：保持原文
- Depth：仅标题骨架
- Format：HTML 文件

### 询问时机

读完 MD 内容、识别出标题结构之后，**调用 `docs +create` 或生成 HTML 之前**。缺哪些问哪些；全部已明确时直接继续。

---

## Step 1: 获取并解析 MD 文档

### 1.1 获取 MD 文本

| 模式 | 怎么拿到 markdown 文本 |
|------|------------------------|
| A 本地路径 | `Read` 工具直接读 |
| B GitHub blob URL | 转成 raw URL（`github.com/.../blob/...` → `raw.githubusercontent.com/.../...`），再用 `WebFetch` 拉取 |
| B GitHub raw URL | 直接 `WebFetch` |
| B 其他 HTTP(S) MD 链接 | 直接 `WebFetch`；如果返回不是 markdown（HTML 渲染页），提示用户给 raw 链接 |

### 1.2 解析结构

提取：

- **标题层级**：h1 ~ h6，保留缩进关系
- **简短摘要**（仅当用户选了"标题加简短摘要"）：每个标题节的第一段非空文本，截取到第一个句号/问号/感叹号为止（最长 30 字）。如果没有正文内容，则不标注摘要

解析规则：
- `#` → h1，`##` → h2，以此类推
- 忽略纯 YAML front matter（开头 `---` 之间的内容）
- 忽略代码块内（` ``` ` 围起的部分）出现的"#"，那些不是标题
- 标题间的正文只取第一句作为摘要，不做深度内容提取
- 如果标题本身超过 20 字，在思维导图中截断到 20 字并加 `…`

---

## Step 2: 补齐用户决策

执行上面「用户决策：只补问缺失的信息」一节：
1. 先从用户原始请求中识别 Language、Depth、Format 是否已经明确指定
2. 对已明确的决策直接采用，不再询问
3. 对缺失的决策，用一次 `AskUserQuestion` 合并询问
4. 如果三个决策都已明确，则跳过 `AskUserQuestion`，直接继续

---

## Step 3: 根据用户选择的 Format 分支执行

### 分支 1：飞书文档（或两者都要）

走下方 Step 4–6（生成 Mermaid → 创建飞书文档 → 写入画板 → 交付）。

### 分支 2：HTML 文件（或两者都要）

走下方「HTML 输出（Markmap）」一节，生成本地 `.html` 文件并用 Markmap 渲染交互式思维导图。

如果用户选了「两者都要」，两个分支都执行。

---

## 飞书文档输出

### Step 4: 生成 Mermaid Mindmap

将解析结果转换为 Mermaid mindmap 语法。详见下方「Mermaid Mindmap 生成规范」。
根据用户在 Step 2 的选择决定节点文字的语言和是否包含摘要。

---

### Step 5: 创建飞书文档（含空白画板占位）

使用 `lark-cli docs +create` 创建文档：

```bash
lark-cli docs +create --api-version v2 --content '<title>📋 [文件名] 结构导图</title>
<callout emoji="🗺️"><p>本文档由 md2mindmap 自动生成，展示原始 Markdown 文档的结构层级。</p></callout>
<h2>结构思维导图</h2>
<whiteboard type="blank"></whiteboard>'
```

从返回的 JSON 中提取：
- `data.document.url` — 最终交付给用户
- `data.document.new_blocks[0].block_token`（type 为 `whiteboard` 的那条）— 画板 token，用于 Step 5 写入

> **文件名取法**：模式 A 取本地路径的 basename（去 `.md` 后缀）；模式 B 取 URL 最后一段（去 `.md` 后缀）；若 MD 首行有 h1，可优先用 h1 文本。

---

### Step 6: 写入思维导图到画板

`lark-cli` 原生支持 `--input_format mermaid`，无需额外工具。

```bash
cd /tmp
cat > mindmap.mmd << 'EOF'
<Mermaid mindmap 内容>
EOF

lark-cli whiteboard +update \
  --whiteboard-token <Step 4 拿到的 block_token> \
  --input_format mermaid \
  --source @./mindmap.mmd \
  --overwrite --as user
```

> **关键**：`--source` 必须是相对路径（如 `@./mindmap.mmd`）。绝对路径（如 `@/tmp/mindmap.mmd`）会报错。先 `cd` 到文件所在目录，再用 `@./filename` 形式引用。

---

### Step 7: 交付（飞书）

向用户报告飞书文档 URL、节点统计。详见下方「交付规范」。

---

## HTML 输出（Markmap）

当用户选择「HTML 文件」或「两者都要」时，生成一个自包含的 HTML 文件，用 Markmap 渲染交互式思维导图。

### 为什么用 Markmap 而不是 Mermaid

- Markmap 专为 Markdown → 思维导图设计，直接支持 `#` 标题语法
- 交互式：缩放、拖拽平移、节点折叠/展开
- 零依赖 CDN 加载（d3 + markmap-view + markmap-lib），无需本地安装
- 深色/浅色主题可切换

### 生成步骤

#### Step H1: 将解析结果转回 Markdown 文本

把 Step 1 解析出的标题层级（根据 Language 和 Depth 选择后的版本）拼回 Markdown 格式，作为 Markmap 的输入。

格式要求：
- 根节点作为 `#` 标题
- 二级标题用 `##`，三级用 `###`，以此类推
- 如果用户选了「标题加简短摘要」，摘要作为标题下的正文段落（不加 `#`）
- 在 Markdown 顶部加入 YAML front matter 配置：

```yaml
---
title: [文档标题] 结构导图
markmap:
  colorFreezeLevel: 2
  maxWidth: 320
  initialExpandLevel: 1
---
```

- `initialExpandLevel: 1`：默认只展开第一级分支，用户打开后先看到整体骨架，再点击展开细节
- 如果用户明确要求全部展开，或节点数很少（< 20），可以改为 `initialExpandLevel: -1`
- 如果用户明确要求默认展开到二级，则使用 `initialExpandLevel: 2`

#### Step H2: 生成 HTML 文件

创建一个 `.html` 文件，内嵌：
1. 深色主题 CSS（背景 `#1a1a2e`，浅色文字，提示条）
2. 一个 `<div class="markmap"><script type="text/template">...</script></div>` 容器
3. CDN 引用 `markmap-autoloader`（从 jsdelivr 加载稳定版），由 autoloader 自动渲染

**优先使用 `markmap-autoloader`，不要手写 `new Markmap(...)` 初始化。**手写初始化在不同 CDN 版本和本地 `file://` 打开时容易因为全局对象暴露差异导致 Chrome 空白或渲染不正常；autoloader 是官方推荐的最稳嵌入方式。

模板结构（每次生成时直接使用这个模板，替换 `__MARKDOWN_CONTENT__` 和 `__TITLE__`）：

```html
<!DOCTYPE html>
<html lang="zh-CN">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>__TITLE__ — Mindmap</title>
<style>
  * { box-sizing: border-box; }
  html, body {
    width: 100%; height: 100%; margin: 0; overflow: hidden;
    background: #1a1a2e;
    font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", sans-serif;
  }
  .header {
    position: fixed; top: 12px; left: 16px; z-index: 10;
    color: rgba(255,255,255,0.82);
    background: rgba(255,255,255,0.08);
    border: 1px solid rgba(255,255,255,0.12);
    border-radius: 10px; padding: 8px 12px;
    backdrop-filter: blur(10px); font-size: 14px;
  }
  .hint {
    position: fixed; bottom: 12px; left: 16px; z-index: 10;
    color: rgba(255,255,255,0.45); font-size: 12px;
    background: rgba(0,0,0,0.18);
    border-radius: 8px; padding: 6px 10px;
  }
  .markmap { width: 100vw; height: 100vh; }
  .markmap > svg { width: 100%; height: 100%; display: block; }
</style>
</head>
<body>
<div class="header">__TITLE__ — mindmap</div>
<div class="hint">Drag to pan · Scroll to zoom · Click nodes to fold/unfold</div>

<div class="markmap">
<script type="text/template">
__MARKDOWN_CONTENT__
</script>
</div>

<script src="https://cdn.jsdelivr.net/npm/markmap-autoloader@0.17"></script>
</body>
</html>
```

**替换规则**：
- `__TITLE__` → 文档标题（取源 MD 的 h1 或文件名）
- `__MARKDOWN_CONTENT__` → Step H1 拼好的 Markdown 文本
- 因为 Markdown 放在 `<script type="text/template">` 中，不需要像 JS 模板字符串那样转义 `$` 或反引号；但如果 Markdown 内容里出现 `</script>`，需要替换为 `<\/script>`，避免提前结束 script 标签

#### Step H3: 保存并打开

将 HTML 文件保存到 `/tmp/[文件名]-mindmap.html`，然后用 `open` 命令在浏览器中打开：

```bash
open /tmp/[文件名]-mindmap.html
```

#### Step H4: 交付（HTML）

告知用户：
- 文件路径
- 交互方式：拖拽平移、滚轮缩放、点击节点折叠/展开
- 文件完全自包含（除了 CDN 引用），可以复制到任何地方

示例：
```
✅ HTML 思维导图已生成！

📄 文件：/tmp/frontend-slides-mindmap.html

交互方式：
- 🖱 拖拽平移 | 滚轮缩放
- 右上角按钮：自适应 / 放大 / 缩小
- 点击节点折叠/展开子节点
- 零依赖，浏览器直接打开即可
```

---

## Mermaid Mindmap 生成规范

### 语法格式

```mermaid
mindmap
  root((文档标题))
    一级标题1
      摘要说明
      二级标题1-1
        摘要说明
      二级标题1-2
    一级标题2
      二级标题2-1
```

### 生成规则

1. `root((文档标题))` — 取文档标题或文件名作为根节点
2. **严格遵守源文档的标题层级，不得改写父子关系**：
   - `h1` 是 root 的直接子节点；`h2` 必须是其上方最近的 `h1` 的子节点；`h3` 必须是其上方最近的 `h2` 的子节点；依此类推
   - **常见错误**：把深层标题（如 h3）误当成顶层分支并列展示。生成前请逐项核对：每一个标题节点的缩进层级是否等于源文档中它的 `#` 数减一
   - **不要因为标题"看起来重要"就把它提到更高层级**，也不要因为某节内容多就把它"扁平化"以求布局好看
3. 摘要作为标题节点的子节点，用普通文本（不加括号），如果摘要为空则不添加
4. 节点文字中如果含有 Mermaid 特殊字符 `()[]{}:;&` 等，**用方括号 `[...]` 包裹**而不是引号：
   - ✅ `[Phase 0: Detect Mode]`
   - ❌ `"Phase 0: Detect Mode"` —— 引号节点后若再跟子节点会报 `Parse error` (Expecting 'SPACELINE', 'NL', 'EOF', got 'NODE_ID')
   - 方括号语法对叶子节点和有子节点的节点都安全
5. 不限制节点数量，全部展示
6. 特殊字符处理：HTML 实体（如 `&lt;`, `&gt;`, `&amp;`）解码后再写入 Mermaid

### 生成前自检（必做）

在写入画板前，对照源文档逐条核对：
- 每个标题节点的缩进层级是否 = 源文档中它的层级（h1=1, h2=2, h3=3, …）？
- 是否有 h3/h4 被错误地提升为 h1 同级？
- 子节点的归属是否正确（它确实在源文档中位于这个父标题之下）？

若有偏差，修正后再写入。

### 大文件处理

对于节点很多的文档，Mermaid 会自动处理布局。但为了让思维导图可读性更好：
- 摘要尽量简短（≤ 30 字），避免单个节点文字过多
- 如果某个 h1 下子节点特别多（> 15 个直接子节点），考虑将摘要省略以减少视觉密度

---

## 交付规范

根据用户选择的 Format，分别交付：

### 飞书文档

向用户报告：
1. 飞书文档 URL
2. 简要说明解析到的标题数量和层级深度

示例：
```
✅ 思维导图已生成！

📄 飞书文档：https://[your-domain].feishu.cn/docx/[document_id]

统计信息：
- 标题节点：23 个
- 最大层级深度：4 层
- 含摘要节点：18 个（或：仅标题骨架）

打开文档即可查看结构思维导图画板。
```

### HTML 文件

向用户报告：
1. 文件路径
2. 交互方式（拖拽、缩放、折叠、自适应按钮）
3. 文件可复制到任意位置，浏览器直接打开

### 两者都要

同时按上述两种格式分别报告。

---

## 依赖

- `lark-cli` — 飞书命令行工具（需已认证）。仅飞书文档输出需要
- `lark-doc` skill — 文档创建参考。仅飞书文档输出需要
- `lark-whiteboard` skill — 画板写入参考。仅飞书文档输出需要
- Markmap — CDN 加载（d3 + markmap-view + markmap-lib），无需本地安装。仅 HTML 输出需要

## 边界情况

- **用户没有指定输入**：询问用户要分析哪个 MD 文件或在线 MD URL
- **本地 MD 文件不存在或无法读取**：明确报错，提示用户检查路径
- **在线 MD URL 拉取失败 / 不是 markdown**：提示用户检查 URL；若是 GitHub blob 页面，建议用 raw URL
- **MD 文件没有任何标题**：提示用户该文件没有 Markdown 标题结构，无法生成有层级的思维导图（可询问是否仍要继续，把段落首句作为扁平节点）
- **lark-cli 认证失败**：提示用户运行 `lark-cli config init` 完成认证
- **画板写入失败**：检查 board_token 是否正确；确认 `--source` 使用了相对路径（如 `@./mindmap.mmd`）而非绝对路径；重试一次；仍失败则将 Mermaid 代码提供给用户，让其手动创建画板
- **HTML 中 Markdown 内容含 `</script>`**：在嵌入 `<script type="text/template">` 前替换为 `<\/script>`，避免提前结束 script 标签
- **CDN 加载失败**（离线环境）：告知用户 Markmap 需要首次加载时联网拉取 CDN 资源，之后浏览器缓存可离线使用
- **Chrome 打开 HTML 显示空白或不正常**：优先确认模板使用的是 `markmap-autoloader`，不要使用手写 `new Markmap(...)` 初始化；必要时换用 `open -a "Google Chrome" /tmp/xxx.html` 重新打开
