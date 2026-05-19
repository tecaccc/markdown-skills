---
name: ppt-planner
description: Analyze a Markdown document and design the page-by-page content plan for a slide deck — chapter division, per-page title/subtitle/body, suggested layout for each content page, and TOC entries. Produces a design draft Markdown file (`<input>.ppt-design.md`) that can be reviewed and edited by the user before being consumed by the `ppt-editor` skill. Use when the user has a Markdown document and wants to plan the PPT structure before generating slides.
argument-hint: <file.md> [--output <file.ppt-design.md>] [--max-bullets <N>] [--style <安恒|generic>]
allowed-tools: Read, Edit, Write, Glob
version: 1.0.0
---

Analyze the Markdown document and design a slide-by-slide content plan for: $ARGUMENTS

If no file is specified, list `*.md` files in the current directory and ask the user which one to plan (skip files ending in `.ppt-design.md` — those are outputs of this skill).

This skill is the **planning stage** that should run **before** `ppt-editor`. It does **not** generate the actual slides — it produces a structured design draft that the user can review/edit and then feed into `ppt-editor`.

## Workflow

### Phase 1 — Read and Analyze the Document

1. **Read the full document.** Do not skim. The plan quality depends on understanding the content as a whole.
2. **Extract structural signals:**
   - Document title (first H1 if present, otherwise filename)
   - Top-level sections (H1 or H2 — see "Chapter detection" below)
   - Sub-sections, lists, tables, code blocks, images, and emphasized text
3. **Identify content types** in each section:
   - **Narrative**: long paragraphs of prose
   - **Enumerated**: bulleted/numbered lists
   - **Comparative**: tables, side-by-side concepts
   - **Visual**: images, diagrams, screenshots referenced
   - **Data**: numbers, metrics, statistics
4. **Estimate density**: roughly count words / list items / table rows per section to decide whether one slide is enough.

### Phase 2 — Decide Chapter Division

**Chapter detection rule:**

- If the document has **2–7 H1 headings**, treat each H1 as one chapter.
- If the document has **only 1 H1** (typical when the H1 is the document title), promote H2 sections to chapters.
- If the document has **>7 H1 headings**, group them by topic into 3–5 chapters (ask the user to confirm the grouping).
- If the document has **0 headings**, ask the user to provide chapter names.

**Constraints:**

- The AnHeng template's TOC supports **up to 5 chapters**. If the plan has more than 5, propose merging or splitting and ask the user to confirm before continuing.
- Each chapter should produce **at least 1 content slide**. If a candidate chapter has almost no body, fold it into a neighbor.

### Phase 3 — Map Content to Slides

For each chapter, walk through its body and decide how many **content slides** it needs. Use these rules:

**Splitting rules:**

| Signal in source | Slide treatment |
|------------------|-----------------|
| A sub-heading (H2/H3 under the chapter) | Start a new content slide; sub-heading becomes the page **subtitle** |
| A list of >7 items | Split into multiple slides (≤7 bullets/slide), or use a 3-column / 2-column layout |
| A wide/long table (>6 rows or >5 cols) | Either dedicate a full slide to the table, or split rows across slides |
| An image referenced in the body | The image gets its own region; pick a "left-image-right-text" or "full-image" layout |
| A long paragraph (>120 words) | Summarize into 3–5 bullet points; do not paste prose verbatim |
| A code block | Keep as a single slide; prefer monospace, smaller font |

**Per-slide content limits** (used to decide whether to split):

- Title: ≤ 20 Chinese characters / ~40 ASCII characters
- Subtitle: ≤ 30 Chinese characters
- Body: ≤ 7 bullets, each ≤ ~25 Chinese characters, OR ≤ ~150 words of prose
- If `--max-bullets <N>` is provided, use `N` as the bullet ceiling.

### Phase 4 — Pick a Layout for Each Content Slide

For each content slide, choose a layout from the table below based on the content's shape. This matches the layout options the `ppt-editor` content page (page-4-content) supports.

| Layout | When to use |
|--------|-------------|
| `single-column` | A single coherent narrative, a single bullet list ≤ 7 items, or a quote |
| `two-column` | Two parallel ideas, before/after, comparison of two items |
| `three-column` | 3 parallel key points or feature highlights |
| `image-left` / `image-right` | One image plus accompanying text |
| `top-bottom` | A headline/metric on top + supporting details below |
| `full-image` | An architecture diagram, full-bleed screenshot, or chart |
| `table` | A small reference table (≤ 6 rows × ≤ 5 cols) shown verbatim |

State the chosen layout explicitly in the design draft so `ppt-editor` can pick it up.

### Phase 5 — Build the Full Slide Sequence

Assemble the slides in this fixed order, matching the AnHeng template:

1. **Cover** (1 slide) — title + subtitle + (optional) speaker/date.
2. **TOC** (1 slide) — chapter names (up to 5; pad with placeholders if fewer).
3. For each chapter:
   - **Chapter Title** (1 slide) — chapter number (01–05) + chapter name.
   - **Content** slides (≥1 each), per Phase 3 + Phase 4.
