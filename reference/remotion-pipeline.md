# reference/remotion-pipeline.md

The pipeline reference. How Remotion + ffmpeg + your voice source + your footage source combine to render an MP4 from a brief.

This file is long. The worker reads it in slices, per `CONTEXT.md`'s routing table. Use `grep -n "## "` to find section headers, then `Read` with `offset` and `limit`.

---

## Why Remotion

Remotion is React for video. You write compositions in JSX-like syntax — they declare what plays when — and Remotion renders frames out to MP4 via Chromium + ffmpeg. The reasons this worker uses Remotion rather than a pure ffmpeg pipeline or a video-editor binary:

- **Declarative composition.** A composition is a file (`src/Composition.tsx` or similar). The brief names which composition and what props. The worker doesn't generate ffmpeg flag strings from scratch — it picks a composition and renders.
- **Programmable typography.** Captions, title cards, lower-thirds, animated text overlays — all standard React/JSX. The visual-system rules from `reference/visual-system.md` map onto component props.
- **Audio sync.** Remotion handles audio-mux during render. No separate ffmpeg audio-mux pass for most jobs.
- **Aspect-conversion via composition.** Different aspect ratios (9:16 / 16:9 / 1:1) are just different composition widths and heights. The same script + footage + audio can render to different aspect compositions.

ffmpeg is still in the stack — for post-render passes (codec conversion, certain re-encodes, caption burn-in for soft-track MP4s, etc.). But the primary render path is Remotion.

## What this repo does NOT ship

- **A working Remotion project.** No `node_modules/`, no `package.json`, no `src/Composition.tsx`. Pinned dependencies rot fast; bundling a Remotion project would mean every fork is months behind by the time anyone clones it.
- **Pre-built compositions.** This file documents the composition *shapes* — what props they accept, what they render. The operator stands up the actual Remotion project alongside this worker.
- **Pre-built ffmpeg passes.** Documented as commands, not as scripts. Adapt to your codec requirements.

This is intentional. The starter is the architecture and the contracts; the build tools you stand up yourself, fresh, against current versions.

## Standing up Remotion alongside this worker (one-time setup)

