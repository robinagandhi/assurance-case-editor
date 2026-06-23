# Agent Instructions — Assurance Case Confidence Maps Builder

This file describes how AI coding agents (Claude Code, Codex, Copilot Workspace, etc.) should work with this repository.

---

## What This Repo Is

A **single-file HTML application** (`assurance-case-builder.html`) for constructing Goal Structuring Notation (GSN) assurance cases. There is no build step, no package manager, and no external dependencies. Everything — HTML, CSS, and JavaScript — lives inline in one file.

The `slides/` folder contains course slides (remark.js format) that define the pedagogical context and terminology. Treat them as the **source of truth** for GSN concepts, node types, and the Baconian probability metric.

---

## Architecture at a Glance

```
assurance-case-builder.html
├── <style>          CSS (layout, palette, sidebar, modal, canvas node styles)
├── <body>           Toolbar · Palette · SVG canvas · Sidebar · Help modal · Context menu · Presenter bar
└── <script>
    ├── TYPE {}      Node type definitions (shape, fill, stroke, dimensions)
    ├── state {}     Single mutable state object
    ├── Render       renderCanvas() / render() / nodeHTML() / edgeHTML()
    ├── Geometry     s2c() / nodeAt() / connPt() / getHandlePositions() / resizeHandleAt()
    ├── Interaction  onDown() / onMove() / onUp() / onDbl() / onKey() / onWheel()
    ├── Touch        onTouchStart() / onTouchMove() / onTouchEnd() — wrap touches into synthetic
    │                mouse events for onDown/onMove/onUp; two-finger pinch handled separately
    ├── Selection    selectedIds() / toggleMultiSelect() / clearSelection() — state.sel (primary)
    │                + state.multi (Set) for marquee/shift-click multi-select and group drag
    ├── Clipboard    cloneSelectionData() / pasteClone() / duplicateSelection() / copySelection() /
    │                pasteClipboard() / pasteClipboardAt()
    ├── Quick-add    quickAddAt() / quickAddChild() — one-click connected child below a node
    ├── Context menu onContextMenu() / showNodeContextMenu() / showEdgeContextMenu() /
    │                showCanvasContextMenu() / selectSubtree() / changeSelectedType()
    ├── Outline      buildOutlineTree() / updateOutline() / flattenOutlineOrder() — shared tree
    │                walk used by both the sidebar Outline tab and Presenter mode
    ├── Collapse     toggleCollapse() / hiddenByCollapse() / expandAncestors() — state.collapsed
    │                is session-only view state, never persisted to the saved file
    ├── Search       onSearchInput() / onSearchKeydown() — state.searchMatchIds, also session-only
    ├── Review mode  state.reviewMode toggles the Reviewer Note field (n.note) in editor/sidebar
    ├── Presenter    startPresenter() / presenterGoTo() / presenterNext()/Prev() — top-down
    │                walkthrough using flattenOutlineOrder(); disables editing while active
    ├── Layout       autoLayout() — BFS + post-order X packing
    ├── History      pushHistory() / undo() / redo()  (JSON snapshots, max 25)
    ├── Persistence  saveToDisc() / openFile() / loadData() / tryRestoreAutosave()
    └── Validation   GUIDANCE{} / checkText() / validateCase() / updateChecklist() / focusNode()
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

### Selection model: primary + multi

`state.sel.nodeId` is the **primary** selection (drives the Properties sidebar and resize handles). `state.multi` is a `Set` of *additional* selected node ids from shift-click or marquee-drag. Always read the combined selection through `selectedIds()` rather than touching `state.sel`/`state.multi` directly — group move, group delete, duplicate, and copy/paste all key off it. Anywhere `state.sel` gets reset to empty (`undo`, `redo`, `loadData`, `newCase`, click-to-deselect), call `clearSelection()` so `state.multi` is cleared too instead of just resetting `state.sel`.

### Session-only view state is never persisted

`state.collapsed` (collapsed branches), `state.searchMatchIds` (find-in-diagram highlights), and `state.multi`/`state.sel` are UI state, not document content — they're excluded from `snapshot()` (undo/redo), `saveToDisc()`, and `saveSVG()`. If you add another view-only flag, follow the same rule: it lives in `state` for convenience, but never touches what gets written to JSON or SVG.

### Hit-testing must skip collapsed-away nodes

`nodeAt()`, `handleAt()`, `edgeAt()`, `quickAddAt()`, and `collapseToggleAt()` all call `hiddenByCollapse()` and skip hidden ids before doing their coordinate math — a node that's hidden by a collapsed ancestor must not be clickable even though it's still in `state.nodes`. Any new hit-test function over `state.nodes`/`state.edges` needs the same guard.

### Touch is translated into the mouse handlers, not duplicated

`onTouchStart`/`onTouchMove`/`onTouchEnd` build a minimal synthetic event (`{ clientX, clientY, button, shiftKey, preventDefault }`) and call the existing `onDown`/`onMove`/`onUp` — there is no separate touch interaction model. Only two-finger pinch-zoom is handled outside that path (in `pinch` state), since it has no mouse equivalent.

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
- `OUTLINE_ORDER` map (sidebar Outline tab / Presenter mode traversal order)
- `QUICK_ADD_TYPES` set if hovering it should show the "+" quick-add child button
- The palette button in the HTML (remember the matching `draggable`/`ondragstart` for palette drag-and-drop)
- The help modal node-types table
- The sidebar `<select>` options
- `GUIDANCE` constant (rule + good example shown in the editor tip and sidebar tip)

---

## Case Checklist & Claim Tips

`GUIDANCE` (rule + good example per type) drives the tip box in both the popup editor (`#ed-tip`) and the sidebar Properties panel (`.sb-tip`). `checkText(type, text)` is a heuristic, regex-based pattern check (no NLP) limited to `claim` and `evidence` — it returns a warning string or `null` and never blocks saving. `validateCase()` runs `checkText()` plus structural checks (disconnected nodes, undeveloped branches, top-level claims with no Context) across the whole graph and feeds the sidebar's Case Checklist (`updateChecklist()`); clicking an item calls `focusNode(id)` to select and re-center the camera on it.

