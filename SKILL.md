---
name: repo-explainer
description: 读取 GitHub 开源仓库（通过 DeepWiki MCP，硬依赖，未安装会先引导安装），生成独立 HTML 幻灯片（Neobrutalism 风格 + 纯 SVG 硬边图表），用小白友好的方式讲解仓库做什么、怎么做、核心概念。会先按仓库类型（ML/CLI/库/框架/应用/基础设施/协议/教程）分类，问题集与幻灯片骨架随类型自适应。触发词：讲解仓库、解读仓库、仓库幻灯片、explain repo、repo slides。用户提供 GitHub URL 或 owner/repo 格式即可。
allowed-tools:
  - mcp__deepwiki__read_wiki_structure
  - mcp__deepwiki__ask_question
  - Read
  - Write
  - Edit
  - Bash
---

# Repo Explainer — 用幻灯片讲解开源仓库

将任意 GitHub 公开仓库转化为**Neobrutalism 风格的独立 HTML 幻灯片**（reveal.js CDN + 纯 SVG 硬边几何），浏览器直接打开即可演示。

> **风格说明**：Neobrutalism 设计系统（色板、字体、组件、SVG helper）全部固化在 `assets/template.html` 里，与用户的 `CLAUDE.md` 偏好无关。无论当前项目声明过什么设计风格，本 skill 都产出统一的 brutalism 幻灯片——这是它的产品特征，不接受运行时风格切换。

## 触发条件

用户提供 GitHub 仓库地址（URL 或 owner/repo 格式）并要求讲解、解读、做幻灯片时触发。

## 输入解析

从用户输入中提取：

- `owner/repo`：从 URL `github.com/{owner}/{repo}` 或直接文本中提取
- 可选参数：
  - **语言**：**默认跟随用户当前对话使用的语言**（用户用中文说话就出中文幻灯片，用英文说话就出英文，其他语言同理）。用户可显式指定覆盖，如"用英文做"/"output in English"。判定依据是用户这一轮请求的语言，不是仓库 README 的语言，也不是 skill 文档本身的语言。
  - **深度**：`快速`（~8 页）/ `标准`（~15 页，默认）/ `深入`（~25 页）
  - **聚焦**：用户可指定只讲某个模块，如"只讲训练部分"
  - **讲解重点**：用户可覆盖默认推导，如"重点讲怎么用" / "重点讲它为什么这么设计"

> 仓库类型、受众、讲解重点会在 Step 1.5 自动判定并路由——用户无需指定，但可显式覆盖讲解重点。

## 执行流程

### Step 0: 前置检查（DeepWiki MCP）

本 skill 强依赖 DeepWiki MCP，提供两个能力：**读仓库 wiki 结构** 和 **就仓库提问**。

**按能力找工具，不要按名字硬匹配。** 各家 harness 的 MCP 工具命名前缀不同：

- Claude Code：`mcp__deepwiki__read_wiki_structure` / `mcp__deepwiki__ask_question`
- 其他 harness：前缀可能是 `deepwiki.` `deepwiki:` 或直接 `read_wiki_structure`

工具可用就直接进 Step 1，别多问。

#### 不可用时：帮用户装，别甩一条命令走人

这是小白第一次用就会卡住的地方。**先判断当前跑在哪个 harness 里**（用 `which` 探测 CLI，或看工具命名风格），再给对应的装法：

**Claude Code**（`which claude` 有结果）：

```bash
claude mcp add --transport http deepwiki https://mcp.deepwiki.com/mcp
```

**Codex**（存在 `~/.codex/config.toml`）：往配置文件里追加一段。这是用户的**全局配置**，改之前先说一声再动；先 `grep deepwiki` 确认没配过，然后追加到文件末尾（新的 `[table]` 头在 EOF 追加是安全的 TOML）：

```toml
[mcp_servers.deepwiki]
url = "https://mcp.deepwiki.com/mcp"
```

**其他 harness**：endpoint 是 `https://mcp.deepwiki.com/mcp`（streamable HTTP，无需本地进程、不用 API key），按该 harness 自己的 MCP 配置方式加进去即可。

