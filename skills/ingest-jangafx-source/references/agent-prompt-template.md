# Research-agent prompt template (Phase 2)

Build **every** parallel research agent from this template. Fill `{{SLICE}}`,
`{{SOURCE}}`, `{{KNOWN_COVERAGE}}`, and `{{SCHEMA_EXTRAS}}`. Do not free-hand
agent prompts — the fixed clauses below are what keep the run trustworthy
(read-only, Zero-Guessing, schema-validated on fan-in).

Dispatch as `general-purpose`, model `sonnet`. All slices in one message.

---

```
READ-ONLY research task. Do NOT edit any files. Do NOT run git write commands.
Do NOT download the software. Output a structured report only.

CONTEXT: We maintain a Claude skill `jangafx-expert` — a principal-level advisor
for the JangaFX real-time simulation suite (EmberGen = fire/smoke/explosions,
LiquiGen = liquids/whitewater, GeoGen = procedural terrain, IlluGen =
illumination) with a curated `references/external-resources.md` and distilled
per-product `references/*.md` files. We are enriching it from this source:

  {{SOURCE}}

We already cover (do NOT re-report these as new — they are deduped upstream):
  {{KNOWN_COVERAGE}}   # from Phase 0; e.g. "embergen.md export section, VDB how-to"

YOUR SLICE:
  {{SLICE}}
  # e.g. "Cartographer — full inventory of docs.jangafx.com/<product>/ pages"
  # or   "Node/parameter reference for <product> — exact node & param names"
  # or   "Export & How-To guides — every documented output format + settings"
  # or   "The sim technique — which settings produce which look (from tutorial)"

ZERO-GUESSING (non-negotiable): Report ONLY node names, parameters, export
formats, menu paths, facts, and URLs you actually retrieved this run. Never
infer a node/parameter/format exists. JangaFX docs deliberately omit some
solver internals (e.g. LiquiGen sub-steps, CFL, meshing controls; minimum-VRAM
figures; scripting APIs) — if the source does NOT state something, report that
ABSENCE under UNVERIFIED rather than supplying a plausible value. A wrong fact
that proceeds silently is worse than a visible gap. Capture exact names
verbatim — parameter labels and node names must match the app exactly.

METHOD:
- Docs: WebFetch the index, then the actual node-list / how-to / settings
  pages — summarize from content you read, never from a title alone. Note the
  page URL for every fact.
- YouTube/video: confirm the video/series exists and get its title, author,
  and (if shown) app version. Report what the video demonstrates ONLY at the
  granularity you can actually confirm from titles/descriptions/transcripts —
  do not fabricate step-by-step contents you did not see. Flag app-version
  drift (node/UI names change between JangaFX releases).
- Verify any docs/tutorial URL actually resolves before reporting it as live.
  Note redirects and whether a page has been superseded by a newer docs path.

DELIVERABLE — return EXACTLY these sections (the orchestrator validates this
schema on fan-in; a missing section fails the report):

## VERIFIED FINDINGS
One row per item you actually confirmed:
`item (node/param/format/technique) | source URL (verified reachable) | what it
is / does verbatim where useful | why it materially helps a JangaFX expert |
recommendation: LINK | DISTILL-INTO-<product>.md | NEW-TOPIC-FILE | SKIP`

## STANDOUT INSIGHTS
The 5–8 most non-obvious, high-value takeaways from what you read — specific
enough to change how someone builds a sim or wires an export (name the
parameter, the interaction, the gotcha, the fix). Skip if your slice is pure
inventory.

## CANONICAL URL(S)
The exact URL(s) to cite, and for each: is it the current canonical docs page,
or a stale/redirecting one that a newer path supersedes? State what you
confirmed.

## CROSS-SIGHTINGS
High-value items you saw outside your slice (so nothing is missed even though
slices are disjoint).

## RECOMMENDATION
For the skill: individual links vs. distilling into an existing product
reference vs. a new topic file. 2–3 sentences, justified. If you'd distill,
sketch the section headings and say which existing reference to edit.

## UNVERIFIED
Everything you could NOT confirm: failed fetches, features/params the source
did NOT document, version-drift caveats, anything assumed. Be explicit.
Record documented ABSENCES here (e.g. "no CLI/scripting mentioned anywhere in
this product's docs"). Never paper over a gap.

{{SCHEMA_EXTRAS}}   # optional slice-specific required fields; else omit

Keep it tight and factual. Other agents and the Arbiter depend on this being
exact, not generous.
```

---

## Notes for the orchestrator

- **Cartographer-first for unknown/large sources.** One agent does the full
  docs inventory; its page list is the ground truth the other slices and your
  Arbiter ruling lean on. Tell the other agents to self-ground on the docs
  index so a single agent failure doesn't blind the run.
- **`{{KNOWN_COVERAGE}}` is load-bearing.** Without it agents re-report things
  jangafx-expert already has (the export formats, the VDB how-to) and you waste
  a dedupe pass. Fill it from Phase 0.
- **Absence is a finding.** JangaFX docs' omissions (no scripting API, no
  published VRAM floor, undocumented solver internals) are exactly the facts
  the expert must know to avoid guessing. Require agents to log them under
  UNVERIFIED, and carry the important ones into the reference as flagged notes.
- **Schema validation is not optional.** On fan-in, check each report has all
  required sections before you read its findings. A malformed report is
  `PRECONDITION_FAILED` — re-run that one slice, do not hand-repair it.
- **The agents never decide integration depth or touch git.** They are
  Workers. You are the Arbiter. Raw agent output is an intermediate artifact.
