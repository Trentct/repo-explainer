---
name: repo-explainer
description: 读取 GitHub 开源仓库（通过 DeepWiki MCP），生成独立 HTML 幻灯片（手绘风格 + rough.js），用小白友好的方式讲解仓库做什么、怎么做、核心概念。触发词：讲解仓库、解读仓库、仓库幻灯片、explain repo、repo slides。用户提供 GitHub URL 或 owner/repo 格式即可。
---

# Repo Explainer — 用幻灯片讲解开源仓库

将任意 GitHub 公开仓库转化为小白友好的**独立 HTML 幻灯片**（基于 reveal.js CDN + rough.js 手绘风格），浏览器直接打开即可演示。

## 触发条件

用户提供 GitHub 仓库地址（URL 或 owner/repo 格式）并要求讲解、解读、做幻灯片时触发。

## 输入解析

从用户输入中提取：
- `owner/repo`：从 URL `github.com/{owner}/{repo}` 或直接文本中提取
- 可选参数：
  - **语言**：默认中文，用户可指定英文
  - **深度**：`快速`（~8 页）/ `标准`（~15 页，默认）/ `深入`（~25 页）
  - **聚焦**：用户可指定只讲某个模块，如"只讲训练部分"

## 执行流程

### Step 1: 获取仓库结构

调用 `mcp__deepwiki__read_wiki_structure` 获取 wiki 目录。

```
read_wiki_structure(repoName: "owner/repo")
```

分析返回的章节列表，确定仓库的核心模块。

### Step 2: 针对性提问（3-5 次 ask_question）

根据幻灯片需要的内容，向 DeepWiki 提问。**不要调用 read_wiki_contents**（返回全量内容太大）。

必问问题（根据语言设置用中文或英文提问）：

1. **一句话总结**："用一句话向完全不懂编程的人解释这个项目是做什么的"
2. **核心概念**："这个项目的 3-5 个核心概念是什么？每个用一句话解释"
3. **架构总览**："描述这个项目的整体架构，包含核心模块和数据流向"
4. **使用场景**："这个项目最常见的 3 个使用场景是什么？"

可选问题（深入模式或用户指定聚焦时）：

5. **关键代码**："展示这个项目最核心的一段代码（<30 行），并逐行注释"
6. **对比**："这个项目和同类项目（如 xxx）相比有什么独特之处？"
7. **上手指南**："一个新手要开始使用这个项目，最少需要哪几步？"

### Step 3: 生成 HTML 幻灯片

将收集到的信息组织成**单个自包含 HTML 文件**，使用 reveal.js（CDN）+ rough.js（CDN）手绘风格。

#### 设计风格：Excalidraw 手绘白板

- **白底奶油纸色**，不是暗色主题
- **Virgil 手写字体**做标题和批注
- **rough.js** 绘制所有架构图、流程图、概念图（不用 mermaid）
- **sketch-card** 卡片组件带手绘边框和 box-shadow
- **三色体系**：蓝 `#1971c2`（主色/强调）、红 `#e03131`（警告/对比）、绿 `#2f9e44`（正面/成功）

#### HTML 模板骨架

