# repo-explainer

一个 Claude Code Skill，将任意 GitHub 开源仓库转化为 **Neobrutalism 风格的交互式幻灯片**，用小白友好的方式讲解仓库做什么、怎么做、核心概念。

A Claude Code Skill that turns any GitHub repository into **Neobrutalism-style interactive slides** — explaining what it does, how it works, and its core concepts in a beginner-friendly way.

<p align="center">
  <img src="https://img.shields.io/badge/Claude_Code-Skill-blue?style=flat-square" alt="Claude Code Skill">
  <img src="https://img.shields.io/badge/reveal.js-v5-orange?style=flat-square" alt="reveal.js v5">
  <img src="https://img.shields.io/badge/style-Neobrutalism-FFDD00?style=flat-square&labelColor=000" alt="Neobrutalism">
  <img src="https://img.shields.io/badge/diagrams-pure_SVG-FF3D7F?style=flat-square&labelColor=000" alt="Pure SVG">
</p>

---

## 效果预览 / Preview

生成的幻灯片具有以下特点：

- **Neobrutalism 视觉语言** — 纯黑粗边框、硬投影（`8px 8px 0 #000`）、零圆角、荧光色块
- **纯 SVG 硬边图表** — 所有架构图、流程图用几何 helper 程序化绘制，raw & honest
- **四色高饱和体系** — 黄 `#FFDD00` / 蓝 `#2E5EFF` / 粉 `#FF3D7F` / 绿 `#00D26A`
- **全大写 display 字体** — Archivo Black 做标题，Space Grotesk 做正文，JetBrains Mono 做代码
- **单文件自包含** — 一个 HTML 文件，浏览器直接打开
- **交互式演示** — 基于 reveal.js，支持翻页、全屏、演讲者视图

The generated slides feature:

- **Neobrutalism visual language** — thick black borders, hard offset shadows (`8px 8px 0 #000`), zero radius, fluorescent color blocks
- **Pure SVG hard-edge diagrams** — all architecture & flow diagrams drawn programmatically with geometric helpers, raw & honest
- **Four-color high-saturation system** — yellow / blue / pink / green
- **All-caps display typography** — Archivo Black for headings, Space Grotesk for body, JetBrains Mono for code
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
| 🟡 Yellow | `#FFDD00` | 主强调 / 高亮块 / Primary accent, highlights |
| 🔵 Blue | `#2E5EFF` | 次强调 / 数据流 / Secondary, information blocks |
| 🌸 Pink | `#FF3D7F` | 警告 / 对比 / Warning, contrast, counter-examples |
| 🟢 Green | `#00D26A` | 正面结果 / Positive results, success state |
| ⚫ Black | `#000000` | 边框、箭头、投影 / Borders, arrows, shadows |
| 📄 Paper | `#FFFCF0` | 米白底色 / Background |

### 字体 / Fonts

| 字体 Font | 用途 Usage |
|---|---|
| Archivo Black | Display 标题（全大写） / Display headings (uppercase) |
| Space Grotesk | 正文 / Body text |
| JetBrains Mono | 代码 / 技术标签 / Code, technical labels |

### 组件 / Components

- **brutal-card** — 硬边卡片（黄/蓝/粉/绿/黑五色） / Hard-edged cards (5 color variants)
- **tilt-left / tilt-right** — 轻微旋转，强化 raw 感 / Slight rotation for raw feel
- **dialog-row** — 对话行（用户/反例/正例） / Dialog rows (user / bad / good)
- **grid-2/3/4** — 网格布局 / Grid layouts
- **tag** — 徽章标签 / Badge tags
- **step** — 大数字步骤块 / Numbered step blocks
- **marker** — 粉色标注 / Pink inline markers
- **stamp** — 倾斜黄色图章（CORE/NEW/HOT） / Tilted yellow stamp
- **big-num** — 超大数字展示 / Display-size numbers
- **blockquote** — 黑底白字大引用 / Black-background big statement

### SVG 图表 Helper / SVG Diagram Helpers

| 函数 Function | 用途 Usage |
|---|---|
| `drawBox(svg, x, y, w, h, label, opts)` | 硬边矩形 + 硬投影 / Hard-edge rect with offset shadow |
| `drawArrow(svg, x1, y1, x2, y2, opts)` | 粗黑直线箭头 / Thick black arrow |
| `drawCircle(svg, cx, cy, d, label, opts)` | 硬边实心圆节点 / Solid circular node |
| `drawText / drawMono` | Space Grotesk / JetBrains Mono 文字 / Typography |
| `drawChip(svg, x, y, text, opts)` | 带阴影的徽章 / Badge with shadow |

## 工作原理 / How It Works

```
GitHub URL → DeepWiki API → 结构化问答 → reveal.js + 纯 SVG → 自包含 HTML
GitHub URL → DeepWiki API → Structured Q&A → reveal.js + Pure SVG → Self-contained HTML
```

1. **获取结构** — 通过 DeepWiki MCP 读取仓库文档结构
2. **针对性提问** — 3-5 次 ask_question 获取关键信息（一句话总结、核心概念、架构、使用场景）
3. **生成幻灯片** — 组织为 reveal.js 幻灯片 + 纯 SVG 硬边图表
4. **输出文件** — 写入单个自包含 HTML 文件并自动打开

---

1. **Get structure** — Read repo documentation structure via DeepWiki MCP
2. **Targeted questions** — 3-5 ask_question calls to gather key info (one-liner, core concepts, architecture, use cases)
3. **Generate slides** — Organize into reveal.js slides + pure SVG hard-edge diagrams
4. **Output file** — Write a self-contained HTML file and auto-open in browser

## 技术栈 / Tech Stack

- [reveal.js v5](https://revealjs.com/) — 幻灯片框架 / Presentation framework
- **Pure SVG + custom helpers** — 硬边几何图表，无图表库依赖 / Hard-edge geometric diagrams, no chart library
- [Google Fonts](https://fonts.google.com/) — Archivo Black / Space Grotesk / JetBrains Mono
- [DeepWiki MCP](https://github.com/asyncfncom/deepwiki-mcp) — 仓库信息获取 / Repository information

## 设计原则 / Design Principles

Neobrutalism 不是装饰，是一套严格的视觉纪律：

1. **Raw, honest, 无装饰** — 不用渐变、半透明、阴影模糊
2. **硬边几何** — 所有边框 3-5px 纯黑，所有投影 `Xpx Xpx 0 #000`
3. **色块暴力对比** — 单图表最多 3 种高饱和色 + 黑白
4. **字号反差极端** — 标题 5-7em，正文 0.8em
5. **允许错位旋转** — 卡片可以 ±1.5deg 倾斜，强化"未打磨"质感
6. **一页一个暴击** — 不堆砌，一个 section 只讲一件事

Neobrutalism is not decoration — it's strict visual discipline:

1. **Raw, honest, undecorated** — no gradients, no translucency, no blurred shadows
2. **Hard-edged geometry** — all borders 3-5px pure black, all shadows `Xpx Xpx 0 #000`
3. **Violent color-block contrast** — max 3 saturated colors + black/white per diagram
4. **Extreme type scale** — headings 5-7em, body 0.8em
5. **Misalignment encouraged** — cards can tilt ±1.5deg for raw feel
6. **One punch per slide** — don't cram; each section makes one point

## License

MIT
