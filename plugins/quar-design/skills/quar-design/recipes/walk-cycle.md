# Recipe: a character walk cycle with a rig

**Goal:** a simple character made of shapes, on a skeleton, walking on the spot in a seamless
24-frame loop at 24 fps (1 s, two steps). The legs are moved by inverse kinematics: you key where
each ankle goes, and the bones above it bend to reach it — far easier to get right than turning
each bone.

## 1. Draw the character in parts

`set_document {frame_rate: 24}` first (an export's fps is at most the document's), and give the
artboard a 24-frame timeline (`set_artboard_duration`). Then one shape per part, standing
straight — the rig is bound in this rest pose — in a 1080 × 1080 artboard at (0, 0), Y up:

| Part | Shape | Centre | Size |
| --- | --- | --- | --- |
| Ground | rectangle | (0, −372) | 1080 × 72 (its top, −336, is where the feet stand) |
| Shadow | ellipse, `#8d99a8` at 0.45 (`layer-blur` 6) | (0, −336), in front of the ground | 240 × 24 |
| Body | rounded rectangle | (0, 70) | 130 × 220, radius 60 |
| Head | ellipse | (0, 250) | radius 70 |
| Thigh L, Thigh R | rounded rectangles | (0, −95) | 44 × 170, radius 22 |
| Shin L, Shin R | rounded rectangles | (0, −245) | 40 × 170, radius 20 |
| Foot L, Foot R | ellipses | (18, −328) | 36 × 15 |
| Arm L, Arm R | rounded rectangles | (0, 80) | 34 × 170, radius 17 |

The far limbs (R) darker and behind the body (add them first); the near ones in front (add them
last). Name the shapes so they can't be mistaken for bones ("Thigh L shape"…), and give them ids
(`thigh-l-shape`…): binding takes ids when names repeat.

**Checkpoint:** a dry-run picture at frame 0 (`preview_node` the artboard): the parts meet at the
joints, the far limbs behind, the feet on the ground.

## 2. The skeleton

`add_bone` from joint to tip in the artboard's space. A bone's `rotation` is relative to the bone
it hangs from (0 = carrying on in its direction), so keep each chain pointing the way it hangs:

```json
[
  { "op": "add_bone", "id": "pelvis", "name": "Pelvis", "parent": "frame", "from": { "x": 0, "y": -10 }, "to": { "x": 0, "y": -20 } },
  { "op": "add_bone", "id": "thigh-l", "name": "Thigh L", "parent": "pelvis", "length": 150 },
  { "op": "add_bone", "id": "shin-l", "name": "Shin L", "parent": "thigh-l", "length": 150 },
  { "op": "add_bone", "id": "thigh-r", "name": "Thigh R", "parent": "pelvis", "length": 150 },
  { "op": "add_bone", "id": "shin-r", "name": "Shin R", "parent": "thigh-r", "length": 150 },
  { "op": "add_bone", "id": "spine", "name": "Spine", "parent": "frame", "from": { "x": 0, "y": -10 }, "to": { "x": 0, "y": 170 } },
  { "op": "add_bone", "id": "neck", "name": "Neck", "parent": "spine", "length": 60 },
  { "op": "add_bone", "id": "arm-l", "name": "Arm L", "parent": "spine", "length": 160, "angle": -90 },
  { "op": "add_bone", "id": "arm-r", "name": "Arm R", "parent": "spine", "length": 160, "angle": -90 }
]
```

The pelvis points down, so the legs hang straight on (their rest rotation is 0). The spine is a
second root pointing up; the arms hang from its tip, pointing down — their rest rotation relative
to the spine is ±180 (`get_node` shows −180), so they swing between 160 and 200, every key on that
side (rotations tween as written: a key at −180 next to one at 200 would spin the arm round). Read
any bone's rest value from `get_node` or `get_timeline` (not `get_scene_context`'s `rotation`,
which is the bone's direction on the page). The server's pictures don't draw bones: check the
joints from the add_bone answers, or `placed`.