4. **Contact** (1 slide) — fixed AnHeng contact info; no analysis needed.

Number each slide globally starting from 1.

### Phase 6 — Write the Design Draft File

Write the output to `<input_basename>.ppt-design.md` next to the source file (or to `--output <file>` if specified). Use the exact structure below — `ppt-editor` relies on these field names.

````markdown
---
source: <input filename>
title: <presentation title>
subtitle: <presentation subtitle, or empty>
chapters: ["Ch1", "Ch2", ...]
total_slides: <integer>
style: 安恒
generated_by: ppt-planner
---

# PPT 设计稿 — <presentation title>

> 本文件由 `ppt-planner` 生成，供用户审阅与编辑。确认后请使用 `ppt-editor` 读取本文件以生成幻灯片。

## 总览

- **主标题**: <title>
- **副标题**: <subtitle>
- **章节数**: <N>
- **总页数**: <total>
- **章节列表**:
  1. <Chapter 1 name>
  2. <Chapter 2 name>
  3. ...

---

## Slide 1 — Cover

- **type**: cover
- **title**: <主标题>
- **subtitle**: <副标题>
- **notes**: <可选，给设计稿读者的一行说明>

---

## Slide 2 — TOC

- **type**: toc
- **entries**:
  - "01" — <Chapter 1>
  - "02" — <Chapter 2>
  - "03" — <Chapter 3>
  - "04" — <Chapter 4 或占位>
  - "05" — <Chapter 5 或占位>

---

## Slide 3 — Chapter Title (Chapter 1)

- **type**: chapter
- **number**: "01"
- **chapter_name**: <Chapter 1 name>

---

## Slide 4 — Content (Chapter 1)

- **type**: content
- **chapter**: <Chapter 1 name>
- **title**: <页面标题>
- **subtitle**: <可选副标题>
- **layout**: <single-column | two-column | three-column | image-left | image-right | top-bottom | full-image | table>
- **body**:
  - <bullet 1>
  - <bullet 2>
  - ...
- **assets**: <可选，列出本页用到的图片相对路径>
- **notes**: <可选，给生成阶段的提示>

---

## Slide 5 — Content (Chapter 1)

...

---

## Slide N — Contact

- **type**: contact
- **note**: 使用 AnHeng 模板默认联系信息，无需额外内容。
````

**Field rules:**

- Each slide block starts with `## Slide <N> — <Type>` and is followed by `---` on its own line.
- Field names (`type`, `title`, `layout`, etc.) are lowercase, written as bold `**field**` followed by `: value`.
- `body` is always a Markdown bullet list — one bullet per planned point.
- For `two-column` / `three-column` / image layouts, structure `body` as nested bullets:
  ```
  - **左栏**:
    - <point>
    - <point>
  - **右栏**:
    - <point>
  ```
- Never invent content that is not supported by the source document. If a slot is empty (e.g., no subtitle), leave it blank rather than fabricating.

### Phase 7 — Show the Plan Summary

After writing the file, print a short summary to the user:

1. **Output file path**.
2. **Slide count** with a breakdown by type (Cover/TOC/Chapter/Content/Contact).
3. **Chapter table** (chapter name → number of content slides).
4. **Open questions** — anything you had to guess at (e.g., missing subtitle, ambiguous chapter grouping). Ask the user to review.
5. **Next step**: tell the user to review/edit the draft, then run `/ppt-editor` against the design file when ready.

## Constraints

- **Do not** call the `pencil` MCP tool. This skill only plans content; slide creation is `ppt-editor`'s job.
- **Do not** modify the source Markdown. All output goes into the design draft file.
- **Do not** silently truncate content. If a section is too dense for one slide, split it and note the split in the plan.
- **Do not** invent dates, speakers, statistics, or quotes that are not in the source.
- If the source contains images via `![alt](path)`, preserve those paths in the design draft's `assets` field so they survive into the generation stage.

## Edge Cases

- **Source has no H1/H2 at all**: ask the user for chapter names; offer to auto-split by paragraph clusters as a fallback.
- **Source already starts with a YAML frontmatter**: read `title`, `subtitle`, `author` from frontmatter if present; do not duplicate them as headings.
- **Source is shorter than ~200 words**: produce a minimum-viable deck — Cover + TOC (single chapter) + 1 Chapter Title + 1 Content + Contact (5 slides total).
- **Source is very long (>5 chapters needed)**: stop and ask the user to choose the top 5 chapters to keep, rather than silently dropping content.
- **Re-running on a file that already has a `.ppt-design.md` sibling**: warn before overwriting; offer to write to a new path with a numeric suffix.

## Output

The skill produces:

1. **One design draft file** at `<input>.ppt-design.md` (or the `--output` path).
2. **A short summary** to the conversation, as described in Phase 7.

Nothing else is written or modified.