装完统一告诉用户三件事，缺一不可：

1. **装好了**（以及改了哪个文件）
2. **需要重启会话** MCP 才会加载（Claude Code 是 `/exit` 后重进；Codex 是退出 TUI 重开）
3. **重启后把刚才那句话原样再发一遍就行**

不要只说"请安装后重试"——用户不知道要重启，也不知道重启后该干嘛。

**不要用 `WebFetch` / `curl` / `gh` 兜底**——本 skill 的内容质量依赖 DeepWiki 的结构化抽取，没有它就不要硬撑出一份低质量幻灯片，直接停在这一步。

### Step 1: 获取仓库结构

```
read_wiki_structure(repoName: "owner/repo")
```

分析返回的章节列表，确定仓库的核心模块。

**如果报错说仓库不存在 / 未索引 / 返回内容明显空泛**（DeepWiki 只对它已索引过的仓库好使，冷门仓库常常没有），**不要硬着头皮继续问**，直接告诉用户：

> 这个仓库 DeepWiki 还没索引过。打开 `https://deepwiki.com/{owner}/{repo}` 点一下索引，通常几分钟就好，完了再回来找我。

顺带确认一下是不是私有仓库——DeepWiki 免费版只支持公开仓库。

### Step 1.5: 仓库分类（路由）⭐

**这是质量的关键一步。别对所有仓库套同一套模板**——nanoGPT 该重点讲模型架构和关键代码，而一个 CLI 工具该重点讲用法和工作流。在提问和做幻灯片之前，先给仓库定个坐标，后续的问题集（Step 2）和幻灯片骨架（Step 3）都按这个坐标路由。

根据 Step 1 拿到的 wiki 结构（章节名、模块构成）+ 仓库名/README 信号，沿 3 个维度判定：

| 维度 | 选项 | 影响 |
|---|---|---|
| **仓库类型** | ML/研究 · CLI工具 · 库/SDK · 框架 · 应用/服务 · 基础设施/DevOps · 协议/规范 · 学习/教程 | Step 2 问题集 + Step 3 幻灯片骨架 |
| **受众** | 小白（默认）· 工程师 | 类比深度、是否上代码细节 |
| **讲解重点** | 怎么用 · 怎么实现 · 为什么存在 | 哪几页占主篇幅 |

**类型判定速查：**

- 有 `train/` `model/` `loss` `dataset` → **ML/研究**
- 主入口是命令、有 `cli/` `commands/`、用 `argparse/cobra/clap` → **CLI工具**
- 通过 `import`/`require` 被别人引用、走 `pip/npm` 发布 → **库/SDK**
- 提供应用骨架/路由/生命周期、让你"在上面盖房子" → **框架**
- 能独立 `docker run` / 部署起来的完整产品 → **应用/服务**
- `k8s/terraform/helm/ansible`、做编排部署 → **基础设施/DevOps**
- 实现某个标准/RFC/wire format → **协议/规范**
- 名字含 `awesome-` `build-your-own` `tutorial` `100-days` → **学习/教程**

**讲解重点**默认由类型推导（见 Step 2 末尾对应关系），用户可显式覆盖。

**输出一个路由字符串**，写在回复里让用户看到，例：`{ 类型: ML/研究, 受众: 小白, 重点: 怎么实现 }`。这个字符串决定 Step 2 问哪些、Step 3 怎么排版。

### Step 2: 针对性提问（按 Step 1.5 类型路由）

向 DeepWiki 提问。**不要调用 `read_wiki_contents`**（返回全量内容太大）。问题数随深度走：快速 3-4 问 / 标准 5 问 / 深入 6-7 问。

**通用必问（所有类型，用「输入解析」判定出的输出语言提问，让 DeepWiki 直接返回可用的文案）：**

1. **一句话总结**："用一句话向完全不懂编程的人解释这个项目是做什么的"
2. **核心概念**："这个项目的 3-5 个核心概念是什么？每个用一句话解释"
3. **架构总览**："描述这个项目的整体架构，包含核心模块和数据流向"

