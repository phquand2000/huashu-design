# React + Babel Project Standards

Technical standards that must be followed when building prototypes with HTML + React + Babel. Violating these will break things.

## Pinned Script Tags (use these exact versions)

Put these three script tags in the HTML `<head>`, using **pinned versions + integrity hashes**:

```html
<script src="https://unpkg.com/react@18.3.1/umd/react.development.js" integrity="sha384-hD6/rw4ppMLGNu3tX5cjIb+uRZ7UkRJ6BPkLpg4hAu/6onKUg4lLsHAs9EBPT82L" crossorigin="anonymous"></script>
<script src="https://unpkg.com/react-dom@18.3.1/umd/react-dom.development.js" integrity="sha384-u6aeetuaXnQ38mYT8rp6sbXaQe3NL9t+IBXmnYxwkUI2Hw4bsp2Wvmx4yRQF1uAm" crossorigin="anonymous"></script>
<script src="https://unpkg.com/@babel/standalone@7.29.0/babel.min.js" integrity="sha384-m08KidiNqLdpJqLq95G/LEi8Qvjl/xUYll3QILypMoQ65QorJ9Lvtp2RXYGBFj1y" crossorigin="anonymous"></script>
```

**Do not** use unpinned versions like `react@18` or `react@latest` — they cause version drift and caching issues.

**Do not** omit `integrity` — it's your defense line if a CDN gets hijacked or tampered with.

## File Structure

```
project-name/
├── index.html               # Main HTML
├── components.jsx           # Component file (loaded with type="text/babel")
├── data.js                  # Data file
└── styles.css               # Extra CSS (optional)
```

Loading order in HTML:

```html
<!-- React + Babel first -->
<script src="https://unpkg.com/react@18.3.1/..."></script>
<script src="https://unpkg.com/react-dom@18.3.1/..."></script>
<script src="https://unpkg.com/@babel/standalone@7.29.0/..."></script>

<!-- Then your component files -->
<script type="text/babel" src="components.jsx"></script>
<script type="text/babel" src="pages.jsx"></script>

<!-- Main entry last -->
<script type="text/babel">
  const root = ReactDOM.createRoot(document.getElementById('root'));
  root.render(<App />);
</script>
```

**Do not** use `type="module"` — it conflicts with Babel.

## Three Non-Negotiable Rules

### Rule 1: styles objects must use unique names

**Wrong** (will always break with multiple components):
```jsx
// components.jsx
const styles = { button: {...}, card: {...} };

// pages.jsx  ← same name, overwrites!
const styles = { container: {...}, header: {...} };
```

**Right**: use a unique prefix for the styles in each component file.

```jsx
// terminal.jsx
const terminalStyles = { 
  screen: {...}, 
  line: {...} 
};

// sidebar.jsx
const sidebarStyles = { 
  container: {...}, 
  item: {...} 
};
```

**Or use inline styles** (recommended for small components):
```jsx
<div style={{ padding: 16, background: '#111' }}>...</div>
```

This rule is **non-negotiable**. Every time you write `const styles = {...}`, you must replace it with a specific name, or you'll get full-stack errors when multiple components load.

### Rule 2: Scope is not shared — you must manually export

**Key insight**: Every `<script type="text/babel">` is compiled independently by Babel — **scopes don't cross**. A `Terminal` component defined in `components.jsx` is **undefined by default** in `pages.jsx`.

**Solution**: At the end of each component file, export the components/utilities you want to share to `window`:

```jsx
// End of components.jsx
function Terminal(props) { ... }
function Line(props) { ... }
const colors = { green: '#...', red: '#...' };

Object.assign(window, {
  Terminal, Line, colors,
  // list everything you want to use elsewhere
});
```

Then `pages.jsx` can use `<Terminal />` directly, because JSX will look it up at `window.Terminal`.

### Rule 3: Never use scrollIntoView

`scrollIntoView` pushes the entire HTML container upward, breaking the web harness layout. **Never use it.**

Alternative:
```js
// Scroll to a position inside a container
container.scrollTop = targetElement.offsetTop;

// Or use element.scrollTo
container.scrollTo({
  top: targetElement.offsetTop - 100,
  behavior: 'smooth'
});
```

## Calling the Claude API (in-HTML)

Some native design-agent environments (like Claude.ai Artifacts) have a zero-config `window.claude.complete`, but most agent environments (Claude Code / Codex / Cursor / Trae / etc.) **don't have it** locally.

If your HTML prototype needs to call an LLM for a demo (e.g., a chat interface), you have two options:

### Option A: Don't make a real call — use a mock

Recommended for demo scenarios. Write a fake helper that returns preset responses:
```jsx
window.claude = {
  async complete(prompt) {
    await new Promise(r => setTimeout(r, 800)); // simulate latency
    return "This is a mock response. Replace with a real API when deploying.";
  }
};
```

### Option B: Call the Anthropic API for real

Requires an API key — the user must enter their own key into the HTML to run it. **Never hardcode a key in the HTML.**

