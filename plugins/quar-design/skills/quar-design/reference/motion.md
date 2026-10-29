# Motion

Frames count at the document's `frameRate` (30 by default; 60 for smooth UI work). At 30 fps:
6 frames = 0.2 s, 15 = 0.5 s, 30 = 1 s.

## Timing

| Motion | At 30 fps | Seconds |
| --- | --- | --- |
| A button press, a toggle | 6–10 frames | 0.2–0.33 |
| A UI element entering | 9–15 | 0.3–0.5 |
| A title or logo entering | 15–30 | 0.5–1 |
| A camera-like move, a big reveal | 30–60 | 1–2 |
| A hold for reading | ~1 s per 3 words, at least 1 s | |

Small things move fast; big things move slower. Leave **holds**: motion reads only when it stops.

## Easing (the curve into a keyframe)

- `easeOutCubic` / `easeOutQuart` — **entering**: arrive fast, settle softly. The default.
- `easeInCubic` — **leaving**: start slowly, exit fast.
- `easeInOutCubic` — **moving** from one place to another on screen.
- `easeOutBack` — arrive with a small overshoot (playful, UI pops). `easeInBack` — pull back before
  leaving (anticipation built in).
- `easeOutBounce` — land with bounces (falling objects, logos). Fall with `easeInQuad` before it.
- `easeOutElastic` — a spring (scale pops, wobble). Use sparingly.
- `linear` — only for constant motion: a spin, a scrolling texture, a loop's steady drift.
- `easeOutExpo` / `easeInOutExpo` — sharp, "tech" motion.
- `hold` (in `animate`) — a cut: the value before stays until this keyframe's frame, then jumps
  (a cursor blink, a pop on, a hard cut between poses).
- All names: `linear`, `easeIn|Out|InOut` + `Sine`, `Quad`, `Cubic`, `Quart`, `Quint`, `Expo`,
  `Circ`, `Back`, `Elastic`, `Bounce`; `hold`. Or `[x1, y1, x2, y2]` (a cubic Bézier, like CSS).
  `get_animatable_properties` lists them.

## Principles that make it look designed

- **Anticipation:** a small move the other way before a big one (a 2–4 frame dip before a jump, a
  slight scale-down before a pop).
- **Follow-through and overlap:** parts don't stop together. Offset related properties or parts by
  2–4 frames (the scale settles 3 frames after the position lands; a tail stops after the body).
- **Staggering:** items in a list or letters in a word start 2–5 frames apart (`move_keyframes`
  shifts a copy's keys in time). Stagger in reading order, or from the focal point outwards.
- **Arcs:** natural paths curve. Animate `x` and `y` separately with different easings (x
  `easeOutCubic`, y `easeOutBack`) to bend a straight move.
- **Squash and stretch:** on impact squash (scale x up, y down — keep the area: 1.2 × 0.83), in fast
  travel stretch along the motion; return to 1 over 4–8 frames.
- **Secondary motion:** a small reaction elsewhere (a shadow shrinking as an object rises, a glow
  pulsing as it lands) makes a scene feel alive.
- **One thing at a time:** sequence the important moves; don't start everything at frame 0.

## Loops

- The value at the timeline's last frame equals the value at frame 0 for every animated property —
  keyframe both ends with the same value. For a cycle of N frames, frame N should look like frame 0
  (the export's last frame is N − 1, so the jump is seamless).
- Spins: 0 → 360 with `linear` over the loop.
- Check: `render_preview` with frames `[0, N]` side by side — they should match.

## Checking motion

- `render_preview` with `count: 10` across the whole timeline, or `frames: […]` at the key moments
  (the start of each move, the impact, the settle). Each cell is labelled with its frame.
- Look for: things that start together but shouldn't; moves too fast to read (a big jump between
  two cells); holds too short; elements popping on without an entrance; a loop's seam.
- `get_timeline` lists every keyframe with its easing: use it to check the plan was made as meant.
