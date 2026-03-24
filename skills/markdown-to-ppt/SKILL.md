---
name: markdown-to-ppt
description: Generate PPT slide content following the AnHeng (安恒信息) corporate template design specification. Produces 5 slide types — cover, table of contents, chapter title, content, and contact page — with precise layout coordinates, colors, fonts, and image placements. Use when the user wants to create a presentation using the AnHeng template.
argument-hint: <title> [--subtitle <subtitle>] [--chapters "Ch1,Ch2,..."] [--output <file>]
allowed-tools: Read, Edit, Write, Glob
version: 1.0.0
---

Generate a PPT presentation following the AnHeng (安恒信息) corporate template specification based on: $ARGUMENTS

If no arguments are provided, ask the user for the presentation title, subtitle, and chapter names.

## Constraints

- **Must use the `pencil` MCP tool** to create and compose all slides. All element placement, styling, and page assembly must be done through pencil tool calls.
- **每页幻灯片必须从零开始创建**，不得复制已有页面或元素后修改。所有元素（背景、文字、图片、形状等）都必须逐一重新创建和定位。

## Specifications

Before creating slides, read the corresponding spec files for detailed layout, positioning, and styling information.

| Spec | File | Description |
|------|------|-------------|
| **Style Guide** | [style-guide.md](specs/style-guide.md) | Canvas size, color palette, typography, spacing, image/shape sizes |
| **Image Assets** | [image-assets.md](specs/image-assets.md) | All image file paths and dimensions (`${CLAUDE_SKILL_DIR}/assets/`) |
| **Page 1 — Cover** | [page-1-cover.md](specs/page-1-cover.md) | Cover page with logo, title, subtitle, building background |
| **Page 2 — TOC** | [page-2-toc.md](specs/page-2-toc.md) | Table of contents with 5 numbered entries and gradient bars |
| **Page 3 — Chapter Title** | [page-3-chapter.md](specs/page-3-chapter.md) | Full-screen chapter title with large number |
| **Page 4 — Content** | [page-4-content.md](specs/page-4-content.md) | Content page with title bar and two-column layout |
| **Page 5 — Contact** | [page-5-contact.md](specs/page-5-contact.md) | Closing page with contact info, QR code, building image |

## Generation Rules

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
3. **Assets required**: List all image files the user needs to provide.
