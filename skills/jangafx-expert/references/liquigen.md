# LiquiGen — Expert Reference (JangaFX Real-Time Liquid Simulation)

> Scope: distilled from the official docs at https://docs.jangafx.com/liquigen/ (crawled July 2026), cross-checked against JangaFX marketing/press for facts absent from docs. Every claim is sourced inline. Where the docs are silent or ambiguous, it is flagged explicitly per the zero-guessing rule — do **not** treat flagged items as confirmed.

---

## 1. What LiquiGen Is — and When to Reach for It

LiquiGen is JangaFX's **real-time, GPU-accelerated liquid simulator**, the liquid counterpart to EmberGen (fire/smoke). It uses a **hybrid particle + voxel solver** (a FLIP/PIC-family hybrid): particles carry liquid position, while a voxel grid computes and stores velocity, pressure, divergence, and force fields. The docs' own analogy: *"The particles are like chess pieces that can be anywhere on the board but get specific directions within specific checkers."* Particles are then **meshed** (surface-reconstructed) into a liquid mesh that gets shaded for rendering.
Source: https://docs.jangafx.com/liquigen/pages/getting_started.html, https://docs.jangafx.com/liquigen/pages/usingLiquiGen.html

**Reach for LiquiGen when you need:**

| Use case | Why LiquiGen fits |
|---|---|
| Fast, art-directable liquid FX for **games** | Native VAT + Mesh Flipbook export, real-time iteration |
| **Splashes, pours, oceans-in-a-box, whitewater/foam/spray** | Built-in whitewater system (spray/foam/bubbles) |
| Liquid **mesh caches for DCC/VFX** (Maya/Houdini/Blender/UE) | Alembic mesh + VDB velocity + particle export |
| Rapid **look-dev / previz** of fluid behaviour | Press-play real-time solve, tweak parameters live |
| Motion-graphics liquid / logo reveals | Node-graph forces, modulators, MIDI/ADSR control |

**When NOT to reach for it (based on documented gaps):**
- You need **headless/CLI/scripted batch** simulation → **no CLI or scripting API is documented** (see §6). It is a GUI-driven tool.
- You need **guaranteed, physically-exact viscosity/surface-tension coefficients** with published solver semantics → these are only partially exposed (see §4). Flag: LiquiGen targets *plausible real-time* liquid, not offline-accurate CFD.
- **macOS**: not supported at 1.0 (Windows 10+ / Linux only); macOS was slated for 1.1.
Source: https://www.cgchannel.com/2025/07/jangafx-releases-liquigen-1-0/

---

## 2. Core Simulation Workflow

### 2.1 The four UI panels
The interface subdivides into: **Viewport**, **Timeline Editor**, **Node Editor**, and **Properties Panel** (each resizable/fullscreenable). The Properties Panel has tabs: **Node Details** (parameters), **Favorites** (starred params), **Timeline** (keyframed params), **Randomized** (variation).
Source: https://docs.jangafx.com/liquigen/pages/usingLiquiGen.html

### 2.2 Building a first sim (documented recipe)
1. **Pick a scale template** in the Project Manager (scale matters — the solver is world-scale aware; see Master Scale in §4).
2. **Emitter + Shape** → connect a `Shape: Primitive` (Sphere/Box/Capsule/Torus/Cylinder/Cone/Ellipsoid) into an `Emitter` to spawn liquid.
3. **Collider(s)** → add obstacles the liquid interacts with.
4. **Forces** → gravity/turbulence/etc.
5. **Simulation node** → configure **voxel size** (detail), **meshing**, and shader/appearance.
6. Press **play**, tune live, then **Export**.
Source: https://docs.jangafx.com/liquigen/pages/getting_started.html, https://docs.jangafx.com/liquigen/pages/usingLiquiGen.html

### 2.3 The node roster (from the Node List)
Source: https://docs.jangafx.com/liquigen/pages/references/node_list.html

