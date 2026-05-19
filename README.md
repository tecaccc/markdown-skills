# markdown-skills

A Claude Code plugin providing skills for working with Markdown documents.

## Installation

```
/plugin install markdown-skills@yourusername/markdown-skills
```

Or browse via `/plugin > Discover`.

## Skills

### `/number-headings` — Heading Numbering

Automatically adds hierarchical numbering to Markdown headings.

- **H1** uses Chinese numerals: `# 一、Introduction`
- **H2+** uses Arabic dot-notation: `## 1.1 Background`, `### 1.1.1 Details`
- Idempotent: re-running strips old numbers and applies fresh ones
- Skips headings inside fenced code blocks and YAML front matter

**Usage:**

```
/number-headings path/to/document.md
/number-headings doc1.md doc2.md
```

**Example:**

| Before | After |
|--------|-------|
| `# Introduction` | `# 一、Introduction` |
| `## Background` | `## 1.1 Background` |
| `### Details` | `### 1.1.1 Details` |
| `# Implementation` | `# 二、Implementation` |
| `## Phase 1` | `## 2.1 Phase 1` |

### `/ppt-planner` — PPT 内容设计

读取一个 Markdown 文档，分析其结构后**为每一页 PPT 规划内容**：章节划分、每页的标题/副标题/正文、推荐的排版方式、TOC 条目等。产出一个 `<input>.ppt-design.md` 设计稿，用户可审阅/修改后交给 `/ppt-editor` 生成幻灯片。

这是 `/ppt-editor` 推荐的前置步骤——先把"每一页装什么"想清楚，再开始排版。

**Usage:**

```
/ppt-planner path/to/document.md
/ppt-planner path/to/document.md --output deck-plan.ppt-design.md --max-bullets 5
```

**输出：** 一份结构化的设计稿 markdown 文件，包含每页的 `type` / `title` / `subtitle` / `layout` / `body` 字段，`ppt-editor` 可直接消费。

### `/ppt-editor` — AnHeng Corporate PPT Generator

Generate PPT slide content following the AnHeng (安恒信息) corporate template. Produces structured HTML with 5 slide types — cover, table of contents, chapter title, content, and contact page — with precise layout coordinates, colors, fonts, and image placements.

**Usage:**

```
/ppt-editor "演示标题" --subtitle "副标题" --chapters "章节1,章节2,章节3"
/ppt-editor deck-plan.ppt-design.md     # 直接消费 ppt-planner 产出的设计稿
```

**Slide Types:**

| Type | Description |
|------|-------------|
| Cover Page | Logo, main title, subtitle, contact info |
| Table of Contents | Up to 5 numbered chapter entries |
| Chapter Title | Full-screen background with chapter number and title |
| Content Page | Title bar with two-column content boxes |
| Contact Page | Contact info, QR code, company info |

## Plugin Structure

```
markdown-skills/
├── .claude-plugin/
│   ├── plugin.json            # Plugin metadata
│   └── marketplace.json       # Marketplace configuration
├── skills/
│   ├── number-headings/
│   │   └── SKILL.md           # Heading numbering skill
│   ├── ppt-planner/
│   │   └── SKILL.md           # PPT content planning skill
│   └── ppt-editor/
│       └── SKILL.md           # PPT generation skill
├── README.md
└── LICENSE
```

## Contributing

Contributions are welcome. To add a new skill:

1. Create a directory under `skills/<skill-name>/`
2. Add a `SKILL.md` with the required frontmatter (`name`, `description`)
3. Update this README with the skill's documentation
4. Open a pull request

## License

MIT
