# Verification: Output Validation Process

Some design-agent native environments (like Claude.ai Artifacts) have a built-in `fork_verifier_agent` that spins up a subagent to check via iframe screenshots. Most agent environments (Claude Code / Codex / Cursor / Trae / etc.) don't have this built-in capability — doing it manually with Playwright covers the same validation scenarios.

## Verification Checklist

After producing any HTML output, run through this checklist once:

### 1. Browser Render Check (Required)

The most basic check: **can the HTML be opened?** On macOS:

```bash
open -a "Google Chrome" "/path/to/your/design.html"
```

Or use Playwright for a screenshot (see next section).

### 2. Console Error Check

The most common problem in HTML files is a JS error causing a white screen. Run it through Playwright:

```bash
python ~/.claude/skills/claude-design/scripts/verify.py path/to/design.html
```

This script will:
1. Open the HTML in headless Chromium
2. Save a screenshot to the project directory
3. Capture console errors
4. Report status

See `scripts/verify.py` for details.

### 3. Multi-Viewport Check

For responsive designs, capture multiple viewports:

```bash
python verify.py design.html --viewports 1920x1080,1440x900,768x1024,375x667
```

### 4. Interaction Check

Tweaks, animations, button toggles — a static screenshot won't show these. **Recommend having the user open a browser and click through it themselves**, or record with Playwright:

```python
page.video.record('interaction.mp4')
```

### 5. Slide-by-Slide Check for Decks

For deck-style HTML, screenshot each slide:

```bash
python verify.py deck.html --slides 10  # capture first 10 slides
```

Generates `deck-slide-01.png`, `deck-slide-02.png`... for easy browsing.

## Playwright Setup

First-time use requires:

```bash
# If not yet installed
npm install -g playwright
npx playwright install chromium

# Or the Python version
pip install playwright
playwright install chromium
```

If the user already has Playwright installed globally, use it directly.

## Screenshot Best Practices

### Capture Full Page

```python
page.screenshot(path='full.png', full_page=True)
```

### Capture Viewport

```python
page.screenshot(path='viewport.png')  # captures only the visible area by default
```

### Capture Specific Element

```python
element = page.query_selector('.hero-section')
element.screenshot(path='hero.png')
```

### High-DPI Screenshot

```python
page = browser.new_page(device_scale_factor=2)  # retina
```

### Wait for Animation to Settle Before Capturing

```python
page.wait_for_timeout(2000)  # wait 2 seconds for animations to settle
page.screenshot(...)
```

## Sharing Screenshots with Users

### Open Local Screenshots Directly

```bash
open screenshot.png
```

The user will see it in their own Preview / Figma / VSCode / browser.

### Upload to Image Host and Share Link

If you need to share with remote collaborators (e.g. Slack / Feishu / WeChat), have the user use their own image hosting tool or MCP to upload:

```bash
python ~/Documents/写作/tools/upload_image.py screenshot.png
```

Returns a permanent ImgBB link that can be pasted anywhere.

## When Verification Fails

### White Screen

The console will definitely have errors. Check first:

1. Is the React+Babel script tag integrity hash correct (see `react-setup.md`)
2. Is there a naming conflict like `const styles = {...}`
3. Do cross-file components have `export` to `window`
4. JSX syntax errors (babel.min.js doesn't report errors clearly — switch to babel.js unminified)

### Animation Stutters

- Record a segment in Chrome DevTools Performance tab
- Look for layout thrashing (frequent reflows)
- Prioritize `transform` and `opacity` for animation (GPU-accelerated)

### Wrong Font

- Check whether the `@font-face` URL is accessible
- Check fallback fonts
- Chinese fonts load slowly: show fallback first, switch when loaded

### Layout Misalignment

- Check whether `box-sizing: border-box` is applied globally
- Check `* { margin: 0; padding: 0 }` reset
- Open gridlines in Chrome DevTools to see actual layout

## Verification = The Designer's Second Set of Eyes

**Always review it yourself.** When AI writes code, these things often happen:

- Looks right but the interaction has a bug
- Static screenshot looks fine but scrolling breaks it
- Looks great on wide screen but breaks on narrow screen
- Dark mode was forgotten
- Some components don't respond after tweaks are toggled

**1 minute of verification at the end can save 1 hour of rework.**

## Common Verification Script Commands

```bash
# Basic: open + screenshot + capture errors
python verify.py design.html

# Multiple viewports
python verify.py design.html --viewports 1920x1080,375x667

# Multiple slides
python verify.py deck.html --slides 10

# Output to specific directory
python verify.py design.html --output ./screenshots/

# headless=false, opens real browser for you to see
python verify.py design.html --show
```
