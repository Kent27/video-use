# Kent's house style

Read this during the **Converse/Propose** step and apply. These are user
defaults: they override the skill's *artistic-freedom* defaults, **not** the
Hard Rules. Confirm per-video — don't blind-apply.

> **Numbers live in `house-defaults.json`** (same dir). `render.py` auto-reads
> `render.*`, `audio.*`, and `grade.default_preset`. Read the rest of that file
> for exact values (accent, caption params, speed). Edit values *there*, not here.

Only what the skill doesn't already say:

## Defaults that OVERRIDE the skill
- **Aspect:** default to vertical short-form (skill defaults to "match source").
- **Grade:** default `neutral_punch` (skill defaults to none). Also enforced via config.
- **Captions:** clean-phrase style — 3–4 word chunks, **sentence case** (skill ships
  2-word UPPERCASE as default). Lower-third safe zone. **Dual-language** (Indonesian
  primary + English below) — see *Dual-language captions* below.
- **Accent:** assume a single accent color by default (skill says never assume) —
  `look.accent_hex` in config; Google topics → Google blue.

## Workflow
- **Open cold on the hook** — kill leading/trailing dead air; first frame = the hook line.
- **B-roll:** prefer *authentic UI recreations* (HyperFrames) over abstract mockups.
  For brand/product topics, replicate the real product UI.
- **B-roll rhythm (retention):** start a visual at the **hook**. Prefer **fewer, longer**
  B-rolls that each carry a whole concept beat (`broll.prefer_fewer_longer`), with
  **snap-zooms on emphasis** filling between (see *Emphasis zooms*). A zoom counts as
  "something visual," so a bare-but-zoomed stretch is fine; just avoid long flat
  talking-head with neither. Never leave more than ~`broll.body_max_gap_s` of nothing.
- **Always ask my niche/business before writing example queries or business names**
  into graphics. Never invent a niche.

## Mechanisms not in the skill
- **Talking-head speed-up:** default a global ~`pacing.talking_head_speed`× pitch-preserved
  speed-up. There is no helper — apply it as a FINAL ffmpeg pass over the composed video:
  `setpts=PTS/N,fps=<fps>` for video + `atempo=N` for audio. Do it after compositing so
  overlays/captions stay in sync. Confirm the factor per video.
- **Variable speed-up on slow stretches (retention):** beyond the global factor, when
  `pacing.boost_slow_regions` is on, find the low-energy spots — long intra-phrase pauses and
  slow words-per-second runs (read gaps/word timings from the transcript) — and speed *those
  regions* harder (~`pacing.slow_region_speed`×) while leaving punchy lines at the base factor.
  Implement as per-segment tempo in the EDL/extract (a region = its own EDL range with its own
  speed), not one global atempo. Keep cuts on word boundaries; re-sync captions/overlays to the
  retimed segment offsets.

## Emphasis zooms (snap-in)
- On emphasized Indonesian phrases, **snap-zoom** the face: immediate punch-in
  (~`zoom.snap_seconds` 0.12s) to ~`zoom.emphasis_peak` (1.27×), **hold** through the
  phrase, snap back out. NOT a slow ramp.
- **A zoom and a B-roll don't stack** — when a beat gets a zoom, it gets no B-roll, and
  vice-versa.
- The **hook uses B-roll, not a zoom** (`zoom.hook_uses_broll`). Add a short **retention
  zoom** ~2–3s after the hook B-roll (`zoom.retention_after_hook`, pulse length
  `zoom.retention_pulse_seconds`) to re-grab attention.
- Implementation: the render.py **`zoom` field** on an EDL range — a number (whole-segment
  punch) or `{"to": peak, "dur": seconds}` for a short pulse inside a longer beat. It's a
  per-frame **scale + center-crop** (1 frame in → 1 frame out, re-centered every frame).
  **Never `zoompan`** — it stutters on startup and drifts A/V sync.

## Dual-language captions (Indonesian + English)

Render **two SEPARATE caption pills**, same font size (`secondary_scale` 1.0): Indonesian
primary on top (white `primary_color` on a dark pill), English in its own pill a bit below
(`secondary_color` light Google-blue text + `secondary_border` blue outline — clearly marks
the translation). A gap between the two pills. Both inside the lower-third safe zone.

Build order matters — **translate whole, then chunk** (never translate cue-by-cue):
1. Take the full Indonesian transcript and **translate the ENTIRE thing to coherent English
   first** (whole-text, so meaning/flow is right).
2. Chunk that English with the **same rules as Indonesian** (`captions.max_words`, sentence
   case).
3. The two languages **do NOT need to match word-for-word within a frame** — literal
   alignment is often impossible. Time the English cues to the Indonesian timeline at the
   **segment/sentence level** (English progresses alongside Indonesian over the same span),
   not per-word.
4. Render English as a second transparent overlay line just under the Indonesian one; same
   safe-zone, same composite-last rule.

## This machine — env constraint (causes silent failure)
- ffmpeg here is built **without libass/freetype** → the `subtitles`, `ass`, and `drawtext`
  filters DO NOT EXIST. Do **not** use `render.py --build-subtitles` or the EDL `subtitles`
  field. Render captions as a transparent PIL overlay track and composite it as the LAST
  overlay (still satisfies Hard Rule 1).
- (Rotated phone clips are now handled automatically by `render.py` — no manual bake needed.)
