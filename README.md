# Assurance Case Confidence Maps Builder

A self-contained, browser-based tool for building **Goal Structuring Notation (GSN)** assurance cases. No installation, no server, no external dependencies — open `assurance-case-builder.html` directly in any modern browser.

Developed to support the *Software Assurance* course at the University of Nebraska Omaha, replacing Lucidchart for in-class exercises.

---

**Try it out here:** [Assurance Case Confidence Maps Builder](https://htmlpreview.github.io/?https://github.com/robinagandhi/assurance-case-editor/blob/main/assurance-case-builder.html)

---

## Getting Started

1. Open `assurance-case-builder.html` in any modern browser (Chrome, Firefox, Safari, Edge).
2. Click a node type in the left palette to add it to the canvas, or drag it onto an exact spot.
3. Double-click a node to edit its label and description.
4. Drag from the midpoint handles (◦) on a node's edges to connect nodes — or hover/select a Claim, Defeater, or Inference Rule and click the **+** button beside it to add an already-connected sub-claim.
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
| Pan | Drag on empty canvas, or Space + drag (one-finger drag on touch) |
| Zoom | Scroll wheel (zooms toward cursor), or pinch with two fingers on touch |
| Select | Click a node or edge |
| Multi-select | Shift+click nodes one at a time, or Shift+drag empty canvas for a marquee box |
| Move node(s) | Drag a selected node — drags the whole selection if more than one is selected |
| Nudge selection | Arrow keys (Shift+arrow for a bigger step) |
| Resize node | Select a node, then drag any blue corner handle |
| Connect nodes | Drag from a midpoint handle (◦), or use **→ Connect** mode in the palette |
| Quick-add child | Hover/select a Claim, Defeater, or Inference Rule, then click the **+** beside it |
| Duplicate | Select, then Ctrl/⌘+D |
| Copy / Paste | Ctrl/⌘+C, then Ctrl/⌘+V (repeatable); or right-click empty canvas → Paste, to drop at that spot |
| Right-click menu | Node: duplicate, copy, select subtree, change type, delete. Edge: delete. Empty canvas: add claim, paste |
| Delete | Select then press **Delete/Backspace** (deletes the whole selection), or use **✕ Delete** mode |
| Undo / Redo | Ctrl+Z / Ctrl+Y (or ⌘Z / ⌘Y on Mac), or buttons in palette |
| Edit node | Double-click |
| Fit all | **Fit** button (bottom-right of canvas) — fits visible (non-collapsed) nodes |
| Auto layout | **Auto Layout** button — arranges into Claim → sub-claims → defeaters → evidence hierarchy |
| Collapse/expand branch | Click the **−**/**+** toggle below a node that has outgoing connections |
| Find in diagram | Toolbar search box — Enter/Shift+Enter cycles matches, Escape clears |

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

## Outline View & Collapsing Branches

The sidebar has a **Checklist** / **Outline** tab switcher. Outline shows the whole case as a nested, clickable tree — useful for jumping around a large diagram. On the canvas itself, any node with outgoing connections gets a **−**/**+** toggle that collapses or expands everything below it. Collapse state is a view preference only — it isn't written to the saved `.gsn.json` file or the exported SVG, and clicking a checklist/outline entry auto-expands whatever was hiding it.

---

## Review Mode (Reviewer Notes)

Toggle **Review Mode** in the toolbar to reveal a *Reviewer Note* field on nodes — separate from the student's own claim/evidence text — in both the popup editor and the Properties sidebar. Intended for instructor or peer feedback on a specific node. Any node carrying a note shows a small amber **!** badge at all times, even with Review Mode off, so comments are easy to spot while browsing.

---

## Presenter Mode

Click **▶ Present** (floating bottom toolbar) to step through the case top-down, one node at a time, in the same order as the Outline view. The current node is ringed and everything else dims; a caption bar shows its label and text. Use →/Space and ← to move, Escape to exit. Editing and right-click are disabled while presenting; exiting restores your previous pan/zoom.

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
slides/                       ← course lecture slides (remark.js presentation)
  assurance-case.html         ← slide deck entry point
  include/
    assurance-case.md         ← slide source (Markdown)
  images/                     ← diagrams and SVG exports
```

---

## Lecture Material

The `slides/` folder contains the slide deck for the GSN / assurance case lecture, built with [remark.js](https://remarkjs.com). Open `slides/assurance-case.html` in a browser to view the slides. Slide source is in `slides/include/assurance-case.md`.

---

## Development Notes

- The builder is a single HTML file with all CSS and JavaScript inline — no build step, no bundler, no CDN.
- SVG canvas uses a `<g id="world">` transform for pan/zoom; all coordinates are in canvas space.
- Full re-render (`renderCanvas()`) is called on every state mutation. `render()` additionally rebuilds the sidebar; property edits call only `renderCanvas()` to avoid destroying textarea focus mid-keystroke.
- Undo/redo is implemented as JSON snapshots (`history[]`, max 25 states).
- Auto-layout uses BFS layer assignment + post-order X packing; auxiliary nodes (Context, Justification, Inference) are placed to the right of their parent rather than in the main tree.
- Selection is `state.sel` (primary) plus `state.multi` (a `Set` of additional ids); `selectedIds()` merges them. Collapse state (`state.collapsed`) and search matches (`state.searchMatchIds`) are session-only — never written to the saved `.gsn.json`.
- Touch input is translated into the existing mouse handlers (`onDown`/`onMove`/`onUp`) via synthetic event objects; two-finger pinch is handled separately for zoom.
- Presenter mode and the Outline view share one traversal helper (`flattenOutlineOrder()` / `buildOutlineTree()`) so both walk the case in the same top-down order.