| Category | Nodes |
|---|---|
| **Shape** | `Shape: Primitive` (Sphere/Box/Capsule/Torus/Cylinder/Cone/Ellipsoid) |
| **Sim interaction** | `Emitter`, `Collider`, `Drain` |
| **Whitewater** | `Whitewater`, `Whitewater Source` |
| **Forces** | `Force: Constant`, `Force: Shape`, `Force: Drag`, `Force: Turbulence`, `Force: Line`, `Force: Toroidal` |
| **Lighting** | `Light: Directional`, `Light: Point`, `Light: Area`, `Skybox` |
| **Look** | `Appearance` |
| **Camera** | `Camera`, `Camera: Look At` |
| **I/O** | `Import`, `Export: Image`, `Export: Mesh`, `Export: VDB`, `Export: Particles` |
| **Modulators** | `Mod: Constant`, `Mod: Oscillator`, `Mod: Cycle`, `Mod: Math`, `Mod: Time Shift`, `Mod: Combine`, `Mod: ADSR`, `Mod: MIDI` |
| **Color** | `Color: Constant`, `Color: Gradient`, `Color Selector` |

> **Note:** There is also a root **Simulation node** (referenced in Getting Started/FAQ) that holds global solver + meshing settings and the **Timestep** parameter (default 60 fps). It is not enumerated as an addable node in the Node List excerpt — it is the scene-global node. Flag: its full parameter list is **not individually documented** on an accessible page (see §4).

---

## 3. Node Parameters That Matter (verbatim)

### 3.1 Emitter — the primary liquid source
Source: https://docs.jangafx.com/liquigen/pages/references/node_list.html

| Section | Parameter | Meaning (verbatim/near-verbatim) |
|---|---|---|
| Activity | Emitter activity | If false the emitter is ignored |
| Behavior | **Emission Location** | **Volume** or **Surface** |
| Behavior | **Emission Mode** | **Fill** or **Continuous Generation** |
| Behavior | Volume flow rate density | "How much liquid volume should be generated by unit of volume" |
| Behavior | Just once | Single simulation-step emission |
| Behavior | Reset previous liquid | Velocity reset within emitter |
| Behavior | Lifetime range | Min/max lifespan for emitted particles |
| Behavior | **Dynamic viscosity coefficient** | Liquid thickness (this is where per-emitter viscosity lives) |
| Behavior | **Stickiness** | "The amount the liquid will stick to colliders" |
| Velocity | Speed | Initial velocity magnitude |
| Velocity | Direction | **Fixed** or **Towards Target** (Fixed direction vector / Target Position) |
| Velocity | Velocity transfer | "Percentage of velocity transfer of the emitter in the simulation" |

> **Interaction insight:** Viscosity and stickiness are **per-emitter**, not a single global. Different emitters can inject liquid of different viscosities into one sim.

### 3.2 Collider & Drain
- **Collider** — `Collider activity`, `Invert shape` ("outside of the shape being considered interior"), `Show`; plus a full **Appearance** block (Diffuse/Refractive/IOR/Transmission/Absorption/Reflectivity/Roughness/Emission) so colliders render as glass/solids.
- **Drain** — `Drain activity`, `Invert shape`, `Show`; removes liquid on contact. Whitewater can optionally honor drains (`Apply liquid drains`).

### 3.3 Whitewater / Foam / Spray / Bubbles
Source: https://docs.jangafx.com/liquigen/pages/references/node_list.html

The **Whitewater** node is the heart of secondary detail. Three independently-toggleable systems: **Simulate spray**, **Simulate foam**, **Simulate bubbles**.

**Source (emission) parameters:**
| Parameter | Meaning |
|---|---|
| **Trapped air: Range** | Range that determines where particles emit due to trapped air |
| **Trapped air: Max Depth** | Max air-trapping depth |
| **Wave crest: Range** | Range that determines when particles emit on wave crests |
| **Trapped air: Emission rate** | Particles per voxel per second (trapped air) |
| **Wave crest: Emission rate** | Particles per voxel per second (wave crests) |
| **Initial HP** | Range of health points assigned per particle |
| **Foam / Bubbles: initial spread** | Randomness of initial direction |

