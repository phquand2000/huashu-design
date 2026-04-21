# Video Export: HTML Animations to MP4/GIF

Once an animated HTML is finished, users often ask "can I export it as a video?" This guide gives the complete workflow.

## When to export

**Timing**:
- Animation runs cleanly, visually verified (Playwright screenshots confirm correct state at various timestamps)
- User has watched it in the browser at least once and confirmed the effect looks good
- **Don't** export while animation bugs are still unresolved — fixing things after exporting to video is more expensive

**Trigger phrases users might say**:
- "Can I export this as a video?"
- "Convert it to MP4"
- "Make it a GIF"
- "60fps"

## Output specs

Default: give three formats at once and let the user choose:

| Format | Specs | Best for | Typical size (30s) |
|---|---|---|---|
| MP4 25fps | 1920×1080 · H.264 · CRF 18 | WeChat articles, Video Channel, YouTube | 1–2 MB |
| MP4 60fps | 1920×1080 · minterpolate frame interpolation · H.264 · CRF 18 | High frame rate showcase, Bilibili, portfolios | 1.5–3 MB |
| GIF | 960×540 · 15fps · palette-optimized | Twitter/X, README, Slack previews | 2–4 MB |

## Toolchain

Two scripts in `scripts/`:

### 1. `render-video.js` — HTML → MP4

Records a 25fps MP4 base version. Depends on global playwright.

```bash
NODE_PATH=$(npm root -g) node /path/to/claude-design/scripts/render-video.js <html-file>
```

Optional parameters:
- `--duration=30` animation duration (seconds)
- `--width=1920 --height=1080` resolution
- `--trim=2.2` seconds to trim from the start of the video (removes reload + font loading time)
- `--fontwait=1.5` font loading wait time (seconds) — increase when there are many fonts

Output: same directory as HTML, same name with `.mp4` extension.

### 2. `add-music.sh` — MP4 + BGM → MP4

Mixes background music into a silent MP4 — choose by mood from the built-in BGM library, or bring your own audio. Automatically matches duration and applies fade in/out.

```bash
bash add-music.sh <input.mp4> [--mood=<name>] [--music=<path>] [--out=<path>]
```

**Built-in BGM library** (in `assets/bgm-<mood>.mp3`):

| `--mood=` | Style | Best for |
|-----------|------|---------|
| `tech` (default) | Apple Silicon / Apple keynote vibe, minimal synths + piano | Product launches, AI tools, skill promos |
| `ad` | Upbeat modern electronic, with build + drop | Social media ads, product teasers, promo reels |
| `educational` | Warm and bright, light guitar/electric piano, inviting | Science explainers, tutorials, course previews |
| `educational-alt` | Same category, alternate track | Same as above |
| `tutorial` | Lo-fi ambient, nearly invisible | Software demos, coding tutorials, long walkthroughs |
| `tutorial-alt` | Same category, alternate track | Same as above |

**Behavior**:
- Music is trimmed to match video duration
- 0.3s fade in + 1s fade out (no hard cuts)
- Video stream `-c:v copy` (no re-encode), audio AAC 192k
- `--music=<path>` takes priority over `--mood` — you can point directly to any external audio
- Passing a wrong mood name lists all available options — no silent failure

**Typical pipeline** (three-piece animation export + music):
```bash
node render-video.js animation.html                        # record
bash convert-formats.sh animation.mp4                      # generate 60fps + GIF
bash add-music.sh animation-60fps.mp4                      # add default tech BGM
# Or for specific scenarios:
bash add-music.sh tutorial-demo.mp4 --mood=tutorial
bash add-music.sh product-promo.mp4 --mood=ad --out=promo-final.mp4
```

### 3. `convert-formats.sh` — MP4 → 60fps MP4 + GIF

Generates a 60fps version and GIF from an existing MP4.

```bash
bash /path/to/claude-design/scripts/convert-formats.sh <input.mp4> [gif_width] [--minterpolate]
```

Output (same directory as input):
- `<name>-60fps.mp4` — defaults to `fps=60` frame duplication (broad compatibility); add `--minterpolate` for high-quality interpolation
- `<name>.gif` — palette-optimized GIF (960px wide by default, adjustable)

**60fps mode selection**:

| Mode | Command | Compatibility | Use case |
|---|---|---|---|
| Frame duplication (default) | `convert-formats.sh in.mp4` | QuickTime/Safari/Chrome/VLC all work | General delivery, platform uploads, social media |
| minterpolate | `convert-formats.sh in.mp4 --minterpolate` | macOS QuickTime/Safari may refuse to play | Bilibili and similar platforms where true interpolation is needed — **must test on target player locally before delivering** |

Why frame duplication became the default: minterpolate's H.264 elementary stream output has a known compat bug — the old default of minterpolate repeatedly caused "macOS QuickTime won't open" issues. See `animation-pitfalls.md` §14 for details.