```html
<!DOCTYPE html>
<html lang="zh-CN">
<head>
  <meta charset="utf-8">
  <title>{repo-name} 讲解</title>
  <link rel="preconnect" href="https://fonts.googleapis.com">
  <link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
  <link href="https://fonts.googleapis.com/css2?family=Caveat:wght@400;500;600;700&family=Inter:wght@300;400;500;600&family=JetBrains+Mono:wght@400;500&display=swap" rel="stylesheet">
  <link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/reveal.js@5/dist/reveal.css">
  <link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/reveal.js@5/dist/theme/white.css">
  <link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/reveal.js@5/plugin/highlight/github.css">
  <style>
    @font-face {
      font-family: 'Virgil';
      src: url('https://raw.githubusercontent.com/nicolo-ribaudo/excalidraw-fonts/7-woff2/Virgil-Regular.woff2') format('woff2');
      font-display: swap;
    }

    :root {
      --blue: #1971c2;
      --blue-light: #d0ebff;
      --blue-bg: #e7f5ff;
      --red: #e03131;
      --red-light: #ffe3e3;
      --green: #2f9e44;
      --green-light: #d3f9d8;
      --ink: #1e1e1e;
      --ink-dim: #495057;
      --ink-muted: #868e96;
      --paper: #FFFCF0;
      --paper-dim: #f8f5ec;
      --hand: 'Virgil', 'Caveat', cursive;
      --body: 'Inter', 'PingFang SC', 'Microsoft YaHei', sans-serif;
      --mono: 'JetBrains Mono', monospace;
    }

    html, body { background: var(--paper); }
    .reveal-viewport { background: var(--paper); }
    .reveal { font-family: var(--body); color: var(--ink); font-size: 22px; }
    .reveal section { text-align: left; padding: 10px 30px; }

    /* Typography */
    .reveal h1, .reveal h2 { font-family: var(--hand); text-transform: none; letter-spacing: 0; color: var(--ink); }
    .reveal h1 { font-size: 4.5em; font-weight: 400; line-height: 1.1; margin-bottom: 0.2em; }
    .reveal h2 { font-size: 2.2em; font-weight: 400; margin-bottom: 0.6em; padding-bottom: 0.15em; border-bottom: 3px solid var(--ink); display: inline-block; }
    .reveal h3 { font-family: var(--hand); font-size: 1.3em; font-weight: 400; color: var(--blue); text-transform: none; margin-bottom: 0.4em; }
    .reveal p { font-size: 0.82em; line-height: 1.65; color: var(--ink-dim); }
    .reveal ul, .reveal ol { font-size: 0.8em; line-height: 1.65; display: block; margin-left: 1.2em; color: var(--ink-dim); }
    .reveal li { margin-bottom: 0.35em; }
    .reveal strong { color: var(--blue); font-weight: 600; }
    .reveal em { color: var(--ink-muted); }
    .reveal a { color: var(--blue); text-decoration: underline; text-decoration-style: wavy; text-underline-offset: 3px; }

    .reveal code { font-family: var(--mono); color: var(--red); background: var(--red-light); padding: 0.1em 0.4em; border-radius: 4px; font-size: 0.85em; }
    .reveal pre { background: #fafafa; border: 2px solid var(--ink); border-radius: 4px; padding: 0.8em 1em; font-size: 0.45em; box-shadow: 4px 4px 0 var(--ink); position: relative; }
    .reveal pre::before { content: attr(data-label); position: absolute; top: -12px; left: 12px; background: var(--paper); padding: 0 8px; font-family: var(--hand); font-size: 1.4em; color: var(--blue); }
    .reveal pre code { font-family: var(--mono); color: var(--ink); background: none; padding: 0; line-height: 1.7; }

    .reveal blockquote { border: none; border-left: 4px solid var(--blue); background: var(--blue-bg); padding: 0.8em 1.2em; border-radius: 0 4px 4px 0; font-family: var(--hand); font-size: 1.05em; color: var(--ink); box-shadow: none; font-style: normal; }

    /* Hand-drawn cards */
    .sketch-card { border: 2px solid var(--ink); border-radius: 3px; padding: 1em 1.2em; position: relative; background: white; box-shadow: 3px 3px 0 var(--ink); }
    .sketch-card.blue { border-color: var(--blue); box-shadow: 3px 3px 0 var(--blue); }
    .sketch-card.red { border-color: var(--red); box-shadow: 3px 3px 0 var(--red); }
    .sketch-card.green { border-color: var(--green); box-shadow: 3px 3px 0 var(--green); }

    /* Chat bubbles */
    .chat-row { display: flex; margin-bottom: 0.35em; }
    .chat-row.right { justify-content: flex-end; }
    .chat-bubble { padding: 0.4em 0.8em; border-radius: 12px; font-size: 0.72em; max-width: 85%; line-height: 1.5; border: 2px solid; }
    .chat-bubble.user { background: var(--blue-light); border-color: var(--blue); color: var(--ink); }
    .chat-bubble.ai-bad { background: var(--red-light); border-color: var(--red); color: var(--red); }
    .chat-bubble.ai-good { background: var(--green-light); border-color: var(--green); color: var(--green); }

    /* Grids */
    .grid-2 { display: grid; grid-template-columns: 1fr 1fr; gap: 1.2em; }
    .grid-3 { display: grid; grid-template-columns: 1fr 1fr 1fr; gap: 1em; }

    /* Tags */
    .tag { display: inline-block; padding: 0.2em 0.6em; border: 2px solid var(--blue); border-radius: 999px; font-family: var(--hand); font-size: 0.75em; color: var(--blue); margin: 0.15em 0.2em; }
    .tag.red { border-color: var(--red); color: var(--red); }
    .tag.green { border-color: var(--green); color: var(--green); }

    /* Step */
    .step { display: flex; align-items: flex-start; gap: 0.8em; margin-bottom: 0.5em; }
    .step-num { width: 32px; height: 32px; border: 2px solid var(--blue); border-radius: 50%; display: flex; align-items: center; justify-content: center; font-family: var(--hand); font-size: 1.1em; font-weight: 700; color: var(--blue); flex-shrink: 0; }
    .step-text { font-size: 0.78em; color: var(--ink-dim); line-height: 1.5; }
    .step-text strong { color: var(--ink); }

    /* Diagram containers */
    .diagram-container { display: flex; justify-content: center; margin: 0.5em 0; }
    .diagram-container svg { overflow: visible; }

    /* Table */
    .reveal table { width: 100%; border-collapse: collapse; font-size: 0.68em; border: 2px solid var(--ink); box-shadow: 3px 3px 0 var(--ink); }
    .reveal th { background: var(--blue-bg); border: 2px solid var(--ink); padding: 0.6em 0.8em; font-weight: 600; text-align: left; font-family: var(--hand); font-size: 1.15em; }
    .reveal td { border: 1px solid #dee2e6; padding: 0.5em 0.8em; text-align: left; }
    .check { color: var(--green); font-weight: 700; }
    .cross { color: var(--ink-muted); }

    /* Subtitle / Handnote */
    .subtitle { font-size: 0.6em; color: var(--ink-muted); }
    .handnote { font-family: var(--hand); color: var(--red); font-size: 0.85em; }
    .hand-underline { text-decoration: underline; text-decoration-color: var(--red); text-decoration-style: wavy; text-underline-offset: 4px; text-decoration-thickness: 2px; }

    /* Reveal overrides */
    .reveal .progress { color: var(--blue); height: 3px; }
    .reveal .controls { color: var(--ink); }
    .reveal .slide-number { font-family: var(--hand); font-size: 14px; color: var(--ink-muted); background: transparent; }
  </style>
</head>
<body>
  <div class="reveal">
    <div class="slides">

      <!-- 每一页是一个 <section> -->
      <section data-transition="fade">
        <h1>仓库名</h1>
        <blockquote>一句话总结</blockquote>
        <p class="subtitle">作者 · GitHub 链接</p>
      </section>

      <!-- ... 更多 section ... -->

    </div>
  </div>

  <script src="https://cdn.jsdelivr.net/npm/reveal.js@5/dist/reveal.js"></script>
  <script src="https://cdn.jsdelivr.net/npm/reveal.js@5/plugin/highlight/highlight.js"></script>
  <script src="https://cdn.jsdelivr.net/npm/reveal.js@5/plugin/notes/notes.js"></script>
  <script src="https://cdn.jsdelivr.net/npm/roughjs@4.6.6/bundled/rough.js"></script>
  <script>
    // ====== ROUGH.JS DRAWING HELPERS ======
    const BLUE = '#1971c2';
    const RED = '#e03131';
    const GREEN = '#2f9e44';
    const INK = '#1e1e1e';
    const INK_DIM = '#495057';
    const SEED = 42;

    function opts(o = {}) {
      return { roughness: 1.2, bowing: 1, strokeWidth: 2, stroke: INK, seed: SEED, ...o };
    }

    function drawArrow(rs, svg, x1, y1, x2, y2, o = {}) {
      const merged = opts(o);
      svg.appendChild(rs.line(x1, y1, x2, y2, merged));
      const angle = Math.atan2(y2 - y1, x2 - x1);
      const sz = 12;
      svg.appendChild(rs.polygon([
        [x2, y2],
        [x2 - sz * Math.cos(angle - 0.4), y2 - sz * Math.sin(angle - 0.4)],
        [x2 - sz * Math.cos(angle + 0.4), y2 - sz * Math.sin(angle + 0.4)],
      ], { ...merged, fill: merged.stroke, fillStyle: 'solid' }));
    }

    function drawText(svg, x, y, text, o = {}) {
      const t = document.createElementNS('http://www.w3.org/2000/svg', 'text');
      t.setAttribute('x', x);
      t.setAttribute('y', y);
      t.setAttribute('text-anchor', o.anchor || 'middle');
      t.setAttribute('fill', o.color || INK);
      t.style.fontFamily = o.font || "'Virgil', 'Caveat', cursive";
      t.style.fontSize = (o.size || 16) + 'px';
      t.style.fontWeight = o.weight || 'normal';
      t.textContent = text;
      svg.appendChild(t);
    }

    function drawBox(rs, svg, x, y, w, h, label, o = {}) {
      svg.appendChild(rs.rectangle(x, y, w, h, opts({
        fill: o.fill || 'transparent',
        fillStyle: o.fillStyle || 'hachure',
        fillWeight: 1,
        hachureGap: 8,
        stroke: o.stroke || INK,
        ...o
      })));
      if (label) drawText(svg, x + w / 2, y + h / 2 + 6, label, { color: o.textColor || INK, size: o.textSize || 15 });
    }

    function drawCircle(rs, svg, cx, cy, d, label, o = {}) {
      svg.appendChild(rs.circle(cx, cy, d, opts({
        fill: o.fill || 'transparent',
        fillStyle: o.fillStyle || 'hachure',
        hachureGap: 6,
        stroke: o.stroke || INK,
        ...o
      })));
      if (label) drawText(svg, cx, cy + 5, label, { color: o.textColor || INK, size: o.textSize || 14 });
    }

    // ====== INITIALIZE REVEAL + DRAW DIAGRAMS ======
    Reveal.initialize({
      hash: true,
      plugins: [RevealHighlight, RevealNotes],
      transition: 'fade',
      transitionSpeed: 'default',
      width: 1280,
      height: 760,
      margin: 0.06,
      slideNumber: 'c/t',
    }).then(() => {
      // 在这里用 rough.js 绘制所有 SVG 图表
      // 参考下方"rough.js 图表绘制指南"
    });
  </script>
</body>
</html>
```

