# Slide Decks: HTML Slide Production Standards

Making slides is a high-frequency design task. This document covers how to do HTML slides well — from architecture choices and per-slide design to the full PDF/PPTX export path.

**What this skill covers**:
- HTML playback/PDF export → this document + `scripts/export_deck_pdf.mjs` / `scripts/export_deck_stage_pdf.mjs`
- Editable PPTX export → `references/editable-pptx.md` + `scripts/html2pptx.js` + `scripts/export_deck_pptx.mjs --mode editable`
- Image-backed PPTX (non-editable but visually faithful) → `scripts/export_deck_pptx.mjs --mode image`

---

## 🛑 Confirm the delivery format before starting (the hardest checkpoint)

**This decision comes before "single file vs. multi-file."** Validated on the 2026-04-20 equity board meeting project: **not confirming the delivery format before starting = 2–3 hours of rework.**

### Decision tree

```
│ Question: what are you ultimately delivering?
├── Browser fullscreen presentation / local HTML only    → maximum visual freedom, build however you want
├── PDF (print / share / archive)                        → maximum visual freedom, any architecture exports fine
└── Editable PPTX (colleagues will edit the text)        → 🛑 write every line of HTML from the start following the 4 hard constraints in `references/editable-pptx.md`
```

### Why "need PPTX means walking Path A from the beginning"

Editable PPTX requires `html2pptx.js` to translate the DOM element-by-element into PowerPoint objects. That requires **4 hard constraints**:

1. Body fixed at 720pt × 405pt (not 1920×1080px)
2. All text wrapped in `<p>`/`<h1>`-`<h6>` (no bare text in divs, no `<span>` as the primary text carrier)
3. `<p>`/`<h*>` tags themselves cannot have background/border/shadow (put those on the outer div)
4. `<div>` cannot use `background-image` (use `<img>` tags)
5. No CSS gradients, no web components, no complex SVG decorations

**This skill's default HTML has high visual freedom** — lots of spans, nested flex, complex SVGs, web components (like `<deck-stage>`), CSS gradients — **almost none of which will naturally pass html2pptx's constraints** (real-world testing shows visually-driven HTML fed directly into html2pptx has a pass rate < 30%).

### Cost comparison of the two real paths (actual lessons from 2026-04-20)

| Path | Approach | Outcome | Cost |
|------|------|------|------|
| ❌ **Write HTML freely first, fix PPTX afterward** | Single-file deck-stage + lots of SVG/span decoration | Only two options remain for editable PPTX:<br>A. Hand-write hundreds of lines of pptxgenjs with hardcoded coordinates<br>B. Rewrite all 17 pages of HTML to Path A format | 2–3 hours of rework, and the hand-written version has **permanent maintenance debt** (change one word in the HTML, manually re-sync the PPTX) |
| ✅ **Write to Path A constraints from step one** | Each page as independent HTML + 4 hard constraints + 720×405pt | One command exports 100% editable PPTX; the same HTML is also browser-presentable (Path A HTML is standard HTML playable in a browser) | 5 extra minutes thinking "how do I wrap this text in `<p>`" when writing HTML — zero rework |

### The question to ask before starting (copy-paste ready)

> Before I start, let me confirm the delivery format:
> - **Browser presentation / PDF** → I'll build with maximum design freedom (animations, web components, complex SVGs, CSS gradients are all fine)
> - **Editable PPTX** (colleagues will edit the text) → I must write the HTML from the start following the 4 hard constraints in `references/editable-pptx.md`. Visual capabilities will be reduced (no gradients, no web components, no complex SVGs), but export is a single command
>
> Which do you want?

### What to do when delivery is mixed

User says "I want HTML presentation **and** editable PPTX" — **this isn't mixed**, it's PPTX requirements covering HTML requirements. HTML written to Path A can be played fullscreen in a browser as-is (just add a `deck_index.html` stitcher). **No extra cost.**

