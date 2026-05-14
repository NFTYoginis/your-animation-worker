# reference/visual-system.md

This is the file the worker reads every dispatch to know how the rendered MP4 should look. Palette, type, motion, transitions, caption style.

You (the reader who forked this repo) edit this file with your brand's visual rules. The worker uses whatever's here. The default below is a working placeholder — neutral, runnable for the worked example, intentionally bland so you replace it with your own.

**Replace it with your own visual system once you've seen the worker behave.**

---

## Visual system — default placeholder

> Replace this with your brand's palette and type system. The worker checks for the literal phrase "Replace this with your brand's palette and type system" — leave the phrase in this file, the gate fires and refuses to render. Delete the phrase (after editing) to flip the gate to "configured."

### Palette

- **Primary type color:** `#16151a` (near-black, on light backgrounds) / `#ecebe6` (ivory, on dark backgrounds)
- **Background — light mode renders:** `#fbfaf7` (paper white)
- **Background — dark mode renders:** `#0a0a0c` (film black)
- **Accent (one — used for the title-card highlight, lower-third caption rule, CTA URL underline):** `#d6913a` (amber)
- **Caption shadow:** `#000000` at 60% opacity, 2px offset, lower-right

Pick ONE accent. More than one accent makes a 30-second vertical visually noisy.

### Typography

- **Primary type (titles, opening line):** Inter (or system sans). Weight 700 for titles, 600 for opening line. Letter-spacing -0.015em on titles.
- **Caption type (burned-in lower-thirds):** Inter (or system sans). Weight 600. White with the caption shadow above.
- **Mono type (URL CTAs, timecodes, technical labels):** JetBrains Mono (or system mono). Weight 500.

If your brand has a specific licensed font, replace these. Remotion can use any font registered via `@remotion/google-fonts` or a `<link>` in the composition.

### Type sizes (at 1080×1920 vertical)

- **Title card text:** 96px at 1080px wide. Centered.
- **Caption text:** 64px, lower-third position (180px above bottom edge of frame). Max 22 characters per line, max 2 lines per cue.
- **URL CTA:** 56px monospace, centered, with a 2px underline in the accent color.

For 16:9 horizontal (1920×1080), scale these down ~30%. For 1:1 square (1080×1080), match the vertical sizes but reposition captions to centered-middle-third (since lower-third clips on square).

### Motion rules

- **Title card:** 0.4-second fade-in, 1.5-second hold, 0.3-second fade-out as the first footage cue enters.
- **Footage cuts:** Hard cuts between sections. No cross-fades unless the brief explicitly asks for them — cross-fades in 30-second verticals waste attention.
- **Zoom:** Maximum 1.0x to 1.10x on any single footage cue (slow push-in). No zoom-out, no rapid zoom, no camera-style movement that wasn't in the source footage.
- **Caption appearance:** Instant. No type-on, no character-by-character, no flicker. Caption appears, holds for its cue duration, disappears at the next cue boundary.
- **Closing frame:** 0.5-second fade-in for the URL. 1.5-second hold. End.

### Transition rules

- Between sections within a single dispatch: hard cuts only.
- Between a title card and the first footage cue: 0.3-second fade-out of the title card while the footage cue starts at frame 1 of its clip.
- Between footage and the closing CTA frame: hard cut.

No video transitions beyond fade. No wipes, no slides, no zoom transitions, no glitch effects.

### Captions

- Always burned-in for short-form social (under 60 seconds). Most viewers watch muted.
- For long-form (over 60 seconds, e.g., a lecture cut), generate a soft-track SRT or VTT and let the platform render it.
- Caption text uses the script verbatim — do not paraphrase the script for shorter caption lines. If the script doesn't fit two lines at the type size above, the script is too long; iterate on the brief.

---

## TODO for the operator (edit these before serious dispatch)

- [ ] Replace the placeholder gate phrase above (once you've replaced the palette).
- [ ] Replace the palette section with your brand's colors. Keep it to one accent.
- [ ] Replace the typography section with your brand's fonts. Provide font-loading details if they're not system fonts.
- [ ] Adjust type sizes if your most-common aspect ratio isn't 1080×1920 vertical.
- [ ] Refine the motion and transition rules to match your brand's tempo. Faster cuts? Add a rule. Slow push-ins? Add the cue.
- [ ] Add a "what this visual system looks like" gallery — paths to two or three previously-rendered MP4s that exemplify it. The worker won't watch them, but you'll use them when you iterate on this file.

Once those edits are made, this file is yours. The worker will read it on every dispatch and render to it.

## What NOT to put in this file

- Footage sources (Pexels keys, library paths). (Different file: `reference/footage-sources.md`.)
- Voice settings (TTS engine, voice-clone provider). (Brief-level, not visual-system-level.)
- Render commands (Remotion CLI flags, ffmpeg passes). (Different file: `reference/remotion-pipeline.md`.)
- Brand history, voice / tone for scripts. (Different worker: `your-content-worker`.)

Visual system = how the frames look. Anything else lives in its own reference file.

---

Last updated: 2026-05-14 (default placeholder; replace with your own system).
