# Sequence Diagram Layout

> Read `layout-fundamentals.md` §1 (Uniform Grid) first. Sequence diagrams use **column = actor**, **row = message index**.

## Core Elements

| Element | Visual | Description |
|---------|--------|-------------|
| Actor/Participant | Box at top + dashed vertical lifeline | Each entity in the interaction |
| Sync message | Solid arrow → | Request or call |
| Async message | Open arrowhead → | Fire-and-forget |
| Return message | Dashed arrow ← | Response |
| Activation bar | Narrow filled rect on lifeline | Entity is processing |
| Self-message | Arrow looping back to same lifeline | Internal processing |
| Note | Rounded rect with folded corner | Annotation |
| Alt/Opt frame | Dashed boundary with label tab | Conditional block |
| Loop frame | Dashed boundary with "loop" tab | Repetition |

## Layout Algorithm

1. **Actor grid** — `N` actors at equal columns: `colX(c)` and `colCX(c)` from `layout-fundamentals.md` §1. Actor box: `x = boxX(c)`, `w = boxWidth`, `h = 45`, `y = marginTop`.
2. **Lifelines** — vertical dashed line at `colCX(c)` from actor bottom to `contentBottom`.
3. **Message grid** — each message gets index `m` (0, 1, 2, …); `msgY(m) = actorBottom + 20 + m * msgRowH` where `msgRowH = rowHeight` (82px default, medium-loose).
4. **Draw messages** — horizontal arrows at `msgY(m)` between `colCX(from)` and `colCX(to)`.
5. **Activation bars** — 10px wide, centered on `colCX(c)`, from `msgY(in)` to `msgY(out)`.

**Never** hand-space actors at arbitrary x positions. **Never** use mixed 40px/60px message spacing — use fixed `msgRowH`.

## Actor Box

```svg
<!-- Actor box -->
<rect x="X" y="20" width="130" height="45" rx="6" fill="#f8fafc"/>
<rect x="X" y="20" width="130" height="45" rx="6" fill="rgba(8,145,178,0.15)" stroke="#0891b2" stroke-width="1.5"/>
<text x="CX" y="47" fill="#0f172a" font-size="11" font-weight="600" text-anchor="middle">Actor Name</text>

<!-- Lifeline -->
<line x1="CX" y1="65" x2="CX" y2="BOTTOM" stroke="#94a3b8" stroke-width="1" stroke-dasharray="6,4"/>
```

## Message Arrows

Horizontal messages start/end at lifeline `CX` (center of actor box). Draw paths in the connections layer; labels in the text layer (after activation bars).

```svg
<!-- Connections layer -->
<line x1="FROM_CX" y1="Y" x2="TO_CX" y2="Y" stroke="#64748b" stroke-width="1.5" stroke-linecap="butt" marker-end="url(#arrow)"/>
<line x1="TO_CX" y1="Y" x2="FROM_CX" y2="Y" stroke="#94a3b8" stroke-width="1" stroke-dasharray="6,3" stroke-linecap="butt" marker-end="url(#arrow)"/>
<path d="M CX,Y L CX+40,Y L CX+40,Y+25 L CX,Y+25" fill="none" stroke="#64748b" stroke-width="1.5" stroke-linecap="butt" marker-end="url(#arrow)"/>

<!-- Text layer -->
<text x="MID_X" y="Y-8" fill="#334155" font-size="9" text-anchor="middle">methodCall()</text>
<text x="MID_X" y="Y-8" fill="#64748b" font-size="8" text-anchor="middle" font-style="italic">response</text>
<text x="CX+45" y="Y+15" fill="#334155" font-size="8">process()</text>
```

## Activation Bar

```svg
<rect x="CX-5" y="START_Y" width="10" height="H" rx="2" fill="rgba(8,145,178,0.3)" stroke="#0891b2" stroke-width="1"/>
```

## Conditional / Loop Frames

```svg
<!-- Frame boundary -->
<rect x="X" y="Y" width="W" height="H" rx="4" fill="none" stroke="#94a3b8" stroke-width="1" stroke-dasharray="4,3"/>
<!-- Frame label tab -->
<rect x="X" y="Y" width="50" height="18" rx="4" fill="rgba(100,116,139,0.15)" stroke="#94a3b8" stroke-width="1"/>
<text x="X+25" y="Y+13" fill="#64748b" font-size="8" font-weight="600" text-anchor="middle">alt</text>
<!-- Condition text -->
<text x="X+60" y="Y+13" fill="#64748b" font-size="8" font-style="italic">[condition]</text>
<!-- Divider line for else -->
<line x1="X" y1="MID_Y" x2="X+W" y2="MID_Y" stroke="#94a3b8" stroke-width="1" stroke-dasharray="4,3"/>
<text x="X+10" y="MID_Y+13" fill="#64748b" font-size="8" font-style="italic">[else]</text>
```

## Numbering

For complex sequences (8+ messages), number each message:

```svg
<circle cx="FROM_CX-15" cy="Y" r="8" fill="rgba(37,99,235,0.12)" stroke="#2563eb" stroke-width="1"/>
<text x="FROM_CX-15" y="Y+3" fill="#2563eb" font-size="7" font-weight="600" text-anchor="middle">1</text>
```

## Color Assignment

Assign each actor a distinct color from the palette. Use that color for:
- Actor box stroke
- Activation bar on that lifeline
- Outgoing arrows from that actor (optional, for visual clarity in complex diagrams)
