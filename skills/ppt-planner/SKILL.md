---
name: ppt-planner
description: Analyze a Markdown document and design a slide-by-slide content plan AND aesthetic plan — chapter division, per-page title/subtitle, **diagram-first content** (flow, matrix, pyramid, icon grid, etc.), key highlights, accent elements, and visual rhythm across the deck. Produces a design draft Markdown file (`<input>.ppt-design.md`) that can be reviewed and edited by the user before being consumed by the `ppt-editor` skill. Use when the user has a Markdown document and wants to plan the PPT structure before generating slides.
argument-hint: <file.md> [--output <file.ppt-design.md>] [--max-bullets <N>] [--style <安恒|generic>]
allowed-tools: Read, Edit, Write, Glob
version: 2.0.0
---

Analyze the Markdown document and design a slide-by-slide content + aesthetic plan for: $ARGUMENTS

If no file is specified, list `*.md` files in the current directory and ask the user which one to plan (skip files ending in `.ppt-design.md` — those are outputs of this skill).

This skill is the **planning stage** that runs **before** `ppt-editor`. It does **not** generate the actual slides — it produces a structured design draft that the user can review/edit and then feed into `ppt-editor`.

## Core Design Principle: 图形优先 (Diagram-First)

**"图是最好的说明文档"**. For every content slide, ask "**can this be a diagram?**" *before* defaulting to bullets. A wall of bullets is the failure mode — flow diagrams, comparison matrices, pyramids, icon grids, big-number callouts, and timelines almost always communicate the same idea more clearly and look far more polished.

Bullets are the *last resort*, not the default. If you produce a content slide whose body is just a flat bullet list, you should be able to defend why no diagram form fits — and write that reason in the slide's `notes` field.

## 强制规则: 数值数据必须图形化 (Numerical Data → Always Visualized)

**任何包含数值的内容都必须以图形方式呈现,绝不能埋在普通正文段落或 bullet 列表中。** 数字埋在文字里 = 设计失败。

判定优先级 (从单到多):

| 数值规模 | 必用展示形式 | 理由 |
|----------|--------------|------|
| 1 个核心数值 | `big-number`(超大字号 + 说明文字) | 给数字最大视觉权重 |
| 2–4 个并列指标 | `kpi-row`(KPI 卡片行) | 多指标横向对比 |
| 5+ 个类别对比(同一维度) | `bar-chart` 柱状图 | 类别间数值对比 |
| 时间序列(>3 个时间点) | `line-chart` 折线图 / `area-chart` 面积图 | 显示趋势走向 |
| 占比 / 构成 (2–6 个部分) | `donut-chart` 环形图(优先) / `pie-chart` 饼图 | 整体的组成关系 |
| 多类别 × 多组件构成 | `stacked-bar` 堆叠柱状图 | 既看总量也看结构 |
| 单一进度 / 完成度 | `progress-bar` 进度条 / `gauge` 仪表盘 | 目标完成情况 |
| 多维度对比(3+ 维度,2–4 主体) | `radar-chart` 雷达图 | 综合素质画像 |
| 两组并列类别对比 | `grouped-bar` 分组柱状图 | 例如"两年同期对比" |

**禁止写法 (反例)**:
> "本季度销售额达 1.2 亿元,环比增长 23%,Q1 为 9750 万,Q2 为 1.05 亿,Q3 为 1.18 亿,Q4 为 1.2 亿。"

**正确做法**:
该段拥有 4 个时间点的数值 → 必须使用 `line-chart` 显示 Q1–Q4 趋势,Q4 末值以加粗节点 + 标签突出。

**例外**: 仅当数值是"附带性说明"(例如版本号、引用编号、不参与对比的常量)时可保留正文形式。判定标准:**该数字是否值得读者关注其量级/对比/趋势?** 若是 → 必须图形化。

## Workflow

### Phase 1 — Read and Analyze the Document

