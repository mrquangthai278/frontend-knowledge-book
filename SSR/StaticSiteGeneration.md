# Static Site Generation (SSG)

## Definition / Concept
**Static Site Generation (SSG)**, also called **pre-rendering** or **static generation**, is a technique where HTML pages are **generated at build time** and deployed as static files to a CDN or web server. Unlike SSR which generates HTML on each request, SSG pre-renders all pages once during the build process. This results in the **fastest possible performance** (no server processing needed), **lowest hosting costs**, and **best scalability**. SSG is ideal for content that doesn't change frequently (blogs, documentation, marketing sites). With **Incremental Static Regeneration (ISR)**, SSG can be combined with dynamic content updates.

- Generates HTML at build time
- Deploys static files (no server needed)
- Fastest performance (CDN-distributed)
- Lowest hosting costs
- Best for content that changes infrequently

## Visual Representation

```
STATIC SITE GENERATION PROCESS:

BUILD TIME:
┌──────────────────────────────────┐
│ npm run build                    │
│ 1. Fetch all data                │
│ 2. Render all pages to HTML      │
│ 3. Generate static files         │
└──────────────────────────────────┘
         ↓
┌──────────────────────────────────┐
│ Static HTML files:               │
│ ├── /index.html                  │
│ ├── /about.html                  │
│ ├── /blog/post-1.html            │
│ ├── /blog/post-2.html            │
│ └── ... (1000+ pages)            │
└──────────────────────────────────┘
         ↓ Deploy to CDN/Server

RUNTIME (Request):
Browser → CDN/Server → Static HTML (instant, no processing)
         (lightning fast)

COMPARISON:
┌─────────────┬─────────────┬──────────────┬─────────────┐
│ Feature     │ SSG         │ SSR          │ CSR         │
├─────────────┼─────────────┼──────────────┼─────────────┤
│ FCP         │ Fastest ⚡  │ Fast ⚡⚡    │ Slow        │
│ Build Time  │ Slow        │ None         │ None        │
│ Server Load │ None        │ High         │ None        │
│ Cost        │ Cheapest    │ Expensive    │ Cheap       │
│ SEO         │ Excellent   │ Good         │ Poor        │
│ Dynamic     │ Limited     │ Full         │ Full        │
└─────────────┴─────────────┴──────────────┴─────────────┘
```

## Example

