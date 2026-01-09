# Incremental Static Regeneration (ISR)

## Definition / Concept
**Incremental Static Regeneration (ISR)** is a hybrid rendering strategy that combines the benefits of **Static Site Generation** and **Server-Side Rendering**. Pages are initially generated at build time as static files, but can be **revalidated and regenerated** periodically or on-demand during runtime. This allows serving **pre-rendered static content** with ultra-fast performance while keeping content **fresh and up-to-date**. ISR is the best of both worlds: SSG's speed and cost efficiency with SSR's ability to update content. It's ideal for e-commerce sites, blogs, and news sites that update frequently but not in real-time.

- Pre-renders pages at build time (SSG)
- Revalidates pages on a schedule or on-demand
- Serves fresh static content without full rebuild
- Combines SSG speed with dynamic content updates
- Perfect for frequently updated content with SSG performance

## Visual Representation

```
ISR WORKFLOW:

BUILD TIME (Initial):
┌──────────────────────────────────┐
│ npm run build                    │
│ 1. Generate all pages            │
│ 2. Create static files           │
│ 3. Deploy to server              │
└──────────────────────────────────┘
         ↓
┌──────────────────────────────────┐
│ Static HTML (Served fast)        │
│ ├── /products/item-1.html        │
│ ├── /products/item-2.html        │
│ └── ... (1000+ pages)            │
└──────────────────────────────────┘

RUNTIME (Request):
Request 1: Serve static HTML (instant)
    ↓
60 seconds pass (revalidate: 60)
    ↓
Request 2: Serve stale HTML + trigger regeneration
    ↓
Regeneration happens in background
    ↓
Request 3+: Serve fresh HTML

COMPARISON:
┌──────────────┬──────────┬───────┬──────────┐
│ Approach     │ FCP      │ Build │ Freshness│
├──────────────┼──────────┼───────┼──────────┤
│ SSG          │ Fastest⚡ │ Slow  │ Stale    │
│ ISR          │ Fast⚡⚡  │ Fast  │ Fresh✓   │
│ SSR          │ Slow     │ None  │ Real-time│
│ CSR          │ Slowest  │ None  │ Dynamic  │
└──────────────┴──────────┴───────┴──────────┘
```

## Example

