# Layout Fundamentals

Shared rules for **all diagram types**. Read this before drawing any SVG.

## 1. Uniform Grid Layout (all diagram types)

**Every diagram** uses a computed grid. Coordinates come from **cell indices** — never hand-picked per element.

### Core rule

```
1. Assign each element a grid address (col, row) — or (layer, slot) / (actor, msgIndex)
2. Compute cell size and gaps from density preset (uniform along each axis)
3. Derive x, y, w, h from grid formulas
4. Route connections between cell boundaries
```

**Never** assign ad-hoc `x=415, y=1160` or uneven gaps (5px then 100px). Uneven layout is the #1 cause of bloated, unprofessional diagrams.

### Density preset — medium-loose (default)

| Token | Value | Applies to |
|-------|-------|------------|
| `marginX` | 24 | All types — canvas side margin |
| `marginTop` | 64 | Below title block |
| `hGap` | 16 | Horizontal gap between columns / actors / lanes |
| `vGap` | 32 | Vertical corridor between rows / messages / steps |
| `cellInset` | 12 | Element inset inside its grid cell |
| `boxHeight` | 50 | Standard box height |
| `rowHeight` | 82 | `boxHeight + vGap` — fixed vertical advance |
| `colWidth` | computed | See column formula below |

Compact (`rowHeight=68`) and loose (`rowHeight=98`) only when the user explicitly requests it.

### Column grid (horizontal uniformity)

For `N` equal columns (architecture layers, swim lanes, sequence actors, ER columns):

```
usableW   = canvasWidth - 2*marginX - (N-1)*hGap
colWidth  = floor(usableW / N)
colX(c)   = marginX + c * (colWidth + hGap)      // c = 0 … N-1
colCX(c)  = colX(c) + colWidth / 2
boxWidth  = colWidth - 2*cellInset
boxX(c)   = colX(c) + cellInset
```

All columns **must** share the same `colWidth` and `hGap`. Content-weighted widening of a single column requires stealing width equally from neighbors (max ±20px each) — default is equal columns.

### Row grid (vertical uniformity)

For elements stacked vertically (flowchart steps, architecture nodes in a layer, sequence messages):

```
rowY(r) = contentTop + r * rowHeight     // r = 0, 1, 2, …
contentTop = marginTop + regionPad         // or + headerH + regionPad for lanes
```

- **No skipped row indices** within a phase — consecutive `r` values only.
- Elements at the same logical beat (cross-column sync, same message round) share the same `r`.
- Phase changes add a `phaseSpacer` annotation (12px), **not** an empty row.

### Grid mapping by diagram type

| Type | Column index `c` | Row index `r` | Type-specific reference |
|------|------------------|---------------|-------------------------|
| Architecture (LTR) | layer / tier | stack slot in layer | `architecture.md` |
| Architecture (TTB) | slot in layer | layer / tier | `architecture.md` |
| Flowchart | branch offset (0=center) | step sequence | `flowchart.md` |
| Swim lane | lane / actor | process step | `swimlane.md` |
| Sequence | actor / participant | message index | `sequence.md` |
| Structural (class/ER) | class column | class row | `structural.md` |
| State machine | optional grid | state row | `flowchart.md` |
| Timeline | time slot | event row (or swap) | SKILL.md timelines |
| Data flow | pipeline stage | entity slot | `architecture.md` |
| Mind map | *(exception)* | radial branches at equal angles | SKILL.md mind maps |

Mind maps are the main exception: use equal angular spacing for branches, but still compute node positions from branch index — do not free-place nodes.

### Canvas size targets

| Diagram | Typical elements | Target viewBox |
|---------|------------------|----------------|
| Flowchart | 8–15 steps | 600–800 × 700–1000 |
| Architecture | 8–16 components, 3–5 layers | 900–1100 × 700–1000 |
| Sequence | 5–8 actors, 10–15 messages | 900–1100 × 800–1100 |
| Swim lane | 5 lanes, 10–15 nodes | 960–1040 × 1000–1150 |
| Structural | 6–12 entities | 800–1100 × 600–900 |

