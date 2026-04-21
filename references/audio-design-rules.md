# Audio Design Rules · huashu-design

> The audio recipe for all animation demos. Use alongside `sfx-library.md` (the asset catalog).
> Battle-tested: huashu-design hero v1–v9 iteration · Gemini deep-dive of Anthropic's three official films · 8000+ A/B comparisons

---

## Core Principle · Dual-Track Audio (Iron Rule)

Animation audio **must be designed as two independent layers** — doing only one layer is not acceptable:

| Layer | Role | Time Scale | Relationship to Visuals | Frequency Range |
|---|---|---|---|---|
| **SFX (beat layer)** | Marks every visual beat | 0.2–2s short bursts | **Strong sync** (frame-level alignment) | **High freq 800Hz+** |
| **BGM (ambient bed)** | Emotional foundation, soundscape | Continuous 20–60s | Weak sync (segment-level) | **Mid-low freq <4kHz** |

**An animation with only BGM is crippled** — the audience subconsciously senses "things are moving on screen but there's no sound responding," and that's the root of the cheap feel.

---

## Gold Standard · Golden Ratios

These numbers come from empirically testing Anthropic's three official films + our own v9 final cut comparison — they are **hard engineering parameters** you can apply directly:

### Volume
- **BGM volume**: `0.40–0.50` (relative to full scale 1.0)
- **SFX volume**: `1.00`
- **Loudness difference**: BGM peak **-6 to -8 dB** below SFX (the prominence comes from the loudness gap, not from SFX absolute loudness)
- **amix parameter**: `normalize=0` (never use normalize=1 — it flattens the dynamic range)

### Frequency Band Isolation (P1 Hard Optimization)
Anthropic's secret isn't "louder SFX" — it's **frequency band separation**:

```bash
[bgm_raw]lowpass=f=4000[bgm]      # BGM restricted to mid-low freq <4kHz
[sfx_raw]highpass=f=800[sfx]      # SFX pushed to mid-high freq 800Hz+
[bgm][sfx]amix=inputs=2:duration=first:normalize=0[a]
```

Why: The human ear is most sensitive to the 2–5kHz range (the "presence band"). If SFX is all in this range and BGM covers the full spectrum, **the high-frequency parts of BGM will mask the SFX**. Using highpass to push SFX higher + lowpass to push BGM down means each occupies its own frequency territory, and SFX clarity jumps up a tier.

### Fade
- BGM in: `afade=in:st=0:d=0.3` (0.3s, avoids a hard cut)
- BGM out: `afade=out:st=N-1.5:d=1.5` (1.5s long tail, sense of convergence)
- SFX has its own envelope — no additional fade needed

---

## SFX Cue Design Rules

### Density (SFX Count per 10 Seconds)
Empirical measurement of Anthropic's three films shows three tiers:

| Film | SFX / 10s | Product Personality | Scenario |
|---|---|---|---|
| Artifacts (ref-1) | **~9 per 10s** | Feature-dense, high information | Complex tool demo |
| Code Desktop (ref-2) | **0** | Pure ambient, meditative | Developer tool focus state |
| Word (ref-3) | **~4 per 10s** | Balanced, office rhythm | Productivity tool |

**Heuristics**:
- Product personality: calm/focused → low SFX density (0–3 per 10s), BGM-led
- Product personality: lively/information-heavy → high SFX density (6–9 per 10s), SFX drives the rhythm
- **Don't fill every visual beat** — negative space is more sophisticated than density. **Deleting 30–50% of cues makes what's left more dramatic.**

### Cue Selection Priority
Not every visual beat needs a SFX. Select by this priority:

**P0 Required** (omitting creates a jarring gap):
- Typing (terminal/input)
- Click/select (moment of user decision)
- Focus switch (visual subject transfers)
- Logo reveal (brand convergence)

**P1 Recommended**:
- Element enter/exit (modal / card)
- Completion/success feedback
- AI generation start/end
- Major transitions (scene changes)

**P2 Optional** (too many will cause clutter):
- hover / focus-in
- Progress tick
- Decorative ambient

