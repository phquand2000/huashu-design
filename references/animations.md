# Animations: Timeline Animation Engine

Read this when doing animation / motion design HTML. Covers the principles, usage, and typical patterns.

## Core Pattern: Stage + Sprite

Our animation system (`assets/animations.jsx`) provides a timeline-driven engine:

- **`<Stage>`**: Container for the entire animation. Automatically provides auto-scale (fit viewport) + scrubber + play/pause/loop controls
- **`<Sprite start end>`**: A time segment. A Sprite is only visible between `start` and `end`. Internally it can read its own local progress `t` (0→1) via the `useSprite()` hook
- **`useTime()`**: Read the current global time (in seconds)
- **`Easing.easeInOut` / `Easing.easeOut` / ...**:  Easing functions
- **`interpolate(t, from, to, easing?)`**: Interpolate based on t

This pattern is inspired by Remotion/After Effects but is lightweight and zero-dependency.

## Getting Started

```html
<script type="text/babel" src="animations.jsx"></script>
<script type="text/babel">
  const { Stage, Sprite, useTime, useSprite, Easing, interpolate } = window.Animations;

  function Title() {
    const { t } = useSprite();  // local progress 0→1
    const opacity = interpolate(t, [0, 1], [0, 1], Easing.easeOut);
    const y = interpolate(t, [0, 1], [40, 0], Easing.easeOut);
    return (
      <h1 style={{ 
        opacity, 
        transform: `translateY(${y}px)`,
        fontSize: 120,
        fontWeight: 900,
      }}>
        Hello.
      </h1>
    );
  }

  function Scene() {
    return (
      <Stage duration={10}>  {/* 10-second animation */}
        <Sprite start={0} end={3}>
          <Title />
        </Sprite>
        <Sprite start={2} end={5}>
          <SubTitle />
        </Sprite>
        {/* ... */}
      </Stage>
    );
  }

  const root = ReactDOM.createRoot(document.getElementById('root'));
  root.render(<Scene />);
</script>
```

## Common Animation Patterns

### 1. Fade In / Fade Out

```jsx
function FadeIn({ children }) {
  const { t } = useSprite();
  const opacity = interpolate(t, [0, 0.3], [0, 1], Easing.easeOut);
  return <div style={{ opacity }}>{children}</div>;
}
```

**Note on the range**: `[0, 0.3]` means the fade-in completes in the first 30% of the sprite's duration, then stays at opacity=1 for the rest.

### 2. Slide In

```jsx
function SlideIn({ children, from = 'left' }) {
  const { t } = useSprite();
  const progress = interpolate(t, [0, 0.4], [0, 1], Easing.easeOut);
  const offset = (1 - progress) * 100;
  const directions = {
    left: `translateX(-${offset}px)`,
    right: `translateX(${offset}px)`,
    top: `translateY(-${offset}px)`,
    bottom: `translateY(${offset}px)`,
  };
  return (
    <div style={{
      transform: directions[from],
      opacity: progress,
    }}>
      {children}
    </div>
  );
}
```

### 3. Character-by-Character Typewriter

```jsx
function Typewriter({ text }) {
  const { t } = useSprite();
  const charCount = Math.floor(text.length * Math.min(t * 2, 1));
  return <span>{text.slice(0, charCount)}</span>;
}
```

### 4. Number Count-Up

```jsx
function CountUp({ from = 0, to = 100, duration = 0.6 }) {
  const { t } = useSprite();
  const progress = interpolate(t, [0, duration], [0, 1], Easing.easeOut);
  const value = Math.floor(from + (to - from) * progress);
  return <span>{value.toLocaleString()}</span>;
}
```

### 5. Segmented Explanation (Typical Teaching Animation)

```jsx
function Scene() {
  return (
    <Stage duration={20}>
      {/* Phase 1: Show the problem */}
      <Sprite start={0} end={4}>
        <Problem />
      </Sprite>

      {/* Phase 2: Show the approach */}
      <Sprite start={4} end={10}>
        <Approach />
      </Sprite>

      {/* Phase 3: Show the result */}
      <Sprite start={10} end={16}>
        <Result />
      </Sprite>

      {/* Subtitle visible throughout */}
      <Sprite start={0} end={20}>
        <Caption />
      </Sprite>
    </Stage>
  );
}
```

