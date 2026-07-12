# Export Pipeline — JangaFX outputs → your destinations

> Cross-cutting reference: what each JangaFX export format is good for and how it lands in **your** stack — WebGPU (now), ComfyUI/Blender NeuralFX (in progress), and Unreal (your engine of choice, next). The JangaFX-side format facts are documented in the per-product references; the destination-side facts are marked as **[established]** (well-known engine behavior) vs **[your-engineering]** (things you build, not a JangaFX feature) vs **⚠️ verify**. Per zero-guessing: JangaFX ships **no live-link plugin** — everything here is **export a file → import/consume it**.

Related: `sim-as-conditioning.md` (using sims to drive generative FX), `embergen.md`/`liquigen.md`/`geogen.md`/`illugen.md` (export node params).

---

## 1. The format → destination matrix

| JangaFX output | Source tool(s) | WebGPU (yours) | ComfyUI / NeuralFX | Blender / DCC | Unreal |
|---|---|---|---|---|---|
| **Flipbook / sub-UV sheet** (PNG/TGA/EXR) | EmberGen, LiquiGen (image), IlluGen | ✅ atlas → animated billboard/soft-particle | ✅ frames as driving video / conditioning | ✅ image-sequence plane | ✅ Niagara/Cascade sub-UV material |
| **Image sequence + data passes** (EXR: Depth, Motion Vector, Normal, Albedo, Alpha, 6-way) | EmberGen, LiquiGen | ✅ multi-texture shading | ✅✅ **best control signals** (see conditioning ref) | ✅ comp in Blender/Nuke | ✅ motion-vector sub-UV, depth soften |
| **OpenVDB** (density/temp/flames/velocity) | EmberGen (LiquiGen: velocity) | ⚠️ needs your own 3D-texture loader | ✅ via Blender/Houdini bridge | ✅✅ native volume import + render | ✅ **Sparse Volume Texture** (SVT) |
| **Alembic (.abc)** mesh/particles | LiquiGen, IlluGen, EmberGen (particles) | ⚠️ convert to glTF/your format | ➖ (mesh, not image) | ✅✅ native | ✅ Geometry Cache / Niagara |
| **VAT** (Vertex Animated Texture) | LiquiGen, IlluGen | ✅ vertex-texture shader | ➖ | △ (engine-oriented) | ✅✅ real-time deforming mesh |
| **Mesh Flipbook** (FBX, vertex-color frames) | LiquiGen, IlluGen | △ decode in shader | ➖ | △ | ✅ shader-driven playback |
| **Heightmap / splat / mask** (16-bit) | GeoGen | ✅ terrain displacement | ➖ | ✅ displacement/texture | ✅✅ Landscape import |
| **Packed texture / flowmap / noise** | IlluGen | ✅✅ shader inputs | ✅ conditioning textures | ✅ material inputs | ✅ material/Niagara inputs |

✅✅ = best-fit primary path · ✅ = works well · △ = possible w/ work · ⚠️ = you build the loader · ➖ = not applicable

---

## 2. WebGPU (your current stack)

Your `webgpu-vfx` lib + browser tooling. JangaFX has no web export — you consume its **files**.

### 2.1 Flipbook atlases → animated billboards **[established + your-engineering]**
- Export from EmberGen/LiquiGen **Render node → Flipbook**, `Columns × Rows` = frame grid (power-of-two sheet, e.g. 8×8). Prefer **EXR** if you want HDR emission; **PNG** for LDR + alpha.
- In WebGPU: upload the sheet as a `texture_2d`, compute the sub-UV from `frame = floor(time*fps)`, `col = frame % cols`, `row = frame / cols`, offset UVs. Bilinear-blend between consecutive frames to avoid stepping.
- Pair the **Motion Vector pass** as a second sheet → in-shader frame interpolation (warp frame N toward N+1 by the MV) for smooth slow-mo without more frames. **[established technique; you implement the shader]**
- Additive blend for fire/emissive; premultiplied-alpha translucent for smoke. Watch the "no additive over white background" gotcha from your emoji-slopes work — composite over scene, not over white.

### 2.2 Volumetric (VDB) in WebGPU ⚠️ **[your-engineering]**
- WebGPU has **no OpenVDB loader**. Path: bake EmberGen VDB → convert to a **dense 3D texture** (`texture_3d<f32>` or packed `rgba8`/`r16f`) offline (Blender/Houdini/`openvdb` Python → raw or KTX2), then **raymarch** it in a fragment/compute shader.
- Cheaper alt for real-time web: skip true volume — use the **flipbook + 6-way lighting** sheet (§2.3) for a volumetric *look* at 2D cost.
- Cross-ref `graphics-api-expert` `references/fluid-simulation.md` / volume raymarching for the WGSL side.

