# Tweaks: Real-Time Design Variant Parameters

Tweaks is one of the core capabilities in this skill — letting users switch between variations and adjust parameters in real time without touching code.

**Cross-agent environment compatibility**: some native design-agent environments (like Claude.ai Artifacts) rely on the host's postMessage to write tweak values back into the source for persistence. This skill uses a **pure frontend localStorage approach** — same effect (state persists across refresh), but persistence happens in browser localStorage rather than source files. This approach works in any agent environment (Claude Code / Codex / Cursor / Trae / etc.).

## When to add Tweaks

- User explicitly asks for "adjustable parameters" / "multiple versions to compare"
- The design has multiple variations that need side-by-side comparison
- User didn't say anything, but you personally judge that **a few insightful tweaks would help the user see the possibility space**

Default recommendation: **add 2–3 tweaks to every design** (color theme / font size / layout variant) even when not asked — showing users the space of possibilities is part of the design service.

## Implementation (pure frontend version)

### Basic structure

```jsx
const TWEAK_DEFAULTS = {
  "primaryColor": "#D97757",
  "fontSize": 16,
  "density": "comfortable",
  "dark": false
};

function useTweaks() {
  const [tweaks, setTweaks] = React.useState(() => {
    try {
      const stored = localStorage.getItem('design-tweaks');
      return stored ? { ...TWEAK_DEFAULTS, ...JSON.parse(stored) } : TWEAK_DEFAULTS;
    } catch {
      return TWEAK_DEFAULTS;
    }
  });

  const update = (patch) => {
    const next = { ...tweaks, ...patch };
    setTweaks(next);
    try {
      localStorage.setItem('design-tweaks', JSON.stringify(next));
    } catch {}
  };

  const reset = () => {
    setTweaks(TWEAK_DEFAULTS);
    try {
      localStorage.removeItem('design-tweaks');
    } catch {}
  };

  return { tweaks, update, reset };
}
```

### Tweaks panel UI

Floating panel in the bottom-right. Collapsible:

```jsx
function TweaksPanel() {
  const { tweaks, update, reset } = useTweaks();
  const [open, setOpen] = React.useState(false);

  return (
    <div style={{
      position: 'fixed',
      bottom: 20,
      right: 20,
      zIndex: 9999,
    }}>
      {open ? (
        <div style={{
          background: 'white',
          border: '1px solid #e5e5e5',
          borderRadius: 12,
          padding: 20,
          boxShadow: '0 10px 40px rgba(0,0,0,0.12)',
          width: 280,
          fontFamily: 'system-ui',
          fontSize: 13,
        }}>
          <div style={{ 
            display: 'flex', 
            justifyContent: 'space-between', 
            alignItems: 'center',
            marginBottom: 16,
          }}>
            <strong>Tweaks</strong>
            <button onClick={() => setOpen(false)} style={{
              border: 'none', background: 'none', cursor: 'pointer', fontSize: 16,
            }}>×</button>
          </div>

          {/* Color */}
          <label style={{ display: 'block', marginBottom: 12 }}>
            <div style={{ marginBottom: 4, color: '#666' }}>Primary color</div>
            <input 
              type="color" 
              value={tweaks.primaryColor} 
              onChange={e => update({ primaryColor: e.target.value })}
              style={{ width: '100%', height: 32 }}
            />
          </label>

          {/* Font size slider */}
          <label style={{ display: 'block', marginBottom: 12 }}>
            <div style={{ marginBottom: 4, color: '#666' }}>Font size ({tweaks.fontSize}px)</div>
            <input 
              type="range" 
              min={12} max={24} step={1}
              value={tweaks.fontSize}
              onChange={e => update({ fontSize: +e.target.value })}
              style={{ width: '100%' }}
            />
          </label>

          {/* Density select */}
          <label style={{ display: 'block', marginBottom: 12 }}>
            <div style={{ marginBottom: 4, color: '#666' }}>Density</div>
            <select 
              value={tweaks.density}
              onChange={e => update({ density: e.target.value })}
              style={{ width: '100%', padding: 6 }}
            >
              <option value="compact">Compact</option>
              <option value="comfortable">Comfortable</option>
              <option value="spacious">Spacious</option>
            </select>
          </label>

          {/* Dark mode toggle */}
          <label style={{ 
            display: 'flex', 
            alignItems: 'center',
            gap: 8,
            marginBottom: 16,
          }}>
            <input 
              type="checkbox" 
              checked={tweaks.dark}
              onChange={e => update({ dark: e.target.checked })}
            />
            <span>Dark mode</span>
          </label>

          <button onClick={reset} style={{
            width: '100%',
            padding: '8px 12px',
            background: '#f5f5f5',
            border: 'none',
            borderRadius: 6,
            cursor: 'pointer',
            fontSize: 12,
          }}>Reset</button>
        </div>
      ) : (
        <button 
          onClick={() => setOpen(true)}
          style={{
            background: '#1A1A1A',
            color: 'white',
            border: 'none',
            borderRadius: 999,
            padding: '10px 16px',
            fontSize: 12,
            cursor: 'pointer',
            boxShadow: '0 4px 12px rgba(0,0,0,0.15)',
          }}
        >⚙ Tweaks</button>
      )}
    </div>
  );
}
```

