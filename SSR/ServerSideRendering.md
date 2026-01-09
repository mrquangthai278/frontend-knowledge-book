# Server-Side Rendering (SSR)

## Definition / Concept
**Server-Side Rendering (SSR)** is a technique where HTML is generated on the **server** during request time and sent to the browser, rather than rendering on the client using JavaScript. The server executes the application code, generates complete HTML with data, and sends it to the browser. This enables **faster initial page load**, **better SEO** (search engines can easily crawl pre-rendered HTML), and **improved performance** on slow devices. SSR is ideal for content-heavy applications, blogs, and e-commerce sites where initial paint speed and search visibility matter.

- Generates HTML on server during request
- Improves initial page load time (FCP)
- Better SEO (pre-rendered content is crawlable)
- Requires JavaScript for hydration to add interactivity
- Higher server load compared to static hosting

## Visual Representation

```
CLIENT-SIDE RENDERING (CSR):
Browser ─────→ Server (empty HTML)
        ←───── HTML with app.js script
        └─→ Download app.js
        └─→ Execute JavaScript
        └─→ Generate HTML (slow FCP)
        └─→ Render DOM (TTI)

SERVER-SIDE RENDERING (SSR):
Browser ─────→ Server (process request)
        │      Server renders component
        │      Server fetches data
        ←───── Complete HTML (fast FCP)
        └─→ Display immediately
        └─→ Download JavaScript
        └─→ Hydrate (TTI)

STATIC GENERATION (SSG):
Build time → Server pre-renders all pages
           → HTML files stored as static
Browser ─→ CDN/Server
       ←── Static HTML (fastest FCP)
       → Download JavaScript (TTI)
```

## Example