#### rough.js 图表绘制指南

**不要用 mermaid**。所有架构图、流程图、概念图都用 rough.js 程序化绘制。

##### 基本用法

在 HTML 中放一个空的 `<svg>` 容器：

```html
<section>
  <h2>架构全景</h2>
  <div class="diagram-container">
    <svg id="arch-diagram" width="1100" height="460"></svg>
  </div>
</section>
```

在 `Reveal.initialize().then()` 回调中绘制：

```javascript
const svg = document.getElementById('arch-diagram');
const rs = rough.svg(svg);

// 画带标签的矩形
drawBox(rs, svg, 20, 50, 140, 55, '输入模块', {
  fill: '#d0ebff', fillStyle: 'solid', stroke: BLUE, textColor: BLUE
});

// 画箭头连接
drawArrow(rs, svg, 165, 78, 215, 78, { stroke: BLUE });

// 画圆形节点
drawCircle(rs, svg, 200, 200, 60, '核心', {
  fill: '#d3f9d8', fillStyle: 'solid', stroke: GREEN, textColor: GREEN
});

// 画文字标注
drawText(svg, 200, 300, '说明文字', { size: 13, color: INK_DIM });
```

##### 图表类型示例

1. **流程图**：一排 drawBox + drawArrow，从左到右
2. **架构图**：上下分层，drawBox 表示模块 + drawArrow 连线
3. **概念卡**：drawBox 做外框 + drawText 写 emoji/标题/描述
4. **关系图**：drawCircle 做节点 + rs.line 做连线 + drawText 写关系
5. **漏斗图**：rs.polygon 画梯形 + drawArrow 做流向
6. **对比图**：左右两列 drawBox 或 sketch-card

