---
name: repo-explainer
description: 读取 GitHub 开源仓库（通过 DeepWiki MCP），生成独立 HTML 幻灯片（Neobrutalism 风格 + 纯 SVG 硬边图表），用小白友好的方式讲解仓库做什么、怎么做、核心概念。触发词：讲解仓库、解读仓库、仓库幻灯片、explain repo、repo slides。用户提供 GitHub URL 或 owner/repo 格式即可。
---

# Repo Explainer — 用幻灯片讲解开源仓库

将任意 GitHub 公开仓库转化为**Neobrutalism 风格的独立 HTML 幻灯片**（基于 reveal.js CDN + 纯 SVG 硬边几何），浏览器直接打开即可演示。

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

将收集到的信息组织成**单个自包含 HTML 文件**，使用 reveal.js（CDN）+ 纯 SVG 硬边图表。

#### 设计风格：Neobrutalism（新野性主义）

**核心原则：raw, honest, 无装饰。**

- **米白或纯白底**，绝不暗色
- **纯黑粗边框**（3-5px），**零圆角**或极小圆角（0-2px）
- **硬投影**：`box-shadow: 8px 8px 0 #000`（无模糊，纯色偏移）
- **荧光高饱和色块**：电子黄、电子蓝、亮粉、荧光绿作为填充色
- **粗无衬线字体**：Space Grotesk / Archivo Black 做标题，常用 **全大写 + letter-spacing**
- **等宽字体**：JetBrains Mono 做代码和技术标签
- **字号反差极端**：标题超大（5-7em），正文中等（0.8em），强烈对比
- **不对齐 / 错位**：卡片允许轻微旋转（±1-2deg）或错位 grid，强化"未打磨"感
- **四色体系**：
  - 黄 `#FFDD00`（主强调 / 高亮块）
  - 蓝 `#2E5EFF`（次强调 / 信息块）
  - 粉 `#FF3D7F`（警告 / 对比 / 反例）
  - 绿 `#00D26A`（正面 / 成功 / 结果）
  - 纯黑 `#000` + 米白 `#FFFCF0`

#### HTML 模板骨架

