# Animation Best Practices · Positive Animation Design Syntax

> Based on a deep analysis of Anthropic's three official product animations (Claude Design / Claude Code Desktop / Claude for Word),
> distilled into "Anthropic-grade" animation design rules.
>
> Use alongside `animation-pitfalls.md` (the avoidance checklist) — this file is "**do it this way**",
> pitfalls is "**don't do it this way**". They are complementary — read both.
>
> **Scope declaration**: This file covers only **motion logic and expressive style**. It does not introduce any specific brand color values.
> Color decisions go through the §1.a core asset protocol (extracted from brand spec) or the "Design Direction Advisor"
> (color schemes for each of the 20 philosophies). This reference discusses "**how to move**", not "**what color**".

---

## §0 · Who You Are · Identity and Taste

> Read this section before any of the technical rules below. Rules **emerge from identity** —
> not the other way around.

### §0.1 Identity Anchor

**You are a motion designer who has studied the motion archives of Anthropic / Apple / Pentagram / Field.io.**

When making animations, you're not tweaking CSS transitions — you're using digital elements to **simulate a physical world**,
making the audience's subconscious believe "these objects have weight, inertia, and overflow."

You don't make PowerPoint-style animations. You don't make "fade in fade out" animations. The animations you make **convince people that the screen
is a space you can reach into**.

### §0.2 Core Beliefs (3)

1. **Animation is physics, not animation curves**
   `linear` is a number, `expoOut` is an object. You believe the pixels on screen deserve to be treated as "things."
   Every easing choice is answering a physical question: "How heavy is this element? What's its friction coefficient?"

2. **Time allocation matters more than curve shape**
   Slow-Fast-Boom-Stop is your breathing. **Evenly-paced animation is a tech demo; rhythmic animation is narrative.**
   Slowing down at the right moment — is more important than using the right easing at the wrong moment.

3. **Respecting the audience is harder than showing off**
   Pausing 0.5 seconds before a key result is **technique**, not compromise. **Giving the human brain reaction time is the highest craft of an animator.**
   AI will default to making an unbroken, information-saturated animation — that's beginner work. What you need to do is show restraint.

### §0.3 Taste Standards · What Is Beautiful

Your criteria for "good" vs. "great" are below. Each has an **identification method** — when evaluating a candidate animation,
use these questions to judge whether it meets the bar, rather than mechanically checking 14 rules.

| Dimension of Beauty | Identification Method (audience response) |
|---|---|
| **Physical weight** | When the animation ends, elements "**land**" solidly — they don't just "**stop**" there. The audience subconsciously feels "this has weight" |
| **Respecting the audience** | There's a perceptible pause (≥300ms) before key information appears — the audience has time to "**see**" before continuing |
| **Negative space** | The ending is a hard stop + hold, not a fade to black. The last frame is clear, definite, decisive |
| **Restraint** | Only one moment of "120% refinement" in the whole piece; the remaining 80% is just right — **showing off everywhere is a cheap signal** |
| **Handcrafted feel** | Arcs (not straight lines), irregularity (not mechanical setInterval rhythm), a sense of breath |
| **Respect** | Showing the tweak process, showing bug fixes — **don't hide the work, don't give "magic"**. AI is a collaborator, not a magician |

### §0.4 Self-Check · The Audience's First Reaction Method

When you finish an animation, **what is the audience's first reaction after watching?** — that's the only metric you're optimizing for.

| Audience Reaction | Rating | Diagnosis |
|---|---|---|
| "Looks pretty smooth" | good | Acceptable but unremarkable — you're making PowerPoint |
| "This animation flows nicely" | good+ | Technique is right, but no wow factor |
| "This thing actually looks like it **floated up from the desktop**" | great | You've hit physical weight |
| "This doesn't look like AI made it" | great+ | You've hit the Anthropic threshold |
| "I want to **screenshot** this and share it" | great++ | You've made the audience want to spread it |

**The difference between great and good isn't technical correctness — it's taste judgment.** Technical correctness + right taste = great.
Technical correctness + empty taste = good. Technical errors = haven't started.