**按【仓库类型】追加：**

| 类型 | 追加问题 |
|---|---|
| ML/研究 | • 最核心的一段模型/训练代码（<30 行）逐行注释 • 训练/推理的数据流是怎样的 • 关键数学直觉用大白话解释 |
| CLI工具 | • 新手最少几步装好并跑通第一个命令 • 最常用的 3-5 个命令及典型工作流 • 它替代了什么手工痛点 |
| 库/SDK | • 最小可用代码示例（<20 行）• 核心抽象/API 是什么、怎么集成进现有项目 • 和同类库相比独特在哪 |
| 框架 | • 一个最小应用的骨架长什么样 • 核心抽象（路由/组件/生命周期）如何组织 • 适合/不适合哪类项目 |
| 应用/服务 | • 最常见的 3 个使用场景 • 部署/自托管最少几步 • 核心数据流/请求生命周期 |
| 基础设施/DevOps | • 它在整个技术栈里处在什么位置 • 一个典型部署拓扑/数据流 • 解决了什么运维痛点 |
| 协议/规范 | • 这个协议解决什么问题、消息怎么流动 • 一段最小的协议交互示例 • 和已有方案相比为什么需要它 |
| 学习/教程 | • 这个仓库带你从 0 做出什么 • 学习路径怎么分阶段 • 读完能掌握哪些核心概念 |

**讲解重点 ← 类型默认推导**（用户未覆盖时）：

- 怎么用 ← CLI工具 / 应用/服务 / 框架 / 学习教程
- 怎么实现 ← ML/研究 / 库/SDK / 协议规范
- 为什么存在 ← 用户显式指定，或项目有强设计主张时

**按【受众】调语气：** 小白多用类比、少上代码；工程师可保留代码细节和架构术语。

### Step 3: 用模板生成 HTML

**不要从零手写 HTML/CSS。** 样式、字体、组件、SVG helper 全在 `assets/template.html` 里，照抄一遍既慢又必然抄错。流程是「复制模板 → 填占位符 → 插内容」：

```bash
# SKILL_DIR = 本 skill 的 base directory（skill 被调用时会告知，
# 通常是 ~/.claude/skills/repo-explainer 或项目内 .claude/skills/repo-explainer）
cp "$SKILL_DIR/assets/template.html" "$OUT_DIR/$FILENAME"
```

模板里有 4 个占位符，用 `Edit` 逐个替换：

| 占位符 | 替换成 |
|---|---|
| `{{LANG}}` | `zh-CN` / `en` / 其他，跟随输出语言 |
| `{{TITLE}}` | `{repo-name} — REPO BREAKDOWN` |
| `<!-- SLIDES -->` | 所有 `<section>` 幻灯片 |
| `// DIAGRAMS` | 所有 `diagram(...)` 绘图调用 |

**不要改模板里的 CSS 和 helper 函数**。需要新样式时优先复用现有 class；确实缺组件再说，不要在 `<section>` 里写一堆内联 style。

#### 幻灯片骨架（按 Step 1.5 类型路由）

封面页 + 一句话暴击 + 结尾页所有类型通用。**中间主体页按类型排序**，重点页占大篇幅：

| 类型 | 推荐主体页序（标准深度，~15 页） |
|---|---|
| ML/研究 | 解决什么问题 → 核心概念标签墙 → 模型架构图 → 训练/推理数据流 → 关键代码逐行 → 数学直觉类比 → 能拿来干嘛 |
| CLI工具 | 替你省了什么手工活 → 安装(step 组件) → 核心命令速查(table) → 典型工作流(drawFlow) → 进阶技巧 |
| 库/SDK | 解决什么问题 → 核心抽象 → 最小示例代码 → 集成方式(drawFlow) → 对比同类(drawCompare) |
| 框架 | 让你少写什么 → 核心概念 → 最小应用骨架代码 → 架构分层图(drawStack) → 适合/不适合 |
| 应用/服务 | 它是什么 → 核心概念 → 架构图(drawStack) → 典型场景 → 自托管步骤 |
| 基础设施/DevOps | 在技术栈里的位置 → 核心概念 → 部署拓扑图 → 数据流(drawFlow) → 解决的运维痛点 |
| 协议/规范 | 解决什么问题 → 核心概念 → 消息流图(drawFlow) → 最小交互示例 → 为什么需要它 |
| 学习/教程 | 你会做出什么 → 学习路径(drawTimeline) → 分阶段里程碑 → 核心概念清单 → 下一步 |