```html
<!DOCTYPE html>
<html lang="zh-CN">
<head>
  <meta charset="utf-8">
  <title>{repo-name} — REPO BREAKDOWN</title>
  <link rel="preconnect" href="https://fonts.googleapis.com">
  <link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
  <link href="https://fonts.googleapis.com/css2?family=Space+Grotesk:wght@400;500;600;700&family=Archivo+Black&family=JetBrains+Mono:wght@400;500;700&display=swap" rel="stylesheet">
  <link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/reveal.js@5/dist/reveal.css">
  <link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/reveal.js@5/dist/theme/white.css">
  <link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/reveal.js@5/plugin/highlight/github.css">
  <style>
    :root {
      --yellow: #FFDD00;
      --blue: #2E5EFF;
      --pink: #FF3D7F;
      --green: #00D26A;
      --black: #000000;
      --paper: #FFFCF0;
      --paper-alt: #F4F1E4;
      --ink-dim: #2a2a2a;
      --ink-muted: #666;
      --display: 'Archivo Black', 'Space Grotesk', 'PingFang SC', sans-serif;
      --body: 'Space Grotesk', 'PingFang SC', 'Microsoft YaHei', sans-serif;
      --mono: 'JetBrains Mono', monospace;
      --border: 3px solid var(--black);
      --border-thick: 5px solid var(--black);
      --shadow: 8px 8px 0 var(--black);
      --shadow-sm: 4px 4px 0 var(--black);
      --shadow-lg: 12px 12px 0 var(--black);
    }

    html, body { background: var(--paper); }
    .reveal-viewport { background: var(--paper); }
    .reveal { font-family: var(--body); color: var(--black); font-size: 22px; }
    .reveal section { text-align: left; padding: 10px 30px; }

    /* Typography — BRUTAL */
    .reveal h1, .reveal h2, .reveal h3 {
      font-family: var(--display);
      text-transform: uppercase;
      letter-spacing: -0.01em;
      color: var(--black);
      font-weight: 900;
      line-height: 0.95;
    }
    .reveal h1 { font-size: 6em; margin-bottom: 0.15em; letter-spacing: -0.03em; }
    .reveal h2 { font-size: 2.6em; margin-bottom: 0.5em; display: inline-block; background: var(--yellow); padding: 0.1em 0.4em; border: var(--border-thick); box-shadow: var(--shadow); }
    .reveal h3 { font-size: 1.4em; letter-spacing: 0.02em; margin-bottom: 0.4em; color: var(--black); }
    .reveal p { font-family: var(--body); font-size: 0.85em; line-height: 1.55; color: var(--ink-dim); font-weight: 500; }
    .reveal ul, .reveal ol { font-size: 0.82em; line-height: 1.55; margin-left: 1.2em; color: var(--ink-dim); font-weight: 500; }
    .reveal li { margin-bottom: 0.35em; }
    .reveal li::marker { color: var(--black); font-weight: 900; }
    .reveal strong { color: var(--black); background: var(--yellow); padding: 0 0.15em; font-weight: 700; }
    .reveal em { font-style: normal; background: var(--pink); color: var(--black); padding: 0 0.2em; font-weight: 600; }
    .reveal a { color: var(--black); text-decoration: underline; text-decoration-thickness: 3px; text-underline-offset: 3px; font-weight: 700; }

    /* Code — BRUTAL */
    .reveal code { font-family: var(--mono); color: var(--black); background: var(--yellow); padding: 0.1em 0.35em; border: 2px solid var(--black); font-size: 0.85em; font-weight: 600; }
    .reveal pre { background: var(--paper-alt); border: var(--border-thick); padding: 1em 1.2em 1em 1.2em; font-size: 0.5em; box-shadow: var(--shadow); position: relative; margin-top: 1.5em; }
    .reveal pre::before {
      content: attr(data-label);
      position: absolute;
      top: -18px;
      left: -5px;
      background: var(--black);
      color: var(--yellow);
      padding: 0.15em 0.6em;
      font-family: var(--mono);
      font-weight: 700;
      font-size: 1.3em;
      text-transform: uppercase;
      letter-spacing: 0.05em;
      border: 3px solid var(--black);
    }
    .reveal pre code { font-family: var(--mono); color: var(--black); background: none; border: none; padding: 0; line-height: 1.7; font-weight: 500; }

    /* Blockquote — BIG STATEMENT */
    .reveal blockquote {
      border: var(--border-thick);
      background: var(--black);
      color: var(--paper);
      padding: 1em 1.4em;
      border-radius: 0;
      font-family: var(--display);
      font-size: 1.6em;
      line-height: 1.15;
      text-transform: uppercase;
      box-shadow: var(--shadow);
      font-style: normal;
      letter-spacing: -0.01em;
    }

    /* Brutal cards */
    .brutal-card {
      border: var(--border);
      padding: 1em 1.2em;
      background: white;
      box-shadow: var(--shadow);
      border-radius: 0;
    }
    .brutal-card.yellow { background: var(--yellow); }
    .brutal-card.blue { background: var(--blue); color: var(--paper); }
    .brutal-card.blue * { color: var(--paper); }
    .brutal-card.pink { background: var(--pink); }
    .brutal-card.green { background: var(--green); }
    .brutal-card.black { background: var(--black); color: var(--paper); }
    .brutal-card.black * { color: var(--paper); }
    .brutal-card.tilt-left { transform: rotate(-1.5deg); }
    .brutal-card.tilt-right { transform: rotate(1.5deg); }

    /* Dialog — brutal, no bubbles */
    .dialog-row { border: var(--border); padding: 0.5em 0.8em; margin-bottom: 0.4em; font-size: 0.75em; box-shadow: var(--shadow-sm); font-weight: 500; }
    .dialog-row.user { background: var(--paper-alt); }
    .dialog-row.bad { background: var(--pink); }
    .dialog-row.good { background: var(--green); }
    .dialog-row .role { font-family: var(--mono); font-weight: 700; text-transform: uppercase; font-size: 0.8em; letter-spacing: 0.1em; display: inline-block; margin-right: 0.5em; padding: 0 0.3em; background: var(--black); color: var(--paper); }

    /* Grids */
    .grid-2 { display: grid; grid-template-columns: 1fr 1fr; gap: 1.5em; }
    .grid-3 { display: grid; grid-template-columns: 1fr 1fr 1fr; gap: 1.2em; }
    .grid-asymmetric { display: grid; grid-template-columns: 2fr 1fr; gap: 1.5em; }

    /* Tags — brutal badges */
    .tag {
      display: inline-block;
      padding: 0.25em 0.7em;
      border: var(--border);
      background: var(--yellow);
      color: var(--black);
      font-family: var(--mono);
      font-size: 0.75em;
      font-weight: 700;
      text-transform: uppercase;
      letter-spacing: 0.05em;
      margin: 0.2em 0.2em 0.2em 0;
      box-shadow: var(--shadow-sm);
    }
    .tag.blue { background: var(--blue); color: var(--paper); }
    .tag.pink { background: var(--pink); }
    .tag.green { background: var(--green); }
    .tag.black { background: var(--black); color: var(--paper); }

    /* Steps — numbered blocks */
    .step { display: flex; align-items: stretch; gap: 0; margin-bottom: 0.6em; border: var(--border); box-shadow: var(--shadow-sm); }
    .step-num {
      width: 56px;
      background: var(--black);
      color: var(--yellow);
      display: flex;
      align-items: center;
      justify-content: center;
      font-family: var(--display);
      font-size: 1.8em;
      font-weight: 900;
      flex-shrink: 0;
    }
    .step-text { padding: 0.6em 0.9em; font-size: 0.82em; color: var(--ink-dim); line-height: 1.5; background: white; flex: 1; font-weight: 500; }
    .step-text strong { color: var(--black); background: var(--yellow); padding: 0 0.2em; }

    /* Diagram container */
    .diagram-container { display: flex; justify-content: center; margin: 0.8em 0; }
    .diagram-container svg { overflow: visible; }

    /* Table — brutal */
    .reveal table { width: 100%; border-collapse: collapse; font-size: 0.72em; border: var(--border-thick); box-shadow: var(--shadow); background: white; }
    .reveal th {
      background: var(--black);
      color: var(--yellow);
      border: 2px solid var(--black);
      padding: 0.6em 0.9em;
      text-align: left;
      font-family: var(--display);
      text-transform: uppercase;
      letter-spacing: 0.03em;
      font-size: 1em;
    }
    .reveal td { border: 2px solid var(--black); padding: 0.55em 0.9em; text-align: left; font-weight: 500; }
    .reveal tr:nth-child(even) td { background: var(--paper-alt); }
    .check { color: var(--black); background: var(--green); padding: 0 0.3em; font-weight: 900; }
    .cross { color: var(--paper); background: var(--black); padding: 0 0.3em; font-weight: 900; }

    /* Marker — annotation style */
    .subtitle { font-family: var(--mono); font-size: 0.65em; color: var(--black); text-transform: uppercase; letter-spacing: 0.1em; font-weight: 700; }
    .marker {
      display: inline-block;
      background: var(--pink);
      color: var(--black);
      padding: 0.15em 0.5em;
      border: 2px solid var(--black);
      font-family: var(--mono);
      font-weight: 700;
      font-size: 0.8em;
      text-transform: uppercase;
    }
    .stamp {
      display: inline-block;
      border: 3px solid var(--black);
      padding: 0.2em 0.6em;
      font-family: var(--display);
      text-transform: uppercase;
      letter-spacing: 0.05em;
      transform: rotate(-3deg);
      background: var(--yellow);
      font-size: 0.9em;
    }

    /* Big number display */
    .big-num { font-family: var(--display); font-size: 5em; line-height: 1; color: var(--black); }
    .big-num.yellow-bg { background: var(--yellow); padding: 0 0.1em; }

    /* Reveal overrides */
    .reveal .progress { color: var(--black); height: 6px; background: transparent; }
    .reveal .progress span { background: var(--black); }
    .reveal .controls { color: var(--black); }
    .reveal .slide-number {
      font-family: var(--mono);
      font-size: 14px;
      color: var(--paper);
      background: var(--black);
      padding: 4px 10px;
      font-weight: 700;
      letter-spacing: 0.1em;
    }

    /* Decorative brutalist grid lines (optional bg) */
    .grid-bg {
      position: absolute;
      inset: 0;
      background-image:
        linear-gradient(var(--black) 1px, transparent 1px),
        linear-gradient(90deg, var(--black) 1px, transparent 1px);
      background-size: 80px 80px;
      opacity: 0.04;
      pointer-events: none;
    }
  </style>
</head>
<body>
  <div class="reveal">
    <div class="slides">

      <!-- 封面页示例 -->
      <section data-transition="none">
        <div class="grid-bg"></div>
        <p class="subtitle">REPO BREAKDOWN // 001</p>
        <h1>REPO-NAME</h1>
        <blockquote>一句话暴击总结</blockquote>
        <p style="margin-top: 1em;"><span class="tag">Python</span><span class="tag blue">LLM</span><span class="tag pink">12.4K★</span></p>
      </section>

      <!-- 更多 section -->

    </div>
  </div>

  <script src="https://cdn.jsdelivr.net/npm/reveal.js@5/dist/reveal.js"></script>
  <script src="https://cdn.jsdelivr.net/npm/reveal.js@5/plugin/highlight/highlight.js"></script>
  <script src="https://cdn.jsdelivr.net/npm/reveal.js@5/plugin/notes/notes.js"></script>
  <script>
    // ====== BRUTAL SVG HELPERS (无 rough.js，纯硬边几何) ======
    const SVG_NS = 'http://www.w3.org/2000/svg';
    const YELLOW = '#FFDD00';
    const BLUE = '#2E5EFF';
    const PINK = '#FF3D7F';
    const GREEN = '#00D26A';
    const BLACK = '#000000';
    const PAPER = '#FFFCF0';

    function el(tag, attrs = {}) {
      const n = document.createElementNS(SVG_NS, tag);
      for (const [k, v] of Object.entries(attrs)) n.setAttribute(k, v);
      return n;
    }

    // Brutal rect: 硬边 + 硬投影（通过多层 rect 模拟 box-shadow）
    function drawBox(svg, x, y, w, h, label, o = {}) {
      const fill = o.fill || 'white';
      const stroke = o.stroke || BLACK;
      const sw = o.strokeWidth || 3;
      const shadowOffset = o.shadow === false ? 0 : (o.shadowOffset || 6);
      // shadow layer
      if (shadowOffset > 0) {
        svg.appendChild(el('rect', {
          x: x + shadowOffset, y: y + shadowOffset, width: w, height: h,
          fill: BLACK
        }));
      }
      // main rect
      svg.appendChild(el('rect', {
        x, y, width: w, height: h,
        fill, stroke, 'stroke-width': sw
      }));
      if (label) {
        const lines = Array.isArray(label) ? label : [label];
        const lineH = o.textSize || 16;
        const totalH = lines.length * lineH * 1.2;
        const startY = y + h / 2 - totalH / 2 + lineH;
        lines.forEach((line, i) => {
          drawText(svg, x + w / 2, startY + i * lineH * 1.2, line, {
            color: o.textColor || BLACK,
            size: o.textSize || 16,
            weight: o.textWeight || 700,
            font: o.textFont
          });
        });
      }
    }

    // Brutal arrow: 粗黑直线 + 三角箭头
    function drawArrow(svg, x1, y1, x2, y2, o = {}) {
      const stroke = o.stroke || BLACK;
      const sw = o.strokeWidth || 4;
      svg.appendChild(el('line', {
        x1, y1, x2, y2,
        stroke, 'stroke-width': sw, 'stroke-linecap': 'square'
      }));
      const angle = Math.atan2(y2 - y1, x2 - x1);
      const sz = o.headSize || 14;
      const p1x = x2 - sz * Math.cos(angle - 0.5);
      const p1y = y2 - sz * Math.sin(angle - 0.5);
      const p2x = x2 - sz * Math.cos(angle + 0.5);
      const p2y = y2 - sz * Math.sin(angle + 0.5);
      svg.appendChild(el('polygon', {
        points: `${x2},${y2} ${p1x},${p1y} ${p2x},${p2y}`,
        fill: stroke, stroke: stroke, 'stroke-width': 1, 'stroke-linejoin': 'miter'
      }));
    }

    // Brutal circle (实心圆节点)
    function drawCircle(svg, cx, cy, d, label, o = {}) {
      const r = d / 2;
      const fill = o.fill || 'white';
      const stroke = o.stroke || BLACK;
      const sw = o.strokeWidth || 3;
      const shadowOffset = o.shadow === false ? 0 : (o.shadowOffset || 5);
      if (shadowOffset > 0) {
        svg.appendChild(el('circle', {
          cx: cx + shadowOffset, cy: cy + shadowOffset, r, fill: BLACK
        }));
      }
      svg.appendChild(el('circle', {
        cx, cy, r, fill, stroke, 'stroke-width': sw
      }));
      if (label) drawText(svg, cx, cy + 5, label, {
        color: o.textColor || BLACK,
        size: o.textSize || 14,
        weight: o.textWeight || 700
      });
    }

    // Text
    function drawText(svg, x, y, text, o = {}) {
      const t = el('text', {
        x, y,
        'text-anchor': o.anchor || 'middle',
        fill: o.color || BLACK,
      });
      t.style.fontFamily = o.font || "'Space Grotesk', 'Archivo Black', sans-serif";
      t.style.fontSize = (o.size || 16) + 'px';
      t.style.fontWeight = o.weight || 700;
      if (o.uppercase) { t.style.textTransform = 'uppercase'; t.style.letterSpacing = '0.05em'; }
      t.textContent = text;
      svg.appendChild(t);
    }

    // Mono label (JetBrains Mono, 常用于技术标签)
    function drawMono(svg, x, y, text, o = {}) {
      drawText(svg, x, y, text, {
        ...o,
        font: "'JetBrains Mono', monospace",
        uppercase: true,
      });
    }

    // Label chip: 小方块 + 文字（类似 CSS .tag）
    function drawChip(svg, x, y, text, o = {}) {
      const fill = o.fill || YELLOW;
      const paddingX = 8, paddingY = 4;
      // 粗估宽度
      const w = text.length * (o.size || 12) * 0.6 + paddingX * 2;
      const h = (o.size || 12) + paddingY * 2;
      drawBox(svg, x, y, w, h, null, { fill, shadowOffset: 3 });
      drawMono(svg, x + w / 2, y + h / 2 + (o.size || 12) / 3, text, {
        size: o.size || 12,
        color: o.textColor || BLACK,
        weight: 700
      });
      return w;
    }

    // ====== INIT REVEAL ======
    Reveal.initialize({
      hash: true,
      plugins: [RevealHighlight, RevealNotes],
      transition: 'none',  // brutalism 不用渐变过渡
      width: 1280,
      height: 760,
      margin: 0.05,
      slideNumber: 'c/t',
    }).then(() => {
      // 在这里绘制所有图表
      // 见下方"SVG 图表绘制指南"
    });
  </script>
</body>
</html>
```

