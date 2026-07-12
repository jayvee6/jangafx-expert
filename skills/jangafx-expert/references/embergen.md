# EmberGen — Expert Reference (JangaFX Real-Time Gaseous Fluid Simulation)

> Real-time GPU voxel gas solver — fire, smoke, explosions. Compiled from https://docs.jangafx.com/embergen/ plus JangaFX marketing/roadmap and integration write-ups. Facts not in official docs are labeled `[non-docs]`; where the docs' auto-summary paraphrased, it's marked "paraphrased." Verify exact slider wording in-app under Advanced mode.

---

## 1. When to reach for EmberGen

EmberGen is a **GPU-resident, real-time voxel gas solver**. The whole simulation lives in VRAM and re-simulates interactively as you scrub/edit — author fire/smoke/explosions in seconds instead of waiting on CPU caches.

- **Game-ready flipbooks / sprite sheets fast** — headline workflow: "generate game-ready, fully assembled flipbooks in seconds… into Unreal Engine or Unity." Emits the render passes (incl. six-way lighting) that Niagara / VFX Graph consume.
- **VDB volumes for offline DCC render** (Maya, Blender, Houdini, C4D) without a slow native gas solve. Export `.vdb` sequences and light/render in the target package.
- **Fast look-dev / previz** of fire & explosions where interactivity beats ultimate fidelity.

**When NOT to use it:**
- **Liquids** → that's **LiquiGen**. EmberGen is gaseous only.
- **macOS** — not supported (1.x). *(EmberGen 2.0 adds macOS `[non-docs, roadmap]`.)*
- **Ultra-high-res hero sims exceeding VRAM** — sim must fit one GPU (no multi-GPU pooling).

---

## 2. Platform, licensing & hardware

| Item | Detail |
|---|---|
| OS | **Windows 10 or Linux**; macOS not supported (1.x) |
| Licensing | Key from pricing page → activate in **License Manager** (Help menu). Offline activation possible. Tiers/floating in the separate Floating License Guide (see `suite-overview.md` §5) |
| Min GPU `[non-docs]` | GTX 1060+ ; AMD supported (GPU-agnostic) |
| Multi-GPU | **Not supported** — one GPU runs the sim; VRAM can't be pooled `[non-docs]` |
| VRAM reality | Entire sim + render must fit one card. 24 GB fits heavy scenes `[non-docs]` |
| CPU role | Minor — mainly slightly faster export disk-writes `[non-docs]` |

**EmberGen 2.0 `[non-docs, roadmap]`:** sparse memory → ~10–15× more voxels than 1.x (scene-dependent), path-tracing renderer, USD, macOS, and a roadmapped **scriptable API**. Directional, not doc-confirmed.

---

## 3. Core concepts & UI

**The sim is voxels** — each voxel stores **fuel, smoke, temperature, flames, velocity**. Trade voxel count/size/upscaling against performance.

**Everything is a node graph.** Categories: **Emitters, Shapes, Forces, Simulation, Colliders, Volume, Shading, Lights, Camera, Render.**

