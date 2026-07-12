---
name: ingest-jangafx-source
description: Productized pipeline to feed a new external source (a JangaFX docs section, a YouTube tutorial/tutorial series, a community node reference, a release/changelog, a forum/Discord thread export, or a blog post) into the jangafx-expert skill. Use whenever the user wants to "feed jangafx-expert", "ingest/absorb <url> into the jangafx expert", "add this EmberGen/LiquiGen/GeoGen/IlluGen source to the skill", "pull what's useful from <this JangaFX tutorial/docs page> into jangafx-expert", or hands over a JangaFX-related link/video/author and wants the expert to get smarter from it. This is the repeatable form of the manual multi-agent research → distill → wire-in run; invoke it instead of improvising an ad-hoc research run whenever the goal is enriching jangafx-expert from an external source.
---

# ingest-jangafx-source

Turn one external JangaFX source into durable jangafx-expert knowledge, via a
defensive multi-agent pipeline. This is the productized form of the run that
first built the skill from the official docs at `docs.jangafx.com`.

**Input (`<source>`):** a JangaFX docs section
(`https://docs.jangafx.com/embergen/`), a YouTube tutorial or series (JangaFX's
own channel or a community teacher), a release notes / changelog page, a
community node reference or wiki, or a forum/Discord thread. Multiple related
sources in one run are fine (treat as one corpus).

**Output:** verified entries appended to jangafx-expert's
`references/external-resources.md`, optionally a distilled `references/<topic>.md`
patterns file (or an update to an existing product reference —
`embergen.md` / `liquigen.md` / `geogen.md` / `illugen.md` /
`export-pipeline.md` / `sim-as-conditioning.md`), the decision tree in
`SKILL.md` wired to reach it, a local commit, and a daily-note closeout.
Nothing is pushed without explicit human approval.

**Canonical target — non-negotiable.** jangafx-expert's authoritative copy is
the **standalone repo** `~/Documents/Development/jangafx-expert` (git remote
`github.com/jayvee6/jangafx-expert`); the `~/.claude/skills/jangafx-expert`
path symlinks into it. All edits land in the standalone. Any second clone under
`~/.claude/plugins/` is a secondary mirror — do not edit it; `git pull` it if it
drifts.

---

## Core contracts (baked in — do not skip)

These come from the user's `best_practices.md`; this pipeline exists to apply
them every time without re-deriving them:

- **Zero-Guessing.** Never invent a node name, parameter, export format, menu
  path, URL, or "this feature probably exists." JangaFX's solver internals are
  partially undocumented (LiquiGen sub-steps/CFL, meshing controls) — agents
  report only what a page/video actually states, and flag the rest as
  UNVERIFIED. A candidate in a prompt is a hypothesis, not a fact. The docs
  themselves omit things (e.g. no published minimum-VRAM figure, no scripting
  API for several tools) — record those *absences* as findings, never fill
  them with plausible values.
- **Dispatch manifest + fan-out ≤ 5.** Record every research task before
  dispatch. Never exceed 5 concurrent agents.
- **Arbiter fan-in.** Reconcile manifest vs. completions. Validate every agent
  report's schema before consuming it. The orchestrator (you, the caller) is
  the Arbiter — agents are read-only Workers; their raw output is an
  intermediate artifact, never the deliverable.
- **Human gate before write, human gate before push.** The integration-depth
  choice is the user's. The commit is local; the push is a separate, explicit
  approval. Verify fast-forward; never force-push.
- **No eval harness for this class.** This is the user's validated
  flexible/append path. The acceptance test is the human reviewing the real
  output, not synthetic evals.

---

## Pipeline

Run these phases in order. Each phase has an explicit gate; do not advance on
elapsed time or assumption. Use `TodoWrite` to track the phases.

### Phase 0 — Bootstrap & precondition

