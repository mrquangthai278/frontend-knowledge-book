# Hydration

## Definition / Concept
**Hydration** is the process of attaching **JavaScript interactivity** to server-rendered HTML. When using **Server-Side Rendering (SSR)**, the server sends pre-rendered HTML to the browser, but this HTML is static and non-interactive. Hydration involves downloading and executing JavaScript on the client side to "breathe life" into the static HTML, adding event listeners, state management, and interactivity. It's called "hydration" because it adds the missing dynamic layer to the initially dry, static HTML. Hydration is critical for SSR applications to maintain performance (fast initial paint) while preserving interactivity.

- Attaches JavaScript to server-rendered HTML
- Makes static HTML interactive
- Essential step in SSR applications
- Happens automatically with React, Vue, Next.js
- Can cause issues if server and client HTML don't match (mismatch)

## Visual Representation

```
SSR Hydration Process:

1. INITIAL REQUEST:
   Browser ────────────→ Server
            ←──────────── Pre-rendered HTML (static)

2. BROWSER RENDERS:
   HTML Parsed & Displayed (Fast, but no interaction)
   ┌─────────────────────────────────┐
   │  Static Content                 │
   │  Buttons (don't work)           │
   │  Forms (don't submit)           │
   │  Event handlers (not attached)  │
   └─────────────────────────────────┘

3. JAVASCRIPT DOWNLOADED & EXECUTED:
   ┌─────────────────────────────────┐
   │  JavaScript Bundle Downloaded   │
   │  React/Vue framework loaded      │
   │  Component tree created          │
   └─────────────────────────────────┘

4. HYDRATION HAPPENS:
   ┌─────────────────────────────────┐
   │  Event listeners attached       │
   │  State initialized              │
   │  Event handlers bound           │
   └─────────────────────────────────┘

   DOM becomes interactive ✓

TIMELINE:
Time → ─────────────────────────────────────────
       ↑ HTML arrives  ↑ JS loads   ↑ Hydrated
       (paint fast)    (download)   (interactive)
       FCP             LCP          TTI
```

## Example