User says "I want PPTX **and** animations / web components" — **this is a real conflict**. Tell the user: editable PPTX means sacrificing those visual capabilities. Let them decide — don't quietly go write a hand-crafted pptxgenjs solution (that becomes permanent maintenance debt).

### What if you only find out PPTX is needed after the fact (emergency fix)

Rare case: HTML is already written and then you discover PPTX is needed. Neither option is perfect:

1. **Image-backed PPTX** (`scripts/export_deck_pptx.mjs` image mode) — 100% visually faithful but text is not editable. Good for "present with PPT, no content changes" scenarios
2. **Hand-write pptxgenjs** (write addText/addShape + embed PNG per page) — text is editable, but position, font, and alignment all need manual tweaking, with high maintenance cost. **Only go this route if the user explicitly accepts "whenever the HTML source changes, you have to re-tune the PPTX by hand"**

Always tell the user the options and let them decide. **Never jump straight to writing pptxgenjs** — that's the absolute last resort.

---

## 🛑 Choose the architecture first: single file or multi-file?

**This choice is the first step in making slides. Getting it wrong means repeated pain. Read this section fully before touching any code.**

### Architecture comparison

| Dimension | Single file + `deck_stage.js` | **Multi-file + `deck_index.html` stitcher** |
|------|--------------------------|--------------------------------------|
| Code structure | One HTML, all slides are `<section>` | Each page is its own HTML, `index.html` stitches with iframes |
| CSS scope | ❌ Global — one page's styles can affect all others | ✅ Naturally isolated — each iframe is its own universe |
| Validation granularity | ❌ Need JS goTo to navigate to a specific page | ✅ Double-click any single-page file to open it in the browser |
| Parallel development | ❌ One file — multiple agents editing will conflict | ✅ Multiple agents can build different pages in parallel, zero-conflict merges |
| Debug difficulty | ❌ One CSS error takes down the whole deck | ✅ One page broken only affects itself |
| In-deck interactivity | ✅ Cross-page shared state is trivial | 🟡 iframes need postMessage to communicate |
| Print to PDF | ✅ Built-in | ✅ Stitcher traverses iframes on beforeprint |
| Keyboard navigation | ✅ Built-in | ✅ Built into the stitcher |

### Which one? (decision tree)

```
│ Question: how many pages do you expect?
├── ≤10 pages, need in-deck animation or cross-page interactivity, pitch deck → single file
└── ≥10 pages, academic lecture, courseware, long deck, multi-agent parallel work → multi-file (recommended)
```

**Default to multi-file**. It's not a "backup option" — it's **the primary path for long decks and team collaboration**. Reason: every advantage of single-file architecture (keyboard navigation, printing, scaling) is also present in multi-file, while multi-file's scope isolation and per-page validateability are things single-file can never compensate for.

### Why this rule is so firm (real incident record)

Single-file architecture hit four consecutive traps during the AI Psychology Lecture deck:

1. **CSS specificity override**: `.emotion-slide { display: grid }` (specificity 10) overrode `deck-stage > section { display: none }` (specificity 2), causing all pages to render stacked on top of each other.
2. **Shadow DOM slot rules crushed by outer CSS**: `::slotted(section) { display: none }` couldn't block outer rule overrides — sections refused to hide.
3. **localStorage + hash navigation race condition**: After refresh, instead of jumping to the hash position, it stopped at the old position stored in localStorage.
4. **High validation cost**: Had to use `page.evaluate(d => d.goTo(n))` to screenshot a specific page, twice as slow as directly `goto(file://.../slides/05-X.html)`, and it errored frequently.

All root causes trace back to a **single global namespace** — multi-file architecture eliminates these problems at the physical level.

---

## Path A (default): Multi-file architecture

### Directory structure

