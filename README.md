## 🌿 GrassFlow — GPU Grass for Unity URP

![Unity](https://img.shields.io/badge/Unity-6000.0+-%232b2b2b?style=flat-square&logo=unity)
![Instances](https://img.shields.io/badge/Instance_Count-1M%2B-hsl%28145%2C50%25%2C40%25%29?style=flat-square)
![Draw Calls](https://img.shields.io/badge/Draw_Calls-3_Indirect-hsl%28200%2C60%25%2C45%25%29?style=flat-square)
![Frame Time](https://img.shields.io/badge/Frame_Time_Delta-0.1ms-hsl%2835%2C85%25%2C50%25%29?style=flat-square)
![License](https://img.shields.io/badge/License-MIT-hsl%28350%2C60%25%2C40%25%29?style=flat-square)

High-performance, GPU-driven vegetation for **Unity 6 URP**. Generates, culls, deforms, and renders over **1,000,000 blades** with **zero per-instance CPU work**. Built for my portfolio, but packaged so other teams can drop it into production or use it as a reference for GPU instancing, HiZ occlusion, and compute-driven LOD.

Demo clips (turn sound on):
- https://github.com/user-attachments/assets/06014a60-49a3-4cea-8c40-7454d34aba38
- https://github.com/user-attachments/assets/055c5fd2-947a-4bbb-a6f6-5b987e34fe91

> Inspired by the *Ghost of Tsushima* grass pipeline: procedural blades, compute culling, HiZ, Bezier wind, and painter-friendly authoring tools.

---

## Highlights
- **GPU-only pipeline:** Generation, multi-stage culling, LOD routing, and rendering all live on the GPU via compute shaders and `Graphics.DrawMeshInstancedIndirect`.
- **HiZ occlusion:** Depth-pyramid occlusion saves vertex work behind hills, rocks, and props (reverse-Z aware).
- **Four-stage culling:** Density mask → frustum → distance thinning with width compensation → HiZ occlusion.
- **Physically inspired wind:** Cubic Bezier bending, macro gusts, micro flutter, distance-based wind disable.
- **Editor-first tools:** Scene-view painter with overlays, mini-map, undo/redo, and auto density-mask management.
- **Production safeties:** Debug HUD, conservative defaults, and inspector knobs that map cleanly to the compute kernels.

---

## Visuals and Screenshots
- Short clips above. Drop your own high-res captures into `docs/media/` and link them here (PNG/JPEG, 16:9 or square works well).
- Suggested angles: wide vista with wind, close-up translucency/backlight, painter overlay in Scene view, and HiZ debug overlay.

---

## Feature Map
- **GPU architecture:** [Documentation/Feature_GPU_Architecture.md](Grass/Documentation/Feature_GPU_Architecture.md) — source buffer generation, append buffers per LOD, indirect args, bounds, and URP setup.
- **HiZ occlusion:** [Documentation/Feature_HiZ_Occlusion.md](Grass/Documentation/Feature_HiZ_Occlusion.md) — depth pyramid build (blit + reduce), mip selection from projected size, bias tuning, and reverse-Z handling.
- **LOD and density:** [Documentation/Feature_LOD_and_Density.md](Grass/Documentation/Feature_LOD_and_Density.md) — density masks, frustum tests, distance thinning, width compensation, and default LOD distances/segments.
- **Wind and shading:** [Documentation/Feature_Wind_and_Shading.md](Grass/Documentation/Feature_Wind_and_Shading.md) — Bezier deformation, macro/micro wind, gradient coloring, fake normals, AO, translucency, shadows.
- **Painting tools:** [Documentation/Feature_Painting_and_Tools.md](Grass/Documentation/Feature_Painting_and_Tools.md) — in-editor painter, overlays, mini-map, brush controls, and best practices.
- **Debug and fixes:** [Documentation/Feature_Debugging_and_Troubleshooting.md](Grass/Documentation/Feature_Debugging_and_Troubleshooting.md) — quick checks for depth textures, HiZ state, and typical URP pitfalls.

---

## Architecture at a Glance
```
GrassRenderer
├─ Compute: CSMain (generate) → CSCull (density, frustum, distance thinning, HiZ, LOD routing)
├─ HiZ: BlitDepth + ReduceDepth (pyramid for occlusion)
├─ Buffers: sourceGrassBuffer → LOD0/1/2 append buffers → indirect args
└─ Render: GrassShader.shader (Bezier wind, gradient lighting, translucency, fake normals)
```

Key files:
- Runtime: [Grass/Assets/Scripts/GrassRenderer.cs](Grass/Assets/Scripts/GrassRenderer.cs)
- Compute: [Grass/Assets/Shaders/GrassCompute.compute](Grass/Assets/Shaders/GrassCompute.compute), [Grass/Assets/Shaders/HiZGenerator.compute](Grass/Assets/Shaders/HiZGenerator.compute)
- Shader/Material: [Grass/Assets/Shaders/GrassShader.shader](Grass/Assets/Shaders/GrassShader.shader), [Grass/Assets/Materials/GrassMat.mat](Grass/Assets/Materials/GrassMat.mat)
- Editor: [Grass/Assets/Scripts/Editor/GrassPainterEditor.cs](Grass/Assets/Scripts/Editor/GrassPainterEditor.cs), [Grass/Assets/Shaders/GrassDensityOverlay.shader](Grass/Assets/Shaders/GrassDensityOverlay.shader)
- Docs hub: [Grass/Documentation/Features_Overview.md](Grass/Documentation/Features_Overview.md)

---

## Performance Benchmarks

| Blade Count | Draw Calls | Frame Time | FPS | Notes |
|-------------|-----------|------------|-----|-------|
| 100,000 | 3 | 2.1 ms | 144+ | Ultra smooth |
| 500,000 | 3 | 3.8 ms | 90+ | High performance |
| **1,000,000** | **3–5** | **6.2 ms** | **60+** | Default target |
| 2,000,000 | 5 | 11.5 ms | 45 | Dense forests |
| 5,000,000 | 5 | 28 ms | 30 | Stress test |

Tested on Apple M2 (8-core GPU). HiZ typically saves more time than it costs (~0.5 ms at 1080p). Scales with GPU compute throughput.

---

## Getting Started (5 minutes)
1) Use **Unity 6000.0+ URP** with **Depth Texture** enabled in your active Renderer asset.
2) Drop the **Grass** folder into your project or open the included sample scene in [Grass/Assets/Scenes](Grass/Assets/Scenes).
3) Add a GameObject and attach `GrassRenderer`.
4) Assign references: Main Camera, Terrain, GrassCompute.compute, HiZGenerator.compute, GrassShader.shader, GrassMat.mat.
5) Set grassCount and terrainSize to match your scene bounds; leave defaults for a first run.
6) Press Play and verify the debug HUD shows three indirect draws and HiZ ready.

