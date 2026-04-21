# Animation Pitfalls: Bugs We've Hit and Rules That Came Out of Them

The most common bugs when building animations and how to avoid them. Every rule here comes from a real failure.

Read this before writing any animation — it'll save you at least one full round of iteration.

## 1. Stacking Contexts — `position: relative` Is a Default Obligation

**The bug**: A sentence-wrap element contained 3 bracket-layer children (`position: absolute`). Without `position: relative` on sentence-wrap, the absolute brackets used `.canvas` as their coordinate origin and floated 200px off the bottom of the screen.

**Rules**:
- Any container that holds `position: absolute` children **must** explicitly declare `position: relative`
- Even when you don't need visual "offset," write `position: relative` as an anchor for the coordinate system
- If you're writing `.parent { ... }` and its children include `.child { position: absolute }`, reflexively add `relative` to the parent

**Quick check**: Every time you write `position: absolute`, walk up the ancestor chain and confirm the nearest positioned ancestor is the coordinate system *you actually want*.

## 2. The Character Trap — Don't Rely on Rare Unicode

**The bug**: Tried to use `␣` (U+2423 OPEN BOX) to visualize a "space token." Neither Noto Serif SC nor Cormorant Garamond have that glyph — it renders as blank/tofu and the audience sees nothing.

**Rules**:
- **Every character that appears in an animation must exist in your chosen font**
- Common rare-character blacklist: `␣ ␀ ␐ ␋ ␨ ↩ ⏎ ⌘ ⌥ ⌃ ⇧ ␦ ␖ ␛`
- To represent meta-characters like "space / enter / tab," build **CSS-constructed semantic boxes**:
  ```html
  <span class="space-key">Space</span>
  ```
  ```css
  .space-key {
    display: inline-flex;
    padding: 4px 14px;
    border: 1.5px solid var(--accent);
    border-radius: 4px;
    font-family: monospace;
    font-size: 0.3em;
    letter-spacing: 0.2em;
    text-transform: uppercase;
  }
  ```
- Verify emoji too: some emoji fall back to a gray box in fonts other than Noto Emoji — use the `emoji` font-family or SVG

## 3. Data-Driven Grid/Flex Templates

**The bug**: Code had `const N = 6` tokens, but CSS was hardcoded as `grid-template-columns: 80px repeat(5, 1fr)`. The 6th token had no column, throwing off the entire matrix.

**Rules**:
- When count comes from a JS array (`TOKENS.length`), the CSS template should also be data-driven
- Option A: inject via CSS variable from JS
  ```js
  el.style.setProperty('--cols', N);
  ```
  ```css
  .grid { grid-template-columns: 80px repeat(var(--cols), 1fr); }
  ```
- Option B: use `grid-auto-flow: column` and let the browser auto-expand
- **Ban the "hardcoded number + JS constant" combination** — when N changes, CSS won't follow

## 4. Transition Gaps — Scene Switches Must Be Continuous

**The bug**: zoom1 (13–19s) → zoom2 (19.2–23s): the main sentence was already hidden, and zoom1 fade-out (0.6s) + zoom2 fade-in (0.6s) + stagger delay (0.2s+) equaled roughly 1 second of pure blank screen. Viewers thought the animation had frozen.

**Rules**:
- When switching scenes continuously, fade-out and fade-in must **overlap**, not wait for the previous to fully disappear before starting the next
  ```js
  // Bad:
  if (t >= 19) hideZoom('zoom1');      // 19.0s out
  if (t >= 19.4) showZoom('zoom2');    // 19.4s in → 0.4s blank in between

  // Good:
  if (t >= 18.6) hideZoom('zoom1');    // start fade-out 0.4s early
  if (t >= 18.6) showZoom('zoom2');    // fade in simultaneously (cross-fade)
  ```
- Alternatively, use an "anchor element" (e.g. the main sentence) as a visual bridge between scenes — have it briefly reappear during zoom transitions
- Calculate CSS transition `duration` carefully so one transition doesn't fire before the previous one finishes

## 5. The Pure Render Principle — Animation State Should Be Seekable

