---
name: svg-diagram-drawer-skill
description: Generate professional SVG diagrams of any type (architecture, flowcharts, sequence, structural, mind maps, timelines, state machines, data flow, illustrative) in light or dark themes. Light is the default. Produces standalone .svg files with embedded styles and fonts. Converts to @2x/@3x PNG via @resvg/resvg-js for crisp output.
version: 1.6.0
---

# SVG Diagram Drawer

Generate professional SVG diagrams across multiple diagram types. All output is a single self-contained `.svg` file with embedded styles and fonts. Optionally export to PNG via `@resvg/resvg-js`.

The **default theme is light**. Use the dark overrides in the [Dark Theme section](#dark-theme) only when the user explicitly requests a dark/black theme.

## Supported Diagram Types

| Type | When to Use | Key Characteristics |
|------|-------------|-------------------|
| **Architecture** | System components & relationships | Grouped boxes, connection arrows, region boundaries |
| **Flowchart** | Decision logic, process steps | Diamond decisions, rounded step boxes, directional flow |
| **Sequence** | Time-ordered interactions between actors | Vertical lifelines, horizontal messages, activation bars |
| **Structural** | Class diagrams, ER diagrams, org charts | Compartmented boxes, typed relationships (inheritance, composition) |
| **Mind Map** | Brainstorming, topic exploration | Central node, radiating branches, organic layout |
| **Timeline** | Chronological events | Horizontal/vertical axis, event markers, period spans |
| **Illustrative** | Conceptual explanations, comparisons | Free-form layout, icons, annotations, visual metaphors |
| **State Machine** | State transitions, lifecycle | Rounded state nodes, labeled transitions, start/end markers |
| **Data Flow** | Data transformation pipelines | Process bubbles, data stores, external entities |

## Design System

### Color Palette (Light theme — default)

Semantic colors for component categories. These are the default colors; use them unless the user requests dark theme.

| Category | Fill (rgba) | Stroke | Use For |
|----------|-------------|--------|---------|
| Primary | `rgba(8, 145, 178, 0.15)` | `#0891b2` (cyan-600) | Frontend, user-facing, inputs |
| Secondary | `rgba(5, 150, 105, 0.15)` | `#059669` (emerald-600) | Backend, services, processing |
| Tertiary | `rgba(124, 58, 237, 0.12)` | `#7c3aed` (violet-600) | Database, storage, persistence |
| Accent | `rgba(217, 119, 6, 0.12)` | `#d97706` (amber-600) | Cloud, infrastructure, regions |
| Alert | `rgba(225, 29, 72, 0.12)` | `#e11d48` (rose-600) | Security, errors, warnings |
| Connector | `rgba(234, 88, 12, 0.12)` | `#ea580c` (orange-600) | Buses, queues, middleware |
| Neutral | `rgba(100, 116, 139, 0.12)` | `#64748b` (slate-500) | External, generic, unknown |
| Highlight | `rgba(37, 99, 235, 0.12)` | `#2563eb` (blue-600) | Active state, focus, current step |

For flowcharts and sequence diagrams, assign colors by role (actor, decision, process) rather than by technology.

For **dark theme**, use the overrides in the [Dark Theme section](#dark-theme).

### Typography

Use system monospace fonts (works in both browser and PNG renderer):

```svg
<style>
  text { font-family: 'Menlo', 'Monaco', 'Courier New', monospace; }
</style>
```

For Chinese text support, add CJK fonts: `'Noto Sans SC', 'PingFang SC', sans-serif'`.

> **Note:** The PNG renderer (`@resvg/resvg-js`) loads system fonts via CoreText on macOS, so `Menlo`, `Monaco`, and `Courier New` are available. Avoid `SF Mono` unless confirmed installed.

Font sizes by role:
- **Title:** 16px, weight 700
- **Component name:** 11-12px, weight 600
- **Sublabel / description:** 9px, weight 400, color `#64748b`
- **Annotation / note:** 8px, weight 400
- **Tiny label (on arrows):** 7-8px

### Core Visual Elements

**Background (default light):** `#f8fafc` with subtle grid:
```svg
<defs>
  <pattern id="grid" width="40" height="40" patternUnits="userSpaceOnUse">
    <path d="M 40 0 L 0 0 0 40" fill="none" stroke="#e2e8f0" stroke-width="0.5"/>
  </pattern>
</defs>
<rect width="100%" height="100%" fill="#f8fafc"/>
<rect width="100%" height="100%" fill="url(#grid)"/>
```

**Arrowhead marker (standard):** always set `markerUnits="userSpaceOnUse"` so arrow size stays constant:
```svg
<marker id="arrow" markerUnits="userSpaceOnUse" markerWidth="10" markerHeight="7" refX="9" refY="3.5" orient="auto">
  <polygon points="0 0, 10 3.5, 0 7" fill="#94a3b8"/>
</marker>
```

**Arrowhead marker (colored) — create per-color as needed:**
```svg
<marker id="arrow-cyan" markerUnits="userSpaceOnUse" markerWidth="10" markerHeight="7" refX="9" refY="3.5" orient="auto">
  <polygon points="0 0, 10 3.5, 0 7" fill="#0891b2"/>
</marker>
```

**Open arrowhead (for async/return messages):**
```svg
<marker id="arrow-open" markerUnits="userSpaceOnUse" markerWidth="10" markerHeight="7" refX="9" refY="3.5" orient="auto">
  <polyline points="0 0, 10 3.5, 0 7" fill="none" stroke="#94a3b8" stroke-width="1.5"/>
</marker>
```

### Layout Fundamentals

→ Read `{baseDir}/references/layout-fundamentals.md` **before every diagram**.

Covers: **uniform grid layout (all types)**, connection anchoring, text layering, and legend band reservation.

### SVG Structure & Layering

Draw elements in this order to get correct z-ordering (SVG paints back-to-front):

1. Background fill + grid pattern
2. Region/group boundaries (dashed outlines)
3. **Connection paths only** (`<line>`, `<path>`) — **never put `<text>` here**
4. Opaque masking rects (same position as component boxes, `fill="#f8fafc"`)
5. Component boxes (semi-transparent fill + stroke)
6. **All text** — component labels, sublabels, arrangement, arrow labels, annotations
7. Legend band (reserved area below content — see [Legend Band](#legend-band))
8. Title block (top-left)

> **Critical:** Arrow labels in layer 3 get covered by mask rects and component fills. Always draw connection labels in layer 6.

The opaque masking rect trick is essential — semi-transparent component fills will show arrows underneath without it:

```svg
<!-- Mask layer: opaque background to hide arrows -->
<rect x="100" y="100" width="160" height="60" rx="6" fill="#f8fafc"/>
<!-- Visual layer: styled component -->
<rect x="100" y="100" width="160" height="60" rx="6" fill="rgba(8,145,178,0.15)" stroke="#0891b2" stroke-width="1.5"/>
<text x="180" y="125" fill="#0f172a" font-size="11" font-weight="600" text-anchor="middle">API Gateway</text>
<text x="180" y="141" fill="#64748b" font-size="9" text-anchor="middle">Kong / Nginx</text>
```

### Connection Anchoring

Compute line endpoints from shape boundaries — never guess coordinates.

| Shape | Top | Bottom | Left | Right |
|-------|-----|--------|------|-------|
| Rect `(x,y,w,h)` | `(x+w/2, y)` | `(x+w/2, y+h)` | `(x, y+h/2)` | `(x+w, y+h/2)` |
| Diamond center `(cx,cy)` hw×hh | `(cx, cy-hh)` | `(cx, cy+hh)` | `(cx-hw, cy)` | `(cx+hw, cy)` |

Use `stroke-linecap="butt"` and `marker-end` with `refX="9"` (tip overlaps target edge by ~1px — correct).

```svg
<!-- A.bottom → B.top -->
<line x1="{A.cx}" y1="{A.y+A.h}" x2="{B.cx}" y2="{B.y}"
      stroke="#0891b2" stroke-width="1.5" stroke-linecap="butt" marker-end="url(#arrow-cyan)"/>
```

**Port distribution:** when multiple arrows land on the same target edge, spread anchors at `i/(n+1)` fractions along that edge — never stack at center. Details → `layout-fundamentals.md`.

### Arrow Label Backgrounds

Priority: **plain text** → **text halo** → **tight pill** (last resort).

```svg
<!-- Preferred: halo when crossing a line (default light theme) -->
<text x="388" y="523" fill="#059669" font-size="8" text-anchor="middle"
      stroke="#f8fafc" stroke-width="3" paint-order="stroke fill">请求评测</text>
```

Do not use wide fixed-width `<rect>` pills — they cover nearby elements. If a pill is required: `width = charWidthSum + 6` (CJK 8px, ASCII 4.5px per char at font-size 8).

### Flowchart Shape Rules

| Semantic | Shape |
|----------|-------|
| Process / action | Rectangle `rx="6"` |
| **Decision / branch** | **Diamond `<polygon>`** — never a rounded rect |
| Start / end | Pill `rx="25"` |

→ Full rules in `{baseDir}/references/flowchart.md`.

### Legend Band

Reserve legend space during layout — do **not** anchor to the viewBox bottom.

```text
contentBottom = max bottom edge of all elements
legendTop     = contentBottom + 40
viewBoxHeight = legendTop + legendHeight + 30
```

Place legend in an opaque panel at `legendTop`. Right-align items inside the panel, or place bottom-left if the right side is congested. The legend must never overlap any diagram element.

```svg
<!-- Light theme legend panel; items right-aligned inside -->
<rect x="20" y="{legendTop}" width="{W-40}" height="36" rx="6" fill="#f8fafc" stroke="#e2e8f0" stroke-width="1"/>
<g font-size="9" fill="#334155">
  <line x1="680" y1="{legendTop+24}" x2="710" y2="{legendTop+24}" stroke="#64748b" stroke-width="1.4" marker-end="url(#arrow)"/>
  <text x="716" y="{legendTop+27}">数据 / 流程</text>
</g>
```

### Uniform Grid Layout (all diagram types)

→ Full specification in `{baseDir}/references/layout-fundamentals.md` §1.

**Mandatory for every diagram:**

1. **Grid address** — assign each element a `(col, row)` or type-specific index (actor, message, layer, slot).
2. **Compute coordinates** — `colX(c)`, `rowY(r)`, `boxWidth` from formulas; never hand-pick x/y.
3. **Equal spacing** — constant `hGap` (16px) and `rowHeight` (82px); same `colWidth` for all columns.
4. **Uniform margins** — use named tokens (`marginX=24`, `regionPad=20`, `legendGap=40`, `marginBottom=30`); region/lane height derived from content, never hard-coded.
5. **Consecutive indices** — no skipped rows/columns creating voids.

```text
colX(c)       = marginX + c × (colWidth + hGap)
contentTop    = marginTop + regionPad
rowY(r)       = contentTop + r × rowHeight
contentBottom = max(element.bottom) + regionPad
boxX(c)       = colX(c) + cellInset
boxWidth      = colWidth - 2 × cellInset
```

| Diagram type | Column = | Row = | Reference |
|--------------|----------|-------|-----------|
| Architecture | layer | stack slot | `architecture.md` |
| Flowchart | branch offset | step | `flowchart.md` |
| Swim lane | lane | step | `swimlane.md` |
| Sequence | actor | message index | `sequence.md` |
| Structural | entity column | entity row | `structural.md` |
| State machine | branch offset | state level | `flowchart.md` |
| Timeline | time slot | event band | equal `slotWidth` per interval |
| Mind map | *(radial)* | branch index | equal angle `360°/N` per branch |

### Spacing Rules (all diagram types)

- **Margin tokens:** `marginX`/`marginBottom` = 24/30 (canvas); `regionPad` = 20 (inside regions/lanes, all sides equal); `legendGap` = 40
- **Region height:** computed as `contentBottom - marginTop` after elements placed — never a fixed value
- **Arrow label clearance:** 10px from any box edge; prefer text halo over pills; pills must be tight-fit
- **Port spacing:** multiple arrows to the same edge use distributed anchors, not shared center
- **Legend band:** `legendTop = contentBottom + legendGap`; panel height 36px
- **Title block:** inside `marginTop` zone, x = `marginX`
- **viewBox:** `canvasWidth × (contentBottom + legendGap + legendHeight + marginBottom)`

### Component Patterns

**Standard box (service/process):**
```svg
<!-- Mask layer -->
<rect x="X" y="Y" width="160" height="60" rx="6" fill="#f8fafc"/>
<!-- Visual layer -->
<rect x="X" y="Y" width="160" height="60" rx="6" fill="FILL" stroke="STROKE" stroke-width="1.5"/>
<text x="CX" y="Y+24" fill="#0f172a" font-size="11" font-weight="600" text-anchor="middle">Name</text>
<text x="CX" y="Y+40" fill="#64748b" font-size="9" text-anchor="middle">description</text>
```

**Component patterns integrate the palette and the mask technique above. Use the semantic fill/stroke from the color palette for each category.**

**Decision diamond (flowchart) — mandatory for any branch/choice node:**
```svg
<g transform="translate(CX, CY)">
  <polygon points="0,-38 56,0 0,38 -56,0" fill="#f8fafc"/>  <!-- mask -->
  <polygon points="0,-38 56,0 0,38 -56,0" fill="rgba(217,119,6,0.12)" stroke="#d97706" stroke-width="1.5"/>
  <text y="-2" fill="#0f172a" font-size="10" font-weight="600" text-anchor="middle">通过?</text>
  <text y="14" fill="#64748b" font-size="8" text-anchor="middle">是 / 否</text>
</g>
```

**Database cylinder:**
```svg
<g transform="translate(X, Y)">
  <rect x="0" y="10" width="120" height="50" rx="2" fill="#f8fafc"/>
  <ellipse cx="60" cy="10" rx="60" ry="12" fill="#f8fafc"/>
  <ellipse cx="60" cy="60" rx="60" ry="12" fill="#f8fafc"/>
  <rect x="0" y="10" width="120" height="50" fill="rgba(124,58,237,0.12)"/>
  <ellipse cx="60" cy="10" rx="60" ry="12" fill="rgba(124,58,237,0.12)" stroke="#7c3aed" stroke-width="1.5"/>
  <ellipse cx="60" cy="60" rx="60" ry="12" fill="rgba(124,58,237,0.12)" stroke="#7c3aed" stroke-width="1.5"/>
  <line x1="0" y1="10" x2="0" y2="60" stroke="#7c3aed" stroke-width="1.5"/>
  <line x1="120" y1="10" x2="120" y2="60" stroke="#7c3aed" stroke-width="1.5"/>
  <text x="60" y="40" fill="#0f172a" font-size="11" font-weight="600" text-anchor="middle">PostgreSQL</text>
</g>
```

**Clear container (region / group):**
```svg
<rect x="X" y="Y" width="W" height="H" rx="12" fill="none" stroke="#d97706" stroke-width="1" stroke-dasharray="8,4"/>
<text x="X+12" y="Y+16" fill="#d97706" font-size="9" font-weight="600">AWS us-east-1</text>
```

**Security group:**
```svg
<rect x="X" y="Y" width="W" height="H" rx="8" fill="none" stroke="#e11d48" stroke-width="1" stroke-dasharray="4,4"/>
<text x="X+10" y="Y+14" fill="#e11d48" font-size="8" font-weight="500">VPC / Security Group</text>
```

---

## Dark Theme

When the user explicitly requests a dark/black theme, use these overrides against the light theme defaults above. **Light remains the default.**

### Background & Grid

```svg
<defs>
  <pattern id="grid" width="40" height="40" patternUnits="userSpaceOnUse">
    <path d="M 40 0 L 0 0 0 40" fill="none" stroke="#1e293b" stroke-width="0.5"/>
  </pattern>
</defs>
<rect width="100%" height="100%" fill="#0f172a"/>
<rect width="100%" height="100%" fill="url(#grid)"/>
```

### Text Colors

| Role | Light Theme Color | Dark Theme Color |
|------|-------------------|------------------|
| Title / Component name | `#0f172a` (slate-900) | `white` |
| Sublabel / description | `#64748b` | `#94a3b8` |
| Annotation / note | `#64748b` | `#94a3b8` |
| Arrow label | `#334155` | `#e2e8f0` |

### Dark Color Palette

Slightly brighter strokes (visit these on dark backgrounds):

| Category | Fill (rgba) | Stroke | Light Stroke Equivalent |
|----------|-------------|--------|------------------------|
| Primary | `rgba(8, 51, 68, 0.4)` | `#22d3ee` (cyan) | `#0891b2` |
| Secondary | `rgba(6, 78, 59, 0.4)` | `#34d399` (emerald) | `#059669` |
| Tertiary | `rgba(76, 29, 149, 0.4)` | `#a78bfa` (violet) | `#7c3aed` |
| Accent | `rgba(120, 53, 15, 0.3)` | `#fbbf24` (amber) | `#d97706` |
| Alert | `rgba(136, 19, 55, 0.4)` | `#fb7185` (rose) | `#e11d48` |
| Connector | `rgba(251, 146, 60, 0.3)` | `#fb923c` (orange) | `#ea580c` |
| Neutral | `rgba(30, 41, 58, 0.5)` | `#94a3b8` (slate) | `#64748b` |
| Highlight | `rgba(59, 130, 246, 0.3)` | `#60a5fa` (blue) | `#2563eb` |

### Mask & Arrow Changes

**Mask rect:** Change `fill="#f8fafc"` to `fill="#0f172a"`.

**Arrowhead markers:** Change `fill="#94a3b8"` to `fill="#64748b"`:

```svg
<marker id="arrow" markerUnits="userSpaceOnUse" markerWidth="10" markerHeight="7" refX="9" refY="3.5" orient="auto">
  <polygon points="0 0, 10 3.5, 0 7" fill="#64748b"/>
</marker>
```

### Dark Component Pattern

Mask uses `#0f172a`; text uses white:

```svg
<rect x="100" y="100" width="160" height="60" rx="6" fill="#0f172a"/>
<rect x="100" y="100" width="160" height="60" rx="6" fill="rgba(8,51,68,0.4)" stroke="#22d3ee" stroke-width="1.5"/>
<text x="180" y="125" fill="white" font-size="11" font-weight="600" text-anchor="middle">API Gateway</text>
<text x="180" y="141" fill="#94a3b8" font-size="9" text-anchor="middle">Kong / Nginx</text>
```

## Type-Specific Layout Guidance

Determine this SKILL.md file's directory path as `{baseDir}` (the skill root). Reference files live in `{baseDir}/references/` and contain detailed layout algorithms and examples. Read the reference for the diagram type before starting layout.

### Architecture Diagrams
→ Read `{baseDir}/references/architecture.md`

Key points: left-to-right or top-to-bottom data flow. Group related services in region boundaries. Use buses/connectors between layers. Place databases at the bottom or right.

### Flowcharts
→ Read `{baseDir}/references/flowchart.md`

Key points: row-index steps (`rowY(r)`), branch columns for decisions, diamonds for choices, medium-loose density.

### Swim Lane Diagrams
→ Read `{baseDir}/references/swimlane.md` (extends the universal grid: lane = column, step = row).

### Architecture / Sequence / Structural
→ Each reference maps the universal grid to its own column/row semantics. Always compute `colX`/`rowY` — never separate coordinates.

### Sequence Diagrams
→ Read `{baseDir}/references/sequence.md`

Key points: actors as boxes at top, vertical dashed lifelines, horizontal arrows for messages (solid=sync, dashed=return). Time flows downward. Activation bars show processing. Number messages if complex.

### Structural Diagrams
→ Read `{baseDir}/references/structural.md`

Key points: compartmented boxes (name / attributes / methods for class diagrams). Relationship lines: solid with filled diamond=composition, solid with empty diamond=aggregation, dashed arrow=dependency, solid triangle=inheritance.

### Mind Maps
Radial layout — the grid exception. Place `N` branches at equal angles: `angle(i) = -90° + i × (360°/N)`. The distance from the center increases by a fixed `radiusStep` per depth level. Vary branch colors using the palette.

### Timelines
Time axis divided into equal slots: `slotX(t) = axisStart + t × slotWidth`. Events snap to slot centers; descriptions alternate sides at even/odd `t`. No irregular time spacing unless proportional to duration (document the scale).

### State Machines
Use flowchart grid: `rowY(r)` for state levels, column offset for branches. Rounded-rect states; filled circle = initial, ring = final. Label transitions `event [guard] / action`.

## Output Rules

1. Output a **single `.svg` file** — no external dependencies; all fonts are system-native
2. Set `viewBox` to fit all content with 30px padding; do **not** set fixed `width`/`height` attributes (let the SVG scale responsively)
3. Include `xmlns="http://www.w3.org/2000/svg"` on the root `<svg>` element
4. Put all `<style>`, `<defs>`, markers, and patterns at the top of the SVG
5. Use `text-anchor="middle"` for centered labels; ensure text doesn't overflow boxes
6. **Chinese text support:** When labels contain Chinese characters, use `font-family: 'Noto Sans SC', 'PingFang SC', sans-serif'` and increase box widths — CJK characters are wider
7. **XML comment safety:** Never use `--` inside XML comments (`<!-- ... -->`) — it prematurely terminates the comment. Use alternatives like ` - ` or `:`.
8. **Save location:** Save to `{projectDir}/diagram/{topic-slug}/`. Create the directory if it doesn't exist

## Script

Determine this SKILL.md file's directory path as `{baseDir}`. Script path: `{baseDir}/scripts/main.ts`.

Resolve `${BUN_X}` runtime: if `bun` installed → `bun`; if `npx` available → `npx -y bun`; else suggest installing bun.

### SVG → PNG Export

After saving the SVG, convert it to a @2x PNG using `@resvg/resvg-js`:

```bash
${BUN_X} {baseDir}/scripts/main.ts <svg-path> [options]
```

Options:
- `-s, --scale <n>` — Scale factor (default: 2, recommended: 3 for sharper output)
- `-o, --output <path>` — Custom output path (default: `<input>@2x.png`)
- `--json` — JSON output

> **Tip:** Use `-s 3` for noticeably sharper text and lines. The default `-s 2` produces good results too.

> **Note:** The script requires `@resvg/resvg-js` (install via `cd scripts && bun install`). `resvg` renders SVG to PNG with proper text support using system fonts — unlike `sharp`, which uses an older resvg build that drops text.

## Process

1. Identify the diagram type from the user's request. Default to the **light theme**; switch to the **dark theme** only when the user explicitly asks for it.
2. Read `{baseDir}/references/layout-fundamentals.md` §1, then the type-specific reference file.
3. **Plan the layout:** assign **grid addresses** `(col, row)` to every element, compute `colX`/`rowY`, build a node registry (no ad-hoc x/y), list connections, compute `contentBottom`, reserve legend band, set `viewBox`.
4. Write the SVG following the layering order above, using the appropriate theme colors (light by default).
5. **Verify before saving** (checklist in layout-fundamentals.md):
   - No `<text>` before component shapes in source order
   - Every connection endpoint lands on a shape boundary (no visible gaps)
   - Multiple arrows to the same edge use port distribution (no overlapping anchors)
   - Decision/branch nodes are diamonds, not rectangles
   - Arrow labels use plain text or halo; no oversized background pills
   - Legend panel at `contentBottom + 40`, not overlapping any element
   - `viewBox` includes legend band + 30px bottom padding
   - Uniform grid: all board coordinates from `colX`/`rowY`; equal gaps
   - Uniform margins: `marginTop`/`regionPad` on all sides; no content flush against borders
6. Save the SVG file
7. Run `${BUN_RUNTIME} {baseDir}/scripts/main.ts <svg-path>` to generate @2x PNG
8. Present both files to the user