**讲解重点决定篇幅分配：** 怎么用 → 上手/命令/示例页占大头；怎么实现 → 架构图 + 关键代码逐行页占大头；为什么存在 → 设计哲学/对比/取舍页占大头。

**深度调节：** `快速` 砍掉代码/对比页只留骨干；`深入` 时每个主体页可拆成 2 页展开。

**生成完数一遍 `<section>` 数量**，跟目标页数差太多（少于 80%）就补页，别嘴上说"~15 页"实际只给 9 页。

#### 页面内容硬约束（防溢出）

幻灯片画布固定 **1280×760**，超出部分是**静默截断**的——不报错，就是看不见。中文比英文密，更容易溢出。所以：

- 正文列表 **≤ 8 条**，每条 ≤ 25 字
- 代码块 **≤ 15 行**（超了就截取核心片段，用 `# ...` 省略）
- 一页里 `<h2>` + 一个图表 + 一段说明就满了，别再塞第二个图表
- 单个 SVG 图表内 **≤ 6 个节点**，超了拆成两页
- 拿不准就拆页。「一页一个暴击」不是修辞，是排版约束

#### SVG 图表：优先用 auto-layout，不要手算坐标

模板提供两层 API。**默认用 auto-layout 层**——只给内容数组，间距/换行/字号/画布尺寸全自动算，不会重叠也不会画出画布外。手算坐标是过去图表翻车的主要原因。

HTML 里放容器（**不用写 width/height**，`autoFit` 会设）：

```html
<div class="diagram-container"><svg id="d-arch"></svg></div>
```

JS 里用 `diagram(id, fn)` 包一层，它会自动调 `autoFit`：

| 函数 | 用途 | 例子 |
|---|---|---|
| `drawFlow(svg, items, o)` | **横向流程 / 数据流**（最常用） | `drawFlow(svg, ['原始文本','分词器','Transformer','输出'], { arrowLabels: ['切分','前向','采样'] })` |
| `drawStack(svg, layers, o)` | **竖向分层架构** | `drawStack(svg, [{label:'应用层', note:'CLI / SDK'}, {label:'核心引擎'}, {label:'存储'}])` |
| `drawGrid(svg, items, o)` | **概念卡网格** | `drawGrid(svg, [{label:'注意力', note:'一眼看全文'}, ...], { cols: 3 })` |
| `drawCompare(svg, l, r, o)` | **左右对比 + VS** | `drawCompare(svg, {title:'手写循环', items:['慢','易错']}, {title:'用它', items:['快','稳']})` |
| `drawTimeline(svg, nodes, o)` | **时间线 / 学习路径** | `drawTimeline(svg, [{label:'跑通 demo', note:'10 分钟'}, ...])` |

items 可以是字符串数组，也可以是 `{ label, fill, note }` 对象数组。不传 `fill` 就按黄→蓝→绿三色自动轮转（守住「单图最多 3 种高饱和色」的规矩）。

完整例子：

```javascript
diagram('d-arch', svg => drawStack(svg, [
  { label: '应用层：CLI 与 Python API', note: 'train.py / sample.py' },
  { label: '模型层：GPT + Block + CausalSelfAttention' },
  { label: '数据层：二进制 token 流 memmap', note: 'data/*.bin' },
]));
```

**底层图元**（auto-layout 表达不了的特殊图才用，用完必须自己调 `autoFit(svg)`）：
`drawBox(svg,x,y,w,h,label,o)` · `drawArrow(svg,x1,y1,x2,y2,o)` · `drawCircle(svg,cx,cy,d,label,o)` · `drawText` · `drawMono` · `drawChip`

