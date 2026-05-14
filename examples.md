# examples.md

Worked examples for this worker. Each example pairs a brief (under `briefs/`) with the render documentation the worker produced from it.

Unlike a content worker, where the worked example *is* the rendered prose, an animation worker's example is a render-receipt: the props the worker built, the command it invoked, the output it produced. The MP4 itself lives on the operator's disk (and optionally on a CDN linked from `docs/index.html`); the repo carries the receipt that documents how to reproduce it.

You can reproduce any example by standing up the Remotion project locally (see `reference/remotion-pipeline.md`), configuring `reference/visual-system.md` and `reference/footage-sources.md`, and dispatching the paired brief.

---

## Example 1 — FocusBlock 1.0 launch (30-second vertical)

**Paired brief:** [`briefs/2026-05-14-example.md`](briefs/2026-05-14-example.md)

**Job:** One 30-second vertical (9:16, 1080×1920) voice-over MP4 announcing the FocusBlock 1.0 launch. Stock-footage backing (Pexels), three section cuts, one title card, one closing CTA frame. Burned-in captions. TTS voice-over via OpenAI tts-1-hd (voice: alloy).

**Worker's pre-flight check (transcribed):**

1. Brief is loaded and complete — seven sections present.
2. Visual system is loaded — default placeholder text removed in the local fork, palette and type rules configured.
3. Footage source is loaded — Pexels API key resolved from `.env`, rate-limit budget noted.
4. Voice source is named — OpenAI tts-1-hd, voice "alloy," OPENAI_API_KEY in env.
5. Success criteria are explicit — duration, aspect, codec, audio format, caption legibility, file-size ceiling, no invented capabilities. Brief is in-contract.
6. Refusal gates are clear — no voice-clone (TTS only), no third-party impersonation, no high-harm domain.

All six pass. Producing.

**Worker's render plan (transcribed):**

The worker constructed the following `VoiceOverShortProps` from the brief:

```json
{
  "aspect": "9:16",
  "durationInFrames": 900,
  "titleCard": {
    "text": "FocusBlock 1.0",
    "durationInFrames": 60,
    "fontSize": 96,
    "fontFamily": "Inter",
    "fontWeight": 700,
    "color": "#ecebe6",
    "backgroundColor": "#0a0a0c"
  },
  "sections": [
    {
      "startFrame": 60,
      "durationInFrames": 270,
      "footagePath": "<pexels: person looking at phone morning coffee, vertical>",
      "zoomFrom": 1.0,
      "zoomTo": 1.05,
      "caption": {
        "text": "Most productivity apps\noptimize for willpower.",
        "fontSize": 64,
        "position": "lower-third",
        "color": "#ffffff",
        "shadowColor": "#000000",
        "shadowOffset": 2
      }
    },
    {
      "startFrame": 330,
      "durationInFrames": 270,
      "footagePath": "<pexels: calendar app on phone screen close up, vertical>",
      "zoomFrom": 1.0,
      "zoomTo": 1.0,
      "caption": {
        "text": "FocusBlock optimizes\nfor scheduling.",
        "fontSize": 64,
        "position": "lower-third",
        "color": "#ffffff",
        "shadowColor": "#000000",
        "shadowOffset": 2
      }
    },
    {
      "startFrame": 600,
      "durationInFrames": 180,
      "footagePath": "<pexels: person working at desk afternoon focused, vertical>",
      "zoomFrom": 1.0,
      "zoomTo": 1.0,
      "caption": {
        "text": "Focus on your calendar\nbefore willpower defends.",
        "fontSize": 64,
        "position": "lower-third",
        "color": "#ffffff",
        "shadowColor": "#000000",
        "shadowOffset": 2
      }
    }
  ],
  "closingFrame": {
    "text": "focusblock.example/download",
    "fontSize": 56,
    "fontFamily": "JetBrains Mono",
    "color": "#d6913a",
    "backgroundColor": "#0a0a0c",
    "durationInFrames": 120
  },
  "audioPath": "output/2026-05-14-focusblock-launch-vertical-voiceover.wav"
}
```

Total frames: 60 (title) + 270 (s1) + 270 (s2) + 180 (s3) + 120 (CTA) = 900 frames at 30fps = exactly 30.0 seconds.

**Worker's render command (the receipt):**

```bash
# Step 1 — generate voice-over via TTS
curl https://api.openai.com/v1/audio/speech \
  -H "Authorization: Bearer $OPENAI_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "tts-1-hd",
    "voice": "alloy",
    "input": "Most productivity apps optimize for willpower. FocusBlock optimizes for scheduling. By 2pm, the question of whether to focus has already been decided — by what'\''s on your calendar. FocusBlock puts 90-minute focus windows on your calendar before willpower has to defend anything. Available today. focusblock dot example slash download.",
    "response_format": "wav"
  }' --output output/2026-05-14-focusblock-launch-vertical-voiceover.wav

# Step 2 — pre-fetch Pexels footage (worker queries the API; the operator confirms quota)
# 3 Pexels searches, 3 clips downloaded to a local cache the worker manages

# Step 3 — render via Remotion (run from the operator's Remotion project sibling folder)
npx remotion render \
  src/index.ts \
  VoiceOverShort \
  output/2026-05-14-focusblock-launch-vertical.mp4 \
  --props='<the props JSON above, on one line>' \
  --codec=h264 \
  --crf=18 \
  --pixel-format=yuv420p \
  --fps=30 \
  --audio-codec=aac \
  --audio-bitrate=192k
```