1. Confirm session bootstrap is done (best_practices.md read; vault
   `AI/Index.md` + today's daily note open). If invoked mid-session that is
   already true — just confirm.
2. `git -C ~/Documents/Development/jangafx-expert status -sb` and
   `log --oneline -3`. Note unpushed commits — they get batched, never
   silently force-pushed over.
3. Read the **current** target state so the run dedupes instead of duplicating:
   - `skills/jangafx-expert/references/external-resources.md`
   - `skills/jangafx-expert/SKILL.md` (decision tree + reference list)
   - `ls skills/jangafx-expert/references/` (existing distilled files)
   - the relevant product reference (`embergen.md` etc.) if the source is
     about a product already covered — an ingest of a new EmberGen tutorial
     usually *updates* `embergen.md`, it doesn't make a new file.
   **Gate:** you can name what jangafx-expert already covers for this source's
   topic area.

### Phase 1 — Scope the source & plan the slices

1. Verify the source exists and bound it (no guessing):
   - Docs site → `WebFetch` the index; enumerate the real page/section list.
   - YouTube → `WebFetch`/`WebSearch` to confirm the video/series exists and
     get titles + durations; do not invent chapter contents.
   - Forum/Discord/blog → fetch the actual thread/post.
2. Carve the corpus into **≤ 5 non-overlapping slices** for parallel agents.
   Add a "cartographer" slice first when the source is large/unknown (full
   inventory grounds the others). Typical slices: docs inventory · a product's
   node/parameter reference · export/how-to guides · the sim technique itself
   (what settings produce what look) · integration/pipeline (engine import,
   VDB/Alembic/VAT consumption). Adapt to the source.
   **Gate:** a written dispatch manifest — `task id | slice | agent role |
   dispatch time | expected output schema` — exists before any dispatch.

### Phase 2 — Dispatch parallel read-only research agents

- Dispatch all slices in **one message** (true parallelism), capped at 5.
- Use `general-purpose` agents on the **sonnet** model (research tier — keep
  the orchestrator/Arbiter context lean; this is the user's token-efficiency
  preference). The caller stays the Arbiter.
- Build each agent prompt from `references/agent-prompt-template.md` — do not
  free-hand agent prompts. The template enforces: READ-ONLY (no edits, no git),
  Zero-Guessing (report only fetched facts, flag absences), and a fixed report
  schema with a mandatory `## UNVERIFIED` section.
  **Gate:** every dispatched task is in the manifest; no agent prompt omits the
  Zero-Guessing or schema clauses.

### Phase 3 — Fan-in & Arbiter synthesis

1. Reconcile: every manifest task has a returned report. A missing one is
   `HUNG` — re-dispatch that slice once from the manifest; do not proceed with
   a hole.
2. Validate each report's schema (§9 of best_practices): present, non-empty,
   has the required sections incl. `UNVERIFIED`. A malformed report is
   `PRECONDITION_FAILED` — re-run that slice, do not coerce it.
3. Arbitrate cross-agent conflicts with evidence, not reasoning. The recurring
   ones for JangaFX sources: **docs page vs. an outdated YouTube tutorial**
   (the app UI/node names change between versions — rule for the current docs
   page and note the video's version drift); **a documented parameter vs. an
   undocumented solver internal** (only cite what a page states; the rest is
   UNVERIFIED). See `references/integration-and-git.md` §arbiter-heuristics.
   **Gate:** one synthesized candidate set; each item attributed to a real,
   agent-fetched source.

### Phase 4 — Verify every proposed URL (Zero-Guessing gate)

`WebFetch` every URL you intend to put in front of the user that was not
already verified by ≥2 agents who fetched it. Drop or flag anything that does
not resolve to the expected content. Stale/redirecting docs URLs (JangaFX
restructures its docs) do not silently pass.
**Gate:** 100% of proposed URLs verified live by you or cross-confirmed.

### Phase 5 — Integration-depth decision (human gate)

Present the synthesized findings concisely + the **one** real decision, via
`AskUserQuestion`:
- **Curated links only** — append verified entries to `external-resources.md`
  + cross-link. The lighter path.
- **Distilled / update a product reference** — the above **plus** either a new
  `references/<topic>.md` or edits into an existing product reference
  (`embergen.md`, `liquigen.md`, `geogen.md`, `illugen.md`, `export-pipeline.md`,
  `sim-as-conditioning.md`), wired into the decision tree so the expert answers
  offline.
Recommend based on the gap: if jangafx-expert has the product reference but
this source adds a real technique/parameter/export detail it lacks, recommend
updating that reference. Do not decide this unilaterally.

### Phase 6 — Apply (atomic, deduped)

Per the user's choice, edit the canonical standalone repo. Mechanics, file
templates, commit message format, and push protocol are in
`references/integration-and-git.md` — follow it exactly. Summary:
- Product/topic reference: add the verified technique/parameter/export detail
  under the right section; keep the "confirmed vs. flagged/undocumented"
  distinction the references use — new undocumented claims go under a flagged
  note, not stated as fact.
- `external-resources.md`: append verified entries under the right product
  section; if a distilled file exists, point to it first.
- `SKILL.md`: add decision-tree rows + the reference-list line.
- Cross-link sibling references (`export-pipeline.md`, `sim-as-conditioning.md`)
  where the source materially extends them.
  **Gate:** `git diff --stat` shows only intended files; no duplicate of an
  already-present resource; any new undocumented claim is flagged, not asserted.

### Phase 7 — Commit local · close out · batch push on OK

- Stage the specific files (never `git add -A`). Commit with the
  `Co-Authored-By` trailer (format in `references/integration-and-git.md`).
- **Do not push.** Report the commit SHA, what it batches with, and ask for
  explicit push approval. On approval: verify fast-forward first; never
  force-push.
- Close the daily note: tagged Session Log entry, Decisions/What-Worked if
  notable, `## Next Session` handoff carrying any unpushed SHA. Update the
  jangafx-expert project note if structure changed.
  **Gate:** working tree clean; daily note updated; push status explicit.

---

## Reference files

- `references/agent-prompt-template.md` — the Zero-Guessing read-only
  research-agent prompt. Build every Phase-2 agent from this; fill the
  `{{SLICE}}` block per slice.
- `references/integration-and-git.md` — Phase 6/7 detail: reference-file
  templates, external-resources entry format, SKILL.md wiring, arbiter
  heuristics, commit message format, the verify-FF / no-force-push push
  protocol, daily-note closeout.

## Failure handling

Classify every failure with the `best_practices.md` taxonomy
(`TRANSIENT` / `INPUT_INVALID` / `PRECONDITION_FAILED` / `HUNG` /
`LOGIC_DEADLOCK` / `FATAL`) and take the mapped action. Never a generic
"error." A source that 404s entirely is `INPUT_INVALID` — surface it, do not
substitute a guessed source. Same research approach failing 3× is
`LOGIC_DEADLOCK` — stop, report, await the human.
