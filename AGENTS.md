# Agent Instructions — Assurance Case Confidence Maps Builder

This file describes how AI coding agents (Claude Code, Codex, Copilot Workspace, etc.) should work with this repository.

---

## What This Repo Is

A **single-file HTML application** (`assurance-case-builder.html`) for constructing Goal Structuring Notation (GSN) assurance cases. There is no build step, no package manager, and no external dependencies. Everything — HTML, CSS, and JavaScript — lives inline in one file.

The `lecture-2/` folder contains course slides (remark.js format) that define the pedagogical context and terminology. Treat them as the **source of truth** for GSN concepts, node types, and the Baconian probability metric.

---

## Architecture at a Glance

```
assurance-case-builder.html
├── <style>          CSS (layout, palette, sidebar, modal, canvas node styles)
├── <body>           Toolbar · Palette · SVG canvas · Sidebar · Help modal
└── <script>
    ├── TYPE {}      Node type definitions (shape, fill, stroke, dimensions)
    ├── state {}     Single mutable state object
    ├── Render       renderCanvas() / render() / nodeHTML() / edgeHTML()
    ├── Geometry     s2c() / nodeAt() / connPt() / getHandlePositions() / resizeHandleAt()
    ├── Interaction  onDown() / onMove() / onUp() / onDbl() / onKey() / onWheel()
    ├── Layout       autoLayout() — BFS + post-order X packing
    ├── History      pushHistory() / undo() / redo()  (JSON snapshots, max 25)
    └── Persistence  saveCase() / loadCase() / exportJSON() / importJSON()
```

---

## Key Conventions

### State is a single object — mutate it, then re-render

```js
// All mutable application state lives here
let state = { nodes, edges, camera, mode, sel, drag, conn, pan, resize, spaceDown, counters };
```

Never store derived data in state. Edge styles (solid/dashed/dotted), branch indicators (diamond/circle), and connection handle positions are all computed at render time from `state.nodes` and `state.edges`.

### Two render functions — call the right one

| Function | What it rebuilds | When to call |
|----------|-----------------|--------------|
| `renderCanvas()` | SVG only (edges + nodes layers, camera transform) | During active drag, resize, property edits mid-keystroke |
| `render()` | SVG **and** sidebar | After any structural state change (add/delete node or edge, mode change, selection change) |

**Never call `render()` from a text input's `oninput` handler** — it rebuilds the sidebar DOM and destroys focus. Call `updateNodeProp(prop, val)` which calls `renderCanvas()` only.

### Coordinate system

All node positions (`n.x`, `n.y`) are in **canvas space**. Convert screen → canvas with `s2c(screenX, screenY)`. The SVG `<g id="world">` carries the camera transform; never apply camera math manually elsewhere.

### Node geometry

Every node has `{ x, y, w, h }` where `(x, y)` is the **center** and `(w, h)` is the full bounding box. Shape-specific geometry (circle radius, ellipse rx/ry, etc.) is derived from these. Connection handles sit at cardinal midpoints of the bounding box. Resize handles sit at corners.

---

## Node Types

Defined in `const TYPE = { ... }`. Each entry has:

```js
{ prefix, w, h, fill, stroke, sw, shape }
// shape: 'rect' | 'circle' | 'ellipse' | 'dogear' | 'parallelogram'
```

When adding a new node type, also update:
- `TYPE` constant
- `shapeHTML()` if the shape is new
- `connPt()` for accurate edge attachment
- `DEFEATER_TYPES` set if it is a defeater
- `CHILD_ORDER` map if auto-layout ordering matters
- The palette button in the HTML
- The help modal node-types table
- The sidebar `<select>` options

---

## What to Avoid

- **Do not add CDN links or external scripts.** The app must work offline with no network access.
- **Do not split into multiple files.** Single-file portability is an explicit requirement.
- **Do not call `render()` inside `onMove()`** for drag/resize — it's too slow. Use `renderCanvas()`.
- **Do not store computed geometry in state.** Keep state minimal; derive shapes at render time.
- **Do not modify `lecture-2/` slide content** without the instructor's approval — these are published course materials.

---

## Testing Without a Build Step

1. Open `assurance-case-builder.html` directly in a browser (file:// protocol, no server needed).
2. Run through the manual verification checklist:
   - Add one node of each type; confirm shape, fill, and label prefix are correct.
   - Connect two nodes; confirm edge arrow renders and leaf decorators disappear on both ends as appropriate.
   - Resize a node by dragging a corner handle; confirm opposite corner stays fixed.
   - Undo the resize (Ctrl+Z); confirm node returns to original size.
   - Click Auto Layout; confirm tree order is Claim → sub-claims → defeaters → evidence.
   - Export JSON; clear the canvas; import the file; confirm identical graph.
   - Open Help (? Help button or Escape closes it); confirm all sections render.

There are no automated tests. Correctness is verified manually in the browser.

---

## Baconian Confidence Metric

```
confidence = addressed_defeaters / total_defeaters
```

- **Defeater** — any node whose type is in `DEFEATER_TYPES` (`rebutting`, `undermining`, `undercutting`).
- **Addressed** — the defeater has at least one outgoing edge (it has been responded to with a sub-argument).
- Computed in `computeBackonianProbability()` and displayed in the sidebar confidence meter.

Do not change this formula without updating the course lecture slides and the help modal explanation.
