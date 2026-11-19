# CLAUDE.md — Nodle

Nodle (Node + Noodle) is a node, graph and state-machine based design/play interface that runs in the browser.
It began as a vanilla-JS node editor MVP and grows one feature at a time. See `ROADMAP.md` for what's done and what's next.

## Hard constraints
- **Vanilla only.** Plain HTML, CSS and JavaScript. No frameworks, no bundler, no npm runtime dependencies, no build step.
- **Must run by double-clicking `index.html`** (a `file://` URL). That means no ES-module `import` between local files, since browsers block that over `file://`. If the code gets split up, use classic `<script src>` tags loaded in order, sharing a single namespace object.
- **Old save files must keep loading.** Every saved board is JSON. Never break loading of an older file (see "Save format" below).
- Keep it dependency-free. Ask before adding any third-party library, even from a CDN.

## Layout (current)
Everything is in `index.html`: the `<style>` block, then the markup, then one `<script>` wrapped in an IIFE. Main sections of the script, in order:

| Section | Key functions |
|---|---|
| constants & helpers | `uid()`, `esc()`, `isHex()`, `trunc()`, `byName` |
| state | `emptyDoc()`, `sampleDoc()`, `normalize(doc)`, `serialize()`, `persist()` |
| geometry | `toWorld(e)`, `clipToRect()` |
| hierarchy | `computeHierarchy()`, `applyHierarchyLayout()` |
| canvas rendering | `renderCanvas()` (SVG: links layer, then nodes layer) |
| mutations | `addNode`, `addLink`, `duplicateNode`, `deleteNode`, `deleteLink`, `addLinkType`, `deleteLinkType`, `addAttribute`, `deleteAttribute`, `moveAttribute`, `addOption`, `deleteOption` |
| selection & view | `select(kind, id, {tab})`, `centerOn`, `fitView`, `zoomBy` |
| pointer interaction | pointerdown/move/up on the SVG: node drag, pan, link drag from `.handle` |
| sidebar | `renderSidebar()` → `renderInspector()` / `renderTypes()` / `renderAttributes()` / `renderOutline()` |
| toolbar & file I/O | `saveJson()`, `readFile()`, `loadDoc()` |
| ui bits | `modal()`, `toast()`, `copyText()` |

Sidebar tabs are `inspect`, `types` (link types), `attrs` (attribute definitions) and `outline` (hierarchy).

## Data model
```js
{
  format: 'nodle', version: 2, title,   // v1 files used format 'node-board'; both load
  defaultLinkTypeId,                 // the "main" link type; can't be deleted
  linkTypes: [{ id, name, color, visible }],
  attributes: [{ id, name, kind: 'text'|'picklist', options: [{ id, label, color }] }],  // array order = display order
  nodes:     [{ id, name, description, color, isRoot, x, y, attributes: { [attrId]: string } }],   // x,y = top-left in world coordinates
  links:     [{ id, name, from, to, typeId }],                   // directed from → to; no self-links
  hierarchy: { typeIds: [] },        // link types followed when computing levels
  ui: { showLevels }, view: { x, y, k }
}
```
- IDs are strings (GUIDs from `uid()`). Treat them as opaque, because loaded files can contain any string.
- Colors are `#rrggbb`. Validate with `isHex()` before putting them into markup.
- Attributes:
  - A node value is the text itself for `text`, or an **option id** for `picklist`, so renaming an option doesn't break values. "Not set" means the key is missing, never `''`.
  - `node.attributes` is created with `dict()` (`Object.create(null)`), so any attribute id, even `__proto__`, is a safe key.
  - Options are shown sorted by label (`byLabel`). Their stored order doesn't matter, and each gets a palette color when created.
  - `options` is always `[]` for text attributes.
  - An attribute's kind can't be changed while any node has a value for it.
  - Deleting an attribute or option clears the matching values, after a confirm modal if any nodes are affected.

## Save format and migrations
- `normalize(doc)` is the **only** way data gets in, whether from a file, localStorage or the sample board. It validates every field, fills in defaults and drops broken references (such as a link to a missing node).
- When you change the schema: bump `version` in `emptyDoc()` and `serialize()`, teach `normalize()` to upgrade every older version, and keep accepting `format: 'node-board'` as well as any new format name.
- Add new fields as optional with defaults, so older files still load.

## Conventions
- **Escape every user string** with `esc()` before it goes into an HTML or SVG template string. Names, descriptions and IDs all come from user files.
- After any data change, call `persist()` (autosaves to localStorage under `nodle.board` and updates the status bar).
- Rendering is "rebuild the layer from state":
  - `renderCanvas()` for visual changes, including on every pointermove during a drag
  - `renderAll()` when the structure changes (anything is added or deleted)
  - Inspector text inputs must **not** re-render the inspector while the user is typing, or the input loses focus. Update state, then call `renderCanvas()` and `persist()` only.
- The page also runs inside a sandboxed claude.ai frame. `alert`, `confirm` and `prompt` don't work there, so use `modal()` and `toast()`. `saveJson()` uses `claude.use('downloads')` when it exists and falls back to a normal `<a download>`. Keep both paths working.
- Keyboard shortcuts must ignore keystrokes while focus is in an input, textarea or select (see the keydown handler).
- Dark theme, styled with the CSS variables in `:root`. Reuse the existing classes (`.h`, `.f`, `.row`, `.box`, `.chip`, `.trow`, `.lrow`) before inventing new ones.

## Working on a feature
1. Read the relevant sections first, then propose a short plan (data-model change, UI, migration) before writing code.
2. Make focused changes. Don't reformat or restructure code you aren't changing.
3. Check it: open `index.html` in a browser (or run a headless Playwright smoke test if one is available). Exercise the feature, then **Save JSON → New → Open** to confirm a round trip. Also load an old-version file.
4. Update `ROADMAP.md` (move the item to Done) and this file if the data model or architecture changed.
5. One commit per feature, with a clear message.

If `index.html` grows past about 2,000 lines, propose splitting it into `css/`, `js/*.js` (classic scripts) before continuing.
