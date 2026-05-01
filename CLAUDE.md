# Claude Instructions — Vibe Blend

## Project Context

This is a 3D design vibe-coding workspace for **Vibe Blend**. Claude controls Blender via the **blender-mcp** MCP server to generate, iterate, and export models optimized for 3D printing.

All project goals, dimensions, and constraints live in [PROJECT_DESIGN.md](PROJECT_DESIGN.md).
Reference images are in the [reference_images/](reference_images/) folder.

---

## Core Workflow

1. **Read PROJECT_DESIGN.md first** before generating any geometry.
2. **Check reference images** in `reference_images/` for visual direction.
3. **Generate in Blender** using `mcp__blender__execute_blender_code`.
4. **Screenshot after every significant change** using `mcp__blender__get_viewport_screenshot` to verify visually.
5. **Iterate** based on feedback — update `PROJECT_DESIGN.md` iteration log after each approved version.

---

## 3D Printing Rules (Always Enforce)

These are non-negotiable constraints on every generated mesh:

- **Manifold mesh only** — no holes, non-manifold edges, or internal faces
- **Minimum wall thickness**: 1.2mm (for FDM); 0.5mm (for resin)
- **Minimum feature size**: 0.8mm
- **Max unsupported overhang**: 45° from vertical
- **No intersecting geometry** — all objects must be boolean-unioned before export
- **Flat bottom face** — orient model so the largest flat face is on the Z=0 plane
- **Units**: Millimeters (Blender scene scale = 0.001)
- **Export formats**: STL or 3MF

---

## Blender MCP Tools Available

| Tool | Use |
|------|-----|
| `mcp__blender__execute_blender_code` | Run any Python/bpy code in Blender |
| `mcp__blender__get_scene_info` | Inspect current scene objects |
| `mcp__blender__get_object_info` | Get mesh details for a specific object |
| `mcp__blender__get_viewport_screenshot` | Screenshot the viewport |
| `mcp__blender__search_polyhaven_assets` | Find free HDRIs/textures |
| `mcp__blender__download_polyhaven_asset` | Download and apply PolyHaven assets |
| `mcp__blender__search_sketchfab_models` | Find reference models on Sketchfab |

---

## Code Style in Blender Scripts

- Always start scripts with `import bpy`
- Clear the scene before building: `bpy.ops.object.select_all(action='SELECT'); bpy.ops.object.delete()`
- Set units to millimeters at script start:
  ```python
  bpy.context.scene.unit_settings.system = 'METRIC'
  bpy.context.scene.unit_settings.scale_length = 0.001
  bpy.context.scene.unit_settings.length_unit = 'MILLIMETERS'
  ```
- Name every object meaningfully (not "Cube.001")
- Apply all transforms before export: `bpy.ops.object.transform_apply(location=True, rotation=True, scale=True)`
- Use bmesh for complex geometry operations

---

## Session Behavior

- **Do not hallucinate dimensions** — always derive from PROJECT_DESIGN.md
- **After each Blender operation**, take a viewport screenshot and describe what was created
- **Flag print issues immediately** — if generated geometry violates print rules, say so and fix before continuing
- **Save a .blend file** at the end of each session in the project root

---

## File Structure

```
Traction/
├── CLAUDE.md              ← you are here
├── README.md              ← project overview & setup
├── SKILLS.md              ← 3D printing + Blender technique reference
├── PROJECT_DESIGN.md      ← active design brief (edit this per project)
├── reference_images/      ← drop reference photos/sketches here
├── exports/               ← STL/3MF files ready for slicing (auto-created)
└── *.blend                ← saved Blender scene files
```
