---
name: svg-diagram-drawer-skill
description: Generate professional SVG diagrams of any type (architecture, flowcharts, sequence, structural, mind maps, timelines, state machines, data flow, illustrative) in dark or light themes. Produces standalone .svg files with embedded styles and fonts. Converts to @2x/@3x PNG via @resvg/resvg-js for crisp output.
version: 1.0.0
---

# SVG Diagram Drawer

Generate professional SVG diagrams across multiple diagram types. All output is a single self-contained `.svg` file with embedded styles and fonts. Optionally export to PNG via `@resvg/resvg-js`.

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

### Color Palette (Dark Theme)

Semantic colors for component categories:

| Category | Fill (rgba) | Stroke | Use For |
|----------|-------------|--------|---------|
| Primary | `rgba(8, 51, 68, 0.4)` | `#22d3ee` (cyan) | Frontend, user-facing, inputs |
| Secondary | `rgba(6, 78, 59, 0.4)` | `#34d399` (emerald) | Backend, services, processing |
| Tertiary | `rgba(76, 29, 149, 0.4)` | `#a78bfa` (violet) | Database, storage, persistence |
| Accent | `rgba(120, 53, 15, 0.3)` | `#fbbf24` (amber) | Cloud, infrastructure, regions |
| Alert | `rgba(136, 19, 55, 0.4)` | `#fb7185` (rose) | Security, errors, warnings |
| Connector | `rgba(251, 146, 60, 0.3)` | `#fb923c` (orange) | Buses, queues, middleware |
| Neutral | `rgba(30, 41, 59, 0.5)` | `#94a3b8` (slate) | External, generic, unknown |
| Highlight | `rgba(59, 130, 246, 0.3)` | `#60a5fa` (blue) | Active state, focus, current step |

For flowcharts and sequence diagrams, assign colors by role (actor, decision, process) rather than by technology.