```javascript
// ============================================
// 1. Basic SSR Setup (Node.js + Express)
// ============================================

import express from 'express';
import React from 'react';
import ReactDOMServer from 'react-dom/server';
import App from './App';

const app = express();

app.get('/', (req, res) => {
  // Render component on server
  const html = ReactDOMServer.renderToString(<App />);

  // Send complete HTML
  res.send(`
    <!DOCTYPE html>
    <html>
      <head>
        <title>SSR App</title>
        <meta charset="UTF-8">
      </head>
      <body>
        <div id="root">${html}</div>
        <script src="/bundle.js"></script>
      </body>
    </html>
  `);
});

app.listen(3000, () => console.log('SSR server running'));

// ============================================
// 2. SSR with Data Fetching
// ============================================

// Server: Fetch data before rendering
app.get('/products', async (req, res) => {
  try {
    // Fetch data on server
    const products = await fetchProductsFromDB();

    // Pass data to component
    const html = ReactDOMServer.renderToString(
      <ProductList products={products} />
    );

    res.send(`
      <!DOCTYPE html>
      <html>
        <body>
          <div id="root">${html}</div>
          <script>
            window.__INITIAL_STATE__ = ${JSON.stringify({ products })};
          </script>
          <script src="/bundle.js"></script>
        </body>
      </html>
    `);
  } catch (error) {
    res.status(500).send('Server error');
  }
});

// Client: Hydrate with initial data
import { hydrateRoot } from 'react-dom/client';

const initialState = window.__INITIAL_STATE__;
hydrateRoot(
  document.getElementById('root'),
  <ProductList products={initialState.products} />
);

// ============================================
// 3. Next.js SSR Example
// ============================================

// pages/products/[id].js
export async function getServerSideProps({ params }) {
  // Fetch data at request time
  const product = await fetch(`/api/products/${params.id}`)
    .then(res => res.json());

  return {
    props: {
      product,
      revalidate: 60  // ISR: revalidate every 60 seconds
    }
  };
}

export default function ProductPage({ product }) {
  return (
    <div>
      <h1>{product.name}</h1>
      <p>{product.description}</p>
      <p>${product.price}</p>
    </div>
  );
}

// Next.js automatically:
// 1. Calls getServerSideProps on server
// 2. Renders component to HTML
// 3. Sends HTML to browser (fast FCP)
// 4. Client hydrates with JavaScript

// ============================================
// 4. Vue 3 SSR Example
// ============================================

// server.ts
import express from 'express';
import { renderToString } from '@vue/server-renderer';
import { createApp } from 'vue';
import App from './App.vue';

const app = express();

app.get('/', async (req, res) => {
  const vueApp = createApp(App);
  const html = await renderToString(vueApp);

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

// client.ts
import { createApp } from 'vue';
import App from './App.vue';

createApp(App).mount('#app');  // Hydrates automatically

// ============================================
// 5. SSR vs CSR vs SSG Comparison
// ============================================

// SSR - Best for: Dynamic content, real-time data
export async function getServerSideProps() {
  const data = await fetchFreshData(); // Fetched on each request
  return { props: { data } };
}

// SSG - Best for: Static content, blog posts
export async function getStaticProps() {
  const data = await fetchData(); // Fetched at build time
  return {
    props: { data },
    revalidate: 3600  // Revalidate every hour
  };
}

// CSR - Best for: Highly interactive apps
// No server-side rendering, component renders in browser

// ============================================
// 6. SSR Performance Optimization
// ============================================

// Stream rendering for faster FCP
app.get('/', (req, res) => {
  res.write(`
    <!DOCTYPE html>
    <html>
      <head><title>App</title></head>
      <body>
        <div id="root">
  `);

  // Start streaming content immediately
  const stream = ReactDOMServer.renderToNodeStream(<App />);
  stream.pipe(res, { end: false });

  stream.on('end', () => {
    res.write(`
        </div>
        <script src="/bundle.js"></script>
      </body>
    </html>
    `);
    res.end();
  });
});

// ============================================
// 7. Common SSR Issues
// ============================================

// Issue 1: Browser APIs in render
function BadComponent() {
  const width = window.innerWidth; // Error on server!
  return <div>{width}</div>;
}

// Solution: Use effect for browser APIs
function GoodComponent() {
  const [width, setWidth] = React.useState(0);

  React.useEffect(() => {
    setWidth(window.innerWidth);
  }, []);

  return <div>{width}</div>;
}

// Issue 2: Data fetching race conditions
function DataComponent() {
  const [data, setData] = React.useState(null);

  // This might miss server data!
  React.useEffect(() => {
    fetchData().then(setData);
  }, []);

  return <div>{data}</div>;
}

// Solution: Initialize from server data
function BetterDataComponent({ serverData }) {
  const [data, setData] = React.useState(serverData);

  React.useEffect(() => {
    // Only refetch if needed
    if (!data) {
      fetchData().then(setData);
    }
  }, []);

  return <div>{data}</div>;
}

// ============================================
// 8. Caching SSR Responses
// ============================================

app.get('/', (req, res) => {
  // Cache HTML for 1 hour
  res.set('Cache-Control', 'public, max-age=3600');

  const html = ReactDOMServer.renderToString(<App />);
  res.send(renderHTML(html));
});

// Or use middleware for caching
const cache = new Map();

function cachedSSR(req, res) {
  const cacheKey = req.url;

  if (cache.has(cacheKey)) {
    return res.send(cache.get(cacheKey));
  }

  const html = ReactDOMServer.renderToString(<App />);
  const fullHTML = renderHTML(html);

  // Cache for 5 minutes
  cache.set(cacheKey, fullHTML);
  setTimeout(() => cache.delete(cacheKey), 5 * 60 * 1000);

  res.send(fullHTML);
}

// ============================================
// 9. Error Handling in SSR
// ============================================

app.get('/', async (req, res) => {
  try {
    const data = await fetchData();
    const html = ReactDOMServer.renderToString(
      <App initialData={data} />
    );
    res.send(renderHTML(html));
  } catch (error) {
    // Send error page
    res.status(500).send(renderHTML(
      <ErrorPage error={error.message} />
    ));
  }
});

// ============================================
// 10. SSR with Next.js Middleware
// ============================================

// middleware.ts (Next.js)
import { NextRequest, NextResponse } from 'next/server';

export function middleware(request: NextRequest) {
  // Can add authentication, redirects, headers, etc.
  const response = NextResponse.next();

  // Add security headers
  response.headers.set('X-Content-Type-Options', 'nosniff');
  response.headers.set('X-Frame-Options', 'DENY');

  return response;
}

export const config = {
  matcher: ['/admin/:path*'],
};
```

