# Apple Gallery Showcase · Gallery Wall Animation Style

> Inspiration: Claude Design official hero video + Apple product page "gallery wall" display
> Battle-tested origin: huashu-design launch hero v5
> Use cases: **product launch hero animations, skill capability demos, portfolio showcases** — any scenario where you need to simultaneously display "multiple high-quality outputs" and guide audience attention

---

## Trigger Judgment: When to Use This Style

**Good fit**:
- You have 10+ real outputs to display simultaneously (PPT, apps, web pages, infographics)
- Audience is a professional group (developers, designers, product managers) sensitive to "quality"
- The tone you want to convey is "restrained, gallery-like, high-end, with a sense of space"
- Need focus and global view to coexist simultaneously (see detail without losing the whole)

**Not a good fit**:
- Single product focus (use the product hero template from frontend-design)
- Emotionally-driven / narrative-heavy animations (use timeline narrative templates)
- Small screens / portrait orientation (tilted perspective gets blurry on small canvases)

---

## Core Visual Tokens

```css
:root {
  /* Light gallery palette */
  --bg:         #F5F5F7;   /* Main canvas background — Apple website gray */
  --bg-warm:    #FAF9F5;   /* Warm off-white variant */
  --ink:        #1D1D1F;   /* Primary text color */
  --ink-80:     #3A3A3D;
  --ink-60:     #545458;
  --muted:      #86868B;   /* Secondary text */
  --dim:        #C7C7CC;
  --hairline:   #E5E5EA;   /* Card 1px border */
  --accent:     #D97757;   /* Terracotta orange — Claude brand */
  --accent-deep:#B85D3D;

  --serif-cn: "Noto Serif SC", "Songti SC", Georgia, serif;
  --serif-en: "Source Serif 4", "Tiempos Headline", Georgia, serif;
  --sans:     "Inter", -apple-system, "PingFang SC", system-ui;
  --mono:     "JetBrains Mono", "SF Mono", ui-monospace;
}
```

**Key Principles**:
1. **Never use a pure black background.** A black background makes work look like film — not "usable deliverables"
2. **Terracotta orange is the only hue accent** — everything else is grayscale + white
3. **Three-font stack** (serif EN + serif CN + sans + mono) creates a "publication" feel rather than "internet product"

---

## Core Layout Patterns

### 1. Floating Cards (The Basic Unit of This Style)

```css
.gallery-card {
  background: #FFFFFF;
  border-radius: 14px;
  padding: 6px;                          /* padding is the "mat board" */
  border: 1px solid var(--hairline);
  box-shadow:
    0 20px 60px -20px rgba(29, 29, 31, 0.12),   /* main shadow, soft and long */
    0 6px 18px -6px rgba(29, 29, 31, 0.06);     /* second close-light layer, creates float */
  aspect-ratio: 16 / 9;                  /* uniform slide ratio */
  overflow: hidden;
}
.gallery-card img {
  width: 100%; height: 100%;
  object-fit: cover;
  border-radius: 9px;                    /* slightly smaller than card radius, visual nesting */
}
```

**Counter-example**: Don't tile edge-to-edge (no padding, no border, no shadow) — that's information-graphic density, not a gallery.

### 2. 3D Tilted Gallery Wall

```css
.gallery-viewport {
  position: absolute; inset: 0;
  overflow: hidden;
  perspective: 2400px;                   /* deeper perspective, tilt not exaggerated */
  perspective-origin: 50% 45%;
}
.gallery-canvas {
  width: 4320px;                         /* canvas = 2.25× viewport */
  height: 2520px;                        /* room to pan */
  transform-origin: center center;
  transform: perspective(2400px)
             rotateX(14deg)              /* tilt back */
             rotateY(-10deg)             /* turn left */
             rotateZ(-2deg);             /* slight tilt, remove too-regular feel */
  display: grid;
  grid-template-columns: repeat(8, 1fr);
  gap: 40px;
  padding: 60px;
}
```

**Parameter sweet spot**:
- rotateX: 10–15deg (more than this looks like an event backdrop)
- rotateY: ±8–12deg (left/right symmetry feel)
- rotateZ: ±2–3deg ("this wasn't placed by a machine" human touch)
- perspective: 2000–2800px (less than 2000 gets fisheye, more than 3000 approaches orthographic)

### 3. 2×2 Four-Corner Converge (Selection Scene)

```css
.grid22 {
  display: grid;
  grid-template-columns: repeat(2, 800px);
  gap: 56px 64px;
  align-items: start;
}
```

