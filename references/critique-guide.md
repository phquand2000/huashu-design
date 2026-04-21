# Design Critique Deep Guide

> Detailed reference for Phase 7. Provides scoring rubrics, scenario-specific focus areas, and a common issues checklist.

---

## Scoring Rubrics Explained

### 1. Philosophy Alignment

| Score | Criteria |
|-------|----------|
| 9-10 | The design perfectly embodies the core spirit of the chosen philosophy; every detail has a philosophical basis |
| 7-8 | Overall direction is correct, core characteristics are in place, occasional details stray |
| 5-6 | Intent is visible, but execution mixes in elements from other styles — not pure enough |
| 3-4 | Surface-level imitation only; the philosophical core is not understood |
| 1-2 | Essentially unrelated to the chosen philosophy |

**Review checklist**:
- Does it use the signature techniques of that designer/institution?
- Do the colors, typography, and layout conform to that philosophical system?
- Are there any "self-contradicting" elements? (e.g., choosing Kenya Hara but cramming in content)

### 2. Visual Hierarchy

| Score | Criteria |
|-------|----------|
| 9-10 | The user's eye flows naturally along the designer's intended path; zero friction in information retrieval |
| 7-8 | Primary/secondary relationships are clear; 1-2 occasional hierarchy ambiguities |
| 5-6 | Headlines and body text are distinguishable, but the middle tiers are muddled |
| 3-4 | Information is flatly presented with no clear visual entry point |
| 1-2 | Chaotic — users don't know where to look first |

**Review checklist**:
- Is the type-size contrast between headlines and body sufficient? (at least 2.5×)
- Do color/weight/size establish 3-4 clear levels?
- Is white space guiding the eye?
- "Squint test": squint your eyes — is the hierarchy still legible?

### 3. Craft Quality

| Score | Criteria |
|-------|----------|
| 9-10 | Pixel-perfect — no flaws in alignment, spacing, or color |
| 7-8 | Polished overall; 1-2 minor alignment/spacing issues |
| 5-6 | Basically aligned, but spacing is inconsistent and color usage is unsystematic |
| 3-4 | Obvious alignment errors, chaotic spacing, too many colors |
| 1-2 | Rough — looks like a draft |

**Review checklist**:
- Is a consistent spacing system used (e.g., an 8pt grid)?
- Is the spacing between like elements consistent?
- Is the number of colors under control? (usually no more than 3-4)
- Is the type family unified? (usually no more than 2)
- Is edge alignment precise?

### 4. Functionality

| Score | Criteria |
|-------|----------|
| 9-10 | Every design element serves the goal; zero redundancy |
| 7-8 | Function-driven intent is clear; a small amount of decoration could be cut |
| 5-6 | Basically usable, but noticeable decorative elements distract |
| 3-4 | Form over function — users have to work to find information |
| 1-2 | Completely overwhelmed by decoration; has lost the ability to communicate information |