Run these in a new sibling folder, NOT inside this repo (keep the worker's contract files separate from build tooling):

```bash
# In a sibling folder, e.g., ~/Code/animation-render/
npm init -y
npm install remotion @remotion/cli @remotion/google-fonts @remotion/media-utils
npx remotion init

# Or use Remotion's official starter:
npx create-video@latest
```

That gives you a Remotion project with a `src/` folder, a default composition, and `npx remotion preview` + `npx remotion render` commands.

The worker, when rendering, calls `npx remotion render` with arguments. The worker does NOT install or update Remotion; you do, as part of your local toolchain.

## The composition shapes

Two compositions cover most dispatches. Documented as TypeScript prop shapes — your Remotion project implements them as actual components.

### Composition 1 — `VoiceOverShort`

For voice-over MP4s with stock-footage backing. 15-60 seconds. Vertical, horizontal, or square.

```typescript
type VoiceOverShortProps = {
  aspect: '9:16' | '16:9' | '1:1';
  durationInFrames: number;  // 30fps × duration in seconds
  titleCard: {
    text: string;
    durationInFrames: number;
    fontSize: number;
    fontFamily: string;
    fontWeight: number;
    color: string;
    backgroundColor: string;
  } | null;
  sections: Array<{
    startFrame: number;
    durationInFrames: number;
    footagePath: string;       // Local path or remote URL
    zoomFrom: number;          // 1.0 default
    zoomTo: number;            // 1.05-1.10 typical for slow push
    caption: {
      text: string;
      fontSize: number;
      position: 'lower-third' | 'centered-middle' | 'centered-bottom';
      color: string;
      shadowColor: string;
      shadowOffset: number;
    } | null;
  }>;
  closingFrame: {
    text: string;
    fontSize: number;
    fontFamily: string;   // Usually monospace for URL CTAs
    color: string;
    backgroundColor: string;
    durationInFrames: number;
  } | null;
  audioPath: string;           // The voice-over track
};
```

The worker, given a brief, fills these props from the brief's sections 3 (script + timing), 4 (visual direction), and the configured visual-system rules. Then renders.

### Composition 2 — `DemoCut`

For product demos with operator-supplied screen-recording footage. 30-180 seconds. Usually 16:9 horizontal.

```typescript
type DemoCutProps = {
  aspect: '16:9' | '9:16';
  durationInFrames: number;
  titleCard: { /* same shape as above */ } | null;
  sections: Array<{
    startFrame: number;
    durationInFrames: number;
    sourceVideoPath: string;     // Operator-supplied recording
    sourceStartFrame: number;    // Trim-in from source
    crop: { x: number; y: number; width: number; height: number } | null;  // Re-frame if needed
    caption: { /* same shape */ } | null;
    annotation: {
      text: string;
      x: number;
      y: number;
      arrowToPoint: { x: number; y: number } | null;
    } | null;
  }>;
  closingFrame: { /* same shape */ } | null;
  audioPath: string;             // Voice-over OR muted source audio
  audioMuxMode: 'voice-over' | 'preserve-source' | 'mix';
};
```

Captures the common demo-cut shape: trim segments from a longer recording, optionally re-frame, optionally annotate, mux with voice-over.

You can define additional compositions in your Remotion project — a `TalkingHeadCut`, a `SquareSocialVariant`, etc. — and add a row to `CONTEXT.md`'s dispatch-type table that names which composition the worker reaches for.

## The render command

The canonical render command the worker invokes (assuming your Remotion project is at `~/Code/animation-render/`):

```bash
cd ~/Code/animation-render

npx remotion render \
  src/index.ts \
  VoiceOverShort \
  output/2026-05-14-focusblock-launch-vertical.mp4 \
  --props='{"aspect":"9:16","durationInFrames":900,"titleCard":{...},"sections":[...],...}' \
  --codec=h264 \
  --crf=18 \
  --pixel-format=yuv420p \
  --fps=30 \
  --audio-codec=aac \
  --audio-bitrate=192k
```

The `--props=` flag carries the entire VoiceOverShortProps object as JSON. The worker constructs this from the brief.

Key flags:
- `--codec=h264` — universal compatibility. Use `h265` only if your target platform supports it and you need the size savings.
- `--crf=18` — quality factor. 18 is "visually lossless at watch resolution." Drop to 23 for smaller files; raise to 15 if quality matters more than size.
- `--pixel-format=yuv420p` — required for most platforms (LinkedIn, Twitter, Instagram). Some platforms reject `yuv444p`.
- `--fps=30` — standard. Use `--fps=60` for screen recordings where motion smoothness matters; use `--fps=24` for cinematic feel.
- `--audio-codec=aac --audio-bitrate=192k` — universal compatibility, sufficient quality for voice-over.

## Audio-mux passes (ffmpeg)

For most dispatches, Remotion's built-in audio handling is sufficient — pass the voice-over track as `audioPath` in the composition props and Remotion muxes it.

For cases where you need separate ffmpeg passes (e.g., source video already has audio you want to mix with voice-over, or you need to add background music):

### Voice-over over silent video

```bash
ffmpeg -i video-silent.mp4 -i voiceover.wav \
  -c:v copy -c:a aac -b:a 192k \
  -map 0:v -map 1:a \
  -shortest \
  output.mp4
```

### Voice-over mixed with quiet source audio (e.g., demo recording with mouse clicks)

```bash
ffmpeg -i video-with-source-audio.mp4 -i voiceover.wav \
  -c:v copy \
  -filter_complex "[0:a]volume=0.2[a0]; [1:a]volume=1.0[a1]; [a0][a1]amix=inputs=2:duration=longest[aout]" \
  -map 0:v -map "[aout]" \
  -c:a aac -b:a 192k \
  output.mp4
```

Source audio is ducked to 20% (`volume=0.2`); voice-over plays at full. Adjust to taste.

### Adding a music bed

Generally don't, for short-form. Music beds in 30-second verticals fight the voice-over and distract from the point. If the brief explicitly asks for music, the brief names the music file path (operator-supplied, licensed). The worker doesn't reach for music speculatively.

## Aspect-conversion passes

Re-framing an existing horizontal MP4 to vertical without going back to the source composition:

```bash
ffmpeg -i input-16x9.mp4 \
  -vf "crop=ih*9/16:ih,scale=1080:1920" \
  -c:v libx264 -crf 18 -pixel-format yuv420p \
  -c:a copy \
  output-9x16.mp4
```

This center-crops. For off-center cropping (subject lives in the left third of the frame), adjust `crop=ih*9/16:ih:x=0:y=0` with the x and y offsets.

For 16:9 to 1:1:

```bash
ffmpeg -i input-16x9.mp4 \
  -vf "crop=ih:ih,scale=1080:1080" \
  -c:v libx264 -crf 18 -pixel-format yuv420p \
  -c:a copy \
  output-1x1.mp4
```

Aspect conversion is destructive — pixels are cropped. The brief must accept this, or the orchestrator should have routed the dispatch to the source composition with a different aspect prop instead.

## B-roll insert pattern

For longer cuts (lecture, talking-head) where the brief names "B-roll inserts at 00:12, 00:34, 00:58":

In the Remotion composition, you implement a `<TalkingHeadCut>` shape where the `sections` array allows section types `talking-head` and `b-roll`. The composition renders the primary footage continuously and overlays the B-roll insert at the named frame range (with optional fade-in / fade-out).

```typescript
sections: [
  { type: 'talking-head', startFrame: 0, durationInFrames: 360 },  // 0:00-0:12 at 30fps
  { type: 'b-roll', startFrame: 360, durationInFrames: 90, footagePath: 'assets/b-roll-1.mp4' },
  { type: 'talking-head', startFrame: 450, durationInFrames: 570 },
  // ...
]
```

The audio track plays continuously underneath; only the video toggles between primary and insert.

## Caption-generation pass

For dispatches that produce SRT or VTT (no render):

The worker takes the brief's script + timing breakdown and outputs an SRT file directly. No Remotion, no ffmpeg. Just text.

```srt
1
00:00:00,000 --> 00:00:05,000
Most productivity apps optimize for willpower.

2
00:00:05,000 --> 00:00:11,000
FocusBlock optimizes for scheduling.

3
00:00:11,000 --> 00:00:20,000
By 2pm, the question of whether to focus
has already been decided — by what's on
your calendar.
```

The script's timing breakdown (`[00:00-00:05]` etc.) maps directly to SRT timestamps. Line breaks in the brief are preserved verbatim — readers see the caption shape the brief specified.

For VTT, prepend the `WEBVTT` header and replace `,` with `.` in timestamps.

## Failure modes the worker should catch before render

- **Composition prop incomplete.** Brief is missing a section, the worker can't fill the prop, the render would fail with a TypeScript error. Worker refuses via gate 3 (missing precondition) before invoking Remotion.
- **Footage path doesn't resolve.** Operator-supplied asset at a path that doesn't exist. Worker confirms paths exist before rendering. If not, the brief is wrong; worker stops.
- **Audio file doesn't match duration.** Voice-over track is 45 seconds but the brief asks for 30. Worker checks duration before rendering. If audio is longer, the brief is wrong; worker stops.
- **Aspect mismatch.** Brief says 9:16 but the footage is portrait-cropped to 16:9. Worker checks; if rendering would require lossy aspect conversion, the worker names the conflict and asks.

The pre-flight checklist catches most of these. The render is the last step; everything before it is verification.

## When to extend this file

When you stand up a new composition in your Remotion project (a `SquareSocialVariant`, a `LectureCut`, an aspect-conversion variant), add a section here documenting its prop shape and the render command. The worker reads this file to know what's available; it doesn't introspect the Remotion project on its own.

Keep the sections in order: composition shapes, then render commands, then post-render passes, then failure modes. Slice-not-file Read habit depends on the section headers staying greppable.

---

Last updated: 2026-05-14 (initial release; extend as you build out your composition library).
