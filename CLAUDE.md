# CLAUDE.md

## What this is

An interactive map of the bank's systems and their integrations ("Integration Atlas"). It opens on a bank-wide overview: one tile per area of the bank (Lending, Deposits, Accounting, Loan servicing, Collections) around the COCC core. Each mapped area opens its own map. Only Lending is mapped so far; the other areas show as "Not mapped yet".

The entire site is one self-contained file: `lending_technology_map.html`. It holds the HTML, CSS and vanilla JS inline, with no build step, no dependencies, no external fonts or CDNs, and no other files.

Keep it that way. The Download button saves the page as a single file for offline sharing, so anything external (a CDN script, a font, a separate `.js`/`.css`/image file) would break the downloaded copy.

## Running it

Open `lending_technology_map.html` in a browser. It works from `file://`, so no server is needed.

## File layout

1. `<style>`: all CSS. Colour tokens are on `:root` (`--core-a/b`, `--mort-a/b`, etc., plus one pair per area: `--lend-a/b`, `--dep-a/b`, `--acct-a/b`, `--serv-a/b`, `--coll-a/b`). The mobile layout is under `@media (max-width:750px)`.
2. Markup: topbar (brand link to the overview, search, Print, Download, Reset), heading (breadcrumb, title, blurb, all filled by the script), filter bar (`#filters`), canvas/world, details panel, legend (built per view), zoom controls, footer, and the empty `#printSheet`.
3. `<script>`: one IIFE.
   - **DATA section** (between the `====` banners): `AREAS`, `OVERVIEW`, one map object per mapped area (`LENDING`, with `LENDING.nodes`, `.edges`, `.lanes`, `.captions`, `.units`), `MAPS`, `FLOW`, `KIND_LABEL`, `KIND_SWATCH`. Most content changes only touch this section.
   - Everything below the banner is rendering and interaction. Each area's map and the overview are **views** built once from the data by the same code (`buildView`: lanes, nodes, edges with animated particles, legend, filter chips) and swapped into the world by `show()`. Then pan/zoom/drag, selection panel, business unit filter, URL hash routing, search, print, download, starfield.
   - **Views and the URL**: `#overview` (or no hash) is the overview, `#lending` an area, `#lending&units=consumer,indirect` a filtered area. Old `#units=…` links open Lending. Opening a view (tile click, breadcrumb, search, portal) adds a history entry so back/forward work; changing the filter replaces it. `#deposits` for an area that isn't mapped opens the overview with that tile selected.
   - **Overview**: built by `overviewView()` from `AREAS` and the maps; `layoutOverview()` places the tiles on an ellipse around the core (two columns on phones), so adding an area needs no positions. Lines are derived from the data, never typed in: an area links to the core when any of its edges touches `OVERVIEW.core`, to another area when it uses a portal to one of that area's systems, and an unmapped area gets a dotted "Not mapped yet" line. Clicking a mapped tile opens its map; clicking an unmapped tile shows a "Not mapped yet" panel.
   - **Search** (topbar, `/` focuses it, a toggle button on phones): matches system name, then type and tag, across every area. Choosing a result opens its area, clears a filter that hides it, centres and selects it. Portals are not listed; a system is found in its home area.
   - **Print**: `buildPrint()` runs on `beforeprint` (so the button and Ctrl/Cmd+P both work). It fills `#printSheet` with a light landscape sheet. From an area: a clone of the current map (respecting the filter and dragged positions) with a legend, then a systems directory grouped by business unit. From the overview: the overview (always in the wide layout) with a connections/areas legend, then a full systems directory for each mapped area. Only the `@media print` styles show it. Print colours for connections are in `PRINT_COLOR`, and the legend text for system types and connections is in `KIND_INFO` / `FLOW_INFO`. If you add a `kind` or `flow`, add it there too (and to `KIND_SWATCH` for the on-screen legend). `.p-map` needs `contain:strict`, otherwise Chrome counts the nodes' unclipped overflow and shrinks the whole printout.

## Editing the data

