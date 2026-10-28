# Recipe: a bouncing logo

**Goal:** a logo (or a word) drops into a square frame, stretches as it falls, squashes on the
floor, bounces twice — lower each time — settles and holds. 90 frames at 30 fps (3 s).

## 1. Set up

- New project: `create_project {name}`; then a batch with `{"op": "set_document", "duration": 90,
  "frame_rate": 30}` and the scene.
- The logo sits in a group anchored **at its bottom**, so scaling squashes it onto the floor, not
  about its middle: the group at the floor line, the logo's shapes offset up by half their height.

```json
[
  { "op": "set_document", "duration": 90, "frame_rate": 30 },
  { "op": "add_node", "type": "artboard", "id": "frame", "name": "Logo", "width": 1080, "height": 1080,
    "x": 0, "y": 0, "fill": "#1b1724", "clipContent": true },
  { "op": "add_node", "type": "ellipse", "id": "shadow", "parent": "frame", "x": 0, "y": -170,
    "radiusX": 170, "radiusY": 26, "fill": "#07060b", "opacity": 0.55 },
  { "op": "add_node", "type": "group", "id": "logo", "name": "Logo", "parent": "frame", "x": 0, "y": -150 },
  { "op": "add_node", "type": "rectangle", "id": "mark", "parent": "logo", "x": 0, "y": 150,
    "width": 300, "height": 300, "cornerRadius": 72,
    "fill": { "type": "linear", "angle": 270, "stops": ["#a855f7", "#6d28d9"] } },
  { "op": "add_node", "type": "text", "id": "word", "parent": "logo", "x": 0, "y": 150, "content": "Q",
    "fontFamily": "Inter", "fontWeight": 700, "fontSize": 200, "textAlign": "center", "fill": "#ffffff" }
]
```

The logo's middle rests near the frame's centre (the group's y −150 is its bottom, the floor),
the shadow 20 px under it. Use the user's logo when they have one (its SVG as a `path` with
`svg`, or its shapes) inside the same group, offset up by half its height.

**Checkpoint:** dry-run with `preview_frame: 0` and `preview_node: "frame"`; look: the logo
centred, resting on the shadow, the shadow clearly visible. Commit ("Logo scene").

## 2. Animate the group

`y` of the group (rest −150; the logo starts above the frame):

| Frame | y | Easing into it |
| --- | --- | --- |
| 0 | 900 | — |
| 18 | −150 | `easeInQuad` (falling, speeding up) |
| 30 | 30 | `easeOutQuad` (rising, slowing) |
| 40 | −150 | `easeInQuad` |
| 47 | −80 | `easeOutQuad` |
| 53 | −150 | `easeInQuad` |

`scale` of the group (`{x, y}`, keep the area): stretched while falling, squashed only on the
frame it lands, back as it rises.

| Frame | scale | Easing |
| --- | --- | --- |
| 0 | {1, 1} | — |
| 17 | {0.92, 1.09} | `easeInQuad` (stretching into the fall) |
| 18 | {1.2, 0.82} | `linear` (the impact) |
| 24 | {1, 1} | `easeOutBack` |
| 39 | {0.95, 1.05} | `easeInQuad` |
| 40 | {1.1, 0.9} | `linear` |
| 45 | {1, 1} | `easeOutQuad` |

A wobble in the rebound, never at the impact (a tilted landing puts a corner through the floor):
`rotation` 0 at 18, −3 at 23, 2 at 30, 0 at 38.

Secondary motion — the shadow grows and darkens as the logo comes down: its `scale` {0.4, 0.4} at
0, {1, 1} at 18, {0.75, 0.75} at 30, {1, 1} at 40, {0.9, 0.9} at 47, {1, 1} at 53; its `opacity`
0.15 at 0, 0.55 at 18, with the same rhythm.

Send them as `animate` ops in one batch; dry-run with `preview_frame: 18` (the first impact).

**Checkpoint:** `render_preview {frames: [0, 9, 16, 17, 18, 22, 30, 39, 40, 47, 53, 75]}` — the fall
speeds up, the stretch shows at 17, the squash sits on the shadow at 18 and 40, each bounce lower,
then a still hold to 90.

## 3. Polish

- A hold matters: the logo is still from frame 53 to 90 (over a second).
- For a loop: end with the logo leaving (y back up to 900 by frame 90, `easeInBack`), so frame 90
  matches frame 0 — or the GIF cuts from the hold to an empty frame.
- A soft shadow (`layer-blur`, radius 10–16) looks better in pictures and GIFs; Lottie drops blurs
  (its export says so), so for Lottie keep the shadow unblurred and lighter.

## 4. Export

- GIF for chat and social: `export {project_id, format: "gif", fps: 30, width: 540}` (any fps up
  to 50 plays at its real speed; unchanged frames cost nothing).
- Lottie for the web: `export {project_id, format: "lottie"}` — gradients, scale, opacity, rotation
  and text (as outlines) carry over.
- `get_export {export_id, wait: true}` → give the user the link (it works once, within an hour).

**Common mistakes:** squashing about the logo's middle (it leaves the floor); squashing before
contact; a tilted landing; a shadow that doesn't react or can't be seen; no hold at the end;
bounces that don't get smaller.