```javascript
// ============================================
// 1. Basic ISR with Time-Based Revalidation
// ============================================

// pages/blog/[slug].js
export async function getStaticPaths() {
  // Pre-render top 10 popular posts
  const popularPosts = await getPopularPosts(10);

  return {
    paths: popularPosts.map(p => ({ params: { slug: p.slug } })),
    fallback: 'blocking'  // Generate other posts on first request
  };
}

export async function getStaticProps({ params }) {
  const post = await getPost(params.slug);

  return {
    props: { post },
    revalidate: 3600  // Revalidate every hour (in seconds)
  };
}

export default function BlogPost({ post }) {
  return (
    <article>
      <h1>{post.title}</h1>
      <p>{post.content}</p>
    </article>
  );
}

// Behavior:
// - Build time: Generates top 10 posts
// - First request to unknown post: Generate on demand (SSR-like)
// - After 60 minutes: Next request triggers background regeneration
// - While regenerating: Serve stale version
// - After regeneration: Serve fresh version

// ============================================
// 2. On-Demand Revalidation (Push-Based)
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
    // Or revalidate multiple paths
    await res.revalidate('/products/item-1');
    await res.revalidate('/products/item-2');

    return res.json({ revalidated: true });
  } catch (err) {
    return res.status(500).send('Error revalidating');
  }
}

// Usage: When blog post is published or updated
// Call: curl /api/revalidate?secret=YOUR_SECRET

// With webhook from CMS:
// CMS publishes post → Calls /api/revalidate → Post regenerated immediately

// ============================================
// 3. ISR with E-Commerce Product Pages
// ============================================

export async function getStaticPaths() {
  // Pre-render featured products
  const featured = await getFeatureProducts();

  return {
    paths: featured.map(p => ({ params: { id: p.id } })),
    fallback: 'blocking'  // Generate others on first request
  };
}

export async function getStaticProps({ params }) {
  const product = await getProductDetails(params.id);
  const reviews = await getProductReviews(params.id);

  return {
    props: {
      product,
      reviews
    },
    revalidate: 300  // Revalidate every 5 minutes
    // Product details may change (price, stock)
    // Reviews are updated frequently
  };
}

export default function ProductPage({ product, reviews }) {
  return (
    <div>
      <h1>{product.name}</h1>
      <p>${product.price}</p>
      <p>Stock: {product.stock}</p>
      <section>
        <h2>Reviews ({reviews.length})</h2>
        {reviews.map(r => <Review key={r.id} review={r} />)}
      </section>
    </div>
  );
}

// ============================================
// 4. ISR with Fallback Modes
// ============================================

// fallback: false - Only pre-rendered paths exist
export async function getStaticPaths() {
  return {
    paths: [{ params: { id: '1' } }, { params: { id: '2' } }],
    fallback: false  // 404 for /products/3
  };
}

// fallback: 'blocking' - Generate on first request (SSR)
export async function getStaticPaths() {
  return {
    paths: [],
    fallback: 'blocking'  // Generate all on first request (slow first load)
  };
}

// fallback: true - Show loading state while generating
export async function getStaticPaths() {
  return {
    paths: [],
    fallback: true  // isFallback: true while generating
  };
}

export default function Product({ product, isFallback }) {
  if (isFallback) {
    return <div>Loading...</div>;
  }

  return <div>{product.name}</div>;
}

// ============================================
// 5. ISR with CMS Webhook Integration
// ============================================

// pages/api/revalidate.js (Webhook handler)
export default async function handler(req, res) {
  // Verify webhook came from CMS
  const signature = req.headers['x-webhook-signature'];
  if (!verifySignature(signature)) {
    return res.status(401).json({ message: 'Invalid signature' });
  }

  try {
    const event = req.body;

    // Post published: revalidate post page
    if (event.type === 'post.published') {
      await res.revalidate(`/blog/${event.slug}`);
      return res.json({ revalidated: true, slug: event.slug });
    }

    // Product updated: revalidate product page
    if (event.type === 'product.updated') {
      await res.revalidate(`/products/${event.id}`);
      // Also revalidate category page that lists products
      await res.revalidate('/products');
      return res.json({ revalidated: true, id: event.id });
    }

    return res.status(400).json({ message: 'Unknown event' });
  } catch (err) {
    return res.status(500).send('Error revalidating');
  }
}

// CMS Configuration:
// On content update, send POST to:
// https://yoursite.com/api/revalidate?secret=YOUR_SECRET
// Body: { type: 'post.published', slug: 'hello-world' }

// ============================================
// 6. ISR with Hybrid Pages
// ============================================

// pages/index.js - Mixed static and dynamic content
export async function getStaticProps() {
  // Static featured content (rarely changes)
  const featured = await getFeaturedItems();

  return {
    props: { featured },
    revalidate: 86400  // Revalidate daily (24 hours)
  };
}

export default function Home({ featured }) {
  // Static featured section
  <section>
    <h2>Featured</h2>
    {featured.map(item => <Item key={item.id} {...item} />)}
  </section>

  // Dynamic section fetched on client (real-time)
  <section>
    <RecentlyViewed />  {/* Fetches from client */}
  </section>

  // Real-time data with polling
  <section>
    <LiveCounter />  {/* Updates in real-time */}
  </section>
}

// ============================================
// 7. ISR Monitoring and Logging
// ============================================

// pages/api/revalidate.js - With monitoring
export default async function handler(req, res) {
  if (req.query.secret !== process.env.REVALIDATE_SECRET) {
    return res.status(401).json({ message: 'Invalid token' });
  }

  const paths = req.body.paths || [req.query.path];
  const results = {
    success: [],
    failed: []
  };

  for (const path of paths) {
    try {
      console.log(`[ISR] Revalidating: ${path}`);
      await res.revalidate(path);
      results.success.push(path);

      // Log to monitoring service
      logToMonitoring({
        event: 'isr_success',
        path,
        timestamp: new Date()
      });
    } catch (err) {
      console.error(`[ISR] Failed: ${path}`, err);
      results.failed.push({ path, error: err.message });

      // Log error
      logToMonitoring({
        event: 'isr_failed',
        path,
        error: err.message,
        timestamp: new Date()
      });
    }
  }

  res.json(results);
}

// ============================================
// 8. ISR Performance Optimization
// ============================================

// Increase revalidation time for slow queries
export async function getStaticProps({ params }) {
  const data = await expensiveDataFetch(params.id);

  return {
    props: { data },
    revalidate: 3600  // 1 hour for expensive operations
  };
}

// Stale-While-Revalidate pattern
export async function getStaticProps() {
  const data = await getCachedData();

  return {
    props: { data },
    revalidate: 10  // Quick revalidate for fresh data
  };
}

// Parallel revalidation requests
async function revalidateMultiple(paths) {
  const results = await Promise.allSettled(
    paths.map(path => res.revalidate(path))
  );
  return results;
}

// ============================================
// 9. ISR vs Time-Based Cache
// ============================================

// ISR Approach
export async function getStaticProps() {
  return {
    props: { data },
    revalidate: 300  // Revalidate every 5 minutes
  };
}

// Alternative: Cache-Control header
export async function getServerSideProps() {
  return {
    props: { data },
    props: { data },
    headers: {
      'Cache-Control': 'public, max-age=300'  // Same effect
    }
  };
}

// ISR is better because:
// - Pre-renders during build (faster initial)
// - Serves from CDN (global distribution)
// - Regenerates in background (no blocking)

// ============================================
// 10. ISR Deployment Considerations
// ============================================

// next.config.js
module.exports = {
  // ISR settings
  swcMinify: true,  // Faster builds

  // Experimental features for ISR
  experimental: {
    isrMemoryCacheSize: 52 * 1024 * 1024,  // 52 MB cache
  },

  // Optimize image revalidation
  images: {
    unoptimizedImagesInDevMode: false,
  },
};

// Environment setup for ISR
// .env.production
ISR_SECRET=your-secret-key
DATABASE_URL=your-db-url

// Vercel deployment (automatic)
// - ISR works out of the box
// - Revalidation triggered by /api/revalidate
// - No additional setup needed

// Self-hosted (Node.js server required)
// - Next.js in production mode (next start)
// - ISR only works with server (not static export)
// - Requires persistent storage for regeneration state
```