`gif_width` parameter:
- 960 (default) — works for general social platforms
- 1280 — sharper but larger file
- 600 — Twitter/X priority loading

## Full workflow (standard recommendation)

When the user says "export video":

```bash
cd <project-directory>

# Assuming $SKILL points to this skill's root directory (replace with your install path)

# 1. Record 25fps base MP4
NODE_PATH=$(npm root -g) node "$SKILL/scripts/render-video.js" my-animation.html

# 2. Generate 60fps MP4 and GIF
bash "$SKILL/scripts/convert-formats.sh" my-animation.mp4

# Outputs:
# my-animation.mp4         (25fps · 1–2 MB)
# my-animation-60fps.mp4   (60fps · 1.5–3 MB)
# my-animation.gif         (15fps · 2–4 MB)
```

## Technical details (for troubleshooting)

### Playwright recordVideo gotchas

- Frame rate is fixed at 25fps — no direct 60fps recording (Chromium headless compositor limit)
- Recording starts when the context is created — must use `trim` to cut out the leading load time
- Default output is webm format — needs ffmpeg conversion to H.264 MP4 for universal playback

`render-video.js` already handles all of the above.

### ffmpeg minterpolate parameters

Current config: `minterpolate=fps=60:mi_mode=mci:mc_mode=aobmc:me_mode=bidir:vsbmc=1`

- `mi_mode=mci` — motion compensation interpolation
- `mc_mode=aobmc` — adaptive overlapped block motion compensation
- `me_mode=bidir` — bidirectional motion estimation
- `vsbmc=1` — variable size block motion compensation

Works well for CSS **transform animations** (translate/scale/rotate).
May produce slight **ghosting on pure fades** — if the user objects, fall back to simple frame duplication:

```bash
ffmpeg -i input.mp4 -r 60 -c:v libx264 ... output.mp4
```

### Why GIF palette requires two passes

GIF supports only 256 colors. A single-pass GIF compresses the entire animation's palette down to 256 generic colors — on subtle color palettes like cream backgrounds with orange accents, this gets muddy.

Two passes:
1. `palettegen=stats_mode=diff` — scan the full video first, generate the **optimal palette for this specific animation**
2. `paletteuse=dither=bayer:bayer_scale=5:diff_mode=rectangle` — encode with that palette; rectangle diff only updates changed regions, greatly reducing file size

For fade transitions, `dither=bayer` is smoother than `none`, but slightly larger file.

## Pre-flight check (before exporting)

30-second self-check before exporting:

- [ ] HTML ran through completely in the browser, no console errors
- [ ] Frame 0 of the animation is the complete initial state (not a blank loading screen)
- [ ] Last frame of the animation is a stable final state (not cut off mid-way)
- [ ] Fonts/images/emoji all rendering correctly (ref `animation-pitfalls.md`)
- [ ] Duration parameter matches actual animation length in the HTML
- [ ] HTML Stage detection `window.__recording` forces loop=false (check any hand-written Stage; `assets/animations.jsx` includes this automatically)
- [ ] Trailing Sprite has `fadeOut={0}` (last frame of video doesn't fade out)
- [ ] Includes "Created by Huashu-Design" watermark (required for animation scenarios only; for third-party branded work, prepend "Unofficial · ". See SKILL.md § "Skill Promotion Watermark")

## Delivery notes to include

Standard note format to give the user after export:

```
**Complete Delivery**

| File | Format | Specs | Size |
|---|---|---|---|
| foo.mp4 | MP4 | 1920×1080 · 25fps · H.264 | X MB |
| foo-60fps.mp4 | MP4 | 1920×1080 · 60fps (motion interpolation) · H.264 | X MB |
| foo.gif | GIF | 960×540 · 15fps · palette-optimized | X MB |

**Notes**
- 60fps uses minterpolate motion estimation — works great for transform animations
- GIF uses palette optimization — a 30s animation compresses to around 3MB

Let me know if you want a different size or frame rate.
```

## Common follow-up requests

| User says | Response |
|---|---|
| "Too large" | MP4: increase CRF to 23–28; GIF: lower resolution to 600 or fps to 10 |
| "GIF is too blurry" | Increase `gif_width` to 1280; or suggest using MP4 instead (WeChat Moments also supports it) |
| "Need portrait 9:16" | Change HTML source with `--width=1080 --height=1920` and re-record |
| "Add a watermark" | ffmpeg with `-vf "drawtext=..."` or overlay a PNG with `overlay=` |
| "Need transparent background" | MP4 doesn't support alpha; use WebM VP9 + alpha or APNG |
| "Need lossless" | Set CRF to 0 + preset veryslow (file will be 10x larger) |