##### 颜色约定

- **蓝色系** `BLUE / #d0ebff / #e7f5ff`：主色调，用于核心模块、主流程
- **红色系** `RED / #ffe3e3 / #fff5f5`：对比、警告、反面例子
- **绿色系** `GREEN / #d3f9d8 / #ebfbee`：成功、正面、结果
- **墨色系** `INK / INK_DIM`：中性元素、边框、连线

##### 关键参数

- `fillStyle: 'solid'`：实色填充（最常用）
- `fillStyle: 'hachure'`：手绘阴影线（用于强调）
- `fillStyle: 'cross-hatch'`：交叉阴影（用于装饰性大面积）
- `roughness: 1.2`：手绘抖动程度（默认值，不要太大）
- `seed: 42`：固定随机种子，保证每次渲染一致

#### 代码高亮写法

用 `data-label` 属性给代码块加手写标签：

```html
<section>
  <h2>核心代码</h2>
  <pre data-label="Python"><code class="language-python">
def forward(self, idx):
    # 中文注释
    tok_emb = self.transformer.wte(idx)
    return tok_emb
  </code></pre>
</section>
```

#### 幻灯片写作原则

1. **每页一个概念**：不堆砌信息，一个 `<section>` 对应一个要点
2. **类比优先**：每个技术概念都配一个生活化类比（如"Transformer 就像一个能同时看到整篇文章的速读高手"）
3. **代码极简**：只展示最核心的代码片段（<15 行），加中文注释
4. **手绘图表**：架构页必须有 rough.js 手绘图，其他页面酌情使用
5. **渐进深入**：从"是什么"到"怎么做"到"怎么用"，不跳跃
6. **零术语假设**：首次出现的技术术语必须立即解释
7. **视觉节奏**：文字页和图表页交替，避免连续纯文字
8. **对比展示**：善用 `.chat-bubble` 和 `.sketch-card` 做有/没有的对比
9. **手写批注**：用 `.handnote` class 加手写批注风格的提示语