### Timestamp Alignment Precision
- **Same-frame alignment** (0ms error): click / focus switch / Logo landing
- **Pre-advance 1–2 frames** (-33ms): fast whoosh (gives audience psychological anticipation)
- **Post-delay 1–2 frames** (+33ms): object landing / impact (matches real-world physics)

---

## BGM Selection Decision Tree

The huashu-design skill ships with 6 BGM tracks (`assets/bgm-*.mp3`):

```
What is the animation's personality?
├─ Product launch / tech demo → bgm-tech.mp3 (minimal synth + piano)
├─ Tutorial / tool walkthrough → bgm-tutorial.mp3 (warm, instructional)
├─ Education / concept explanation → bgm-educational.mp3 (curious, thoughtful)
├─ Marketing / brand promo → bgm-ad.mp3 (upbeat, promotional)
└─ Same style needs variation → bgm-*-alt.mp3 (alternate versions)
```

### Scenarios for No BGM (Worth Considering)
Reference Anthropic Code Desktop (ref-2): **0 SFX + pure Lo-fi BGM** can also be very high-end.

**When to choose no BGM**:
- Animation duration <10s (BGM can't establish itself)
- Product personality is "focused/meditative"
- The scene already has ambient sound / voiceover narration
- SFX density is high (avoid auditory overload)

---

## Scene Recipes (Ready to Use)

### Recipe A · Product Launch Hero (same as huashu-design v9)
```
Duration: 25 seconds
BGM: bgm-tech.mp3 · 45% · freq band <4kHz
SFX density: ~6 per 10s

Cues:
  Terminal typing → type × 4 (0.6s intervals)
  Enter key       → enter
  Cards converge  → card × 4 (staggered 0.2s)
  Selection       → click
  Ripple          → whoosh
  4 focus moments → focus × 4
  Logo            → thud (1.5s)

Volume: BGM 0.45 / SFX 1.0 · amix normalize=0
```

### Recipe B · Tool Feature Demo (Reference: Anthropic Code Desktop)
```
Duration: 30–45 seconds
BGM: bgm-tutorial.mp3 · 50%
SFX density: 0–2 per 10s (very sparse)

Strategy: Let BGM + voiceover drive the piece; SFX only at decisive moments (file save / command execution complete)
```

### Recipe C · AI Generation Demo
```
Duration: 15–20 seconds
BGM: bgm-tech.mp3 or no BGM
SFX density: ~8 per 10s (high density)

Cues:
  User input           → type + enter
  AI starts processing → magic/ai-process (1.2s loop)
  Generation complete  → feedback/complete-done
  Result appears       → magic/sparkle
  
Highlight: ai-process can loop 2–3 times throughout the entire generation process
```

### Recipe D · Pure Ambient Long Shot (Reference: Artifacts)
```
Duration: 10–15 seconds
BGM: none
SFX: 3–5 carefully crafted cues used alone

Strategy: Each SFX is the star, no BGM "muddying" the mix.
Best for: single-product slow motion, close-up showcase
```

---

## ffmpeg Synthesis Templates

### Template 1 · Layer a Single SFX onto Video
```bash
ffmpeg -y -i video.mp4 -itsoffset 2.5 -i sfx.mp3 \
  -filter_complex "[0:a][1:a]amix=inputs=2:normalize=0[a]" \
  -map 0:v -map "[a]" output.mp4
```

### Template 2 · Multi-SFX Timeline Synthesis (aligned by cue time)
```bash
ffmpeg -y \
  -i sfx-type.mp3 -i sfx-enter.mp3 -i sfx-click.mp3 -i sfx-thud.mp3 \
  -filter_complex "\
[0:a]adelay=1100|1100[a0];\
[1:a]adelay=3200|3200[a1];\
[2:a]adelay=7000|7000[a2];\
[3:a]adelay=21800|21800[a3];\
[a0][a1][a2][a3]amix=inputs=4:duration=longest:normalize=0[mixed]" \
  -map "[mixed]" -t 25 sfx-track.mp3
```
**Key parameters**:
- `adelay=N|N`: first is left channel delay (ms), second is right channel — write both to ensure stereo alignment
- `normalize=0`: preserves dynamic range, critical!
- `-t 25`: trim to specified duration

### Template 3 · Video + SFX Track + BGM (with frequency band isolation)
```bash
ffmpeg -y -i video.mp4 -i sfx-track.mp3 -i bgm.mp3 \
  -filter_complex "\
[2:a]atrim=0:25,afade=in:st=0:d=0.3,afade=out:st=23.5:d=1.5,\
     lowpass=f=4000,volume=0.45[bgm];\
[1:a]highpass=f=800,volume=1.0[sfx];\
[bgm][sfx]amix=inputs=2:duration=first:normalize=0[a]" \
  -map 0:v -map "[a]" -c:v copy -c:a aac -b:a 192k final.mp4
```

---

## Failure Mode Quick Reference

| Symptom | Root Cause | Fix |
|---|---|---|
| SFX inaudible | BGM high frequencies masking it | Add `lowpass=f=4000` to BGM + `highpass=f=800` to SFX |
| Sound effects too loud and harsh | SFX absolute volume too high | Lower SFX volume to 0.7, also lower BGM to 0.3, maintain the gap |
| BGM and SFX rhythm conflicts | Wrong BGM choice (used music with strong beat) | Switch to ambient / minimal synth BGM |
| BGM suddenly cuts off at animation end | No fade-out applied | `afade=out:st=N-1.5:d=1.5` |
| SFX overlap into mush | Cues too dense + each SFX too long | Keep SFX duration under 0.5s, cue intervals ≥ 0.2s |
| WeChat mp4 has no sound | WeChat sometimes mutes auto-play | Don't worry about it — users who tap will hear it; GIFs never have sound anyway |

---

## Syncing with Visuals (Advanced)

### SFX Timbre Should Match Visual Style
- Warm beige / paper-textured visuals → SFX with **wood / soft** tones (Morse, paper snap, soft click)
- Cold dark tech visuals → SFX with **metal / digital** tones (beep, pulse, glitch)
- Hand-drawn / playful visuals → SFX with **cartoon / exaggerated** tones (boing, pop, zap)

Our current `apple-gallery-showcase.md` warm beige background → pairs with `keyboard/type.mp3` (mechanical) + `container/card-snap.mp3` (soft) + `impact/logo-reveal-v2.mp3` (cinematic bass)

### SFX Can Guide Visual Rhythm
Advanced technique: **design the SFX timeline first, then adjust visual animation to align with SFX** (not the other way around).
Because every SFX cue is a "clock tick," visual animation adapting to the SFX rhythm stays very solid — conversely, SFX chasing visuals often misses by ±1 frame and creates a jarring feel.

---

## Quality Checklist (Pre-Release Self-Check)

- [ ] Loudness gap: SFX peak - BGM peak = -6 to -8 dB?
- [ ] Frequency bands: BGM lowpass 4kHz + SFX highpass 800Hz?
- [ ] amix normalize=0 (preserves dynamic range)?
- [ ] BGM fade-in 0.3s + fade-out 1.5s?
- [ ] SFX count appropriate (choose density based on scene personality)?
- [ ] Each SFX frame-aligned with visual beat (within ±1 frame)?
- [ ] Logo reveal sound long enough (recommend 1.5s)?
- [ ] Listen with BGM muted: does SFX alone have enough rhythmic sense?
- [ ] Listen with SFX muted: does BGM alone have emotional movement?

Each layer should stand on its own when listened to separately. If it only sounds good with both layers combined, the work isn't done well.

---

## References

- SFX asset catalog: `sfx-library.md`
- Visual style reference: `apple-gallery-showcase.md`
- Anthropic three-film deep audio analysis: `/Users/alchain/Documents/写作/01-公众号写作/项目/2026.04-huashu-design发布/参考动画/AUDIO-BEST-PRACTICES.md`
- huashu-design v9 case study: `/Users/alchain/Documents/写作/01-公众号写作/项目/2026.04-huashu-design发布/配图/hero-animation-v9-final.mp4`
