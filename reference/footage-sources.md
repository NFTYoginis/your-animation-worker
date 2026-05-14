# reference/footage-sources.md

This is the file the worker reads every dispatch to know where the footage comes from.

You (the reader who forked this repo) edit this file with your footage source configuration. The worker uses whatever's here. The default below is a placeholder — the worker refuses to render against it.

**Configure your footage source before the first real dispatch.**

---

## Footage source — default placeholder

> Configure your footage source here. The worker checks for the literal phrase "Configure your footage source here" — leave the phrase in this file, the gate fires and refuses to render. Delete the phrase (after configuring) to flip the gate to "configured."

### Option A — Pexels API (most common)

If you're using stock footage from Pexels:

1. **Get a Pexels API key.** Free tier: `https://www.pexels.com/api/`. Generate the key on your own account.
2. **Store the key in an environment variable.** Use `.env` in this folder (gitignored — see `.gitignore`). The variable name the worker expects:

   ```
   PEXELS_API_KEY=your_key_here
   ```

3. **Do not commit `.env`.** Already gitignored. Confirm by running `git status` after creating it — it should not appear in untracked files.
4. **Confirm your Pexels license terms.** Pexels content is generally free for commercial use, but check the per-clip attribution requirements and the rate-limit terms before dispatching large batches.

The worker queries Pexels through its API using the keywords each brief names in section 4 (Visual direction). One query per cue, best match, render. No speculative comparison queries.

**Rate limits:** Pexels free tier allows ~200 requests per hour and 20,000 per month. Each cue in a dispatch is one request. A 30-second vertical with 3 cues = 3 requests. Be miserly. If a brief comes in with 10 cues, the orchestrator can probably consolidate it down to 4-5 — and should.

### Option B — Local footage library

If you license footage from a paid stock provider (Artgrid, Storyblocks, Adobe Stock, etc.) and keep clips locally:

1. **Pick a folder path.** Common pattern: `~/Footage/<provider>/<clip-id>.mp4`.
2. **Tell the worker where it is.** Edit the line below:

   ```
   FOOTAGE_LIBRARY_PATH=/absolute/path/to/your/footage
   ```

3. **Inside that folder, organize by keyword tag.** The worker will look for clips matching the brief's footage cues. A simple convention: filename includes the searchable keywords (e.g., `person-typing-laptop-morning.mp4`).
4. **Keep a license record per clip.** A markdown file at the root of your library — `LICENSES.md` — recording where each clip came from, when, under what license, and the renewal date if applicable. The worker doesn't read this; you do, when an auditor or platform asks.

### Option C — Per-brief asset slots

If the brief itself provides the footage assets (operator-supplied screen recordings, operator's own talking-head footage, licensed clips delivered for a specific project), the brief's "Visual direction" section names the paths directly:

```
Section 1 footage: assets/demo-take-3.mp4 (operator-supplied, 0:00-0:08 of source)
Section 2 footage: assets/screen-record-2026-05-12.mp4 (full clip, 9 seconds)
```

For this option, you don't configure anything in this file beyond noting that per-brief assets are an acceptable source. The worker reads the brief, opens the asset paths, renders.

You can combine all three options. A typical dispatch uses Pexels for B-roll, a local library for branded establishing shots, and per-brief assets for the operator's own footage.

---

## TODO for the operator (configure before first real dispatch)

- [ ] Pick your default option (A, B, C, or combination).
- [ ] For option A: get a Pexels key, store it in `.env`, confirm `.env` is gitignored.
- [ ] For option B: name the absolute path to your footage folder, organize the folder, write the licenses log.
- [ ] For option C: confirm the brief template's "Visual direction" section names paths the worker can resolve.
- [ ] Delete the placeholder gate phrase above.

Once those steps are done, this file is configured. The worker will read it on every dispatch and source footage from where you said.

## What NOT to put in this file

- API keys themselves. (Use `.env`, gitignored. This file references the variable name, not the value.)
- Visual style rules (colors, typography, motion). (Different file: `reference/visual-system.md`.)
- Voice source configuration. (Brief-level, not footage-level.)

Footage source = where the visual content comes from. Anything else lives in its own reference file.

## A note on rights and attribution

You are responsible for the licensing of every clip the worker renders into your MP4. Pexels is generally permissive but has attribution rules for some content. Your paid stock licenses have their own terms. The worker enforces "footage from configured sources only" because the alternative — scraping random footage off the web — exposes you to copyright claims that will arrive months after publication.

Configure carefully. Keep the licenses record. The worker is a tool; the rights are your responsibility.

---

Last updated: 2026-05-14 (default placeholder; configure before dispatch).
