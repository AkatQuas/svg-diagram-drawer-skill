# Architecture Diagram Layout

> Read `layout-fundamentals.md` §1 (Uniform Grid) first, then connection anchoring and legend band.

## Flow Direction

Choose one primary direction:
- **Left-to-Right (LTR):** Best for data pipelines, request flows. Clients on left, data on right.
- **Top-to-Bottom (TTB):** Best for layered architectures. Clients at top, infrastructure at bottom.

## Layout Algorithm

1. **Identify layers** — assign each layer a column index `c` (LTR) or row index `r` (TTB).
2. **Compute grid** — equal `colWidth` (LTR) or equal `rowHeight` (TTB) from `layout-fundamentals.md` §1.
3. **Place components** — each component gets `(c, slot)` where `slot` is its stack index within the layer:
   - **LTR:** `x = boxX(c)`, `y = rowY(slot)`, `w = boxWidth`, `h = boxHeight`
   - **TTB:** `x = colX(slot)`, `y = rowY(r)`, `w = boxWidth`, `h = boxHeight`
4. **Region boundaries** — draw **after** node placement. `regionPad` (20px) on all sides around contained cells; height/width computed from content bounds, never fixed.
5. **Connectors** — route between layer columns/rows through `hGap` / `vGap` corridors.

**Never** use hand-picked column starts (200, 250, 460, 670). All columns derive from `colX(c)`.

## Typical Layer Structure (LTR)

4 layers, `canvasWidth=1000`, `N=4`, `hGap=16`:

```
colX(0)   colX(1)   colX(2)   colX(3)
  │         │         │         │
[Client] → [Gateway] → [Services] → [Database]
  │         │         │         │
 slot 0    slot 0    slot 0,1    slot 0
```

Multiple services in one layer: consecutive `slot` values (`rowY(0)`, `rowY(1)`, …) in the same column `c`.

## Typical Layer Structure (TTB)

4 layers, components placed at `colX(slot)` within each `rowY(r)`:

```
rowY(0):  [ Browser ]  [ Mobile ]  [ API Client ]     ← slot 0,1,2
rowY(1):  [      Load Balancer / API Gateway      ]   ← span or slot 0 wide box
rowY(2):  [ Auth ]  [ User Svc ]  [ Order Svc ]
rowY(3):  [ Redis ]  [ PostgreSQL ]  [ S3 ]
```

Wide spanning boxes (gateway): `w = colWidth * spanCols + hGap * (spanCols - 1)` — still aligned to grid edges.

## Connection Routing

- Anchor every endpoint to a shape boundary (see layout-fundamentals.md) — horizontal connections use `(x+w, cy)` → `(x, cy)`, vertical use `(cx, y+h)` → `(cx, y)`
- Prefer straight horizontal or vertical lines
- For connections that would cross components, use two-segment (L-shaped) paths:
  ```svg
  <path d="M {srcRight} L {midX} {srcCy} L {midX} {tgtCy} L {tgtLeft}"
        fill="none" stroke="#64748b" stroke-linecap="butt" marker-end="url(#arrow)"/>
  ```
- For busy diagrams, use `stroke-opacity="0.6"` on less important connections
- Label important connections in the **text layer** (after shapes), near the path midpoint — add an opaque pill if the label crosses a line

## Message Bus / Event Bus Pattern

When services communicate through a shared bus, draw it as a horizontal bar between the service layer:

```
Services:  [ Svc A ]    [ Svc B ]    [ Svc C ]
              │              │            │
Bus:     ════╪══════════════╪════════════╪═══════
              │              │            │
Data:    [ DB A ]        [ DB B ]     [ Cache ]
```

Use the Connector color (orange) for the bus bar.

## Multi-Region / Multi-Cloud

Nest region boundaries:
- Outer boundary: Cloud provider (AWS, GCP)
- Inner boundary: Region or VPC
- Innermost: Availability zones or subnets

Use different dash patterns to distinguish nesting levels:
- Outer: `stroke-dasharray="12,4"`
- Middle: `stroke-dasharray="8,4"`
- Inner: `stroke-dasharray="4,4"`
