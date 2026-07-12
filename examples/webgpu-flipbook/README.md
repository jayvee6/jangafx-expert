# WebGPU flipbook — the EmberGen → WebGPU consume path

A self-contained, single-file WebGPU demo of how a JangaFX (EmberGen/LiquiGen)
**flipbook + motion-vector export** is consumed in a WebGPU renderer — the
technique described in
[`export-pipeline.md` §2.1](../../skills/jangafx-expert/references/export-pipeline.md).

Because JangaFX can't run on macOS/here, the demo **bakes** a procedural
explosion into the two atlases EmberGen would export (a `color` sub-UV sheet and
a `motionVector` sheet), then **consumes** them exactly as you would a real
export:

- **Left half** — naïve sub-UV: hard frame stepping (what you get if you just
  advance the atlas cell). Watch it strobe at low fps.
- **Right half** — motion-vector–warped interpolation: sample frame *N* and
  *N+1*, warp each by the MV pass, cross-fade. Smooth motion from 64 stored
  frames.

Controls: playback fps (drop it low to expose stepping), MV warp strength,
additive/translucent blend, an MV-atlas debug view, and a split-screen toggle.

## Run

```bash
cd examples/webgpu-flipbook
python3 -m http.server 5177
# open http://localhost:5177/ in Chrome/Edge 113+ or Safari 18+ (WebGPU)
```

WebGPU needs a secure context — serve over `localhost` (not `file://`).

## What maps to a real EmberGen export

| Demo | Real pipeline |
|---|---|
| baked `color` atlas (8×8=64) | `Render → Flipbook`, Columns×Rows=8×8, PNG/EXR |
| baked `motionVector` atlas | the **Motion Vector** render pass, same frame range |
| MV encode/decode (`ENC=8`) | ⚠️ verify EmberGen's MV convention (screen-space px vs normalized, channel meaning) before trusting it — a flipped/mis-scaled MV makes playback *worse* |
| additive blend over dark bg | emissive fire composite (don't composite additive over white) |

The actual WGSL sub-UV + MV-warp shader is `graphics-api-expert` territory;
this example is the reference implementation of that consume step.

`window.__demo` is exposed (`tick(ms)`, `state`, `frameAt`) for headless
verification.