```
MyDeck/
├── index.html              # Copy from assets/deck_index.html, edit MANIFEST
├── shared/
│   ├── tokens.css          # Shared design tokens (colors/type/common chrome)
│   └── fonts.html          # <link> to Google Fonts (included per page)
└── slides/
    ├── 01-cover.html       # Each file is a complete 1920×1080 HTML
    ├── 02-agenda.html
    ├── 03-problem.html
    └── ...
```

### Slide template skeleton

```html
<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<title>P05 · Chapter Title</title>
<link href="https://fonts.googleapis.com/css2?family=..." rel="stylesheet">
<link rel="stylesheet" href="../shared/tokens.css">
<style>
  /* Styles unique to this page. Any class names here won't pollute other pages. */
  body { padding: 120px; }
  .my-thing { ... }
</style>
</head>
<body>
  <!-- 1920×1080 content (body width/height locked by tokens.css) -->
  <div class="page-header">...</div>
  <div>...</div>
  <div class="page-footer">...</div>
</body>
</html>
```

**Key constraints**:
- `<body>` is the canvas — lay out directly on it. Don't wrap in `<section>` or other containers.
- `width: 1920px; height: 1080px` is locked by the `body` rule in `shared/tokens.css`.
- Include `shared/tokens.css` for shared design tokens (colors, type scale, page-header/footer, etc.).
- Write the font `<link>` in each page independently (font imports are cheap, and it ensures each page opens standalone).

### Stitcher: `deck_index.html`

**Copy directly from `assets/deck_index.html`**. You only need to change one thing — the `window.DECK_MANIFEST` array, listing all slide filenames and human-readable labels in order:

```js
window.DECK_MANIFEST = [
  { file: "slides/01-cover.html",    label: "Cover" },
  { file: "slides/02-agenda.html",   label: "Agenda" },
  { file: "slides/03-problem.html",  label: "Problem Statement" },
  // ...
];
```

The stitcher has built-in: keyboard navigation (←/→/Home/End/number keys/P to print), scale + letterbox, bottom-right counter, localStorage memory, hash-based navigation, print mode (traverse iframes and output PDF page by page).

### Per-page validation (the killer advantage of multi-file)

Every slide is a standalone HTML file. **When you finish one, double-click it and open in the browser**:

```bash
open slides/05-personas.html
```

Playwright screenshots go directly to `goto(file://.../slides/05-personas.html)`, no need for JS navigation, and no risk of interference from other pages' CSS. This makes the "change a little, validate a little" workflow nearly zero-cost.

### Parallel development

Split each slide into a task for a different agent, run them all simultaneously — HTML files are independent of each other, no conflicts at merge time. For long decks this parallel approach can cut production time to 1/N.

### What goes in `shared/tokens.css`

Only put things that are **genuinely shared across pages**:

- CSS variables (color palette, type scale, spacing scale)
- Canvas lock like `body { width: 1920px; height: 1080px; }`
- Chrome that's identical on every page, like `.page-header` / `.page-footer`

**Don't** stuff single-page layout classes in here — that degrades back to the global pollution problem of single-file architecture.

---

## Path B (small decks): Single file + `deck_stage.js`

Suitable for ≤10 pages, needing cross-page shared state (e.g., a React tweaks panel controlling all pages), or pitch deck demos that demand extreme compactness.

### Basic usage

1. Read the content from `assets/deck_stage.js`, embed it in a `<script>` in the HTML (or `<script src="deck_stage.js">`)
2. Use `<deck-stage>` in the body to wrap the slides
3. 🛑 **script tag must go after `</deck-stage>`** (see hard constraint below)

```html
<body>

  <deck-stage>
    <section>
      <h1>Slide 1</h1>
    </section>
    <section>
      <h1>Slide 2</h1>
    </section>
  </deck-stage>

  <!-- ✅ Correct: script after deck-stage -->
  <script src="deck_stage.js"></script>

</body>
```

### 🛑 Script placement hard constraint (real trap from 2026-04-20)

