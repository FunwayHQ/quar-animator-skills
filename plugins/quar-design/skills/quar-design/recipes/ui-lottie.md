# Recipe: a UI micro-interaction, exported to Lottie

**Goal:** a small, crisp interaction — a "like" heart that pops and bursts — exported as Lottie
JSON for a web or mobile app. 512 × 512, 30 frames at 30 fps (1 s), ending still (apps play it
once on tap).

## 1. Build it, Lottie-friendly

Lottie carries shapes, fills (solid and gradient), strokes, transforms and opacity well. It gets
no effects (shadows, blurs) and no blend modes — the export's details count them — and text is
carried as outlines.

The artboard must be transparent (`"fill": "none"`), so the app's own background shows through.
Layers are drawn in the order they're added: the dots first, then the ring, then the heart on top.

```json
[
  { "op": "add_node", "type": "artboard", "id": "frame", "name": "Like", "width": 512, "height": 512, "fill": "none" },
  { "op": "set_artboard_duration", "artboard": "frame", "duration": 30 },
  { "op": "add_node", "type": "ellipse", "id": "dot-1", "name": "Dot 1", "parent": "frame", "x": 0, "y": 195,
    "radius": 12, "fill": "#fb7185", "opacity": 0 },
  { "op": "add_node", "type": "ellipse", "id": "ring", "name": "Ring", "parent": "frame", "x": 0, "y": 0, "radius": 120,
    "fill": "none", "stroke": { "color": "#fb7185", "width": 12 }, "opacity": 0 },
  { "op": "add_node", "type": "path", "id": "heart", "name": "Heart", "parent": "frame", "x": 0, "y": 0,
    "svg": "M0 -60 C -40 -100 -110 -70 -110 -10 C -110 40 -50 80 0 110 C 50 80 110 40 110 -10 C 110 -70 40 -100 0 -60 Z",
    "fill": "#a1a1aa" }
]
```

(SVG path data is Y-down: the heart's point is at the bottom on screen.) Make 8 dots in all, one
every 45° on a circle of radius ~195 (alternating two warm colours), each at `opacity: 0`.

**Checkpoint:** dry-run with `preview_frame` at the burst moment (frame 11, once animated) or with
the dots' opacity set to 1 for a moment, to check the layout: everything centred, the dots inside
the frame with a margin.

## 2. Animate

- Heart `scale`: {1, 1} at 0 → {0.7, 0.7} at 4 (`easeOutQuad`, the press) → {1.2, 1.2} at 10
  (`easeOutCubic`, into the peak) → {1, 1} at 17 (`easeOutBack`, the spring).
- Heart `fill`: grey `#a1a1aa` at 0 → red `#e11d48` at 6 (it fills as it pops).
- Ring: `scale` {0.9, 0.9} at 6 → {1.35, 1.35} at 16 (`easeOutQuart`); `opacity` 0 at 6, 1 at 7,
  0 at 16; `stroke.width` 12 at 6 → 0 at 16 (`easeInQuad`).
- Dots: each moves from a third of the way out to its place, frames 7 → 16 (`easeOutCubic`), a
  frame after the ring starts (they're behind it, so they don't cross its stroke); `opacity` 0 at 7,
  1 at 9, 0 at 17; `scale` {0.4, 0.4} at 7 → {1, 1} at 11 → {0.6, 0.6} at 17.
- Everything still from 18 to 30.

**Checkpoint:** `render_preview {frames: [0, 4, 7, 10, 12, 14, 17, 29]}` — press, pop, ring and
burst, settle; the last frame clean (no leftover ring or dots). Check the idle grey heart against
both a white and a dark background: an app shows it on its own.

## 3. Export to Lottie

```text
export     {project_id, format: "lottie", node: "frame", width: 512, height: 512}
get_export {export_id, wait: true, include_file: true}
```

- Read the export's details: `unsupported` lists node types it skipped and counts layers whose
  effects or blend modes it left out — remove or replace those, and export again.
- Give the user the download link, the size, the frame count and frame rate, and how to play it
  (lottie-web / lottie-ios / lottie-android, once on tap; or `@lottiefiles/dotlottie`).

**Common mistakes:** a white artboard (the app can't show through); the heart under the ring;
an overshooting easing into the peak (it overshoots twice); a burst that collides with the ring;
effects Lottie drops; motion that doesn't end still; too long (UI feedback is 0.3–1 s).
