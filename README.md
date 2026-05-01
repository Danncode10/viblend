# Vibe Blend

A Claude + Blender workflow for designing 3D-printable models through natural language — describe what you want, Claude builds it in Blender, you iterate in plain English.

---

## What This Is

This template connects **Claude Code** to **Blender** via the [blender-mcp](https://github.com/ahujasid/blender-mcp) MCP server. You describe a 3D object in natural language, and Claude writes Blender Python scripts in real time to model, iterate, and export print-ready geometry.

Watch the setup walkthrough: [youtube.com/watch?v=pCwOj_tGav0](https://youtu.be/pCwOj_tGav0?si=xVQu4V2RMzbBbg7_)

---

## Project Files

| File | Purpose |
|------|---------|
| [PROJECT_DESIGN.md](PROJECT_DESIGN.md) | **Start here** — fill in your design brief, dimensions, and constraints |
| [CLAUDE.md](CLAUDE.md) | Instructions Claude follows automatically in this project |
| [SKILLS.md](SKILLS.md) | Blender + 3D printing technique reference |
| [reference_images/](reference_images/) | Drop your sketches, photos, or inspiration images here |
| `exports/` | STL and 3MF files go here (created automatically on first export) |
| `*.blend` | Blender scene files saved during sessions |

---

## Prerequisites

### 1. Blender
Download from [blender.org](https://www.blender.org/download/). Version 3.6 LTS or 4.x recommended.

### 2. blender-mcp Addon
1. Go to [github.com/ahujasid/blender-mcp](https://github.com/ahujasid/blender-mcp)
2. Download **`addon.py`** (the Blender addon file)
3. In Blender: **Edit → Preferences → Add-ons → Install** → select `addon.py` → enable it
4. Open the N-panel (press **N** in viewport) → find the **Blender MCP** tab → click **Start MCP Server**

### 3. Claude Code
Install from [claude.ai/code](https://claude.ai/code) or via npm:
```bash
npm install -g @anthropic-ai/claude-code
```

### 4. Configure the MCP Server
Add this to your Claude Code MCP config (`~/.config/claude/claude_desktop_config.json` or equivalent):

```json
{
  "mcpServers": {
    "blender": {
      "command": "uvx",
      "args": ["blender-mcp"]
    }
  }
}
```

> If `uvx` is not installed: `pip install uv` or `brew install uv`

Then restart Claude Code. You should see the Blender tools available.

---

## Quick Start

1. **Open Blender** and start the MCP server (N-panel → Blender MCP → Start MCP Server)
2. **Fill in [PROJECT_DESIGN.md](PROJECT_DESIGN.md)** with what you want to build
3. **Add reference images** to `reference_images/` if you have any
4. **Open Claude Code** in this directory and start prompting:

```
Read PROJECT_DESIGN.md and the reference images, then build the model in Blender.
```

5. Claude will generate the geometry, take viewport screenshots for you to review, and iterate on your feedback
6. When satisfied: ask Claude to export as STL, then slice in Cura or PrusaSlicer

---

## Example Prompts

```
Build the model described in PROJECT_DESIGN.md. Start with a basic shape and show me a screenshot.
```

```
Round all the top edges with a 2mm fillet.
```

```
Add a mounting hole in the center, 4mm diameter, going all the way through.
```

```
Make it 20% taller and check that there are no overhangs greater than 45 degrees.
```

```
Export the final model as an STL to the exports folder.
```

---

## Workflow

```
Fill PROJECT_DESIGN.md
        ↓
Add reference images
        ↓
Claude generates in Blender  ←──────────────┐
        ↓                                   │
Review viewport screenshot                  │
        ↓                            give feedback
Check print constraints                     │
 (walls, overhangs, manifold)               │
        ↓                                   │
Looks good? ──── No ────────────────────────┘
        │
       Yes
        ↓
Export STL / 3MF
        ↓
Slice (Cura / PrusaSlicer)
        ↓
Print
```

---

## Tips

- **Be specific about dimensions** in PROJECT_DESIGN.md — "small box" is less useful than "50mm × 30mm × 20mm box"
- **Iterate in small steps** — ask for one change at a time, review, then continue
- **Screenshot often** — ask Claude to take a viewport screenshot after any major change
- **Check the exports/ folder** before sending to your slicer — Claude should have applied all transforms and ensured a manifold mesh

---

## Troubleshooting

| Problem | Fix |
|---------|-----|
| Claude can't see Blender tools | Make sure the MCP server is running in Blender's N-panel and the config JSON is correct |
| Model has holes / non-manifold | Ask Claude: "Check manifold and fix any issues" |
| STL scale is wrong in slicer | Confirm Blender scene units are set to Millimeters (Claude should do this automatically) |
| Blender crashes | Save .blend first, restart Blender, restart MCP server |