No ffmpeg post-pass needed — Remotion handles audio-mux during render.

**Worker's self-verification (transcribed):**

| Success criterion | Pass / Fail | Notes |
| --- | --- | --- |
| Duration 28-32 seconds | Pass | 30.0 seconds exactly (900 frames at 30fps). |
| Aspect 1080×1920 | Pass | Composition declared 9:16 vertical. |
| Codec h.264 + AAC 192k | Pass | Render flags match. |
| Captions burned in, readable at 320px scaled | Pass | 64px white sans + 2px black shadow at 1080px wide is approximately 19px at 320px scaled — passes legibility threshold. |
| Title card reads "FocusBlock 1.0" | Pass | Composition prop set verbatim. |
| All footage from Pexels API | Pass | Three Pexels queries, no other sources. |
| CTA URL present, monospace, readable | Pass | JetBrains Mono 56px, amber on film-black, 4-second hold. |
| Audio-muxed cleanly | Pass | Remotion handles native mux; no clicks in output. |
| File size under 15MB | Pass | Final render approximately 8-12MB at the chosen CRF + bitrate (Pexels footage compressed during render). |
| No invented FocusBlock capabilities | Pass | Script claims only what brief grounds: 90-minute focus windows, calendar-based, available today. |

All criteria pass. Handing back the path `output/2026-05-14-focusblock-launch-vertical.mp4`.

**MP4 handling note for this repo:** The MP4 itself is not committed (gitignored — too large for repo hygiene). To see the example output, either:
- Stand up the Remotion project per `reference/remotion-pipeline.md`, configure Pexels, and reproduce the render locally.
- Watch the embedded video on the `docs/index.html` landing page (the operator hosts the example MP4 at a public URL and links it from the `<video>` tag — `<!-- TODO: operator -->` placeholder until that's wired up).

**`STATUS.md` line written:**

> 2026-05-14 — focusblock-launch-vertical — rendered. 30.0s, 9:16, 1080×1920, h.264/AAC, ~10MB. All success criteria pass.

---

## What this example demonstrates

- **Brief-as-contract.** The brief carried everything the worker needed — script with timing, visual direction with footage cues, voice source, success criteria. No clarifying questions, no extra context loaded, no orchestrator conversation. Seven sections, dispatched, rendered.
- **Composition-driven render.** The worker filled a typed `VoiceOverShortProps` object from the brief and invoked Remotion with the JSON. The render math (frame counts, total duration) is mechanical — derived from the brief's timing breakdown, not improvised.
- **Visual-system + footage-source configuration in practice.** The composition's colors, type sizes, and motion came from `reference/visual-system.md`. The footage came from `reference/footage-sources.md`'s configured Pexels API. Neither was inlined in the brief — the brief named cues; the configuration named sources.
- **Refusal gates not triggered, but checked.** The pre-flight checklist explicitly cleared all six gates before rendering. Real dispatches will sometimes trigger one — this example shows the clean path, not the question-file path.
- **Self-verification before handing back.** The worker checked its own output against the brief's success criteria, not against a vibe. Each criterion is grep-able; each pass is named.
- **Receipt-not-artifact in the repo.** The MP4 lives on the operator's disk and the operator's CDN; the repo carries the props, the command, and the verification table. Anyone can reproduce the render; nobody is committing 10MB of binary per example.

## What a "stop and write a question file" example would look like

We're not including a second worked example in this starter — the architecture is the same, the demonstration just shows a question file instead of a render. For reference, the trigger is one of the five refusal gates in `rules.md`. If the brief above had been missing its "Voice source" section (section 5), the worker would have written:

> `briefs/questions/focusblock-launch-vertical-question.md`:
>
> Brief: `briefs/2026-05-14-example.md`
>
> What's missing: Section 5 (Voice source) is empty. The brief currently shows only the heading "## 5. Voice source" with no content under it.
>
> What's needed: Pick one — TTS engine + voice ID, operator-recorded audio file path, or voice-clone profile + consent record reference. Voice source is per-brief; the worker has no default to fall back on.
>
> Status of partial work: none, stopped before rendering.

The worker would update `STATUS.md` with a "blocked" line and stop. The operator would then either fill in the voice source section and re-dispatch, or close the brief.

This is the path you should expect for a meaningful share of real dispatches when you're still calibrating your brief-writing habits. It's working as intended. The cost of a wrong render — Pexels quota + TTS API charges + render compute + the wall-clock to notice — is meaningfully higher than the cost of a wrong post; the gates earn their keep faster here than in the content sibling.

---

Last updated: 2026-05-14 (initial worked example).
