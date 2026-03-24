---
name: markdown-to-ppt
description: Generate PPT slide content following the AnHeng (安恒信息) corporate template design specification. Produces structured Markdown or HTML representing 5 slide types — cover, table of contents, chapter title, content, and contact page — with precise layout coordinates, colors, fonts, and image placements. Use when the user wants to create a presentation using the AnHeng template.
argument-hint: <title> [--subtitle <subtitle>] [--chapters "Ch1,Ch2,..."] [--output <file>]
allowed-tools: Read, Edit, Write, Glob
version: 1.0.0
---

Generate a PPT presentation following the AnHeng (安恒信息) corporate template specification based on: $ARGUMENTS

If no arguments are provided, ask the user for the presentation title, subtitle, and chapter names.

---

## Canvas

```
Width:  1280px
Height: 720px
Ratio:  16:9
```

---

## Color Palette

| Name | Hex | RGB | Usage |
|------|-----|-----|-------|
| **Brand Dark Blue** | `#08287F` | rgb(8, 40, 127) | Main titles, important text |
| **Title Blue** | `#03277F` | rgb(3, 39, 127) | Section title prefix |
| **Brand Red** | `#C70019` | rgb(199, 0, 25) | Accent, numbering blocks |
| **Divider Red** | `#C00000` | rgb(192, 0, 0) | Decorative lines |
| **White** | `#FFFFFF` | rgb(255, 255, 255) | Text on dark backgrounds |
| **Light Gray BG** | `#F5F5F5` | — | Page background |

---

## Typography

### Font Family
- **Primary**: 微软雅黑 (Microsoft YaHei)
- **Secondary**: Agency FB (section title prefix)

### Size Hierarchy

| Level | Size | Bold | Usage |
|-------|------|------|-------|
| **XL** | 60pt | Yes | Chapter large number |
| **H1** | 37pt | Yes | Cover main title |
| **H2** | 32pt | Yes | Page title, chapter title, TOC title |
| **H3** | 24pt | Yes | Contact page title |
| **H4** | 21pt | Yes | Subtitle |
| **Body** | 16pt | No | TOC text, body content |
| **Subtitle** | 15pt | Yes | Cover subtitle |
| **Caption** | 11pt | No | Contact info |
| **Small** | 8pt | No | Annotations, copyright |

---

## Image Assets

### Backgrounds

| Filename | Original Size | Purpose | Pages |
|----------|--------------|---------|-------|
| **公司大楼.jpeg** | 1665×1649px | Right-side decorative BG (blue-red gradient overlay) | Page 1, 5 |
| **标题背景.jpeg** | 2933×289px | Top title bar (purple-blue gradient) | Page 2, 4 |
| **章节标题页背景.jpeg** | 2933×1650px | Full-screen BG (dark blue + red curves) | Page 3 |

### Logos & Decorations

| Filename | Original Size | Purpose | Pages |
|----------|--------------|---------|-------|
| **亚奥理事会合作Logo.png** | 725×408px | Top-left company logo | Page 1, 5 |
| **标题背景文字.png** | 915×126px | White text logo on title bar right side | Page 2, 4 |
| **安恒信息文字Logo.png** | 304×118px | Top-left white logo | Page 3 |
| **安恒信息官方微信二维码.jpeg** | 198×198px | QR code | Page 5 |

---

## Slide Types

### Page 1 — Cover Page

**Layout:**
```
Background: Light gray
Right side: 公司大楼.jpeg (blue-red gradient) fills right ~50-55%
Top-left:   亚奥理事会合作Logo.png
Left side:  Text content area
Bottom:     Contact info
```

**Logo (top-left):**
- Image: 亚奥理事会合作Logo.png
- Size: ~180-200px × 100-110px
- Position: x=40-60px, y=30-50px

**Right background:**
- Image: 公司大楼.jpeg
- Starts at ~x=640px, full height (720px), occupies right ~50-55%

