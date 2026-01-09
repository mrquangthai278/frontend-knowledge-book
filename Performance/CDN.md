# CDN (Content Delivery Network)

## Definition / Concept
A **Content Delivery Network (CDN)** is a geographically distributed network of servers that cache and serve content (HTML, CSS, JavaScript, images, videos) from locations closest to end users. CDNs reduce **latency** and **bandwidth usage** by serving content from edge servers near the user instead of from a single origin server. This improves page load speed, reduces server load, and enhances user experience globally. CDNs are essential for modern web applications, especially when serving users across different continents.

- Distributes content across multiple geographic locations
- Serves content from the server closest to the user
- Reduces latency and improves load times
- Decreases origin server bandwidth costs
- Provides global scalability and reliability

## Visual Representation

```
Without CDN:
┌─────────────────────────────────────────────┐
│  User in Tokyo      User in New York        │
│  (Long distance)    (Long distance)         │
└────────┬───────────────────────────┬────────┘
         │                           │
         └───────────────┬───────────┘
                         │ (High latency)
                    ┌────▼─────┐
                    │ Origin    │
                    │ Server    │
                    └──────────┘

With CDN:
┌──────────────────────────────────┬────────────────────────────────┐
│ User in Tokyo                    │ User in New York               │
└────┬─────────────────────────────┴──────┬──────────────────────────┘
     │ (Low latency)                      │ (Low latency)
┌────▼──────────┐              ┌─────────▼────────┐
│ CDN Edge      │              │ CDN Edge         │
│ Tokyo         │              │ New York         │
└────┬──────────┘              └─────────┬────────┘
     │                                   │
     └─────────────────┬─────────────────┘
                       │ (Replicates from origin)
                  ┌────▼──────┐
                  │ Origin     │
                  │ Server     │
                  └───────────┘
```

## Example

```javascript
// Using CDN for third-party libraries
// Instead of bundling jQuery, serve from CDN

// HTML: Link from CDN
<script src="https://code.jquery.com/jquery-3.6.0.min.js"></script>

// Or for CSS frameworks
<link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/bootstrap@5.0.0/dist/css/bootstrap.min.css">

// ============================================
// Configuring a bundler to use CDN
// ============================================

// webpack.config.js
module.exports = {
  mode: 'production',
  output: {
    path: path.resolve(__dirname, 'dist'),
    publicPath: 'https://cdn.example.com/assets/',  // CDN URL
  },
  externals: {
    jquery: 'jQuery',  // Load jQuery from CDN, don't bundle
  },
};

// ============================================
// Using CDN for image optimization
// ============================================

// Original image URL
// https://example.com/images/photo.jpg (1MB)

// Through CDN with optimization
// https://cdn.example.com/images/photo.jpg?w=800&h=600&q=80
// Returns optimized version (50KB)

// ============================================
// Cache busting with CDN
// ============================================

// Version in URL for cache busting
<script src="https://cdn.example.com/app.js?v=1.2.3"></script>
<script src="https://cdn.example.com/app.js?v=1.2.4"></script>

// Or use content hash
<script src="https://cdn.example.com/app.a1b2c3d4.js"></script>

// ============================================
// HTML markup for CDN resources
// ============================================

<!DOCTYPE html>
<html lang="en">
<head>
  <!-- CSS from CDN -->
  <link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/bootstrap@5.0.0/dist/css/bootstrap.min.css">
</head>
<body>
  <div id="app"></div>

  <!-- Preload critical resources -->
  <link rel="preload" href="https://cdn.example.com/fonts/inter.woff2" as="font" crossorigin>

  <!-- DNS prefetch for CDN domain -->
  <link rel="dns-prefetch" href="https://cdn.example.com">

  <!-- JavaScript from CDN -->
  <script src="https://cdn.jsdelivr.net/npm/react@18/umd/react.production.min.js"></script>
  <script src="https://cdn.example.com/app.js" defer></script>
</body>
</html>
```

## Usage