If the canvas exceeds these by >30%, re-check for skipped grid indices, uneven gaps, or ad-hoc coordinates.

### Uniform margin system

Margins are **three nested zones**. Every gap must use a named token — never mix ad-hoc values (12px here, 0px there).

```
┌─ viewBox ─────────────────────────────────────────────┐
│ marginX │                                  │ marginX │
│         │ ┌─ region / lane border ──────┐  │         │
│         │ │ regionPad                   │  │         │
│         │ │  ┌─ cell ─────────────┐     │  │         │
│         │ │  │cellInset           │     │  │         │
│         │ │  │  [ component ]     │     │  │         │
│         │ │  └────────────────────┘     │  │         │
│         │ │ regionPad                   │  │         │
│         │ └─────────────────────────────┘  │         │
│         │         legendGap (40)           │         │
│         │ ┌─ legend band ─────────────┐    │         │
│         └─────────────────────────────┘  │         │
│ marginBottom (30)                                    │
└──────────────────────────────────────────────────────┘
```

| Token | Value | Zone | Rule |
|-------|-------|------|------|
| `marginX` | 24 | Canvas ↔ outermost content / region | Equal left and right |
| `marginTop` | 64 | Title block ↔ diagram content | Title sits above this |
| `marginBottom` | 30 | Legend band ↔ viewBox bottom | Equal to side feel |
| `regionPad` | 20 | Region/lane border ↔ innermost content | **Equal on all four sides** (below header in lanes) |
| `cellInset` | 12 | Grid cell ↔ box | ≤ `regionPad`; boxes inside cells |
| `legendGap` | 40 | Content area ↔ legend panel top | Never overlap content |
| `legendHeight` | 36 | Legend panel height | Single row |

**Row origin with margins:**

```
contentTop = marginTop + regionPad                    // no region wrapper
contentTop = marginTop + headerH + regionPad          // swim lane / framed region
rowY(r)    = contentTop + r * rowHeight
```

**Content bounds (compute before drawing region borders):**

```
contentBottom = max(all element bottom edges) + regionPad
```

> **Never** set a fixed region/lane `height` (e.g. `height="918"`) before placing nodes. Derive height from content + padding.

**Region / lane rectangle (all lanes share one height):**

```
regionTop    = marginTop
regionHeight = contentBottom - regionTop
regionX(c)   = colX(c)          // lane = column
regionW      = colWidth
```

```svg
<!-- Draw AFTER node placement; height from formula, not guessed -->
<rect x="{colX(c)}" y="{regionTop}" width="{colWidth}" height="{regionHeight}" .../>
```

**Per-side verification** (mandatory for every element):

```
node.left   >= regionX + regionPad
node.right  <= regionX + regionW - regionPad
node.top    >= regionTop + headerH + regionPad    (lanes)
node.bottom <= regionTop + regionHeight - regionPad
```

**Common failure** (from `skill-publish-flow.svg`): platform lane `height=918` ends at y=982; "上架发布" box bottom also y=982 → **0px bottom `regionPad`**. Fix: `contentBottom = 982 + 20 = 1002`, `regionHeight = 938`.

**Canvas width / height:**

```
canvasWidth  = 2*marginX + N*colWidth + (N-1)*hGap
canvasHeight = contentBottom + legendGap + legendHeight + marginBottom
viewBox      = "0 0 {canvasWidth} {canvasHeight}"
```

---

## 2. Layout Planning (before writing SVG)

Build a layout plan on paper or in comments first:

1. **Choose density** — default **medium-loose** (table above).
2. **Grid plan** — assign every element a `(col, row)` or type-specific index; list grid dimensions `N` columns × `M` rows.
3. **Node registry** — `id`, `col`, `row`, `shape`, `w`, `h`. Compute `x,y` from grid formulas only.
4. **Connection list** — `from`, `to`, `side`, optional label; assign **ports** when edges share a target side.
5. **Place nodes** — derive all coordinates from grid + margin tokens.
6. **Compute `contentBottom`** — `max(element bottoms) + regionPad`.
7. **Draw region borders** — height = `contentBottom - marginTop` (computed, never fixed).
8. **Reserve legend band** — `legendTop = contentBottom + legendGap`.
9. **Set `viewBox`** — `canvasWidth × canvasHeight` from margin formulas.

