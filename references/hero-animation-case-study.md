# Gallery Ripple + Multi-Focus · Scene Orchestration Philosophy

> A **reusable visual orchestration structure** distilled from the huashu-design hero animation v9 (25 seconds, 8 scenes).
> Not an animation production pipeline — this is about **when this orchestration is the "right" choice** for a given scenario.
> Live reference: [demos/hero-animation-v9.mp4](../demos/hero-animation-v9.mp4) · [https://www.huasheng.ai/huashu-design-hero/](https://www.huasheng.ai/huashu-design-hero/)

## One-Line Summary

> **When you have 20+ visually homogeneous assets and the scene needs to "express scale and depth," reach for Gallery Ripple + Multi-Focus first — not layout stacking.**

Generic SaaS feature animations, product launches, skill promotions, series portfolio showcases — as long as you have enough assets and a consistent style, this structure almost always delivers.

---

## What This Technique Is Actually Expressing

It's not "showing off assets" — it's telling a narrative through **two rhythm changes**:

**Beat 1 · Ripple Expansion (~1.5s)**: 48 cards radiate outward from the center; the audience is hit by the **volume** — "oh, this thing has this many outputs."

**Beat 2 · Multi-Focus (~8s, 4 cycles)**: While the camera slowly pans, 4 times it dims + desaturates the background, and magnifies one card to the screen center — the audience shifts from "shock at quantity" to "contemplating quality," each cycle stable at 1.7s.

**Core narrative structure**: **Scale (Ripple) → Gaze (Focus × 4) → Fade Out (Walloff)**. These three beats together express "Breadth × Depth" — not just able to produce a lot, but each one worth stopping to look at.

Compare with counter-examples:

| Approach | Audience Perception |
|------|---------|
| 48 cards arranged statically (no Ripple) | Beautiful but no narrative, like a grid screenshot |
| One card at a time quick-cut (no Gallery context) | Feels like a slideshow, loses "sense of scale" |
| Only Ripple, no Focus | Shocked by quantity but doesn't remember any specific one |
| **Ripple + Focus × 4 (this recipe)** | **First overwhelmed by quantity, then contemplates quality, then calm fade-out — a complete emotional arc** |

---

## Prerequisites (All Must Be Met)

This orchestration **is not universal** — all 4 of these are required:

1. **Asset count ≥ 20, ideally 30+**
   Fewer than 20 and the Ripple will look "sparse" — all 48 cells need to be in motion for the density to feel right. v9 used 48 cells × 32 images (cycling to fill).

2. **Assets have a consistent visual style**
   All 16:9 slide previews / all app screenshots / all cover designs — the aspect ratio, color tone, and layout need to look like "a set." Mixed styles make the gallery look like a clipboard.

3. **Assets are still readable when zoomed in**
   Focus zooms a card to 960px wide — if the original is blurry when enlarged or the information is sparse, the Focus beat breaks. Reverse validation: can you pick 4 cards from the 48 as "most representative"? If you can't, the asset quality isn't consistent.

4. **The scene is landscape or square, not portrait**
   The gallery's 3D tilt (`rotateX(14deg) rotateY(-10deg)`) needs horizontal extension — portrait orientation makes the tilt look narrow and awkward.

**Fallback paths when prerequisites aren't met**:

| Missing What | Fall Back To |
|-------|-----------|
| Assets < 20 | Change to "3–5 cards side-by-side static display + one-by-one focus" |
| Inconsistent style | Change to "cover + 3 chapter full-bleed images" keynote style |
| Information too sparse | Change to "data-driven dashboard" or "quote + large text" |
| Portrait orientation | Change to "vertical scroll + sticky cards" |

---

## Technical Recipe (v9 Production Parameters)

### 4-Layer Structure

```
viewport (1920×1080, perspective: 2400px)
  └─ canvas (4320×2520, oversized overflow) → 3D tilt + pan
      └─ 8×6 grid = 48 cards (gap 40px, padding 60px)
          └─ img (16:9, border-radius 9px)
      └─ focus-overlay (absolute center, z-index 40)
          └─ img (matches selected slide)
```

**Key**: The canvas is 2.25× the viewport — this is what gives the pan its "peeking into a larger world" feeling.

### Ripple Expansion (Distance-Delay Algorithm)

```js
// Each card's entry time = distance from center × 0.8s delay
const col = i % 8, row = Math.floor(i / 8);
const dc = col - 3.5, dr = row - 2.5;       // offset to center
const dist = Math.hypot(dc, dr);
const maxDist = Math.hypot(3.5, 2.5);
const delay = (dist / maxDist) * 0.8;       // 0 → 0.8s
const localT = Math.max(0, (t - rippleStart - delay) / 0.7);
const opacity = expoOut(Math.min(1, localT));
```

**Core parameters**:
- Total duration 1.7s (`T.s3_ripple: [8.3, 10.0]`)
- Maximum delay 0.8s (center appears first, corners last)
- Each card's entry duration 0.7s
- Easing: `expoOut` (explosive feel, not smooth)

**Simultaneously**: canvas scales from 1.25 → 0.94 (zoom out to reveal) — synchronized push-back feel with the appearance.

### Multi-Focus (4-Cycle Rhythm)

```js
T.focuses = [
  { start: 11.0, end: 12.7, idx: 2  },  // 1.7s
  { start: 13.3, end: 15.0, idx: 3  },  // 1.7s
  { start: 15.6, end: 17.3, idx: 10 },  // 1.7s
  { start: 17.9, end: 19.6, idx: 16 },  // 1.7s
];
```

**Rhythm rule**: Each focus is 1.7s, with 0.6s breathing gaps. Total 8s (11.0–19.6s).

**Inside each focus cycle**:
- In ramp: 0.4s (`expoOut`)
- Hold: middle 0.9s (`focusIntensity = 1`)
- Out ramp: 0.4s (`easeOut`)

**Background changes (this is the key)**:

```js
if (focusIntensity > 0) {
  const dimOp = entryOp * (1 - 0.6 * focusIntensity);  // dim to 40%
  const brt = 1 - 0.32 * focusIntensity;                // brightness 68%
  const sat = 1 - 0.35 * focusIntensity;                // saturate 65%
  card.style.filter = `brightness(${brt}) saturate(${sat})`;
}
```

**Not just opacity — simultaneously desaturate + darken.** This makes the foreground overlay's colors "pop out," rather than just "becoming a bit brighter."

**Focus overlay size animation**:
- From 400×225 (entrance) → 960×540 (hold state)
- Surrounded by 3 layers of shadow + 3px accent-color outline ring, creating a "framed" feeling

### Pan (Continuous Motion Keeps Static From Getting Boring)

```js
const panT = Math.max(0, t - 8.6);
const panX = Math.sin(panT * 0.12) * 220 - panT * 8;
const panY = Math.cos(panT * 0.09) * 120 - panT * 5;
```

- Sine wave + linear drift dual-layer motion — not a pure loop; every moment is a different position
- X/Y at different frequencies (0.12 vs 0.09) prevents the audience from spotting a "regular cycle"
- Clamped to ±900/500px to prevent drifting off screen

**Why not pure linear pan**: With pure linear, the audience "predicts" where things will be in one second; sine + drift makes every second new, and under 3D tilt creates a "micro-seasick" feeling (the good kind) that holds attention.

---

## 5 Reusable Patterns (Distilled from v6→v9 Iteration)

### 1. **expoOut as Primary Easing, Not cubicOut**

`easeOut = 1 - (1-t)³` (smooth) vs `expoOut = 1 - 2^(-10t)` (explosive then rapidly converging).

**Choice rationale**: expoOut reaches 90% in the first 30% of time, more like physical damping — fits the intuition of "a heavy thing landing." Especially good for:
- Card entrances (sense of weight)
- Ripple expansion (shockwave)
- Brand floating up (sense of landing)

**When to still use cubicOut**: focus out ramp, symmetric micro-animations.

### 2. **Paper-Texture Background + Terracotta Orange Accent (Anthropic DNA)**

```css
--bg: #F7F4EE;        /* warm paper */
--ink: #1D1D1F;       /* near-black */
--accent: #D97757;    /* terracotta orange */
--hairline: #E4DED2;  /* warm line */
```

**Why**: A warm background retains its "breathing quality" even after GIF compression — unlike pure white, which turns into "screen glare." Terracotta orange as the sole accent runs through terminal prompt, dir-card selection, cursor, brand hyphen, focus ring — every visual anchor is strung together by this one color.

**v5 lesson**: Added a noise overlay to simulate "paper grain" — which destroyed GIF frame compression (every frame was different). v6 changed to "only warm background + warm shadow," retaining 90% of the paper feel with 60% reduction in GIF file size.

### 3. **Two Tiers of Shadow to Simulate Depth, Not Real 3D**

```css
.gallery-card.depth-near { box-shadow: 0 32px 80px -22px rgba(60,40,20,0.22), ... }
.gallery-card.depth-far  { box-shadow: 0 14px 40px -16px rgba(60,40,20,0.10), ... }
```

Use a `sin(i × 1.7) + cos(i × 0.73)` deterministic algorithm to assign each card to near/mid/far shadow tiers — **visually gives a "three-dimensional stacking" feel, but the transform never changes each frame, GPU cost is zero**.

**The cost of real 3D**: Each card with individual `translateZ`, GPU calculating 48 transforms + shadow blur every frame. Tried in v4, Playwright struggled to record at 25fps. The two-tier shadow in v6 has less than 5% visible difference, but at 10× less cost.

### 4. **Font Weight Change (font-variation-settings) Is More Cinematic Than Font Size Change**

```js
const wght = 100 + (700 - 100) * morphP;  // 100 → 700 over 0.9s
wordmark.style.fontVariationSettings = `"wght" ${wght.toFixed(0)}`;
```

Brand wordmark goes from Thin → Bold over 0.9s, combined with subtle letter-spacing adjustment (-0.045 → -0.048em).

**Why it's better than scale up/down**:
- Scale up/down: audiences have seen it too many times, expectations are set
- Weight change is "internal fullness," like a balloon being inflated — not "being pushed closer"
- Variable fonts are a feature that became widespread post-2020; audiences subconsciously feel "modern"

**Limitation**: Must use a font that supports variable font (Inter / Roboto Flex / Recursive, etc.). Regular static fonts can only simulate it (switching between a few fixed weights creates jumps).

### 5. **Corner Brand Low-Intensity Persistent Signature**

During the gallery phase, there's a small `HUASHU · DESIGN` label in the upper left corner, 16% opacity value, 12px font size, wide letter spacing.

**Why add this**:
- After the Ripple explosion, audiences can easily "lose focus" and forget what they're watching — the light upper-left label helps anchor them
- More high-end than a full-screen big logo — people who do branding know, a brand signature doesn't need to shout
- When the GIF is screenshot and shared, it still carries an attribution signal

**Rule**: Only appears during the middle section (when the screen is busy) — off during the opening (so it doesn't cover the terminal) and off at the end (brand reveal is the star).

---

## Counter-Examples: When NOT to Use This Orchestration

**❌ Product demos (need to show features)**: Gallery makes every card flash by — the audience won't remember any specific feature. Change to "single-screen focus + tooltip annotations."

**❌ Data-driven content**: Audience needs to read numbers; Gallery's fast pace doesn't give reading time. Change to "data charts + item-by-item reveal."

**❌ Story narrative**: Gallery is a "parallel" structure; stories need "cause and effect." Change to keynote chapter cuts.

**❌ Only 3–5 assets**: Ripple density isn't enough — looks like a "patch." Change to "static arrangement + one-at-a-time highlight."

**❌ Portrait (9:16)**: 3D tilt needs horizontal extension; portrait makes the tilt feel "skewed" rather than "spread out."

---

## How to Determine If Your Task Suits This Orchestration

Three-step quick check:

**Step 1 · Asset count**: Count how many same-type visual assets you have. < 15 → stop; 15–25 → stretch it; 25+ → go for it.

**Step 2 · Consistency test**: Put 4 random assets side by side. Do they look like "a set"? No → unify the style first, then build, or change approach.

**Step 3 · Narrative match**: Are you expressing "Breadth × Depth" (quantity × quality)? Or "process," "features," "story"? If not the former, don't force-fit.

All three yes — fork the v6 HTML directly, change the `SLIDE_FILES` array and timeline, and you can reuse it. Change the palette at `--bg / --accent / --ink`, swap the skin without changing the skeleton.

---

## Related References

- Full technical process: [references/animations.md](animations.md) · [references/animation-best-practices.md](animation-best-practices.md)
- Animation export pipeline: [references/video-export.md](video-export.md)
- Audio configuration (BGM + SFX dual-track): [references/audio-design-rules.md](audio-design-rules.md)
- Cross-reference for Apple gallery style: [references/apple-gallery-showcase.md](apple-gallery-showcase.md)
- Source HTML (v6 + audio integration): `www.huasheng.ai/huashu-design-hero/index.html`
