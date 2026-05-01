# Skills Reference — viblend

Quick reference for Claude when working on viblend. All techniques are oriented toward producing printable geometry.

---

## Blender MCP Setup

### Prerequisites
1. **Blender** installed (3.x or 4.x)
2. **blender-mcp addon** installed in Blender:
   - Download `addon.py` from [ahujasid/blender-mcp](https://github.com/ahujasid/blender-mcp)
   - In Blender: Edit → Preferences → Add-ons → Install → select `addon.py` → Enable it
   - A panel appears in the N-sidebar under "Blender MCP" — click **Start MCP Server**
3. **Claude Code** with MCP configured (see README.md for `claude_desktop_config.json` snippet)

---

## Python (bpy) Patterns for 3D Printing

### Scene Setup
```python
import bpy, bmesh
# Clear scene
bpy.ops.object.select_all(action='SELECT')
bpy.ops.object.delete()
# Set mm units
scene = bpy.context.scene
scene.unit_settings.system = 'METRIC'
scene.unit_settings.scale_length = 0.001
scene.unit_settings.length_unit = 'MILLIMETERS'
```

### Create a Basic Printable Box (dimensions in mm)
```python
bpy.ops.mesh.primitive_cube_add(size=1)
obj = bpy.context.active_object
obj.name = "MyBox"
obj.scale = (width_mm/1000, depth_mm/1000, height_mm/1000)
bpy.ops.object.transform_apply(scale=True)
```

### Boolean Union (merge two objects)
```python
mod = base_obj.modifiers.new(name="Union", type='BOOLEAN')
mod.operation = 'UNION'
mod.object = cutter_obj
bpy.context.view_layer.objects.active = base_obj
bpy.ops.object.modifier_apply(modifier="Union")
```

### Check Manifold (watertight)
```python
bm = bmesh.new()
bm.from_mesh(obj.data)
non_manifold = [e for e in bm.edges if not e.is_manifold]
print(f"Non-manifold edges: {len(non_manifold)}")  # must be 0
bm.free()
```

### Save to main.blend
```python
bpy.ops.wm.save_as_mainfile(filepath=bpy.path.abspath("//main.blend"))
```

### Pre-Export Print Checklist (run before every export)
```python
import bpy, bmesh, os

issues = []

for obj in bpy.context.scene.objects:
    if obj.type != 'MESH':
        continue

    # 1. Check manifold
    bm = bmesh.new()
    bm.from_mesh(obj.data)
    bm.verts.ensure_lookup_table()
    non_manifold_edges = [e for e in bm.edges if not e.is_manifold]
    if non_manifold_edges:
        issues.append(f"{obj.name}: {len(non_manifold_edges)} non-manifold edge(s)")

    # 2. Check for loose geometry
    loose_verts = [v for v in bm.verts if not v.link_edges]
    if loose_verts:
        issues.append(f"{obj.name}: {len(loose_verts)} loose vertex(es)")

    bm.free()

    # 3. Check transforms are applied
    if any(abs(s - 1.0) > 0.001 for s in obj.scale):
        issues.append(f"{obj.name}: unapplied scale {list(obj.scale)} — run Apply > Scale")

    # 4. Check flat bottom (Z min should be at or near 0)
    world_z = [obj.matrix_world @ v.co for v in obj.data.vertices]
    z_min = min(v.z for v in world_z)
    if abs(z_min) > 0.01:
        issues.append(f"{obj.name}: bottom face not at Z=0 (min Z = {z_min:.3f}mm)")

if issues:
    print("PRINT ISSUES FOUND:")
    for i in issues:
        print(f"  ✗ {i}")
else:
    print("All checks passed — safe to export.")
```

### Export STL (Blender 4.x)
```python
import bpy, os
project_name = "my_model"  # change per project
version = "v1"
exports_dir = bpy.path.abspath("//exports/")
os.makedirs(exports_dir, exist_ok=True)

bpy.ops.object.select_all(action='SELECT')
bpy.ops.wm.stl_export(
    filepath=os.path.join(exports_dir, f"{project_name}_{version}.stl"),
    apply_modifiers=True,
    export_selected_objects=True,
    global_scale=1.0,
    use_scene_unit=True,
    ascii_format=False
)
```

### Export 3MF (Blender 4.x)
> Note: Blender 4.x has no built-in `bpy.ops` for 3MF. Use the STL workflow above,
> then convert in PrusaSlicer (File → Save As → .3mf) or use the `blender-3mf` addon.

---

## 3D Printing Design Principles

### Wall Thickness
| Material | Minimum | Recommended |
|----------|---------|-------------|
| PLA (FDM) | 1.2mm | 2.4mm+ |
| PETG (FDM) | 1.2mm | 2.0mm+ |
| Resin | 0.5mm | 1.0mm+ |

### Overhangs
- **Under 45°** from vertical: prints without support on most FDM printers
- **45°–90°**: requires support material
- Design bridging spans under 50mm to avoid sagging

### Holes and Tolerances
- Add **0.2mm tolerance** to holes that need to fit a bolt or shaft
- Vertical holes print undersized — compensate by designing 0.2mm larger
- Horizontal holes are more accurate

### Text and Embossing
- Minimum embossed letter height: 5mm
- Minimum relief depth: 0.5mm
- Use bold/sans-serif fonts for readability at small sizes

### Supports and Orientation
- Orient the model to minimize overhangs
- Put the most detailed surface face-up (away from bed)
- Use a brim (5–10mm) for thin/tall models to prevent warping

---

## Iterative Vibe Coding Loop

```
1. Describe shape in PROJECT_DESIGN.md
2. Ask Claude to generate in Blender (working in main.blend)
3. Screenshot → review viewport
4. "Make it taller", "round that edge", "add a hole here"
5. Check manifold, wall thickness, overhangs
6. Save to main.blend
7. Export STL → slice in Cura/PrusaSlicer → preview
8. Print → iterate
```

---

## Useful Blender Shortcuts (while reviewing)

| Shortcut | Action |
|----------|--------|
| Numpad 1/3/7 | Front/Side/Top view |
| Numpad 5 | Toggle orthographic |
| Alt+Z | Toggle X-ray mode |
| N | Toggle N-panel (MCP server here) |
| Tab | Edit mode toggle |
| A | Select all |

---

## Reference Links

- [blender-mcp GitHub](https://github.com/ahujasid/blender-mcp)
- [Blender Python API Docs](https://docs.blender.org/api/current/)
- [Printables](https://www.printables.com) — community models for inspiration
- [PolyHaven](https://polyhaven.com) — free HDRIs and textures (available via MCP)
- [Meshmixer](https://meshmixer.com) — free mesh repair tool
