---
name: jangafx-expert
description: Expert advisor for the JangaFX real-time simulation suite — EmberGen (fire/smoke/explosions), LiquiGen (liquids/whitewater/foam), GeoGen (procedural terrain/planets), and IlluGen (procedural VFX asset generation). Use when authoring or debugging a JangaFX simulation; choosing which JangaFX tool fits a task; setting up emitters/forces/colliders/whitewater/erosion; tuning voxel resolution, substeps, dissipation, buoyancy, combustion, viscosity, or vorticity; and above all when EXPORTING sims into a pipeline — OpenVDB, Alembic, VAT, flipbooks/sub-UV sheets, EXR data passes (depth/normal/motion-vector/6-way), or heightmaps — and consuming them in Unreal/Niagara, Unreal Sparse Volume Textures, Unity VFX Graph, Blender/Houdini, WebGPU, or ComfyUI/NeuralFX generative pipelines. Trigger on "EmberGen", "LiquiGen", "GeoGen", "IlluGen", "JangaFX", "VDB export", "flipbook", "sub-UV", "6-way lighting", "whitewater", "VAT export", "sim to Niagara", "sim as ControlNet/conditioning", "elevate particle effects", or any real-time fluid/smoke/fire/liquid/terrain sim-to-engine question — even if the user doesn't name the specific app.
---

# jangafx-expert

Principal-level advisor for the **JangaFX real-time simulation suite**. Backed by
distilled references for each product plus the cross-cutting export pipeline.
JangaFX tools **simulate/author on the GPU in real time, then bake to standard
interchange files** — there is no live-link plugin, so the expert's real value is
(1) which tool + which parameters get the look, and (2) which export format lands
cleanly in *your* destination.

**Sibling skills — the other principal engineers.**
- **`graphics-api-expert`** owns *how it's drawn on the GPU* — WGSL/HLSL/MSL shaders, raymarching a VDB as a 3D texture, sub-UV/flipbook shaders, the render pipeline. When a JangaFX asset needs a custom shader to render (WebGPU volume raymarch, 6-way relight math), hand to `graphics-api-expert`.
- **`game-dev-expert`** owns *above the rendering line* — spawning/pooling the FX, Niagara/VFX-Graph system architecture, budgets, LOD. When the question is "how do I drive/spawn this effect in-game," hand to `game-dev-expert`.
- This skill owns *the JangaFX app and its export boundary*.

---

## First, three load-bearing truths (correct common wrong assumptions)

1. **IlluGen is NOT a lighting/illumination tool.** Despite the name, it's a **procedural 2D/3D VFX *asset* generator** (noise, flowmaps, masks, flipbooks, VFX meshes, VAT). Never recommend it for scene lighting/IES/GI. → `references/illugen.md`.
2. **No JangaFX tool ships a scripting/CLI/headless API today.** GUI-driven only. The one CLI is the floating-license server (`jfs`). EmberGen has a post-export shell hook (`$quit`) + User Variables; a scriptable API is roadmapped for EmberGen **2.0** only. Automate *downstream* over exported files, never the app. → `references/suite-overview.md` §3.
3. **GeoGen development is currently frozen** (team moved to IlluGen; still 0.5.x beta; docs are a placeholder; forum shut down). Fine for exploration, risky for a deadline-bound pipeline. → `references/geogen.md`.

Two more suite-wide constants: sims are **GPU-resident → VRAM is the hard ceiling** (single-GPU, no pooling); and the **60 Hz timeline-sync rule** (Timestep = Import FPS = Backplate FPS → Frame Stride = 1) is the #1 export gotcha across every tool.

---

## Which tool? (pick the simulator)

| You need… | Tool | Reference |
|---|---|---|
| Fire, smoke, explosions, magic wisps (gaseous volumetrics) | **EmberGen** | `embergen.md` |
| Liquids, splashes, pours, oceans-in-a-box, foam/spray/whitewater | **LiquiGen** | `liquigen.md` |
| Procedural terrain, heightmaps, whole planets, erosion | **GeoGen** *(dev frozen — see caveat)* | `geogen.md` |
| Noise/flowmaps/masks/beams, VFX meshes, flipbook repack, VAT authoring | **IlluGen** | `illugen.md` |
| Suite-wide: licensing, hardware, shared node graph, interop | — | `suite-overview.md` |

Gaseous vs liquid is the common fork: EmberGen ≠ liquids, LiquiGen ≠ gas. They're separate apps.

---

## Decision tree — route to the right reference fast

