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
   - **Quantitative**: numbers, metrics, percentages → **big-number callout / KPI block**
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

**Per-slide content limits:**
- Title: ≤ 20 Chinese characters / ~40 ASCII
- Subtitle: ≤ 30 Chinese characters
- Body bullets (only when no diagram fits): ≤ 5 bullets, each ≤ 20 Chinese characters
- `--max-bullets <N>` overrides the bullet ceiling

### Phase 4 — 图形优先排版决策 (Diagram-First Layout)

For each content slide, walk this **decision tree in order** — pick the first match.

1. **Is it sequential/process?** → **`flow`** (horizontal or vertical chain of steps with arrows)
2. **Is it hierarchical/tiered?** → **`pyramid`** (3–5 layers, broad base or apex emphasis)
3. **Is it comparative (A vs B)?** → **`comparison`** (two cards side-by-side with parallel rows)
4. **Is it 2×2 trade-off space?** → **`quadrant`** (4 cells, labeled axes)
5. **Is it 3–6 parallel features/capabilities?** → **`icon-grid`** (each item = icon + short label + 1-line description)
6. **Is it a single dominant metric?** → **`big-number`** (giant number/percent + supporting caption)
7. **Is it 2–4 KPIs together?** → **`kpi-row`** (row of big-number cards)
8. **Is it cyclical/loop?** → **`circular-flow`** (3–6 nodes around a circle with arrows)
9. **Is it a timeline (dates / phases over time)?** → **`timeline`** (horizontal axis with milestones)
10. **Is it components with connections (architecture)?** → **`architecture`** (boxes + lines/arrows)
11. **Has it a referenced image asset?** → **`image-left`** / **`image-right`** / **`full-image`**
12. **Is it a short reference table?** → **`table`**
13. **Headline metric + supporting detail below?** → **`top-bottom`**
14. **Single coherent narrative or quote?** → **`single-column`** (last resort for prose; extract a key sentence as the visual hero)
15. **Two parallel ideas not strict A vs B?** → **`two-column`**
16. **None of the above and you have 3+ ideas?** → reconsider the source; you are probably hiding a structure. Re-read and pick a diagram type.

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

For text-based layouts (`single-column`, `two-column`, `top-bottom`, `full-image`, image-left/right), keep using the original `body` bullet format from v1.

### Phase 9 — Show the Plan Summary

Print to the user:

1. **Output file path**.
2. **Slide count** with breakdown by type (Cover/TOC/Chapter/Content/Contact).
3. **Chapter table** — chapter name → number of content slides → dominant layout used.
4. **Diagram coverage** — count of content slides using diagrams vs. text-only bullets, expressed as `X / Y diagrams (Z%)`. If <60%, warn that the deck may look bullet-heavy and suggest re-planning specific slides.
5. **Open questions** — anything you had to guess (missing subtitle, ambiguous chapter grouping, slide where no diagram fit).
6. **Next step**: review/edit the draft, run `/ppt-editor` against the design file, then optionally `/ppt-beautifier` for visual polish.

## Constraints

- **Do not** call the `pencil` MCP tool. This skill only plans; slide creation is `ppt-editor`'s job.
- **Do not** modify the source Markdown. All output goes into the design draft file.
- **Do not** silently truncate content. Split when too dense, and note the split.
- **Do not** invent dates, speakers, statistics, or quotes that are not in the source.
- **Do not** default to bullet lists. Every content slide must pass the diagram-first decision tree (Phase 4) — if you fall back to bullets, justify it in `notes`.
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
