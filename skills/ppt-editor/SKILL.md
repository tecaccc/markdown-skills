---
name: ppt-editor
description: Generate PPT slide content following the AnHeng (安恒信息) corporate template design specification. Produces 5 slide types — cover, table of contents, chapter title, content, and contact page — with precise layout coordinates, colors, fonts, and image placements. Renders diagram-rich content pages (flow / pyramid / comparison / quadrant / icon-grid / big-number / kpi-row / circular-flow / timeline / architecture / table) when the design draft requests them. Use when the user wants to create a presentation using the AnHeng template.
argument-hint: <title> [--subtitle <subtitle>] [--chapters "Ch1,Ch2,..."] [--output <file>]
allowed-tools: Read, Edit, Write, Glob
version: 2.0.0
---

Generate a PPT presentation following the AnHeng (安恒信息) corporate template specification based on: $ARGUMENTS

If no arguments are provided, ask the user for the presentation title, subtitle, and chapter names.

## Constraints

- **Must use the `pencil` MCP tool** to create and compose all slides. All element placement, styling, and page assembly must be done through pencil tool calls.
- **每页幻灯片必须从零开始创建**，不得复制已有页面或元素后修改。所有元素（背景、文字、图片、形状等）都必须逐一重新创建和定位。
- **每页幻灯片必须放在 pencil 画布的不同位置**，避免堆叠。每页幻灯片（1280×720px）沿垂直方向依次排列，页间间距 100px。即：第 1 页 y=0，第 2 页 y=820，第 3 页 y=1640，第 N 页 y=(N-1)×820。页内所有元素的 y 坐标需加上该页的起始偏移量。

## Specifications

Before creating slides, read the corresponding spec files for detailed layout, positioning, and styling information.

| Spec | File | Description |
|------|------|-------------|
| **Style Guide** | [style-guide.md](specs/style-guide.md) | Canvas size, color palette, typography, spacing, image/shape sizes |
| **Image Assets** | [image-assets.md](specs/image-assets.md) | All image file paths and dimensions (`${CLAUDE_SKILL_DIR}/assets/`) |
| **Page 1 — Cover** | [page-1-cover.md](specs/page-1-cover.md) | Cover page with logo, title, subtitle, building background |
| **Page 2 — TOC** | [page-2-toc.md](specs/page-2-toc.md) | Table of contents with 5 numbered entries and gradient bars |
| **Page 3 — Chapter Title** | [page-3-chapter.md](specs/page-3-chapter.md) | Full-screen chapter title with large number |
| **Page 4 — Content** | [page-4-content.md](specs/page-4-content.md) | Content page with title bar and AI-adaptive layout |
| **Page 5 — Contact** | [page-5-contact.md](specs/page-5-contact.md) | Closing page with contact info, QR code, building image |
| **Diagram Patterns** | [diagram-patterns.md](specs/diagram-patterns.md) | Rendering recipes for all diagram layouts (flow, pyramid, comparison, quadrant, icon-grid, big-number, kpi-row, circular-flow, timeline, architecture, table) plus accent element rendering |

## Workflow

### Phase 1 — 文档分析与内容规划

> **推荐**：复杂文档请先使用 `/ppt-planner <file.md>` 生成 `.ppt-design.md` 设计稿，由用户审阅/修改后再交给本技能。本技能识别到输入是 `.ppt-design.md` 文件时，直接消费其中的每页内容，跳过本阶段的分析。

在使用 pencil 创建之前，必须先完成以下分析（如果输入是 ppt-planner 产出的设计稿则直接读取即可）：

1. **读取并理解文档**：完整阅读用户提供的 Markdown 文档内容。
2. **提取结构**：识别文档标题、副标题、章节划分。
3. **逐页规划内容**：为每一页 PPT 明确分配内容，列出：
   - 页面类型（Cover / TOC / Chapter Title / Content / Contact）
   - 该页包含的具体文字、数据或图片
   - 内容是否需要拆分为多页（单页内容过多时）
4. **输出内容大纲**：将规划结果以清单形式展示给用户确认，再进入创建阶段。

**消费设计稿格式：** 当输入文件以 `.ppt-design.md` 结尾或 frontmatter 中包含 `generated_by: ppt-planner` 时：
- 读取 frontmatter 拿到 `title` / `subtitle` / `chapters`。
- 按 `## Slide <N> — <Type>` 分块解析每页。
- 每页的 `type` / `title` / `subtitle` / `layout` / `body` 字段直接映射到对应的 Page 1–5 spec。
- `layout` 字段决定 Page 4 内容页的排版：
  - **图形类布局**(`flow` / `pyramid` / `comparison` / `quadrant` / `icon-grid` / `big-number` / `kpi-row` / `circular-flow` / `timeline` / `architecture` / `table`) → 参考 [diagram-patterns.md](specs/diagram-patterns.md) 的渲染配方,基于 `diagram` 字段的结构化数据用 pencil 原语(矩形、线条、椭圆、文字)绘制图形。
  - **文字类布局**(`single-column` / `two-column` / `top-bottom` / `image-left` / `image-right` / `full-image`) → 参考 [page-4-content.md](specs/page-4-content.md),基于 `body` 字段渲染。
- **`key_highlight` 字段**:每页核心金句。在图形类布局中渲染为顶部 24pt 加粗 `#08287F` 一行(参考 diagram-patterns.md 末尾说明);在文字类布局中作为视觉焦点放大显示。
- **`accent_elements` 字段**:装饰元素列表。按 [diagram-patterns.md](specs/diagram-patterns.md) "Accent rendering" 章节叠加到所选 layout 之上。
- **`density` 字段**(`sparse` / `medium` / `dense`):调节字号、间距、元素数量。参考 diagram-patterns.md "Density tuning"。
- **`visual_role` / `mood` 字段**:仅用作背景理解,不直接影响坐标;但 `mood: aspirational` 可适度加大主标题字号(+10%)。

### Phase 2 — 使用 Pencil 创建 PPT

确认内容规划后，按以下规则使用 `pencil` MCP tool 逐页创建：

1. **Slide order**: Always generate pages in order — Cover → TOC → Chapter Title(s) → Content(s) → Contact.
2. **Chapter titles**: Generate one Page 3 (chapter title) before each chapter's content pages.
3. **TOC auto-fill**: Populate the 5 TOC entries from the chapter names provided. If fewer than 5 chapters, leave remaining slots with placeholder text. If more than 5, use only the first 5.
4. **Numbering**: TOC uses "01"–"05". Chapter title pages use the same numbering format.
5. **Content pages**: For each chapter, generate at least one Page 4 (content page). Use the chapter name as the page title.
6. **Contact page**: Always the last page. Use the fixed contact information specified above.
7. **Output format**: Use the `pencil` MCP tool to create the presentation, following the coordinates and styles specified above.

## Output

After generation, report:
1. **Slides created**: Total number of slides, with type breakdown.
2. **Chapters**: List of chapter names used.
3. **Diagram coverage**: count of content slides rendered as diagrams vs. text-based, as `X / Y diagrams`.
4. **Assets required**: List all image files the user needs to provide.
5. **Next step**: suggest running `/ppt-beautifier <file.pen>` to polish the result (alignment, spacing, visual hierarchy refinements).