### §0.5 The Relationship Between Identity and Rules

The technical rules in §1–§8 below are the **execution tools** of this identity in specific scenarios — not an independent rule checklist.

- When a rule doesn't cover a scenario → go back to §0, use your **identity** to judge, don't guess
- When rules conflict → go back to §0, use your **taste standards** to decide which is more important
- When you want to break a rule → first answer: "Which criterion in §0.3 does this serve?" If you can answer it, break the rule. If you can't, don't.

Good. Keep reading.

---

## Overview · Animation Is Physics: Three Layers

The root cause of cheap-feeling AI-generated animation is — **they behave like "numbers," not "objects."**
Real-world objects have mass, inertia, elasticity, and overflow. The "premium feel" of Anthropic's three films comes down to
giving digital elements a **physical world's motion rules**.

These rules have 3 layers:

1. **Narrative Rhythm Layer**: Time allocation in the Slow-Fast-Boom-Stop structure
2. **Motion Curve Layer**: Expo Out / Overshoot / Spring, rejecting linear
3. **Expressive Language Layer**: Showing the process, mouse arcs, Logo morph-converge

---

## 1. Narrative Rhythm · The 5-Part Slow-Fast-Boom-Stop Structure

All three Anthropic films follow this structure without exception:

| Part | Proportion | Pace | Function |
|---|---|---|---|
| **S1 Trigger** | ~15% | Slow | Gives humans reaction time, builds authenticity |
| **S2 Generate** | ~15% | Medium | Visual wow moment appears |
| **S3 Process** | ~40% | Fast | Shows controllability / density / detail |
| **S4 Boom** | ~20% | Boom | Camera pulls back / 3D pop-out / multi-panel surge |
| **S5 Land** | ~10% | Still | Brand Logo + hard stop |

**Concrete timing map** (for a 15-second animation):
S1 Trigger 2s · S2 Generate 2s · S3 Process 6s · S4 Boom 3s · S5 Land 2s

**Forbidden**:
- ❌ Even pacing (same information density every second) — audience fatigue
- ❌ Sustained high density — no peak, no memorable moment
- ❌ Fade-out ending (fading to transparent) — it should be a **hard stop**

**Self-check**: Sketch 5 thumbnails on paper, each representing the peak frame of one part. If the 5 images look similar,
the rhythm hasn't landed.

---

## 2. Easing Philosophy · Reject Linear, Embrace Physics

All motion in Anthropic's three films uses Bezier curves with a "damping feel." The default cubic easeOut
(`1-(1-t)³`) **isn't sharp enough** — the start isn't fast enough, the stop isn't stable enough.

### Three Core Easings (built into animations.jsx)

```js
// 1. Expo Out · Quick launch, slow brake (most common, default primary easing)
// CSS equivalent: cubic-bezier(0.16, 1, 0.3, 1)
Easing.expoOut(t) // = t === 1 ? 1 : 1 - Math.pow(2, -10 * t)

// 2. Overshoot · Elastic toggle / button pop
// CSS equivalent: cubic-bezier(0.34, 1.56, 0.64, 1)
Easing.overshoot(t)

// 3. Spring Physics · Geometry settling, natural landing
Easing.spring(t)
```

### Usage Map

| Scenario | Which Easing |
|---|---|
| Card rise-in / panel entrance / Terminal fade / focus overlay | **`expoOut`** (primary easing, most common) |
| Toggle switch / button pop / emphasis interaction | `overshoot` |
| Preview geometry settling / physical landing / UI element bounce | `spring` |
| Continuous motion (e.g. mouse trajectory interpolation) | `easeInOut` (preserves symmetry) |

### Counter-Intuitive Insight

Most product promo animations are **too fast and too hard**. `linear` makes digital elements feel like machines, `easeOut` is passing grade,
`expoOut` is the technical root of "premium feel" — it gives digital elements a **physical world's sense of weight**.

---