Keep these as soft nudges, not hard validation — the heuristics will have false positives/negatives, and saves must never be blocked on them.

---

## What to Avoid

- **Do not add CDN links or external scripts.** The app must work offline with no network access.
- **Do not split into multiple files.** Single-file portability is an explicit requirement.
- **Do not call `render()` inside `onMove()`** for drag/resize — it's too slow. Use `renderCanvas()`.
- **Do not store computed geometry in state.** Keep state minimal; derive shapes at render time.
- **Do not modify `slides/` content** without the instructor's approval — these are published course materials.
- **Do not persist `state.collapsed`, `state.searchMatchIds`, `state.sel`, or `state.multi`** into `saveToDisc()`/`saveSVG()`/`snapshot()` — they're view state, not case content.
- **Do not add a new hit-test function over `state.nodes`** without filtering through `hiddenByCollapse()` first, or it'll make collapsed-away nodes clickable.
- **Do not build a second touch interaction model.** Route single-touch through the existing mouse handlers via a synthetic event, as `onTouchStart`/`onTouchMove`/`onTouchEnd` already do.

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
   - Shift-click two nodes, then shift-drag a marquee over a third; drag the group; Delete; confirm all three are gone in one undo step.
   - Select a node, Ctrl+D to duplicate, then Ctrl+C/Ctrl+V; right-click a node and try Select Subtree → Copy → right-click empty canvas → Paste.
   - Hover a Claim's "+" button; confirm it adds a connected sub-claim and opens its editor.
   - Collapse a node from its canvas toggle; confirm descendants disappear and Fit/hit-testing ignore them; expand again.
   - Type in the Find box; confirm matches highlight and Enter/Shift+Enter cycle them.
   - Toggle Review Mode, add a note to a node; confirm the amber badge persists after toggling Review Mode back off.
   - Click ▶ Present; step through with → and ←; confirm editing is blocked while presenting and the prior pan/zoom returns on Exit.

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