For **light theme**, use the overrides in the [Light Theme section](#light-theme).

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
- **Sublabel / description:** 9px, weight 400, color `#94a3b8`
- **Annotation / note:** 8px, weight 400
- **Tiny label (on arrows):** 7-8px

### Core Visual Elements

**Background:** `#0f172a` (slate-900) with subtle grid:
```svg
<defs>
  <pattern id="grid" width="40" height="40" patternUnits="userSpaceOnUse">
    <path d="M 40 0 L 0 0 0 40" fill="none" stroke="#1e293b" stroke-width="0.5"/>
  </pattern>
</defs>
<rect width="100%" height="100%" fill="#0f172a"/>
<rect width="100%" height="100%" fill="url(#grid)"/>
```

**Arrowhead marker (standard):**
```svg
<marker id="arrow" markerWidth="10" markerHeight="7" refX="9" refY="3.5" orient="auto">
  <polygon points="0 0, 10 3.5, 0 7" fill="#64748b"/>
</marker>
```

**Arrowhead marker (colored) — create per-color as needed:**
```svg
<marker id="arrow-cyan" markerWidth="10" markerHeight="7" refX="9" refY="3.5" orient="auto">
  <polygon points="0 0, 10 3.5, 0 7" fill="#22d3ee"/>
</marker>
```

**Open arrowhead (for async/return messages):**
```svg
<marker id="arrow-open" markerWidth="10" markerHeight="7" refX="9" refY="3.5" orient="auto">
  <polyline points="0 0, 10 3.5, 0 7" fill="none" stroke="#64748b" stroke-width="1.5"/>
</marker>
```

### SVG Structure & Layering

Draw elements in this order to get correct z-ordering (SVG paints back-to-front):

1. Background fill + grid pattern
2. Region/group boundaries (dashed outlines)
3. Connection arrows and lines
4. Opaque masking rects (same position as component boxes, `fill="#0f172a"`)
5. Component boxes (semi-transparent fill + stroke)
6. Text labels
7. Legend (bottom-right or bottom area, outside all boundaries)
8. Title block (top-left)

The opaque masking rect trick is essential — semi-transparent component fills will show arrows underneath without it:

```svg
<!-- Mask layer: opaque background to hide arrows -->
<rect x="100" y="100" width="160" height="60" rx="6" fill="#0f172a"/>
<!-- Visual layer: styled component -->
<rect x="100" y="100" width="160" height="60" rx="6" fill="rgba(8,51,68,0.4)" stroke="#22d3ee" stroke-width="1.5"/>
<text x="180" y="125" fill="white" font-size="11" font-weight="600" text-anchor="middle">API Gateway</text>
<text x="180" y="141" fill="#94a3b8" font-size="9" text-anchor="middle">Kong / Nginx</text>
```

### Spacing Rules

These prevent overlapping — follow them strictly:

- **Component box height:** 50-70px (standard), 80-120px (large/complex)
- **Minimum gap between components:** 40px vertical, 30px horizontal
- **Arrow label clearance:** 10px from any box edge
- **Region boundary padding:** 20px inside edges around contained components
- **Legend placement:** At least 20px below the lowest diagram element
- **Title block:** 20px from top-left, outside diagram content area
- **viewBox:** Always extend to fit all content + 30px padding on all sides

### Component Patterns

**Standard box (service/process):**
```svg
<rect x="X" y="Y" width="160" height="60" rx="6" fill="#0f172a"/>
<rect x="X" y="Y" width="160" height="60" rx="6" fill="FILL" stroke="STROKE" stroke-width="1.5"/>
<text x="CX" y="Y+24" fill="white" font-size="11" font-weight="600" text-anchor="middle">Name</text>
<text x="CX" y="Y+40" fill="#94a3b8" font-size="9" text-anchor="middle">description</text>
```

**Decision diamond (flowchart):**
```svg
<g transform="translate(CX, CY)">
  <polygon points="0,-35 50,0 0,35 -50,0" fill="#0f172a"/>
  <polygon points="0,-35 50,0 0,35 -50,0" fill="rgba(120,53,15,0.3)" stroke="#fbbf24" stroke-width="1.5"/>
  <text y="4" fill="white" font-size="10" font-weight="600" text-anchor="middle">Condition?</text>
</g>
```

**Database cylinder:**
```svg
<g transform="translate(X, Y)">
  <rect x="0" y="10" width="120" height="50" rx="2" fill="#0f172a"/>
  <ellipse cx="60" cy="10" rx="60" ry="12" fill="#0f172a"/>
  <ellipse cx="60" cy="60" rx="60" ry="12" fill="#0f172a"/>
  <rect x="0" y="10" width="120" height="50" fill="rgba(76,29,149,0.4)"/>
  <ellipse cx="60" cy="10" rx="60" ry="12" fill="rgba(76,29,149,0.4)" stroke="#a78bfa" stroke-width="1.5"/>
  <ellipse cx="60" cy="60" rx="60" ry="12" fill="rgba(76,29,149,0.4)" stroke="#a78bfa" stroke-width="1.5"/>
  <line x1="0" y1="10" x2="0" y2="60" stroke="#a78bfa" stroke-width="1.5"/>
  <line x1="120" y1="10" x2="120" y2="60" stroke="#a78bfa" stroke-width="1.5"/>
  <text x="60" y="40" fill="white" font-size="11" font-weight="600" text-anchor="middle">PostgreSQL</text>
</g>
```

**Region boundary:**
```svg
<rect x="X" y="Y" width="W" height="H" rx="12" fill="none" stroke="#fbbf24" stroke-width="1" stroke-dasharray="8,4"/>
<text x="X+12" y="Y+16" fill="#fbbf24" font-size="9" font-weight="600">AWS us-east-1</text>
```

**Security group:**
```svg
<rect x="X" y="Y" width="W" height="H" rx="8" fill="none" stroke="#fb7185" stroke-width="1" stroke-dasharray="4,4"/>
<text x="X+10" y="Y+14" fill="#fb7185" font-size="8" font-weight="500">VPC / Security Group</text>
```

---

## Light Theme

When the user requests a light/white theme, use these overrides against the dark theme defaults above.

### Background & Grid

```svg
<defs>
  <pattern id="grid" width="40" height="40" patternUnits="userSpaceOnUse">
    <path d="M 40 0 L 0 0 0 40" fill="none" stroke="#e2e8f0" stroke-width="0.5"/>
  </pattern>
</defs>
<rect width="100%" height="100%" fill="#f8fafc"/>
<rect width="100%" height="100%" fill="url(#grid)"/>
```

### Text Colors

| Role | Dark Theme Color | Light Theme Color |
|------|-----------------|-------------------|
| Title / Component name | `white` | `#0f172a` (slate-900) |
| Sublabel / description | `#94a3b8` | `#64748b` (slate-500) |
| Annotation / note | `#94a3b8` | `#64748b` |
| Arrow label | `#e2e8f0` | `#334155` |

### Light Color Palette

Slightly darker strokes and more transparent fills on white background:

| Category | Fill (rgba) | Stroke | Dark Stroke Equivalent |
|----------|-------------|--------|----------------------|
| Primary | `rgba(8, 145, 178, 0.15)` | `#0891b2` (cyan-600) | `#22d3ee` |
| Secondary | `rgba(5, 150, 105, 0.15)` | `#059669` (emerald-600) | `#34d399` |
| Tertiary | `rgba(124, 58, 237, 0.12)` | `#7c3aed` (violet-600) | `#a78bfa` |
| Accent | `rgba(217, 119, 6, 0.12)` | `#d97706` (amber-600) | `#fbbf24` |
| Alert | `rgba(225, 29, 72, 0.12)` | `#e11d48` (rose-600) | `#fb7185` |
| Connector | `rgba(234, 88, 12, 0.12)` | `#ea580c` (orange-600) | `#fb923c` |
| Neutral | `rgba(100, 116, 139, 0.12)` | `#64748b` (slate-500) | `#94a3b8` |
| Highlight | `rgba(37, 99, 235, 0.12)` | `#2563eb` (blue-600) | `#60a5fa` |

### Mask & Arrow Changes

**Mask rect:** Change `fill="#0f172a"` to `fill="#f8fafc"`

**Arrowhead markers:** Change `fill="#64748b"` to `fill="#94a3b8"`

```svg
<marker id="arrow" markerWidth="10" markerHeight="7" refX="9" refY="3.5" orient="auto">
  <polygon points="0 0, 10 3.5, 0 7" fill="#94a3b8"/>
</marker>
```

### Light Component Pattern

Mask rect uses background color `#f8fafc`. Text uses `#0f172a` (dark slate):

```svg
<!-- Mask layer: opaque background to hide arrows -->
<rect x="100" y="100" width="160" height="60" rx="6" fill="#f8fafc"/>
<!-- Visual layer: styled component -->
<rect x="100" y="100" width="160" height="60" rx="6" fill="rgba(8,145,178,0.15)" stroke="#0891b2" stroke-width="1.5"/>
<text x="180" y="125" fill="#0f172a" font-size="11" font-weight="600" text-anchor="middle">API Gateway</text>
<text x="180" y="141" fill="#64748b" font-size="9" text-anchor="middle">Kong / Nginx</text>
```

### Light Database Cylinder

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

### Light Region Boundary

```svg
<rect x="X" y="Y" width="W" height="H" rx="12" fill="none" stroke="#d97706" stroke-width="1" stroke-dasharray="8,4"/>
<text x="X+12" y="Y+16" fill="#d97706" font-size="9" font-weight="600">AWS us-east-1</text>
```

## Type-Specific Layout Guidance

Determine this SKILL.md file's directory path as `{baseDir}` (the repo root). Read the reference file for the specific diagram type before starting layout. Reference files are located at `{baseDir}/references/` and contain detailed layout algorithms and examples.

### Architecture Diagrams
→ Read `{baseDir}/references/architecture.md`

Key points: left-to-right or top-to-bottom data flow. Group related services in region boundaries. Use buses/connectors between layers. Place databases at the bottom or right.

### Flowcharts
→ Read `{baseDir}/references/flowchart.md`

Key points: top-to-bottom primary flow. Diamonds for decisions with Yes/No labels on exit arrows. Rounded rectangles for start/end. Use the Highlight color for the happy path.

### Sequence Diagrams
→ Read `{baseDir}/references/sequence.md`

Key points: actors as boxes at top, vertical dashed lifelines, horizontal arrows for messages (solid=sync, dashed=return). Time flows downward. Activation bars show processing. Number messages if complex.

### Structural Diagrams
→ Read `{baseDir}/references/structural.md`

Key points: compartmented boxes (name / attributes / methods for class diagrams). Relationship lines: solid with filled diamond=composition, solid with empty diamond=aggregation, dashed arrow=dependency, solid triangle=inheritance.

### Mind Maps
Free-form radiating layout from a central concept. Use organic curves (`<path>` with cubic beziers) for branches. Vary branch colors using the palette. Larger font for central node, decreasing as you go outward.

### Timelines
Horizontal or vertical axis line. Event markers as circles or diamonds on the axis. Description text offset to alternating sides to avoid overlap. Use color to categorize event types.

### State Machines
Rounded-rect states with double-border for composite states. Filled circle for initial state, bullseye for final state. Curved arrows for self-transitions. Label all transitions with `event [guard] / action` format.

## Output Rules

1. Output a **single `.svg` file** — no external dependencies; all fonts are system-native
2. Set `viewBox` to fit all content with 30px padding; do NOT set fixed `width`/`height` attributes (let the SVG scale responsively)
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

> **Note:** The script requires `@resvg/resvg-js` (install via `cd scripts && bun install`). `resvg` renders SVG to PNG with proper text support using system fonts — unlike `sharp` which uses an older resvg build that drops text.

## Process

1. Identify the diagram type from the user's request. Also determine if the user wants a **dark theme** (default) or **light theme**.
2. Read the relevant reference file if one exists for that type
3. Plan the layout: list all components, determine grouping and flow direction, calculate positions
4. Write the SVG following the layering order above, using the appropriate theme colors
5. Verify spacing rules — no overlaps, legends outside boundaries, viewBox large enough
6. Save the SVG file
7. Run `${BUN_X} {baseDir}/scripts/main.ts <svg-path>` to generate @2x PNG
8. Present both files to the user