```javascript
// ============================================
// 1. Next.js Static Generation (getStaticProps)
// ============================================

// pages/blog/[slug].js
export async function getStaticPaths() {
  // Generate paths at build time
  const posts = await fetch('/api/posts')
    .then(res => res.json());

  const paths = posts.map(post => ({
    params: { slug: post.slug }
  }));

  return {
    paths,
    fallback: false  // 404 for unknown paths
  };
}

export async function getStaticProps({ params }) {
  // Fetch data at build time
  const post = await fetch(`/api/posts/${params.slug}`)
    .then(res => res.json());

  return {
    props: { post },
    revalidate: 3600  // ISR: revalidate every hour
  };
}

export default function BlogPost({ post }) {
  return (
    <article>
      <h1>{post.title}</h1>
      <p>{post.content}</p>
      <p>Published: {post.date}</p>
    </article>
  );
}

// Build output:
// /blog/hello-world.html
// /blog/my-journey.html
// /blog/web-dev-tips.html
// ... (all posts pre-rendered)

// ============================================
// 2. Incremental Static Regeneration (ISR)
// ============================================

export async function getStaticProps({ params }) {
  const product = await fetchProduct(params.id);

  return {
    props: { product },
    revalidate: 60  // Revalidate every 60 seconds
    // At most, pages are regenerated once per minute
    // Stale content served while regenerating in background
  };
}

// With ISR:
// First request: Serve static HTML (fast)
// After 60s: Next request triggers regeneration
// While regenerating: Serve stale HTML
// After regeneration: Serve fresh HTML
// Subsequent requests: Serve new static HTML

// ============================================
// 3. On-Demand Revalidation (ISR)
// ============================================

// pages/api/revalidate.js
export default async function handler(req, res) {
  // Check secret token for security
  if (req.query.secret !== process.env.REVALIDATE_SECRET) {
    return res.status(401).json({ message: 'Invalid token' });
  }

  try {
    // Revalidate specific path
    await res.revalidate('/blog/my-post');
    return res.json({ revalidated: true });
  } catch (err) {
    return res.status(500).send('Error revalidating');
  }
}

// Usage: When content updates, call:
// curl /api/revalidate?secret=YOUR_SECRET

// ============================================
// 4. Static Generation with Gatsby
// ============================================

// gatsby-node.js
exports.createPages = async ({ graphql, actions }) => {
  const { createPage } = actions;

  // Query all data at build time
  const result = await graphql(`
    query {
      allMarkdownRemark {
        edges {
          node {
            fields {
              slug
            }
          }
        }
      }
    }
  `);

  // Create page for each result
  result.data.allMarkdownRemark.edges.forEach(({ node }) => {
    createPage({
      path: node.fields.slug,
      component: path.resolve(`./src/templates/blog-post.js`),
      context: {
        slug: node.fields.slug,
      },
    });
  });
};

// Gatsby generates static HTML for all pages at build time

// ============================================
// 5. Static Generation vs Dynamic Routes
// ============================================

// Pre-rendered at build time (static)
export async function getStaticPaths() {
  return {
    paths: [
      { params: { id: '1' } },
      { params: { id: '2' } },
      { params: { id: '3' } },
    ],
    fallback: false  // Other IDs return 404
  };
}

// With fallback: 'blocking' (generate on first request)
export async function getStaticPaths() {
  return {
    paths: [],
    fallback: 'blocking'  // Generate on first request (SSR)
  };
}

// With fallback: true (generate in background)
export async function getStaticPaths() {
  return {
    paths: [],
    fallback: true  // Show placeholder while generating
  };
}

// ============================================
// 6. Static Site Generation Workflow
// ============================================

// package.json
{
  "scripts": {
    "build": "next build && next export",  // Generate static files
    "start": "next start",  // Start server (for ISR)
    "export": "next export"  // Export as static HTML only
  }
}

// .env.production
NEXT_PUBLIC_API_URL=https://api.example.com

// next.config.js
module.exports = {
  exportPathMap: async function(defaultPathMap) {
    // Custom path mapping for static export
    return {
      '/': { page: '/' },
      '/about': { page: '/about' },
      '/contact': { page: '/contact' },
    };
  },
};

// ============================================
// 7. Static Generation with Nuxt
// ============================================

// nuxt.config.ts
export default defineNuxtConfig({
  nitro: {
    prerender: {
      // Generate these routes at build time
      routes: [
        '/',
        '/about',
        '/blog',
        '/blog/post-1',
        '/blog/post-2',
      ],
    },
  },
});

// pages/blog/[slug].vue
<script setup>
const route = useRoute();

// Fetch data (only runs at build time)
const post = await $fetch(`/api/posts/${route.params.slug}`);
</script>

<template>
  <article>
    <h1>{{ post.title }}</h1>
    <p>{{ post.content }}</p>
  </article>
</template>

// ============================================
// 8. Static Generation Benefits and Tradeoffs
// ============================================

// Benefits:
// ✓ Fastest performance (static files on CDN)
// ✓ Cheapest hosting (no server needed)
// ✓ Best SEO (static HTML)
// ✓ Scales infinitely (CDN handles traffic)
// ✓ Security (no server to attack)

// Tradeoffs:
// ✗ Slow build time (must render all pages)
// ✗ Can't handle real-time data
// ✗ Limited dynamic content
// ✗ Large builds (1000+ pages)
// Solution: Use ISR to regenerate on demand

// ============================================
// 9. Choosing SSG vs SSR vs CSR
// ============================================

// Use SSG when:
// - Content doesn't change often
// - Want fastest performance
// - Have small-to-medium number of pages
// - SEO is critical
// Example: Blog, documentation, marketing site

// Use ISR when:
// - Content changes occasionally
// - Want SSG speed + fresh content
// - Can wait a few seconds for regeneration
// Example: E-commerce product pages, news articles

// Use SSR when:
// - Content changes per request (personalized)
// - Need real-time data
// - Have unlimited dynamic routes
// Example: User profiles, real-time dashboards

// Use CSR when:
// - Building highly interactive app
// - SEO not critical
// - Want instant client-side routing
// Example: SPA, admin dashboard

// ============================================
// 10. Static Generation Optimization
// ============================================

// Optimize build time for large sites
export async function getStaticProps() {
  // Cache expensive computations
  const data = cache.get('posts') || await fetchPosts();
  cache.set('posts', data);

  return {
    props: { data },
    revalidate: 86400  // Cache for 24 hours
  };
}

// Parallelize data fetching
export async function getStaticProps() {
  const [posts, authors] = await Promise.all([
    fetchPosts(),
    fetchAuthors(),
  ]);

  return {
    props: { posts, authors },
    revalidate: 3600
  };
}

// Generate only critical pages at build time
export async function getStaticPaths() {
  // Generate top 100 posts at build time
  const popularPosts = await getPopularPosts(100);

  return {
    paths: popularPosts.map(post => ({
      params: { slug: post.slug }
    })),
    fallback: 'blocking'  // Generate others on first request
  };
}
```