Each card slides in from its corresponding corner (tl/tr/bl/br) toward the center + fade in. The `cornerEntry` vectors:

```js
const cornerEntry = {
  tl: { dx: -700, dy: -500 },
  tr: { dx:  700, dy: -500 },
  bl: { dx: -700, dy:  500 },
  br: { dx:  700, dy:  500 },
};
```

---

## Five Core Animation Modes

### Mode A · Four-Corner Converge (0.8–1.2s)

4 elements slide in from the viewport corners, scaling 0.85→1.0 simultaneously, with ease-out. Good for openings that "show choices from multiple directions."

```js
const inP = easeOut(clampLerp(t, start, end));
card.style.transform = `translate3d(${(1-inP)*ce.dx}px, ${(1-inP)*ce.dy}px, 0) scale(${0.85 + 0.15*inP})`;
card.style.opacity = inP;
```

### Mode B · Selected Card Zooms + Others Slide Out (0.8s)

The selected card zooms 1.0→1.28, other cards fade out + blur + drift back to their corners:

```js
// Selected card
card.style.transform = `translate3d(${cellDx*outP}px, ${cellDy*outP}px, 0) scale(${1 + 0.28*easeOut(zoomP)})`;
// Non-selected cards
card.style.opacity = 1 - outP;
card.style.filter = `blur(${outP * 1.5}px)`;
```

**Key**: Non-selected cards must be blurred, not just faded. Blur simulates depth of field, visually "pushing" the selected card forward.

### Mode C · Ripple Expansion (1.7s)

From the center outward, delayed by distance — each card fades in + scales from 1.25x down to 0.94x ("camera pull back"):

```js
const col = i % COLS, row = Math.floor(i / COLS);
const dc = col - (COLS-1)/2, dr = row - (ROWS-1)/2;
const dist = Math.sqrt(dc*dc + dr*dr);
const delay = (dist / maxDist) * 0.8;
const localT = Math.max(0, (t - rippleStart - delay) / 0.7);
card.style.opacity = easeOut(Math.min(1, localT));

// Simultaneously scale entire gallery 1.25→0.94
const galleryScale = 1.25 - 0.31 * easeOut(rippleProgress);
```

### Mode D · Sinusoidal Pan (Continuous Drift)

Combine sine wave + linear drift to avoid the "has a start and end" loop feel of marquee:

```js
const panX = Math.sin(panT * 0.12) * 220 - panT * 8;    // drift left
const panY = Math.cos(panT * 0.09) * 120 - panT * 5;    // drift up
const clampedX = Math.max(-900, Math.min(900, panX));   // prevent edge exposure
```

**Parameters**:
- Sine period `0.09–0.15 rad/s` (slow, one full swing roughly every 30–50 seconds)
- Linear drift `5–8 px/s` (slower than an eye blink)
- Amplitude `120–220 px` (large enough to feel, small enough not to cause dizziness)

### Mode E · Focus Overlay (Focus Switch)

**Key design**: The focus overlay is a **flat element** (not tilted), floating above the tilted canvas. The selected slide scales from its tile position (~400×225) to the screen center (960×540), while the background canvas doesn't change tilt but **dims to 45%**:

```js
// Focus overlay (flat, centered)
focusOverlay.style.width = (startW + (endW - startW) * focusIntensity) + 'px';
focusOverlay.style.height = (startH + (endH - startH) * focusIntensity) + 'px';
focusOverlay.style.opacity = focusIntensity;

// Background cards dim but remain visible (critical! don't use 100% overlay)
card.style.opacity = entryOp * (1 - 0.55 * focusIntensity);   // 1 → 0.45
card.style.filter = `brightness(${1 - 0.3 * focusIntensity})`;
```

**Clarity iron rule**:
- The `<img>` in the focus overlay must have `src` pointing directly to the original image — **don't reuse the compressed thumbnail from the gallery**
- Preload all original images into a `new Image()[]` array in advance
- The overlay's own `width/height` is calculated frame-by-frame; the browser resamples the original each frame

---

## Timeline Architecture (Reusable Skeleton)