```javascript
// ============================================
// Next.js Example with Hydration
// ============================================

// pages/index.js (runs on server and client)
export default function Home({ initialData }) {
  const [count, setCount] = React.useState(0);

  return (
    <div>
      <h1>Counter: {count}</h1>
      <button onClick={() => setCount(count + 1)}>
        Increment
      </button>
      <p>Data: {initialData}</p>
    </div>
  );
}

// Server renders this to HTML:
// <div>
//   <h1>Counter: 0</h1>
//   <button>Increment</button>
//   <p>Data: server-value</p>
// </div>

// Browser receives HTML (renders immediately - fast paint)
// Then JavaScript loads
// Hydration happens - button becomes clickable

// ============================================
// React Hydration with ReactDOM.hydrate()
// ============================================

// server.js (Node.js server)
import React from 'react';
import ReactDOMServer from 'react-dom/server';
import App from './App';

app.get('/', (req, res) => {
  const html = ReactDOMServer.renderToString(<App />);
  res.send(`
    <!DOCTYPE html>
    <html>
      <head>
        <title>SSR App</title>
      </head>
      <body>
        <div id="root">${html}</div>
        <script src="/app.js"></script>
      </body>
    </html>
  `);
});

// client.js (Browser)
import React from 'react';
import ReactDOM from 'react-dom';
import App from './App';

// Hydrate instead of render
ReactDOM.hydrate(<App />, document.getElementById('root'));

// hydrate() matches server-rendered HTML without re-rendering

// ============================================
// Hydration Mismatch Example
// ============================================

function MismatchExample() {
  const [mounted, setMounted] = React.useState(false);

  // Problem: This causes hydration mismatch
  React.useEffect(() => {
    setMounted(true);
  }, []);

  // Server renders: <div>Loading...</div>
  // Client renders first: <div>Loading...</div> (match ✓)
  // After effect: <div>Content loaded</div> (mismatch ✗)
  return mounted ? <div>Content loaded</div> : <div>Loading...</div>;
}

// Correct approach: Match server and client output
function CorrectExample() {
  return (
    <div>
      {/* Same output on server and client */}
      <h1>Welcome</h1>
      {/* Static content doesn't need useEffect */}
    </div>
  );
}

// ============================================
// Selective Hydration (React 18+)
// ============================================

import { hydrateRoot } from 'react-dom/client';

// Only hydrate specific components
const root = hydrateRoot(
  document.getElementById('root'),
  <App />
);

// React 18 enables progressive hydration
// Hydrates high-priority components first

// ============================================
// Deferred Hydration Pattern
// ============================================

function AppWithDeferredHydration() {
  const [isHydrated, setIsHydrated] = React.useState(false);

  React.useEffect(() => {
    setIsHydrated(true);
  }, []);

  if (!isHydrated) {
    // Show static content while hydrating
    return <StaticShell />;
  }

  return <InteractiveApp />;
}

// ============================================
// Next.js Hydration Example
// ============================================

// pages/_document.js (Next.js)
import { Html, Head, Main, NextScript } from 'next/document';

export default function Document() {
  return (
    <Html>
      <Head />
      <body>
        <Main />  {/* Server-rendered content */}
        <NextScript />  {/* Loads JavaScript for hydration */}
      </body>
    </Html>
  );
}

// pages/index.js
export async function getServerSideProps() {
  return {
    props: {
      initialData: 'from server'
    }
  };
}

export default function Home({ initialData }) {
  const [count, setCount] = React.useState(0);

  // Automatically hydrated by Next.js
  return (
    <div>
      <h1>{initialData}</h1>
      <button onClick={() => setCount(count + 1)}>
        {count}
      </button>
    </div>
  );
}

// ============================================
// Vue 3 Hydration Example
// ============================================

// server.ts (Node.js)
import { renderToString } from '@vue/server-renderer';
import { createApp } from 'vue';
import App from './App.vue';

app.get('/', async (req, res) => {
  const app = createApp(App);
  const html = await renderToString(app);
  res.send(`
    <!DOCTYPE html>
    <html>
      <body>
        <div id="app">${html}</div>
        <script src="/app.js"></script>
      </body>
    </html>
  `);
});

// client.ts (Browser)
import { createApp } from 'vue';
import App from './App.vue';

// Vue automatically hydrates on client
createApp(App).mount('#app');

// ============================================
// Common Hydration Issues
// ============================================

// Issue 1: Mismatch due to random data
function BadComponent() {
  const id = Math.random(); // Different on server and client!
  return <div id={id}>Content</div>;
}

// Issue 2: Browser-only APIs
function AnotherBad() {
  const href = window.location.href; // undefined on server!
  return <a href={href}>Link</a>;
}

// Issue 3: Conditional rendering based on environment
function YetAnother() {
  if (typeof window === 'undefined') {
    return <div>Server</div>;
  }
  return <div>Client</div>; // Mismatch!
}

// Correct solutions:
function GoodComponent() {
  const id = React.useId(); // Stable ID on server and client
  return <div id={id}>Content</div>;
}

function AnotherGood() {
  const [href, setHref] = React.useState('');
  React.useEffect(() => {
    setHref(window.location.href);
  }, []);
  return <a href={href}>Link</a>;
}

function YetAnotherGood() {
  const [isMounted, setIsMounted] = React.useState(false);

  React.useEffect(() => {
    setIsMounted(true);
  }, []);

  // Only render client-specific content after hydration
  if (!isMounted) {
    return <div>Loading...</div>;
  }

  return <div>Client content</div>;
}
```

## Usage