### Applying Tweaks

Use Tweaks in the main component:

```jsx
function App() {
  const { tweaks } = useTweaks();

  return (
    <div style={{
      '--primary': tweaks.primaryColor,
      '--font-size': `${tweaks.fontSize}px`,
      background: tweaks.dark ? '#0A0A0A' : '#FAFAFA',
      color: tweaks.dark ? '#FAFAFA' : '#1A1A1A',
    }}>
      {/* Your content */}
      <TweaksPanel />
    </div>
  );
}
```

Use variables in CSS:

```css
button.cta {
  background: var(--primary);
  color: white;
  font-size: var(--font-size);
}
```

## Typical Tweak Options

What tweaks to add for different types of designs:

### General
- Primary color (color picker)
- Font size (slider 12–24px)
- Typeface (select: display font vs body font)
- Dark mode (toggle)

### Slide decks
- Theme (light/dark/brand)
- Background style (solid/gradient/image)
- Font contrast (more decorative vs more restrained)
- Information density (minimal/standard/dense)

### Product prototypes
- Layout variant (layout A / B / C)
- Interaction speed (animation speed 0.5x–2x)
- Data volume (mock data count: 5/20/100)
- State (empty/loading/success/error)

### Animations
- Speed (0.5x–2x)
- Loop (once/loop/ping-pong)
- Easing (linear/easeOut/spring)

### Landing pages
- Hero style (image/gradient/pattern/solid)
- CTA copy (several variants)
- Structure (single column / two column / sidebar)

## Tweaks Design Principles

### 1. Meaningful options, not noise

Every tweak must expose **real design choices**. Don't add tweaks that nobody would actually switch (e.g., a border-radius slider from 0–50px — users adjust it and find every middle value looks bad).

Good tweaks expose **discrete, considered variations**:
- "Corner style": sharp / slightly rounded / fully rounded (three options)
- Not: "border-radius": 0–50px slider

### 2. Less is more

A Tweaks panel for a single design should have **at most 5–6 options**. More than that and it becomes a "config page," losing its purpose as rapid variation exploration.

### 3. Defaults are the finished design

Tweaks are **a bonus**. The default values must already be a complete, shippable design on their own. What the user sees when they close the Tweaks panel is the output.

### 4. Group sensibly

When there are many options, display them in groups:

```
---- Visual ----
Primary color | Font size | Dark mode

---- Layout ----
Density | Sidebar position

---- Content ----
Data volume | State
```

## Forward-compatible with source-level persistence hosts

If you later want to upload the design to an environment that supports source-level tweaks (like Claude.ai Artifacts), keep the **EDITMODE marker block**:

```jsx
const TWEAK_DEFAULTS = /*EDITMODE-BEGIN*/{
  "primaryColor": "#D97757",
  "fontSize": 16,
  "density": "comfortable",
  "dark": false
}/*EDITMODE-END*/;
```

The marker block has **no effect** in the localStorage approach (it's just a regular comment), but in hosts that support writing back to source it will be picked up, enabling source-level persistence. Adding this is harmless in the current environment while keeping forward compatibility.

## Common Issues

**Tweaks panel is blocking the design content**
→ Make it closeable. Closed by default, showing only a small button — the user clicks to expand.

**User has to re-set tweaks after every interaction**
→ localStorage is already being used. If state doesn't persist after refresh, check whether localStorage is available (private/incognito mode will fail — must catch the error).

**Multiple HTML pages want to share tweaks**
→ Add a project name to the localStorage key: `design-tweaks-[projectName]`.

**I want tweaks to have linked/dependent behavior**
→ Add logic inside `update`:

```jsx
const update = (patch) => {
  let next = { ...tweaks, ...patch };
  // Linked behavior: switching to dark mode automatically changes text color
  if (patch.dark === true && !patch.textColor) {
    next.textColor = '#F0EEE6';
  }
  setTweaks(next);
  localStorage.setItem(...);
};
```