**The bug**: Used `setTimeout` + `fireOnce(key, fn)` to chain animation state. Fine during normal playback, but when doing frame-by-frame recording or seeking to an arbitrary time, those already-fired setTimeouts can't be "unwound."

**Rules**:
- Ideally, `render(t)` is a **pure function**: given t, it produces a single deterministic DOM state
- If side effects are necessary (e.g. class toggling), use a `fired` set with an explicit reset:
  ```js
  const fired = new Set();
  function fireOnce(key, fn) { if (!fired.has(key)) { fired.add(key); fn(); } }
  function reset() { fired.clear(); /* clear all .show classes */ }
  ```
- Expose `window.__seek(t)` for Playwright / debugging:
  ```js
  window.__seek = (t) => { reset(); render(t); };
  ```
- Animation-related setTimeouts shouldn't span >1 second, otherwise seeking backward will produce chaos

## 6. Measuring Before Fonts Load = Measuring Wrong

**The bug**: Called `charRect(idx)` to measure bracket positions at `DOMContentLoaded`. Fonts weren't loaded yet, so every character width was the fallback font's width, and all positions were wrong. Once the fonts loaded (~500ms later), the `left: Xpx` values were still stale — permanently misaligned.

**Rules**:
- Any layout code that depends on DOM measurement (`getBoundingClientRect`, `offsetWidth`) **must** be wrapped in `document.fonts.ready.then()`
  ```js
  document.fonts.ready.then(() => {
    requestAnimationFrame(() => {
      buildBrackets(...);  // fonts are ready now, measurements are accurate
      tick();              // animation starts
    });
  });
  ```
- The extra `requestAnimationFrame` gives the browser one frame to commit the layout
- If using Google Fonts CDN, add `<link rel="preconnect">` to speed up the first load

## 7. Recording Prep — Build Handles for Video Export

**The bug**: Playwright `recordVideo` defaults to 25fps and starts recording the moment the context is created. The first 2 seconds of page load and font loading get recorded. The delivered video has 2 seconds of blank/white flash at the start.

**Rules**:
- Provide a `render-video.js` utility that handles: warmup navigate → reload to restart animation → wait for duration → ffmpeg trim head + convert to H.264 MP4
- **Frame 0** of the animation must be the complete initial state with layout already in place (not blank, not "loading")
- Want 60fps? Use ffmpeg `minterpolate` as post-processing — don't rely on browser frame rate
- Want GIF? Two-pass palette (`palettegen` + `paletteuse`) — can compress a 30s 1080p animation to 3MB

See `video-export.md` for the complete script invocation.

## 8. Batch Export — tmp Directories Must Include PID to Prevent Concurrency Collisions

**The bug**: Used `render-video.js` to record 3 HTML files in parallel across 3 processes. Because `TMP_DIR` was named with only `Date.now()`, 3 processes starting within the same millisecond all shared the same tmp directory. The first process to finish cleaned up tmp, and the other two hit `ENOENT` when reading from it — all three crashed.

**Rules**:
- Any temporary directory potentially shared by multiple processes must include a **PID or random suffix** in its name:
  ```js
  const TMP_DIR = path.join(DIR, '.video-tmp-' + Date.now() + '-' + process.pid);
  ```
- If you genuinely need to parallelize multiple files, use shell `&` + `wait` rather than forking inside a single node script
- When batch-recording multiple HTML files, the conservative approach is **serial** execution (2 in parallel is fine; 3 or more should queue)

## 9. Progress Bar / Replay Button in Recordings — Chrome Elements Polluting the Video

**The bug**: The animation HTML included a `.progress` bar, `.replay` button, and `.counter` timestamp for human debugging convenience. When exported to MP4 for delivery, these elements appeared at the bottom of the video — as if developer tools had been screen-captured.

**Rules**:
- Keep "chrome elements" intended for humans (progress bar / replay button / footer / masthead / counter / phase labels) and the actual video content managed separately
- **Adopt the convention** `.no-record`: any element with this class is automatically hidden by the recording script
- The script (`render-video.js`) injects CSS by default to hide common chrome class names:
  ```
  .progress .counter .phases .replay .masthead .footer .no-record [data-role="chrome"]
  ```
