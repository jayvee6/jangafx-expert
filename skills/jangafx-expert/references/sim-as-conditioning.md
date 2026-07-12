# Sim-as-Conditioning — JangaFX outputs as control signals for NeuralFX / generative video

> How to use JangaFX (mainly EmberGen + LiquiGen) as a **controllable driving/conditioning generator** for your ComfyUI/NeuralFX pipeline, instead of scraping footage. The JangaFX-side facts (which passes exist) are documented in `embergen.md` §6.1 / `liquigen.md` §5.4. The generative-side mapping below is **[established practice]** for ControlNet/V2V pipelines + **[your-engineering]** for how it wires into your specific graphs. Ties into your `comfyui-video-restyle` skill and SCAIL neural-VFX method.

---

## 1. Why sims beat scraped footage here

A JangaFX render gives you, **for free and perfectly registered per-frame**, the exact conditioning signals diffusion pipelines want:
- **clean alpha/masks** (no rotoscoping),
- **true depth** (no MiDaS estimation error),
- **true normals** (no normal-from-depth artifacts),
- **true motion vectors** (no optical-flow estimation noise),
- **art-directable motion** (you keyframe the sim; you own the timing).

That means the structure/motion is *ground truth*, so the AI only has to do what it's good at — style/texture/detail — while the sim locks composition and temporal coherence. This is the core reason to route generative FX through JangaFX rather than reference clips.

---

## 2. The passes → conditioning map

Export EmberGen/LiquiGen **Render node → Sequence, format EXR** (16/32-bit; linear). Per-pass roles:

| JangaFX pass | Generative role | Notes |
|---|---|---|
| **Render All / beauty** | V2V driving video (img2img/AnimateDiff init) | the thing being restyled |
| **Alpha / Smoke Mask / Flames Mask** | inpaint mask / matte / region control | isolate the FX from background; composite later |
| **Depth** | Depth ControlNet | locks 3D structure; strongest single control for volumes |
| **Six Point Normal Map** / **Gradient Based Normal Map** | Normal ControlNet | locks surface orientation / lighting-consistent detail |
| **Motion Vector** | temporal consistency / optical-flow guidance | feed flow-guided nodes; reduces boil/flicker frame-to-frame |
| **Albedo** | color/base conditioning | keep palette continuity |
| **Temperature** | drive emissive/color ramps in post or as a mask | hot-core vs cool-edge control |
| **LiquiGen Thickness/Reflect/Refract** | refraction/relative-depth cues for liquids | cheap glassiness control |

**Rule of thumb:** Depth + Motion Vector is the minimum viable control pair for a coherent restyle; add Normal for lighting-locked detail; add masks for clean compositing.

---

## 3. Reference wiring (ComfyUI)

**[your-engineering — adapt to your graphs]**

1. **Sim in EmberGen/LiquiGen** — author the motion you want; keyframe timing to match your shot. Export EXR sequence with beauty + Depth + Motion Vector + Normal + Alpha.
2. **Load sequences** in ComfyUI (image-batch loaders). Keep them frame-aligned — same First Frame / Frame Stride across passes (they already are if exported together).
3. **Restyle** via your AnimateDiff/V2V or SCAIL restyle graph, with:
   - Depth + Normal → ControlNet stack (structure lock),
   - Motion Vector → flow-guided / temporal node (coherence),
   - Alpha → inpaint/matte so the background stays yours.
4. **Composite** the restyled FX back over your plate using the sim's alpha.
5. **Repack for the engine** via **IlluGen flipbook encode** → RGBA sub-UV atlas → WebGPU/Unreal. This closes sim → AI → game-asset.

Cross-ref: `comfyui-video-restyle` skill (temporal consistency), `authoring-comfyui-workflows` (API-format graph authoring), SCAIL method (memory: `reference_scail_neural_vfx`).

---

## 4. Why the round-trip matters (IlluGen)

IlluGen's **flipbook encode/decode** is the piece that makes this production-viable: it *"import[s] simulations created in applications like EmberGen or Houdini as flipbooks, modif[ies] them… then export[s] the new flipbooks to game engines."* So the pipeline is:

```
EmberGen sim → EXR passes → ComfyUI restyle → image sequence
   → IlluGen (encode to RGBA packed flipbook, pack channels, handle alpha)
   → Unreal Niagara / Unity VFX Graph / your WebGPU atlas shader
```

Without IlluGen you'd hand-pack the atlas; with it, channel packing + alpha handling + flipbook assembly is a graph.

---

## 5. Gotchas

- **Export EXR, not PNG, for control passes** — Depth/Motion Vector/Normal need float precision; 8-bit PNG will band and corrupt the signal.
- **Motion Vector convention** — confirm the pass's vector encoding (screen-space px vs normalized, R/G channel meaning) before feeding a flow node; a flipped or mis-scaled MV makes coherence worse, not better. ⚠️ verify against your flow node's expected format.
- **Keep passes frame-locked** — export all passes from the same Render node in one go so First Frame / Stride match; a one-frame offset between depth and beauty desyncs the conditioning.
- **Alpha vs mask** — EmberGen has both an `Alpha` pass and `Smoke Mask`/`Flames Mask`; pick the one that isolates what you're restyling.
- **Determinism** — re-simulating with the same seed/params reproduces the drive exactly, so you can iterate the AI style without the motion drifting. Lock the sim (`Increment and Save`) before a long restyle run.
- **This is a file pipeline** — no live link; script the *ComfyUI* side, not JangaFX (no render CLI).
