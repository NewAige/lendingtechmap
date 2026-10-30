# CLAUDE.md

## What this is

An interactive map of the bank's lending systems and their integrations ("Integration Atlas"). The entire site is one self-contained file: `lending_technology_map.html`. It holds the HTML, CSS and vanilla JS inline, with no build step, no dependencies, no external fonts or CDNs, and no other files.

Keep it that way. The Download button saves the page as a single file for offline sharing, so anything external (a CDN script, a font, a separate `.js`/`.css`/image file) would break the downloaded copy.

## Running it

Open `lending_technology_map.html` in a browser. It works from `file://`, so no server is needed.

## File layout

1. `<style>`: all CSS. Colour tokens are on `:root` (`--core-a/b`, `--mort-a/b`, etc.). The mobile layout is under `@media (max-width:750px)`.
2. Markup: topbar (Download, Reset), heading, filter bar (`#filters`), canvas/world, details panel, legend, zoom controls, footer.
3. `<script>`: one IIFE.
   - **DATA section** (between the `====` banners): `NODES`, `EDGES`, `FLOW`, `LANES`, `CAPTIONS`, `UNITS`, `KIND_LABEL`. Most content changes only touch this section.
   - Everything below the banner is rendering and interaction: lanes, nodes, edges with animated particles, pan/zoom/drag, selection panel, business unit filter, download, starfield.

## Editing the data

- **Node**: `{ id, name, type, tag?, icon, kind, unit, x, y, desc }`
  - `kind` sets the colour and style: `core | mortgage | commercial | consumer | indirect | smallbiz | storage | construction | ai | integration`.
  - `unit` is the business unit(s) used by the filter. It's a single id or an array (e.g. Built is `["construction", "commercial", "residential"]`). Each id must exist in `UNITS`.
  - `x`/`y` are positions in the 2440 × 1780 world. COCC is at the centre (1220, 820). Place a node inside its unit's lane box.
  - The same vendor used by two units gets two nodes with different ids (e.g. `xactus` / `xactusc`, `experian` / `experiansb`). Say so in `desc`.
- **Edge**: `{ from, to, flow, manual?, note?, bend?, c1?, c2? }`
  - `flow`: `both | one | none | inactive`. For `one`, data moves from `from` to `to`.
  - `manual: true` draws a pink dashed line for a manual process.
  - `c1`/`c2` are optional control-point offsets for hand-tuned curves.
- **Lane / caption**: each has `units: [...]` so it hides when those units are filtered out.
- When you add a node or connection, update the descriptions (`desc`) of the hub or core nodes it connects to as well. They describe their connections in prose.

## Conventions

- Match the existing style: compact single-line CSS rules, `const`/arrow functions, 2-space indent, no frameworks.
- Set user-supplied text with `textContent`, not `innerHTML`.
- Respect `prefers-reduced-motion`: no particles or animations when it's set.
- Check the layout at desktop width and at phone width (~390px).

## Verifying changes

Chromium and Playwright are available in cloud sessions. Load the file, click through the filter chips and the Download button, check there are no console/page errors, and take screenshots at 1440×900 and 390×800.
