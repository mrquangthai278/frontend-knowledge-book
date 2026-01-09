# Module Script

## Definition / Concept
A **module script** is a JavaScript file executed with `<script type="module">` that uses **ES6 module syntax** (import/export). Module scripts have their own scope, preventing global namespace pollution and allowing explicit dependency management. They are automatically **deferred** (execute after HTML parsing), support **dynamic imports**, and enable **tree shaking** by bundlers. Module scripts are the modern standard for organizing JavaScript code and are essential for building scalable, maintainable applications.

- Uses ES6 import/export syntax
- Automatically deferred execution
- Creates isolated scope (no global leaking)
- Enables tree shaking and code splitting
- Supports dynamic imports for lazy loading

## Visual Representation

```
Regular Script (Global Scope Pollution):
┌─────────────────────────────┐
│  Global Scope               │
│  ├── function utils() { }   │
│  ├── const config = {}      │
│  ├── var x = 1              │
│  └── ... (pollutes global)  │
└─────────────────────────────┘

Module Script (Isolated Scope):
┌─────────────────────────────┐
│  Module 1 Scope             │
│  ├── function utils() { }   │
│  └── export const config    │
└─────────────────────────────┘
         │ (explicit imports)
         ↓
┌─────────────────────────────┐
│  Module 2 Scope             │
│  ├── import { config }      │
│  └── export function main() │
└─────────────────────────────┘
         (No global pollution)
```

## Example

```html
<!-- Regular script (pollutes global scope) -->
<script src="app.js"></script>

<!-- Module script (isolated scope) -->
<script type="module">
  export const add = (a, b) => a + b;
  const privateHelper = () => {};  // Not global
</script>

<!-- Import from module -->
<script type="module">
  import { add } from './math.js';
  console.log(add(5, 3));
</script>

<!-- Dynamic import for code splitting -->
<script type="module">
  const btn = document.getElementById('btn');
  btn.addEventListener('click', async () => {
    const module = await import('./feature.js');
    module.initialize();
  });
</script>

<!-- Fallback for older browsers -->
<script type="module" src="app.js"></script>
<script nomodule src="app-legacy.js"></script>
```

## Usage

- **When to use**: All modern JavaScript applications should use module scripts for code organization and dependency management
- **Real-world example**: React apps use `<script type="module" src="main.jsx"></script>` as entry point, Vue apps use module scripts for component imports
- **Best practices**:
  - Always use `type="module"` for modern applications
  - Use named exports for better tree shaking
  - Leverage dynamic imports for code splitting
  - Combine with bundlers (Webpack, Vite, Rollup) for production
  - Use `nomodule` attribute for older browser support
  - Keep module files focused and single-responsibility
  - Use relative paths for imports (`.js` extension recommended for clarity)
  - Avoid mixing module and non-module scripts when possible

## FAQ / Interview Questions

**Q: What is a module script and how is it different from a regular script?**
A: A module script uses `<script type="module">` to load JavaScript files as ES6 modules. Key differences:
- **Module script**: Creates isolated scope, uses import/export, automatically deferred, supports dynamic imports
- **Regular script**: Uses global scope, no native module syntax, blocks rendering
Module scripts are essential for modern, scalable applications while preventing global namespace pollution.

**Q: Why is scope isolation important in module scripts?**
A: Scope isolation prevents **global namespace pollution**, where variables from different scripts accidentally conflict. With regular scripts:
```javascript
// script1.js
var config = { ... };

// script2.js
var config = { ... };  // Overwrites script1's config!
```
Module scripts fix this:
```javascript
// module1.js
export const config = { ... };

// module2.js
export const config = { ... };  // No conflict, different scopes
```

**Q: Are module scripts automatically deferred?**
A: Yes, module scripts are **automatically deferred**, meaning they execute after HTML parsing completes. This is equivalent to adding the `defer` attribute to regular scripts. This ensures the DOM is ready before the script runs, improving performance and preventing rendering blocks.

**Q: How do dynamic imports work in module scripts?**
A: Dynamic imports use `import()` as a function (not a statement) to load modules at runtime:
```javascript
// Static import: loaded upfront
import { feature } from './feature.js';

// Dynamic import: loaded when needed (code splitting)
document.getElementById('btn').addEventListener('click', async () => {
  const module = await import('./heavy-feature.js');
  module.initialize();
});
```
This enables **lazy loading** and **code splitting** for better performance.

**Q: How do I support older browsers that don't understand module scripts?**
A: Use the `nomodule` attribute to provide a fallback:
```html
<!-- Modern browsers: loads module script -->
<script type="module" src="app.js"></script>

<!-- Older browsers: ignores module, loads this instead -->
<script nomodule src="app-legacy.js"></script>
```
The browser will execute one or the other, but not both. Use tools like Babel to transpile modern code to legacy versions for the `nomodule` version.

**Q: Can I use CommonJS require() in module scripts?**
A: No, module scripts only support **ES6 import/export syntax**. If you need CommonJS modules, use regular `<script>` tags or bundle your code first. However, when using a bundler (Webpack, Vite), you can write CommonJS and it gets transpiled to ES6 modules for the browser. Best practice is to use ES6 modules directly in module scripts.

## References
- [MDN Web Docs - JavaScript modules](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Modules)
- [MDN Web Docs - <script type="module">](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/script#module)
- [JavaScript.info - Modules, introduction](https://javascript.info/modules-intro)
- [Web.dev - Modules](https://web.dev/modules/)

---
*See also: [Async/Defer Script Loading](AsyncDeferScript.md), [Script Tag](ScriptTag.md), [Dynamic Imports](../JavaScript/DynamicImports.md)*