```
contentBottom = max(element.bottom) + regionPad
legendTop     = contentBottom + legendGap
viewBoxHeight = legendTop + legendHeight + marginBottom
```

## 3. Z-Order (layering)

SVG paints back-to-front. Use this order strictly:

| Layer | Content | Notes |
|-------|---------|-------|
| 1 | Background + grid | |
| 2 | Region / lane boundaries | Dashed outlines, swim-lane separators |
| 3 | **Connection paths only** | `<line>`, `<path>` — **no `<text>`** |
| 4 | Opaque mask rects | Same bounds as each component, `fill` = background color |
| 5 | Component shapes | Semi-transparent fill + stroke |
| 6 | **All text** | Component labels, sublabels, arrow labels, annotations |
| 7 | Legend band | Opaque panel + legend items |
| 8 | Title block | Top-left, can also be layer 1 if preferred |

**Never put `<text>` inside the connections group.** Arrow labels drawn under shapes will be covered by mask rects and component fills — this is the #1 cause of hidden text.

## 4. Connection Anchoring

Line endpoints must land on the **shape boundary**, not on center coordinates or approximate positions. A visible gap between arrow tip and box edge means the endpoint was wrong.

### Rectangle / rounded-rect anchors

Given box `(x, y, w, h)` and center `(cx, cy)` where `cx = x + w/2`, `cy = y + h/2`:

| Direction | Anchor point |
|-----------|--------------|
| Top | `(cx, y)` |
| Bottom | `(cx, y + h)` |
| Left | `(x, cy)` |
| Right | `(x + w, cy)` |

For `rx >= 20` (pill/start-end nodes), use the flat edge midpoint — do not anchor on the curved cap.

### Diamond anchors

Given diamond center `(cx, cy)` with half-width `hw` and half-height `hh`:

| Direction | Anchor point |
|-----------|--------------|
| Top | `(cx, cy - hh)` |
| Bottom | `(cx, cy + hh)` |
| Left | `(cx - hw, cy)` |
| Right | `(cx + hw, cy)` |

### Cylinder anchors

Use side midpoints of the body rect (ignore ellipse caps for horizontal connections): left `(x, y + 10 + h/2)`, right `(x + w, y + 10 + h/2)`, top/bottom at ellipse centers.

### Marker tip compensation

Standard marker: `refX="9"` on a 10px-wide arrowhead → tip extends **1px past** the path endpoint.

- **Target (marker-end):** endpoint = target boundary (tip slightly overlaps into shape — correct).
- **Source (no marker):** startpoint = source boundary.

Use `markerUnits="userSpaceOnUse"` on all markers so size stays constant regardless of stroke width:

```svg
<marker id="arrow" markerUnits="userSpaceOnUse" markerWidth="10" markerHeight="7"
        refX="9" refY="3.5" orient="auto">
  <polygon points="0 0, 10 3.5, 0 7" fill="#64748b"/>
</marker>
```

Use `stroke-linecap="butt"` on connection paths (default, but be explicit on busy diagrams).

### Routing patterns

**Vertical (stacked boxes):**
```svg
<!-- from bottom of A to top of B -->
<line x1="{A.cx}" y1="{A.y + A.h}" x2="{B.cx}" y2="{B.y}"
      stroke="..." stroke-width="1.5" stroke-linecap="butt" marker-end="url(#arrow)"/>
```

**Horizontal (adjacent boxes):**
```svg
<!-- from right of A to left of B -->
<line x1="{A.x + A.w}" y1="{A.cy}" x2="{B.x}" y2="{B.cy}"
      stroke="..." stroke-width="1.5" stroke-linecap="butt" marker-end="url(#arrow)"/>
```

