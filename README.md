# repo-explainer

一个 Claude Code Skill，将任意 GitHub 开源仓库转化为**手绘风格的交互式幻灯片**，用小白友好的方式讲解仓库做什么、怎么做、核心概念。

A Claude Code Skill that turns any GitHub repository into **hand-drawn interactive slides**, explaining what it does, how it works, and its core concepts in a beginner-friendly way.

<p align="center">
  <img src="https://img.shields.io/badge/Claude_Code-Skill-blue?style=flat-square" alt="Claude Code Skill">
  <img src="https://img.shields.io/badge/reveal.js-v5-orange?style=flat-square" alt="reveal.js v5">
  <img src="https://img.shields.io/badge/rough.js-v4.6.6-green?style=flat-square" alt="rough.js v4.6.6">
</p>

---

## 效果预览 / Preview

生成的幻灯片具有以下特点：

- **Excalidraw 手绘白板风格** — 白底奶油纸色 + Virgil 手写字体
- **rough.js 手绘图表** — 所有架构图、流程图均由程序化手绘生成
- **单文件自包含** — 一个 HTML 文件，浏览器直接打开
- **交互式演示** — 基于 reveal.js，支持翻页、全屏、演讲者视图

The generated slides feature:

- **Excalidraw hand-drawn whiteboard style** — cream paper background + Virgil handwritten font
- **rough.js hand-drawn diagrams** — all architecture and flow diagrams are programmatically sketched
- **Self-contained single file** — one HTML file, open directly in browser
- **Interactive presentation** — powered by reveal.js with navigation, fullscreen, and speaker view

## 安装 / Installation

将 `SKILL.md` 放入你的 Claude Code skills 目录：

Place `SKILL.md` into your Claude Code skills directory:

```bash
# 创建 skill 目录 / Create skill directory
mkdir -p ~/.claude/skills/repo-explainer

# 复制文件 / Copy file
cp SKILL.md ~/.claude/skills/repo-explainer/SKILL.md
```

### 前置要求 / Prerequisites

- [Claude Code](https://docs.anthropic.com/en/docs/claude-code) CLI
- [DeepWiki MCP Server](https://github.com/asyncfncom/deepwiki-mcp) — 用于获取仓库信息 / for fetching repository information

## 使用方法 / Usage

在 Claude Code 中使用以下任一方式触发：

Trigger in Claude Code with any of the following:

```
/repo-explainer https://github.com/karpathy/nanochat

/repo-explainer owner/repo

讲解仓库 https://github.com/NousResearch/hermes-agent
```

### 参数 / Parameters

| 参数 Parameter | 说明 Description | 默认值 Default |
|---|---|---|
| 语言 Language | 中文 / English | 中文 Chinese |
| 深度 Depth | 快速 Quick (~8p) / 标准 Standard (~15p) / 深入 Deep (~25p) | 标准 Standard |
| 聚焦 Focus | 指定模块 Specific module | 全部 All |

### 示例 / Examples

```
# 标准讲解 / Standard explanation
/repo-explainer https://github.com/karpathy/nanochat

# 英文输出 / English output
/repo-explainer https://github.com/karpathy/nanochat 英文

# 快速概览 / Quick overview
/repo-explainer https://github.com/karpathy/nanochat 快速

# 聚焦模块 / Focus on module
/repo-explainer https://github.com/karpathy/nanochat 只讲训练部分
```

## 设计系统 / Design System

### 色彩 / Colors

| 角色 Role | 颜色 Color | 用途 Usage |
|---|---|---|
| 🔵 主色 Primary | `#1971c2` | 核心模块、主流程 / Core modules, main flow |
| 🔴 警告 Warning | `#e03131` | 对比、反面例子 / Contrast, negative examples |
| 🟢 成功 Success | `#2f9e44` | 正面结果 / Positive results |
| ⚫ 墨色 Ink | `#1e1e1e` | 边框、连线 / Borders, connections |

### 字体 / Fonts

| 字体 Font | 用途 Usage |
|---|---|
| Virgil | 标题、手写标注 / Titles, hand annotations |
| Caveat | 手写 fallback |
| Inter | 正文 / Body text |
| JetBrains Mono | 代码 / Code |

### 组件 / Components

- **sketch-card** — 手绘边框卡片 / Hand-drawn border cards
- **chat-bubble** — 对话气泡（对比展示） / Chat bubbles (comparison)
- **grid-2/3** — 网格布局 / Grid layouts
- **tag** — 标签胶囊 / Tag pills
- **step** — 步骤条 / Step indicators
- **handnote** — 手写批注 / Hand-drawn annotations
- **blockquote** — 引用块 / Quote blocks

## 工作原理 / How It Works

```
GitHub URL → DeepWiki API → 结构化问答 → reveal.js + rough.js → 自包含 HTML
GitHub URL → DeepWiki API → Structured Q&A → reveal.js + rough.js → Self-contained HTML
```

1. **获取结构** — 通过 DeepWiki MCP 读取仓库文档结构
2. **针对性提问** — 3-5 次 ask_question 获取关键信息
3. **生成幻灯片** — 组织为 reveal.js 幻灯片 + rough.js 手绘图表
4. **输出文件** — 写入单个自包含 HTML 文件并自动打开

---

1. **Get structure** — Read repo documentation structure via DeepWiki MCP
2. **Targeted questions** — 3-5 ask_question calls to gather key information
3. **Generate slides** — Organize into reveal.js slides + rough.js hand-drawn diagrams
4. **Output file** — Write a self-contained HTML file and auto-open in browser

## 技术栈 / Tech Stack

- [reveal.js v5](https://revealjs.com/) — 幻灯片框架 / Presentation framework
- [rough.js v4.6.6](https://roughjs.com/) — 手绘图形库 / Hand-drawn graphics
- [Excalidraw Fonts](https://github.com/excalidraw/excalidraw) — Virgil 手写字体 / Handwritten font
- [DeepWiki MCP](https://github.com/asyncfncom/deepwiki-mcp) — 仓库信息获取 / Repository information

## License

MIT