- **Area**: `{ id, label, icon, color, desc }` in `AREAS`, in overview order (clockwise from the top). `color` is a token prefix from `:root` (`"lend"` → `--lend-a` / `--lend-b`); add a new pair for a new area. `desc` is one short line for the tile.
- **Mapping an area**: copy the shape of `LENDING` (`title`, `blurb`, `W`, `H`, `center`, `contextUnits`, then `.nodes`, `.edges`, `.lanes`, `.captions`, `.units`), and add it to `MAPS` under the area's id (`const MAPS = { lending:LENDING, deposits:DEPOSITS }`). The overview tile, counts, core line, search and print directory pick it up automatically. `center` is the system the halo and orbits circle; `contextUnits` are units always shown in full as context when filtering (Lending: `core`, `ai`).
- **Node**: `{ id, name, type, tag?, icon, kind, unit, x, y, desc }`
  - `id` must be unique across **all** maps (search and portals find a system by id).
  - `kind` sets the colour and style: `core | mortgage | commercial | consumer | indirect | smallbiz | storage | construction | ai | integration`.
  - `unit` is the business unit(s) used by the filter. It's a single id or an array (e.g. Built is `["construction", "commercial", "residential"]`). Each id must exist in that map's `units`.
  - `x`/`y` are positions in the map's `W` × `H` world. Lending is 2240 × 980, laid out as three 700-wide columns (commercial and consumer on the left, small business, core and indirect in the centre, residential and AI on the right). COCC is at (1120, 420). Place a node inside its unit's lane box. Keep the grid: nodes in a row are 230 apart and rows about 110 apart.
  - The same vendor used by two units gets two nodes with different ids (e.g. `xactus` / `xactusc`, `experian` / `experiansb`). Say so in `desc`.
- **Portal** (a system from another area shown in this map): `{ ref:"<system id>", x, y, unit?, desc? }` in the map's nodes. It keeps the system's name, icon and colour, and edges in this map can use it by that id. It is drawn dimmed and dashed with its home area as its tag; clicking it opens the home area and selects the system. A reference to a `core` system (e.g. `{ ref:"cocc", unit:"core", x, y, desc:"…" }`) is drawn at full strength instead and selects in place, since every area hangs off the core; its panel has an "Open in the Lending map" button. `unit` is optional (without it, the portal shows when a connected unit is filtered in); `desc` replaces the description in this map. References are not counted as systems of the area. COCC is defined in Lending today, so other areas reference it rather than defining it again.
- **Edge**: `{ from, to, flow, manual?, note?, bend?, c1?, c2?, via? }`
  - `flow`: `both | one | none | inactive`. For `one`, data moves from `from` to `to`. (`unmapped` is used only by the overview.)
  - `manual: true` draws a pink dashed line for a manual process.
  - `c1`/`c2` are optional control-point offsets for hand-tuned curves.
  - `via: [[x, y], ...]` routes a long line through world waypoints with rounded corners (e.g. Built → VeraPath runs around the bottom of the map). Use it when a curve would otherwise pass under an unrelated node.
- **Lane / caption**: each has `units: [...]` so it hides when those units are filtered out.
- When you add a node or connection, update the descriptions (`desc`) of the hub or core nodes it connects to as well. They describe their connections in prose.
- Never invent systems, vendors or integrations for an area; leave it unmapped until real data is available.

## Conventions

- Match the existing style: compact single-line CSS rules, `const`/arrow functions, 2-space indent, no frameworks.
- Set user-supplied text with `textContent`, not `innerHTML`.
- Respect `prefers-reduced-motion`: no particles or animations when it's set.
- Check the layout at desktop width and at phone width (~390px).
- Add new views through `buildView`/`show` rather than a separate rendering path.

## Verifying changes

Chromium and Playwright are available in cloud sessions. Load the file, open each area from the overview and come back (breadcrumb, browser back/forward, reload), click through the filter chips, search for a system from the overview, and use the Download button (then open the downloaded file). Check the print output from the overview and from an area (dispatch `beforeprint`, then `page.pdf({ landscape:true })` at Letter and A4), check there are no console/page errors, and take screenshots of the overview and each area at 1440×900 and 390×800.