### "Which tool should I use / what is X?"
| Question | Go to |
|---|---|
| Which of the four apps fits my task? | table above |
| What is IlluGen actually for? | `illugen.md` §1 (it's asset gen, not lighting) |
| Licensing (Elemental Suite, floating server, activation) | `suite-overview.md` §5 |
| Hardware / GPU / VRAM requirements | `suite-overview.md` §6 · per-product §7/§2 |
| Shared node-graph / keyframes / modulators / randomization | `suite-overview.md` §2 |
| Is there a CLI / Python / headless mode? | `suite-overview.md` §3 (no — GUI + downstream) |

### EmberGen (fire/smoke/explosions)
| Task | Go to |
|---|---|
| Core sim workflow (emitter → sim → volume → render) | `embergen.md` §4 |
| Why is there no fire? / temperature couples ignition + buoyancy | `embergen.md` §5.1 |
| Simulation params (voxel res, upscaling, dissipation, vorticity, combustion) | `embergen.md` §5.1 |
| Sparks/embers/debris (particles) + injecting back into the grid | `embergen.md` §5.3 |
| Forces (curl noise, line, point, toroidal / smoke rings) | `embergen.md` §5.4 |
| VRAM blew up / sim too slow | `embergen.md` §8 (container vs voxel-size mental model) |

### LiquiGen (liquids/whitewater)
| Task | Go to |
|---|---|
| Core FLIP/particle sim workflow | `liquigen.md` §2 |
| Emitter (viscosity is PER-EMITTER), collider, drain | `liquigen.md` §3.1–3.2 |
| Whitewater / foam / spray / bubbles (HP + decay model) | `liquigen.md` §3.3 |
| Voxel size + what solver internals are/aren't documented | `liquigen.md` §4 |
| Meshing / surface reconstruction | `liquigen.md` §4.2 (flagged undocumented) |

### GeoGen (terrain)
| Task | Go to |
|---|---|
| Procedural terrain workflow (source → operators → erosion → mask → texture) | `geogen.md` §2 |
| Erosion (grid vs particle, mask extraction, deterministic Crumble) | `geogen.md` §3.2 |
| Heightmap/mesh/splat export + bit-depth caveats | `geogen.md` §4 |
| Should I build on GeoGen right now? | `geogen.md` top warning (dev frozen) |

### IlluGen (VFX asset gen)
| Task | Go to |
|---|---|
| What can it make / export | `illugen.md` §2–3 |
| VAT / caustics / optical-flow nodes | `illugen.md` §4 |
| Flipbook encode/decode round-trip | `illugen.md` §3 + `sim-as-conditioning.md` §4 |

### Export & integration (the payoff)
| Question | Go to |
|---|---|
| Which format for which destination? (matrix) | `export-pipeline.md` §1 |
| Sim → **WebGPU** (flipbook atlas, VDB→3D-texture, 6-way relight) | `export-pipeline.md` §2 |
| Sim → **Unreal** (Sparse Volume Textures, Niagara sub-UV, Landscape) | `export-pipeline.md` §5 |
| Sim → **Blender/Houdini** (VDB, Alembic, velocity for mblur) | `export-pipeline.md` §4 |
| Sim → **ComfyUI / NeuralFX** (depth/normal/MV/alpha as control) | `sim-as-conditioning.md` |
| VDB export details (channels, coord system, transform pin) | `embergen.md` §6.3 |
| Alembic / VAT / mesh-flipbook details | `liquigen.md` §5.1 · `illugen.md` §3 |
| 6-way lighting flipbooks | `embergen.md` §6.1–6.2 · `export-pipeline.md` §2.3 |
| Export gotchas (network drive, timeline sync, missing channels) | `export-pipeline.md` §7 |

---

## How to answer well (working method)

1. **Confirm gaseous vs liquid vs terrain vs asset-gen first** — that picks the app and the reference. Don't answer an EmberGen question with LiquiGen params.
2. **Lead with the destination.** "Elevate our particle effects" almost always means an *export* question. Ask/confirm the target (WebGPU / Unreal / ComfyUI / Blender) and route through `export-pipeline.md` — the sim is only half the job.
3. **Zero-guessing.** JangaFX docs deliberately omit solver internals (LiquiGen sub-steps/CFL/meshing, VRAM floors, several node param lists) and IlluGen/GeoGen docs are placeholders. When a reference flags something as undocumented, say so and point to the in-app tooltip / roadmap — do **not** invent a parameter name or value. The references mark `[non-docs]` / ⚠️ / "flag" exactly so you don't have to guess.
4. **Respect the pipeline reality.** No live-link, no render CLI, VRAM ceiling, 60 Hz timeline sync. Design pipelines around baked files + downstream automation.
5. **Tag in the siblings** when the question crosses the line (shader authoring → `graphics-api-expert`; in-engine spawning/architecture → `game-dev-expert`).

---

## Reference files in this skill

- `references/embergen.md` — EmberGen: gaseous sim workflow, params, VDB/flipbook/particle export, perf, gotchas.
- `references/liquigen.md` — LiquiGen: FLIP liquid sim, whitewater, meshing, Alembic/VAT/VDB/image export.
- `references/geogen.md` — GeoGen: procedural terrain/planets, erosion, heightmap/mesh export. **Dev-frozen caveat up top.**
- `references/illugen.md` — IlluGen: procedural VFX *asset* generation (NOT lighting), VAT, flipbook round-trip.
- `references/suite-overview.md` — shared node graph, licensing/floating server, hardware, interop, scripting reality, resources.
- `references/export-pipeline.md` — format → destination matrix; WebGPU, Unreal (SVT/Niagara/Landscape), Blender, ComfyUI.
- `references/sim-as-conditioning.md` — using sim passes (depth/normal/motion-vector/alpha) as generative control for NeuralFX/ComfyUI.
- `references/external-resources.md` — verified source URLs (docs, product, roadmap, learning).

Enrich this skill from a new source (docs update, tutorial, release) with the sibling **`ingest-jangafx-source`** skill — it runs the defensive multi-agent research → verify → wire-in pipeline.