- Inject this via Playwright's `addInitScript` (takes effect before every navigate, reliable across reloads too)
- To preview the raw HTML with chrome visible, add a `--keep-chrome` flag

## 10. Repeated Animation at the Start of a Recording — Warmup Frame Leakage

**The bug**: The old `render-video.js` flow was `goto → wait fonts 1.5s → reload → wait duration`. Recording starts from context creation, so the animation had already played partway through the warmup phase; after reload it restarts from 0. The result: the video's first few seconds show "animation mid-playback → cut → animation starting from 0" — a jarring repetition.

**Rules**:
- **Warmup and Record must use separate contexts**:
  - Warmup context (no `recordVideo` option): only responsible for loading the URL, waiting for fonts, then closing
  - Record context (with `recordVideo`): starts fresh, animation records from t=0
- ffmpeg `-ss trim` can only cut a small amount of Playwright startup latency (~0.3s). **It cannot** be used to paper over warmup frames — the source must be clean
- Closing the record context = WebM file written to disk (a Playwright constraint)
- Relevant code pattern:
  ```js
  // Phase 1: warmup (throwaway)
  const warmupCtx = await browser.newContext({ viewport });
  const warmupPage = await warmupCtx.newPage();
  await warmupPage.goto(url, { waitUntil: 'networkidle' });
  await warmupPage.waitForTimeout(1200);
  await warmupCtx.close();

  // Phase 2: record (fresh)
  const recordCtx = await browser.newContext({ viewport, recordVideo });
  const page = await recordCtx.newPage();
  await page.goto(url, { waitUntil: 'networkidle' });
  await page.waitForTimeout(DURATION * 1000);
  await page.close();
  await recordCtx.close();
  ```

## 11. Don't Draw "Fake Chrome" In-Frame — Decorative Player UI Collides with Real Chrome

**The bug**: The animation used a `Stage` component that already had a scrubber + timecode + pause button (belonging to `.no-record` chrome, auto-hidden on export). I also drew a "magazine page number" style decorative progress bar at the bottom of the frame reading `00:60 ──── CLAUDE-DESIGN / ANATOMY` — thought it looked polished. **Result**: users saw two progress bars — one from the Stage controller and one I'd drawn. Visually a complete collision. Flagged as a bug. "Why is there another progress bar inside the video?"

**Rules**:

- Stage already provides: scrubber + timecode + pause/replay buttons. **Do not draw** additional progress indicators, timecodes, copyright banners, or chapter counters inside the frame — they either collide with chrome or are filler slop (violating the "earn its place" principle).
- "Page-number feel," "magazine feel," "bottom attribution bar" — these are **decorative impulses** that AI adds automatically at high frequency. Treat every one as a red flag — does it communicate information that truly can't be conveyed otherwise? Or is it just filling empty space?
- If you're convinced some bottom bar must exist (e.g. the animation is literally about player UI), it must be **narratively necessary** and **visually distinct from the Stage scrubber** (different position, different form, different tone).

**Element ownership test** (every element drawn to canvas must be able to answer this):

| What it belongs to | Action |
|---|---|
| The narrative content of a specific scene | OK, keep it |
| Global chrome (for controls/debugging) | Add `.no-record` class, hidden on export |
| **Neither any scene nor chrome** | **Delete it.** It's an ownerless object — it's filler slop by definition |

**Self-check (3 seconds before delivery)**: Take a static screenshot and ask yourself —

- Is there anything in the frame that "looks like video player UI" (horizontal progress bar, timecode, control button shapes)?
- If so, does removing it damage the narrative? If not, delete it.
- Does the same category of information (progress / time / attribution) appear more than once? Consolidate it into one chrome location.

**Counter-examples**: Drawing `00:42 ──── PROJECT NAME` at the bottom, "CH 03 / 06" chapter count in the lower right, version number "v0.3.1" along the edge — all fake chrome filler.

