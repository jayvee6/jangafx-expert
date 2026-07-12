# GeoGen — Expert Reference (JangaFX Real-Time Procedural Terrain / Planet Generator)

> **READ FIRST — two status warnings.**
> 1. **Development is currently frozen.** JangaFX publicly stated GeoGen dev is paused ("for the time being, except macOS"); the team was reassigned to IlluGen after GeoGen underperformed in sales, with intent to resume "when their financial position improves." GeoGen is still **pre-1.0 beta** (0.5.x). This is the single biggest adoption risk — do not put a deadline-bound pipeline solely on GeoGen. Source: https://jangafx.com/roadmap/geogen
> 2. **Docs are a placeholder** (https://docs.jangafx.com/geogen/ = "still work in progress"; subpages 404) and the **JangaFX forum has been shut down**. Below is built from JangaFX product/roadmap pages + detailed release coverage (CG Channel, 80.lv, Animation Magazine), with every non-first-party claim flagged. **Exact node/parameter UI strings and numeric ranges are largely undocumented publicly** — verify in the app's node browser / preset files.

---

## 1. What GeoGen is, and when to reach for it

GeoGen is JangaFX's **real-time, GPU-accelerated, node-based terrain and whole-planet generator** — repeatedly described as **"Substance 3D Designer, but for terrain."** A non-destructive node graph drives heightmaps you detail with geometry, color, masks and erosion, then export as game-ready assets.

### Reach for it when…
| Scenario | Why GeoGen fits |
|---|---|
| Procedural, non-destructive heightmap terrain for an engine | Node graph + game-oriented exports (heightmaps, meshes, masks) |
| **GPU-fast erosion iteration** | Erosion/weathering GPU-accelerated, "compute quickly" — real-time tweaking |
| **Entire 3D planets**, not just tiles | Planet mode w/ octahedral/equirectangular heightmap export + atmosphere |
| Already in the JangaFX ecosystem | Shared UI/node muscle memory |
| Lightweight single-window alt to World Machine / Gaea / Houdini terrain | Fast viewport, preset-driven |

### Look elsewhere when…
- You need a **stable, documented, actively-developed** tool today → dev is frozen (see warning).
- You need **scripting/automation** → none documented (§5).
- You need **guaranteed export bit-depths/containers** → loosely documented; verify in-app (§4).
- You need **vendor docs / forum support** → docs WIP, forum gone; support = Discord + support@jangafx.com.

Version history: 0.2 alpha (Feb 2024) → public beta (Jul 2024) → 0.5.0 (Jul 2025, +macOS) → 0.5.2 (Dec 2025) → 0.5.3 latest `[third-party]`.

---

## 2. Core procedural workflow (mirrors Substance Designer)

1. **Source the base heightmap** — procedural noise, primitives, or **imported images/meshes as resources**.
2. **Shape with heightmap operators & modifiers** — Terrace, Faulting, Craters, Ridge & Bulge, Crystallizer.
3. **Simulate erosion/weathering** — grid + particle erosion, Fluvial filter/mask, Crumble (§3).
4. **Mask by terrain properties** — Height, Curvature, Occlusion, Slope, Shadow; blend; spline masks for rivers/roads/canyons.
5. **Texture / color** — gradients driven by elevation/slope → color maps / splat maps / texture masks.
6. **Organize with Subgraphs** (custom node groups).
7. **Preview / render** — real-time viewport: raymarched, lightmapped, or **path-traced**; atmosphere, clouds, water, DOF.
8. **Export** via dedicated export nodes (§4).

**Learning path (docs absent):** GeoGen ships **presets that double as tutorials** — deconstruct their node trees; plus the JangaFX YouTube "GeoGen Free Lessons" playlist.

Traits: non-destructive graph, node re-routing, **multi-resolution** erosion, curve/gradient presets, node-search dropdown.

---

## 3. Nodes & parameters that matter

> **Verbatim caveat:** names below are the ones JangaFX/coverage use in prose; only a subset are confirmed exact UI labels, and ranges/params are not publicly documented. Inspect the node browser / preset `.json` in-app.

### 3.1 Node categories (feature-level)
| Category | Purpose | Named examples |
|---|---|---|
| **Sourcing** | Base signal | procedural noises; primitives; image/mesh import |
| **Heightmap operators** | Shape structure | **Terrace, Faulting, Craters, Crystallizer, Ridge & Bulge** (verbatim) |
| **Modifiers** | Transform heightfield | (category named; specifics undocumented) |
| **Filters** | Post-process field | **Fluvial** filter; erosion filters |
| **Masks** | Selection by property | **Height, Curvature, Occlusion, Slope, Shadow** (verbatim); Fluvial mask; spline masks |
| **Erosion / weathering** | Physical detail | **Crumble** node; grid + particle erosion |
| **Blend ops** | Combine | Substance-like blends |
| **Subgraphs** | Organize | custom node groups |
| **Export** | Emit assets | §4 |

### 3.2 Erosion — the headline feature
- **Two solver families:** **grid-based** (hydraulic field solver) and **particle-based** (droplet sim).
- **Multi-resolution:** works at multiple resolutions at once — resolve large channels and fine detail together.
- **Mask extraction:** erosion emits usable masks (deposition vs bedrock, flow) — route these into texturing rather than hand-painting. **This is the crucial bit for splat/material work.**
- **Fluvial filter & mask:** river/water-flow erosion + matching mask.
- **Crumble node:** crumbling rock / talus; **reworked to be deterministic** (same inputs → same result) — reproducible exports.
- **Wave phase parameter** (verbatim) animates water/waves; export nodes can emit **image sequences** so time-varying sims bake frame-by-frame.

> **Practical loop (inference, flagged):** erosion is GPU-fast and Crumble is deterministic, so the intended loop is *tweak → watch live → extract masks → texture*. Determinism means the exported heightmap matches your preview — an edge over stochastic CPU erosion tools.

### 3.3 Masks & texturing
Masks from terrain properties (Height/Slope/Curvature/Occlusion/Shadow) **and** erosion output; combine via blends; feed color maps / texture masks / **splat maps**. **Spline masks** carve rivers/roads/canyons. Texturing is gradient/elevation/slope-driven (Substance-style) → per-layer masks for engine material blending.

---

## 4. Export pipeline (engines + DCC)

GeoGen's reason to exist: "game-ready data types" that "plug into Unity, Unreal Engine, Blender, or custom pipelines."

### 4.1 What you can export
| Domain | Outputs |
|---|---|
| **Terrain** | Meshes, heightmaps, normal maps, color maps, texture masks, **splat maps**, water planes |
| **Planet** | Planet mesh, **octahedral OR equirectangular** heightmaps, texture masks, color maps |
| **Utility** | Masks (incl. erosion-extracted), utility maps |

### 4.2 Formats & resolution (what's actually stated)
| Aspect | Value | Confidence |
|---|---|---|
| **Mesh formats** | **OBJ or FBX** | stated in coverage; verify in-app |
| **Texture/map cap** | up to **16K** (newer); 8192² cited at 0.2 | 16K is the newer figure |
| **Heightmap bit depth** | not published; third-party assumes **16-bit** | ⚠️ UNCONFIRMED |
| **Heightmap container** (PNG/EXR/RAW/R16) | **not confirmed by any first-party source** | ⚠️ UNCONFIRMED / may need conversion |
| **Image sequences** | all export nodes can emit them (0.2+) | stated |
| **Multi-node export** | export multiple nodes at once; export viewport state (0.2+) | stated |

> **Zero-guessing:** no first-party statement enumerates heightmap containers/bit depth. Only **OBJ/FBX** meshes are explicitly named. Verify the export-node UI; budget a **Krita/Photoshop 16-bit conversion** if your engine needs a specific container. FBX/Alembic animated meshes and RAW/R16 heightmaps are **not confirmed**.

### 4.3 Export → Unreal (Landscape)
- **Roadmap:** *"Export large tiled terrains to unreal engines with just a few clicks"* — one-click **tiled** Unreal export announced but **marked delayed/frozen**. ⚠️ May be incomplete — verify in your build; keep the manual path as fallback.
- **Manual path:** export a **16-bit grayscale heightmap** → Unreal *Landscape mode → Import from File*. Large worlds → **World Partition** (set Grid Size + Region Size, select tiled heightmap, Import Type = Subregion, Mode = All).
  - **Z-scale formula:** `Z scale = (max height m) × 100 × 0.001953125`.
  - Unreal accepts **16-bit grayscale `.png`** or **`.r16`**.
- **Tiling caveat:** Unreal breaks past int32 vertex limits; oversized maps must be tiled, and mismatched tile sizes cause crashes / flat-plane imports (general tiled-heightmap pitfall).

### 4.4 Export → Unity (Terrain)
Named as supported; no GeoGen-specific steps documented. Practical: export **16-bit heightmap** → Unity Terrain (uses RAW 16-bit), use exported splat/texture masks for Terrain layers. ⚠️ Exact steps undocumented.

### 4.5 Export → DCC (Blender / Maya / Houdini)
- **Meshes:** OBJ/FBX straight into Blender or any DCC (Blender named explicitly).
- **Maps:** heightmaps/normals/color/masks as textures for shading or displacement.
- Planet meshes + octahedral/equirectangular maps for spherical workflows.

### 4.6 Interop with other JangaFX tools
No first-party documented direct bridge (e.g. GeoGen→EmberGen). Shared traits are UI/licensing, not a data bridge. ⚠️ Assume **file-based interop** (export mesh/map, import elsewhere). GeoGen terrain as an EmberGen collider/emission surface is a plausible file-based workflow (FBX/OBJ import) but not a documented feature.

---

## 5. Automation / CLI / scripting

**No publicly documented scripting API, Python interface, or headless/CLI mode.** Searching returned nothing first-party.

| Capability | Status |
|---|---|
| Python / scripting API | not documented / assume absent ⚠️ |
| Headless render / batch CLI | not documented / assume absent ⚠️ |
| Command-line export | not documented ⚠️ |
| In-app batch export | **Yes (GUI):** multi-node export + image-sequence export (0.2+) |

Plan around **file-based GUI export** + downstream engine/DCC scripting. Per Zero-Guessing: do not assume a GeoGen scripting surface exists.

---

## 6. Performance, hardware & gotchas

### 6.1 Platform & hardware (first-party via CG Channel 0.5)
| Requirement | Value |
|---|---|
| **OS** | Windows 10+, **RHEL 9+** Linux, **macOS 15 (Sequoia)+** |
| **Min GPU** | GTX 1060 / RX 580 / **Apple M1 Pro** |
| **macOS chips** | **Apple Silicon Pro/Max/Ultra only** — base M1/M2/M3/M4 **NOT supported** |
| RAM | 8 GB min / 16 GB rec `[third-party]` |
| VRAM | no official min; treat **≥6–8 GB** as a floor for large terrains ⚠️ inferred |
| Graphics API | not officially stated ⚠️ |

### 6.2 Performance
GPU-accelerated backend; erosion computes quickly; multi-resolution erosion previews cheap then resolves detail. Dedicated GPU needed for a smooth viewport. Large exports (16K textures, big tiled terrains) are memory-bound — VRAM/RAM ceiling governs max terrain size.

### 6.3 Gotchas (prioritized)
1. **Dev frozen / beta** — biggest risk; no committed resume timeline.
2. **Docs placeholder, forum gone** — learn from presets + YouTube; support Discord/email only.
3. **One-click tiled Unreal export is roadmap-delayed** — test before relying; keep manual World-Partition fallback.
4. **Heightmap format/bit-depth unconfirmed** — verify 16-bit + engine-acceptable container; budget a conversion step.
5. **"Black heightmap on export" class** `[reported pre-shutdown]` — standard cause is value-range/normalization: export **16-bit**, confirm height range/normalization on the export node, inspect in a 16-bit-aware viewer.
6. **macOS base-chip exclusion** — Pro/Max/Ultra only.
7. **Unreal tiling math** — respect int32 vertex limits; mismatched tile resolution → flat plane/crash.
8. **Determinism only where stated** — Crumble is deterministic; don't assume every stochastic node is frame-stable across versions (roadmap notes backward-compat-affecting bugfixes). Re-verify graphs after updates.

---

## 7. Licensing (for planning, `[third-party]`, subject to change)
Indie subscription (<$1M/yr): **$9.99/mo** → perpetual after 18 months. Indie perpetual **$149.99**. Studio node-locked **$1,400**; Studio floating **$2,300**. Free trial. *(0.2-era coverage cited different figures; confirm on the buy page.)* Covered by the Elemental Suite the user holds — see `suite-overview.md` §5.

---

## 8. Quick reference
| Question | Answer |
|---|---|
| Category | Real-time node-based terrain + planet generator ("Substance for terrain") |
| Core loop | Source → operators → erosion → masks → texture → export |
| Erosion | grid + particle, multi-res, mask extraction; Fluvial; deterministic Crumble |
| Masks | Height, Curvature, Occlusion, Slope, Shadow, Fluvial, spline |
| Operators | Terrace, Faulting, Craters, Crystallizer, Ridge & Bulge |
| Mesh export | OBJ, FBX |
| Map export | heightmaps, normals, color, texture masks, splat maps, water planes; ≤16K; image sequences |
| Planet export | planet mesh + octahedral / equirectangular heightmaps |
| Engines | Unreal (tiled export roadmapped/delayed; manual World-Partition works), Unity, Blender, custom |
| Scripting/CLI | none documented — assume absent |
| Min GPU | GTX 1060 / RX 580 / M1 Pro; Win10+/RHEL9+/macOS15+ |
| Biggest caveat | **development frozen; docs WIP; forum shut down** |

---

### Primary sources
- Product — https://jangafx.com/software/geogen · Roadmap — https://jangafx.com/roadmap/geogen · Docs (placeholder) — https://docs.jangafx.com/geogen/
- CG Channel 0.2 — https://www.cgchannel.com/2024/02/jangafx-releases-geogen-in-public-alpha/ · 0.5 — https://www.cgchannel.com/2025/07/jangafx-releases-geogen-0-5/
- 80.lv 0.2 — https://80.lv/articles/jangafx-released-geogen-0-2 · Animation Magazine review — https://www.animationmagazine.net/2024/05/tech-reviews-geogen-liquigen-from-jangafx-tyflow-for-3ds-max/