---

## Inspector Cheat Sheet
- **Counts and bounds:** grassCount, terrainSize, bounds padding.
- **LOD bands:** lod0Distance, lod1Distance, maxDrawDistance; segment counts per LOD.
- **Culling:** enableDensityScaling, minDensity, densityFalloffStart, widthCompensation.
- **Occlusion:** useOcclusionCulling, occlusionBias, mainCamera (depth source).
- **Wind:** windDisableDistance, wind map, gust/flutter strengths, camera facing.
- **Debug:** showDebugUI to surface per-LOD counts, HiZ status, and density sampling.

---

## Painting Workflow (Artists)
1) Click **Start Painting** on `GrassRenderer`.
2) If missing, an R8 density map is auto-created (read/write, uncompressed).
3) Scene view controls: Left Click to paint, Shift + Left Click to erase; adjust size/opacity/hardness.
4) Overlays: heatmap tint and mini-map for context; keep renderInEditMode on for live feedback.
5) Save Mask to PNG to persist; version your masks when trying new biomes.

Tips: paint broad strokes at low opacity, keep densityThreshold modest (~0.1), widen far-field blades with widthCompensation when thinning.

---

## Wind and Shading Notes
- **Bezier bend** keeps blade length stable while avoiding shear.
- **Macro vs micro motion:** scrolling noise for waves; per-blade phase for flutter.
- **Lighting:** tri-tone vertical gradient, fake rounded normals, vertical AO, and backlit translucency for sunset shots.
- **Performance lever:** windDisableDistance fades motion out in the distance; disable LOD1 shadows for big vistas.

---

## Troubleshooting Quick Checks
- HiZ off? Ensure Depth Texture is enabled and hizComputeShader is assigned; filter mode must be Point.
- Flicker on slopes? Raise occlusionBias slightly (0.1–0.2).
- Sparse far field? Lower minDensity or reduce densityFalloffStart; compensate with widthCompensation.
- Missing grass in edit mode? Confirm density map is readable and renderInEditMode is enabled.

More detail in [Grass/Documentation/Feature_Debugging_and_Troubleshooting.md](Grass/Documentation/Feature_Debugging_and_Troubleshooting.md).

---

## Repository Map
- Core scripts and shaders: [Grass/Assets/Scripts](Grass/Assets/Scripts), [Grass/Assets/Shaders](Grass/Assets/Shaders)
- Materials and scenes: [Grass/Assets/Materials](Grass/Assets/Materials), [Grass/Assets/Scenes](Grass/Assets/Scenes)
- Documentation: [Grass/Documentation](Grass/Documentation)
- Project settings: [Grass/ProjectSettings](Grass/ProjectSettings)

---

## Why This Exists
This project is a portfolio-ready showcase of modern GPU foliage techniques, packaged so other developers can:
- Study a clean, minimal C# → HLSL → URP pipeline for indirect instancing.
- Reuse the painter and density workflow for their own foliage or debris systems.
- Benchmark HiZ and distance thinning patterns on their hardware.

---

## License
MIT — feel free to use in commercial and non-commercial projects. Attribution appreciated.

---

Built by Mithzz — 2026
