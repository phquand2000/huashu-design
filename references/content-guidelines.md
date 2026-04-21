# Content Guidelines: Anti-AI Slop, Content Principles, Scale Standards

The easiest trap to fall into in AI design. This is a list of "what NOT to do" — more important than "what to do" — because AI slop is the default. If you don't actively avoid it, it will happen.

## Complete AI Slop Blacklist

### Visual Traps

**❌ Aggressive gradient backgrounds**
- Purple → pink → blue full-screen gradients (the signature smell of AI-generated pages)
- Rainbow gradients in any direction
- Mesh gradients covering the entire background
- ✅ If you must use gradients: subtle, monochromatic, used intentionally as accents (e.g. button hover)

**❌ Rounded cards + left border accent color**
```css
/* This is the signature fingerprint of an AI-flavored card */
.card {
  border-radius: 12px;
  border-left: 4px solid #3b82f6;
  padding: 16px;
}
```
This card style is everywhere in AI-generated dashboards. Want to create emphasis? Use a more design-intentional approach: background color contrast, weight/size contrast, a plain divider, or skip the card entirely.

**❌ Emoji decoration**
Unless the brand itself uses emoji (like Notion or Slack), don't put emoji in the UI. **Especially avoid**:
- 🚀 ⚡️ ✨ 🎯 💡 before headings
- ✅ in feature lists
- → inside CTA buttons (a standalone arrow is fine; an emoji arrow is not)

No icons? Use a real icon library (Lucide/Heroicons/Phosphor), or use a placeholder.

**❌ SVG-drawn imagery**
Don't try to draw people, scenes, devices, objects, or abstract art in SVG. AI-generated SVG imagery looks AI-made at a glance — childish and cheap. **A grey rectangle with a text label "Illustration placeholder 1200×800" is 100× better than a clumsy SVG hero illustration.**

The only acceptable SVG uses:
- True icons (16×16 to 32×32 scale)
- Geometric shapes as decorative elements
- Charts in data visualization

**❌ Excessive iconography**
Not every heading/feature/section needs an icon. Overusing icons makes the interface look like a toy. Less is more.