**Review checklist**:
- If you remove any element, does the design get worse? (If not, remove it)
- Is the CTA/key information in the most prominent position?
- Are there elements added "because they look good"?
- Does the information density match the medium? (slides shouldn't be too dense; PDFs can be denser)

### 5. Originality

| Score | Criteria |
|-------|----------|
| 9-10 | Refreshingly distinctive — finds a unique expression within the chosen philosophical framework |
| 7-8 | Has its own ideas; not a simple template application |
| 5-6 | Safe and conventional — looks like a template |
| 3-4 | Heavy use of clichés (e.g., gradient orbs to represent AI) |
| 1-2 | Entirely a template or clip-art assemblage |

**Review checklist**:
- Does it avoid common clichés? (see "Common Issues" list below)
- Is there personal expression within adherence to the design philosophy?
- Are there "unexpected yet perfectly fitting" design decisions?

---

## Scenario-Specific Review Focus

Different output types call for different review priorities:

| Scenario | Most Important | Second Priority | Can Relax |
|----------|---------------|-----------------|-----------|
| WeChat article cover / illustration | Originality, Visual Hierarchy | Philosophy Alignment | Functionality (single image, no interaction) |
| Infographic | Functionality, Visual Hierarchy | Craft Quality | Originality (accuracy first) |
| PPT / Keynote | Visual Hierarchy, Functionality | Craft Quality | Originality (clarity first) |
| PDF / Whitepaper | Craft Quality, Functionality | Visual Hierarchy | Originality (professionalism first) |
| Landing page / website | Functionality, Visual Hierarchy | Originality | — (full requirements) |
| App UI | Functionality, Craft Quality | Visual Hierarchy | Philosophy Alignment (usability first) |
| Xiaohongshu illustration | Originality, Visual Hierarchy | Philosophy Alignment | Craft Quality (atmosphere first) |

---

## Top 10 Common Design Problems

### 1. AI-tech clichés
**Problem**: Gradient orbs, digital rain, blue circuit boards, robot faces
**Why it's a problem**: Users are visually fatigued by these; you cannot be distinguished from everyone else
**Fix**: Replace literal symbols with abstract metaphors (e.g., use a "conversation" metaphor rather than a chat bubble icon)

### 2. Insufficient type-size hierarchy
**Problem**: Gap between headline and body is too small (<2.5×)
**Why it's a problem**: Users cannot quickly locate key information
**Fix**: Headlines should be at least 3× body size (e.g., body 16px → headline 48-64px)

### 3. Too many colors
**Problem**: Using 5+ colors with no clear primary/secondary relationship
**Why it's a problem**: Visual chaos, weak brand identity
**Fix**: Limit to 1 primary + 1 secondary + 1 accent + grayscale

### 4. Inconsistent spacing
**Problem**: Element spacing is arbitrary, with no system
**Why it's a problem**: Looks unprofessional; visual rhythm is disrupted
**Fix**: Establish an 8pt grid system (use only 8/16/24/32/48/64px for spacing)

### 5. Insufficient white space
**Problem**: Every area of space is filled with content
**Why it's a problem**: Crowded information causes reading fatigue and actually reduces information transmission efficiency
**Fix**: White space should occupy at least 40% of the total area (60%+ for minimalist styles)

### 6. Too many typefaces
**Problem**: Using 3+ fonts
**Why it's a problem**: Visual noise; undermines unity
**Fix**: Maximum 2 typefaces (1 for headlines + 1 for body); use weight and size to create variation

### 7. Inconsistent alignment
**Problem**: Some elements left-aligned, some centered, some right-aligned
**Why it's a problem**: Destroys the sense of visual order
**Fix**: Choose one alignment (left-align recommended) and apply it globally

### 8. Decoration over content
**Problem**: Background patterns/gradients/shadows overshadow the main content
**Why it's a problem**: Gets the priorities backwards — users come for information, not decoration
**Fix**: "If I remove this decoration, does the design get worse?" If not, remove it

### 9. Cyberpunk neon overuse
**Problem**: Dark blue background (`#0D1117`) + neon glow effects
**Why it's a problem**: Default aesthetic no-go zone (the taste baseline of this skill), and it has become one of the biggest clichés — users can override with their own brand
**Fix**: Choose a more distinctive color scheme (refer to the color systems of the 20 style categories)

### 10. Information density mismatched to medium
**Problem**: A full page of text on a slide / 10 elements crammed into a cover image
**Why it's a problem**: Different mediums have different optimal information densities
**Fix**:
- PPT: 1 core idea per slide
- Cover image: 1 visual focal point
- Infographic: layered presentation
- PDF: can be denser, but requires clear navigation

---

## Critique Output Template

```
## Design Critique Report

**Overall Score**: X.X/10 [Excellent (8+) / Good (6-7.9) / Needs Improvement (4-5.9) / Failing (<4)]

**Dimension Scores**:
- Philosophy Alignment: X/10 [one-sentence note]
- Visual Hierarchy: X/10 [one-sentence note]
- Craft Quality: X/10 [one-sentence note]
- Functionality: X/10 [one-sentence note]
- Originality: X/10 [one-sentence note]

### What's Working (Keep)
- [Specific callouts of what was done well, described in design language]

### Problems (Fix)
[Ordered by severity]

**1. [Issue name]** — ⚠️ Critical / ⚡ Important / 💡 Polish
- Current: [describe the present state]
- Problem: [why this is an issue]
- Fix: [specific action, including numerical values]

### Quick Wins
If you only have 5 minutes, prioritize these 3 things:
- [ ] [Highest-impact fix]
- [ ] [Second most important fix]
- [ ] [Third most important fix]
```

---

**Version**: v1.0
**Updated**: 2026-02-13