### Four panels
| Panel | Role | Key controls |
|---|---|---|
| **Viewport** | Visualize | Quality **Ctrl+1/2/3**; **Unlit = U** (faster); on-screen Voxels / Sim Time / Sim Speed stats |
| **Node Graph** | Build | search, tree, **seed randomize**, zoom |
| **Timeline** | Time/keyframes | **Space** play, **Z** step frame, **L** loop, **`** curve editor |
| **Properties** | Node params | per-node tabs; **Override State cycler**: No Override → Timeline (keyframe) → Pin (external modulation) → Randomize |

Menus: File **Increment and Save Alt+Shift+S** (versioning); Simulation **Hard Reset Ctrl+R**, Reset R, Step Z, Pause Space. **Simple vs Advanced** param toggle — Advanced exposes "all backend parameters" (fine dissipation/vorticity) `[non-docs, paraphrased]`.

---

## 4. Core simulation workflow

1. **Emitter** (Volume or Particles) injects fuel/smoke/temperature into voxels.
2. Emitter → Simulation node's **Emitters** input pin.
3. Add **Forces** (noise/line/point/toroidal) + **Colliders** to shape motion.
4. **Volume** node reads the sim; **Volume Processing** sharpens / post-modulates.
5. **Shading / Lights / Camera** define the look.
6. **Render** node exports images/flipbooks; **Export: VDB / Particles** exports data.

**Timeline sync rule (critical gotcha, FAQ):** for imported animation/camera to line up, match: **Timestep** in Simulation (default **60 Hz**), **FPS** in Import→Playback, **Backplate frame rate** in Camera. When aligned, set **Frame Stride = 1** in Render.

---

## 5. Parameters that actually matter

### 5.1 Simulation node (names reliable; descriptions paraphrased)
| Parameter | What it does |
|---|---|
| **Simulation Mode** | Fuel/Smoke vs **Colored Smoke** (unlocks Smoke Color) |
| **Voxel Resolution / Grid Size** | Voxel dims of the container — primary detail/cost driver |
| **Upscaling** | Runs some channels (smoke) higher-res than others (velocity) — detail without full-res everything (costs VRAM) |
| **Timestep** | Sim rate; default **60 Hz** (match import FPS / backplate) |
| **Substeps** | Iterations per frame — stability/accuracy at higher cost |
| **Dissipation** | Rate density/temperature fades over time |
| **Buoyancy** | Upward force on hot material — **driven by the temperature channel** |
| **Vorticity** | Reintroduces swirl detail (`[non-docs]` vorticity ramp added 0.5.6) |
| **Turbulence** | Chaotic flow detail |
| **Combustion** | Fuel→flame conversion; "mimics real-world fireballs consuming fuel/oxygen" |
| **Gas Pressure Release** | Adds pressure / expands the sim as fuel is consumed (`[non-docs]` 0.5.6) |
| **Wind** | Directional force |

> **Key interaction — temperature drives everything hot.** *Temperature* is the target temperature in Kelvin and the amount of heat added. **Without temperature, fuel won't ignite and no flames appear**, and temperature *also* drives buoyancy (rise). Ignition and rise are coupled through heat — you can't tune rise independently without compensating.

### 5.2 Emitter: Volume
- **Emission tab:** Fuel/Smoke/Temperature modes: **No Emission, Add, Add Clamped, Replace**. "Temperature: target temperature in Kelvin." Emission gradient for natural diffuse emission. Smoke Color (Colored Smoke only).
- **Pressure tab:** "Additional pressure rate: **positive → explode, negative → implode**." + pressure-random intensity/scale/seed/speed.
- **Forces tab:** "Velocity transfer: percentage of velocity transferred."

### 5.3 Emitter: Particles (sparks, embers, debris)
Continuous toggle, Seed, **Emission Rate**, Burst + burst size, Spawn on surface, Jitter, styles **Regular/Entangled/Clumped**. Lifetime range. Initial velocity None/Range/Cone, speed scale, velocity transfer. Volume influence 0–100%, gravity multiplier, forces tightness, velocity damping. Collisions: bounciness, friction, repulsion, **Bound behavior: Ignore/Kill/Stop/Bounce**. Render **Camera/Velocity Aligned**, blend **Translucent/Additive/Translucent Additive**. **Injection:** particles can inject **Smoke, Fuel, Flames, Temperature, Velocity** back into the grid (feedback) with age-curve modulation.

### 5.4 Forces
- **Force: Noise** — 3D-noise force, **curl noise** option (go-to for organic turbulence).
- **Force: Line** — push / twist / repel along a line.
- **Force: Point** — attraction/repulsion.
- **Force: Toroidal** — donut force w/ rotation + radius (smoke-ring / mushroom shapes).

### 5.5 Volume / Volume Processing / shading
**Density Scale** (opacity), **Shadow/Scattering** (light interaction), **Smoke Color** (Colored Smoke), **Emission/Blackbody** (temperature-driven self-illumination). **Volume Processing** = sharpening + Post Modulation (add noise, visualize vorticity) — **these bake into the VDB** (see §6.3).

### 5.6 Collider
"Makes smoke, flames, and particles collide with connected shapes, optionally injecting velocities."

---

## 6. Export pipeline

"Instantly simulate, render, and export flipbooks, image sequences, and VDB volumes."

### 6.1 Render node → images / flipbooks
- **Export Mode:** Flipbook or Sequence.
- **Formats:** **PNG, TGA, EXR** (EXR = HDR/linear route for comp + data passes).
- **Flipbook:** *Flipbook Size*, *Columns/Rows* — "Columns × Rows = max frames" (your sub-UV sheet for Niagara/VFX Graph).
- **Sequence:** presets 720p/1080p/4K + custom.
- **Frame range:** First Frame, Number of Frames, Frame Stride, Numbering offset, Absolute Frames.

**Render passes (the integration payoff):**
`Render Viewport, Render All, Smoke, Emissive and Scattering, Emissive, Scattering, Direct Light, Ambient Light, Alpha, Depth, Motion Vector, Six Point 1 (TLR), Six Point 2 (BBF), Six Point Normal Map, Gradient Based Normal Map, Albedo, Smoke Mask, Flames Mask, Temperature` + a parallel `... Shapes` set.
- **Motion Vector + Depth** → temporal reprojection / motion blur / depth comp in engine or Nuke.
- **Six Point 1 (TLR) / Six Point 2 (BBF) / Six Point Normal Map** → **6-way lighting** flipbooks for real-time dynamic relighting.

### 6.2 Into Unreal Niagara / Unity VFX Graph `[non-docs]`
- **Standard flipbook:** export Columns×Rows sheet (+ optional motion-vector sheet), drive a sub-UV material, spawn via Niagara / VFX Graph.
- **6-way lighting:** packs lightmaps for six directions into two textures + alpha + optional emissive mask → dynamic relighting / internal shadowing / back-light rims on a flat sheet. **Caveat (Unity):** "not officially supported because we don't have control over the light intensity in these maps" — expect to hand-tune intensity; Unity's VFXToolbox/Image Sequencer can repack. Flat sheet — best for background/ornamental FX and explosions, not true 3D volume. Source: https://unity.com/blog/engine-platform/realistic-smoke-with-6-way-lighting-in-vfx-graph
- Community: Epic "EmberGen vs Niagara Fluids" series; RedefineFX EmberGen Bootcamp.

See `export-pipeline.md` for the full destination matrix (incl. Unreal **Sparse Volume Textures** for VDB, WebGPU, ComfyUI).

### 6.3 Export: VDB → OpenVDB for DCC
- Connect **Export: VDB** to the **Volume node's VDB output pin** so Volume-Processing edits bake in. **Shading/lighting are NOT stored** — only volume channels.
- **Transform input pin:** connect an Import node's Transform output → applies the **inverted** import transform so the VDB re-aligns in the source app.
- **Directory/Filename** dynamic variables: `$(projectdir)` = path of the `.ember` file; a run of `#` = zero-padded frame (`####` → 0000…).
- **Frame controls:** First Frame, Num Frames, Frame Stride; also draggable on the Timeline.
- **Channels:** Density, Temperature, Fuel, Flames, Velocity — **only Density + Flames on by default**. Rename grids per target app; set grid-class tags; **single velocity grid** vs three scalar components.
- **Coordinate System:** **Blender → Z Up Right Handed** (default); **Maya → Y Up Right Handed**; + **Length Unit**.
- **Export Now** / **Export All** (right-click, multiple export nodes); **Open Folder** button.