```js
const T = {
  DURATION: 25.0,
  s1_in: [0.0, 0.8],    s1_type: [1.0, 3.2],  s1_out: [3.5, 4.0],
  s2_in: [3.9, 5.1],    s2_hold: [5.1, 7.0],  s2_out: [7.0, 7.8],
  s3_hold: [7.8, 8.3],  s3_ripple: [8.3, 10.0],
  panStart: 8.6,
  focuses: [
    { start: 11.0, end: 12.7, idx: 2  },
    { start: 13.3, end: 15.0, idx: 3  },
    { start: 15.6, end: 17.3, idx: 10 },
    { start: 17.9, end: 19.6, idx: 16 },
  ],
  s4_walloff: [21.1, 21.8], s4_in: [21.8, 22.7], s4_hold: [23.7, 25.0],
};

// Core easings
const easeOut = t => 1 - Math.pow(1 - t, 3);
const easeInOut = t => t < 0.5 ? 4*t*t*t : 1 - Math.pow(-2*t+2, 3)/2;
function lerp(time, start, end, fromV, toV, easing) {
  if (time <= start) return fromV;
  if (time >= end) return toV;
  let p = (time - start) / (end - start);
  if (easing) p = easing(p);
  return fromV + (toV - fromV) * p;
}

// Single render(t) function reads timestamp, writes all elements
function render(t) { /* ... */ }
requestAnimationFrame(function tick(now) {
  const t = ((now - startMs) / 1000) % T.DURATION;
  render(t);
  requestAnimationFrame(tick);
});
```

**Architecture essence**: **All state is derived from timestamp t** — no state machines, no setTimeout. This means:
- Jump to any moment with `window.__setTime(12.3)` instantly (useful for Playwright frame-by-frame screenshots)
- Loop is naturally seamless (t mod DURATION)
- Can freeze any frame for debugging

---

## Quality Details (Easy to Miss, But Fatal If Ignored)

### 1. SVG Noise Texture

A light background is most at risk of looking "too flat." Layer an extremely faint fractalNoise:

```html
<style>
.stage::before {
  content: '';
  position: absolute; inset: 0;
  background-image: url("data:image/svg+xml;utf8,<svg xmlns='http://www.w3.org/2000/svg' width='200' height='200'><filter id='n'><feTurbulence type='fractalNoise' baseFrequency='0.85' numOctaves='2' stitchTiles='stitch'/><feColorMatrix values='0 0 0 0 0.078  0 0 0 0 0.078  0 0 0 0 0.074  0 0 0 0.035 0'/></filter><rect width='100%' height='100%' filter='url(%23n)'/></svg>");
  opacity: 0.5;
  pointer-events: none;
  z-index: 30;
}
</style>
```

Looks identical without it — until you remove it.

### 2. Corner Brand Mark

```html
<div class="corner-brand">
  <div class="mark"></div>
  <div>HUASHU · DESIGN</div>
</div>
```

```css
.corner-brand {
  position: absolute; top: 48px; left: 72px;
  font-family: var(--mono);
  font-size: 12px;
  letter-spacing: 0.22em;
  text-transform: uppercase;
  color: var(--muted);
}
```

Only visible during the gallery wall scene, fades in and out. Like a museum label.

### 3. Brand Convergence Wordmark

```css
.brand-wordmark {
  font-family: var(--sans);
  font-size: 148px;
  font-weight: 700;
  letter-spacing: -0.045em;   /* negative tracking is key — makes letters tight like a logo */
}
.brand-wordmark .accent {
  color: var(--accent);
  font-weight: 500;           /* accent character is lighter — creates visual contrast */
}
```

`letter-spacing: -0.045em` is standard practice for large type on Apple product pages.

---

## Common Failure Modes

| Symptom | Cause | Solution |
|---|---|---|
| Looks like a PPT template | Cards have no shadow / hairline | Add two-layer box-shadow + 1px border |
| Tilt feels cheap | Only used rotateY without rotateZ | Add ±2–3deg rotateZ to break the regularity |
| Pan feels "choppy" | Used setTimeout or CSS keyframe loop | Use rAF + sin/cos continuous functions |
| Focus text unreadable | Reused low-res gallery tile image | Independent overlay + original image src directly |
| Background too empty | Plain `#F5F5F7` color | Layer SVG fractalNoise at 0.5 opacity |
| Font feels "too internet" | Only Inter | Add Serif (one each for CN and EN) + mono three-stack |

---

## References

- Complete implementation sample: `/Users/alchain/Documents/写作/01-公众号写作/项目/2026.04-huashu-design发布/配图/hero-animation-v5.html`
- Original inspiration: claude.ai/design hero video
- Reference aesthetics: Apple product pages, Dribbble shot collection pages

When you encounter an animation need of "displaying multiple high-quality outputs," copy the skeleton from this file directly, swap in content + adjust timing.