**Main title:**
- Position: x=50px, y=200px
- Font: 微软雅黑, 37pt, Bold
- Color: #08287F

**Subtitle / Speaker info:**
- Position: x=50px, y=400px
- Font: 微软雅黑, 15pt, Bold
- Color: #08287F

**Bottom contact:**
- Position: x=50px, y=660px
- Font: 微软雅黑, 11pt
- Color: #08287F
- Text: "www.dbappsecurity.com.cn    400-6059-110"

---

### Page 2 — Table of Contents

**Layout (three-column):**
```
Top:    标题背景.jpeg (full-width gradient bar)
        标题背景文字.png (right side of bar)

Body (left → right):
  ├─ Left:   "目录" large text (right-aligned)
  ├─ Center: Vertical divider line
  └─ Right:  5 TOC entries
```

**Title bar:**
- 标题背景.jpeg: 1280px × 75-80px, y=0
- 标题背景文字.png: ~280-300px × 40-45px, right-aligned, ~20-30px from right edge

**"目录" text (left area):**
- Position: x≈200-280px (right-aligned to divider), y≈240-260px
- Font: 微软雅黑, 32pt, Bold
- Color: #08287F
- Alignment: Right

**Vertical divider:**
- Position: x≈320-350px
- Y range: 200px → 680px (height ~480px)
- Style: 2-3px, color #CCCCCC or #E0E0E0

**TOC entries (5 items):**
- Start: x≈400px, y≈240px
- Vertical spacing: ~77px center-to-center
- Item height: 52px

Each entry structure (left → right):