Bind each part to its own bone — `bind_mesh {nodes: [<shape ids>], bones: [<bone id>]}` — at most
10 `bind_mesh` ops in a batch (bind several shapes in one op where they share a bone: each foot
with its shin, the head with the neck, the body with the spine).

## 3. The legs: IK on the ankles

First give the knees a slight forward bend at rest — `pose_bone` with no frame: thigh-l and
thigh-r `rotation: 8`, shin-l and shin-r `rotation: -16`. IK bends a chain the way it already
bends: from straight legs a knee can fold backwards.

`set_ik_target {bone: "shin-l", depth: 2, at, frame}` makes a two-bone chain (thigh and shin) and
moves its target — the ankle — to `at`, keyed at `frame`. Give `depth` only on each leg's first
key (the chain is made then; again, it's refused). Key both ankles every 3 frames, `easing:
"linear"` (dense keys; an ease on each stops the motion every 3 frames). The left foot is planted
for frames 0–12, sliding back at a steady speed, then lifts and swings forward; the right foot
does the same 12 frames later:

| Frame | Left ankle | Right ankle |
| --- | --- | --- |
| 0 | (90, −320) | (−90, −320) |
| 3 | (45, −320) | (−62, −297) |
| 6 | (0, −320) | (0, −284) |
| 9 | (−45, −320) | (62, −297) |
| 12 | (−90, −320) | (90, −320) |
| 15 | (−62, −297) | (45, −320) |
| 18 | (0, −284) | (0, −320) |
| 21 | (62, −297) | (−45, −320) |
| 24 | (90, −320) | (−90, −320) |

The body bobs twice a cycle, lowest just after each contact: `animate` the pelvis's and the
spine's `y` (both roots, the same keys; their rest is −10) — `y` −25 at 0, −29 at 3, −16 at 6,
−12 at 9, −25 at 12, −29 at 15, −16 at 18, −12 at 21, −25 at 24, `linear`. The lower hip keeps the
knees soft, so no leg locks straight at full stride.

## 4. Arms, head, and the shadow

- Arms swing opposite the legs, 2 frames late (overlap), `pose_bone {bone, rotation, frame,
  easing}` — Arm L: 165 at 0, 155 at 2 (`easeOutQuad`), 205 at 14 (`easeInOutQuad`), 165 at 24
  (`easeInQuad`); Arm R the mirror (195, 205, 155, 195). The far arm needs that swing (±25°) to
  show past the body.
- The head counters the bob, 2 frames late: the neck's rotation 1.5 at 0, −3 at 5, 3 at 11, −3 at
  17, 3 at 23, 1.5 at 24.
- The contact shadow (from step 1) breathes with the bob: its `scale` {1.05, 1.05} at the low
  points (3, 15), {0.94, 0.94} at the high ones (9, 21), back at 0 and 24, `linear`.
- Bound to the shins, the feet tip with them: a few pixels of heel strike and toe push-off, which
  reads as a walk at normal size.

**Checkpoint:** `render_preview {node: "frame", frames: [0, 3, 6, 9, 12, 15, 18, 21, 24]}` — legs
alternate, a foot always on the ground, the swing foot lifting past the planted one at 6 and 18,
the body bobbing twice, frames 0 and 24 the same. Bound shapes pose in the server's pictures; a
character that's a picture (an image bound to bones) needs `capture_canvas` to check.

## 5. Export

A GIF loops by itself: `export {project_id, format: "gif", node: "frame", fps: 24}`; Lottie keeps
the bound shapes as animated paths (`format: "lottie"`).

**Common mistakes:** binding before the character stands straight; IK from straight legs (knees
fold backwards); `depth` on every IK key; shapes and bones with the same names; keying a bone's rotation around 0 when its rest isn't 0 (an arm under an upward spine rests
at 180); easing every dense key; a 30 fps document for a 24 fps cycle; feet that float or sink
(check the ankles against the ground line); a loop whose frame 24 doesn't match frame 0.