**L-shaped (cross-lane):**
```svg
<!-- exit bottom of A, enter top of B in another column -->
<path d="M {A.cx} {A.y + A.h} L {A.cx} {midY} L {B.cx} {midY} L {B.cx} {B.y}"
      fill="none" stroke="..." stroke-width="1.5" stroke-linecap="butt" marker-end="url(#arrow)"/>
```

**Corridor routing:** pick `midY` or `midX` in the **gap between rows/columns**, at least 10px from every box edge. Never route through a box bounding rect.

### Port distribution

When **two or more connections** attach to the **same edge** of the same node (e.g. A→B.top and C→B.top), never reuse the center anchor — arrows will overlap.

**Algorithm:**

1. Group connections by `(targetId, side)` — e.g. all edges landing on `B.top`.
2. Sort by source position along the perpendicular axis (for `top`/`bottom`, sort by source `cx`; for `left`/`right`, sort by source `cy`).
3. For `n` connections on one edge, place anchors at fractions `i / (n + 1)` for `i = 1 … n`:

```
top/bottom edge:  anchorX = target.x + target.w * i / (n + 1)
left/right edge:  anchorY = target.y + target.h * i / (n + 1)
```

**Example:** `B = (250, 505, 160, 54)` receives two arrows on `top` from the left and from above:

```
n = 2
port 1: (250 + 160/3, 505) = (303, 505)
port 2: (250 + 320/3, 505) = (357, 505)
```

Route each incoming line to its assigned port; the last segment should be perpendicular to the edge.

**Overflow:** if `n > 3` on one edge, prefer splitting across two edges (e.g. two on `top`, one on `left`) or widen the target box.

### Common mistakes (from real diagrams)

| Wrong | Right |
|-------|-------|
| Line from `(324, 530)` when source right edge is `410` | Start at `(410, 532)` |
| Line to `(426, 530)` when target left edge is `430` | End at `(430, 532)` |
| Line to `y=634` when target top is `636` | End at `y=636` |
| Arrow label inside connections `<g>` | Move label to text layer (layer 6) |
| Two arrows both to `B.top` at `(B.cx, B.y)` | Assign ports at `B.x + B.w/3` and `B.x + 2*B.w/3` |
| Decision node drawn as `<rect rx="5">` | Use diamond `<polygon>` (see flowchart.md) |

## 5. Text Placement

### Component text
- Always in layer 6, after shapes.
- Title line: `y + 24`, sublabel: `y + 40` (standard 60px box).

### Arrow / edge labels

Place in **layer 6**, offset 8–12px from the connection midpoint. Keep **10px clearance** from any box edge.

**Label background priority** (use the lightest option that keeps text readable):

| Priority | When | How |
|----------|------|-----|
| 1 — Plain text | Label sits in an empty corridor, not crossing any line | `<text>` only, no background |
| 2 — Text halo | Label crosses one connection line | `paint-order="stroke fill"` with background-colored stroke |
| 3 — Tight pill | Crosses multiple colored lines and halo is insufficient | `<rect>` sized to text (see below) |

**Prefer text halo over pills** — no width guessing, no oversized backgrounds:

```svg
<!-- Light theme: stroke matches background -->
<text x="388" y="523" fill="#059669" font-size="8" text-anchor="middle"
      stroke="#f8fafc" stroke-width="3" paint-order="stroke fill">请求评测</text>

<!-- Dark theme -->
<text x="388" y="523" fill="#34d399" font-size="8" text-anchor="middle"
      stroke="#0f172a" stroke-width="3" paint-order="stroke fill">请求评测</text>
```

**Tight pill** (last resort only) — width must hug the text, never use a fixed or generous width:

```
estimatedWidth = Σ charWidth + 6        (3px padding each side)
  CJK / fullwidth @ 8px font: 8px per char
  ASCII @ 8px font: 4.5px per char
pillW = min(estimatedWidth, corridorFreeWidth)
pillH = 13
pillX = textX - pillW/2   (when text-anchor="middle")
```