**Do not put `<script src="deck_stage.js">` in `<head>`.** Even though it can define `customElements` in `<head>`, the parser triggers `connectedCallback` when it encounters the `<deck-stage>` opening tag — at that point the child `<section>` elements haven't been parsed yet, `_collectSlides()` gets an empty array, the counter shows `1 / 0`, and all pages render stacked.

**Three valid approaches** (pick one):

```html
<!-- ✅ Most recommended: script after </deck-stage> -->
</deck-stage>
<script src="deck_stage.js"></script>

<!-- ✅ Also fine: script in head but with defer -->
<head><script src="deck_stage.js" defer></script></head>

<!-- ✅ Also fine: module scripts are naturally deferred -->
<head><script src="deck_stage.js" type="module"></script></head>
```

`deck_stage.js` already has a built-in `DOMContentLoaded` defense so even if the script is in the head it won't completely blow up — but `defer` or placing it at the bottom of body is still cleaner, and avoids relying on the defense branch.

### ⚠️ CSS traps in single-file architecture (must read)

The most common trap in single-file architecture — **the `display` property getting hijacked by a single-page style**.

Common mistake 1 (writing display: flex directly on section):

```css
/* ❌ Outer CSS specificity 2, overrides shadow DOM ::slotted(section){display:none} (also 2) */
deck-stage > section {
  display: flex;            /* All pages will render stacked! */
  flex-direction: column;
  padding: 80px;
  ...
}
```

Common mistake 2 (section has a higher-specificity class):

```css
.emotion-slide { display: grid; }   /* Specificity: 10, even worse */
```

Both will cause **all slides to render stacked simultaneously** — the counter might show `1 / 10` as if everything's normal, but visually page one is on top of page two on top of page three.

### ✅ Starter CSS (copy directly to avoid traps)

**The section itself** only handles "visible/hidden"; **layout (flex/grid etc.) goes on `.active`**:

```css
/* section only defines non-display shared styles */
deck-stage > section {
  background: var(--paper);
  padding: 80px 120px;
  overflow: hidden;
  position: relative;
  /* ⚠️ Do not write display here! */
}

/* Lock "inactive means hidden" — specificity + weight double insurance */
deck-stage > section:not(.active) {
  display: none !important;
}

/* Only the active page gets its display + layout */
deck-stage > section.active {
  display: flex;
  flex-direction: column;
  justify-content: center;
}

/* Print mode: all pages must show, override :not(.active) */
@media print {
  deck-stage > section { display: flex !important; }
  deck-stage > section:not(.active) { display: flex !important; }
}
```

Alternative: **put each page's flex/grid on an inner wrapper `<div>`**, the section itself is always just a `display: block/none` toggle. This is the cleanest approach:

```html
<deck-stage>
  <section>
    <div class="slide-content flex-layout">...</div>
  </section>
</deck-stage>
```

### Custom dimensions

```html
<deck-stage width="1080" height="1920">
  <!-- 9:16 portrait -->
</deck-stage>
```

---

## Slide Labels

Both deck_stage and deck_index label each page (shown in the counter). Give them **more meaningful** labels:

**Multi-file**: write `{ file, label: "04 Problem Statement" }` in the `MANIFEST`
**Single file**: add `<section data-screen-label="04 Problem Statement">` to the section

**Key: slide numbers start at 1, not 0.**

When a user says "slide 5," they mean the 5th slide — never array index `[4]`. Humans don't think in 0-indexed.

---

## Speaker Notes

**Don't add by default** — only add when the user explicitly requests them.

Once you add speaker notes you can strip down the text on slides to a minimum and focus on impactful visuals — the notes carry the full script.

### Format

**Multi-file**: write in `<head>` of `index.html`:

```html
<script type="application/json" id="speaker-notes">
[
  "Script for slide 1...",
  "Script for slide 2...",
  "..."
]
</script>
```

**Single file**: same location.

### Notes writing principles