- **When to use**: Every SSR application needs hydration to become interactive; happens automatically with Next.js, Nuxt, or similar frameworks
- **Real-world example**: A Next.js e-commerce site renders product pages on the server for fast initial load, then hydration enables adding items to cart and checkout interactions
- **Best practices**:
  - Ensure server and client render the same HTML (avoid hydration mismatch)
  - Avoid using `Math.random()`, timestamps, or browser APIs in render logic
  - Use `useEffect` for client-only logic and browser API access
  - Keep hydration-critical code minimal for fast Time-to-Interactive
  - Use `React.useId()` instead of `Math.random()` for stable IDs
  - Consider progressive/selective hydration for large applications
  - Test SSR hydration thoroughly to catch mismatches
  - Monitor hydration performance metrics (TTI, INP)

## FAQ / Interview Questions

**Q: What is hydration and why is it necessary?**
A: Hydration is attaching JavaScript interactivity to server-rendered HTML. It's necessary because:
- **SSR sends static HTML**: Fast initial paint but non-interactive
- **Hydration adds JavaScript**: Makes it interactive (event listeners, state, etc.)
- **User experience**: Users see content immediately (FCP), then it becomes interactive (TTI)
- **Performance**: Combines fast paint with eventual full interactivity
- **SEO**: Server-rendered HTML is indexable by search engines
Without hydration, SSR-rendered pages would be static and non-functional.

**Q: What's the difference between hydration and rendering?**
A: **Rendering** creates HTML from components. **Hydration** connects JavaScript to existing HTML:
- **Rendering** (server): Converts React components to HTML strings
- **Hydration** (client): Connects JavaScript to pre-existing HTML without re-creating it
Hydration is more efficient than re-rendering because the browser doesn't need to create DOM nodes—it just attaches event listeners and state to existing elements.

**Q: What is a hydration mismatch and why is it a problem?**
A: A hydration mismatch occurs when the server and client render different HTML. This happens when you:
```javascript
// Wrong: Different random value on server and client
<div>{Math.random()}</div>

// Wrong: window undefined on server
<div>{window.location.href}</div>

// Wrong: Conditional rendering with different results
{typeof window === 'undefined' ? 'Server' : 'Client'}
```
Mismatches cause React to re-render the entire page, defeating the purpose of hydration and harming performance. Always ensure server and client render identical HTML until `useEffect` runs.

**Q: How do I avoid hydration mismatches?**
A: Follow these practices:
1. Use stable values: `React.useId()` instead of `Math.random()`
2. Access browser APIs only in `useEffect`: `window`, `localStorage`, etc.
3. Keep render logic identical on server and client
4. Avoid random data or timestamps in render logic
5. Use conditional rendering cautiously
6. Test SSR + hydration thoroughly
7. Use suppressHydrationWarning as last resort (not recommended)

**Q: What's the difference between ReactDOM.render() and ReactDOM.hydrate()?**
A: - **render()**: Creates new DOM nodes from scratch, replaces existing content
- **hydrate()**: Connects JavaScript to existing HTML without re-creating nodes
For SSR, always use `hydrate()` on the client. Using `render()` would re-render everything, losing the SSR benefit. React 18 introduced `hydrateRoot()` for improved hydration.

**Q: How do frameworks like Next.js and Nuxt handle hydration automatically?**
A: These frameworks:
1. Render components on the server to HTML
2. Send HTML + JavaScript bundle to browser
3. Browser executes JavaScript (Next.js/Nuxt code)
4. Framework automatically calls hydration method
5. Same components run on client, hydrate existing HTML
You don't manually call hydrate functions—the framework handles it. Just write components that work on both server and client.

## References
- [MDN Web Docs - Server-side rendering](https://developer.mozilla.org/en-US/docs/Glossary/Server-side_rendering)
- [React - ReactDOM.hydrate()](https://react.dev/reference/react-dom/hydrate)
- [Next.js - Data Fetching and Hydration](https://nextjs.org/docs/basic-features/data-fetching)
- [Web.dev - Rendering on the Web](https://web.dev/rendering-on-the-web/)

---
*See also: [Server-Side Rendering (SSR)](../WebStandards/ServerSideRendering.md), [Static Site Generation (SSG)](StaticSiteGeneration.md), [Component Lifecycle](ComponentLifecycle.md)*