#### SVG 图表绘制指南

**不用 rough.js，不用 mermaid**。所有架构图、流程图、关系图都用纯 SVG + helper 函数绘制，保证 brutalism 的硬边几何质感。

##### 基本用法

```html
<section>
  <h2>ARCHITECTURE</h2>
  <div class="diagram-container">
    <svg id="arch-diagram" width="1100" height="460"></svg>
  </div>
</section>
```

```javascript
const svg = document.getElementById('arch-diagram');

// 画带标签的硬边矩形（自带黑色硬投影）
drawBox(svg, 40, 60, 200, 80, 'INPUT', {
  fill: YELLOW, textColor: BLACK, textSize: 22, textWeight: 900
});

// 粗黑箭头
drawArrow(svg, 240, 100, 310, 100, { strokeWidth: 5 });

// 多行标签
drawBox(svg, 310, 60, 220, 80, ['TRANSFORMER', 'BLOCK'], {
  fill: BLUE, textColor: PAPER, textSize: 22
});

// 圆形节点
drawCircle(svg, 700, 100, 80, 'CORE', {
  fill: PINK, textSize: 20, textWeight: 900
});

// 代码风格的 mono 标签
drawMono(svg, 550, 250, '// data flow', { size: 14, color: '#666' });

// Chip 标签（带硬投影的小方块）
drawChip(svg, 40, 400, 'v2.1.0', { fill: GREEN });
```

