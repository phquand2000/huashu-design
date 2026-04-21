# Design Context: Starting from Existing Context

**This is the single most important thing about this skill.**

Good hi-fi design always grows from existing design context. **Creating hi-fi from scratch is a last resort and will always produce generic work.** So at the start of every design task, ask: is there anything I can reference?

## What Is Design Context

In order from highest to lowest priority:

### 1. User's Design System / UI Kit
The component library, color tokens, typography guidelines, and icon system from the user's existing product. **The ideal scenario.**

### 2. User's Codebase
If the user provides a codebase, it has living component implementations. Read those component files:
- `theme.ts` / `colors.ts` / `tokens.css` / `_variables.scss`
- Concrete components (Button.tsx, Card.tsx)
- Layout scaffold (App.tsx, MainLayout.tsx)
- Global stylesheets

**Read the code and copy exact values**: hex codes, spacing scale, font stack, border radius. Don't redraw from memory.

### 3. User's Published Product
If the user has a live product but hasn't provided code, use Playwright or ask the user to provide screenshots.

```bash
# Screenshot a public URL with Playwright
npx playwright screenshot https://example.com screenshot.png --viewport-size=1920,1080
```

This lets you see the real visual vocabulary.

### 4. Brand Guidelines / Logo / Existing Assets
The user may have: logo files, brand color specs, marketing materials, slide templates. All of these are context.

### 5. Competitor References
The user says "like XX website" — have them provide a URL or screenshot. **Don't** rely on your hazy training data memory.

### 6. Known Design Systems (fallback)
If none of the above exist, use a well-regarded design system as a base:
- Apple HIG
- Material Design 3
- Radix Colors (color palette)
- shadcn/ui (components)
- Tailwind default palette

Tell the user explicitly what you're using, so they know this is a starting point, not a final design.

## Process for Getting Context

### Step 1: Ask the User

Required questions at the start of any task (from `workflow.md`):

```markdown
1. Do you have an existing design system / UI kit / component library? Where is it?
2. Do you have brand guidelines, color / font specs?
3. Can you give me screenshots or a URL of your existing product?
4. Is there a codebase I can read?
```

### Step 2: When the User Says "None," Help Them Find One

Don't give up immediately. Try:

```markdown
Let me look for clues:
- Do any of your previous projects have related designs?
- What colors / fonts does your company's marketing website use?
- What style is your product's logo? Can you send me one?
- Is there a product you admire that I can use as a reference?
```

### Step 3: Read All the Context You Can Find

If the user provides a codebase path, read:
1. **List the file structure first**: Find files related to styles/theme/components
2. **Read theme/token files**: Lift specific hex/px values
3. **Read 2–3 representative components**: See the visual vocabulary (hover state, shadow, border, padding patterns)
4. **Read global stylesheet**: Base resets, font loading
5. **If there's a Figma link/screenshot**: Look at it, but **trust the code more**

**Important**: **Don't** glance at something and work from impression. You've truly lifted the context only when you have 30+ specific values.

### Step 4: Vocalize the System You're Going to Use

After reviewing the context, tell the user what system you'll be using:

```markdown
Based on your codebase and product screenshots, here's the design system I've extracted:

**Colors**
- Primary: #C27558 (from tokens.css)
- Background: #FDF9F0
- Text: #1A1A1A
- Muted: #6B6B6B

**Typography**
- Display: Instrument Serif (from @font-face in global.css)
- Body: Geist Sans
- Mono: JetBrains Mono

**Spacing** (from your scale system)
- 4, 8, 12, 16, 24, 32, 48, 64

**Shadow pattern**
- `0 1px 2px rgba(0,0,0,0.04)` (subtle card)
- `0 10px 40px rgba(0,0,0,0.1)` (elevated modal)

**Border-radius**
- Small components 4px, cards 12px, buttons 8px

**Component vocabulary**
- Button: filled primary, outlined secondary, ghost tertiary — all 8px border-radius
- Card: white background, subtle shadow, no border

I'll start building with this system. Confirm this looks right?
```

Confirm with the user before proceeding.

## Designing Without Context (Fallback When There's Nothing)

**Strong warning**: Output quality will drop significantly in this situation. Tell the user clearly.

```markdown
You have no design context, so I can only work from general intuition.
The output will be "looks okay but lacks distinctiveness."
Would you like to continue, or first gather some reference materials?
```

If the user insists, make decisions in this order:

### 1. Choose an Aesthetic Direction
Don't produce a generic result. Pick a clear direction:
- brutally minimal
- editorial/magazine
- brutalist/raw
- organic/natural
- luxury/refined
- playful/toy
- retro-futuristic
- soft/pastel

Tell the user which one you've chosen.

### 2. Choose a Known Design System as the Skeleton
- Use Radix Colors for the color palette (https://www.radix-ui.com/colors)
- Use shadcn/ui for component vocabulary (https://ui.shadcn.com)
- Use Tailwind spacing scale (multiples of 4)

### 3. Choose Distinctive Font Pairings

Don't use Inter/Roboto. Suggested combinations (from Google Fonts):
- Instrument Serif + Geist Sans
- Cormorant Garamond + Inter Tight
- Bricolage Grotesque + Söhne (paid)
- Fraunces + Work Sans (note: Fraunces is already overused by AI)
- JetBrains Mono + Geist Sans (technical feel)

### 4. Every Key Decision Has Reasoning

Don't make choices silently. Write in the HTML comments:

```html
<!--
Design decisions:
- Primary color: warm terracotta (oklch 0.65 0.18 25) — fits the "editorial" direction  
- Display: Instrument Serif for humanist, literary feel
- Body: Geist Sans for cleanness contrast
- No gradients — committed to minimal, no AI slop
- Spacing: 8px base, golden ratio friendly (8/13/21/34)
-->
```

## Import Strategy (When the User Provides a Codebase)

If the user says "import this codebase as reference":

### Small (<50 files)
Read everything, internalize the context.

### Medium (50–500 files)
Focus on:
- `src/components/` or `components/`
- All styles/tokens/theme-related files
- 2–3 representative full-page components (Home.tsx, Dashboard.tsx)

### Large (>500 files)
Have the user specify the focus:
- "I want to build a settings page" → read existing settings-related files
- "I want to build a new feature" → read the overall shell + the closest reference
- Don't try to read everything — be targeted

## Working with Figma / Design Files

If the user provides a Figma link:

- **Don't** expect to directly "convert Figma to HTML" — that requires additional tools
- Figma links are usually not publicly accessible
- Ask the user to: export as **screenshots** to send you + tell you the specific color/spacing values

If only given a Figma screenshot, tell the user:
- I can see the visuals, but I can't extract precise values
- Please tell me key numbers (hex, px), or export as code (Figma supports this)

## Final Reminder

**The ceiling on a project's design quality is determined by the quality of context you receive.**

Spending 10 minutes collecting context is more valuable than spending 1 hour drawing hi-fi from scratch.

**When you encounter a context-free situation, first ask the user for it — don't just forge ahead.**