## Usage

- **When to use**: E-commerce sites (products update regularly), blogs with frequent updates, news sites, documentation with occasional changes
- **Real-world example**: Vercel (uses ISR in Next.js), Shopify (product pages), Stripe Docs (API documentation)
- **Best practices**:
  - Use ISR for content that changes but not in real-time
  - Set appropriate revalidation times (balancing freshness vs performance)
  - Implement webhook-based on-demand revalidation for immediate updates
  - Monitor ISR performance and revalidation failures
  - Combine static and dynamic content (hybrid approach)
  - Use fallback modes strategically (blocking for popular pages, true for niche)
  - Test ISR behavior thoroughly before deployment
  - Track revalidation metrics and optimize

## FAQ / Interview Questions

**Q: What is Incremental Static Regeneration and how does it work?**
A: ISR combines SSG and SSR:
1. **Build time**: Generate pages as static files
2. **Runtime**: Serve pre-rendered static pages (fast)
3. **Background**: Periodically regenerate pages in background
4. **Update**: Serve fresh content after regeneration

Users get SSG speed while content stays fresh. It's the best of both worlds.

**Q: How is ISR different from SSR and SSG?**
A: - **SSG**: Static at build time, never updates (unless rebuild)
   - **ISR**: Static at build time, updates periodically/on-demand
   - **SSR**: Dynamic, generated per request

**SSG**: Fastest, cheapest, but stale content
**ISR**: Fast (static), cheap, keeps content fresh
**SSR**: Always fresh, expensive, slower

**Q: What's the difference between time-based and on-demand revalidation?**
A: - **Time-based** (`revalidate: 300`): Regenerate every 5 minutes automatically
   - **On-demand** (webhook): Regenerate immediately when content changes

Time-based is simpler. On-demand is faster for urgent updates. Combine both for best results.

**Q: How do I handle high traffic with ISR?**
A: ISR handles traffic well because:
- Pages are static (scale to infinite requests)
- Regeneration happens in background (doesn't block users)
- Can use CDN in front (global edge caching)
- Serve stale content while regenerating

For massive scale, use fallback modes strategically and monitor revalidation failures.

**Q: Can I revalidate multiple pages at once?**
A: Yes, with on-demand revalidation:
```javascript
const paths = ['/blog/post-1', '/blog/post-2', '/products'];
await Promise.all(paths.map(p => res.revalidate(p)));
```
Or use CMS webhooks to trigger multiple revalidations in one request.

**Q: What happens if revalidation fails?**
A: ISR gracefully handles failures:
- Old static content continues serving
- Revalidation retried on next request
- No user-facing errors
- Monitor and log failures for debugging

If revalidation frequently fails (data source down), consider fallback data or increased retry logic.

## References
- [Next.js - Incremental Static Regeneration](https://nextjs.org/docs/basic-features/data-fetching/incremental-static-regeneration)
- [Vercel - ISR Documentation](https://vercel.com/docs/concepts/next.js/incremental-static-regeneration)
- [Web.dev - Rendering Strategies](https://web.dev/rendering-on-the-web/)
- [Next.js On-Demand Revalidation](https://nextjs.org/docs/basic-features/data-fetching/incremental-static-regeneration#on-demand-revalidation)

---
*See also: [Server-Side Rendering (SSR)](ServerSideRendering.md), [Static Site Generation (SSG)](StaticSiteGeneration.md), [Hydration](Hydration.md), [Caching Strategies](CachingStrategies.md)*
