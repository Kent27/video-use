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
  2-word UPPERCASE as default). Lower-third safe zone.
- **Accent:** assume a single accent color by default (skill says never assume) —
  `look.accent_hex` in config; Google topics → Google blue.

## Workflow
- **Open cold on the hook** — kill leading/trailing dead air; first frame = the hook line.
- **B-roll:** prefer *authentic UI recreations* (HyperFrames) over abstract mockups.
  For brand/product topics, replicate the real product UI.
- **Always ask my niche/business before writing example queries or business names**
  into graphics. Never invent a niche.

## Mechanisms not in the skill
- **Talking-head speed-up:** default a global ~`pacing.talking_head_speed`× pitch-preserved
  speed-up. There is no helper — apply it as a FINAL ffmpeg pass over the composed video:
  `setpts=PTS/N,fps=<fps>` for video + `atempo=N` for audio. Do it after compositing so
  overlays/captions stay in sync. Confirm the factor per video.

## This machine — env constraint (causes silent failure)
- ffmpeg here is built **without libass/freetype** → the `subtitles`, `ass`, and `drawtext`
  filters DO NOT EXIST. Do **not** use `render.py --build-subtitles` or the EDL `subtitles`
  field. Render captions as a transparent PIL overlay track and composite it as the LAST
  overlay (still satisfies Hard Rule 1).
- (Rotated phone clips are now handled automatically by `render.py` — no manual bake needed.)