### 2.3 6-way lighting flipbooks (dynamic relight, 2D cost) **[established]**
- EmberGen Render passes `Six Point 1 (TLR)`, `Six Point 2 (BBF)`, `Six Point Normal Map` pack lightmaps for six directions.
- In WebGPU: sample all six, dot each against your scene light direction, sum → the smoke/fire relights to your scene lighting on a flat billboard. Best for background/ornamental FX and explosions, not hero close-ups.
- **Caveat (from Unity's writeup):** JangaFX doesn't control the intensity in these maps — expect to add an intensity multiplier uniform and hand-tune.

---

## 3. ComfyUI / NeuralFX (in progress)

This is where JangaFX becomes a **control-signal generator** for generative video. Full treatment in `sim-as-conditioning.md`; summary:

- Export EmberGen/LiquiGen **image sequences with data passes** (EXR): **Depth**, **Motion Vector**, **Normal** (gradient or six-point normal), **Albedo**, **Alpha/masks**.
- Feed the **beauty pass or alpha/mask** as a driving video for AnimateDiff/V2V or SCAIL-style restyle; feed **Depth/Normal** as ControlNet-style conditioning to lock structure; use **Motion Vector** for temporal consistency / optical-flow guidance.
- IlluGen's **flipbook encode/decode** repacks the result back into an engine-ready RGBA atlas — the round-trip that closes the loop from sim → AI restyle → game asset.
- Ties into your existing `comfyui-video-restyle` skill and SCAIL neural-VFX method (memory): JangaFX gives you *clean, controllable* driving/conditioning inputs instead of scraped footage.

---

## 4. Blender / DCC (NeuralFX authoring + offline render)

- **VDB is the primary currency.** EmberGen `Export: VDB` → Blender **Volume** object (File → Import → OpenVDB, or drag the `.vdb` sequence). Set **Coordinate System = Z Up Right Handed** in EmberGen for Blender. Relight in Blender — **VDB carries no shading**, only density/temp/flames/velocity grids. Enable **Temperature** channel in the export if you want blackbody fire in Cycles.
- **Velocity grid** → Blender/Cycles/Redshift motion blur. Export it explicitly (off by default).
- **Alembic** for LiquiGen liquid surfaces + whitewater (separate particle streams). Enable **Export velocity** for motion blur; verify the bake frame-by-frame (1.0-era Alembic foam-popping caveat).
- Use Blender as the **bridge to WebGPU/ComfyUI**: it's your converter (VDB → 3D texture, Alembic → glTF, render → image sequence).

---

## 5. Unreal (your engine of choice, next)

All **[established]** UE features — JangaFX exports the files, UE imports them.

### 5.1 Real-time volumetrics — Sparse Volume Textures (SVT)
- UE 5.3+ imports **OpenVDB** sequences as **Sparse Volume Textures** → render via the **Heterogeneous Volume** component / material domain. This is the modern path for EmberGen smoke/fire as *true 3D volume* in-engine (not a flipbook).
- Export EmberGen VDB with **Density + Temperature** (temperature → blackbody emission in the volume material). Frame-sequence VDB → animated SVT.
- ⚠️ verify current UE version's SVT import limits (resolution/VRAM) for your target platform.

### 5.2 Flipbooks → Niagara / Cascade sub-UV
- Export `Columns × Rows` sheet + **Motion Vector** sheet. Niagara **SubUV Animation** module + **SubUVMotionBlur/blend** for smooth playback. Depth pass → soft-particle depth fade.
- 6-way lighting sheets → the standard UE 6-way material for relightable smoke.

### 5.3 GeoGen → Landscape
- 16-bit heightmap → Landscape *Import from File*; World Partition for large tiled worlds; splat/mask maps → Landscape material layers. See `geogen.md` §4.3 for the Z-scale formula and tiling int32 caveat. ⚠️ GeoGen's own one-click tiled export is roadmap-delayed — use the manual path.

### 5.4 LiquiGen liquids → UE
- **VAT** (mesh or particles) for real-time deforming liquid, or **Alembic** Geometry Cache for VFX-quality (heavier). Image **Thickness/Reflect/Refract** passes → cheap faked refraction material without shipping geometry.

---

## 6. Choosing the right export (decision shortcuts)

- **"Real-time, must be cheap"** → **flipbook** (+ motion vector, + 6-way if relightable). Works everywhere.
- **"Real-time, must be true 3D volume"** → **VDB** → UE **SVT** (or your WebGPU 3D-texture raymarcher).
- **"Deforming liquid/mesh in a game engine"** → **VAT** (real-time) or **Mesh Flipbook**.
- **"VFX-quality offline / hero shot"** → **VDB** (gas) or **Alembic** (liquid) into Blender/Houdini + path trace.
- **"Feed a generative/AI pipeline"** → **EXR image sequence with Depth/Normal/Motion-Vector/Alpha passes** → see `sim-as-conditioning.md`.
- **"Terrain for a game"** → GeoGen **16-bit heightmap + splat maps** → Landscape/Unity Terrain (mind the frozen-dev + format caveats).

---

## 7. Universal export gotchas (all tools)

- **Export locally, never to a network drive** (EmberGen docs — instability in sims/renders). Applies suite-wide.
- **Timeline sync (60 Hz default):** Timestep = Import FPS = Backplate FPS, then Frame Stride = 1. #1 cause of wrong playback speed.
- **VDB/Alembic carry no shading** — relight/shade in the destination.
- **Enable non-default channels** — VDB defaults to Density+Flames; enable Temperature/Velocity if needed. Particle Alembic: enable colors/densities/lifetimes/sizes.
- **Coordinate system per target** — Blender Z-up, Maya Y-up; set it on the export node, not after.
- **No JangaFX render CLI** — automate *downstream* over the exported files (your scripts), not the app.
