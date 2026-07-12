# IlluGen — Expert Reference (JangaFX Procedural VFX Asset Generator)

> **READ THIS FIRST — the name is misleading.** Despite the "Illu-" prefix, IlluGen is **NOT** a lighting / illumination / IES / GI-baking tool. It is a **node-based procedural asset-generation tool for real-time VFX and tech artists** — it authors 2D textures and 3D VFX meshes (noise, flowmaps, masks, flipbooks, beams, shockwaves, VAT) for game engines. Think "Substance Designer + a bit of Houdini + After Effects, for real-time VFX assets," not a renderer's light rig.
>
> **Documentation status:** the official docs at https://docs.jangafx.com/illugen/ are a **placeholder** — *"Documentation for IlluGen is still work in progress."* Everything below is sourced from JangaFX's product page, public roadmap, and launch press, **not** a formal reference manual. Treat node/parameter specifics as provisional and verify against the in-app UI. Per zero-guessing: do **not** assume EmberGen/LiquiGen node semantics carry into IlluGen 1:1.

---

## 1. When to reach for IlluGen

Reach for IlluGen when you need to **author the 2D/3D building blocks of a real-time VFX effect in one graph** instead of bouncing between Photoshop, Substance Designer, Houdini, and After Effects.

| Use case | Why IlluGen fits |
|---|---|
| Tiling noise / channel-packed noise textures | Native procedural noise + RGBA packing |
| Flowmaps / UV-distortion maps | Dedicated flowmap + optical-flow (motion-vector) nodes |
| Gradient masks, beams, shockwaves | Purpose-built VFX primitives |
| Custom **RGBA packed flipbooks** | Flipbook packing/unpacking with alpha handling |
| VFX-specific / pivot-baked 3D meshes | Mesh authoring + pivot painting for shader-driven motion |
| **Editing an EmberGen/Houdini sim as a flipbook** | Flipbook **encode/decode** round-trip (import sim → tweak → re-export) |
| Converting/repacking assets for UE/Unity/Godot | File-based export in standard real-time formats |

**Its own example (verbatim):** for a fiery sword-swipe you need an arc mesh, an edge-softening mask, and panning noise. Normally that's three tools; *"with IlluGen you can create the 3D mesh, masks, and noises in one singular graph… author and preview how all assets interact before putting them in your game."*
Source: https://jangafx.com/software/illugen

---

## 2. What it produces (asset types)

Tiling noise · flowmaps · VFX meshes · UV distortions · pivot-baked meshes · gradient masks · custom RGBA packed flipbooks · channel-packed noises · color gradients · beam textures · shockwaves · masks. Roadmap/press also list normal maps, caustics, and 3D FX meshes.
Source: https://jangafx.com/software/illugen

---

## 3. Export formats

| Export | Notes |
|---|---|
| 2D textures (square **and non-square**) | Baked textures |
| Flipbooks | Full-effect flipbook animation; **encode and decode** (round-trip) |
| Custom packed atlases | RGBA channel packing |
| Flowmaps / UV-distortion maps | Panning/flow + distortion |
| 3D meshes | Conventional meshes |
| **Mesh flipbooks** | Geometry of each animation frame, separated by **vertex color** |
| **VAT (Vertex Animated Textures)** | positions/normals/tangents/colors via `VAT Encode`; playback via `VAT Decode` |
| Pivot-baked meshes | Pivot painting for shader-driven motion |
| Alembic caches | Standard VFX interchange |

**Downstream (verbatim):** *"Exports support the standard VFX formats: Alembic caches, flipbooks, and baked textures, ready for Unreal, Unity, Godot, or in-house engines. Direct pipeline links to EmberGen, LiquiGen, and future JangaFX products are included by design."* The pipeline is **file export → import in engine**, not a live-link runtime plugin.
Source: https://www.cgchannel.com/2025/07/jangafx-releases-illugen-1-0/ , https://jangafx.com/roadmap/illugen

**Round-trip flipbook (verbatim):** IlluGen can *"import simulations created in applications like EmberGen or Houdini as flipbooks, modify them in IlluGen, then export the new flipbooks to game engines."* → This is the key EmberGen↔IlluGen bridge for your pipeline.

---

## 4. Node / workflow model (what's publicly known)

- A **procedural node graph** that "looks like a shader editor," marrying 2D and 3D asset authoring in one graph with live preview.
- Includes a **full animation timeline**, flipbook packing/unpacking, and easy alpha-channel handling.
- **Named nodes disclosed so far** (roadmap/press, not a manual):
  - `VAT Encode` / `VAT Decode` — bake and play back vertex-animated textures.
  - `Caustics Node` — *"simulate complex light interactions via a 2D path tracing algorithm"* for lightning, stylized effects, or tiling caustics.
  - `Optical Flow Node` — *"generates the optical flow map… standard motion vector outputs, flow pin, and dual flow outputs"* for any animated input.

> **Zero-guessing flag:** the exact node list, per-node pin schema, and parameter reference are **not documented** anywhere official. The four nodes above are the only ones JangaFX has published names for.

---

## 5. Release status & roadmap

| Fact | Detail |
|---|---|
| 1.0 public release | **July 28, 2025** |
| Current line | 1.1.x (floating-license guide references IlluGen **1.1.2+**) |
| Platforms | Windows, Mac, Linux |
| Trial | 14-day full-function demo, **minus exporting** |
| Next major | **1.2** (2026) — engine **shader workflow** IlluGen → Unreal/Unity/Godot: a two-stage *asset-creation → shader-manipulation* pipeline exporting both assets and shaders; 3D particles convertible to 2D texture projections, exportable as **JSON, VAT, Alembic** |

Live changelog / feature hub: https://jangafx.com/roadmap/illugen
Sources: https://www.cgchannel.com/2025/07/jangafx-releases-illugen-1-0/ , https://80.lv/articles/the-first-look-at-illugen-jangafx-s-new-tool-for-vfx-in-games-tech-art

---

## 6. Gotchas

- **Not a lighting tool** — do not use it (or recommend it) for scene illumination, IES, or GI baking. It makes VFX *assets*.
- **Docs are a placeholder** — cite the product/roadmap pages, and verify node/parameter details in-app before asserting them.
- **Export is disabled in the trial** — full evaluation of the output pipeline needs a license (the user has the Elemental Suite, so this is moot for them).
- **Flipbook encode/decode is the interop superpower** — for your ComfyUI/NeuralFX work, IlluGen is the natural place to repack EmberGen sims into engine-ready RGBA flipbook atlases. See `export-pipeline.md` and `sim-as-conditioning.md`.

---

### Source URLs
- Product page — https://jangafx.com/software/illugen
- Roadmap / changelog — https://jangafx.com/roadmap/illugen
- Docs (placeholder) — https://docs.jangafx.com/illugen/
- 1.0 launch — https://www.cgchannel.com/2025/07/jangafx-releases-illugen-1-0/ , https://80.lv/articles/jangafx-s-illugen-is-out-now
- First look — https://80.lv/articles/the-first-look-at-illugen-jangafx-s-new-tool-for-vfx-in-games-tech-art
- RealTimeVFX thread — https://realtimevfx.com/t/illugen-by-jangafx-our-new-vfx-asset-generation-software-3d-meshes-2d-textures/29403