**Dynamics parameters:** `Surface offset`, `Apply liquid drains`, `Spray: Apply liquid forces`, `Spray: Buoyancy`, `Spray: Decay`, `Foam: Decay`, `Foam: Drag`, `Bubbles: Decay`, `Bubbles: Buoyancy`, `Bubbles: Drag`, **`Bubbles to foam rate`** ("amount of bubbles that will turn into foam instead of disappearing").
**Appearance:** per-class `Radius` + `Opacity` for Spray / Foam / Bubbles.

**Whitewater Source** node adds a **Mask** section: `Shapes offset`, `Falloff distance` — lets you spatially restrict/seed whitewater to specific shapes.

> **Mental model of the HP system:** whitewater particles spawn with **Initial HP**, lose HP via per-class **Decay** each second, and die at 0 — except bubbles, a fraction of which (**Bubbles to foam rate**) convert to foam. Tune lifetime/density via emission rate + Initial HP + Decay together; they interact multiplicatively on visible particle counts and VRAM.

### 3.4 Forces (all share Activity + a Mask falloff block)
Source: https://docs.jangafx.com/liquigen/pages/references/node_list.html
Common Mask: `Mask Falloff` (None/Linear/Quadratic/Cubic/Custom), `Mask falloff exponent`, `Mask falloff distance`, `Mask dilation`.

| Force | Key behavior params |
|---|---|
| **Constant** | `Apply force` (per-axis constant acceleration) — used for gravity/directional push |
| **Shape** | `Matching Inside/Outside values`, `Constant force dampening`, `Attract Strength`, `Falloff`, `Shape dilation` |
| **Drag** | `Strength`, `Power` ("particles moving faster are slowed down more") |
| **Turbulence** | `Scale`, `Seed`, `Octaves`, `Lacunarity`, `Gain`, `Strength`, `Animation Speed`, `Bias`, `Rotation` (fBm noise) |
| **Line** | `Position`, `Rotation`, `Push/Twist/Repel Strength`, `Falloff` (+ exponent/percent/radius/inner bound), `Use line segment`, `Line segment length`, `Inverse falloff` — tornadoes, wind shears, push forces |
| **Toroidal** | `Position`, `Rotation`, `Push/Twist/Repel Strength`, `Radius`, `Falloff` set — "akin to a magnetic field" |

### 3.5 Appearance (liquid shading)
`Diffuse color`, `Refractive` → reveals `IOR`, `Transmission color`, `Absorption strength`, `Reflectivity` (Fresnel at normal incidence), `Roughness`, `Emission intensity/color`. This drives the meshed liquid's look for the built-in path tracer/rasterizer render passes.

### 3.6 Import (animated meshes/colliders in)
- **Asset Settings:** `Voxel Sizing` (Relative % of sim, or Absolute), `Relative Resolution` (voxel scaling of imported mesh vs sim).
- **Transform:** Position/Rotation/Scale + `Master scale` ("large scale multiplier, used to easily scale very large or very small scenes").
- **Playback:** loop + `Override FPS`.
Formats: **FBX, OBJ, ABC** import (static or animated), plus **Camera Import**.
Source: https://docs.jangafx.com/liquigen/pages/references/node_list.html, https://docs.jangafx.com/liquigen/pages/FAQ.html

---

## 4. Solver / Global Settings — What's Documented (and the gaps)

### 4.1 Confirmed
- **Voxel size** — lives in the **Simulation node → Simulation tab**. Verbatim: *"determines the size of the smallest possible detail in your simulation and therefore directly affects the realism and level of detail… It also directly affects the simulation speed as more voxels and particles take longer to compute."* This is **the** master detail/perf knob. Smaller voxel = more particles/detail = slower + more VRAM.
- **Timestep** — Simulation node, default **60 fps** (this is the sim rate that must be matched across Import FPS, Camera backplate rate, and Export frame stride — see §5.3).
- **Viscosity** — exposed **per-Emitter** as `Dynamic viscosity coefficient`.
- **Stickiness** — per-Emitter (adhesion to colliders).
- **Gravity / directional acceleration** — via `Force: Constant`.
Source: https://docs.jangafx.com/liquigen/pages/usingLiquiGen.html, https://docs.jangafx.com/liquigen/pages/FAQ.html