**❌ "Data slop"**
Fabricated stats used as decoration:
- "10,000+ happy customers" (you don't even know if that's true)
- "99.9% uptime" (don't write it without real data)
- Decorative "metric cards" made of icon + number + phrase
- Mock table data dressed up to look impressive

If you don't have real data, leave a placeholder or ask the user.

**❌ "Quote slop"**
Fabricated user testimonials or celebrity quotes used to decorate a page. Leave a placeholder and ask the user for real quotes.

### Typography Traps

**❌ Avoid these overused fonts**:
- Inter (the default of AI-generated pages)
- Roboto
- Arial / Helvetica
- Pure system font stack
- Fraunces (once AI discovered it, it got overused)
- Space Grotesk (currently AI's favorite)

**✅ Use a distinctive display + body pairing.** Directions to explore:
- Serif display + sans-serif body (editorial feel)
- Mono display + sans body (technical feel)
- Heavy display + light body (contrast)
- Variable font for weight animation in hero sections

Font resources:
- Underused gems on Google Fonts (Instrument Serif, Cormorant, Bricolage Grotesque, JetBrains Mono)
- Open-source font sites (sibling fonts to Fraunces, Adobe Fonts)
- Don't invent font names that don't exist

### Color Traps

**❌ Inventing colors from scratch**
Don't design a full color palette from nothing if you're not familiar with color theory. The result is usually inharmonious.

**✅ Strategy**:
1. Brand colors exist → use them; fill missing color tokens with oklch interpolation
2. No brand colors but references exist → eyedrop from screenshots of the reference product
3. Starting completely from zero → pick a known color system (Radix Colors / Tailwind default palette / Anthropic brand); don't tune your own

**Defining colors in oklch** is the most modern approach:
```css
:root {
  --primary: oklch(0.65 0.18 25);      /* warm terracotta */
  --primary-light: oklch(0.85 0.08 25); /* lighter tint of the same hue */
  --primary-dark: oklch(0.45 0.20 25);  /* darker shade of the same hue */
}
```
oklch guarantees hue won't shift when you adjust lightness — more reliable than hsl.

**❌ Casually adding dark mode by inverting colors**
It's not a simple color invert. Good dark mode requires retuning saturation, contrast, and accent colors. If you don't want to do dark mode properly, don't do it at all.

### Layout Traps

**❌ Bento grid overuse**
Every AI-generated landing page tries to do bento. Unless your information structure genuinely suits bento, use a different layout.

**❌ Big hero + 3-column features + testimonials + CTA**
This landing page template has been done to death. If you want to innovate, actually innovate.

**❌ Every card in a card grid looks identical**
Asymmetric layouts, cards of different sizes, some with images and some without, some spanning columns — that's what a real designer would do.

## Content Principles

### 1. Don't add filler content

Every element must earn its place. White space is a design problem — solve it with **composition** (contrast, rhythm, breathing room), **not** by stuffing in content.

**Questions to identify filler**:
- If you removed this content, would the design get worse? If the answer is "no," remove it.
- What real problem does this element solve? If the answer is "makes the page feel less empty," delete it.
- Is this stat/quote/feature backed by real data? If not, don't make it up.

"One thousand no's for every yes."

### 2. Ask before adding material

You think adding an extra section/page/block would improve things? Ask the user first — don't add it unilaterally.

Why:
- The user knows their audience better than you do
- Adding content has a cost; the user may not want it
- Unilaterally adding content violates the "junior designer reporting in" relationship

### 3. Create a system up front

After exploring the design context, **verbally state the system you plan to use** and let the user confirm:

```markdown
My design system:
- Colors: #1A1A1A primary + #F0EEE6 background + #D97757 accent (from your brand)
- Typography: Instrument Serif for display + Geist Sans for body
- Rhythm: section titles use full-bleed colored background + white text; regular sections use white background
- Images: hero uses full-bleed photo; feature sections use placeholders until you provide assets
- Maximum 2 background colors to avoid visual clutter

Confirm this direction and I'll start building.
```

Only start after the user confirms. This check-in prevents "getting halfway through and realizing the direction was wrong."

## Scale Standards

### Slides (1920×1080)

- Body text minimum **24px**, ideal 28–36px
- Headings 60–120px
- Section titles 80–160px
- Hero headline can go 180–240px
- Never use text smaller than 24px on slides

### Print Documents

- Body text minimum **10pt** (≈13.3px), ideal 11–12pt
- Headings 18–36pt
- Captions 8–9pt

### Web and Mobile

- Body text minimum **14px** (16px for accessibility/elderly-friendly)
- Mobile body text **16px** (avoids iOS auto-zoom)
- Hit targets (tappable elements) minimum **44×44px**
- Line height 1.5–1.7 (1.7–1.8 for CJK text)

### Contrast

- Body text vs. background **at least 4.5:1** (WCAG AA)
- Large text vs. background **at least 3:1**
- Check with Chrome DevTools accessibility panel

## CSS Power Tools

**Advanced CSS features** are a designer's best friend — use them freely:

### Typography

```css
/* Make headings wrap more naturally — no orphaned single word on the last line */
h1, h2, h3 { text-wrap: balance; }

/* Body text wrapping — avoids widows and orphans */
p { text-wrap: pretty; }

/* CJK typography: punctuation compression, line-start/end control */
p { 
  text-spacing-trim: space-all;
  hanging-punctuation: first;
}
```

### Layout

```css
/* CSS Grid + named areas = maximum readability */
.layout {
  display: grid;
  grid-template-areas:
    "header header"
    "sidebar main"
    "footer footer";
  grid-template-columns: 240px 1fr;
  grid-template-rows: auto 1fr auto;
}

/* Subgrid to align card content */
.card { display: grid; grid-template-rows: subgrid; }
```

### Visual Effects

```css
/* Styled scrollbar */
* { scrollbar-width: thin; scrollbar-color: #666 transparent; }

/* Glassmorphism (use sparingly) */
.glass {
  backdrop-filter: blur(20px) saturate(150%);
  background: color-mix(in oklch, white 70%, transparent);
}

/* View Transitions API for smooth page changes */
@view-transition { navigation: auto; }
```

### Interaction

```css
/* :has() selector makes conditional styling easy */
.card:has(img) { padding-top: 0; } /* no top padding on cards that have images */

/* Container queries for truly responsive components */
@container (min-width: 500px) { ... }

/* New color-mix function */
.button:hover {
  background: color-mix(in oklch, var(--primary) 85%, black);
}
```

## Quick Decision Reference: When You're Not Sure

- Want to add a gradient? → Probably don't
- Want to add an emoji? → Don't
- Want to give a card rounded corners + border-left accent? → Don't; find another way
- Want to draw a hero illustration in SVG? → Don't; use a placeholder
- Want to add a decorative quote? → Ask the user if they have a real quote first
- Want to add a row of icon features? → Ask whether icons are even needed; they probably aren't
- Using Inter? → Switch to something more distinctive
- Using a purple gradient? → Switch to a color palette with a real rationale

**When you think "adding this would make it look better" — that's usually a sign of AI slop.** Start with the simplest version; only add when the user asks for it.
