# Diagram Patterns — Rendering Recipes

This spec tells `ppt-editor` how to render each diagram `layout` produced by `ppt-planner` using pencil primitives (rectangles, lines, text, ellipses). All coordinates assume the standard content area:

```
Content area: x=46, y=220, width=1188, height=440
```

Add the per-page y-offset (page index × 820) to every y value below.

## Color Palette (reuse from style-guide.md)

| Token | Hex | Use |
|-------|-----|-----|
| `brand-blue` | `#08287F` | Primary text, node fills (dark variant) |
| `brand-red` | `#C70019` | Accent, hero number, callouts |
| `divider-red` | `#C00000` | Decorative short lines |
| `node-blue` | `#1E3A8A` | Diagram node fill (medium) |
| `node-purple` | `#7C3AED` | Gradient end / variation |
| `light-bg` | `#F5F5F5` | Card background |
| `border-gray` | `#CCCCCC` | Soft borders, dashed lines |
| `support-blue` | `#4A6FA5` | Captions / supporting text |
| `white` | `#FFFFFF` | Text on dark nodes |

## Universal styling rules

- **Title area** (above all diagrams): use the standard Page 4 title + optional subtitle at the top (x=46, y=120 / y=180).
- **Divider line** (when `accent_elements` includes `divider-line`): 68×5 rect, fill `#C00000`, placed at x=46, y=205, *above* the diagram area.
- **Vertical bar** accent: 6×440 rect, fill `#08287F`, at x=30, y=220.
- **Callout card**: rounded rect (corner radius 8), fill `#F5F5F5`, 1px `#CCCCCC` border, padding 16px.
- **Hero text** (key_highlight rendering): 24pt bold `#08287F`, max width 1100px, placed at y=220 if no diagram precedes it, otherwise below the diagram.

---

## `flow` — Sequential / Process

Horizontal chain (default) of N nodes connected by arrows.

```
Layout (horizontal, N nodes):
  node_width  = floor((1188 - (N-1)×40) / N)   # gap 40px between nodes
  node_height = 120
  y_top       = 270   # diagram vertical center ≈ 330
  arrow_width = 40
```

**Each node**:
- Rect: `node_width × 120`, fill `#08287F`, no border.
- Label: 18pt bold `#FFFFFF`, centered horizontally and vertically inside rect.
- Optional desc: 12pt `#FFFFFF` (60% opacity is fine), below the label, 6px gap.

**Arrow between nodes** (horizontal):
- Line from `(node_right_x, node_center_y)` to `(next_node_left_x − 8, node_center_y)`, 2px stroke `#08287F`.
- Arrowhead: small triangle (8×8) pointing right, fill `#08287F`, at the line's end.

**Vertical variant**: same logic rotated 90°. `node_width=900` centered horizontally, `node_height=80`, gap 24. Arrows point down.

---

## `pyramid` — Hierarchical Layers

3–5 horizontal layers, narrow at top, wide at bottom.

```
Pyramid bounds: x=290 to x=990, y=240 to y=640 (centered, 700×400)
Layer height = 400 / N
Layer i (0 = top): width = 200 + (i × ((700 − 200) / (N − 1)))
                   x_left = (canvas_center − width/2)
                   y_top  = 240 + i × layer_height
```

**Each layer**:
- Trapezoid (approximate using a rect; or use a polygon if pencil supports it). Fill color graded: top layer brand-red `#C70019`, bottom layer brand-blue `#08287F`, intermediate layers linearly interpolated.
- Layer label: 18pt bold `#FFFFFF`, centered.
- Optional desc to the right of the trapezoid: 14pt `#08287F`, x = layer_right + 24.

---

## `comparison` — Two-card Side-by-side

```
Left card:  x=86,  y=240, width=540, height=400
Right card: x=654, y=240, width=540, height=400
Gap:        28px between cards
```

**Each card**:
- Rect fill `#F5F5F5`, 1px border `#CCCCCC`, corner radius 8.
- Top band (60px tall): fill `#08287F` (left) or `#C70019` (right). Card title 20pt bold `#FFFFFF`, centered in band.
- Body: list of rows below the band, each row 24pt of vertical space, 16pt `#08287F`, left padding 24px. Prefix with a small dot (8×8 ellipse, fill matching the band color).

---

## `quadrant` — 2×2 Matrix

```
Diagram bounds: x=140, y=240, width=1000, height=400
Cell size: 500 × 200
Axes drawn at x=640 (vertical) and y=440 (horizontal), 2px stroke #08287F
```

**Cells** (top-left → top-right → bottom-left → bottom-right):
- No fill, just text inside.
- Cell label: 16pt bold `#08287F`, top-left of cell (8px padding).
- Cell items: 14pt `#4A6FA5`, listed below the label.

**Axis labels**:
- y_axis.high: 12pt `#08287F` at (130, 230), right-aligned to x=140.
- y_axis.low:  12pt `#08287F` at (130, 640), right-aligned.
- x_axis.low:  12pt `#08287F` at (140, 650), left-aligned.
- x_axis.high: 12pt `#08287F` at (1140, 650), right-aligned.

---

## `icon-grid` — Parallel Features

```
Columns = configured value (2–4)
Rows    = ceil(items.length / columns)
Cell width  = (1188 − (columns−1)×30) / columns
Cell height = (440 − (rows−1)×30) / rows
```