1. **Read the full document.** Do not skim. The plan quality depends on understanding the content as a whole.
2. **Extract structural signals:**
   - Document title (first H1 if present, otherwise filename)
   - Top-level sections (H1 or H2 — see "Chapter detection" below)
   - Sub-sections, lists, tables, code blocks, images, and emphasized text
3. **Identify content shape** in each section (this drives diagram choice in Phase 4):
   - **Sequential / Process**: step 1 → step 2 → step 3, lifecycle, workflow → **flow / timeline**
   - **Hierarchical**: levels, layers, importance order → **pyramid / tree**
   - **Comparative**: A vs B, before/after, options weighed → **comparison cards / quadrant / table**
   - **Parallel features**: 3–6 coordinate items, "key capabilities" → **icon grid / card grid**
   - **Quantitative single value**: one dominant metric → **`big-number`**
   - **Quantitative multi-metric (2–4)**: side-by-side KPIs → **`kpi-row`**
   - **Quantitative comparison (5+ categories)**: ranking/comparing values → **`bar-chart`**
   - **Quantitative time series**: trends over time → **`line-chart`** / **`area-chart`**
   - **Quantitative proportion**: parts of whole → **`donut-chart`** / **`pie-chart`**
   - **Quantitative composition**: multi-component over categories → **`stacked-bar`**
   - **Quantitative single progress**: completion / utilization → **`progress-bar`** / **`gauge`**
   - **Quantitative multi-dimensional**: comparing entities across many axes → **`radar-chart`**
   - **Relational / Architectural**: components and their connections → **architecture diagram / concept map**
   - **Cyclical**: feedback loop, repeating process → **circular flow**
   - **Narrative**: long prose with no inherent structure → **summary + key takeaway**
4. **Estimate density**: count words / list items / table rows per section to decide split vs. merge.

### Phase 2 — Decide Chapter Division

**Chapter detection rule:**

- 2–7 H1 headings → one chapter per H1.
- Only 1 H1 (typical when H1 is the document title) → promote H2 sections to chapters.
- \>7 H1 headings → group into 3–5 chapters by topic, ask user to confirm.
- 0 headings → ask the user to provide chapter names.

**Constraints:**

- AnHeng template TOC supports **up to 5 chapters**. More than 5 → propose merging/splitting, ask user to confirm.
- Each chapter should produce **at least 1 content slide**. If a candidate chapter has almost no body, fold it into a neighbor.

### Phase 3 — Map Content to Slides

For each chapter, walk through its body and decide how many **content slides** it needs.

**Splitting rules:**

| Signal in source | Slide treatment |
|------------------|-----------------|
| A sub-heading (H2/H3 under the chapter) | Start a new content slide; sub-heading becomes the page **subtitle** |
| A list of 3–6 parallel items | Convert to **icon-grid / card-grid** diagram (do NOT use bullets) |
| A list of >7 items | Split across slides AND prefer diagram conversion |
| Steps / numbered procedure | Convert to **flow diagram** |
| Before/after, A vs B | Convert to **comparison cards** or **quadrant** |
| Tiered / ranked / pyramid concepts | Convert to **pyramid diagram** |
| A table (>6 rows or >5 cols) | Dedicate a full slide; consider splitting rows |
| Image already in source | Use **image-left** / **image-right** / **full-image** layout |
| Long paragraph (>120 words) | Extract **one key sentence as headline**, summarize rest into ≤3 short bullets |
| A code block | Single slide, monospace, smaller font |
| Standalone metric / statistic | Convert to **big-number callout** (XL font + supporting label) |
| Numbers in prose / bullets (2+ comparable values) | **Extract into chart** per the 强制规则 table — never leave numbers inline |
| Time series (≥3 time points) | **`line-chart`** or **`area-chart`** — never describe trend in words |
| Percentage breakdown (parts of whole) | **`donut-chart`** (preferred) or **`pie-chart`** |
| Category ranking / comparison | **`bar-chart`** (vertical for ≤6 categories, horizontal for longer labels or 6+ categories) |

