# Integration & git protocol (Phase 6/7)

How synthesized findings become durable jangafx-expert knowledge. Codifies how a
run is applied so every ingestion is consistent.

Paths are relative to `~/Documents/Development/jangafx-expert/` (the canonical
standalone repo — the only place edits land; the `~/.claude/skills/jangafx-expert`
path symlinks into it; any `~/.claude/plugins/` clone is a secondary mirror only).

---

## §arbiter-heuristics

Recurring cross-source conflicts and how to rule (with evidence, not reasoning):

- **Docs page vs. an older YouTube tutorial.** JangaFX changes UI/node names
  between versions. **Rule:** cite the current docs page; note the video's app
  version and flag drift. Never let a tutorial's node name override a live docs
  node name.
- **Documented parameter vs. undocumented solver internal.** Only cite what a
  page states. Sub-steps/CFL/meshing (LiquiGen), exact heightmap containers
  (GeoGen), IlluGen node schemas — these are UNVERIFIED unless a page says
  otherwise. Carry them into a reference **only** as a flagged note, never as
  fact.
- **First-party vs. press/third-party.** Prefer docs/product/roadmap. When only
  press has a number (hardware min-spec, pricing, Alembic caveat), tag it
  `[non-docs]` / `⚠️` in the reference exactly as the existing files do.
- **Absence as a finding.** "No CLI," "no published VRAM floor," "docs are a
  placeholder" are real, valuable facts — record them, don't paper over them.
- **Stale docs path.** JangaFX restructures docs; a `How-To Guides/...` URL may
  move. Verify live in Phase 4; cite the resolving URL.

## §reference-file — where content lands

Most ingestions **update an existing reference**, not create a new file:

| Source is about… | Edit |
|---|---|
| An EmberGen feature/param/export | `references/embergen.md` |
| A LiquiGen feature/param/export | `references/liquigen.md` |
| A GeoGen feature/export | `references/geogen.md` |
| An IlluGen feature/export | `references/illugen.md` |
| Licensing / hardware / shared graph / interop | `references/suite-overview.md` |
| A new destination or format-consumption detail | `references/export-pipeline.md` |
| A generative/NeuralFX conditioning technique | `references/sim-as-conditioning.md` |
| Any verified URL | `references/external-resources.md` (always) |

Create a **new `references/<topic>.md`** only when the source opens a genuinely
new topic area none of the above covers (e.g. a deep third-party technique
series). Match the house style: sourced claims, `[non-docs]`/`⚠️`/"flag" markers
for anything not first-party-documented, a Sources section at the bottom.

### New-file template
```
# <Topic> — <one-line scope>

> Sourced from <source> (crawled <date>). First-party facts unmarked; press/
> third-party tagged [non-docs]; undocumented/uncertain items flagged ⚠️.

## 1. When this matters
## 2. <content, non-obvious-first>
...
### Sources
- <name> — <verified URL>
```

## §external-resources — entry format

Append under the correct existing section in `references/external-resources.md`
(Official docs / Product & roadmap / Learning / Integration write-ups / Press).
Never re-add an existing URL (dedupe against Phase 0). If a distilled reference
was updated, the value is in that reference — external-resources just holds the
verified link.

## §SKILL.md-wiring

In `skills/jangafx-expert/SKILL.md`:
1. Add decision-tree rows under the right section pointing at the new/updated
   reference, phrased as the question a user actually asks.
2. If a new reference file was created, add one line to "Reference files in this
   skill."
3. If the source corrects a load-bearing assumption (like the IlluGen-isn't-
   lighting / GeoGen-frozen / no-CLI truths), update the "three load-bearing
   truths" section — those are the highest-value corrections the skill makes.

## §commit

Stage **specific files only** (never `git add -A`). One commit per ingestion:

```
Add <topic> from <source> to jangafx-expert

<2–4 lines: what was distilled and the gap it closes; which references +
SKILL.md rows were wired; any arbiter ruling (docs vs. video version); note
which claims are first-party vs. flagged; "all cited URLs verified live".>

Co-Authored-By: Claude Opus 4.8 <noreply@anthropic.com>
```

## §push-protocol

**The push is always a separate, explicit human approval — never bundled into
"apply".**

1. After commit: `git -C ~/Documents/Development/jangafx-expert status -sb`
   (note `ahead N`; list every unpushed SHA incl. ones batched from prior runs).
2. Report SHAs + what they batch. Ask for explicit push approval.
3. On approval only: `git fetch`, confirm local is a clean **fast-forward** over
   `origin/main`.
   - Fast-forward → `git push origin main`.
   - **Non-fast-forward → STOP.** Surface it; the user decides. **Never
     force-push** (concurrent instances may have advanced origin).
4. Never `--no-verify`; never amend a pushed commit.

## §closeout

- Daily note (`/Users/jdot/Documents/Obsidian/joeOS/Daily Notes/<today>.md`):
  append a `### [Mac] HH:MM — jangafx-expert: ingested <source>` Session Log
  entry (what was added, arbiter rulings, commit SHA + push state). Append —
  never overwrite a concurrent instance's blocks. Add `_jangafx-expert:_`
  bullets to Decisions/What-Worked if notable, and a `## Next Session` handoff
  carrying any unpushed SHA.
- If the skill's structure/coverage changed materially, update the
  jangafx-expert project note in `Projects/`.