**Each cell**:
- Rect: fill `#F5F5F5`, corner radius 8, 1px border `#CCCCCC`.
- Top: icon area (60×60 circle, fill `#08287F`, centered horizontally, 24px from top). Icon glyph or single Chinese character inside, 24pt `#FFFFFF` bold.
  - If no real icon asset, render the first char of `icon_hint` (e.g. "盾" for "盾牌").
- Label: 18pt bold `#08287F`, centered horizontally, below icon.
- Desc: 14pt `#4A6FA5`, centered horizontally, max 2 lines, ellipsis if longer.

---

## `big-number` — Hero Metric

```
Number text: x=46, y=240, width=1188, height=280
Font: 微软雅黑, 140pt, Bold
Color: #C70019
Alignment: center
```

**Optional unit** (`%`, `+`, `M` …): 60pt, same color, baseline-aligned beside the number (small superscript offset 20px up).

**Caption** (required): 28pt bold `#08287F`, centered, y=540.

**Support text** (optional): 16pt `#4A6FA5`, centered, y=600.

**Accent**: a 68×5 `#C00000` divider line centered at (640, 590) between caption and support.

---

## `kpi-row` — Multi-metric Row

```
N items (2–4)
Card width  = (1188 − (N−1)×30) / N
Card height = 320
y_top       = 280
```

**Each KPI card**:
- No fill, just a 4px top border `#C70019`.
- Number: 72pt bold `#08287F`, centered, y_top + 30.
- Caption: 18pt `#4A6FA5`, centered, y_top + 130.

---

## `circular-flow` — Cycle

```
Circle center: (640, 440)
Radius:        160
Node count:    N (3–6)
```

**Each node** at angle `θ_i = -90° + i × (360°/N)` (12 o'clock start):
- Node center: `(center_x + R×cos θ, center_y + R×sin θ)`
- Node: 80×80 circle, fill `#08287F`, 2px border `#FFFFFF`.
- Label: 14pt bold `#FFFFFF`, centered inside circle.

**Arrows**: curved arc between consecutive nodes, stroke 2px `#C70019`, arrowhead at the receiving end. If pencil cannot draw curved arrows, use straight chords.

**Center label** (optional): 20pt bold `#08287F`, centered at (640, 440).

---

## `timeline` — Chronological Milestones

```
Horizontal axis: line from (90, 440) to (1190, 440), 3px stroke #08287F
Milestones evenly distributed along axis: x_i = 90 + i × (1100 / (N−1))
```

**Each milestone**:
- Tick: 12×12 circle, fill `#C70019`, border 2px `#FFFFFF`, centered on the axis at (x_i, 440).
- Date label: 14pt bold `#08287F`, above the tick (y=400), centered on x_i.
- Event label: 16pt `#08287F`, below the tick (y=470), centered on x_i, max width = step/0.9.
- Alternate above/below if labels are long, to avoid collision.

---

## `architecture` — Layered Components

```
Layer count: L (typically 2–4)
Layer height = (440 − (L−1)×16) / L
y_top of layer i = 220 + i × (layer_height + 16)
```

**Each layer**:
- Background rect: full content width, fill `#F5F5F5`, 1px border `#CCCCCC`, corner radius 4.
- Layer name (left tab): 100×layer_height rect, fill `#08287F`, label 16pt bold `#FFFFFF` centered (vertical text or rotated if it fits better).
- Items: laid out horizontally inside the layer to the right of the tab. Each item is a 160×56 rect with fill `#FFFFFF`, 1px border `#08287F`, label 14pt `#08287F` centered. Gap 16px between items.

**Connections** (optional): thin 1px arrows between specific items, color `#4A6FA5`.

---

## `table` — Reference Table

```
Headers row: fill #08287F, 16pt bold #FFFFFF, height 40, padding 12px
Body rows:   alternating fill #FFFFFF / #F5F5F5, 14pt #08287F, height 32
Column widths: equal, total 1188
```

Center the table vertically within the content area based on row count.

---

## Layouts inherited from v1 (text-based, unchanged)

`single-column`, `two-column`, `top-bottom`, `image-left`, `image-right`, `full-image` — render as before per [page-4-content.md](page-4-content.md). When the planner specifies `key_highlight` on a text-based layout, render the highlight as a 24pt bold `#08287F` line at the top of the content area (y=220), and shift the body down by 50px.

---

## Accent rendering (orthogonal to layout)

When `accent_elements` includes an item, apply it on top of the chosen layout:

| Accent | Render |
|--------|--------|
| `large-number` | If layout isn't already `big-number`, render the highlight's primary number at 72pt bold `#C70019` in the top-left of the content area; shift the diagram down by 90px. |
| `vertical-bar` | 6×440 rect, fill `#08287F`, at x=30, y=220. |
| `divider-line` | 68×5 rect, fill `#C00000`, at x=46, y=205. |
| `callout-card` | Rounded rect (corner 8), fill `#F5F5F5`, 1px `#CCCCCC` border, around the `key_highlight` text. Padding 16px. |
| `icon` | Pick a single representative character for the icon hint and render as 32pt bold `#08287F` in a 60×60 light circle at the top-left of the content area. |
| `quote-mark` | Render an oversized `"` at 120pt `#C70019` at (46, 220); shift content right by 80px. |
| `corner-accent` | 24×24 right triangle in the top-right corner of the content area, fill `#C70019`. |

## Density tuning

- `sparse`: increase all gap values by 50%; reduce content to highest-priority items only; bias toward `big-number` / `single-column` with hero text.
- `medium`: use values as specified above.
- `dense`: shrink gaps by 25%, allow smaller fonts (drop one size level), allow more items per diagram. Reserve for `architecture`, `table`, multi-row `comparison`.
