# Design rubric

Use this to plan a frame and to judge every picture you render. Decide the frame, the hierarchy
and the palette before you add a shape.

## The frame

| For | Artboard |
| --- | --- |
| Video, slides, YouTube | 1920 × 1080 |
| Square social post | 1080 × 1080 |
| Vertical story, reel | 1080 × 1920 |
| App or web micro-interaction (Lottie) | 512 × 512 (or the size it's shown at, ×2) |
| Banner | the placement's size (e.g. 1200 × 628, 728 × 90) |

- The artboard's centre is its `x`, `y`; its children's places are in its own space, so with the
  artboard at (0, 0) a child at (0, 0) sits in its middle and (−960, 540) is the top-left corner of
  a 1920 × 1080 frame.
- Keep a **safe margin** of 5–8 % of the short side clear of text and important shapes (~60 px
  on 1080). Backgrounds and decoration can bleed to the edge.
- Turn on `clipContent` on an artboard when shapes fly in from outside it.

## Hierarchy

- **One focal point** per frame: the thing the viewer should see first is the biggest, the most
  contrasted, or the one that moves. Everything else supports it.
- **Scale contrast:** the headline at least ~1.5–2× the next level of type; a hero shape clearly
  bigger than its neighbours. Timid size steps look like mistakes.
- **Grouping:** related things close together, unrelated things apart (more space between groups
  than within them). Group related shapes (`group_nodes`) and name them.
- **Alignment:** align to a few lines — the centre, or a left edge, or a grid. Read `bounds` (from
  a dry run's `placed`, or `get_scene_context`) to check edges really line up; eyeballing a picture
  misses a few pixels. Text bounds are its glyphs' in Inter.
- **Spacing scale:** pick spacings from one scale (8 px base: 8, 16, 24, 32, 48, 64, 96…) and reuse
  them.

## Colour

- **A small palette:** 1 dominant (background or large areas, ~60 %), 1 secondary (~30 %), 1 accent
  (~10 %, for the focal point or the call to action), plus neutrals. Five colours is plenty.
- Choose with intent: warm accents on cool grounds (or the reverse) separate the focal point.
  Avoid pure `#000000` and `#ffffff` for large areas — near-black and off-white feel finished.
- **Contrast:** text against what's behind it at least 4.5 : 1 (3 : 1 for big display type).
  If in doubt, darken the background or put text on a solid panel.
- Gradients: subtle (two neighbouring hues, or one hue light → dark), along a meaningful
  direction (light from above: `angle` 270, top → bottom).
- With a brand, use its colours exactly (ask for them, or read them from the user's files).

## Type

- Inter (what the server draws): weights 400, 500, 600, 700; italics. Use two weights at most —
  e.g. 700 for headlines, 400–500 for the rest.
- A size scale (×1.25 or ×1.333): e.g. 24, 32, 42, 56, 75, 100 px. On 1920 × 1080: headline
  72–120, subhead 36–48, body 24–32; nothing under 20.
- Big display type: tighten `letterSpacing` a little (it's in pixels: −1 to −3 % of the size,
  e.g. −2 at 100 px) and `lineHeight` 1.0–1.15; body `lineHeight` 1.4–1.5.
- One idea per line; short lines. Centre-align short titles; left-align anything longer.
- Text is as wide as its line (`textResize: "auto-width"`, the default). For a column that wraps,
  `"auto-height"` with `boxWidth`; for a fixed box, `"fixed"` with `boxWidth` and `boxHeight`.

## Shape and depth

- Consistent corner radii (one or two values) and stroke widths across a piece.
- Depth with restraint: one light direction; soft shadows (`drop-shadow`, `offsetY` −8 to −24,
  `blur` 24–48, opacity 0.15–0.35 in a dark tint of the background, not grey).
- Layers: background → mid-ground → subject → foreground details → type on top.

## The checklist for every picture

Look at the rendered picture (not at your intent) and answer each:

1. Is there one clear focal point, and is it what the user asked about?
2. Is anything cut off, touching the edge, or inside the safe margin that shouldn't be?
3. Do edges and centres line up where they should? (Check `bounds`.)
4. Is the spacing even within groups and larger between them?
5. Is text readable — size, weight, contrast against what's behind it?
6. Are there more than ~5 colours, or colours that fight?
7. Does anything look accidental (an overlap, a tiny offset, a stray shape)?
8. Would the user's brand or request recognise itself here?
9. Will it work on the background it's shown on (an app's white or dark screen, a slide deck)?

Fix what fails, then render again. Two or three rounds is normal; stop when the checklist passes.
