# reference/dispatch-pattern.md

## The role boundary

**The orchestrator dispatches. The worker builds. Crossing the boundary is the bug.**

That's the whole rule. The rest of this file is what it looks like in practice.

## What an orchestrator does

- Picks the work. (Which video, which platform, which deadline.)
- Writes the brief. (A seven-section file at `briefs/<date>-<slug>.md` — animation briefs carry one extra section vs. content briefs because visual direction is separate from script.)
- Dispatches the worker with one paste prompt. ("Read the brief at briefs/2026-XX-XX-slug.md and execute.")
- Verifies the rendered MP4 came back matching the brief.
- Moves the MP4 to its destination (the CDN / YouTube / platform where it gets published).
- Logs the dispatch outcome.

Orchestration is the 30% layer in the 60-30-10 framework. Its output is briefs and routing decisions, not artifacts.

## What the worker does

- Reads the brief.
- Reads the visual system and footage source configurations.
- Loads only the slices of `reference/remotion-pipeline.md` the brief names.
- Runs the pre-flight checklist.
- Renders the artifact (Remotion composition → MP4, possibly with an ffmpeg post pass for audio-mux or aspect conversion).
- Verifies against the brief's success criteria (duration in target, aspect exact, captions readable, no un-configured footage).
- Updates `STATUS.md`.
- Hands back.

The worker's output is MP4s and the render-receipt markdown that records how each MP4 was produced. Not briefs, not calendars, not routing decisions, not other workers' work.

## What "crossing the boundary" looks like

Markers, in order of severity:

1. **The orchestrator Reads any file longer than 100 lines without an offset + limit.** Single most reliable signal. If the orchestrator is Reading `reference/remotion-pipeline.md` in full, it's hoarding context it won't dispatch with.
2. **The orchestrator queries Pexels themselves to "see what's available."** No. The worker queries through its configured footage source. The orchestrator points at the keyword.
3. **The orchestrator drafts the script before dispatching it.** That belongs in the content worker, or in the operator's own head. The animation worker takes a finished script as input.
4. **The orchestrator produces any artifact other than a brief.** Test renders, scratch compositions, "let me see what this would look like" — all worker territory. If the orchestrator is rendering frames, it has crossed.
5. **The worker writes its own brief.** "Let me figure out what visual direction would be useful here" — that's orchestration work. The worker stops and asks for a real brief.
6. **The worker renders extra variants.** "I also made a 1:1 cut for you" — that's the worker picking work, which is orchestration. The worker delivers what the brief asked for, nothing more.
7. **The dispatch step takes longer than the worker's actual render.** If briefs take twenty minutes and the render ships in eight, the orchestrator is doing 60% of the work twice.

If any of these show up, the fix is not to make either side faster. It's to make each side do less.

## The brief-as-contract

The brief is the only context the worker needs. If a brief is complete, the worker can cold-start, render, and hand back without any conversation. If a brief is incomplete, the worker stops and writes a question file — does not "do its best."

A complete animation brief has seven sections:

1. **What to produce** — the deliverable, in concrete terms (aspect, duration, format).
2. **Audience and platform** — who watches this, where, on what device, with sound or muted.
3. **Script + script timing** — the exact voice-over text with timing breakdown. The worker does not draft script.
4. **Visual direction** — aspect, frame rate, codec, per-section footage cues, title-card spec, caption style.
5. **Voice source** — TTS engine and voice, or operator-recorded audio file, or voice-clone profile (with consent record reference if a clone).
6. **Success criteria** — grep-able list of what "rendered" means.
7. **Refusal context** — any voice-clone consent, high-harm-domain authorization, or claims-requiring-source notes.

See `briefs/_BRIEF-TEMPLATE.md` for the actual template. The template is the contract shape; the brief's content varies per dispatch.

## Why this matters in practice for video

Most starter kits fuse orchestrator and worker into one Claude session. The reader pastes a vague intent ("make me a launch video"), Claude has to guess audience, platform, aspect, duration, voice source, script, footage, and then render, all in one shot.

For prose, that's expensive but recoverable. For video, the cost compounds because the failure modes are bigger:

1. **Token burn for the script + composition prop generation.** Same as prose — the Claude session loads context speculatively to compensate for the missing brief.
2. **Pexels quota burn.** Speculative footage queries cost real money against the operator's Pexels Pro budget.
3. **TTS / voice-clone API burn.** Wrong voice-over → re-render → API charges twice. Voice-clone services typically charge per character generated; mistakes are expensive.
4. **Render-time burn.** Remotion renders aren't free in wall-clock time. A wrong 30-second vertical with the wrong visual direction = render, watch, re-brief, re-render. Twice the wall-clock, twice the CPU.
5. **Inconsistent output.** With no brief to verify against, the artifact's "rightness" is a vibe. Some days it lands; some days the captions are unreadable on phone. The variance is the cost of the boundary not being enforced.

Splitting orchestrator from worker means the worker's job becomes mechanical: given a complete brief, render an MP4 that matches the success criteria. The variance moves where it belongs — into the brief, where the orchestrator can iterate cheaply on paper before any render kicks off.

## When you're acting as your own orchestrator

You probably are, while you're getting started. That's fine — the boundary still applies. Practical discipline:

- **Write the brief in a separate session or before opening the worker.** Don't write the brief inside the same Claude session that will render the MP4. Use a markdown file. Iterate on it. When it's complete, dispatch.
- **Test-read the script aloud at target duration.** A 30-second voice-over is roughly 75 words at a natural pace; if your script is 130 words, your "30-second video" is going to be 50 seconds. Catch the timing problem in the brief, not at render time.
- **Don't carry on a "what should we make today" conversation with the worker.** That's orchestration. Either you do it in your head, or you build a separate orchestrator session that talks to a calendar.
- **Watch for the marker symptoms above.** When you see one in your own workflow, treat it the same way you'd treat a code smell.

## When to build a real orchestrator

You build a separate orchestrator session — its own folder, its own `CLAUDE.md`, its own job — when:

- You're dispatching the same kind of video routinely (weekly social cut, daily product update, etc.).
- The brief-writing itself has become the bottleneck.
- You want a calendar-aware system that proposes briefs based on what's scheduled (a product launch, a campaign milestone, a tied-to-publication-date drop).

The orchestrator's CLAUDE.md says, in essence: "Read today's row from the calendar. Write a brief per worker that's due. Output the paste prompts. Stop." That's the whole job. The orchestrator doesn't render anything. The orchestrator doesn't load worker contexts. The orchestrator points.

The other two repos in the operator-stack series — [`your-content-worker`](https://github.com/NFTYoginis/your-content-worker) and [`your-design-worker`](https://github.com/NFTYoginis/your-design-worker) — follow the same pattern. Once you have all three plus an orchestrator, a launch dispatch looks like: orchestrator writes three briefs (one per worker), dispatches all three, content writes the script, design produces the title-card still, animation renders the video with content's script and design's still as the title card. Each worker reads its own contract; the orchestrator never enters any worker's reference layer.

## Why this is the harder boundary to hold for video

For a prose worker, the artifact is text — cheap to produce, cheap to throw away, cheap to revise. The orchestrator-worker boundary is mostly a discipline issue.

For a video worker, every artifact has real-money cost (Pexels footage, voice-clone characters, render compute, your wall-clock attention). The orchestrator-worker boundary is also an economic issue. If you let it leak, you pay for it in API charges.

This is the strongest argument for the architecture: it's not just cleaner; for video specifically, it's cheaper.

---

Last updated: 2026-05-14.
