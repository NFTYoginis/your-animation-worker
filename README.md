# your-animation-worker

A voice-to-video Claude worker, ICM-structured, fork-ready. Dispatched a brief, renders a short MP4 via Remotion. MIT licensed.

Live landing page (once you've pushed and enabled Pages): `https://<your-username>.github.io/your-animation-worker/`

---

## Why this exists

There are other Claude-worker starter kits. This one is opinionated about five things, each enforced by the architecture rather than by README aspiration:

1. **ICM rigor.** The three always-relevant files — `CLAUDE.md`, `CONTEXT.md`, `STATUS.md` — plus `identity.md`, `rules.md`, `examples.md`, and `reference/`. Not "a CLAUDE.md and hope." Structural, named, enforced.
2. **60-30-10 layer separation.** Infrastructure / Orchestration / AI, in the corrected definition (see `reference/icm-layer-model.md`). AI is the 10%; the other 90% lives in Remotion, ffmpeg, your footage source, your hosting — outside Claude. Most kits make Claude the whole stack. This one keeps it in its lane.
3. **Brief-as-contract dispatch.** Clean role boundary between orchestrator and worker. The worker is meant to be called, not driven free-form. The brief is durable, audit-trail-friendly, and complete-or-refused — no "I'll do my best with what's missing." Especially important for video, where wrong renders cost real money in Pexels quota + voice-clone API charges + render compute.
4. **Pages-ready landing.** Every repo IS its own marketing surface. Push it, enable GitHub Pages from `/docs`, public URL in sixty seconds.
5. **Tied to a real article series.** The architecture is documented in the operator-AI series, starting with "I burned 800,000 tokens on one daily routine." Read the article for the receipts; read this repo for the code that runs the fix.

The article: [Article 1 — destination TBD]. Update this link once the Medium URL is live.

---

## What this worker does

| Job | Input | Output |
| --- | --- | --- |
| Voice-over short | Brief naming script, duration, aspect, voice source, footage cues | One MP4 in the configured visual system |
| Product demo cut | Brief naming script, screen-recording paths, key moments | One demo MP4, audio-muxed, aspect-correct |
| Square social variant | Brief naming source MP4 + re-frame / re-time instructions | One 1:1 MP4 cut, captioned if asked |
| Caption / subtitle pass | Brief naming source MP4 + script + timing | One SRT or VTT file. No render. |
| Lecture / talking-head cut | Brief naming primary footage + B-roll cues + arc | One MP4 with primary + B-roll inserts |

Full job definitions in [identity.md](identity.md). Routing logic in [CONTEXT.md](CONTEXT.md). Pipeline reference (composition shapes, render commands, ffmpeg passes) in [reference/remotion-pipeline.md](reference/remotion-pipeline.md).

---

## Five-minute setup

### 1. Fork or clone

```bash
git clone https://github.com/NFTYoginis/your-animation-worker.git
cd your-animation-worker
```

Or click "Fork" on GitHub if you want your own copy under your account.

### 2. Stand up Remotion alongside this worker

The repo does NOT bundle Remotion (pinned dependencies rot fast). Stand it up in a sibling folder:

```bash
# In a sibling folder, e.g., ~/Code/animation-render/
npx create-video@latest

# Or manually:
npm init -y
npm install remotion @remotion/cli @remotion/google-fonts @remotion/media-utils
npx remotion init
```

You'll also need `ffmpeg` installed locally (`brew install ffmpeg` on macOS, `apt install ffmpeg` on Linux). See [reference/remotion-pipeline.md](reference/remotion-pipeline.md) for the composition shapes the worker expects.

### 3. Configure the visual system and footage source

Two files to edit:

- [reference/visual-system.md](reference/visual-system.md) — palette, typography, motion rules, transitions. Replace the placeholder.
- [reference/footage-sources.md](reference/footage-sources.md) — Pexels API key (env-loaded), local footage library path, or per-brief asset slots. Replace the placeholder.

The worker refuses to render if either file still shows its placeholder text. The gates are there to prevent rendering against an unconfigured stack — get them green before the first dispatch.

### 4. Open in a Claude session

Either:

- **Claude Code** in the terminal: `cd` into the folder, run `claude`. The worker reads `CLAUDE.md` as its entry point.
- **Claude Project** (claude.ai): create a project, upload the folder. Same entry point.

### 5. Write your first brief

Copy `briefs/_BRIEF-TEMPLATE.md` to `briefs/<today>-<slug>.md`. Fill the seven sections:

1. What to produce
2. Audience and platform
3. Script + script timing
4. Visual direction
5. Voice source
6. Success criteria
7. Refusal context (if relevant)

The brief should fit on one screen. If yours is sprawling, you're orchestrating in the wrong place — split it.

### 6. Dispatch

Paste this into the Claude session:

```
Read the brief at briefs/<your-filename>.md and execute.
```

That's the entire dispatch. The worker takes it from there: reads STATUS, reads the brief, reads the visual system + footage source, runs the pre-flight checklist (six items for animation, one more than the content worker because voice source is per-brief), constructs the composition props, invokes Remotion, self-verifies, updates STATUS, hands back the path to the rendered MP4.

---

## File map

```
your-animation-worker/
├── README.md             ← you are here
├── CLAUDE.md             ← worker entry point (first read every session)
├── CONTEXT.md            ← routing / load logic — the 30% orchestration made explicit
├── STATUS.md             ← first read, last write (you maintain)
├── identity.md           ← who the worker is, who it serves, what it does
├── rules.md              ← always / never / refusal gates / escalation
├── examples.md           ← worked example — study to understand worker behavior
├── briefs/
│   ├── _BRIEF-TEMPLATE.md       ← copy this for each dispatch
│   └── 2026-05-14-example.md    ← paired with the worked example in examples.md
├── reference/
│   ├── icm-layer-model.md       ← 60-30-10, corrected definition, attributed to Van Clief
│   ├── dispatch-pattern.md      ← orchestrator-worker boundary, explained for video specifically
│   ├── visual-system.md         ← YOU EDIT THIS — palette, type, motion rules
│   ├── footage-sources.md       ← YOU EDIT THIS — Pexels key / library path / per-brief slots
│   └── remotion-pipeline.md     ← composition shapes, render commands, ffmpeg passes (read in slices)
├── docs/
│   └── index.html               ← Pages-ready landing page
├── LICENSE                      ← MIT
└── .gitignore                   ← includes *.mp4, output/, .env (rendered MP4s stay local)
```

---

## What this worker doesn't do

The full list is in [rules.md](rules.md). Headline items:

- Doesn't orchestrate. Doesn't pick its own topics, set its own publish dates, or dispatch other workers.
- Doesn't invent facts. If a brief asks for a claim the brief doesn't ground, the worker writes a question file and stops.
- Doesn't render in an unconfigured visual system. Empty `reference/visual-system.md` or unedited placeholder triggers a refusal.
- Doesn't pull footage from un-configured sources. Pexels (with the configured API key) or operator-supplied assets only — no scraping random footage off the internet.
- Doesn't clone a real third party's voice without explicit consent confirmation in the brief.
- Doesn't deepfake real faces. Operator's own face or stock people in licensed footage only.
- Doesn't fabricate product demos. A demo can only show capabilities the operator confirms exist.
- Doesn't render content for high-harm domains without explicit operator authorization in the brief.
- Doesn't extend scope. One brief, one render. No bonus variants.

When any refusal gate fires, the worker writes `briefs/questions/<slug>-question.md` with verbatim quotes from the brief and the specific information needed to proceed. Then it stops.

---

## A note on MP4s and repo size

Rendered MP4s are not committed. They're gitignored (`*.mp4`, `output/`) for repo hygiene — a 30-second vertical at h.264 CRF 18 is roughly 8-15MB, and a year of weekly renders would balloon the repo past comfortable clone size.

Where the MP4s actually live:

- **The operator's local disk** at `output/<date>-<slug>.mp4`. Always.
- **The operator's CDN / YouTube / S3 / wherever they publish.** Per the dispatch's deployment step.
- **For this repo's example specifically:** an operator-supplied public URL referenced from `docs/index.html`'s `<video>` tag (currently a `<!-- TODO: operator -->` placeholder until wired up).

The repo carries render *receipts* (the props, the command, the verification table) — not the binaries. Anyone can reproduce a render by standing up Remotion + Pexels + a TTS engine and re-dispatching the paired brief.

---

## Push to GitHub + enable Pages

```bash
# 1. Initialize the repo (if you haven't already)
git init
git add .
git commit -m "Initial commit"

# 2. Create the remote and push (assumes gh CLI authenticated)
gh repo create <your-username>/your-animation-worker --public --source=. --remote=origin --push

# 3. Enable GitHub Pages serving from /docs
gh api repos/<your-username>/your-animation-worker/pages \
  -X POST \
  -f "source[branch]=main" \
  -f "source[path]=/docs"

# 4. Wait ~30-60 seconds, then verify
open https://<your-username>.github.io/your-animation-worker/
```

If `gh` CLI isn't authenticated, use the GitHub web UI: Settings → Pages → Source = `main` branch, `/docs` folder → Save.

---

## The series

This repo is one of three in the operator-stack series. Same architecture, different domains:

- **[your-content-worker](https://github.com/NFTYoginis/your-content-worker)** — prose
- **[your-design-worker](https://github.com/NFTYoginis/your-design-worker)** — images, HTML previews, social variants
- **your-animation-worker** (this repo) — voice-to-video MP4 via Remotion

The three pair naturally — content writes the script, design produces the title-card still, animation renders the video.

---

## Article series

The architecture is documented in:

- **Article 1** — "I burned 800,000 tokens on one daily routine. Here's the architecture that killed it." [Medium URL TBD]

Future articles will link from this section as they publish.

---

## License

MIT. See [LICENSE](LICENSE). Replace `<YOUR NAME OR HANDLE>` in the copyright line when you fork.

## Acknowledgments

- The 60-30-10 layer framework is from Jake Van Clief's ICM (Internal Coherence Maximization) teaching.
- The Remotion-based render pipeline pattern is adapted from an in-house voice-to-video worker the maintainer built for their own daily routine; the public version here is sanitized of brand-specific assets and configured for fork-ready neutrality.
- The `docs/index.html` Pages-ready discipline is adapted from the existing public landing-page pattern in this maintainer's earlier worker repositories.

---

Last updated: 2026-05-14.