`drawBox` 的 label 传字符串会按框宽自动换行（中英混排安全）并在放不下时自动缩字号；传数组则按数组逐行渲染。填充色是深色（蓝/黑）时文字自动转浅色，不用手写 `textColor`。

颜色常量：`YELLOW` `BLUE` `PINK` `GREEN` `BLACK` `PAPER` `WHITE`。用途：黄=主角模块，蓝=数据流/处理，粉=警告/反例，绿=输出/成功，黑=所有边框箭头文字。

#### 组件速查（HTML class，全在模板里）

| 组件 | Class | 用途 |
|---|---|---|
| 硬边卡片 | `.brutal-card` + `.yellow/.blue/.pink/.green/.black` | 内容容器，自带硬投影 |
| 倾斜卡片 | `.brutal-card.tilt-left/.tilt-right` | ±1.5deg，增强 raw 感 |
| 对话行 | `.dialog-row.user/.bad/.good` | 用户/反例/正例对比 |
| 网格 | `.grid-2` `.grid-3` `.grid-asymmetric` | 分栏 |
| 徽章 | `.tag` + `.blue/.pink/.green/.black` | 技术词、状态 |
| 步骤条 | `.step` > `.step-num` + `.step-text` | 大数字 + 说明 |
| 标记 | `.marker` | 粉色短标注 |
| 图章 | `.stamp` | 倾斜黄印章（CORE / NEW / HOT）|
| 大数字 | `.big-num` `.big-num.yellow-bg` | 5em+ 数字 |
| 大引用 | `<blockquote>` | 黑底黄字大标语 |
| 代码块 | `<pre data-label="PYTHON"><code class="language-python">` | 左上角 brutal tab |
| 背景网格 | `.grid-bg` | 封面/章节页装饰 |

封面页写法：

```html
<section>
  <div class="grid-bg"></div>
  <p class="subtitle">REPO BREAKDOWN // 001</p>
  <h1>NANOGPT</h1>
  <blockquote>300 行代码复现 GPT-2</blockquote>
  <p><span class="tag">Python</span><span class="tag blue">LLM</span><span class="tag pink">38K★</span></p>
</section>
```

#### 写作原则

1. **一页一个暴击**：一个 `<section>` 只讲一件事，宁可多翻几页
2. **标题全大写**：h1/h2/h3 全大写，视觉冲击最大
3. **类比要直白**："Transformer = 能同时看到整篇文章的速读高手" 比 "注意力机制允许模型关注输入序列不同位置" 强 10 倍
4. **代码极简**：≤15 行，注释用输出语言，旁边贴个 `.stamp` 写 "CORE"
5. **色块分页**：每隔 3-4 页做一页全色块过渡页（整屏黄 / 整屏黑）分隔章节
6. **大数字 / 大引用**：关键数据用 `.big-num`，关键观点用 `<blockquote>`
7. **对比暴力**：好/坏、旧/新用 `.dialog-row.bad`（粉）和 `.good`（绿）并列
8. **零装饰**：不要渐变、半透明、模糊阴影。所有阴影都是硬偏移
9. **术语 = 标签**：技术术语首次出现立刻配一个 `.tag`
10. **装饰性短词保持英文全大写**：`ARCHITECTURE`、`CORE`、`PYTHON` 这类 display 标题 / tag / stamp 不翻译，这是设计系统的一部分；正文跟随用户语言

### Step 4: 判定输出路径并写入文件

按优先级找第一条匹配的：

1. **项目自带配置**：当前目录有 `.repo-explainer.json` 且含 `outDir` 字段，用它
2. **通用 slides 目录**：存在 `docs/slides/` 或 `slides/` 就用
3. **兜底**：当前工作目录

```bash
OUT_DIR="."
if [ -f ".repo-explainer.json" ]; then
  CFG=$(python3 -c "import json;print(json.load(open('.repo-explainer.json')).get('outDir',''))" 2>/dev/null)
  [ -n "$CFG" ] && [ -d "$CFG" ] && OUT_DIR="$CFG"
fi
if [ "$OUT_DIR" = "." ]; then
  if [ -d "docs/slides" ]; then OUT_DIR="docs/slides"
  elif [ -d "slides" ]; then OUT_DIR="slides"; fi
fi
```

