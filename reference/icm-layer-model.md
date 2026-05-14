# reference/icm-layer-model.md

## The 60-30-10 layer framework

From ICM (Internal Coherence Maximization), Jake Van Clief's framework for thinking about where the value of an AI workflow actually lives. A real workflow is three layers, by share of value:

- **60% Infrastructure** — databases, file storage, render binaries, codecs, hosting, CDN. Systems that already exist and shouldn't be replaced.
- **30% Orchestration** — templates, rules, decision logic, brief shapes, composition definitions. The connective tissue that makes raw tools useful for a specific context.
- **10% AI** — script generation, footage-cue matching, caption phrasing, light editorial decisions on what plays under which line.

AI is the *smallest* layer. Most of the value lives in the other two.

This is the corrected definition. If you've seen 60-30-10 framed as a context-budget split ("60% always-loaded, 30% task-scoped, 10% on-demand") — that's a different concept and not what Van Clief teaches. The numbers happen to match; the layers don't.

## How this worker maps to the framework

| Layer | What it is in this repo |
| --- | --- |
| **60% Infrastructure** | Node + Remotion + ffmpeg locally. Pexels API. Your TTS or voice-clone provider. Your CDN / YouTube / S3. The platforms where the MP4 ultimately gets watched. Your file system. Claude doesn't touch most of this. |
| **30% Orchestration** | `CLAUDE.md` + `CONTEXT.md` + `STATUS.md` + `rules.md` + `briefs/_BRIEF-TEMPLATE.md` + the routing table + the Remotion composition shapes documented in `reference/remotion-pipeline.md` + the visual-system rules + the footage-source config. Declarative rules that tell the worker what to do, what to load, what to refuse, what shape the output takes. Read by Claude, but not derived by Claude. |
| **10% AI** | The actual script-to-frame mapping. Take the brief + the script + the visual system + the footage cues → name the composition props, name the Pexels search keywords, name the audio-mux choices. This is the only step where Claude's reasoning is load-bearing. The render itself is Remotion + ffmpeg infrastructure. |

The crucial discipline: do not let the 10% layer absorb work that belongs in the 60% or 30%. Symptoms of the 10% absorbing too much:

- The worker re-derives composition prop shapes every render (should be declarative in `reference/remotion-pipeline.md`).
- The worker re-reads its own past renders to "remember the visual system" (should be a stable rule in `reference/visual-system.md`).
- The worker queries Pexels five times comparing options (should be one query, best match, one render — orchestration creep into the worker).
- The worker drafts the script (should be the orchestrator's job, or a separate dispatch to the content worker).

When the 10% absorbs the other layers, token cost explodes, render time balloons, and output gets inconsistent. The fix is always: move that decision out of Claude and into a file or a script.

## Why the 60-30-10 split matters more for video than for prose

A prose worker is mostly the 10% — Claude generates text, end of story, the infrastructure is "save the markdown file." A video worker isn't. Roughly:

- The 60% infrastructure layer is *huge*: Remotion, ffmpeg, the codec stack, the footage API, the TTS / voice-clone API, the CDN. Most of these are services you don't own.
- The 30% orchestration layer is *load-bearing*: composition prop shapes, audio-mux passes, aspect conversion, caption burn-in passes. Get these wrong and the render fails before AI gets a chance to contribute.
- The 10% AI layer is genuinely small: pick the right footage keyword, pick where the script breaks, name the title-card text.

If you set up the 60% and the 30% right, the 10% becomes mechanical — and the cost of the render drops by an order of magnitude. If you let Claude reason about composition shapes and ffmpeg flags every render, you're paying for the AI to re-derive your build system every time.

## Worked example from Article 1

The operator's daily content routine was costing roughly 800,000 tokens per morning. Same routine every day, same outputs. The orchestrator (then a single Claude session doing everything) was:

- Reading whole reference files when it needed a slice — infrastructure leak into AI.
- Re-deriving worker boundaries every morning — orchestration leak into AI.
- Drafting partial artifacts before dispatching — AI doing work that belonged to the dispatched worker.

After applying the role boundary and slicing reads, the same daily routine cost roughly 8,000-10,000 tokens. Two orders of magnitude. Same outputs. Same quality.

The savings weren't from a smarter model. They were from removing work the orchestrator was never supposed to be doing.

For a voice-to-video worker, the same lesson applies — only more so, because the per-render infrastructure cost (Pexels quota, TTS API charges, voice-clone API charges) is higher than the per-render prose cost. Token economy compounds with API economy.

## Attribution and further reading

The 60-30-10 layer framework comes from Jake Van Clief's ICM teaching. We don't reproduce the source material here — go to the source.

If you want the practical version applied to a solo operator's daily routine, see Article 1 of the operator-AI series ("I burned 800,000 tokens on one daily routine") — link in the README.

## Why this file exists in this repo

You forked this starter to put it to work. The 60-30-10 framework is the rationale for everything in the architecture — why `CLAUDE.md` is short, why `CONTEXT.md` exists as a separate file, why the Remotion-pipeline reference is sliced not full-read, why the brief is a contract, why scope extension is a bug.

When you find yourself tempted to "just let Claude figure out the render flags" — re-read this file. The architecture exists to keep AI in its 10% lane.

---

Last updated: 2026-05-14.
