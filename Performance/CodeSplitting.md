# Code Splitting

## Definition / Concept
**Code splitting** is a technique that divides a JavaScript bundle into smaller, separate chunks that load on demand rather than loading everything upfront. This reduces the **initial payload size**, improves **Time to Interactive (TTI)**, and allows browsers to fetch only the code needed for the current route or feature. Code splitting works by using **dynamic imports** or **lazy loading** to defer non-critical code.

- Reduces initial bundle size by deferring non-critical code
- Enables faster page load and Time to Interactive
- Works by splitting into route chunks, vendor chunks, and app chunks
- Essential for optimizing large Single Page Applications (SPAs)

## Visual Representation

```
┌─────────────────────────────────────────┐
│        Initial Bundle (All Code)        │
│                                         │
│   App Logic + Routes + Vendor Code      │
│   Size: 500KB (uncompressed)            │
└─────────────────────────────────────────┘
           │
           ↓ (Code Splitting)
┌──────────────────────────────────────────────────────────┐
│           Split into Multiple Chunks                    │
├──────────────────────────────────────────────────────────┤
│ main.js (50KB)   - Core app logic, router              │
│ vendor.js (150KB) - React, React-DOM, dependencies     │
│ route-home.js (40KB) - Home page code (lazy load)     │
│ route-profile.js (60KB) - Profile page (lazy load)   │
│ route-dashboard.js (70KB) - Dashboard (lazy load)    │
└──────────────────────────────────────────────────────────┘
           │
           ↓ (Delivery)
┌───────────────────────────────────────────────────┐
│ User loads app:                                   │
│ 1. Download: main.js + vendor.js (200KB)          │
│ 2. Display initial page                           │
│ 3. User navigates → download route-profile.js    │
│ 4. User visits dashboard → download route-dashboard.js │
└───────────────────────────────────────────────────┘
```

## Example

### Route-Based Code Splitting (React.lazy)

```javascript
// router.js - Using React.lazy for route-based splitting
import React, { lazy, Suspense } from 'react';
import { BrowserRouter, Routes, Route } from 'react-router-dom';

// Lazy load pages - each gets its own chunk
const Home = lazy(() => import('./pages/Home'));
const Profile = lazy(() => import('./pages/Profile'));
const Dashboard = lazy(() => import('./pages/Dashboard'));

function App() {
  return (
    <BrowserRouter>
      <Suspense fallback={<div>Loading...</div>}>
        <Routes>
          <Route path="/" element={<Home />} />
          <Route path="/profile" element={<Profile />} />
          <Route path="/dashboard" element={<Dashboard />} />
        </Routes>
      </Suspense>
    </BrowserRouter>
  );
}

export default App;
```

### Dynamic Imports (Modern JavaScript)

```javascript
// Without code splitting - loads everything upfront
import { expensiveLibrary } from './heavy-module';

// With dynamic import - loads only when needed
async function handleButtonClick() {
  const { expensiveLibrary } = await import('./heavy-module');
  expensiveLibrary.process();
}

button.addEventListener('click', handleButtonClick);
```

### Webpack Configuration (Manual Chunks)

```javascript
// webpack.config.js - Vendor and app code splitting
module.exports = {
  mode: 'production',
  entry: './src/index.js',
  output: {
    filename: '[name].[contenthash].js',
    path: path.resolve(__dirname, 'dist'),
  },
  optimization: {
    splitChunks: {
      chunks: 'all',
      cacheGroups: {
        // Extract vendor code into separate chunk
        vendor: {
          test: /[\\/]node_modules[\\/]/,
          name: 'vendors',
          priority: 10,
          reuseExistingChunk: true,
        },
        // Extract common code shared between chunks
        common: {
          minChunks: 2,
          priority: 5,
          reuseExistingChunk: true,
          name: 'common',
        },
      },
    },
  },
};
```

## Usage

- **When to use**: For SPAs with multiple routes or lazy-loaded features; when initial bundle exceeds 100-150KB gzip
- **Real-world example**: E-commerce site loads home page chunk immediately but defers checkout, admin, and account settings chunks until user navigates there
- **Best practices**:
  - Split by route boundaries (most common pattern)
  - Separate vendor code from app code
  - Monitor chunk sizes to prevent individual chunks from becoming too large
  - Use `Suspense` boundaries to show loading states
  - Measure impact using bundle analysis tools
  - Avoid over-splitting (too many small chunks increases HTTP overhead)

## FAQ / Interview Questions

**Q: What is the difference between code splitting and tree shaking?**
A: **Code splitting** divides a bundle into separate chunks that load on demand (different timing). **Tree shaking** removes unused code from modules during bundling (same bundle). Both optimize size but work differently: tree shaking eliminates dead code, code splitting defers loading non-critical code. You can use both together—split chunks and tree-shake within each chunk.

**Q: How does route-based code splitting improve performance?**
A: Route-based splitting defers loading page components until users navigate to them. Example: A user visits your home page and downloads only the home chunk (40KB). When they navigate to `/profile`, the profile chunk (60KB) downloads in background. Initial page loads faster because the browser isn't downloading code for pages the user might never visit.

**Q: Can you create too many code splits?**
A: Yes. Too many small chunks increase **HTTP request overhead**—each chunk requires a separate network request. Modern HTTP/2 reduces this penalty, but general guidance is: avoid splits smaller than 20-30KB. One large unnecessary chunk (400KB) is worse than multiple medium chunks (100KB each), but 10 tiny chunks (10KB each) creates request overhead. Balance is key.

**Q: How do you measure if code splitting is actually helping?**
A: Compare metrics:
1. **Initial JS payload**: Does main chunk load faster? (Check Chrome DevTools Network tab)
2. **Time to Interactive (TTI)**: Is page interactive sooner? (Lighthouse, Web Vitals)
3. **Bundle size**: Is initial chunk smaller? (webpack-bundle-analyzer)
4. **User experience**: Does Suspense fallback show? Time from navigation to chunk load? (Real User Monitoring)

Example: If initial chunk drops from 300KB to 100KB but TTI is the same, tree-shaking or minification may be the limiting factor, not code splitting.

**Q: What happens if a lazy-loaded chunk fails to download?**
A: The component wrapped in Suspense will remain in loading state indefinitely. Best practice is to add error boundaries and timeout handling:
```javascript
// Error boundary catches chunk load failures
class ErrorBoundary extends React.Component {
  state = { hasError: false };
  static getDerivedStateFromError() {
    return { hasError: true };
  }
  render() {
    if (this.state.hasError) {
      return <div>Failed to load component. Please refresh.</div>;
    }
    return this.props.children;
  }
}

<ErrorBoundary>
  <Suspense fallback={<Loading />}>
    <LazyComponent />
  </Suspense>
</ErrorBoundary>
```

## References
- [webpack - Code Splitting](https://webpack.js.org/guides/code-splitting/)
- [React - Code Splitting with React.lazy()](https://react.dev/reference/react/lazy)
- [MDN - Dynamic Import](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/import)
- [web.dev - Code Splitting](https://web.dev/reduce-javascript-for-better-performance/#code-split)
- [Vite - Dynamic Import](https://vitejs.dev/guide/features.html#dynamic-import)

---
*See also: [Bundle Analysis](./BundleAnalysis.md) | [Tree Shaking](./TreeShaking.md) | [Bundle Optimization](./BundleOptimization.md) | [Lazy Loading](./LazyLoading.md)*