- **Complete**: not an outline — the actual words you're going to say
- **Conversational**: the way you'd talk, not formal writing
- **Aligned**: the Nth item in the array corresponds to the Nth slide
- **Length**: 200–400 words is ideal
- **Emotional arc**: mark emphasis, pauses, stress points

---

## Slide Design Patterns

### 1. Establish a system first (always)

After exploring the design context, **verbally declare the system you're going to use**:

```markdown
Deck system:
- Backgrounds: max 2 (90% white + 10% dark section dividers)
- Type: Instrument Serif for display, Geist Sans for body
- Rhythm: section dividers use full-bleed color + white text, regular slides have white backgrounds
- Imagery: hero slides use full-bleed photos, data slides use charts

I'll build to this system — tell me if something's off.
```

Get user confirmation before proceeding.

### 2. Common slide layouts

- **Title slide**: solid background + huge title + subtitle + author/date
- **Section divider**: colored background + section number + section title
- **Content slide**: white background + title + 1–3 bullet points
- **Data slide**: title + large chart/number + brief caption
- **Image slide**: full-bleed photo + small caption at bottom
- **Quote slide**: whitespace + massive quote + attribution
- **Two-column**: left-right comparison (vs / before-after / problem-solution)

Use at most 4–5 layouts per deck.

### 3. Scale (again)

- Body text minimum **24px**, ideally 28–36px
- Headings **60–120px**
- Hero text **180–240px**
- Slides are read from 10 meters away — text needs to be big enough

### 4. Visual rhythm

Decks need **intentional variety**:

- Color rhythm: mostly white background + occasional colored section divider + occasional dark segment
- Density rhythm: a few text-heavy slides + a few image-heavy ones + a few sparse quote slides
- Type rhythm: normal headings + occasional massive hero text

**Don't make every slide look the same** — that's a PPT template, not design.

### 5. Breathing room (essential reading for data-dense pages)

**The most common beginner mistake**: stuffing every piece of information onto one page.

Information density ≠ effective information delivery. Academic/presentation decks especially need restraint:

- Lists/matrices: don't draw N elements all the same size. Use **hierarchy** — make the 5 you're discussing today large and prominent, shrink the other 16 to background hints.
- Big number pages: the number itself is the visual hero. Don't let the surrounding caption exceed 3 lines — the audience's eyes will bounce around.
- Quote pages: leave whitespace between the quote and the attribution. Don't press them together.

Self-check against "is the data the hero?" and "is the text crowded?" — keep editing until the whitespace makes you slightly uncomfortable.

---

## Print to PDF

**Multi-file**: `deck_index.html` already handles the `beforeprint` event and outputs PDF page by page.

**Single file**: `deck_stage.js` handles the same.

Print styles are already written — no need to add extra `@media print` CSS.

---

## Export as PPTX / PDF (self-serve scripts)

HTML-first is the primary citizen. But users often need PPTX/PDF delivery. Two general-purpose scripts are provided — **any multi-file deck can use them** — in `scripts/`:

### `export_deck_pdf.mjs` — export vector PDF (multi-file architecture)

```bash
node scripts/export_deck_pdf.mjs --slides <slides-dir> --out deck.pdf
```