## Usage

- **When to use**: Blogs, documentation, marketing sites, content that changes infrequently, need ultra-fast performance
- **Real-world example**: GitHub Pages (uses Jekyll for SSG), Vercel (Next.js SSG), Netlify (Gatsby SSG)
- **Best practices**:
  - Use SSG for content-heavy sites with infrequent updates
  - Combine with ISR for dynamic content with occasional updates
  - Use on-demand revalidation for content changes
  - Optimize build time with parallel data fetching
  - Monitor build times and optimize for large sites
  - Use fallback modes for handling unknown routes
  - Combine with CDN for global edge caching
  - Consider build time cost vs hosting savings

## FAQ / Interview Questions

**Q: What is Static Site Generation and what are its advantages?**
A: SSG generates HTML pages at **build time** and deploys them as static files. Advantages:
- **Fastest performance**: No server processing, CDN-served files
- **Lowest cost**: No server needed, CDN/static hosting is cheap
- **Best scalability**: Handles unlimited traffic (CDN scales)
- **Best SEO**: Static HTML is easily crawlable
- **Security**: No server-side vulnerabilities
It's ideal for blogs, documentation, and marketing sites.

**Q: How is SSG different from SSR?**
A: - **SSG**: Generates HTML at **build time**, static files, no server processing
   - **SSR**: Generates HTML at **request time**, server processes each request

**SSG**: Fastest, cheapest, best for static content
**SSR**: Slower, expensive, needed for dynamic/real-time content

**Q: What is Incremental Static Regeneration (ISR)?**
A: ISR combines SSG and SSR benefits:
- Pre-render pages at build time (SSG speed)
- Revalidate pages periodically (update fresh content)
- Generate on-demand when content changes
With ISR, you get fast static performance + fresh content updates. Pages are regenerated in the background without blocking requests.

**Q: How do I handle dynamic routes in SSG?**
A: Use `getStaticPaths()` to define which dynamic routes to pre-render:
```javascript
export async function getStaticPaths() {
  const posts = await fetchAllPosts();
  return {
    paths: posts.map(p => ({ params: { slug: p.slug } })),
    fallback: 'blocking'  // Generate others on first request
  };
}
```
With `fallback: 'blocking'`, unknown routes are generated on first request (ISR behavior).

**Q: What are the build time considerations for SSG?**
A: Large sites (1000+ pages) have long build times:
- **Optimize data fetching**: Use parallel requests, caching
- **Generate critical pages only**: Use `fallback: 'blocking'` for less common pages
- **Use ISR**: Generate less frequently accessed pages on-demand
- **Monitor build time**: Track and optimize slow rendering
- **Incremental builds**: Only rebuild changed pages (some tools support this)
For sites with thousands of pages, consider hybrid approach.

**Q: Can SSG handle real-time data?**
A: Pure SSG cannot (static files don't update without rebuild). Solutions:
- **ISR**: Revalidate pages periodically or on-demand
- **Client-side data fetching**: Fetch real-time data after page loads
- **WebSockets/polling**: Update content after initial page load
- **Hybrid**: SSG + client-side updates for real-time parts
For truly real-time content, SSR or CSR is better.

## References
- [Next.js - Static Generation](https://nextjs.org/docs/basic-features/pages#static-generation-recommended)
- [Gatsby - Static Site Generation](https://www.gatsbyjs.com/)
- [Web.dev - Rendering on the Web](https://web.dev/rendering-on-the-web/)
- [Incremental Static Regeneration](https://nextjs.org/docs/basic-features/data-fetching/incremental-static-regeneration)

---
*See also: [Server-Side Rendering (SSR)](ServerSideRendering.md), [Hydration](Hydration.md), [Incremental Static Regeneration (ISR)](IncrementalStaticRegeneration.md), [Caching Strategies](CachingStrategies.md)*