#### 常用组件速查

| 组件 | Class | 用途 |
|------|-------|------|
| 手绘卡片 | `.sketch-card` `.sketch-card.blue/red/green` | 内容容器 |
| 对话气泡 | `.chat-bubble.user` `.chat-bubble.ai-bad/ai-good` | 对比展示 |
| 网格布局 | `.grid-2` `.grid-3` | 分栏 |
| 标签胶囊 | `.tag` `.tag.red/green` | 关键词标注 |
| 步骤条 | `.step` + `.step-num` + `.step-text` | 流程说明 |
| 手写批注 | `.handnote` | 红色手写提示 |
| 波浪下划线 | `.hand-underline` | 强调关键词 |
| 引用块 | `<blockquote>` | 手写字体引用 |
| 代码块 | `<pre data-label="语言">` | 带手写标签的代码 |

### Step 4: 写入文件

将生成的 HTML 文件写入：

```
04-产出/幻灯片/{repo-name}-讲解.html
```

如果 `04-产出/幻灯片/` 目录不存在，先创建。

### Step 5: 自动打开并提示用户

1. 用 `open` 命令在浏览器中打开生成的 HTML 文件
2. 输出：
   - 文件路径
   - 页数统计
   - 操作提示：方向键翻页，`F` 全屏，`S` 演讲者视图，`ESC` 总览
   - 询问是否要调整某些页面或深入某个模块

## 注意事项

- **不要调用 `read_wiki_contents`**：返回全量 wiki 太大，浪费 token。用 `ask_question` 按需获取
- **不要用 mermaid**：所有图表一律用 rough.js 手绘风格绘制
- **DeepWiki 只支持公开仓库**：私有仓库需要 DeepWiki 付费版
- **HTML 自包含**：所有依赖通过 CDN 加载，单文件即可运行
- **rough.js 图表**：在 HTML 中放 `<svg id="xxx">` 容器，在 JS 的 `Reveal.initialize().then()` 中用 rough.js 绘制
- **代码高亮**：用 `<pre data-label="语言"><code class="language-xxx">` 包裹，reveal.js 的 highlight 插件 + github.css 浅色主题
- **文件命名**：用仓库名（不含 owner），如 `nanoGPT-讲解.html`
- **reveal.js 版本**：v5 + `theme/white.css` + `highlight/github.css`
- **rough.js 版本**：v4.6.6（CDN: cdn.jsdelivr.net/npm/roughjs@4.6.6/bundled/rough.js）
- **字体**：Virgil（Excalidraw）+ Caveat（手写 fallback）+ Inter（正文）+ JetBrains Mono（代码）
- **尺寸**：`width: 1280, height: 760, margin: 0.06`
- **过渡**：`transition: 'fade'`
- **页码**：`slideNumber: 'c/t'`
