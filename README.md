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

### Branch indicators (automatic)

These appear on their own — there's nothing to add from the palette:

- **◆ Black diamond** — appears automatically below any Claim or Defeater with no outgoing connections, marking the branch as *undeveloped*. Disappears as soon as you connect something below it.
- **⬤ Grey circle** — appears automatically below any Evidence node with no outgoing connections, marking the branch as *complete*. Disappears as soon as you connect something below it.

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

## Case Checklist & Claim-Writing Tips

Below the confidence meter, the **Case Checklist** flags structural issues across the whole diagram — disconnected nodes, undeveloped branches, missing descriptions, top-level claims with no Context, Claims that read like an action/method instead of a property, and Evidence that reads like a full sentence instead of a noun-phrase. Click any item to jump to that node.

When you add or edit a node (popup editor or the sidebar Properties panel), a short writing tip for that node type appears with a good example, plus a live warning if your text looks off. These are heuristic suggestions, not hard rules — they don't block saving.

---

## Save, Load, Export

| Action | Where |
|--------|-------|
| Save to disc | **Save to Disc** button or Ctrl+S — downloads a `<name>.gsn.json` file |
| Open a file | **Open File** button — loads a previously saved `.gsn.json` file |
| Export SVG | **Save SVG** button — downloads a `<name>.svg` |
| New case | **New** button — prompts if unsaved changes exist |
| Crash recovery | Every change is auto-saved to `localStorage`. On reopening the tool, you'll be prompted to restore it if found |

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