**Per-slide content limits:**
- Title: ≤ 20 Chinese characters / ~40 ASCII
- Subtitle: ≤ 30 Chinese characters
- Body bullets (only when no diagram fits): ≤ 5 bullets, each ≤ 20 Chinese characters
- `--max-bullets <N>` overrides the bullet ceiling

### Phase 4 — 图形优先排版决策 (Diagram-First Layout)

For each content slide, walk this **decision tree in order** — pick the first match. **Numerical content always wins** — if the slide contains comparable numbers, jump straight to the numerical branch (steps 1–8).

**Numerical branch (强制优先):**

1. **Single dominant metric?** → **`big-number`** (giant number/percent + supporting caption)
2. **2–4 KPIs together?** → **`kpi-row`** (row of big-number cards)
3. **Single progress / utilization rate?** → **`progress-bar`** or **`gauge`**
4. **Time series (≥3 time points)?** → **`line-chart`** (1–3 series) / **`area-chart`** (cumulative or single-series volume)
5. **Parts of a whole (2–6 categories, sum = 100%)?** → **`donut-chart`** (preferred for modern look) / **`pie-chart`**
6. **Category comparison (≥5 categories, single metric)?** → **`bar-chart`** (vertical default; horizontal if labels are long)
7. **Multi-component breakdown over categories?** → **`stacked-bar`** (each bar shows composition)
8. **Multi-dimensional entity comparison (2–4 entities × 3+ dimensions)?** → **`radar-chart`**

**Structural branch (non-numerical):**

9. **Is it sequential/process?** → **`flow`** (horizontal or vertical chain of steps with arrows)
10. **Is it hierarchical/tiered?** → **`pyramid`** (3–5 layers, broad base or apex emphasis)
11. **Is it comparative (A vs B, non-numerical)?** → **`comparison`** (two cards side-by-side with parallel rows)
12. **Is it 2×2 trade-off space?** → **`quadrant`** (4 cells, labeled axes)
13. **Is it 3–6 parallel features/capabilities?** → **`icon-grid`**
14. **Is it cyclical/loop?** → **`circular-flow`**
15. **Is it a timeline (non-numerical milestones over time)?** → **`timeline`**
16. **Is it components with connections (architecture)?** → **`architecture`**
17. **Has it a referenced image asset?** → **`image-left`** / **`image-right`** / **`full-image`**
18. **Is it a short reference table?** → **`table`**

**Fallback branch (text-based — last resort):**

19. **Headline metric + supporting detail below?** → **`top-bottom`**
20. **Single coherent narrative or quote?** → **`single-column`** (extract a key sentence as the visual hero)
21. **Two parallel ideas not strict A vs B?** → **`two-column`**
22. **None of the above and you have 3+ ideas?** → reconsider the source; you are probably hiding a structure. Re-read and pick a diagram type.

State the chosen **`layout`** explicitly in the design draft so `ppt-editor` can render it.

### Phase 5 — Aesthetic Design (per-slide visual planning)

For **every** slide (especially content slides), specify these aesthetic fields:

#### `key_highlight` — the one thing the audience should remember
- A single sentence, ≤ 25 Chinese characters, that captures the slide's punchline.
- The editor will render this as the visual hero (large font, brand color, possibly on an accent block).
- If you cannot extract a single highlight, the slide is probably too generic — sharpen the scope.

#### `visual_role` — what role this slide plays in the narrative
One of:
- `opening` — sets up a section, mood-heavy, low density
- `concept` — explains an idea (often diagrammatic)
- `data` — leads with a number or chart
- `comparison` — pits two or more things against each other
- `process` — shows steps / flow
- `evidence` — case study, example, screenshot
- `summary` — wraps up, transitions, or calls to action

#### `accent_elements` — decorative cues for the editor
Pick from (one or more):
- `large-number` (giant numeric callout, brand red or brand blue)
- `vertical-bar` (left-edge accent stripe)
- `divider-line` (short red 68×5 line under the title)
- `callout-card` (rounded rectangle highlighting `key_highlight`)
- `icon` (suggest an icon concept — the editor will pick the actual asset/glyph)
- `quote-mark` (oversized opening quote for narrative slides)
- `corner-accent` (small triangle/dot in a corner for rhythm)

