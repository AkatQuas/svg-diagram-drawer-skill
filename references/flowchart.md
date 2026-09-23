# Flowchart Layout

> Read `layout-fundamentals.md` §1 (Uniform Grid) first, then connection anchoring and text layering.

## Shape Vocabulary

| Shape | Meaning | SVG Element |
|-------|---------|-------------|
| Rounded rect (large radius) | Start / End | `<rect rx="25">` |
| Rectangle | Process / Action | `<rect rx="6">` |
| **Diamond** | **Decision / branch** | `<polygon>` — **mandatory for choices** |
| Parallelogram | Input / Output | `<polygon>` with skew |
| Cylinder | Data store | Ellipse + rect combo |

### Decision vs Process — mandatory distinction

Use a **diamond** whenever the step involves a **choice, gate, or branching outcome**. Use a **rectangle** only for deterministic actions with a single outcome.

| Node content | Shape | Example |
|--------------|-------|---------|
| 创建 / 上传 / 接收 / 发起 / 查询 / 上架 | Rectangle | "接收发布申请" |
| 通过 / 驳回 / 是 / 否 / 是否 / 决策 / 判断 | **Diamond** | "管理员决策", "通过 / 驳回" |
| 开始 / 结束 / 成功终态 | Pill rect `rx="25"` | "Skill 上架发布 ✓" |

> **Never** draw a decision as `<rect rx="5">`. A box labeled "决策" or "通过 / 驳回" is always a diamond, even inside swim lanes.

**Decision diamond pattern** (mask + shape + text, Accent/rose color):

```svg
<!-- Center at (CX, CY), hw=56, hh=38 → bounding box ≈ 112×76 -->
<g transform="translate(CX, CY)">
  <!-- mask (light theme) -->
  <polygon points="0,-38 56,0 0,38 -56,0" fill="#f8fafc"/>
  <!-- visual -->
  <polygon points="0,-38 56,0 0,38 -56,0"
           fill="rgba(217,119,6,0.12)" stroke="#d97706" stroke-width="1.5"/>
  <text y="-2" fill="#0f172a" font-size="10" font-weight="600" text-anchor="middle">管理员决策</text>
  <text y="14" fill="#64748b" font-size="8" text-anchor="middle">通过 / 驳回</text>
</g>
```

Register diamonds in the node registry with `shape: diamond`, `cx`, `cy`, `hw`, `hh` — not `x, y, w, h`.

## Flow Direction

Primary flow: **top to bottom**. Branch flows go left/right from decisions.

## Layout Algorithm

1. **Assign row indices** — main-path steps get consecutive `r = 0, 1, 2, …`; `y = rowY(r)`.
2. **Column for branches** — happy path at `col = 0` (canvas center); "No" branches at `col = ±1`, `±2` using `colX(center + offset)`.
3. **Identify the main path** — runs straight down center column
4. **Branch from decisions:** "Yes" continues down (`r+1`, same col); "No" branches to adjacent column
5. **Merge paths:** route branches back to main column at the next free `r`
6. **Loop-backs:** route on far left/right margin columns, not by skipping rows

```
centerCol = floor(N/2)   // or single column: x = canvasCX - boxWidth/2
branchX(offset) = colCX(centerCol + offset)
```

## Spacing

**Default density: medium-loose** (`rowHeight=82`, `vGap=32`, `boxHeight=50`). Do not use 60–80px step gaps or 40px minimum vertical gaps on swim-lane diagrams — those produce bloated canvases.

| Context | Vertical unit | Value |
|---------|---------------|-------|
| Simple flowchart (no lanes) | step gap | 48–56px between box edges |
| **Swim lane (default)** | `rowHeight` | **82px** per row (50 box + 32 corridor) |
| Phase annotation only | `phaseSpacer` | 12px extra, not a full row |

- Decision diamond: hw=56, hh=38 (fits in one row band)
- Branch horizontal offset: route in `laneGap` corridors between equal lanes
- Merge connector clearance: 10px from any box (medium-loose)

## Decision Labels

Place "Yes" / "No" (or "True" / "False", "是" / "否", "同意" / "驳回") on the **exit arrows**, 10px from the diamond edge — not inside the diamond. **Draw labels in the text layer (after shapes), not inside the connections group.**

Exit arrows anchor to diamond boundary points: `bottom` = 通过/同意 (continue down), `left`/`right` = 驳回/分支, per layout.

When a diamond has **multiple incoming** arrows, assign ports on the target edge (see layout-fundamentals.md port distribution) — do not stack all at `top` center.

Anchor lines to diamond boundary points (see layout-fundamentals.md), not to arbitrary offsets:

```svg
<!-- Decision diamond at (400, 200), hw=50, hh=35 -->
<!-- Connections layer -->
<line x1="400" y1="235" x2="400" y2="300" stroke="#64748b" stroke-linecap="butt" marker-end="url(#arrow)"/>
<line x1="450" y1="200" x2="550" y2="200" stroke="#64748b" stroke-linecap="butt" marker-end="url(#arrow)"/>

<!-- Text layer (after all shapes) -->
<text x="412" y="260" fill="#34d399" font-size="8">Yes</text>
<text x="480" y="193" fill="#fb7185" font-size="8">No</text>
```

## Coloring Strategy

- **Start/End nodes:** Highlight color (blue)
- **Process steps:** Primary (cyan) or Secondary (emerald)
- **Decision diamonds:** Accent (amber) — they draw the eye naturally
- **Error/exception paths:** Alert (rose) dashed arrows
- **Happy path arrows:** Slightly brighter than branch arrows (`stroke-opacity` difference)

## Complex Flowcharts

For flowcharts with 10+ steps:
- Group related steps into swim lanes (vertical columns with header bars)
- Add a "phase" row header at the top of each swim lane
- Use the region boundary pattern from Architecture for swim lanes

## Swim Lane Flowcharts

→ **Read `{baseDir}/references/swimlane.md` in full** before laying out any multi-lane diagram.

Summary (details in swimlane.md):

1. **Equal lane grid** — identical `laneWidth` and `laneGap` for every column; compute `laneX(i)` from canvas width, never hand-pick per-lane widths.
2. **Row index grid** — assign each node a row `r`; `y = rowY(r)`. Cross-lane steps at the same beat share the same `r`. No ad-hoc Y coordinates.
3. **Medium-loose density** — `rowHeight=82`, `boxHeight=50`, `vGap=32`. Target viewBox ≤ 1100×1150 for ~15-node / 5-lane flows.
4. **Cross-lane connections:** exit at source boundary, route through `laneGap` corridor, enter target with port distribution.
5. **Async returns / loop-backs:** dashed lines; labels in text layer inside `vGap` corridor; reject loops in `loopMargin` (x < marginX), not by adding empty rows.
6. **Admin / approval:** "查看审查意见" = rectangle; "决策 / 通过 / 驳回" = diamond.
