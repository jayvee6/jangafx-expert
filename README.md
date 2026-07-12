# jangafx-expert

A Claude skill plugin providing principal-level advice for the **JangaFX
real-time simulation suite** — and, critically, how to get its output into your
pipeline (Unreal, Blender, WebGPU, ComfyUI/NeuralFX). JangaFX tools simulate on
the GPU in real time and **bake to standard interchange files** (there's no
live-link plugin), so the skill's value is *which tool + which parameters get the
look* and *which export format lands cleanly downstream*.

## The suite

| Tool | What it is | Docs status |
|---|---|---|
| **EmberGen** | Real-time gaseous fluid sim — fire, smoke, explosions | Full |
| **LiquiGen** | Real-time liquid sim — FLIP liquids, whitewater, foam, spray | Full |
| **GeoGen** | Procedural terrain & planet generator, GPU erosion | ⚠️ placeholder; **dev frozen** (0.5.x beta) |
| **IlluGen** | Procedural **VFX asset** generator (noise/flowmaps/masks/flipbooks/VAT) — **not a lighting tool** | ⚠️ placeholder |

## Three things the skill exists to correct

1. **IlluGen is not a lighting/illumination tool** — it's procedural VFX asset generation.
2. **No JangaFX tool ships a scripting/CLI/headless API today** — GUI-driven; automate downstream over exported files.
3. **GeoGen development is currently frozen** — usable for exploration, risky for a deadline-bound pipeline.

## Skills in this plugin

- **`jangafx-expert`** — the advisor. Router `SKILL.md` → per-product + cross-cutting references.
- **`ingest-jangafx-source`** — defensive multi-agent pipeline to enrich the expert from a new source (docs update, tutorial, release), with zero-guessing verification and a human gate before any commit/push.

## Reference files (`skills/jangafx-expert/references/`)

- `embergen.md` — gaseous sim workflow, params, VDB/flipbook/particle export, perf.
- `liquigen.md` — FLIP liquid sim, whitewater, meshing, Alembic/VAT/VDB/image export.
- `geogen.md` — procedural terrain/planets, erosion, heightmap export (+ dev-frozen caveat).
- `illugen.md` — procedural VFX asset generation, VAT, flipbook round-trip.
- `suite-overview.md` — shared node graph, licensing/floating server, hardware, interop, scripting reality.
- `export-pipeline.md` — format → destination matrix: WebGPU, Unreal (SVT/Niagara/Landscape), Blender, ComfyUI.
- `sim-as-conditioning.md` — sim passes (depth/normal/motion-vector/alpha) as generative control for NeuralFX/ComfyUI.
- `external-resources.md` — verified source URLs.

## Sibling experts

- `graphics-api-expert` — GPU/shader side (raymarching a VDB, sub-UV/flipbook shaders, 6-way relight math).
- `game-dev-expert` — above the rendering line (spawning/pooling FX, Niagara/VFX-Graph system architecture, budgets).

## Sources

Built from the official docs at [docs.jangafx.com](https://docs.jangafx.com/),
JangaFX product/roadmap pages, and release coverage — every claim sourced, with
undocumented items explicitly flagged rather than guessed. See
`external-resources.md`.

## License

MIT