##### 图表类型示例

1. **流程图**：水平排列 drawBox + drawArrow，色块交替（黄→蓝→粉→绿）
2. **架构图**：上下分层，每层一个大 drawBox 用不同填充色，用粗箭头连接
3. **概念卡**：drawBox 做外框 + drawText 做大号标题 + drawMono 做代码标签
4. **关系图**：drawCircle 做节点（不同颜色区分类型） + 粗直线（`el('line')`）做连接
5. **对比图**：左右两列 drawBox，颜色对立（如左绿右粉），中间夹 "VS" 大字
6. **时间线**：一条粗黑水平线 + 多个 drawCircle 节点 + 上下交替标签

##### 颜色使用规则

- **黄 `YELLOW`**：高亮/主角模块（一个图表里最重要的那块）
- **蓝 `BLUE`**：技术性/数据流/处理过程（深色配白字）
- **粉 `PINK`**：警告/反例/对比面
- **绿 `GREEN`**：输出/结果/成功态
- **黑 `BLACK`**：所有边框、箭头、文字（一律纯黑，不用灰）
- **白/米白 `PAPER`**：次要模块或浅色卡片底

**规则**：单个图表最多用 3 种高饱和色 + 黑白。全用亮色会视觉爆炸。

##### SVG 中的字体