**Features**:
- Text is **kept as vectors** (copyable, searchable)
- Visually 100% faithful (Playwright's embedded Chromium renders then prints)
- **No changes needed to any HTML**
- Each slide gets its own `page.pdf()`, then merged with `pdf-lib`

**Dependencies**: `npm install playwright pdf-lib`

**Limitation**: text in the PDF is not editable — go back to the HTML to make changes.

### `export_deck_stage_pdf.mjs` — for single-file deck-stage architecture only ⚠️

**When to use this**: the deck is a single HTML file with `<deck-stage>` web component wrapping N `<section>` elements (i.e., Path B architecture). In this case the "one `page.pdf()` per HTML" approach of `export_deck_pdf.mjs` won't work — use this dedicated script.

```bash
node scripts/export_deck_stage_pdf.mjs --html deck.html --out deck.pdf
```

**Why you can't reuse export_deck_pdf.mjs** (real lessons from 2026-04-20):

1. **Shadow DOM beats `!important`**: deck-stage's shadow CSS has `::slotted(section) { display: none }` (only the active slide gets `display: block`). Even `@media print { deck-stage > section { display: block !important } }` in the light DOM can't override it — after `page.pdf()` triggers the print media, Chromium's final render only shows the active slide, resulting in **the entire PDF being 1 page** (the current active slide repeated).

2. **Looping goto per page still only yields 1 page**: the intuitive fix of "navigate to each `#slide-N` then `page.pdf({pageRanges:'1'})`" also fails — because the print CSS that overrides `deck-stage > section { display: block }` in the shadow DOM results in the final render always being the first section (not the one you navigated to). Result: 17 loops produce 17 copies of the P01 cover.

3. **Absolute children spill to the next page**: even if all sections render successfully, if a section itself is `position: static`, its absolutely-positioned `cover-footer`/`slide-footer` children are positioned relative to the initial containing block — when the section is forced to 1080px height by print, the absolute footer can get pushed to the next page (shows up as the PDF having one more page than the number of sections, with the extra page being just an orphaned footer).

**Fix strategy** (already implemented in the script):

```js
// After opening the HTML, use page.evaluate to extract sections from deck-stage's slot,
// mount them directly under body in a regular div, and inline style to ensure position:relative + fixed dimensions
await page.evaluate(() => {
  const stage = document.querySelector('deck-stage');
  const sections = Array.from(stage.querySelectorAll(':scope > section'));
  document.head.appendChild(Object.assign(document.createElement('style'), {
    textContent: `
      @page { size: 1920px 1080px; margin: 0; }
      html, body { margin: 0 !important; padding: 0 !important; }
      deck-stage { display: none !important; }
    `,
  }));
  const container = document.createElement('div');
  sections.forEach(s => {
    s.style.cssText = 'width:1920px!important;height:1080px!important;display:block!important;position:relative!important;overflow:hidden!important;page-break-after:always!important;break-after:page!important;background:#F7F4EF;margin:0!important;padding:0!important;';
    container.appendChild(s);
  });
  // Disable page break on last page to avoid trailing blank page
  sections[sections.length - 1].style.pageBreakAfter = 'auto';
  sections[sections.length - 1].style.breakAfter = 'auto';
  document.body.appendChild(container);
});

await page.pdf({ width: '1920px', height: '1080px', printBackground: true, preferCSSPageSize: true });
```

**Why this works**:
- Pulling sections from the shadow DOM slot into a regular light DOM div — completely bypasses the `::slotted(section) { display: none }` rule
- Inlining `position: relative` makes absolutely-positioned children position relative to the section, not overflow
- `page-break-after: always` makes the browser give each section its own page when printing
- No page break on `:last-child` prevents a trailing blank page

**When validating with `mdls -name kMDItemNumberOfPages`**: macOS Spotlight metadata is cached — after rewriting the PDF, run `mdimport file.pdf` to force a refresh, otherwise you'll see the old page count. Use `pdfinfo` or count files from `pdftoppm` for the real number.

---

### `export_deck_pptx.mjs` — export PPTX (two modes)

```bash
# Image-backed (100% visually faithful, text not editable)
node scripts/export_deck_pptx.mjs --slides <dir> --out deck.pptx --mode image

# Each text block as its own text box (editable, but fonts may fall back)
node scripts/export_deck_pptx.mjs --slides <dir> --out deck.pptx --mode editable
```

| Mode | Visual fidelity | Text editable | How it works | Limitations |
|------|---------|----------|---------|------|
| `image` | ✅ 100% | ❌ | Playwright screenshot → pptxgenjs addImage | Text becomes images |
| `editable` | 🟡 ~70% | ✅ | html2pptx extracts each text box | See constraints below |

**Hard constraints for editable mode** (user's HTML must satisfy these, otherwise that page is skipped):
- All text must be inside `<p>`/`<h1>`-`<h6>`/`<ul>`/`<ol>` (no bare text in divs)
- `<p>`/`<h*>` tags themselves cannot have background/border/shadow (put those on outer divs)
- No `::before`/`::after` decorative text (pseudo-elements can't be extracted)
- Inline elements (span/em/strong) cannot have margin
- No CSS gradients (can't be rendered)
- No `background-image` on divs (use `<img>`)

The script has a built-in **auto preprocessor** that wraps bare text in leaf divs into `<p>` (preserving class). This solves the most common violation (bare text). But other violations (border on p, margin on span, etc.) still need to be fixed at the HTML source level.

**Another caveat for editable mode — font fallback**:
- Playwright uses webfonts to measure text-box dimensions; PowerPoint/Keynote renders with local fonts
- When they differ, you get **overflow or misalignment** — review every page by eye
- Recommended: install the fonts used in the HTML on the target machine, or fall back to `system-ui`

### Making HTML export-friendly from the start

For the most reliable deck: **write the HTML to editable mode constraints from day one**. Then `--mode editable` will pass everything. The extra effort is minimal:

```html
<!-- ❌ Bad -->
<div class="title">Key Finding</div>

<!-- ✅ Good (p wrapping, class inherited) -->
<p class="title">Key Finding</p>

<!-- ❌ Bad (border on p) -->
<p class="stat" style="border-left: 3px solid red;">41%</p>

<!-- ✅ Good (border on outer div) -->
<div class="stat-wrap" style="border-left: 3px solid red;">
  <p class="stat">41%</p>
</div>
```

### When to use which

| Scenario | Recommendation |
|------|------|
| For organizers / archival | **PDF** (universal, high fidelity, searchable text) |
| For collaborators who need to tweak text | **PPTX editable** (accept font fallback) |
| For live presentation, no content changes | **PDF** or **PPTX image** |
| HTML is the primary presentation medium | Play in browser directly — export is just backup |

## Deep path to editable PPTX (long-term projects only)

If your deck will be maintained long-term, revised repeatedly, and edited by a team — it's worth **writing the HTML to html2pptx constraints from the start** so `--mode editable` passes reliably. Details in `references/editable-pptx.md` (4 hard constraints + HTML template + common error quick-reference).

---

## Common Issues

**Multi-file: iframe page won't open / blank screen**
→ Check whether the `file` path in `MANIFEST` is correct relative to `index.html`. Use browser DevTools to check if the iframe's src is directly accessible.

**Multi-file: one page's styles conflicting with another**
→ Impossible (iframe isolation). If it feels like a conflict, it's cache — Cmd+Shift+R force refresh.

**Single-file: multiple slides rendering stacked**
→ CSS specificity issue. See the "CSS traps in single-file architecture" section above.

**Single-file: scaling looks wrong**
→ Check that all slides are directly under `<deck-stage>` as `<section>`. No wrapping `<div>` in between.

**Single-file: want to jump to a specific slide**
→ Add a hash to the URL: `index.html#slide-5` jumps to the 5th slide.

**Both architectures: text position inconsistent across screens**
→ Use fixed dimensions (1920×1080) and `px` units — no `vw`/`vh` or `%`. Let scaling handle the rest uniformly.

---

## Validation checklist (run through this when the deck is done)

1. [ ] Open `index.html` (or main HTML) directly in browser — verify no broken images, fonts loaded
2. [ ] Press → to flip through every page — no blank pages, no layout misalignments
3. [ ] Press P to print preview — each page is exactly one A4 (or 1920×1080) with no cropping
4. [ ] Cmd+Shift+R force-refresh 3 random pages — localStorage memory works correctly
5. [ ] Playwright batch screenshots (multi-page: loop `slides/*.html`; single-file: use goTo to switch) — review manually by eye
6. [ ] Search for leftover `TODO` / `placeholder` strings — confirm all cleaned up