#### `density` — visual density target
- `sparse` — ≤ 3 visual elements, heavy whitespace (use for openings, transitions, hero metrics)
- `medium` — 4–7 elements (normal content)
- `dense` — 8+ elements (architecture diagrams, comparison tables — use sparingly)

#### `mood` — emotional tone (chapter-level guidance)
Default to the chapter's mood:
- `serious` (data, compliance, security — cool blues)
- `confident` (capabilities, results — brand red accents)
- `educational` (concept explanations — neutral, generous whitespace)
- `aspirational` (vision, future — gradient backgrounds, larger type)

### Phase 6 — Visual Rhythm Check (deck-level)

Before writing the file, scan the whole sequence and fix rhythm problems:

- **No 3 consecutive content slides should share the same `layout`.** Alternate diagrams to keep the eye fresh.
- **No 3 consecutive content slides should share the same `density`.** Insert a `sparse` slide (often a `big-number` or single key takeaway) between dense ones.
- **Each chapter should open with its strongest visual** — usually a `data` or `concept` slide, not a wall of text.
- **End each chapter with a `summary` slide** if the chapter has ≥3 content slides.
- **The overall deck should follow** Cover (sparse) → TOC (medium) → [Chapter title (sparse) → Content (varied) → ...] → Contact (sparse).

If you have to break a rhythm rule due to source constraints, note it in the affected slide's `notes`.

### Phase 7 — Build the Full Slide Sequence

Assemble in this fixed order:

1. **Cover** (1 slide) — title + subtitle + optional speaker/date.
2. **TOC** (1 slide) — chapter names (≤5; pad with placeholders if fewer).
3. For each chapter:
   - **Chapter Title** (1 slide) — number (01–05) + chapter name.
   - **Content** slides (≥1 each), per Phases 3–5.
4. **Contact** (1 slide) — fixed AnHeng contact info.

Number slides globally starting from 1.

### Phase 8 — Write the Design Draft File

Write to `<input_basename>.ppt-design.md` (or `--output <file>`). Use the exact structure below — `ppt-editor` relies on these field names.

````markdown
---
source: <input filename>
title: <presentation title>
subtitle: <presentation subtitle, or empty>
chapters: ["Ch1", "Ch2", ...]
total_slides: <integer>
style: 安恒
generated_by: ppt-planner
planner_version: 2.0.0
---

# PPT 设计稿 — <presentation title>

> 本文件由 `ppt-planner` 生成,供用户审阅与编辑。确认后请使用 `ppt-editor` 读取本文件生成幻灯片,再可选地用 `ppt-beautifier` 做美化打磨。

## 总览

- **主标题**: <title>
- **副标题**: <subtitle>
- **章节数**: <N>
- **总页数**: <total>
- **章节列表**:
  1. <Ch1>
  2. <Ch2>
  3. ...
- **整体视觉基调 (mood)**: <serious | confident | educational | aspirational>
- **视觉节奏摘要**: <一句话描述全篇排版变化,例如 "数据章节密,案例章节疏,结尾收束">

---

## Slide 1 — Cover

- **type**: cover
- **title**: <主标题>
- **subtitle**: <副标题>
- **key_highlight**: <一句金句作为封面视觉焦点,可选>
- **mood**: <opening 基调>
- **notes**: <可选说明>

---

## Slide 2 — TOC

- **type**: toc
- **entries**:
  - "01" — <Ch1>
  - "02" — <Ch2>
  - "03" — <Ch3>
  - "04" — <Ch4 或占位>
  - "05" — <Ch5 或占位>

---

## Slide 3 — Chapter Title (Chapter 1)

- **type**: chapter
- **number**: "01"
- **chapter_name**: <Ch1>
- **mood**: <chapter mood>
- **key_highlight**: <章节核心立意,可选>

---

## Slide 4 — Content (Chapter 1)