**不要硬编码任何人的私人目录结构。** 用户想固定输出位置，让他在项目根目录放一个：

```json
{ "outDir": "3-产出/幻灯片" }
```

**文件名按输出语言取后缀**（语言来自「输入解析」的判定）：

- 中文输出 → `{repo-name}-讲解.html`
- 英文及其他语言 → `{repo-name}-slides.html`

### Step 4.5: 排版自检

模板内置了溢出检测。写完文件后**必须做一次自检**，按能力从高到低选一种：

1. **有浏览器工具时**（`mcp__claude-in-chrome__*`）：打开文件读 console，找 `[repo-explainer]` 开头的消息。报了溢出页就把那几页拆开重写，改完再验一次。
2. **没有浏览器工具时**：按「页面内容硬约束」逐页数一遍——列表条数、代码行数、图表节点数。超标的直接拆页。

无论哪种方式，都在最后告诉用户：**给 URL 加 `?debug=1` 打开，溢出的页会被粉色虚线框标出来**。这是让小白能自己发现并反馈问题的唯一入口，别省掉。

### Step 5: 按平台提示用户打开

**不要自动执行 `open`**（macOS 专属，跨平台会失败）。先用 `uname -s` 检测平台，给命令让用户自己跑：

| 平台 | `uname -s` | 命令 |
|---|---|---|
| macOS | `Darwin` | `open "<绝对路径>"` |
| Linux | `Linux` | `xdg-open "<绝对路径>"` |
| Windows / WSL | `MINGW*` / `CYGWIN*` / `MSYS*` | `start "" "<绝对路径>"` |
| 未知 | 其他 | 直接给路径 |

输出格式：

```
✅ 已生成：/绝对/路径/到/{repo-name}-讲解.html
📄 共 {N} 页 · 路由 { 类型: X, 受众: Y, 重点: Z }
🖥️  打开命令（{平台}）：
    open "/绝对/路径/..."

操作提示：方向键翻页，F 全屏，S 演讲者视图，ESC 总览
排版有问题？在地址栏 URL 后加 ?debug=1 重新打开，溢出的页会标红框
是否要调整某些页面或深入某个模块？
```

## 注意事项

- **DeepWiki MCP 是硬依赖**：见 Step 0，未装就帮用户装并说明要重启会话
- **不要调用 `read_wiki_contents`**：返回全量 wiki 太大，浪费 token。用 `ask_question` 按需获取
- **DeepWiki 只支持公开且已索引的仓库**：未索引引导用户去 deepwiki.com 触发一次；私有仓库需付费版
- **不要手抄模板**：`cp assets/template.html` 再填占位符，不要重新生成 CSS/helper 源码
- **不用 mermaid，不用 rough.js**：图表一律走模板里的 SVG helper
- **图表优先 auto-layout**：`drawFlow/drawStack/drawGrid/drawCompare/drawTimeline` + `diagram()` 包一层；手算坐标是翻车主因
- **画布 1280×760，溢出静默截断**：遵守「页面内容硬约束」，Step 4.5 必须自检
- **语言跟随用户**：正文、标题、代码注释、收尾提示都用用户当前对话语言，`<html lang>` 同步；装饰性短词（display 标题 / tag / stamp）保持英文全大写
- **输出路径**：按 Step 4 判定，不要硬编码任何人的私人目录
- **打开方式**：按 Step 5 给命令，不要自动执行 `open`
- **CDN**：模板内置 jsDelivr → Fastly → unpkg 三级回退，全失败会显示提示横幅。国内用户可能需要代理
- **色号锁死**：Yellow `#FFDD00` / Blue `#2E5EFF` / Pink `#FF3D7F` / Green `#00D26A` / Black `#000` / Paper `#FFFCF0`
- **尺寸与过渡**：`width: 1280, height: 760, margin: 0.05`，`transition: 'none'`，`slideNumber: 'c/t'`（都已在模板里配好，不用改）
