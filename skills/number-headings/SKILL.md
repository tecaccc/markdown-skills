---
name: number-headings
description: Normalize and number Markdown headings. First fixes heading level structure (ensures H1 is the top level, no skipped levels), then adds hierarchical numbering: H1 uses Chinese numerals (一、二、三…), H2 and below use Arabic dot-notation (1.1, 1.1.1…). Use when the user wants to number or re-number headings in a Markdown document.
argument-hint: <file.md> [file2.md ...]
allowed-tools: Read, Edit, Write, Glob
version: 1.1.0
---

Normalize heading levels and add hierarchical numbering to the Markdown file(s) specified by: $ARGUMENTS

If no file is specified, look for Markdown files (`*.md`) in the current directory and ask the user which one to process (or process all if there's only one).

## Phase 1 — Normalize Heading Levels

Before numbering, fix the heading structure so it is logically consistent:

### Step 1: Promote to H1

Find the minimum heading level used in the document (ignoring headings inside code blocks and YAML front matter). If the minimum is not H1, shift **all** headings up so the minimum becomes H1.

Example: a document that starts at H2 (`##`) gets every heading promoted by 1 level.

### Step 2: Fix Skipped Levels

After promotion, scan headings in order. A heading may not be more than **1 level deeper** than the previous heading. If a gap exists, reduce the heading's level to be exactly 1 deeper than the previous one.

Rules:
- The first heading in the document is always treated as H1 (already guaranteed by Step 1).
- When a heading drops back to a shallower level, that is always allowed — only deepening by more than 1 is forbidden.

Example of gap fixing:

| Before | After (normalized) | Reason |
|--------|-------------------|--------|
| `# A` | `# A` | — |
| `### B` | `## B` | H3 after H1 → gap of 2, reduce to H2 |
| `#### C` | `### C` | H4 after (normalized) H2 → gap of 2, reduce to H3 |
| `## D` | `## D` | Allowed (shallower) |

> **Important:** Only the `#` prefix is changed. The heading text is never modified during normalization.

---

## Phase 2 — Number Headings

After normalization, apply hierarchical numbering using the rules below.

### Numbering Format

- **H1 (`#`)** → Chinese numeral + 、: `# 一、Title`
- **H2 (`##`)** → `## 1.1 Title`
- **H3 (`###`)** → `### 1.1.1 Title`
- **H4 (`####`)** → `#### 1.1.1.1 Title`
- Continue the same dot-notation pattern for H5 and H6.

**Chinese numeral mapping** (for H1):

| 1 | 2 | 3 | 4 | 5 | 6 | 7 | 8 | 9 | 10 | 11 | 12 |
|---|---|---|---|---|---|---|---|---|----|----|----|
| 一 | 二 | 三 | 四 | 五 | 六 | 七 | 八 | 九 | 十 | 十一 | 十二 |

Continue standard Chinese numeral composition rules beyond 12.

### Numbering Algorithm

1. Maintain a counter array `[c1, c2, c3, c4, c5, c6]` initialized to all zeros.
2. For each heading line (outside code blocks and front matter):
   - Strip any existing numbering:
     - H1: strip leading Chinese numeral + `、` (e.g., `一、`).
     - H2+: strip leading Arabic dot-notation + space (e.g., `1.2 `, `1.2.3 `).
   - Let `L` = heading level after normalization.
   - Increment `c[L]`; reset `c[L+1]` through `c[6]` to 0.
   - Build prefix:
     - `L == 1`: `{chinese}、` (e.g., `三、`)
     - `L >= 2`: `c[1].c[2]...c[L]` joined by dots, no trailing dot (e.g., `2.1.3`)
   - Reconstruct line: `{'#' × L} {prefix} {clean_title}`

---

## Full Example

**Input (disordered levels):**
```markdown
## Overview
#### Background
### Goals
###### Primary Goal
## Implementation
### Phase 1
##### Detail
```

**After Phase 1 — Normalize:**
```markdown
# Overview
## Background
## Goals
### Primary Goal
# Implementation
## Phase 1
### Detail
```
*(H2→H1, H4→H2 because gap from H1, H3→H2 because gap from H1, H6→H3 because gap from H2, H2→H1, H3→H2, H5→H3 because gap from H2)*

**After Phase 2 — Number:**
```markdown
# 一、Overview
## 1.1 Background
## 1.2 Goals
### 1.2.1 Primary Goal
# 二、Implementation
## 2.1 Phase 1
### 2.1.1 Detail
```

---

## Output

After processing, report:
1. **Normalization changes**: list each heading that had its level changed, showing before → after.
2. **Numbering summary**: total headings numbered, breakdown by level.

If no level changes were needed, state "Heading levels are already well-structured."

---

## Edge Cases

- Skip headings inside fenced code blocks (` ``` ` or `~~~`).
- Skip YAML front matter (lines between `---` at the very top of the file).
- If a heading already has numbering (Chinese or Arabic), strip it before re-numbering (idempotent).
- If the document has only one heading level, normalization promotes it to H1 and numbering uses Chinese numerals only.