## Usage

- **When to use**: Content-heavy sites (blogs, news, e-commerce), need SEO, need fast initial load, dynamic content per request
- **Real-world example**: News websites (CNN, BBC) use SSR for fast initial paint and SEO; e-commerce (Amazon, Shopify) uses SSR for product pages
- **Best practices**:
  - Use SSR for content-heavy, SEO-critical pages
  - Combine with ISR (Incremental Static Regeneration) for hybrid approach
  - Cache SSR responses to reduce server load
  - Implement streaming rendering for faster FCP
  - Keep server-side logic minimal to avoid bottlenecks
  - Use CDN in front of SSR server for edge caching
  - Monitor server performance and scaling
  - Use tools like Next.js, Nuxt, or Remix for easier SSR
  - Test hydration thoroughly to avoid mismatches

## FAQ / Interview Questions

**Q: What is Server-Side Rendering and what are its benefits?**
A: SSR generates HTML on the server during request time and sends it to the browser. Benefits:
- **Faster FCP (First Contentful Paint)**: Browser shows content immediately
- **Better SEO**: Pre-rendered HTML is easily crawlable
- **Works on slow devices**: No JavaScript required for initial render
- **Better performance metrics**: LCP and FID improved
- **Dynamic content**: Can fetch fresh data per request
Downside: Higher server load and complexity compared to static generation.

**Q: How is SSR different from CSR (Client-Side Rendering)?**
A: - **SSR**: Server generates HTML, browser receives complete page, then hydrates
   - **CSR**: Browser receives empty HTML, downloads JavaScript, renders in browser

**SSR advantages**: Faster initial load, better SEO, works on slow networks
**CSR advantages**: Lower server load, faster subsequent navigations, better UX for interactive apps

**Q: What is hydration in SSR context?**
A: Hydration is attaching JavaScript interactivity to server-rendered HTML. The server sends static HTML (fast), the browser displays it immediately (FCP), then JavaScript loads and hydrates the page (adds event listeners, state, interactivity). Hydration is essential for SSR to work properly.

**Q: Should I use SSR, SSG, or ISR?**
A: Choose based on content type:
- **SSR**: Dynamic content that changes per request (user profiles, real-time data)
- **SSG**: Static content that doesn't change often (blog posts, landing pages)
- **ISR (Incremental Static Regeneration)**: Best of both—static with periodic revalidation
- **Hybrid**: Use different strategies for different pages (Next.js supports all three)

**Q: What are common SSR pitfalls?**
A: - **Hydration mismatches**: Server and client render different HTML
- **Browser APIs in render**: `window`, `localStorage` cause errors
- **Data fetching conflicts**: Server and client both fetch
- **Performance**: Slow server rendering blocks FCP
- **Scaling**: High server load with many concurrent requests
Avoid with proper testing and using frameworks that handle SSR correctly.

**Q: How do I scale an SSR application?**
A: - **Caching**: Cache rendered HTML for repeated requests
- **CDN**: Use edge caching for static assets
- **Load balancing**: Distribute server load across multiple servers
- **ISR/SSG**: Use static generation where possible
- **Streaming**: Use streaming rendering for faster FCP
- **Optimize rendering**: Keep server-side logic fast
- **Monitoring**: Monitor server performance and scale proactively

## References
- [Web.dev - Rendering on the Web](https://web.dev/rendering-on-the-web/)
- [Next.js - Data Fetching](https://nextjs.org/docs/basic-features/data-fetching)
- [Nuxt - Server-Side Rendering](https://nuxt.com/docs/guide/concepts/rendering)
- [React - Server Components](https://react.dev/reference/rsc/server-components)

---
*See also: [Hydration](Hydration.md), [Static Site Generation (SSG)](StaticSiteGeneration.md), [Incremental Static Regeneration (ISR)](IncrementalStaticRegeneration.md), [Caching Strategies](CachingStrategies.md)*
