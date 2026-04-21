# Editable PPTX Export: HTML Hard Constraints + Dimension Decisions + Common Errors

This document covers the path of **using `scripts/html2pptx.js` + `pptxgenjs` to translate HTML element-by-element into truly editable PowerPoint text boxes**. This is distinct from `export_deck_pptx.mjs --mode image` (screenshot-backed, text becomes images, not editable).

> **Core prerequisite**: to take this path, the HTML must be written to the 4 constraints below from line one. **This is not a convert-afterward thing** — retrofitting triggers 2–3 hours of rework (validated on the 2026-04-20 equity board meeting project).

---

## Canvas dimensions: use 960×540pt (LAYOUT_WIDE)

PPTX units are **inches** (physical dimensions), not px. The decision principle: the body's computedStyle dimensions must **match the presentation layout's inch dimensions** (±0.1", enforced by `html2pptx.js`'s `validateDimensions` check).

### 3 candidate sizes compared

| HTML body | Physical size | PPT layout | When to use |
|---|---|---|---|
| **`960pt × 540pt`** | **13.333″ × 7.5″** | **pptxgenjs `LAYOUT_WIDE`** | ✅ **Default recommendation** (modern PowerPoint 16:9 standard) |
| `720pt × 405pt` | 10″ × 5.625″ | Custom | Only when the user specifies an "old PowerPoint Widescreen" template |
| `1920px × 1080px` | 20″ × 11.25″ | Custom | ❌ Non-standard size — fonts will appear abnormally small when projected |

**Don't think of HTML dimensions as resolution.** PPTX is a vector document — body size determines **physical dimensions**, not clarity. A huge body (20″×11.25″) won't make text sharper — it just makes the pt font size small relative to the canvas, which actually looks worse when projected or printed.

### Three equivalent body declarations (pick one)

```css
body { width: 960pt;  height: 540pt; }    /* Clearest — recommended */
body { width: 1280px; height: 720px; }    /* Equivalent, px convention */
body { width: 13.333in; height: 7.5in; }  /* Equivalent, inch intuition */
```

Matching pptxgenjs code:

```js
const pptx = new pptxgen();
pptx.layout = 'LAYOUT_WIDE';  // 13.333 × 7.5 inch, no custom layout needed
```

---

## 4 Hard Constraints (violating these causes immediate errors)

`html2pptx.js` translates the HTML DOM element-by-element into PowerPoint objects. PowerPoint's format constraints projected onto HTML = the 4 rules below.

### Rule 1: No bare text directly in a DIV — must be wrapped in `<p>` or `<h1>`-`<h6>`

```html
<!-- ❌ Wrong: text directly in a div -->
<div class="title">Q3 Revenue Up 23%</div>

<!-- ✅ Right: text inside <p> or <h1>-<h6> -->
<div class="title"><h1>Q3 Revenue Up 23%</h1></div>
<div class="body"><p>New users are the main driver</p></div>
```

**Why**: PowerPoint text must live inside a text frame, and text frames correspond to HTML block-level elements (p/h*/li). A bare `<div>` has no corresponding text container in PPTX.

**Also can't use `<span>` as the primary text carrier** — span is an inline element and can't stand alone as a text box. Spans can only **nest inside p/h\*** to apply local styles (bold, color change).

### Rule 2: No CSS gradients — solid colors only

```css
/* ❌ Wrong */
background: linear-gradient(to right, #FF6B6B, #4ECDC4);

/* ✅ Right: solid color */
background: #FF6B6B;

/* ✅ If you need multi-color stripes, use flex children with individual solid colors */
.stripe-bar { display: flex; }
.stripe-bar div { flex: 1; }
.red   { background: #FF6B6B; }
.teal  { background: #4ECDC4; }
```

**Why**: PowerPoint's shape fill supports solid/gradient-fill, but pptxgenjs's `fill: { color: ... }` only maps to solid. Gradients via PowerPoint's native gradient fill require a different structure — currently not supported by the toolchain.

### Rule 3: Background/border/shadow only on DIV, not on text tags

```html
<!-- ❌ Wrong: <p> has a background color -->
<p style="background: #FFD700; border-radius: 4px;">Key content</p>

<!-- ✅ Right: outer div carries background/border, <p> only handles text -->
<div style="background: #FFD700; border-radius: 4px; padding: 8pt 12pt;">
  <p>Key content</p>
</div>
```

**Why**: In PowerPoint, a shape (rectangle/rounded rectangle) and a text frame are two separate objects. HTML's `<p>` only translates to a text frame — background/border/shadow belong to the shape, which must be on the **div wrapping the text**.

### Rule 4: DIV cannot use `background-image` — use `<img>` tags

```html
<!-- ❌ Wrong -->
<div style="background-image: url('chart.png')"></div>

<!-- ✅ Right -->
<img src="chart.png" style="position: absolute; left: 50%; top: 20%; width: 300pt; height: 200pt;" />
```

**Why**: `html2pptx.js` extracts image paths from `<img>` elements only — it does not parse CSS `background-image` URLs.

---

## Path A HTML template skeleton

Each slide is a separate HTML file, scopes isolated from each other (avoids CSS pollution of single-file decks).

```html
<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<style>
  * { margin: 0; padding: 0; box-sizing: border-box; }
  body {
    width: 960pt; height: 540pt;           /* ⚠️ Match LAYOUT_WIDE */
    font-family: system-ui, -apple-system, sans-serif;
    background: #FEFEF9;                    /* Solid color, no gradients */
    overflow: hidden;
  }
  /* DIV handles layout/background/border */
  .card {
    position: absolute;
    background: #1A4A8A;                    /* Background on the DIV */
    border-radius: 4pt;
    padding: 12pt 16pt;
  }
  /* Text tags only handle font styles — no background or border */
  .card h2 { font-size: 24pt; color: #FFFFFF; font-weight: 700; }
  .card p  { font-size: 14pt; color: rgba(255,255,255,0.85); }
</style>
</head>
<body>

  <!-- Title area: outer div for positioning, inner text tags -->
  <div style="position: absolute; top: 40pt; left: 60pt; right: 60pt;">
    <h1 style="font-size: 36pt; color: #1A1A1A; font-weight: 700;">Title as an assertion, not a topic</h1>
    <p style="font-size: 16pt; color: #555555; margin-top: 10pt;">Subtitle adds supporting detail</p>
  </div>

  <!-- Content card: div carries background, h2/p carry text -->
  <div class="card" style="top: 130pt; left: 60pt; width: 240pt; height: 160pt;">
    <h2>Key Point One</h2>
    <p>Brief explanatory text</p>
  </div>

  <!-- List: use ul/li, not manual bullet symbols -->
  <div style="position: absolute; top: 320pt; left: 60pt; width: 540pt;">
    <ul style="font-size: 16pt; color: #1A1A1A; padding-left: 24pt; list-style: disc;">
      <li>First key point</li>
      <li>Second key point</li>
      <li>Third key point</li>
    </ul>
  </div>

  <!-- Illustration: use <img> tag, not background-image -->
  <img src="illustration.png" style="position: absolute; right: 60pt; top: 110pt; width: 320pt; height: 240pt;" />

</body>
</html>
```

---

## Common Error Quick Reference

| Error message | Cause | Fix |
|---------|------|---------|
| `DIV element contains unwrapped text "XXX"` | Bare text directly in a div | Wrap text in `<p>` or `<h1>`-`<h6>` |
| `CSS gradients are not supported` | Used linear/radial-gradient | Switch to solid color, or use flex children for stripes |
| `Text element <p> has background` | `<p>` tag has a background color | Wrap it in a `<div>` for the background, `<p>` only has text |
| `Background images on DIV elements are not supported` | div uses background-image | Switch to `<img>` tag |
| `HTML content overflows body by Xpt vertically` | Content exceeds 540pt | Reduce content or shrink font size, or use `overflow: hidden` to clip |
| `HTML dimensions don't match presentation layout` | body size doesn't match the presentation layout | Use `960pt × 540pt` body with `LAYOUT_WIDE`; or defineLayout for custom size |
| `Text box "XXX" ends too close to bottom edge` | Large font `<p>` is less than 0.5 inch from the body bottom edge | Move it up and leave enough bottom margin; PPT's bottom is partially obscured anyway |

---

## Basic Workflow (3 steps to PPTX)

### Step 1: Write each page as independent HTML per the constraints

```
MyDeck/
├── slides/
│   ├── 01-cover.html    # Each file is a complete 960×540pt HTML
│   ├── 02-agenda.html
│   └── ...
└── illustration/        # All images referenced by <img>
    ├── chart1.png
    └── ...
```

### Step 2: Write build.js calling `html2pptx.js`

```js
const pptxgen = require('pptxgenjs');
const html2pptx = require('../scripts/html2pptx.js');  // this skill's script

(async () => {
  const pres = new pptxgen();
  pres.layout = 'LAYOUT_WIDE';  // 13.333 × 7.5 inch, matches HTML's 960×540pt

  const slides = ['01-cover.html', '02-agenda.html', '03-content.html'];
  for (const file of slides) {
    await html2pptx(`./slides/${file}`, pres);
  }

  await pres.writeFile({ fileName: 'deck.pptx' });
})();
```

### Step 3: Open and inspect

- Open the exported PPTX in PowerPoint/Keynote
- Double-click any text — it should be directly editable (if it's an image, Rule 1 was violated)
- Verify overflow: each page should stay within the body bounds with nothing clipped

---

## This path vs. other options (when to choose what)

| Need | Choose |
|------|------|
| Colleagues will edit the PPTX text / sending to non-technical people for further editing | **This path** (editable, requires writing HTML to the 4 constraints from the start) |
| For presentation only / archival, no further edits | `export_deck_pdf.mjs` (multi-file) or `export_deck_stage_pdf.mjs` (single-file deck-stage), producing vector PDF |
| Visual freedom is priority (animations, web components, CSS gradients, complex SVGs), non-editable is acceptable | `export_deck_pptx.mjs --mode image` (image-backed PPTX) |

**Never run html2pptx on HTML that was written for visual freedom** — real-world testing shows visually-driven HTML has a pass rate < 30%, and retrofitting each page is slower than a full rewrite.

---

## Why the 4 constraints aren't bugs — they're physical constraints

These 4 rules aren't the `html2pptx.js` author being lazy — they are the constraints of the **PowerPoint file format (OOXML) itself** projected onto HTML:

- PPTX text must live in a text frame (`<a:txBody>`), corresponding to block-level HTML elements
- PPTX shapes and text frames are two separate objects — you can't paint a background and write text on the same element simultaneously
- PPTX shape fill has limited gradient support (only certain preset gradients — arbitrary CSS angle gradients are not supported)
- PPTX picture objects must reference real image files, not CSS properties

Once you understand this, **don't expect the tools to get smarter** — the HTML writing style needs to adapt to the PPTX format, not the other way around.
