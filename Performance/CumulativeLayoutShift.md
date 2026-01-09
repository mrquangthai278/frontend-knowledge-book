# Cumulative Layout Shift

## Definition / Concept
**Cumulative Layout Shift (CLS)** measures the visual stability of a webpage by tracking unexpected layout shifts during the loading and interaction phases. It quantifies how much visible content moves without user input, affecting user experience and search engine ranking. CLS is one of Google's **Core Web Vitals** metrics, with a good score being 0.1 or less.

- Key point 1: Measures unexpected movement of page elements
- Key point 2: Critical metric for user experience and SEO
- Key point 3: Caused by fonts, images, ads, and dynamically injected content

## Visual Representation
```
Before Load:        During Load:           After Stabilization:
┌──────────────┐   ┌──────────────┐      ┌──────────────┐
│   Header     │   │   Header     │      │   Header     │
├──────────────┤   ├──────────────┤      ├──────────────┤
│              │   │  Ad (SHIFT!) │      │  Ad (stable) │
│  Content     │   │              │      │              │
│              │   │  Content ↓   │      │  Content     │
└──────────────┘   └──────────────┘      └──────────────┘

CLS = sum of all layout shift fractions
Good: < 0.1  |  Needs Work: 0.1 - 0.25  |  Poor: > 0.25
```

## Example
```javascript
// Monitoring CLS using the PerformanceObserver API
let clsValue = 0;

const observer = new PerformanceObserver((list) => {
  for (const entry of list.getEntries()) {
    // Only count shifts that don't have hadRecentInput
    if (!entry.hadRecentInput) {
      clsValue += entry.value;
      console.log('CLS updated:', clsValue);
    }
  }
});

// Start observing layout shifts
observer.observe({ type: 'layout-shift', buffered: true });

// Report CLS value when user leaves the page
window.addEventListener('beforeunload', () => {
  console.log('Final CLS:', clsValue.toFixed(3));
});
```

## Usage
- **When to use**: Monitor all websites to maintain good Core Web Vitals
- **Real-world example**: An e-commerce site loading ads dynamically causes content shift, hurting CLS score and search rankings
- **Best practices**:
  - Reserve space for images, ads, and embeds using aspect-ratio or size containers
  - Use `font-display: swap` for web fonts
  - Load third-party scripts asynchronously
  - Avoid inserting content above existing content
  - Use transforms for animations instead of layout properties

## Common Causes of Layout Shift

1. **Images without dimensions** - Add width/height attributes or aspect-ratio
2. **Ads and embeds** - Reserve container space in advance
3. **Web fonts** - Use `font-display: swap` to prevent invisible text
4. **Dynamically injected content** - Above-the-fold content shifts everything down
5. **Animations using layout properties** - Use `transform` instead of margin/width

## FAQ / Interview Questions

**Q: What is Cumulative Layout Shift and why does it matter?**
A: CLS measures the sum of individual layout shift scores for unexpected shifts during page load and interaction. It matters because:
- It directly impacts user experience (frustrating to click wrong elements)
- It's a Core Web Vital that affects Google search rankings
- Poor CLS indicates a fragile, unoptimized website
- It's one of three metrics in the Core Web Vitals assessment

**Q: How is CLS calculated?**
A: CLS is calculated as: Impact Fraction × Distance Fraction. Impact Fraction is the percentage of viewport affected by the shift, Distance Fraction is the distance moved as a percentage of viewport. For example, if an element takes up 50% of the viewport and shifts down by 20% of viewport height, that shift scores 0.5 × 0.2 = 0.1.

**Q: What's the difference between expected and unexpected layout shifts?**
A: Expected shifts (caused by user input like scrolling or clicking) are not counted in CLS. The `hadRecentInput` flag filters these out. Only unexpected shifts (ads loading, content injecting above) are counted, which is why monitoring code checks this property.

**Q: How do I fix a high CLS score?**
A: The most common fixes are:
- Add explicit dimensions to images (width/height or aspect-ratio)
- Reserve space for dynamic content with placeholder containers
- Use `font-display: swap` for web fonts
- Load ads and third-party content asynchronously
- Avoid inserting content above the fold without user interaction

**Q: What tools can I use to measure CLS?**
A: Several tools measure CLS:
- **Chrome DevTools** - Lighthouse audits show CLS
- **Web Vitals library** - npm package for programmatic measurement
- **PageSpeed Insights** - Google's tool with field data
- **SearchConsole** - Shows CLS data from real users
- **PerformanceObserver API** - JavaScript native API for monitoring

## References
- [Web Vitals - Cumulative Layout Shift](https://web.dev/articles/cls)
- [MDN - Layout Instability API](https://developer.mozilla.org/en-US/docs/Web/API/Layout_Instability_API)
- [Google's Core Web Vitals Guide](https://developers.google.com/search/docs/appearance/core-web-vitals)
- [Web.dev - Optimize Cumulative Layout Shift](https://web.dev/articles/optimize-cls)

---

*See also: [WebVitals](../Performance/WebVitals.md), [Performance Optimization](../Performance/PerformanceOptimization.md)*