```html
<input id="api-key" placeholder="Paste your Anthropic API key" />
<script>
window.claude = {
  async complete(prompt) {
    const key = document.getElementById('api-key').value;
    const res = await fetch('https://api.anthropic.com/v1/messages', {
      method: 'POST',
      headers: {
        'x-api-key': key,
        'anthropic-version': '2023-06-01',
        'content-type': 'application/json',
      },
      body: JSON.stringify({
        model: 'claude-haiku-4-5',
        max_tokens: 1024,
        messages: [{ role: 'user', content: prompt }]
      })
    });
    const data = await res.json();
    return data.content[0].text;
  }
};
</script>
```

**Note**: Calling the Anthropic API directly from the browser will hit CORS issues. If the user's preview environment doesn't support CORS bypass, this won't work. In that case use Option A mock, or tell the user they need a proxy backend.

### Option C: Use the agent-side LLM to generate mock data

If you just need it for local demo purposes, you can use the current agent session's LLM (or a multi-model skill the user has installed) to generate mock response data in advance, then hardcode it into the HTML. The HTML at runtime then has zero API dependencies.

## Typical HTML Starter Template

Copy this template as the skeleton for React prototypes:

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Your Prototype Name</title>

  <!-- React + Babel pinned -->
  <script src="https://unpkg.com/react@18.3.1/umd/react.development.js" integrity="sha384-hD6/rw4ppMLGNu3tX5cjIb+uRZ7UkRJ6BPkLpg4hAu/6onKUg4lLsHAs9EBPT82L" crossorigin="anonymous"></script>
  <script src="https://unpkg.com/react-dom@18.3.1/umd/react-dom.development.js" integrity="sha384-u6aeetuaXnQ38mYT8rp6sbXaQe3NL9t+IBXmnYxwkUI2Hw4bsp2Wvmx4yRQF1uAm" crossorigin="anonymous"></script>
  <script src="https://unpkg.com/@babel/standalone@7.29.0/babel.min.js" integrity="sha384-m08KidiNqLdpJqLq95G/LEi8Qvjl/xUYll3QILypMoQ65QorJ9Lvtp2RXYGBFj1y" crossorigin="anonymous"></script>

  <style>
    * { box-sizing: border-box; margin: 0; padding: 0; }
    html, body { height: 100%; width: 100%; }
    body { 
      font-family: -apple-system, 'SF Pro Text', sans-serif;
      background: #FAFAFA;
      color: #1A1A1A;
    }
    #root { min-height: 100vh; }
  </style>
</head>
<body>
  <div id="root"></div>

  <!-- Your component files -->
  <script type="text/babel" src="components.jsx"></script>

  <!-- Main entry -->
  <script type="text/babel">
    const { useState, useEffect } = React;

    function App() {
      return (
        <div style={{padding: 40}}>
          <h1>Hello</h1>
        </div>
      );
    }

    const root = ReactDOM.createRoot(document.getElementById('root'));
    root.render(<App />);
  </script>
</body>
</html>
```

## Common Errors and Fixes

**`styles is not defined` or `Cannot read property 'button' of undefined`**
→ You defined `const styles` in one file and another file overwrote it. Give each one a specific name.

**`Terminal is not defined`**
→ Cross-file references don't share scope. Add `Object.assign(window, {Terminal})` at the end of the file where Terminal is defined.

**Entire page is blank, no console errors**
→ Most likely a JSX syntax error that Babel didn't surface in the console. Temporarily swap `babel.min.js` with the unminified `babel.js` — error messages will be clearer.

**ReactDOM.createRoot is not a function**
→ Wrong version. Confirm you're using react-dom@18.3.1 (not 17 or anything else).

**`Objects are not valid as a React child`**
→ You're rendering an object instead of JSX/string. Usually `{someObj}` when you meant `{someObj.name}`.

## How to Split Large Projects into Files

**Single files over 1000 lines** are hard to maintain. Here's a splitting strategy:

```
project/
├── index.html
├── src/
│   ├── primitives.jsx      # Base elements: Button, Card, Badge...
│   ├── components.jsx      # Business components: UserCard, PostList...
│   ├── pages/
│   │   ├── home.jsx        # Home page
│   │   ├── detail.jsx      # Detail page
│   │   └── settings.jsx    # Settings page
│   ├── router.jsx          # Simple routing (React state switching)
│   └── app.jsx             # Entry component
└── data.js                 # Mock data
```

Load in order in HTML:
```html
<script type="text/babel" src="src/primitives.jsx"></script>
<script type="text/babel" src="src/components.jsx"></script>
<script type="text/babel" src="src/pages/home.jsx"></script>
<script type="text/babel" src="src/pages/detail.jsx"></script>
<script type="text/babel" src="src/pages/settings.jsx"></script>
<script type="text/babel" src="src/router.jsx"></script>
<script type="text/babel" src="src/app.jsx"></script>
```

**At the end of every file**, use `Object.assign(window, {...})` to export what needs to be shared.