1. **Circle dot**: 20×20px, white fill, #08287F border 2px
2. **Dashed line**: ~60-80px, thin dashed, #CCCCCC
3. **Red number block**: 52×52px, fill #C70019, no rounded corners
4. **Number text** (inside block): 微软雅黑 16pt, white, centered — "01"–"05"
5. **Gradient bar**: starts right of red block, ~620px wide × 52px, left-to-right gradient (deep blue #1E3A8A → purple #7C3AED)
6. **Entry text** (inside bar): 微软雅黑 16pt, white, left-aligned, ~20px left padding

---

### Page 3 — Chapter Title Page

**Layout:**
```
Background: 章节标题页背景.jpeg (full-screen)
Top-left:   安恒信息文字Logo.png
Center-left: Large chapter number + title
```

**Background:**
- 章节标题页背景.jpeg: 1280×720px, fill entire canvas

**Logo:**
- 安恒信息文字Logo.png: ~90-100px × 35-40px, position x=40-50px, y=30-50px

**Chapter number:**
- Position: x=100px, y=295px
- Font: 微软雅黑, 60pt, Bold
- Color: #FFFFFF

**Chapter title:**
- Position: x=350px, y=312px
- Font: 微软雅黑, 32pt, Bold
- Color: #FFFFFF
- Alignment: Left

---

### Page 4 — Content Page

**Layout:**
```
Top:  标题背景.jpeg + 标题背景文字.png (same as Page 2)
Body: Title area + Content area
```

**Title bar:** Same as Page 2.

**Page title:**
- Position: x=46px, y=120px
- Font: 微软雅黑, 32pt, Bold
- Color: #08287F

**Subtitle:**
- Position: x=46px, y=180px
- Font: 微软雅黑, 21pt, Bold
- Color: #08287F

**Content boxes (two-column):**

| Box | Position | Size | Fill |
|-----|----------|------|------|
| Left | x=72, y=220 | 423×272px | #08287F |
| Right | x=625, y=220 | 423×272px | #08287F |

- Text inside: 微软雅黑 16pt, white, padding ~20-30px

---

### Page 5 — Contact / Closing Page

**Layout (left-right split):**
```
Left area (~45-50% width):
  ├─ Top-left:    亚奥理事会合作Logo.png
  ├─ Center:      Contact info
  ├─ Center-low:  QR code
  └─ Bottom-left: Website & phone

Right area (~50-55% width):
  └─ 公司大楼.jpeg (independent, no overlap with text)
```

**Logo:** Same as Page 1.

**"联系我们" title:**
- Position: x≈300px (centered in left area), y≈270px
- Font: 微软雅黑, 24pt, Bold
- Color: #08287F

**"Contact Us" subtitle:**
- Position: x≈330px, y≈310px
- Font: 微软雅黑, 11pt
- Color: #08287F

**Red divider line:**
- Position: x≈300px (centered), y≈345px
- Size: 68px × 5px
- Fill: #C00000

**Company address:**
- Position: x≈300px (centered), y≈365px
- Font: 微软雅黑, 11pt, Color: #08287F
- Text: "浙江省杭州市滨江区西兴街道联慧街188号安恒大厦"
- Max width: ~480px

**Website & phone:**
- Position: x≈300px (centered), y≈390px
- Font: 微软雅黑, 11pt, Color: #08287F
- Text (two lines): "www.dbappsecurity.com.cn" / "400-6059-110"

**QR code:**
- Image: 安恒信息官方微信二维码.jpeg
- Position: x≈270px (centered in left area), y≈470px
- Size: 87×87px

**QR label:**
- Position: x≈300px, y≈565px
- Font: 微软雅黑, 8pt, Color: #08287F
- Text: "安恒信息官方微信"

**Bottom contact:**
- Position: x=50px, y=660px
- Font: 微软雅黑, 11pt, Color: #08287F
- Text: "www.dbappsecurity.com.cn    400-6059-110"

**Right side building image:**
- Image: 公司大楼.jpeg
- Start: x≈640px, width≈640px, height=720px (full height)
- Right-aligned, with blue-red gradient decoration
- Must NOT overlap with left-side text

---

## Standard Spacing Reference

| Item | Size |
|------|------|
| Page margin (left/right) | 40-60px |
| Page margin (top/bottom) | 30-50px |
| Title-to-content gap | 15-25px |
| TOC item vertical spacing | 77px |
| Divider-to-"目录" text gap | 40-60px |
| Divider-to-TOC entries gap | 50-80px |

## Image Display Sizes

| Image | Suggested Size |
|-------|---------------|
| 亚奥理事会Logo | 180-200px × 100-110px |
| 标题背景 | 1280px × 75-80px |
| 标题文字Logo | 280-300px × 40-45px |
| 安恒文字Logo | 90-100px × 35-40px |
| 公司大楼背景 | 640px × 720px |
| 章节背景 | 1280px × 720px |
| 二维码 | 87px × 87px |

## Shape Sizes

| Element | Size |
|---------|------|
| TOC number block | 52×52px |
| Decorative dot | 20×20px |
| Divider line | 68×5px |
| Vertical line (TOC) | 2-3px × 480px |
| Content box | 423×272px |

---

## Generation Rules

1. **Slide order**: Always generate pages in order — Cover → TOC → Chapter Title(s) → Content(s) → Contact.
2. **Chapter titles**: Generate one Page 3 (chapter title) before each chapter's content pages.
3. **TOC auto-fill**: Populate the 5 TOC entries from the chapter names provided. If fewer than 5 chapters, leave remaining slots with placeholder text. If more than 5, use only the first 5.
4. **Numbering**: TOC uses "01"–"05". Chapter title pages use the same numbering format.
5. **Content pages**: For each chapter, generate at least one Page 4 (content page). Use the chapter name as the page title.
6. **Contact page**: Always the last page. Use the fixed contact information specified above.
7. **Output format**: Generate structured HTML with inline styles matching the coordinates and styles above, or Markdown with embedded layout comments, depending on user preference. Default to HTML for pixel-accurate layout.

---

## Output

After generation, report:
1. **Slides created**: Total number of slides, with type breakdown.
2. **Chapters**: List of chapter names used.
3. **Assets required**: List all image files the user needs to provide.