```svg
<!-- "发布事件" ≈ 4×8 + 6 = 38px, NOT 104px -->
<rect x="659" y="600" width="38" height="13" rx="2" fill="#f8fafc"/>
<text x="678" y="611" fill="#ea580c" font-size="8" text-anchor="middle">发布事件</text>
```

**Never** place a pill over a component box or another label. If the corridor is too narrow, shorten the label or move it along the path.

### Annotations between nodes
- Place in the **gap** between boxes, not overlapping either box's bounding rect.
- If the gap is < 20px tall, move the annotation to the side or shorten the label.

### Rotated labels (loop-back corridors)
- Keep in layer 6.
- Position on empty margin space (left/right of swim lanes), not over lane content.

## 6. Legend Band

The legend is **not** "bottom-right of the viewBox" — it is a **reserved horizontal band below all content**.

### Sizing

```
legendHeight  = 36   (single row) or 56 (two rows)
legendTop     = contentBottom + legendGap    // legendGap = 40
legendWidth   = canvasWidth - 2*marginX
```

### Placement

- **Preferred:** full-width band at `legendTop`, items right-aligned inside the band.
- **Alternative:** bottom-left if the right side is congested.
- **Never** overlap the legend with any node, connection corridor, or loop-back path.

### Legend panel pattern

Always give the legend an opaque background so it never visually competes with content above:

```svg
<!-- Dark theme -->
<rect x="20" y="{legendTop}" width="{totalWidth - 40}" height="{legendHeight}"
      rx="6" fill="#0f172a" stroke="#334155" stroke-width="1"/>
<g transform="translate({legendTop + 12}, ...)">
  <!-- legend items -->
</g>
```

```svg
<!-- Light theme -->
<rect x="20" y="{legendTop}" width="{totalWidth - 40}" height="{legendHeight}"
      rx="6" fill="#f8fafc" stroke="#e2e8f0" stroke-width="1"/>
```

### viewBox must include the legend

```svg
<!-- BAD: legend at y=1292 inside viewBox 0 0 1040 1300 while content ends at y=1294 -->
<!-- GOOD: contentBottom=1294 → legendTop=1334 → viewBox height=1334+36+30=1400 -->
<svg viewBox="0 0 1040 1400" ...>
```

## 7. Pre-Save Verification Checklist

Before saving, verify:

- [ ] **Grid:** every element has a `(col, row)` address; all `x,y` derived from `colX`/`rowY` formulas
- [ ] **Uniform gaps:** `hGap` and `vGap` (or `rowHeight`) are constant — no mixed 5px/100px spacing
- [ ] **Equal columns:** all columns share the same `colWidth` (unless documented redistribution)
- [ ] **No skipped rows:** consecutive row indices within each phase; no 500px+ voids
- [ ] **Canvas size:** within target range for diagram type (see §1 table)
- [ ] **Margins:** all gaps use named tokens (`marginX`, `regionPad`, `legendGap`, …) — no 0px region padding on any side
- [ ] **Region bounds:** `regionHeight` computed from `contentBottom`; no fixed/guessed lane height
- [ ] **Padding check:** every element satisfies `bottom <= contentBottom - regionPad` and `top >= contentTop`
- [ ] No `<text>` elements appear before component shapes in the SVG source order
- [ ] Every connection endpoint matches a computed anchor from the node registry
- [ ] No arrow tip has a visible gap (> 2px) from its target shape
- [ ] `legendTop >= contentBottom + 40` and legend panel does not overlap any shape
- [ ] `viewBox` height includes legend band + 30px bottom padding
- [ ] Arrow labels have 10px clearance from all box edges
- [ ] Label backgrounds are text halos or tight pills — no oversized `<rect>` behind labels
- [ ] No two connections share the same anchor on the same target edge (ports distributed)
- [ ] Decision / branch nodes use diamond polygons, not rectangles
- [ ] Annotations sit in gaps, not on top of boxes
