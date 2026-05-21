# Brief — script-to-screen-demo

A deliberately domain-agnostic proof-of-output. Its only job is to show, in a few
seconds, what this worker does: it takes a written script (the kind you'd hand to a
voice-over) and turns it into timed, animated video. No product, no industry, no brand —
just the pipeline itself, demonstrated on itself.

---

## 1. What to produce

One short (~6s) vertical (9:16, 1080×1920) silent motion-graphic MP4, plus one
representative still frame as PNG. Self-contained: kinetic typography only, no stock
footage, no photos, no external fonts, no voice track. The render must work offline.

## 2. Audience

Anyone evaluating the worker. The clip should read as "oh — I write a line, it becomes a
scene," with zero domain knowledge required.

## 3. Script (the thing being visualized)

Three beats, each a single narration line. The line is shown as on-screen text the way a
caption would track a voice-over — this is the voice→video mapping made literal.

```
[Beat 1 · 0.0–2.0s]  "You write a line."
[Beat 2 · 2.0–4.0s]  "It becomes a timed scene."
[Beat 3 · 4.0–6.0s]  "Script → screen."   (closing card)
```

## 4. Visual direction

```
Aspect:      9:16 (1080×1920)
Frame rate:  30 fps
Duration:    180 frames (6.0s)
Codec:       h.264 high, yuv420p, no audio
Palette:     near-black ground (#0E1116), off-white type (#F4F1EA), one accent (#E0A458)
Type:        system serif headline + system sans label (no web-font dependency)

Beat 1: An audio waveform of vertical bars pulses at center (sine-driven), standing in for
        the incoming voice track. Caption types in beneath it. Label: "VOICE IN".

Beat 2: The waveform bars collapse into a single baseline; the caption re-flows as a
        timed caption with a progress underline sweeping left→right. Label: "TIMED CAPTION".

Beat 3: Clean closing card. "Script → screen." centered, accent arrow draws in, a small
        play-triangle pulses once. Label: "VIDEO OUT".
```

## 5. Voice source

None — silent by design. The "voice" is represented visually (waveform) so the demo needs
no TTS provider and no API key.

## 6. Success criteria

- 6.0s ± 0.2s, 1080×1920, 30 fps, h.264, no audio track.
- Reads as voice→timed-caption→video with no copy explaining it.
- Renders offline through `scripts/render-video-only.sh` (2-step JPEG→ffmpeg path).
- No brand, no domain, no API keys, no machine-absolute paths in any committed artifact.
