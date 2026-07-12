# JangaFX Suite — Overview, Licensing, Hardware & Interop

> Suite-wide reference shared by all four tools. Sourced from docs.jangafx.com, jangafx.com product/roadmap/pricing pages, JangaFX forums, and launch press (crawled July 2026). Facts absent from official docs are flagged; do not treat flagged items as confirmed.

---

## 1. The four products at a glance

| Product | What it is | Primary exports | Docs status |
|---|---|---|---|
| **EmberGen** | Real-time GPU volumetric fluid sim — fire, smoke, explosions, magic | Flipbooks, EXR/PNG/TGA image sequences, **VDB** volumes | Full (`embergen.md`) |
| **LiquiGen** | Real-time liquid sim (hybrid PIC/FLIP) + real-time meshing + whitewater (spray/foam/bubbles) | Alembic mesh, VDB velocity, VAT, mesh flipbooks, EXR passes | Full (`liquigen.md`) |
| **GeoGen** | Node-based procedural terrain & planet generation with erosion sims + spline control (**beta**) | Heightmaps, meshes, masks/splat maps | WIP/partial (`geogen.md`) |
| **IlluGen** | Procedural 2D/3D **VFX asset** generation (NOT lighting) | Textures, flipbooks, VFX meshes, VAT, Alembic | Placeholder (`illugen.md`) |

The user has the **Elemental Suite** = all four apps.

---

## 2. Shared node-graph paradigm

The suite deliberately shares one UX language: LiquiGen's docs describe *"an intuitive UI derived from EmberGen"* — *"assemble a small node graph… press play and tweak parameters in real-time, and when you're ready to export, you simply click export."*

Common UI shell: **Menubar · Viewport · Node Graph · Timeline Editor · Properties Panel.**
Source: https://docs.jangafx.com/embergen/pages/references/ui_reference.html

Shared node-graph & workflow concepts (documented for EmberGen, mirrored in LiquiGen):

| Concept | Notes |
|---|---|
| Node types | Emitters, Forces, Simulation, Volume, Render/Export, Import |
| **Keyframe Animation** | Set a parameter's override state to **Timeline Override**; the Timeline tab lists all keyframed params |
| Modulation Curves / Parameter Modulation | Curve-driven parameter control |
| Randomization | "Randomize All" + per-parameter **range sliders** (min/max); Randomized tab |
| Palettes / Color | Color + palette handling |
| Expressions | Documented for LiquiGen ("Expressions" guide) |
| Import | **FBX** (primary), OBJ, ABC (**Alembic OGAWA only**) as emitters/colliders/holdouts; FBX cameras for compositing |

The How-To set (Camera Import, Render Pass Mapping, Color, FBX/OBJ/ABC Import, Range Sliders, Keyframe Animation, Modulation Curves, Parameter Modulation, Palettes, Randomization) appears under **both** EmberGen and LiquiGen — a genuinely shared feature model. Learn it once, apply it across the suite.
Source: https://docs.jangafx.com/embergen/index.html

---

## 3. Scripting / automation / CLI / headless — **mostly absent today**

> **Zero-guessing flag:** there is **no documented Python/Lua API, no command-line render/headless mode, and no scripting interface** in the current shipping releases of any of the four apps. The docs describe an entirely **GUI-driven** workflow: build node graph → play → tweak → click Export.

- **The one CLI is the Floating License Server (`jfs`)** — licensing infrastructure, not content automation (see §5).
- A **"scriptable API" is a roadmap item for EmberGen 2.0** — language (Python vs. other) and whether it enables headless/CLI rendering are **not specified**. Don't promise it exists yet.
  Source: https://www.cgchannel.com/2025/01/check-out-the-new-features-due-in-embergen-2-0/
- **In-GUI "automation" today** = keyframe timeline + parameter modulation + randomization (range sliders). For batch variation, drive parameters with modulators/expressions and export a frame range in one click; there is no farm/headless path to rely on.

**Practical implication for pipelines:** design around **manual export of baked assets**, then automate *downstream* (your own scripts consuming the VDB/Alembic/flipbook files). Do not architect anything that assumes a JangaFX render CLI.

---

## 4. Cross-app & DCC interop (file-based, not live-link)

The interop model is **file interchange**, not a shared-scene bus:

