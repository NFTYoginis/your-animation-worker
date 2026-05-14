# Brief — <DATE>-<SLUG>

The brief is the worker's contract. Copy this template to `briefs/<YYYY-MM-DD>-<slug>.md` and fill it in. If a section is genuinely N/A, write "N/A — [one-line reason]" rather than deleting the heading.

---

## 1. What to produce

The deliverable, in concrete terms. The worker should be able to grep this section and know exactly what to render.

**Examples of complete answers:**
- "One 30-second vertical (9:16) voice-over MP4 for a product launch. Stock-footage background, one piece of B-roll. Captions burned in."
- "Three 15-second social cuts of the source MP4 at `output/2026-05-14-demo.mp4`, re-framed for square (1:1)."
- "A 90-second product demo MP4. Operator-supplied screen recording at `assets/demo-recording.mp4`. Voice-over from a TTS pass on the script below."

**Examples of incomplete answers:**
- "A short video about the launch." (Duration? Aspect? Voice source? Footage source?)
- "Cut this for socials." (Which aspect? Which platforms? Which sections?)

---

## 2. Audience and platform

Who watches this. Where they watch it. What format the platform expects.

**Useful answer:** "Solo founders scrolling LinkedIn on desktop. Vertical 9:16 plays small (about 320px wide in the feed), so the script's key visual moments need to read at that size. Average watch-through on this account's LinkedIn videos is 22 seconds; design the script so the point lands before 18 seconds."

**Useless answer:** "Anyone on social media."

---

## 3. Script + script timing

The exact text of the voice-over (or burned-in captions for a silent cut). For a 30-second voice-over, target roughly 70-90 words; for a 60-second, 140-180 words. The brief is the contract — paste the script verbatim, with intended emphasis marked.

Format suggestion:
```
[00:00-00:08]  Opening line. The hook. Carries the first eight seconds.
[00:08-00:20]  Middle. The point. The visual cue plays under this.
[00:20-00:30]  Close. Specific CTA. URL or next action.
```

If the cut has no voice-over (silent text-over-footage), paste the burned-in caption sequence here with the same timing format.

---

## 4. Visual direction

Aspect ratio. Footage cues per section (Pexels keywords, or asset filenames the operator has supplied). Title-card spec if there is one. Caption style (burned-in / hard / soft / none).

**Example:**
```
Aspect: 9:16 (1080×1920)
Title card: 1.5 seconds, "FocusBlock 1.0 — Today"
Section 1 footage: Pexels search "person typing at laptop morning light," 1 clip, 8 seconds
Section 2 footage: Pexels search "calendar on screen close up," 1 clip, 12 seconds
Section 3 footage: Title card with URL, no footage
Captions: burned-in, white sans-serif, lower-third, 2-second hold per line
```

If the operator has supplied a specific asset (their own screen recording, their own footage), reference it by repo-relative path: `assets/demo-take-3.mp4`.

---

## 5. Voice source

Either:
- "Use TTS — engine: [Eleven Labs / OpenAI tts-1-hd / etc.], voice ID: [provider's voice name or ID, env-key referenced not embedded]."
- "Use operator-recorded audio at `assets/voice/<file>.wav`."
- "Use voice clone — provider: [name], voice profile: [name], consent record: [reference to the consent confirmation]." (Triggers refusal gate 4 if consent isn't shown.)

For an example dispatch, TTS is the safest default. Voice clones live behind the refusal gate; configure them carefully.

---

## 6. Success criteria

What "rendered" looks like, in concrete terms the worker can verify itself against before handing back.

**Examples:**
- "Duration between 28 and 32 seconds. Aspect 9:16, 1080×1920, h.264, AAC audio at 192k. Captions burned in and readable at 320px scaled width. Title card reads correctly. No footage attributable to an un-configured source."
- "Three SRT files for the three social cuts, timestamp-accurate to source, line-broken to fit one line at 22 characters."
- "Demo MP4 audio-muxed cleanly (no clicks at cut points). Screen-recording readable at 1080p playback. Length within 5 seconds of target."

Make this list grep-able. The worker will check each item before declaring rendered.

---

## 7. Refusal context (if relevant)

If this brief touches a domain where the worker needs extra care — voice-cloning of a real person, deepfake-adjacent imagery, claims requiring sources, high-harm domains, regulatory considerations — note that here. The worker uses this section to decide whether to apply any of the refusal gates from `rules.md`.

If the brief uses a voice clone of a named third party, this section MUST include the line:

> consent-confirmed: I have authorization from [name] to publish content using their voice, recorded at [reference to consent record].

If the brief is in a high-harm domain (medical claims, political microtargeting, regulated financial promotion, mental-health crisis content), this section MUST include the line:

> operator-authorized: I have reviewed the regulatory and harm context for this domain, and I authorize the worker to render content here.

Without those lines where applicable, the worker refuses.

Otherwise, leave this section empty or write "N/A — no third-party voice clone, no high-harm domain, no claims requiring external sources."

---

## Notes for the operator (delete this section before dispatch)

- A complete brief should fit on one screen. If yours is sprawling, you're orchestrating in the wrong place.
- The brief is permanent. Once dispatched, it lives in `briefs/` as the contract for what was rendered. Don't edit it after the fact.
- If you find yourself writing the brief twice for similar dispatches, the duplicated content probably belongs in a `reference/` file, not in every brief.
- Render time, footage cost, and voice-clone API charges add up faster than prose-generation cost. Get the brief right before dispatching — re-renders are not free.