## 12. Leading Blank in Recording + Recording Start Offset — The `__ready` × tick × lastTick Triple Trap

**The bug (A · leading blank)**: A 60-second animation exported to MP4 had 2–3 seconds of blank screen at the start. `ffmpeg --trim=0.3` couldn't cut it.

**The bug (B · start offset, real incident 2026-04-20)**: Exported a 24-second video; user's experience was "the video doesn't start playing until 19 seconds in." In reality the animation started recording at t=5, recorded to t=24, then looped back to t=0, and recorded another 5 seconds to end — so the final 5 seconds of the video were actually the true beginning of the animation.

**Root cause** (shared by both bugs):

Playwright `recordVideo` starts writing WebM the moment `newContext()` is called, while Babel/React/font loading takes L seconds (2–6s). The recording script waits for `window.__ready = true` as the anchor for "animation starts here" — and it must be strictly paired with animation `time = 0`. Two common ways to get this wrong:

| Mistake | Symptom |
|---|---|
| `__ready` is set during `useEffect` or synchronous setup (before the first tick frame) | Recording script thinks animation has started, but WebM is still recording a blank page → **leading blank** |
| tick's `lastTick = performance.now()` initialized at **script top level** | Font loading's L seconds get counted into the first frame's `dt`, `time` instantly jumps to L → recording lags by L seconds throughout → **start offset** |

**✅ Correct full starter tick template** (hand-written animations must use this skeleton):

```js
// ━━━━━━ state ━━━━━━
let time = 0;
let playing = false;   // ❗ don't play by default — start after fonts are ready
let lastTick = null;   // ❗ sentinel — dt is forced to 0 on tick's first frame (don't use performance.now())
const fired = new Set();

// ━━━━━━ tick ━━━━━━
function tick(now) {
  if (lastTick === null) {
    lastTick = now;
    window.__ready = true;   // ✅ paired: "recording start point" and "animation t=0" in the same frame
    render(0);               // render once more to ensure DOM is ready (fonts are ready at this point)
    requestAnimationFrame(tick);
    return;
  }
  const dt = (now - lastTick) / 1000;   // dt only starts advancing after the first frame
  lastTick = now;

  if (playing) {
    let t = time + dt;
    if (t >= DURATION) {
      t = window.__recording ? DURATION - 0.001 : 0;  // don't loop during recording; keep 0.001s to preserve last frame
      if (!window.__recording) fired.clear();
    }
    time = t;
    render(time);
  }
  requestAnimationFrame(tick);
}

// ━━━━━━ boot ━━━━━━
// Don't rAF immediately at the top level — wait for fonts to load first
document.fonts.ready.then(() => {
  render(0);                 // draw the initial frame first (fonts are ready)
  playing = true;
  requestAnimationFrame(tick);  // first tick will pair __ready + t=0
});

// ━━━━━━ seek interface (for render-video defensive correction) ━━━━━━
window.__seek = (t) => { fired.clear(); time = t; lastTick = null; render(t); };
```

**Why this template is correct**:

| Step | Why it must be this way |
|---|---|
| `lastTick = null` + return on first frame | Prevents the L seconds between "script loaded" and "tick first fires" from being counted as animation time |
| `playing = false` by default | Even if `tick` runs during font loading, it doesn't advance time, avoiding render misalignment |
| `__ready` set on tick's first frame | Recording script starts its timer at this moment, which corresponds to the animation's actual t=0 |
| tick only starts inside `document.fonts.ready.then(...)` | Avoids font fallback width measurements, prevents first-frame font pop |
| `window.__seek` exists | Lets `render-video.js` actively correct — a second line of defense |

**Corresponding defenses on the recording script side**:
1. `addInitScript` injects `window.__recording = true` (before `page goto`)
2. `waitForFunction(() => window.__ready === true)`, record the offset at this moment for ffmpeg trim
3. **Additionally**: after `__ready`, actively call `page.evaluate(() => window.__seek && window.__seek(0))` to force any potential time drift in the HTML back to zero — this is the second line of defense, for HTML that doesn't strictly follow the starter template