所有 SVG 文字默认使用 `Space Grotesk` 粗体。技术性标签（变量名、版本号、commit hash）用 `drawMono()` 切到 `JetBrains Mono`。

#### 代码高亮写法

代码块用 `data-label` 属性作为 brutal tab 标签（黑底黄字，定位在左上角外侧）：

```html
<section>
  <h2>CORE CODE</h2>
  <pre data-label="PYTHON"><code class="language-python">
def forward(self, idx):
    # attention 的入口
    tok_emb = self.transformer.wte(idx)
    return tok_emb
  </code></pre>
</section>
```

#### 幻灯片写作原则（Brutalism 版）

1. **一页一个暴击**：不要堆砌，一个 `<section>` 只讲一件事，宁可多翻几页
2. **标题全大写**：所有 h1/h2/h3 用全大写，视觉冲击力最大化
3. **类比要直白**："Transformer = 能同时看到整篇文章的速读高手"比"注意力机制允许模型关注输入序列不同位置"强 10 倍
4. **代码极简**：<15 行，中文注释，用 `.stamp` 组件在旁边贴个"CORE" / "MAGIC"
5. **色块分页**：每隔 3-4 页，做一页"全色块"过渡页（整屏黄 / 整屏黑）做章节分隔
6. **错位与旋转**：封面/章节页可以让卡片 `.tilt-left` 或 `.tilt-right` 轻微旋转，强化"raw"感
7. **大数字 / 大引用**：关键数据用 `.big-num`（5em 起步），关键观点用 `<blockquote>` 黑底白字
8. **对比暴力**：好/坏、旧/新、有/没有 —— 用 `.dialog-row.bad`（粉底）和 `.dialog-row.good`（绿底）直接并列
9. **零装饰**：不要渐变、不要半透明、不要阴影模糊。所有阴影都是硬偏移 `8px 8px 0 #000`
10. **术语 = 标签**：每个首次出现的技术术语，立刻配一个 `.tag` 贴标签，让技术词看起来像徽章