- **type**: content
- **chapter**: <Ch1>
- **title**: <页面标题>
- **subtitle**: <可选副标题>
- **layout**: <layout 名,见 Phase 4 列表>
- **visual_role**: <opening | concept | data | comparison | process | evidence | summary>
- **density**: <sparse | medium | dense>
- **key_highlight**: <本页一句话核心结论>
- **accent_elements**: [<large-number | vertical-bar | divider-line | callout-card | icon | quote-mark | corner-accent>, ...]
- **diagram**: <见下方"Diagram schema",仅当 layout 为图形类时填写>
- **body**: <仅当 layout 为 single-column / two-column / top-bottom 等文字型时填写>
  - <bullet 1>
  - <bullet 2>
- **assets**: <可选,图片路径>
- **notes**: <可选,给生成阶段的提示。如果本页是 fallback 到 bullets,在此说明原因>

---

## Slide N — Contact

- **type**: contact
- **note**: 使用 AnHeng 模板默认联系信息,无需额外内容。
````

#### Diagram schema (per `layout`)

Fill the `diagram` block according to the chosen layout. Keep node labels ≤ 8 Chinese characters; descriptions ≤ 20.

**flow** (sequential steps):
```
- **diagram**:
  - direction: horizontal | vertical
  - nodes:
    - { label: "数据采集", desc: "多源接入" }
    - { label: "数据清洗", desc: "去噪去重" }
    - { label: "数据分析", desc: "模型推理" }
```

**pyramid** (hierarchical layers, top → bottom):
```
- **diagram**:
  - layers:
    - { label: "战略目标", desc: "顶层方向" }
    - { label: "战术规划", desc: "中层落地" }
    - { label: "执行细节", desc: "基础动作" }
```

**comparison** (A vs B):
```
- **diagram**:
  - left:
    - title: "传统方案"
    - rows: ["人工监控", "事后响应", "成本高"]
  - right:
    - title: "AI 方案"
    - rows: ["自动巡检", "实时阻断", "成本降 60%"]
```

**quadrant** (2×2):
```
- **diagram**:
  - x_axis: { low: "成本低", high: "成本高" }
  - y_axis: { low: "价值低", high: "价值高" }
  - cells:
    - q1: { label: "明星", items: ["..."] }
    - q2: { label: "金牛", items: ["..."] }
    - q3: { label: "瘦狗", items: ["..."] }
    - q4: { label: "问号", items: ["..."] }
```

**icon-grid** (3–6 parallel features):
```
- **diagram**:
  - columns: 3
  - items:
    - { icon_hint: "盾牌", label: "主动防御", desc: "..." }
    - { icon_hint: "雷达", label: "持续监测", desc: "..." }
    - { icon_hint: "齿轮", label: "自动响应", desc: "..." }
```

**big-number** (single dominant metric):
```
- **diagram**:
  - number: "98.7%"
  - unit: "%"   # 可选,会以较小字体附在数字旁
  - caption: "威胁拦截准确率"
  - support: "基于 2025 H1 全网数据" # 可选下方小字
```

**kpi-row** (2–4 metrics together):
```
- **diagram**:
  - items:
    - { number: "120+", caption: "服务客户" }
    - { number: "98.7%", caption: "拦截率" }
    - { number: "<5min", caption: "响应时延" }
```

**circular-flow** (cycle / loop):
```
- **diagram**:
  - center_label: "持续改进"   # 可选
  - nodes:
    - { label: "Plan" }
    - { label: "Do" }
    - { label: "Check" }
    - { label: "Act" }
```

**timeline** (dates / phases):
```
- **diagram**:
  - milestones:
    - { date: "2024 Q1", label: "立项" }
    - { date: "2024 Q3", label: "试点" }
    - { date: "2025 Q1", label: "推广" }
```

**architecture** (components + connections):
```
- **diagram**:
  - layers:
    - { name: "接入层", items: ["API 网关", "负载均衡"] }
    - { name: "服务层", items: ["业务服务", "AI 引擎", "规则引擎"] }
    - { name: "数据层", items: ["实时库", "数仓", "对象存储"] }
  - connections: <可选,描述跨层连接>
```

