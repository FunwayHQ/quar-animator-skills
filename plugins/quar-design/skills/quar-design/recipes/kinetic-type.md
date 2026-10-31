# Recipe: kinetic typography

**Goal:** a short line ("Make it move. Make it yours.") animated word by word in a 1920 × 1080
frame, one emphasised word per line, a hold for reading, and a clean exit. About 4–4.5 s at 30
fps, one sentence per slide.

## 1. Plan the text

- One `text` node per word (or short phrase), so each can move on its own, on one line, centred
  as a whole. Inter 700 at 160–190 px — kinetic type should fill the frame: measure, and size the
  line to 60–70 % of its width — with `letterSpacing` about −4 (pixels) and gaps of about a
  quarter of the size. The emphasised word in the accent
  colour, the rest near-white on near-black.
- Reading time: about 1 s for every 3 words after the last word lands.
- Two sentences → two artboards (slides), each with its own length (`set_artboard_duration`):
  they play one after another in presentations, in the editor's video exports, and in a GIF of
  the page — the frontmost first. Add slide 2 with `"index": 0` (behind slide 1) and place it
  below slide 1 on the page, so the canvas reads in the order they play.

## 2. Lay it out, measured

Words start where the one before ends plus a gap (about a third of the font size). Text `bounds`
are the glyphs' real width in Inter, so lay out in two steps:

1. Dry-run the words at x 0 with `"dry_run": true`: `placed` gives each word's `bounds`
   `[left, bottom, width, height]`.
2. Work out each word's `x` (its centre) so the gaps are even and the whole line is centred, and
   dry-run again with `preview_node` set to the artboard and `preview_size: 1024` to check.

```json
[
  { "op": "add_node", "type": "artboard", "id": "s1", "name": "Slide 1", "width": 1920, "height": 1080,
    "fill": "#0f0e13", "clipContent": true },
  { "op": "set_artboard_duration", "artboard": "s1", "duration": 66 },
  { "op": "add_node", "type": "text", "id": "s1-w1", "parent": "s1", "content": "Make", "x": 0, "y": 0,
    "fontFamily": "Inter", "fontWeight": 700, "fontSize": 150, "letterSpacing": -3, "fill": "#f5f3ff" },
  { "op": "add_node", "type": "text", "id": "s1-w2", "parent": "s1", "content": "it", "x": 0, "y": 0,
    "fontFamily": "Inter", "fontWeight": 700, "fontSize": 150, "letterSpacing": -3, "fill": "#f5f3ff" },
  { "op": "add_node", "type": "text", "id": "s1-w3", "parent": "s1", "content": "move.", "x": 0, "y": 0,
    "fontFamily": "Inter", "fontWeight": 700, "fontSize": 150, "letterSpacing": -3, "fill": "#a855f7" }
]
```

The emphasised word grows as it lands (below): put it in a group pivoted at its **left edge**,
so it grows to the right and the gap before it stays even — once the layout is measured:

```json
[
  { "op": "group_nodes", "nodes": ["s1-w3"], "id": "s1-pop" },
  { "op": "set_pivot", "node": "s1-pop", "at": "left" }
]
```

(the word stays where it is on screen; the group's `x`, `y` is now the word's left edge, and its
scale grows from there. Text can't be pivoted itself: the group carries the pivot.)

For letters that move one by one — a cascade, a wave, one letter recoloured — turn the word into
outlines once its layout is final: `convert_to_outlines {node: "s1-w3", id: "s1-w3-o"}` gives a
group of letter paths (`s1-w3-o-1`, `s1-w3-o-2`…) where the word was (the group needs an id of its
own: the word's is still taken while it's outlined); stagger their `y` and `opacity` by
2–3 frames. Do it last: outlines can't be edited as text.
Keyframes on the words are in their parent's space: inside an artboard that isn't at the page's
centre, key around each word's own `x`/`y`, not its `at`.

**Checkpoint:** the line centred, the gaps even, the emphasis reads first. Commit.

## 3. Animate each word in

For each word, starting 4 frames after the one before (`w1` at 0, `w2` at 4, `w3` at 8):

- `y`: from 40 px below its rest to its rest over 14 frames, `easeOutCubic`.
- `opacity`: 0 → 1 over the first 8 of those frames.
- The emphasised word's group: `scale` {0.85, 0.85} → {1.08, 1.08} over 10 frames (`easeOutCubic`)
  → {1, 1} 6 frames later (`easeOutBack`) — it arrives with a little extra, and last.

Alternative entrances (pick one per piece, don't mix many): a typewriter (`visibleCharacters` from
0 to the word's length, `linear`, one character every 2 frames), a slide from the side with the
artboard's `clipContent` hiding it, or a scale-up from 0.6 with `easeOutBack`.

## 4. Hold, then leave

- Hold everything still for the reading time (about 1 s here).
- Exit faster than the entrance: words staggered 2 frames, `y` up 30 px and `opacity` → 0 over 8
  frames, `easeInCubic`, finishing on frame N−1 (the last frame a slide of N frames plays).
- Make the second sentence the climax: a longer hold, a stronger pop on its emphasis.

**Checkpoint:** `render_preview {node: "s1", count: 10}` (spread over the slide's own length) —
words arriving one after another, the emphasis landing last and strongest, a real hold, a quick
exit. Then the same for `s2`.

## 5. Export

- A GIF of the page plays both slides one after another: `export {project_id, format: "gif",
  fps: 30, width: 960}`; the details list the slides. One slide: add `node: "s1"`.
- MP4 or WebM: from the editor (File › Export), not the MCP — tell the user.

**Common mistakes:** words moving together (no stagger); uneven gaps (measure, don't guess); the
emphasised word squeezing its neighbour (grow it from its edge); too little time to read;
entrance and exit at the same speed; more than one emphasis per line.
