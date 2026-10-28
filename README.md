# Quar Animator skills for Claude

Claude skills for [Quar Animator](https://animator.quar.pro), the web-native 2D animation editor.

## quar-design

Teaches Claude to design and animate in Quar Animator through its MCP server, the way a motion
designer works when they can see their work: plan the composition, build it, look at it (the
server renders every step), fix it, animate it with real timing and easing, check the motion on a
contact sheet, and export Lottie, GIF, PNG, PNG sequences or SVG.

It brings a design rubric (composition, colour, type, depth, a checklist for every picture), a
motion guide (timing, easing, overlap, staggering, loops), the pitfalls met in real projects, and
step-by-step recipes: a bouncing logo, kinetic typography across slides, a UI micro-interaction
for Lottie, and a rigged walk cycle.

### What you need

- A Quar Animator account (free during the beta): sign up at https://animator.quar.pro.
- Claude with the Quar MCP connected (below). When you connect it, you choose what Claude may
  reach: all your projects or one, reading only or editing, and exporting.

### Claude Code

```text
/plugin marketplace add FunwayHQ/quar-animator-skills
/plugin install quar-design@quar-animator-skills
```

The plugin brings both the skill and the Quar MCP connection. Then run `/mcp`, choose
**quar-animator** and **Authenticate**: your browser opens Quar Animator's sign-in and consent
page.

### claude.ai and Claude Desktop

1. **Connect the MCP:** Settings › Connectors › Add custom connector, with the URL
   `https://api.animator.quar.pro/mcp`; sign in to Quar Animator and allow access.
2. **Add the skill:** download [`quar-design.zip`](https://animator.quar.pro/skills/quar-design.zip)
   (also attached to this repo's releases), then Customize › Skills › + › Create skill › Upload a
   skill. Skills need code execution turned on (Settings › Capabilities).

### Using it

Ask for what you want; the skill takes over when it's about Quar Animator:

- "Make a 3-second bouncing logo for our brand in a new Quar project, and give me a GIF."
- "In my project *Launch*, add a title card that types on, then fades out."
- "Animate 'Make it move. Make it yours.' as kinetic type, one sentence per slide."
- "Design a like-button animation for our app and export it as Lottie."
- "Rig the character in *Walker* and make it walk in place, a 24-frame loop."

Claude tells you its plan, shows you pictures as it goes, and every step it saves is a version
in your project's history: undo any of them in the editor (the clock icon) or by asking Claude.
Keep the project open in the editor to watch it change live.

### Troubleshooting

| Problem | What to do |
| --- | --- |
| Claude says the Quar tools aren't there | Connect the MCP (above); in Claude Code, start a new session after installing |
| "Not allowed" when editing or exporting | Reconnect and allow editing / exporting for that project |
| An edit fails with a version conflict | You (or someone) saved meanwhile: ask Claude to read the project again and redo it |
| "past the end of its artboard's timeline" | Each artboard has its own length: ask Claude to lengthen it |
| The font looks different in Claude's pictures | The server draws text in Inter; the editor shows your font. Ask Claude to check your canvas |
| A draw-on stroke looks complete in Claude's picture | The server doesn't draw stroke progress; it plays in the editor |
| Storage full | Each account has 92 MB during the beta: delete cloud projects you don't need |

## Licence

MIT — see [LICENSE](LICENSE). Quar Animator itself is open source too:
[FunwayHQ/quar-animator](https://github.com/FunwayHQ/quar-animator).