**table** (small reference table):
```
- **diagram**:
  - headers: ["维度", "方案 A", "方案 B"]
  - rows:
    - ["成本", "100 万", "30 万"]
    - ["上线周期", "6 个月", "2 个月"]
```

#### Chart schemas (数值图表)

When the source has numerical data that triggers the numerical branch (Phase 4 steps 1–8), use the schemas below. Every data point must be a real number from the source — never invent values.

**bar-chart** (类别对比):
```
- **diagram**:
  - orientation: vertical | horizontal   # 类别名长时优先 horizontal
  - x_label: "客户行业"   # 类别轴名称(可选)
  - y_label: "签约客户数" # 数值轴名称(可选)
  - unit: "家"             # 数值单位(可选,标注在轴或数据标签上)
  - series:
    - { label: "金融", value: 42 }
    - { label: "政府", value: 38 }
    - { label: "能源", value: 27 }
    - { label: "医疗", value: 19 }
    - { label: "制造", value: 15 }
  - highlight: "金融"      # 可选,该类别将以 #C70019 高亮,其他用 #08287F
```

**grouped-bar** (多组类别对比,如年度对比):
```
- **diagram**:
  - orientation: vertical
  - x_label: "季度"
  - y_label: "营收 (亿元)"
  - groups: ["Q1", "Q2", "Q3", "Q4"]
  - series:
    - { label: "2024", values: [0.85, 1.02, 1.10, 1.18], color: "#4A6FA5" }
    - { label: "2025", values: [0.98, 1.20, 1.35, 1.50], color: "#C70019" }
```

**line-chart** (时间序列趋势):
```
- **diagram**:
  - x_label: "月份"
  - y_label: "DAU (万)"
  - x_categories: ["Jan", "Feb", "Mar", "Apr", "May", "Jun"]
  - series:
    - { label: "活跃用户", values: [12, 14, 18, 23, 29, 35], color: "#08287F" }
    - { label: "付费用户", values: [3, 4, 5, 7, 9, 12], color: "#C70019" } # 可选第二条
  - highlight_point: { x: "Jun", note: "上线推广活动" }   # 可选关键点标注
```

**area-chart** (累积量 / 单序列体量):
```
- **diagram**:
  - x_label: "月份"
  - y_label: "累计交易额 (亿元)"
  - x_categories: ["Jan", "Feb", "Mar", "Apr", "May", "Jun"]
  - values: [1.2, 2.5, 4.1, 6.0, 8.3, 11.0]
  - fill_color: "#08287F"   # 默认品牌蓝,半透明
```

**donut-chart** / **pie-chart** (占比构成):
```
- **diagram**:
  - center_label: "100 亿"   # donut 中心总量(可选,仅 donut)
  - center_caption: "市场规模"   # donut 中心说明(可选)
  - slices:
    - { label: "金融", value: 42, color: "#08287F" }
    - { label: "政府", value: 28, color: "#1E3A8A" }
    - { label: "能源", value: 18, color: "#4A6FA5" }
    - { label: "其他", value: 12, color: "#CCCCCC" }
  - show_percent: true       # 数据标签显示百分比
```

**stacked-bar** (多组件构成 × 多类别):
```
- **diagram**:
  - orientation: vertical
  - x_label: "年份"
  - y_label: "收入构成 (亿元)"
  - categories: ["2022", "2023", "2024", "2025"]
  - components:
    - { label: "产品", values: [3.2, 4.5, 6.1, 8.0], color: "#08287F" }
    - { label: "服务", values: [1.5, 2.2, 3.0, 4.5], color: "#4A6FA5" }
    - { label: "订阅", values: [0.3, 0.8, 1.6, 3.0], color: "#C70019" }
```

**progress-bar** (单一进度):
```
- **diagram**:
  - label: "年度目标完成度"
  - current: 78
  - target: 100
  - unit: "%"
  - color: "#08287F"
  - milestone_label: "Q3 末进度"   # 可选
```