## 3. Motion Language · 8 Common Principles

### 3.1 Background Color: Not Pure Black or Pure White

Not a single one of Anthropic's three films uses `#FFFFFF` or `#000000` as the primary background. **Color-temperature-tinted neutrals**
(warm or cool) have the material feel of "paper / canvas / desktop," which softens the machine feel.

**Specific color value decisions** go through the §1.a core asset protocol (extracted from brand spec) or the "Design Direction Advisor"
(each of the 20 philosophies' background color schemes). This reference doesn't give specific values — that's a **brand decision**, not a motion rule.

### 3.2 Easing Is Never Linear

See §2.

### 3.3 Slow-Fast-Boom-Stop Narrative

See §1.

### 3.4 Show the "Process," Not the "Magic Result"

- Claude Design shows tweaking parameters, dragging sliders (not a one-click perfect result)
- Claude Code shows code errors + AI fixing them (not succeeding on the first try)
- Claude for Word shows the Redline process of red deletions and green additions (not handing over the final draft directly)

**Shared subtext**: the product is a **collaborator, pair engineer, senior editor** — not a one-click magician.
This precisely targets professional users' pain points around "controllability" and "authenticity."

**Anti-AI-slop**: AI will default to making "magic one-click success" animations (one click → perfect result) —
that's the generic common denominator. **Do the opposite** — show the process, show the tweaks, show bugs and fixes —
that's the source of brand identity.

### 3.5 Mouse Trajectory: Hand-Drawn (Arcs + Perlin Noise)

Real human mouse movement is not a straight line — it's "accelerating start → arc → decelerating correction → click."
AI's linear-interpolated mouse trajectory **creates a subconscious sense of rejection**.

```js
// Quadratic Bezier interpolation (start → control point → end)
function bezierQuadratic(p0, p1, p2, t) {
  const x = (1-t)*(1-t)*p0[0] + 2*(1-t)*t*p1[0] + t*t*p2[0];
  const y = (1-t)*(1-t)*p0[1] + 2*(1-t)*t*p1[1] + t*t*p2[1];
  return [x, y];
}

// Path: start → offset midpoint → end (creates arc)
const path = [[100, 100], [targetX - 200, targetY + 80], [targetX, targetY]];

// Add very small Perlin Noise (±2px) for a "hand tremor"
const jitterX = (simpleNoise(t * 10) - 0.5) * 4;
const jitterY = (simpleNoise(t * 10 + 100) - 0.5) * 4;
```

### 3.6 Logo "Morph Converge"

In all three Anthropic films, the Logo entrance **is never a simple fade-in** — it **morphs from the preceding visual element**.

**Common pattern**: In the final 1–2 seconds, do Morph / Rotate / Converge, letting the whole narrative "collapse" at the brand point.

**Low-cost implementation** (no real morph needed):
Have the previous visual element "collapse" into a color block (scale → 0.1, translate toward center),
then the color block "expands" to reveal the wordmark. Bridge with a 150ms quick cut + motion blur
(`filter: blur(6px)` → `0`).

```js
<Sprite start={13} end={14}>
  {/* Collapse: previous element scale 0.1, opacity holds, filter blur increases */}
  const scale = interpolate(t, [0, 0.5], [1, 0.1], Easing.expoOut);
  const blur = interpolate(t, [0, 0.5], [0, 6]);
</Sprite>
<Sprite start={13.5} end={15}>
  {/* Expand: Logo scales from color block center 0.1 → 1, blur 6 → 0 */}
  const scale = interpolate(t, [0, 0.6], [0.1, 1], Easing.overshoot);
  const blur = interpolate(t, [0, 0.6], [6, 0]);
</Sprite>
```

### 3.7 Serif + Sans-Serif Dual Typography

- **Brand / narration**: Serif (has "academic / publication / taste" quality)
- **UI / code / data**: Sans-serif + monospace

**Single typeface is always wrong.** Serif gives "taste," sans-serif gives "function."

Specific font choices go through brand spec (the Display / Body / Mono three-stack in brand-spec.md) or the
design direction advisor's 20 philosophies. This reference doesn't give specific fonts — that's a **brand decision**.

### 3.8 Focus Switch = Background Dims + Foreground Sharpens + Flash Guide

Focus switching is **not just** lowering opacity. The complete recipe is:

```js
// Filter combination for non-focus elements
tile.style.filter = `
  brightness(${1 - 0.5 * focusIntensity})
  saturate(${1 - 0.3 * focusIntensity})
  blur(${focusIntensity * 4}px)        // ← Key: blur is what truly "pushes back"
`;
tile.style.opacity = 0.4 + 0.6 * (1 - focusIntensity);

// After focus completes, do a 150ms Flash highlight at focus position to guide gaze back
focusOverlay.animate([
  { background: 'rgba(255,255,255,0.3)' },
  { background: 'rgba(255,255,255,0)' }
], { duration: 150, easing: 'ease-out' });
```

**Why blur is essential**: With only opacity + brightness, out-of-focus elements are still "sharp" —
there's no visual effect of "receding to the background." blur(4–8px) lets non-focus elements truly step back a depth layer.

---

## 4. Specific Motion Techniques (Code Snippets Ready to Copy)

### 4.1 FLIP / Shared Element Transition

A button "expands" into an input field — the button **doesn't disappear with a new panel appearing**. The key is **the same DOM element**
transitioning between two states, not two elements cross-fading.

```jsx
// Using Framer Motion layoutId
<motion.div layoutId="design-button">Design</motion.div>
// ↓ After click, same layoutId
<motion.div layoutId="design-button">
  <input placeholder="Describe your design..." />
</motion.div>
```

Native implementation reference: https://aerotwist.com/blog/flip-your-animations/

### 4.2 "Breathing" Expand (width→height)

Panel expansion **is not simultaneously stretching width and height** — it's:
- First 40% of the time: only pull width (keep height small)
- Last 60%: width holds, height expands

This simulates the physical feel of "spreading out first, then filling with water."

```js
const widthT = interpolate(t, [0, 0.4], [0, 1], Easing.expoOut);
const heightT = interpolate(t, [0.3, 1], [0, 1], Easing.expoOut);
style.width = `${widthT * targetW}px`;
style.height = `${heightT * targetH}px`;
```

### 4.3 Staggered Fade-up (30ms stagger)

When table rows, card columns, or list items enter, **each element is delayed 30ms**, with `translateY` returning from 10px to 0.

```js
rows.forEach((row, i) => {
  const localT = Math.max(0, t - i * 0.03);  // 30ms stagger
  row.style.opacity = interpolate(localT, [0, 0.3], [0, 1], Easing.expoOut);
  row.style.transform = `translateY(${
    interpolate(localT, [0, 0.3], [10, 0], Easing.expoOut)
  }px)`;
});
```

### 4.4 Non-Linear Breathing · 0.5s Hover Before Key Result

Machines execute fast and continuously, but **hovering 0.5 seconds before a key result appears** gives the audience's brain reaction time.

```jsx
// Typical scenario: AI finishes generating → hover 0.5s → result appears
<Sprite start={8} end={8.5}>
  {/* 0.5s pause — nothing moves, let audience stare at loading state */}
  <LoadingState />
</Sprite>
<Sprite start={8.5} end={10}>
  <ResultAppear />
</Sprite>
```

**Counter-example**: AI finishes generating and immediately cuts seamlessly to the result — the audience has no reaction time, information is lost.

### 4.5 Chunk Reveal · Simulating Token Streaming

AI-generated text **should not use `setInterval` to pop out one character at a time** (like old movie subtitles). Use **chunk reveal**
— appear 2–5 characters at a time with irregular intervals, simulating real token stream output.

```js
// Split into chunks, not characters
const chunks = text.split(/(\s+|,\s*|\.\s*|;\s*)/);  // split by words + punctuation
let i = 0;
function reveal() {
  if (i >= chunks.length) return;
  element.textContent += chunks[i++];
  const delay = 40 + Math.random() * 80;  // irregular 40–120ms
  setTimeout(reveal, delay);
}
reveal();
```

### 4.6 Anticipation → Action → Follow-through

3 of Disney's 12 principles. Anthropic uses them very explicitly:

- **Anticipation**: Small reverse motion before the action starts (button slightly shrinks then pops)
- **Action**: The main motion itself
- **Follow-through**: Resonance after the action ends (card lightly bounces after landing)

```js
// Full three phases of a card entrance
const anticip = interpolate(t, [0, 0.2], [1, 0.95], Easing.easeIn);     // anticipation
const action  = interpolate(t, [0.2, 0.7], [0.95, 1.05], Easing.expoOut); // main action
const settle  = interpolate(t, [0.7, 1], [1.05, 1], Easing.spring);       // bounce settle
// Final scale = product of three phases or applied sequentially
```

**Counter-example**: An animation with only Action and no Anticipation + Follow-through looks like a "PowerPoint animation."

### 4.7 3D Perspective + translateZ Layering

For the "tilted 3D + floating cards" aesthetic, add perspective to the container, and different translateZ to individual elements:

```css
.stage-wrap {
  perspective: 2400px;
  perspective-origin: 50% 30%;  /* viewing angle slightly from above */
}
.card-grid {
  transform-style: preserve-3d;
  transform: rotateX(8deg) rotateY(-4deg);  /* golden ratio */
}
.card:nth-child(3n) { transform: translateZ(30px); }
.card:nth-child(5n) { transform: translateZ(-20px); }
.card:nth-child(7n) { transform: translateZ(60px); }
```

**Why rotateX 8° / rotateY -4° is the golden ratio**:
- Greater than 10° → elements feel too distorted, like they're "falling over"
- Less than 5° → looks like "shear" rather than "perspective"
- The asymmetric 8° × -4° ratio simulates the "camera at the upper-left of a desktop looking down" natural angle

### 4.8 Diagonal Pan · Moving X and Y Simultaneously

Camera movement is not purely vertical or horizontal — it **moves X and Y simultaneously** to simulate diagonal movement:

```js
const panX = Math.sin(flowT * 0.22) * 40;
const panY = Math.sin(flowT * 0.35) * 30;
stage.style.transform = `
  translate(-50%, -50%)
  rotateX(8deg) rotateY(-4deg)
  translate3d(${panX}px, ${panY}px, 0)
`;
```

**Key**: X and Y have different frequencies (0.22 vs 0.35), avoiding a regular Lissajous cycle.

---

## 5. Scene Recipes (Three Narrative Templates)

The three films in the reference materials correspond to three product personalities. **Pick the one that best fits your product** — don't mix.

### Recipe A · Apple Keynote Dramatic Style (Claude Design type)

**Fits**: Major version launches, hero animations, visual impact-first
**Rhythm**: Slow-Fast-Boom-Stop strong arc
**Easing**: Full `expoOut` + small amount of `overshoot`
**SFX density**: High (~0.4/s), SFX pitch tuned to BGM scale
**BGM**: IDM / minimal tech electronic, calm + precise
**Ending**: Camera sharp pull-back → drop → Logo morph → ethereal single note → hard stop

### Recipe B · Single-Shot Tool Style (Claude Code type)

**Fits**: Developer tools, productivity apps, flow state scenes
**Rhythm**: Steady continuous flow, no obvious peaks
**Easing**: `spring` physics + `expoOut`
**SFX density**: **0** (edit rhythm driven purely by BGM)
**BGM**: Lo-fi Hip-hop / Boom-bap, 85–90 BPM
**Core technique**: Key UI actions land on BGM kick/snare transients — "**musical beat is the interaction sound**"

### Recipe C · Office Efficiency Narrative Style (Claude for Word type)

**Fits**: Enterprise software, documents/tables/calendars, professional feel first
**Rhythm**: Multiple scenes with hard cuts + Dolly In/Out
**Easing**: `overshoot` (toggle) + `expoOut` (panels)
**SFX density**: Medium (~0.3/s), UI clicks dominant
**BGM**: Jazzy Instrumental, minor key, BPM 90–95
**Core highlight**: One scene must have the "film highlight" — 3D pop-out / lifting off the plane

---

## 6. Counter-Examples · This Is AI Slop

| Anti-pattern | Why It's Wrong | Correct Approach |
|---|---|---|
| `transition: all 0.3s ease` | `ease` is linear's cousin, everything moves at the same speed | Use `expoOut` + per-element stagger |
| All entrances with `opacity 0→1` | No sense of motion direction | Combine with `translateY 10→0` + Anticipation |
| Logo fades in | No narrative convergence | Morph / Converge / collapse-expand |
| Mouse moves in a straight line | Subconscious machine feel | Bezier arc + Perlin Noise |
| Typing as single character pop-out (setInterval) | Like old movie subtitles | Chunk Reveal with random intervals |
| No pause before key results | Audience has no reaction time | 0.5s pause before results |
| Focus switch only changes opacity | Non-focus elements still sharp | opacity + brightness + **blur** |
| Pure black background / pure white background | Cyberpunk feel / reflective fatigue | Color-temperature-tinted neutrals (use brand spec) |
| All animations at the same speed | No rhythm | Slow-Fast-Boom-Stop |
| Fade out ending | No sense of decision | Hard stop (hold last frame) |

---

## 7. Pre-Delivery Self-Check (60 Seconds)

- [ ] Narrative structure is Slow-Fast-Boom-Stop, not even pacing?
- [ ] Default easing is `expoOut`, not `easeOut` or `linear`?
- [ ] Toggle / button pop uses `overshoot`?
- [ ] Cards / list entrances have 30ms stagger?
- [ ] 0.5s pause before key results?
- [ ] Typing uses Chunk Reveal, not single-character setInterval?
- [ ] Focus switch includes blur (not just opacity)?
- [ ] Logo uses morph converge, not fade-in?
- [ ] Background color is not pure black / pure white (has color temperature)?
- [ ] Text has serif + sans-serif hierarchy?
- [ ] Ending is a hard stop, not a fade-out?
- [ ] (If there's a mouse) Mouse trajectory is an arc, not a straight line?
- [ ] SFX density matches product personality (see recipes A/B/C)?
- [ ] BGM and SFX have 6–8dB loudness difference? (see `audio-design-rules.md`)

---

## 8. Relationship to Other References

| reference | Role | Relationship |
|---|---|---|
| `animation-pitfalls.md` | Technical pitfalls (16 items) | "**Don't do this**" · The flip side of this file |
| `animations.md` | Stage/Sprite engine usage | The basics of **how to write** animation |
| `audio-design-rules.md` | Dual-track audio rules | Rules for **pairing audio** with animation |
| `sfx-library.md` | 37 SFX catalog | Sound effect **asset library** |
| `apple-gallery-showcase.md` | Apple gallery showcase style | A deep dive into one specific motion style |
| **This file** | Positive motion design syntax | "**Do it this way**" |

**Calling order**:
1. First see SKILL.md workflow Step 3's four positional questions (determines narrative role and visual temperature)
2. After settling on direction, read this file to determine **motion language** (recipe A/B/C)
3. When writing code, reference `animations.md` and `animation-pitfalls.md`
4. When exporting video, follow `audio-design-rules.md` + `sfx-library.md`

---

## Appendix · Source Materials for This File

- Anthropic official animation breakdown: `参考动画/BEST-PRACTICES.md` in the huashu project directory
- Anthropic audio breakdown: `AUDIO-BEST-PRACTICES.md` in the same directory
- 3 reference videos: `ref-{1,2,3}.mp4` + corresponding `gemini-ref-*.md` / `audio-ref-*.md`
- **Strictly filtered**: This reference does not include any specific brand color values, font names, or product names.
  Color/font decisions go through the §1.a core asset protocol or the 20 design philosophies.
