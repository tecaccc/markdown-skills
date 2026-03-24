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

### `/markdown-to-ppt` — AnHeng Corporate PPT Generator

Generate PPT slide content following the AnHeng (安恒信息) corporate template. Produces structured HTML with 5 slide types — cover, table of contents, chapter title, content, and contact page — with precise layout coordinates, colors, fonts, and image placements.

**Usage:**

```
/markdown-to-ppt "演示标题" --subtitle "副标题" --chapters "章节1,章节2,章节3"
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
│   └── markdown-to-ppt/
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
