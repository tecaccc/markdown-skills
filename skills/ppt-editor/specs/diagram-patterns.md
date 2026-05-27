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

## Chart layouts (数值图表)

Charts share a common plotting area inside the content region:

```
Plot area:  x=120, y=260, width=1040, height=340
Axis stroke: 1.5px, color #4A6FA5
Tick stroke: 1px,   color #CCCCCC
Tick length: 4px
```

Reserve `y=220–250` for an optional chart title, and `y=620–660` for an optional axis label / source caption (10pt `#4A6FA5`).

### `bar-chart` — Category Comparison

**Vertical orientation** (default):
```
N bars, evenly spaced inside the plot area.
bar_width = floor((plot_width - (N+1)×gap) / N)    # gap = 20
bar_x_i   = plot_x + gap + i × (bar_width + gap)
max_value = max(series.value) × 1.1                # 10% headroom
bar_height_i = (value_i / max_value) × plot_height
bar_y_i      = plot_y + (plot_height - bar_height_i)
```
**Each bar**:
- Fill `#08287F` by default; if `highlight == label`, fill `#C70019`.
- Value label above bar: 14pt bold matching bar color, centered on bar.
- Category label below x-axis: 12pt `#08287F`, centered.
- Y-axis: 5 evenly spaced ticks from 0 to ceil(max_value) with grid-line 0.5px `#E5E5E5`.

**Horizontal orientation**: swap x/y. `bar_height=24`, gap=12. Labels right of bars; categories left of plot area, right-aligned.

### `grouped-bar` — Multi-group Comparison

```
G groups × S series per group
slot_width = (plot_width - (G+1)×group_gap) / G    # group_gap = 30
bar_width  = (slot_width - (S+1)×bar_gap) / S      # bar_gap = 4
```
Each series uses its `color`; group label below the slot center, 12pt `#08287F`. Legend in top-right of plot area: small color swatch + 12pt label, one per series, spaced 18px apart.

### `line-chart` — Time Series

```
For each series:
  x_i = plot_x + i × (plot_width / (N-1))
  y_i = plot_y + plot_height - (value_i / max_value) × plot_height
  draw polyline through points, stroke 2.5px, color = series.color
  at each point draw 6×6 circle, fill = series.color, white 1px border
```

- X-axis: tick + label at each `x_i`, 12pt `#08287F`.
- Y-axis: 5 evenly spaced ticks, grid 0.5px `#E5E5E5`.
- **highlight_point** (optional): enlarge that point to 10×10 with brand-red fill, draw a 1px dashed `#C70019` vertical line from point to x-axis, and place the `note` as a 12pt bold `#C70019` callout 16px above the point.
- Legend (only if >1 series): top-right of plot area, same as grouped-bar.

### `area-chart` — Cumulative / Volume

Same as line-chart for one series, but:
- Fill the polygon from the polyline down to the x-axis with `fill_color` at 30% opacity.
- Stroke the top edge 2.5px with `fill_color` at 100% opacity.

### `donut-chart` — Proportion (preferred)

```
center = (640, plot_y + plot_height/2)   # ≈ (640, 430)
outer_radius = 150
inner_radius = 90       # donut hole
```
For each slice with cumulative angle `θ_start → θ_end` (start at -90°, sweep clockwise = value/total × 360°):
- Draw an arc-segment (annular sector) from `θ_start` to `θ_end`, fill = slice.color.
- If the pencil tool cannot draw true arcs, approximate by tessellating into 6° triangular wedges.
- Slice label: outside the donut at the midpoint angle, 14pt `#08287F`. If `show_percent`, append `(XX%)` in 12pt.
- Lead line: 1px `#CCCCCC` from outer edge to label start when label is more than 8px from arc midpoint.

**Center text** (donut only):
- `center_label`: 28pt bold `#08287F`, centered.
- `center_caption`: 12pt `#4A6FA5`, centered, 6px below center_label.

### `pie-chart` — Proportion (legacy)

Same as `donut-chart` but `inner_radius = 0` (no hole, no center text). Use only when the planner explicitly requests pie.

### `stacked-bar` — Multi-component Composition

```
N category columns
column_width = (plot_width - (N+1)×gap) / N        # gap = 30
For each column k:
  total_k = sum(components[*].values[k])
  stack components bottom-up:
    seg_height = (value / max_total) × plot_height
    seg fill   = component.color
```
- Total label above each column: 14pt bold `#08287F`.
- Optional value-in-segment label: 11pt `#FFFFFF` centered, only if segment height ≥ 32px.
- Legend top-right.

### `progress-bar` — Single Progress

```
Track: x=160, y=400, width=960, height=40
       fill #F5F5F5, 1px border #CCCCCC, corner radius 20
Fill:  same y/height, width = (current/target) × track_width
       fill = color (default #08287F), corner radius 20
```
- `label`: 18pt bold `#08287F`, x=160, y=350.
- Current/target text: right-aligned at (1120, 350), 16pt `#08287F` — format `78% / 100%` or `78 / 100 (unit)`.
- `milestone_label` (optional): 12pt `#C70019`, placed below the fill end at (fill_end_x, 450), centered.

### `gauge` — Semi-circular Gauge

```
center = (640, 480)
outer_radius = 200, inner_radius = 160        # arc thickness 40
sweep: -180° (left) → 0° (right), total 180°
```

**Track**: draw the full 180° annular arc, fill `#F5F5F5`.

**Thresholds**: split the 180° arc into segments per `thresholds[]`. For each threshold, the arc from previous upto to current upto is filled with its color.

**Needle**: line from center pointing to angle = `-180° + (value/max) × 180°`, length = `inner_radius - 10`, stroke 4px `#08287F`, with a 14×14 circle at center, fill `#08287F`.

**Value label**: `{value}{unit}` at (640, 470), 40pt bold `#08287F`, centered.
**Caption**: `label` at (640, 540), 16pt `#4A6FA5`, centered.
**Threshold tick labels**: tiny 10pt `#4A6FA5` at each threshold boundary, outside the outer radius.

### `radar-chart` — Multi-dimensional

```
center = (640, 440)
radius = 180
axes count = A
```
For each axis i: direction angle `θ_i = -90° + i × (360°/A)`.

**Background grid**: 4 concentric polygons at 25/50/75/100% of `max_value`, stroke 1px `#E5E5E5`.

**Axis spokes**: line from center to outer vertex for each axis, stroke 1px `#CCCCCC`.

**Axis labels**: 12pt `#08287F` just outside outer vertex, anchored based on quadrant.

**Each series**:
- Compute vertex for each axis: `vertex_i = center + (value_i/max_value) × radius × (cos θ_i, sin θ_i)`.
- Draw closed polygon through vertices, stroke 2px `series.color`, fill `series.color` at 20% opacity.
- Vertex dots: 5×5 circles, fill `series.color`.

Legend top-right.

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