- **When to use**: Serving static assets (JS, CSS, images, fonts), third-party libraries, videos, or any content to global audiences
- **Real-world example**:
  - Netflix uses CDN for video streaming globally
  - GitHub uses CDN for JavaScript library distribution (jsDelivr)
  - WordPress sites use Cloudflare CDN for image optimization
- **Best practices**:
  - Use CDN for static, cacheable content (images, CSS, JS, fonts)
  - Implement cache busting with version numbers or content hashes
  - Set appropriate Cache-Control headers on origin
  - Use DNS prefetch and preload hints for critical CDN resources
  - Monitor CDN performance and cost
  - Consider CDN for third-party scripts to reduce origin server load
  - Use HTTP/2 or HTTP/3 enabled CDNs for better performance
  - Enable gzip/brotli compression for text assets

## FAQ / Interview Questions

**Q: What is a CDN and why is it important?**
A: A CDN is a network of servers distributed globally that cache and serve content from locations closest to users. It's important because:
- Reduces **latency** by serving from nearby servers
- Improves page load speed and user experience
- Reduces bandwidth costs for origin servers
- Provides **redundancy** and **reliability** (if one server fails, another serves content)
- Essential for serving global audiences efficiently

**Q: How does a CDN work?**
A: CDNs work through these steps:
1. User requests content from a CDN URL
2. DNS resolves to the nearest edge server
3. Edge server checks cache for the content
4. If cached, returns immediately (cache hit)
5. If not cached (cache miss), fetches from origin server
6. Caches the content and returns to user
7. Subsequent users get faster responses from the cache

**Q: What's the difference between a CDN and a web server?**
A: **Web servers** store and serve your application from a single location. **CDNs** are distributed networks that cache content on multiple servers worldwide. CDNs are optimized for serving static content with low latency, while web servers run applications. Most modern architectures use both: a web server for dynamic content and a CDN for static assets.

**Q: How do I use a CDN in my project?**
A: There are several approaches:
1. **Link directly**: `<script src="https://cdn.example.com/lib.js"></script>`
2. **Configure build tools**: Set `publicPath` in Webpack to CDN URL
3. **Use service**: Upload assets to CDN service (Cloudflare, AWS CloudFront, etc.)
4. **Pull-zone**: CDN automatically pulls and caches from your origin server
5. **Push-zone**: You manually upload files to CDN storage

**Q: What should I put on a CDN vs. origin server?**
A: **On CDN (cacheable)**:
- JavaScript bundles
- CSS stylesheets
- Images and optimized images
- Fonts
- Videos
- PDFs and static documents

**On origin server (dynamic)**:
- HTML pages (can cache, but often needs validation)
- API endpoints
- Server-side rendered content
- Dynamic personalized content
- Form submissions

**Q: How do I handle cache invalidation with CDNs?**
A: CDNs need to know when content changes. Common strategies:
- **Version in URL**: `app.js?v=1.2.3` → change version, CDN serves new file
- **Content hash**: `app.a1b2c3d4.js` → hash changes with content, build tools handle it
- **Cache busting headers**: Use `Cache-Control: max-age=31536000` for immutable assets
- **Manual invalidation**: CDN providers allow purging specific files (slower, use rarely)
- **Automatic revalidation**: Use `ETag` headers so edge servers check origin periodically

## References
- [MDN Web Docs - Content Delivery Networks (CDN)](https://developer.mozilla.org/en-US/docs/Glossary/CDN)
- [Cloudflare - What is a CDN?](https://www.cloudflare.com/learning/cdn/what-is-a-cdn/)
- [Web.dev - Optimize resource loading](https://web.dev/optimize-resource-loading/)
- [HTTP Caching - MDN](https://developer.mozilla.org/en-US/docs/Web/HTTP/Caching)

---
*See also: [HTTP Caching](HTTPCaching.md), [Performance Optimization](../Performance/WebVitals.md), [DNS](DNS.md), [HTTP/2](HTTP2.md)*