#### 常用组件速查

| 组件 | Class / Fn | 用途 |
|------|-----------|------|
| 硬边卡片 | `.brutal-card` `.yellow/blue/pink/green/black` | 内容容器，自带硬投影 |
| 倾斜卡片 | `.brutal-card.tilt-left/right` | 轻微旋转，增强 raw 感 |
| 对话行 | `.dialog-row.user/bad/good` | 用户/反例/正例对比 |
| 网格布局 | `.grid-2` `.grid-3` `.grid-asymmetric` | 分栏 |
| 徽章标签 | `.tag` `.tag.blue/pink/green/black` | 技术词、状态标注 |
| 步骤条 | `.step` + `.step-num` + `.step-text` | 大数字 + 说明 |
| 标记块 | `.marker` | 粉色短标注 |
| 图章 | `.stamp` | 倾斜的黄色印章，标注"CORE / NEW / HOT" |
| 大数字 | `.big-num` `.big-num.yellow-bg` | 5em+ 数字展示 |
| 大引用 | `<blockquote>` | 黑底黄字大标语 |
| 代码块 | `<pre data-label="语言">` | 左上角带 brutal tab 标签 |
| 表格 | `<table>` | 黑边 + 黑表头 + 黄字表头 |
| SVG 图表 | `drawBox / drawArrow / drawCircle / drawText / drawMono / drawChip` | 纯 SVG helper |

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
- **不用 mermaid，不用 rough.js**：所有图表用纯 SVG + helper 函数，保证 brutalism 的硬边几何质感
- **DeepWiki 只支持公开仓库**：私有仓库需要 DeepWiki 付费版
- **HTML 自包含**：所有依赖通过 CDN 加载，单文件即可运行
- **SVG 图表**：在 HTML 中放 `<svg id="xxx">` 容器，在 JS 的 `Reveal.initialize().then()` 中用 helper 函数绘制
- **代码高亮**：用 `<pre data-label="语言"><code class="language-xxx">` 包裹，reveal.js 的 highlight 插件 + github.css 浅色主题
- **文件命名**：用仓库名（不含 owner），如 `nanoGPT-讲解.html`
- **reveal.js 版本**：v5 + `theme/white.css` + `highlight/github.css`
- **字体**：Archivo Black（display 标题）+ Space Grotesk（正文）+ JetBrains Mono（代码/标签）
- **尺寸**：`width: 1280, height: 760, margin: 0.05`
- **过渡**：`transition: 'none'`（brutalism 不用平滑过渡，硬切更酷）
- **页码**：`slideNumber: 'c/t'`，页码本身也是黑底黄字的 brutal 标签
- **色号锁死**：Yellow `#FFDD00` / Blue `#2E5EFF` / Pink `#FF3D7F` / Green `#00D26A` / Black `#000` / Paper `#FFFCF0`，不要随意用其他颜色