- **EmberGen → DCC/engine:** `Export: VDB` node (from the Volume node's VDB output). VDB reads into Maya/Blender/Houdini/Unreal/Unity. A `Transform` input pin can apply the inverted Import transform to align grids with the source app.
  Source: https://docs.jangafx.com/embergen/pages/references/How-To%20Guides/exportVDB.html
- **Into the apps:** FBX/OBJ/ABC meshes & animation as emitters/colliders; FBX cameras for pixel-perfect compositing.
  Source: https://docs.jangafx.com/embergen/pages/references/How-To%20Guides/import_animation.html
- **IlluGen ↔ suite:** advertises *"direct pipeline links to EmberGen, LiquiGen, and future JangaFX products… by design"*; **flipbook encode/decode** lets EmberGen/Houdini sims be imported, edited, re-exported.
- **LiquiGen:** whitewater (spray/foam/bubbles) *"can be exported separately from the main body of the fluid."*

> **Zero-guessing flag:** no documented native/live-link engine plugin (no "EmberGen for Unreal" runtime bridge) was found. Pipeline = export file → import in engine. If such a plugin exists, it isn't in the reviewed docs.

See `export-pipeline.md` for the full per-format consumption matrix (Unreal SVT/Niagara, Blender, WebGPU, ComfyUI).

---

## 5. Licensing & activation (Elemental Suite)

Two license classes (EULA + Floating License Guide):

| Type | Behavior |
|---|---|
| **Node-Locked** | Tied to one machine per key at a time (1 install; org: 1 per machine per key) |
| **Floating** | Unlimited installs in one physical location; **one concurrent user per license**; check-out/check-in a "lease" from a Floating License Server. N licenses = N concurrent users |

**Floating License Server (`jfs`, command line):**
- Hands out **leases** for a session or until released. Windows (cmd / background service) or Linux (bash).
- Listens on **UDP, default port 3942**; optional per-port lease limits; multiple products at once.
- Key flags: `--product <name>:<key>`, `--port <port>[:<lease_limit>]`, `--host <ipv4>`, `--activation-file <path>:<key>`.
- **Offline activation:** generate request file → upload to `license0.jangafx.com/activate` → run server with the response file.
- Modern server needs: EmberGen **1.2.7+**, LiquiGen **1.0.6+**, GeoGen **0.5.4-beta+**, IlluGen **1.1.2+** (older = legacy TurboFloat). Rules are named `"<software> 1.x floating license server"`.

Source: https://docs.jangafx.com/licensing/ , https://jangafx.com/legal/end-user-license-agreement

**Pricing (Indie/Hobby tier, < $1M revenue), for reference:** Elemental Suite permanent **$525** (+$315/yr optional maintenance); individual apps ~$300 permanent; GeoGen Beta $150; IlluGen subscription $20/mo. Rent-to-own available. Educational: free to universities, ~$60 individual.
Source: https://jangafx.com/software/pricing

---

## 6. Hardware / GPU

> **Zero-guessing flag:** official docs **do not publish formal system requirements**. Figures below are from JangaFX forums + third-party listings — indicative, not authoritative.

| | Minimum | Recommended |
|---|---|---|
| OS | Win10 64-bit / Linux (Mac for IlluGen, LiquiGen 1.1, EmberGen 2.0) | Win10+ / Linux |
| CPU | i5 / Ryzen equiv | i7 / Ryzen 7 |
| RAM | 8 GB | 16 GB+ |
| GPU | GTX 1060 / RX 570 | RTX 3070 / RX 6800; **24 GB VRAM** for heaviest sims |
| API | DirectX 11 | DirectX 12 |

Key facts (JangaFX forums):
- **GPU-bound** — performance scales directly with the GPU.
- **Single-GPU only** — no multi-GPU support.
- **VRAM caps simulation size** — 24 GB fits the heaviest scenes; this is the real ceiling, not compute.
- **NVIDIA and AMD** both supported.
- **EmberGen 2.0** raises the ceiling: rewritten GPU particle system targeting *"over 500 million"* particles on a 24 GB GPU, plus a **path-tracing renderer, USD support, and macOS**.

> **Flag:** the internal graphics API (Vulkan vs DirectX vs Metal) is **unconfirmed** from official sources; third-party listings cite DirectX 11/12.
Source: https://forums.jangafx.com/t/optimal-hardware-specs-for-embergen/106

---

## 7. Versions, roadmap & learning resources

- **Public roadmap** (`jangafx.com/roadmap`, per-product sub-pages) doubles as **changelog + feature-request + bug hub** (replaced the old Trello).
- Timeline: **2024** EmberGen/LiquiGen/GeoGen released; **2025** LiquiGen 1.0 (Jul), IlluGen 1.0 (Jul 28); **2026 roadmap** EmberGen 2.0, IlluGen 1.2, LiquiGen 1.1. GeoGen remains **beta (0.5.x)**.
- Point releases are frequent; permanent licenses don't expire, updates covered by optional annual maintenance.

| Resource | URL |
|---|---|
| Docs | https://docs.jangafx.com/ (full: EmberGen/LiquiGen; placeholder: IlluGen/GeoGen) |
| YouTube | official channel — per-product tutorials & livestreams |
| Discord | community chat / announcements |
| Forums | https://forums.jangafx.com/ |
| Roadmap | https://jangafx.com/roadmap |
| Node reference | per-app **"Node List"** under docs References (EmberGen/LiquiGen only) |
| Support | support@jangafx.com |

---

## 8. Shared export philosophy

**Simulate/author in real time → bake to standard game-ready interchange → consume via files in-engine.** Common currencies across the suite:
- **Volumes:** VDB (EmberGen; LiquiGen velocity) for offline/volumetric.
- **Real-time textures:** flipbooks + image sequences (EXR/PNG/TGA), channel-packed atlases.
- **Geometry/animation:** Alembic (OGAWA), FBX, mesh flipbooks, VAT.
- Targets: **Unreal, Unity, Godot, in-house** — plus cross-app JangaFX links "by design."
