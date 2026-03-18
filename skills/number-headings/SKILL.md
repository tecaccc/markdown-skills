---
name: number-headings
description: Add hierarchical numbering to Markdown headings. H1 uses Chinese numerals (一、二、三…), H2 and below use Arabic dot-notation (1.1, 1.1.1…). Use when the user wants to number or re-number headings in a Markdown document.
argument-hint: <file.md> [file2.md ...]
allowed-tools: Read, Edit, Write, Glob
version: 1.0.0
---

Add hierarchical numbering to the headings of the Markdown file(s) specified by: $ARGUMENTS

If no file is specified, look for Markdown files (`*.md`) in the current directory and ask the user which one to process (or process all if there's only one).

## Numbering Rules

- **H1 (`#`)** → Chinese numeral + 、: `# 一、Title`
- **H2 (`##`)** → `## 1.1 Title` (Arabic, using the H1 index in Arabic as the first component)
- **H3 (`###`)** → `### 1.1.1 Title`
- **H4 (`####`)** → `#### 1.1.1.1 Title`
- Continue the same dot-notation pattern for H5 and H6.

**Chinese numeral mapping** (for H1 only):

| Index | Chinese |
|-------|---------|
| 1 | 一 |
| 2 | 二 |
| 3 | 三 |
| 4 | 四 |
| 5 | 五 |
| 6 | 六 |
| 7 | 七 |
| 8 | 八 |
| 9 | 九 |
| 10 | 十 |
| 11 | 十一 |
| 12 | 十二 |

For values beyond 12, continue the standard Chinese numeral composition rules.

When a higher-level heading is encountered, reset all lower-level counters to zero.

## Algorithm

1. Read the target file.
2. Process lines one by one, tracking state:
   - Maintain a counter array `[c1, c2, c3, c4, c5, c6]` initialized to all zeros.
   - Track whether we are inside a fenced code block (lines between ` ``` ` or `~~~`). **Skip headings inside code blocks.**
   - Skip YAML front matter (lines between `---` delimiters at the very top of the file).
3. For each heading line outside a code block:
   - Determine the heading level `L` (number of leading `#` characters).
   - Strip any existing numbering from the heading text:
     - For H1: strip a leading Chinese numeral followed by `、` (e.g., `一、`).
     - For H2+: strip a leading Arabic dot-notation number followed by a space (e.g., `1.1 `, `1.2.3 `).
   - Increment `c[L]` by 1.
   - Reset `c[L+1]` through `c[6]` to 0.
   - Build the number prefix:
     - If `L == 1`: convert `c[1]` to its Chinese numeral, format as `{chinese}、` (e.g., `一、`).
     - If `L >= 2`: join `c[1]` through `c[L]` with dots using Arabic numerals (e.g., `1.2.3`), no trailing dot.
   - Reconstruct the line:
     - H1: `# {chinese}、{clean_title}` (e.g., `# 二、实现`)
     - H2+: `## {arabic_prefix} {clean_title}` (e.g., `## 2.1 概述`)
4. Write the updated content back to the file using Edit (preferred) or Write.
5. Report a summary: how many headings were numbered and at what levels.

## Example

**Input:**
```markdown
# Introduction
## Background
## Goals
### Primary Goal
### Secondary Goal
# Implementation
## Phase 1
## Phase 2
```

**Output:**
```markdown
# 一、Introduction
## 1.1 Background
## 1.2 Goals
### 1.2.1 Primary Goal
### 1.2.2 Secondary Goal
# 二、Implementation
## 2.1 Phase 1
## 2.2 Phase 2
```

## Edge Cases

- If the document has no H1, start numbering from the first heading level present. H1-level rules (Chinese numerals) only apply when actual `#` headings exist.
- Do not modify headings inside code blocks (fenced with ` ``` ` or `~~~`).
- If a heading already has numbering (Chinese or Arabic), strip it before applying new numbering (idempotent re-numbering).
