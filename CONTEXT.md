# CONTEXT.md — routing and load logic

This file is the 30% orchestration layer made explicit. It tells the worker which reference files to load for which kind of dispatch, so context doesn't get loaded speculatively.

`CLAUDE.md` is the entry point — read first. This file is referenced from there.

## Why this file exists

In the 60-30-10 framework (see `reference/icm-layer-model.md`), Orchestration is 30% of the workflow's value: templates, rules, decision logic, the connective tissue that makes raw tools useful for a specific job. Most starter kits put orchestration logic *inside* Claude's reasoning — "decide which file to read, which composition to render, which ffmpeg pass to apply." That's expensive and inconsistent.

This file moves that decision out of Claude's working context and into a declarative rule list. Worker reads this once, applies the rule, loads only what the rule says to load. No speculation.

## The dispatch types this worker handles

| Dispatch type | What it produces | Files to load (in this order) |
| --- | --- | --- |
| **Voice-over short** | 15-60 second voice-over MP4 with stock footage backing (vertical 9:16 or horizontal 16:9) | `reference/visual-system.md` → `reference/footage-sources.md` → the brief → `reference/remotion-pipeline.md` (slice: composition + render command sections) |
| **Product demo cut** | 30-120 second demo MP4 from a script + screen-recording footage the operator supplies | `reference/visual-system.md` → the brief → `reference/remotion-pipeline.md` (slice: composition + audio-mux sections) |
| **Square social variant** | 1:1 cut of an existing horizontal MP4, re-framed and re-timed for feed-friendly scroll | the brief → `reference/visual-system.md` (slice: typography rules for over-the-shoulder captions) → `reference/remotion-pipeline.md` (slice: aspect-conversion section) |
| **Caption / subtitle pass** | SRT or VTT file from a script + timing | the brief → `reference/remotion-pipeline.md` (slice: caption-generation section). No render. |
| **Lecture / talking-head cut** | 60-300 second cut from operator-supplied footage of themselves speaking, with B-roll inserts | the brief → `reference/footage-sources.md` → `reference/visual-system.md` → `reference/remotion-pipeline.md` (slice: composition + b-roll-insert sections) |

Add a row above when you (the operator) start dispatching a new kind of work to this worker. Don't let the routing live in Claude's head — declare it here.

## What never gets loaded automatically

- The full `briefs/` history. Each render is grounded by ONE brief. Loading prior briefs is speculative context — exactly the mistake that burns tokens.
- Examples from `examples.md` — that file is for *you* to study the worker's behavior, not for the worker to study its own past output. Past output is usually noise during a fresh dispatch.
- Render artifacts from prior dispatches. Same reason. MP4s are gitignored anyway.
- Any file under `output/` — write-only from the worker's perspective.
- The full `reference/remotion-pipeline.md` file. It is the longest reference here by design (it documents composition shape, render commands, audio-mux options, aspect conversion, B-roll inserts, caption generation). Slice it.

The worker reads what the brief requires, and nothing more. If a brief asks the worker to "produce a square cut of last week's demo," the brief itself names that prior MP4's source files; the worker reads only the named files, not the whole archive.

## Slice-not-file Read habit (mechanical rule)

For any reference file longer than 100 lines (which includes `reference/remotion-pipeline.md`):

1. `grep -n "<section-header>" reference/<file>.md` — find the line number.
2. `Read reference/<file>.md` with `offset=<line>` and `limit=<lines-needed>` — pull only the slice.

Never `Read` the whole pipeline reference to find the 30 lines that matter for this dispatch. That's the single most expensive habit in this kind of build (Article 1, Mistake 2).

## Render-pipeline state

This worker assumes the following are available locally (the operator's machine, not the repo):

- **Node + npm** for Remotion. `npx remotion render …` is the canonical render command.
- **ffmpeg** for any post-render passes (aspect conversion, audio-mux, caption burn-in). Not bundled in this repo — operator installs.
- **A voice source.** Either a TTS engine the brief names, a voice-clone API the brief names with an env-stored key, or a pre-recorded audio file the brief points at.
- **A footage source.** Either Pexels API key (env-stored) or a local folder of operator-licensed footage. Configured at `reference/footage-sources.md`.

The repo does NOT ship a working Remotion project (no `node_modules/`, no compiled compositions). The first dispatch with a Remotion-rendered output requires the operator to `npm init` a Remotion project alongside this worker. The pipeline reference documents the composition shape; the operator creates the actual `Composition` files when they start dispatching real work.

This is intentional. Pinned dependencies rot fast and a starter that bundles them goes stale within months.

## When the operator configures a new dispatch type

1. Add a row to the dispatch-type table above.
2. Decide which reference files load by default. Be miserly.
3. If a new reference file is needed (campaign aspect-spec, voice-clone provider notes, footage-license catalog), add it under `reference/` with a clear filename.
4. Add a one-line note to `STATUS.md` so the worker knows the routing changed.

## Open routing question (for the operator, not the worker)

If a brief names a dispatch type that isn't in the table above, the worker writes a question file at `briefs/questions/<slug>-question.md` and stops. It does not invent a routing decision on the fly. Add the row first, dispatch second.

---

Read this file when in doubt about what to load. Read `CLAUDE.md` first.
