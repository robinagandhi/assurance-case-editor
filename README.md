# Assurance Case Confidence Maps Builder

A self-contained, browser-based tool for building **Goal Structuring Notation (GSN)** assurance cases. No installation, no server, no external dependencies — open `assurance-case-builder.html` directly in any modern browser.

Developed to support the *Software Assurance* course at the University of Nebraska Omaha, replacing Lucidchart for in-class exercises.

---

**Try it out here:** [Assurance Case Confidence Maps Builder](https://htmlpreview.github.io/?https://github.com/robinagandhi/assurance-case-editor/blob/main/assurance-case-builder.html)

---

## Getting Started

1. Open `assurance-case-builder.html` in any modern browser (Chrome, Firefox, Safari, Edge).
2. Click a node type in the left palette to add it to the canvas.
3. Double-click a node to edit its label and description.
4. Drag from the midpoint handles (◦) on a node's edges to connect nodes.
5. Click **Load Sample** in the toolbar to see a complete worked example (Tweety-can-fly).

---

## Node Types

| Symbol | Type | Shape | Purpose |
|--------|------|-------|---------|
| **C** | Claim | Rectangle | A proposition about a system property. Written as *noun-phrase + verb-phrase*. |
| **E** | Evidence | Circle | Tangible, observable support for a claim. Written as a *noun-phrase only*. |
| **CT** | Context | Dog-ear box | Scope or definition that constrains how a claim is interpreted. |
| **J** | Justification | Ellipse | Rationale for why a claim or inference was introduced. |
| **IR** | Inference Rule | Parallelogram | Makes an implicit inference explicit so it can be challenged. |
| **R** | Rebutting Defeater | Light-yellow box | *Unless…* doubt that challenges a **claim**. |
| **U** | Undermining Defeater | Light-blue box | *Unless…* doubt that challenges **evidence**. |
| **UC** | Undercutting Defeater | Light-green box | *Unless…* doubt that challenges an **inference rule**. |

### Branch end markers (manual)

Add these from the **Branch Ends** section in the palette, then connect them to the last node in a branch:

- **◆ Undeveloped** — small black diamond. Place after a Claim or Defeater to signal the branch still needs further argument or evidence.
- **● Complete** — small grey filled circle. Place after an Evidence node to signal the branch is fully supported.

---

## Canvas Controls

| Action | How |
|--------|-----|
| Pan | Drag on empty canvas, or Space + drag |
| Zoom | Scroll wheel (zooms toward cursor) |
| Select | Click a node or edge |
| Move node | Drag a selected or unselected node |
| Resize node | Select a node, then drag any blue corner handle |
| Connect nodes | Drag from a midpoint handle (◦), or use **→ Connect** mode in the palette |
| Delete | Select then press **Delete/Backspace**, or use **✕ Delete** mode |
| Undo / Redo | Ctrl+Z / Ctrl+Y (or ⌘Z / ⌘Y on Mac), or buttons in palette |
| Edit node | Double-click |
| Fit all | **Fit** button (bottom-right of canvas) |
| Auto layout | **Auto Layout** button — arranges into Claim → sub-claims → defeaters → evidence hierarchy |

---

## Baconian Confidence Meter

The sidebar shows a **Confidence** score based on *Baconian probability* (eliminative induction):

```
confidence = addressed_defeaters / total_defeaters
```

A defeater is *addressed* when it has at least one outgoing connection (i.e., it has been rebutted, undermined, or undercut by further argument). The meter shows green (≥ 80 %), orange (40–79 %), or red (< 40 %).

---

## Save, Load, Export

| Action | Where |
|--------|-------|
| Save to browser | **Save** button or Ctrl+S — stores in `localStorage` keyed by case name |
| Load from browser | **Load** button — lists saved cases |
| Export JSON | **Export** button — downloads `<name>.json` |
| Import JSON | **Import** button — loads a previously exported file |
| New case | **New** button — prompts if unsaved changes exist |

---

## File Structure

```
assurance-case-builder.html   ← the entire app (single file, no dependencies)
lecture-2/                    ← course lecture slides (remark.js presentations)
  assurance-case.html         ← slide deck entry point
  assurance-case-exercise.html
  include/
    assurance-case.md         ← slide source (Markdown)
  images/                     ← diagrams and SVG exports
```

---

## Lecture Material

The `lecture-2/` folder contains slide decks for the GSN / assurance case lecture, built with [remark.js](https://remarkjs.com). Open `lecture-2/assurance-case.html` in a browser to view the slides. Slides source is in `lecture-2/include/assurance-case.md`.

---

## Development Notes

- The builder is a single HTML file with all CSS and JavaScript inline — no build step, no bundler, no CDN.
- SVG canvas uses a `<g id="world">` transform for pan/zoom; all coordinates are in canvas space.
- Full re-render (`renderCanvas()`) is called on every state mutation. `render()` additionally rebuilds the sidebar; property edits call only `renderCanvas()` to avoid destroying textarea focus mid-keystroke.
- Undo/redo is implemented as JSON snapshots (`history[]`, max 25 states).
- Auto-layout uses BFS layer assignment + post-order X packing; auxiliary nodes (Context, Justification, Inference) are placed to the right of their parent rather than in the main tree.