**gauge** (仪表盘式进度):
```
- **diagram**:
  - label: "系统健康度"
  - value: 92
  - max: 100
  - unit: "%"
  - thresholds:
    - { upto: 60, color: "#C70019", label: "风险" }
    - { upto: 80, color: "#E0A030", label: "关注" }
    - { upto: 100, color: "#08287F", label: "健康" }
```

**radar-chart** (多维度对比):
```
- **diagram**:
  - axes: ["性能", "安全", "易用", "成本", "可扩展", "支持"]
  - max_value: 10
  - series:
    - { label: "本方案", values: [9, 9, 8, 7, 9, 8], color: "#C70019" }
    - { label: "友商 A", values: [7, 6, 6, 9, 5, 6], color: "#4A6FA5" }
```

For text-based layouts (`single-column`, `two-column`, `top-bottom`, `full-image`, image-left/right), keep using the original `body` bullet format from v1.

### Phase 9 — Show the Plan Summary

Print to the user:

1. **Output file path**.
2. **Slide count** with breakdown by type (Cover/TOC/Chapter/Content/Contact).
3. **Chapter table** — chapter name → number of content slides → dominant layout used.
4. **Diagram coverage** — count of content slides using diagrams vs. text-only bullets, expressed as `X / Y diagrams (Z%)`. If <60%, warn that the deck may look bullet-heavy and suggest re-planning specific slides.
5. **Numerical visualization audit** — for every numerical value in the source (run a quick scan: digits + `%` + 数字 + 「万/亿/千/百」 + 「倍/次/年」 etc.), verify it appears in a chart/big-number/kpi-row/progress-bar/gauge slide rather than inline text. List any orphan numbers that ended up in bullets or prose and propose which slide to convert.
6. **Open questions** — anything you had to guess (missing subtitle, ambiguous chapter grouping, slide where no diagram fit).
7. **Next step**: review/edit the draft, run `/ppt-editor` against the design file, then optionally `/ppt-beautifier` for visual polish.

## Constraints

- **Do not** call the `pencil` MCP tool. This skill only plans; slide creation is `ppt-editor`'s job.
- **Do not** modify the source Markdown. All output goes into the design draft file.
- **Do not** silently truncate content. Split when too dense, and note the split.
- **Do not** invent dates, speakers, statistics, or quotes that are not in the source.
- **Do not** default to bullet lists. Every content slide must pass the diagram-first decision tree (Phase 4) — if you fall back to bullets, justify it in `notes`.
- **Do not** leave numerical data in prose or bullets. Any number worth comparing must end up in a chart / big-number / kpi-row / progress / gauge slide per the 强制规则 (top of this skill).
- **Do not** invent data points. Chart values must come verbatim from the source; if the source gives only some data points of a series, render only those and add a `notes` entry noting the gap.
- If the source contains `![alt](path)`, preserve those paths in `assets` so they survive into generation.

## Edge Cases

- **Source has no H1/H2 at all**: ask the user for chapter names; offer to auto-split by paragraph clusters.
- **Source already starts with YAML frontmatter**: read `title` / `subtitle` / `author` from it; don't duplicate as headings.
- **Source <200 words**: produce a minimum-viable deck — Cover + TOC + 1 Chapter Title + 1 Content + Contact (5 slides). Still pick a diagram for the content slide.
- **Source needs >5 chapters**: stop, ask the user to choose the top 5.
- **Re-running on a file that already has a `.ppt-design.md` sibling**: warn before overwriting; offer numeric suffix.
- **Slide where no diagram type honestly fits** (e.g., a single dense quote, a philosophical paragraph): use `single-column` with the key sentence enlarged as a hero, add `accent_elements: [quote-mark, divider-line]`, and explain in `notes` why no diagram was chosen.

## Output

The skill produces:

1. **One design draft file** at `<input>.ppt-design.md` (or `--output`).
2. **A short summary** to the conversation (Phase 9).

Nothing else is written or modified.