**Verification**: After exporting MP4:
```bash
ffmpeg -i video.mp4 -ss 0 -vframes 1 frame-0.png
ffmpeg -i video.mp4 -ss $DURATION-0.1 -vframes 1 frame-end.png
```
Frame 0 must be the animation's t=0 initial state (not mid-playback, not black). The last frame must be the animation's final state (not some moment in a second loop).

**Reference implementation**: The `Stage` component in `assets/animations.jsx` and `scripts/render-video.js` both already implement this protocol. Hand-written HTML must use the starter tick template — every line defends against a specific real bug.

## 13. No Looping During Recording — The `window.__recording` Signal

**The bug**: Animation Stage defaults to `loop=true` (convenient for viewing in the browser). `render-video.js` waits an extra 300ms as a buffer after the duration completes before stopping, and that 300ms lets Stage enter the next loop iteration. When ffmpeg `-t DURATION` clips the output, the last 0.5–1s lands inside the next loop — the video suddenly snaps back to the first frame (Scene 1) at the end. Viewers think the video has a bug. "What's happening at the end of the video?"

**Root cause**: No "I'm recording" handshake between the recording script and the HTML. The HTML doesn't know it's being recorded and keeps looping as it would in an interactive browser session.

**Rules**:

1. **Recording script**: inject `window.__recording = true` in `addInitScript` (before `page goto`):
   ```js
   await recordCtx.addInitScript(() => { window.__recording = true; });
   ```

2. **Stage component**: recognize this signal and force `loop=false`:
   ```js
   const effectiveLoop = (typeof window !== 'undefined' && window.__recording) ? false : loop;
   // ...
   if (next >= duration) return effectiveLoop ? 0 : duration - 0.001;
   //                                                       ↑ keep 0.001 to prevent Sprite with end=duration from being turned off
   ```

3. **Ending Sprite's fadeOut**: in recording mode, set `fadeOut={0}` — otherwise the video ends with a fade to transparent/dark. Users expect to stop on a crisp final frame, not a fade-out. For hand-written HTML, all ending Sprites are recommended to use `fadeOut={0}`.

**Reference implementation**: The `Stage` in `assets/animations.jsx` and `scripts/render-video.js` already have the handshake built in. Any hand-written Stage must implement `__recording` detection — otherwise recording will always hit this pitfall.

**Verification**: After exporting MP4, run `ffmpeg -ss 19.8 -i video.mp4 -frames:v 1 end.png` and check that the last 0.2 seconds is still the expected final frame, not a sudden cut to another scene.

## 14. 60fps Video Defaults to Frame Duplication — minterpolate Has Compatibility Issues

**The bug**: `convert-formats.sh` using `minterpolate=fps=60:mi_mode=mci...` to generate 60fps MP4 produced files that couldn't be opened on some versions of macOS QuickTime / Safari (black screen or outright rejection). VLC / Chrome could open them.

**Root cause**: minterpolate outputs H.264 elementary streams containing SEI / SPS fields that some players have trouble parsing.

**Rules**:

- Default 60fps uses a simple `fps=60` filter (frame duplication) — broad compatibility (works in QuickTime/Safari/Chrome/VLC)
- High-quality frame interpolation uses an explicit `--minterpolate` flag — but **must be tested locally** against the target player before delivery
- The value of the 60fps label is **platform algorithm recognition** (Bilibili / YouTube prioritize streaming 60fps-labeled content); actual perceived smoothness improvement for CSS animations is minimal
- Add `-profile:v high -level 4.0` to improve H.264 universal compatibility

**`convert-formats.sh` has already been changed to compatibility mode by default.** If you need interpolated high-quality output, add the `--minterpolate` flag:
```bash
bash convert-formats.sh input.mp4 --minterpolate
```

## 15. `file://` + External `.jsx` CORS Trap — Single-File Delivery Must Inline the Engine

**The bug**: Animation HTML loaded the engine with `<script type="text/babel" src="animations.jsx"></script>`. Opening locally by double-clicking (`file://` protocol) → Babel Standalone tries to fetch `.jsx` via XHR → Chrome throws `Cross origin requests are only supported for protocol schemes: http, https, chrome, chrome-extension...` → completely black screen. No `pageerror` is thrown, only a console error, so it's easy to misdiagnose as "the animation didn't trigger."

