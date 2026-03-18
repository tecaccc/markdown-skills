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

## Plugin Structure

```
markdown-skills/
├── .claude-plugin/
│   └── plugin.json          # Plugin metadata
├── skills/
│   └── number-headings/
│       └── SKILL.md         # Heading numbering skill
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
