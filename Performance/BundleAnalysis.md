# Bundle Analysis

## Definition / Concept
**Bundle analysis** is the process of examining and understanding the size, composition, and content of JavaScript bundles generated during the build process. It helps developers identify **code bloat**, duplicate dependencies, unused code, and optimization opportunities. Bundle analysis is critical for improving **load times**, **Core Web Vitals**, and overall application performance in production environments.

- Measures the actual size of your JavaScript and asset bundles
- Identifies large or unnecessary dependencies
- Detects code splitting opportunities and unused exports
- Provides visibility into what's being shipped to users

## Visual Representation
```
Build Process
    ↓
[Source Code] → [Bundler (Webpack/Vite/Esbuild)] → [Output Bundle]
    ↓
Bundle Analysis Tools
    ↓
[Size Report] [Dependency Map] [Import Analysis]
    ↓
Optimization Insights
    ↓
Remove Unused Code | Code Split | Replace Dependencies | Lazy Load
```

## Build-Time Tools

Tools you use during development to analyze bundles at build time:

### Webpack Bundle Analyzer
```javascript
// webpack.config.js
const BundleAnalyzerPlugin = require('webpack-bundle-analyzer').BundleAnalyzerPlugin;

module.exports = {
  plugins: [
    new BundleAnalyzerPlugin({
      analyzerMode: 'static',
      reportFilename: 'bundle-report.html',
    })
  ]
};
```
Generates interactive HTML showing which modules consume the most space.

### Vite → rollup-plugin-visualizer
```javascript
// vite.config.js
import { visualizer } from 'rollup-plugin-visualizer';

export default {
  plugins: [visualizer({ open: true })]
};
```
Visualizes bundle composition in treemap format.

### Source Map Explorer
```bash
npm install -D source-map-explorer
source-map-explorer 'dist/*.js'
```
Shows what source code contributes to bundle size using source maps.

### esbuild metafile
```bash
esbuild input.js --metafile=meta.json --bundle
# Outputs detailed module breakdown in JSON
```
Low-level analysis of what's included in your bundle.

**Goal**: Identify which modules consume the most space, detect duplicates, and find tree-shaking failures.

## Delivery / Real-User Metrics

Tools to measure what actually reaches users and impacts performance:

### Chrome DevTools → Network
- Check **Content-Encoding** (gzip/brotli)
- Verify **gzip/brotli size** vs original size
- Monitor download time vs parse/execution time

### Lighthouse
```bash
# Run audit
lighthouse https://yoursite.com --view
```
Grades JavaScript bundle impact on **TTI** and **FCP**.

### WebPageTest
Real-world performance testing with actual browsers and network throttling.

### Real User Monitoring (RUM)
Monitor actual users with:
- **Sentry**: JS errors + performance monitoring
- **Datadog**: Full-stack RUM
- **New Relic**: Application performance monitoring

Tracks metrics like:
- **Initial JS payload** download time
- **Parse + execution time**
- **Main thread blocking time**

## Key Metrics to Monitor

Focus on these numbers:

### Bundle Size Metrics
- **Bundle size (gzip/brotli)**: What users actually download
- **Initial JS payload**: Critical path JavaScript
- **Per-route chunk size**: Size of lazy-loaded route bundles
- **Vendor vs app code ratio**: Should be ~40% vendor, ~60% app

### Performance Metrics
- **TTI (Time to Interactive)**: When app responds to user input
- **Main thread blocking time**: How long JS blocks user interactions
- **JS parse + execution time**: How long browsers spend parsing/running code

## Red Flags 🚩

Critical issues to watch for during analysis:

### Bundle Size Red Flags
- **vendor.js > 300–400KB gzip**: Vendor bundles over 400KB indicate bloat
- **Single giant chunk**: Code splitting not working—all code bundled together
- **Multiple versions of same dependency**: `npm ls [package]` reveals duplicates

### Large Libraries with Smaller Alternatives
- **moment.js** (67KB gzip) → **day.js** (2KB) or native Date
- **lodash** (70KB) → lodash-es (tree-shake individual functions) or native methods
- **chart.js** (35KB) → **recharts** (20KB) for React apps

### Tree-Shaking Failures
Common causes of dead code not being removed:
- **CommonJS instead of ES modules**: Tree-shaking only works with ES6 imports/exports
- **sideEffects misconfiguration**: Mark pure modules in package.json
  ```json
  {
    "sideEffects": false  // Only set if no side effects
  }
  ```
- **Dynamic imports**: `require()` or `import()` can't be statically analyzed

### Missing Code Splitting
- Entire app in single bundle → all code needed upfront
- Should split by: routes, features, vendors, third-party
- Lazy load non-critical routes with `React.lazy()` or dynamic imports

## FAQ / Interview Questions

**Q: What is the difference between bundle size and parsed/evaluated size?**
A: **Bundle size** is the compressed file size users download (gzipped). **Parsed size** is what the JavaScript engine parses and compiles (usually 3-4x larger). **Evaluated size** is memory used after execution. Example: 100KB gzip → 300KB parsed → 500KB evaluated. All three impact performance, but different stages matter differently.

**Q: How do you detect and fix duplicate dependencies?**
A: Use `npm ls [package-name]` to see all versions installed. For example:
```bash
npm ls react
# react@18.2.0 (primary)
#   react@17.0.2 (duplicate from old dependency)
```
Fix by: upgrading dependencies, using **npm audit** to patch, or explicitly installing a single version. Use tools like **npm dedupe** to consolidate where possible.

**Q: Why is gzipped size more important than actual file size?**
A: Users download the **gzipped version**, not raw bundles. Gzipping compresses text 60-80%, so a 350KB raw bundle becomes 100KB gzipped. The 100KB is what impacts load time and user experience. Always optimize for **gzipped/brotli size** in production.

**Q: How do you enable tree-shaking for your dependencies?**
A: Tree-shaking requires:
1. **ES6 modules** (not CommonJS): `import X from 'lib'` instead of `require('lib')`
2. **sideEffects: false** in package.json for pure modules
3. **Named imports**: `import { method } from 'lib'` instead of `import * as lib`

Example with lodash:
```javascript
// Bad: Full library bundled
import _ from 'lodash'
const result = _.map(arr, x => x * 2)

// Good: Only map() is bundled
import map from 'lodash-es'
const result = map(arr, x => x * 2)
```

**Q: What bundle size thresholds should you set?**
A: Typical budgets by app type:
- **SPA (React/Vue)**: 50-100KB gzip for initial load
- **Vendor chunk**: 40-100KB gzip
- **Per-route chunks**: 30-80KB gzip
- **Total app**: 150-300KB gzip depending on complexity

Set stricter budgets in CI to catch regressions early. Tools like **bundlesize** or **Lighthouse CI** can enforce these automatically.

## References
- [webpack-bundle-analyzer](https://github.com/webpack-contrib/webpack-bundle-analyzer)
- [Vite - Analyzing Bundle Size](https://vitejs.dev/guide/troubleshooting.html#analyzing-bundle-size)
- [Web.dev - Bundle Size Best Practices](https://web.dev/reduce-javascript-for-better-performance/)
- [source-map-explorer Documentation](https://github.com/danvk/source-map-explorer)
- [BundleStats - Bundle Analysis & Comparison](https://bundlestats.com/)

---
*See also: [Code Splitting](./CodeSplitting.md) | [Performance Metrics](./PerformanceMetrics.md) | [Tree Shaking](./TreeShaking.md)*