### 6.4 Export: Particles → Alembic (.abc)
Single file or sequence; frame stride; numbering offset. Optional channels: **colors, densities (transparency), lifetimes, sizes**. Selectable coordinate system. **No mesh export / no direct engine integration** for particles — point/Alembic data only.

### 6.5 Import
**FBX, OBJ, ABC** mesh + animation; camera import → emission sources, colliders, camera-match. Remember the §4 three-way FPS sync.

---

## 7. Automation / CLI / scripting — reality check

**No documented headless CLI or scripting API** (no `--` batch render, no Python). What exists:
- **Post-Export Command** (Preferences): runs a shell command after export — documented example `$quit` to close EmberGen after an overnight export. Source: https://docs.jangafx.com/embergen/pages/references/settings.html
- **User Variables:** custom vars in export-node paths — change one to re-point many export nodes (batch-path management).
- **Filename variables** (`$(projectdir)`, `####`) for scripted-style naming.
- **In-app procedural control** instead of scripting: Keyframe Animation, Modulation Curves, Randomization / Randomize All Seeds, Palettes, Range Slider, Histogram; per-param Override cycler.

**Bottom line:** automation is GUI-driven + a post-export shell hook, not a render-farm CLI. A scriptable API is roadmapped for 2.0 only — don't promise it exists. Automate *downstream* (your own scripts over the exported VDB/flipbook/Alembic files).

---

## 8. Performance & gotchas

- **Export locally, never to a network drive** — "can cause instability in simulations and renders" (all file types).
- **Timeline misalignment** = #1 import gotcha — sync Timestep (60 Hz) / Import FPS / Backplate FPS, then Frame Stride = 1.
- **VDB excludes shading/lighting** — carries volume channels only; relight in DCC.
- Default VDB channels are only Density + Flames — **explicitly enable Temperature/Velocity** if your downstream shader/motion-blur needs them.
- **Unlit Mode (U)** + lower viewport quality (Ctrl+1) speed interactive work; watch Sim Time / Sim Speed (mVox/s) / Voxels (mVox).
- `[non-docs]` **VRAM is the hard ceiling** — whole sim on one GPU; UI lag/judder is the over-budget warning.
- `[non-docs]` **Enlarging the container spikes VRAM**; changing voxel size just shrinks the container. Reduce voxel count/resolution *before* enlarging bounds (different mental model from TurbulenceFD).
- `[non-docs]` **Upscaling costs memory** — apply selectively (smoke only), not globally.

---

## 9. Quick node index (documented names)
`Camera` · `Camera: Look At` · `Collider` · `Color` · `Color Gradient` · `Color Selector` · `Emitter: Volume` · `Emitter: Particles` · `Force: Line` · `Force: Toroidal` · `Force: Noise` · `Force: Point` · `Simulation` · `Volume` (+ Volume Processing) · Shading/`Light` nodes · `Shape` nodes · `Import` (FBX/OBJ/ABC) · `Render` · `Export: VDB` · `Export: Particles`.

---

## 10. Primary sources
- Docs — https://docs.jangafx.com/embergen/ · Getting Started — /pages/getting_started.html · UI Reference — /pages/references/ui_reference.html · Node List — /pages/references/node_list.html · Settings — /pages/references/settings.html · VDB Export — /pages/references/How-To%20Guides/exportVDB.html · FAQ — /pages/FAQ.html
- Product — https://jangafx.com/software/embergen
- 6-way lighting (Unity) — https://unity.com/blog/engine-platform/realistic-smoke-with-6-way-lighting-in-vfx-graph