Starting an HTTP server may not save you either — on machines with a global proxy, `localhost` goes through the proxy too and returns 502 / connection failure.

**Rules**:

- **Single-file delivery (HTML that works by double-clicking)** → `animations.jsx` must be **inlined** inside a `<script type="text/babel">...</script>` tag. Do not use `src="animations.jsx"`
- **Multi-file project (served via HTTP for demos)** → external loading is fine, but make the `python3 -m http.server 8000` command explicit in the delivery
- The deciding question: is the deliverable "an HTML file" or "a project directory with a server"? If the former, inline
- Stage component / animations.jsx is often 200+ lines — pasting it into an HTML `<script>` block is completely acceptable. Don't be afraid of file size

**Minimum verification**: Double-click the HTML you generated — do **not** open it through any server. If Stage correctly shows the animation's first frame, it passes.

## 16. Cross-Scene Color Inversion Context — Don't Hardcode Colors for In-Frame Elements

**The bug**: In a multi-scene animation, elements that **appear across scenes** (`ChapterLabel` / `SceneNumber` / `Watermark`) had `color: '#1A1A1A'` (dark text) hardcoded in the component. The first 4 light-background scenes were fine; in the 5th black-background scene, "05" and the watermark completely disappeared — no errors, no checks triggered, critical information invisible.

**Rules**:

- **In-frame elements reused across multiple scenes** (chapter labels / scene numbers / timecodes / watermarks / copyright bars) **must not have hardcoded color values**
- Use one of three approaches instead:
  1. **`currentColor` inheritance**: element only specifies `color: currentColor`, parent scene container sets `color: <computed value>`
  2. **invert prop**: component accepts `<ChapterLabel invert />` to manually toggle light/dark
  3. **Auto-compute from background**: `color: contrast-color(var(--scene-bg))` (CSS Level 4 API, or compute in JS)
- Before delivery, use Playwright to extract **a representative frame from each scene** and visually verify "cross-scene elements" are visible in all of them

The insidious nature of this pitfall is — **there's no bug alert**. Only a human eye or OCR can catch it.

## Quick Self-Check Checklist (5 Seconds Before Starting)

- [ ] Does every `position: absolute` element have a parent with `position: relative`?
- [ ] Do all special characters in the animation (`␣` `⌘` `emoji`) exist in the font?
- [ ] Does the Grid/Flex template's count match the length of the JS data?
- [ ] Do scene transitions cross-fade, with no pure blank gaps >0.3s?
- [ ] Is all DOM measurement code wrapped in `document.fonts.ready.then()`?
- [ ] Is `render(t)` pure, or is there a clear reset mechanism?
- [ ] Is frame 0 a complete initial state, not blank?
- [ ] Is there no "fake chrome" decoration in the frame (progress bars/timecodes/bottom attribution bars colliding with Stage scrubber)?
- [ ] Does the animation tick's first frame synchronously set `window.__ready = true`? (handled by animations.jsx; hand-written HTML must add this manually)
- [ ] Does Stage detect `window.__recording` and force `loop=false`? (required for hand-written HTML)
- [ ] Is the ending Sprite's `fadeOut` set to 0 (video ends on a crisp last frame)?
- [ ] Does 60fps MP4 default to frame duplication mode (compatibility), with `--minterpolate` only for high-quality interpolation?
- [ ] After export, have you verified frame 0 and the last frame are the animation's initial/final states?
- [ ] Involving specific brands (Stripe/Anthropic/Lovart/...): did you complete the "brand assets protocol" (SKILL.md §1.a five steps)? Did you write `brand-spec.md`?
- [ ] For single-file delivery HTML: is `animations.jsx` inlined, not `src="..."`? (external .jsx under file:// causes CORS black screen)
- [ ] Do cross-scene elements (chapter labels/watermarks/scene numbers) have no hardcoded colors? Are they visible against every scene's background?