### 4.2 Explicitly flagged as NOT documented in accessible pages
The following commonly-expected FLIP parameters are **not enumerated verbatim** anywhere reachable in the public docs (the Simulation node's full tab-by-tab parameter list has no dedicated reference page in the crawl):
- **Sub-steps / adaptive sub-stepping** — not documented. *(Do not assume a value.)*
- **CFL number** — not documented.
- **Pressure-solve iterations** — not documented.
- **Global surface tension coefficient** — not documented as a named parameter. *(Only whitewater surface behavior and per-emitter viscosity are exposed.)*
- **Meshing tab parameters** (particle radius, smoothing/blur, mesh resolution independent of voxel size) — the meshing *step* is documented conceptually ("particles go through meshing to produce the liquid mesh"), but individual meshing controls are **not** listed.
- **Domain bounds / adaptive (dynamic) domain** — not documented in the crawl. *(EmberGen has an adaptive container; whether LiquiGen matches is unconfirmed — flag.)*

> Practical guidance: treat **voxel size + emitter viscosity/stickiness + force setup** as your primary levers, since those are the confirmed ones. For sub-steps/CFL/meshing fine-tuning, consult the in-app Simulation node tooltips or JangaFX Discord — the web docs don't cover them.

### 4.3 Project/Preferences settings (confirmed)
Source: https://docs.jangafx.com/liquigen/pages/references/settings.html
- **Project Settings:** general tags, shape scaling, read-only, parameter favoriting, seed randomization.
- **Renderer Settings:** viewport visibility toggles.
- **Preferences → Simulation Preemption:** *"Time spent on a simulation step before pausing to render and update the UI"* — the UI-responsiveness vs solve-throughput tradeoff.
- Also: UI scaling, export warnings, camera controls, keyboard shortcuts, user variables.

---

## 5. Export Pipeline (the payoff)

LiquiGen ships four export nodes. All share: **First frame, Number of frames, Frame stride, Numbering offset, Directory, Filename** (filename supports dynamic variables).
Source: https://docs.jangafx.com/liquigen/pages/references/node_list.html

### 5.1 `Export: Mesh` — liquid surface out
**Formats:** **Alembic (.abc)**, **FBX**, **Mesh Flipbook**, **Vertex Animated Texture (VAT)**, **OBJ**.
Format-specific:
- **Alembic / Mesh Flipbook:** `Export velocity` checkbox (writes velocity for motion blur downstream).
- **VAT:** `Maximum lookup texture width`, `Maximum texture width`.

| Format | Best for | Notes |
|---|---|---|
| **Alembic** | Maya / Houdini / Blender / UE VFX pipelines | Animated mesh cache; enable `Export velocity` for mblur |
| **Mesh Flipbook** | Games — one FBX with all frames packed, separated by **vertex color** per frame | Single-file mesh sequence |
| **VAT** | Real-time engines (UE/Unity) — bake deformation to textures | Set lookup/texture widths |
| **OBJ / FBX** | Per-frame interchange | Classic sequence |

> **Wiring:** the `Export: Mesh` node connects to the **Mesh pin of the Simulation node**.
> **Known 1.0-era caveat (press, not docs):** early reports of Alembic foam particles popping and some frames not fully baking — JangaFX flagged the Alembic exporter for review. Verify your bake frame-by-frame. Source: https://digitalproduction.com/2025/08/01/jangafx-unleashes-liquigen-1-0-liquids-meet-production%E2%80%91ready-polish/

### 5.2 `Export: VDB` — volumetric velocity fields
**Fields:** `Velocity staggered`, `Velocity centered`. Params: `Field name` (custom grid naming), `Length unit`, plus the standard frame-range/dir/filename set.
Use for velocity-driven motion blur or volumetric re-sim in Houdini/Blender/Arnold/Redshift. (VDB export workflow mirrors EmberGen's — see interop note §7.)

### 5.3 `Export: Particles` — raw particles out
**Formats:** **Alembic**, **VAT**.
Alembic toggles let you choose which particle streams to write: **Main liquid**, **Spray**, **Foam**, **Bubbles** (independent checkboxes). VAT: `Maximum texture width`.
> Flag: the docs list **Alembic + VAT only** for particles. **`.bgeo` (Houdini) is NOT a documented LiquiGen export format** — do not assume it exists.

### 5.4 `Export: Image` — rendered flipbooks & passes
**Formats:** PNG (8/16-bit), TGA, EXR (16/32-bit, uncompressed/HDR). **Export Mode:** Flipbook or Sequence.
**Render passes:** Render Viewport, Render All (Path Tracer), Render All (Rasterizer), Alpha, Diffuse, Reflect, Refract, Depth, Normals, Albedo, Roughness, Reflectivity, Rim_Reflectivity, Emission, Transmission, **Thickness**.
Render settings: `Shapes Rendering` (Ignore / Regular Lit / Regular Unlit / Regular Black / **Holdout**), `Supersampling`, Path-Tracer `Samples per-pixel` & `Samples per-frame`, Depth `Min/Max`, Thickness `Scale`.
> The Thickness/Reflect/Refract passes are the game-friendly way to fake refractive liquid cheaply in-engine without shipping geometry.

### 5.5 Timeline alignment (critical for correct playback speed)
Source: https://docs.jangafx.com/liquigen/pages/FAQ.html — these four **must** agree:
1. **Timestep** in the Simulation node (default 60 fps)
2. **FPS** in the Import node's Playback tab
3. **Backplate frame rate** in the Camera node
4. **Frame Stride** in the Export node (set to **1** when the others match)
Mismatch = liquid that plays too fast/slow relative to imported animation or backplate.

### 5.6 Recommended export targets by destination

| Destination | Recommended export |
|---|---|
| **Unreal / Unity (games, real-time)** | **VAT** (mesh or particles) or **Mesh Flipbook**; image passes for cheap refraction |
| **Unreal (VFX-quality)** | **Alembic** mesh (+velocity) — note early Alembic caveats above |
| **Houdini / Maya / Blender (DCC)** | **Alembic** mesh; **VDB** velocity for mblur/re-sim; **Alembic particles** for whitewater |
| **Compositing (Nuke/AE)** | **Export: Image** EXR multi-pass (Thickness/Reflect/Refract/Depth/Normals) |

---

## 6. Automation / CLI / Scripting

**Status: no documented CLI, headless, or scripting API for LiquiGen.** The crawl surfaced only GUI-driven workflows (assemble node graph → play → click Export). Web search for a LiquiGen command-line/headless mode returned nothing official (results conflated an unrelated Ruby "liquigen" gem).
Source: https://docs.jangafx.com/liquigen/, https://jangafx.com/software/liquigen

**What passes for "automation" inside the GUI (documented):**
- **Modulators** (`Mod: Oscillator/Cycle/Math/Combine/Time Shift/ADSR/MIDI`) procedurally drive any parameter — including live MIDI control.
- **Keyframe Animation** + **Modulation Curves** for timeline-driven parameter animation.
- **Randomization** and **Expressions** (there's an Expressions how-to) for parametric variation.
- **Export nodes** batch a full frame range in one click via First frame / Num frames / Frame stride.

> If a caller needs render-farm/headless batch, that capability is **not confirmed to exist** — direct them to the JangaFX roadmap/forum rather than assuming it. Flag as undocumented.

---

## 7. Hardware, Performance & Interop

### 7.1 System requirements (from press — NOT the docs; docs omit hardware)
Source: https://www.cgchannel.com/2025/07/jangafx-releases-liquigen-1-0/

| Requirement | Spec |
|---|---|
| **Min GPU (NVIDIA)** | GeForce **GTX 1060** or better |
| **Min GPU (AMD)** | Radeon **RX 580** or better |
| **OS** | Windows 10+ and Linux (macOS targeted for 1.1) |
| **Recommended** | Modern high-VRAM GPU (RTX-class) for high-res/large domains — demos ran on RTX 4090 |

> Flag: JangaFX does **not** publish a hard minimum-VRAM figure. Because the solver is GPU-resident, **VRAM is the practical ceiling** on voxel resolution + whitewater particle counts. Treat GTX 1060/RX 580 as a floor, not a production target.

### 7.2 Performance tuning levers (confirmed)
1. **Voxel size** — the dominant cost. Halving voxel size roughly multiplies particle/voxel counts (and VRAM) by ~8× in 3D. Prototype coarse, finalize fine.
2. **Whitewater particle budget** — emission rate × Initial HP × (1/Decay) governs live particle counts; the biggest secondary VRAM/perf sink. Disable spray/foam/bubbles you don't need.
3. **Simulation Preemption** (Preferences) — raise for faster solving, lower for a more responsive UI while scrubbing.
4. **Supersampling / Path-Tracer samples** — render-time only; separate from sim cost.
5. **Master scale / correct project scale template** — wrong world scale distorts gravity/surface behaviour and forces you into bad voxel sizes.

### 7.3 Interop with EmberGen / GeoGen / IlluGen
- LiquiGen shares EmberGen's UI paradigm, node/modulator system, VDB export approach, and How-To structure (the LiquiGen node-list page cross-links EmberGen How-To guides). The **VDB export workflow is documented more fully on EmberGen's page** and applies in principle: https://docs.jangafx.com/embergen/pages/references/How-To%20Guides/exportVDB.html
- **GeoGen / IlluGen** are sibling JangaFX tools referenced in the shared docs nav; **no documented direct scene-interchange pipeline** LiquiGen↔GeoGen exists in the crawl beyond common formats (FBX/OBJ/ABC import, VDB). Flag: any tighter integration is undocumented.
- Cross-tool data exchange is effectively via the **standard formats**: import FBX/OBJ/ABC meshes + cameras into LiquiGen; export Alembic/VDB/VAT out to any DCC or to EmberGen-adjacent pipelines.

---

## 8. Gotchas Checklist

- **Alembic bake integrity (1.0-era):** foam particles reported popping; some frames may not fully bake. Verify frame-by-frame before handing off. (Press-sourced.)
- **`.bgeo` is not a documented export** — particles export as **Alembic or VAT only**. Don't promise Houdini-native particles.
- **No CLI/headless/scripting** — GUI-only; plan pipelines around manual export or in-GUI modulators.
- **Timeline mis-sync** — the four-way match (Timestep / Import FPS / Backplate rate / Frame Stride) is the #1 cause of wrong liquid playback speed.
- **Solver internals undocumented** — sub-steps, CFL, iterations, global surface tension, and meshing controls aren't in the public docs. Don't cite specific values; read in-app tooltips.
- **Viscosity is per-emitter**, not global (`Dynamic viscosity coefficient`) — set it on each Emitter.
- **macOS unsupported at 1.0.**
- **VRAM is the real limit** — no published minimum; high voxel resolution + heavy whitewater will OOM before compute becomes the bottleneck.
- **Simulation node isn't in the "add node" list** — it's the scene-global node holding voxel size, meshing, timestep, and the Mesh output pin.

---

### Source URLs
- LiquiGen docs home — https://docs.jangafx.com/liquigen/
- Getting Started — https://docs.jangafx.com/liquigen/pages/getting_started.html
- Using LiquiGen — https://docs.jangafx.com/liquigen/pages/usingLiquiGen.html
- Node List (all node params) — https://docs.jangafx.com/liquigen/pages/references/node_list.html
- Settings — https://docs.jangafx.com/liquigen/pages/references/settings.html
- FAQ (export + timeline alignment) — https://docs.jangafx.com/liquigen/pages/FAQ.html
- EmberGen VDB export (shared workflow) — https://docs.jangafx.com/embergen/pages/references/How-To%20Guides/exportVDB.html
- LiquiGen product / download — https://jangafx.com/software/liquigen
- LiquiGen 1.0 release (hardware, features, Alembic caveat) — https://www.cgchannel.com/2025/07/jangafx-releases-liquigen-1-0/ ; https://digitalproduction.com/2025/08/01/jangafx-unleashes-liquigen-1-0-liquids-meet-production%E2%80%91ready-polish/