## Easing Functions

Preset easing curves:

| Easing | Characteristic | Use for |
|--------|------|------|
| `linear` | Constant speed | Scrolling text, continuous animation |
| `easeIn` | Slow→Fast | Exit / disappear |
| `easeOut` | Fast→Slow | Entrance / appear |
| `easeInOut` | Slow→Fast→Slow | Position changes |
| **`expoOut`** ⭐ | **Exponential ease-out** | **Primary easing for Anthropic-grade work** (physical weight feel) |
| **`overshoot`** ⭐ | **Elastic bounce-back** | **Toggle / button pop / emphasis interactions** |
| `spring` | Spring physics | Interaction feedback, geometry settling |
| `anticipation` | Reverse first, then forward | Emphasizing actions |

**Default primary easing is `expoOut`** (not `easeOut`) — see `animation-best-practices.md` §2.
Use `expoOut` for entrances, `easeIn` for exits, `overshoot` for toggles — the foundational rule for Anthropic-grade animation.

## Timing and Duration Guidelines

### Micro-interactions (0.1–0.3s)
- Button hover
- Card expand
- Tooltip appear

### UI Transitions (0.3–0.8s)
- Page transitions
- Modal appear
- List item addition

### Narrative Animation (2–10s per segment)
- One phase of a concept explanation
- Data chart reveal
- Scene transition

### A single narrative animation segment should never exceed 10 seconds
Human attention is limited. Spend 10 seconds on one thing, then move on.

## Thinking Order for Designing Animation

### 1. Content/story first, animation second

**Wrong**: Starting with fancy animation ideas and cramming content in afterward
**Right**: First figure out what information to convey, then use animation to serve that information

Animation is a **signal**, not **decoration**. A fade-in says "this matters, look here" — if everything fades in, the signal breaks down.

### 2. Write the timeline by scene

```
0:00 - 0:03   Problem appears (fade in)
0:03 - 0:06   Problem zooms/expands (zoom+pan)
0:06 - 0:09   Solution appears (slide in from right)
0:09 - 0:12   Solution explained (typewriter)
0:12 - 0:15   Result demo (counter up + chart reveal)
0:15 - 0:18   Summary line (static, read for 3 seconds)
0:18 - 0:20   CTA or fade out
```

Write the timeline first, then write the components.

### 3. Assets first

Images/icons/fonts needed for the animation should be prepared **before** you start. Don't go hunting for assets mid-build — it breaks the rhythm.

## Common Issues

**Animation stutters**
→ Usually layout thrashing. Use `transform` and `opacity` — don't touch `top`/`left`/`width`/`height`/`margin`. Browsers GPU-accelerate `transform`.

**Animation too fast, can't read it**
→ A human needs 100–150ms to read one character, 300–500ms for a word. If you're telling a story with text, leave at least 3 seconds per sentence.

**Animation too slow, audience is bored**
→ Interesting visual changes should be dense. A static frame lasting more than 5 seconds will feel dull.

**Multiple animations interfering with each other**
→ Use CSS `will-change: transform` to tell the browser in advance that this element will move, reducing reflow.

**Exporting to video**
→ Use the built-in toolchain from this skill (one command, three output formats): see `video-export.md`
- `scripts/render-video.js` — HTML → 25fps MP4 (Playwright + ffmpeg)
- `scripts/convert-formats.sh` — 25fps MP4 → 60fps MP4 + optimized GIF
- Want more precise frame rendering? Make render(t) a pure function — see `animation-pitfalls.md` item 5

## Working with Video Tools

This skill produces **HTML animations** (running in the browser). If the final output needs to be used as video material:

- **Short animations / concept demos**: Use the HTML animation approach here → screen record
- **Long-form video / narrative**: This skill focuses on HTML animation; use AI video generation skills or professional video software for long-form
- **Motion graphics**: Dedicated tools like After Effects / Motion Canvas are more appropriate

## On Libraries Like Popmotion

If you genuinely need physics animation (spring, decay, keyframes with precise timing) that our engine can't handle, you can fall back to Popmotion:

```html
<script src="https://unpkg.com/popmotion@11.0.5/dist/popmotion.min.js"></script>
```

But **try our engine first**. It covers 90% of cases.
