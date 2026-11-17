# Nodle

Node and graph and state-machine based design/play interface.

Nodle is a browser-based board for building graphs: create nodes, connect them with typed links, organize them into hierarchies, and save everything as plain JSON. It's vanilla HTML/CSS/JS with no install and no build.

## Run it
Open `index.html` in any modern browser. That's it.

## What it does today
- **Nodes:** GUID (read-only), name, description, color
- **Links:** directed, named, typed. Drag a node's ● handle onto another node, or add links from the node panel.
- **Link types:** any number, each with a name and color. One default ("main") type. Show/hide types on the canvas.
- **Hierarchy:** mark root nodes, choose which link types to follow, and see levels (L0, L1, …) counted from the roots. One click lays the board out by level.
- **Save / Open:** boards save as `.json` files. Open a file or drop it onto the canvas. Work is also autosaved in the browser.

## Controls
| Action | How |
|---|---|
| Add node | Double-click the canvas · **+ Node** · `N` |
| Link nodes | Drag the ● handle onto another node |
| Pan / zoom | Drag empty space / scroll (pinch on a trackpad) |
| Fit everything | `F` or **Fit** |
| Delete selection | `Del` / `Backspace` |
| Save / Open | `Ctrl/⌘ S` / `Ctrl/⌘ O` |

## File format
A board is a single JSON object: `nodes`, `links`, `linkTypes`, plus a little view and settings state. See `CLAUDE.md` → *Data model*. Older files always remain loadable.

## Roadmap
See [`ROADMAP.md`](ROADMAP.md).
