# Tree Shaking

## Definition / Concept
**Tree shaking** is a build-time optimization technique that removes unused code (dead code) from JavaScript bundles. It relies on **static module analysis** to identify exports that are never imported, eliminating them during the bundling process. This reduces bundle size and improves application load time. Tree shaking works best with **ES6 modules** because they have a static structure that can be analyzed before runtime.

- Removes dead code automatically during bundling
- Requires ES6 module syntax (import/export)
- Enables smaller, faster applications
- Critical for performance optimization

## Visual Representation

```
┌─────────────────────────────────────┐
│   Source Code (All Functions)       │
│                                     │
│  export function used() { ... }     │
│  export function unused() { ... }   │
│  export function helper() { ... }   │
└─────────────────────────────────────┘
         │
         ↓ (Bundle & Analyze)
┌─────────────────────────────────────┐
│   Bundled Code (Only Used)          │
│                                     │
│  function used() { ... }            │
│  function helper() { ... }          │
│  // unused() removed                │
└─────────────────────────────────────┘
```

## Example

```javascript
// math.js - Utility module with multiple exports
export const add = (a, b) => a + b;
export const subtract = (a, b) => a - b;
export const multiply = (a, b) => a * b;
export const divide = (a, b) => a / b;
export const square = (a) => a * a;
export const cube = (a) => a * a * a;

// app.js - Only imports what's needed
import { add, multiply } from './math.js';

const result1 = add(5, 3);       // 8
const result2 = multiply(4, 2);  // 8

// subtract, divide, square, cube are NOT imported
```

**With tree shaking enabled:**
The bundler removes `subtract`, `divide`, `square`, and `cube` from the final bundle because they're never imported.

```javascript
// bundle.js - Final output (only what's used)
const add = (a, b) => a + b;
const multiply = (a, b) => a * b;

const result1 = add(5, 3);
const result2 = multiply(4, 2);
```

## Usage

- **When to use**: When using build tools like **Webpack**, **Rollup**, or **Vite** that support tree shaking
- **Real-world example**: Importing only `map` from Lodash instead of the entire library reduces bundle size significantly
- **Best practices**:
  - Use ES6 `import/export` syntax (not CommonJS `require`)
  - Configure your bundler with `production` mode
  - Mark side-effect-free modules in `package.json` with `"sideEffects": false`
  - Avoid dynamic imports that prevent static analysis
  - Regularly audit bundle size to ensure dead code is removed

## Production vs Development Tree Shaking

Tree shaking behavior can differ significantly between development and production builds:

### Development Build
- Tree shaking may be **disabled** or less aggressive (faster rebuild)
- Unused code often remains for easier debugging
- Webpack config: `mode: 'development'` skips optimization
- Result: Larger bundle, but readable code

### Production Build
- Tree shaking is **aggressive** and enabled by default
- Bundler uses **Terser** or similar to remove dead code
- Dead code elimination combined with minification
- Result: Smaller bundle, but harder to debug

**Example**: A utility file with 5 functions where only 1 is imported:
```javascript
// dev.bundle.js (300KB)
// All 5 functions included for easier debugging

// prod.bundle.js (100KB)
// Only 1 function included, other 4 removed by tree-shaking + minification
```

### Why Tree Shaking Might Fail in Production

- **CommonJS in dependencies**: If a third-party library uses CommonJS, tree shaking can't analyze it
- **sideEffects not configured**: Misconfigured `sideEffects` flag prevents removal
- **Dynamic imports**: Code using `require()` or dynamic `import()` can't be statically analyzed
- **Module concatenation disabled**: Webpack's module concatenation improves tree-shaking; if disabled, it may not work

## FAQ / Interview Questions

**Q: What is tree shaking and how does it work?**
A: Tree shaking is a code optimization technique that removes unused code during the bundling process. It analyzes which exports from a module are actually imported and used elsewhere. Only the "live" code paths are included in the final bundle, while unused exports are "shaken off" like dead leaves from a tree.

**Q: Why does tree shaking require ES6 modules?**
A: ES6 modules have a **static structure** where imports and exports are explicitly declared at the top level, making them analyzable at build time. CommonJS uses `require()` dynamically, which cannot be statically analyzed. Without knowing which modules are actually used, tree shaking cannot safely remove code.

**Q: How can I ensure tree shaking works in my project?**
A: Follow these steps:
1. Use ES6 module syntax (`import`/`export`)
2. Configure your bundler in **production mode** (Webpack, Rollup, Vite)
3. Add `"sideEffects": false` in `package.json` if your module has no side effects
4. Avoid default exports when possible (named exports are easier to tree shake)
5. Test bundle size with tools like `webpack-bundle-analyzer`

**Q: What's the difference between tree shaking and code splitting?**
A: **Tree shaking** removes unused code from modules. **Code splitting** divides code into separate chunks that load on demand. Both optimize bundle size but work differently—tree shaking eliminates dead code, while code splitting defers loading of non-critical code.

**Q: Can tree shaking remove side effects?**
A: Tree shaking is conservative—it won't remove code with **side effects** (code that modifies global state, imports with initialization logic, etc.). You can use `"sideEffects": false` in `package.json` to tell bundlers the module is safe to remove, but this should only be used if you're certain there are no side effects.

**Q: Why does tree shaking work differently in production vs development?**
A: In development, tree shaking is often disabled or minimized for faster rebuilds and easier debugging. In production, tree shaking is aggressive and combined with minification by Terser. This is why a 40% bundle increase can happen only in production—development might include debug code that gets tree-shaken in production, or vice versa. Always compare actual production builds to identify tree-shaking issues.

## References
- [MDN Web Docs - Tree shaking](https://developer.mozilla.org/en-US/docs/Glossary/Tree_shaking)
- [Webpack - Tree Shaking](https://webpack.js.org/guides/tree-shaking/)
- [Rollup - Tree Shaking](https://rollupjs.org/guide/en/#tree-shaking)
- [Vite - Features](https://vitejs.dev/guide/features.html)

---
*See also: [Code Splitting](./CodeSplitting.md) | [Bundle Optimization](./BundleOptimization.md) | [Lazy Loading](./LazyLoading.md)*
