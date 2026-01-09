# Async/Defer Script Loading

## Definition / Concept
**Async** and **defer** are HTML script attributes that control how and when JavaScript files are downloaded and executed. By default, scripts block HTML parsing, slowing down page load. The **async** attribute loads scripts in parallel without blocking parsing, executing immediately when ready. The **defer** attribute also loads in parallel but delays execution until after HTML parsing completes. Both optimize performance by decoupling script loading from page rendering.

- Prevent scripts from blocking HTML parsing
- Enable parallel script downloads
- Improve perceived page load time
- Critical for performance optimization

## Visual Representation

```
NORMAL (no async/defer):
|--HTML Parse--|--Script Download--|--Script Execute--|--HTML Parse--|
                ↓ (Blocking)
Load time: ~5s

ASYNC:
|--HTML Parse & Script Download (parallel)--|--Script Execute--|
                                           ↓ (When ready)
Load time: ~3s

DEFER:
|--HTML Parse (parallel with download)--|--HTML Parse Complete--|--Script Execute--|
                                       ↓ (After parsing)
Load time: ~4s
```

## Example

```html
<!-- Normal: Blocks HTML parsing -->
<script src="script.js"></script>

<!-- Async: Loads in parallel, executes immediately -->
<script async src="analytics.js"></script>

<!-- Defer: Loads in parallel, executes after parsing -->
<script defer src="app.js"></script>

<!-- Multiple defer scripts execute in order -->
<script defer src="library.js"></script>
<script defer src="app.js"></script>
```

## Usage

- **When to use async**: Third-party scripts that don't depend on your code (analytics, ads, tracking)
- **When to use defer**: Application code that depends on DOM being ready and execution order matters
- **Real-world example**:
  - Use `async` for Google Analytics or Facebook Pixel
  - Use `defer` for your main application bundle
  - Use normal (blocking) only for critical above-the-fold content
- **Best practices**:
  - Place scripts at the end of `<body>` to minimize blocking
  - Use `defer` for your main application code
  - Use `async` for independent third-party scripts
  - Combine with module bundlers for better performance
  - Avoid mixing async/defer dependencies (e.g., async tracking code that app.js depends on)

## FAQ / Interview Questions

**Q: What's the difference between async and defer attributes?**
A: Both `async` and `defer` load scripts in parallel without blocking HTML parsing. The key difference:
- **Async**: Executes immediately when the script finishes downloading (order not guaranteed)
- **Defer**: Waits until after HTML parsing completes, maintaining script order
Use async for independent scripts, defer for code that needs the DOM or depends on other scripts.

**Q: Why should I use defer instead of placing scripts at the end of the body?**
A: Both approaches work, but `defer` offers advantages:
1. Scripts start downloading earlier (in the `<head>` during initial parsing)
2. Clearer intent in your HTML (explicit about script timing)
3. Guaranteed execution order with multiple deferred scripts
4. Better for build tools and asset management

**Q: Can I use async for scripts that depend on other scripts?**
A: No, you should not. Async scripts execute when ready, not in order. If `script-b.js` depends on `script-a.js`, use `defer` instead:
```html
<script defer src="script-a.js"></script>
<script defer src="script-b.js"></script>
<!-- Guaranteed: script-a.js executes first -->
```

**Q: What happens if a deferred script references DOM elements?**
A: Deferred scripts execute after HTML parsing, so the DOM is fully constructed. This is safe:
```html
<body>
  <div id="app"></div>
  <script defer src="app.js"></script>
  <!-- app.js can safely access #app -->
</body>
```

**Q: How do async/defer affect performance metrics like LCP and FID?**
A: Using `defer` and `async` appropriately improves **Largest Contentful Paint (LCP)** and **First Input Delay (FID)** because:
- Non-blocking scripts allow faster HTML parsing and rendering
- Parsing completes sooner, allowing interaction
- Less JavaScript blocking the main thread
Place critical rendering logic in the main bundle and third-party scripts as `async` to achieve better Core Web Vitals.

## References
- [MDN Web Docs - <script>: async attribute](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/script#attr-async)
- [MDN Web Docs - <script>: defer attribute](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/script#attr-defer)
- [Web.dev - Third-party scripts](https://web.dev/third-party-javascript/)
- [HTML Living Standard - Script element](https://html.spec.whatwg.org/multipage/scripting.html#the-script-element)

---
*See also: [Script Tag](ScriptTag.md), [Performance Optimization](../Performance/WebVitals.md), [Lazy Loading](../Performance/LazyLoading.md)*
