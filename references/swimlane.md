# Swim Lane Layout

> Read `layout-fundamentals.md` §1 (Uniform Grid) first. Swim lanes are a **column = lane, row = step** grid. Density: **medium-loose**.

Swim-lane specifics below. Grid formulas (`colX`, `rowY`, `colWidth`, `hGap`, `rowHeight`) are defined in `layout-fundamentals.md` — use `lane` as alias for `col`:

```
laneX(i)  = colX(i)
laneWidth = colWidth
laneGap   = hGap
laneInset = cellInset
```

## Layout algorithm

```
1. Assign each node (lane i, row r)
2. Compute equal column grid (layout-fundamentals §1)
3. Place: x = boxX(i), y = rowY(r) + headerOffset, w = boxWidth
4. Route connections; port distribution for shared edges
5. contentBottom = rowY(maxRow) + boxHeight + loopMargin
```

**Swim-lane-only tokens:**

| Token | Value |
|-------|-------|
| `headerH` | 28 — lane header band |
| `regionPad` | 20 — **equal** padding inside lane border (see layout-fundamentals §1) |
| `boxHeightSm` | 38 — aggregator / compact row |
| `phaseSpacer` | 12 — annotation between phases, not a full row |
| `loopMargin` | 24 — left margin for reject loop-backs (x < marginX) |

```
contentTop = marginTop + headerH + regionPad
rowY(r)    = contentTop + r * rowHeight
```

**Lane height — compute last, never hard-code:**

```
contentBottom = max(all node/diamond bottoms) + regionPad
laneHeight    = contentBottom - marginTop
```

All lanes share identical `laneHeight`. Draw lane `<rect>` only after `contentBottom` is known.

---

## 1. Row assignment

Map each process step to a row index before computing Y.

### Rules

- **One primary node per lane per row** (except intentional parallel splits).
- **Same-phase steps in consecutive rows** — no skipped row indices.
- **Cross-lane sync:** nodes that interact in one beat share the same `r` (e.g. platform "请求评测" and node "创建 EvalTask" both at `r=3`).
- **Parallel fan-out:** children share one row `r`, placed side-by-side within their lane's inner width.
- **Phase change:** increment `r` by 1; add `phaseSpacer` (12px) to the corridor annotation only — **do not** insert empty rows.

### Row plan template (Skill 发布类流程)

| r | 用户 | 平台 | ai-agent-node | MQ | 管理员 |
|---|------|------|---------------|-----|--------|
| 0 | 创建 Skill | | | | |
| 1 | 提交申请 | 接收申请 | | | |
| 2 | | 请求评测 | 创建 EvalTask | | |
| 3 | | | 生产者 | MQ 队列 | |
| 4 | | | 消费者 | | |
| 5 | | | 格式 / 安全 / AB (×3) | | |
| 6 | | | taskId 聚合 | | |
| 7 | | 查询/订阅 | | | |
| 8 | | 展示意见 | | | 查看意见 |
| 9 | | | | | 决策 ◇ |
| 10 | | | | | 上架 ✓ |

11 rows × 82px ≈ 902px content + header + legend ≈ **viewBox height ~1050–1100** (vs 1730 in loose layouts).

Empty cells = no box. Loop-back reject path runs in `loopMargin` column (x < marginX), not by adding rows.

---

## 2. Box placement within a lane

**Single box (default):**

```
x = boxX(i)
y = rowY(r)
w = boxWidth
h = boxHeight
cx = laneCX(i)
```

**Parallel boxes (n items in one lane, one row):**

```
inner = boxWidth
gap = 8
itemW = floor((inner - (n-1)*gap) / n)
itemX(k) = boxX(i) + k * (itemW + gap)     // k = 0 … n-1
y = rowY(r)
h = boxHeight
```

Center the group if `n * itemW + (n-1) * gap < inner`.

**Diamond in lane:**

```
hw = min(56, floor(boxWidth/2) - 4)
hh = 38
cx = laneCX(i)
cy = rowY(r) + boxHeight/2 + hh - boxHeight/2   // align visually in row band
```

**Pill (terminal):** same `x, w` as lane box; `h = boxHeight`; `rx = 25`.

---

## 3. Vertical uniformity checks

Before drawing, verify:

```
adjacent occupied rows: rowY(r+1) - rowY(r) === rowHeight   (always)
same-row cross-lane nodes: share identical y
max row index * rowHeight < 1100 - marginTop - legendBand   (else compress)
```

**Corridor height** between box bottoms and next box tops = `vGap` (32px). Arrow labels sit inside this corridor — if text doesn't fit, shorten the label, not the gap.

---

## 4. Lane boundary drawing

Draw lane borders **after** all nodes are placed and `contentBottom` is computed.

```svg
<!-- laneHeight = contentBottom - marginTop; NOT a guessed constant like 918 -->
<rect x="{colX(i)}" y="{marginTop}" width="{colWidth}" height="{laneHeight}"
      rx="8" fill="none" stroke="{laneColor}" stroke-width="1" stroke-dasharray="8,4"/>
```

Verify: lowest box in any lane has `box.bottom + regionPad <= marginTop + laneHeight`.

Header band: same `x`, `width` as lane; `height="{headerH}"`; `y="{marginTop}"`.

Lane label: `text-anchor="middle"` at `(colCX(i), marginTop + 18)`.

---

## 5. Pre-save swimlane checklist

See `layout-fundamentals.md` §7 (universal grid checks), plus:

- [ ] Cross-lane pairs at the same step share the same row index `r`
- [ ] Lane boundary rects share identical `colWidth`, top Y, and computed `laneHeight`
- [ ] `regionPad` (20px) clearance on all sides between content and lane border
- [ ] Reject loop-backs use `loopMargin` column, not extra rows
