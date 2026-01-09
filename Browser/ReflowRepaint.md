# Reflow / Repaint

## Definition / Concept
**Reflow** (also called **layout**) and **Repaint** (also called **redraw**) are browser rendering processes that occur when the DOM or styles change. **Reflow** recalculates the position and geometry of elements in the document, while **Repaint** updates the pixel representation on screen. Reflow is more expensive than repaint because it triggers layout calculations, and **reflow always triggers repaint**, but repaint can happen independently.

- **Reflow**: Recalculates element positions, dimensions, and layout (expensive)
- **Repaint**: Redraws pixels on screen without layout changes (less expensive)
- Excessive reflows/repaints cause **performance issues** and janky animations
- Understanding these processes is crucial for **frontend performance optimization**

## Visual Representation
```
DOM/Style Change
       ↓
   [Reflow?]
   YES ↓         NO ↓
Calculate      Skip to
Layout         Repaint
   ↓              ↓
[Repaint] ←──────┘
   ↓
Paint pixels
on screen

Reflow triggers:
- Geometry changes (width, height, position)
- DOM manipulation (add/remove elements)
- Content changes (text length)
- Window resize

Repaint only triggers:
- Color changes
- Visibility changes (opacity, visibility)
- Background changes
```

## Example
```javascript
// ❌ Causes REFLOW - changes layout/geometry
element.style.width = '100px';
element.style.height = '100px';
element.style.padding = '20px';
element.style.margin = '10px';
element.style.display = 'block';
element.style.position = 'absolute';

// ❌ Causes REPAINT only - visual changes
element.style.color = 'red';
element.style.backgroundColor = 'blue';
element.style.visibility = 'hidden';
element.style.outline = '1px solid red';

// ❌ BAD: Forces multiple reflows (layout thrashing)
for (let i = 0; i < 100; i++) {
  const width = element.offsetWidth; // Read - triggers reflow
  element.style.width = width + 10 + 'px'; // Write - invalidates layout
}

// ✅ GOOD: Batch reads and writes
const width = element.offsetWidth; // Read once
for (let i = 0; i < 100; i++) {
  element.style.width = (width + (i * 10)) + 'px'; // Write only
}

// ❌ BAD: Interleaved reads/writes
element.style.width = '100px';  // Write
const h1 = el1.offsetHeight;     // Read - forces reflow
element.style.height = '100px';  // Write
const h2 = el2.offsetHeight;     // Read - forces reflow

// ✅ GOOD: Batch all reads, then all writes
const h1 = el1.offsetHeight;     // Read
const h2 = el2.offsetHeight;     // Read
element.style.width = '100px';   // Write
element.style.height = '100px';  // Write

// Properties that trigger reflow when read:
const reflow1 = element.offsetWidth;
const reflow2 = element.offsetHeight;
const reflow3 = element.offsetTop;
const reflow4 = element.offsetLeft;
const reflow5 = element.clientWidth;
const reflow6 = element.clientHeight;
const reflow7 = element.scrollWidth;
const reflow8 = element.scrollHeight;
const reflow9 = element.getBoundingClientRect();
const reflow10 = window.getComputedStyle(element);

// ✅ Avoid reflows with CSS classes
// Bad: Inline styles cause multiple reflows
element.style.width = '200px';
element.style.height = '200px';
element.style.backgroundColor = 'red';

// Good: CSS class causes single reflow
element.classList.add('expanded');

// ✅ Use DocumentFragment for multiple DOM insertions
// Bad: Causes reflow for each appendChild
for (let i = 0; i < 100; i++) {
  const li = document.createElement('li');
  li.textContent = `Item ${i}`;
  list.appendChild(li); // Reflow on each iteration!
}

// Good: Single reflow with DocumentFragment
const fragment = document.createDocumentFragment();
for (let i = 0; i < 100; i++) {
  const li = document.createElement('li');
  li.textContent = `Item ${i}`;
  fragment.appendChild(li);
}
list.appendChild(fragment); // Single reflow

// ✅ Use position: absolute/fixed for animations
// Elements taken out of flow don't affect other elements
const animatedElement = document.querySelector('.animated');
animatedElement.style.position = 'absolute';
// Now animating this element won't cause reflow of siblings

// ✅ Use transform instead of top/left
// Bad: Causes reflow
element.style.left = '100px';
element.style.top = '100px';

// Good: Only causes repaint (GPU accelerated)
element.style.transform = 'translate(100px, 100px)';

// ✅ Minimize reflows with display: none
element.style.display = 'none'; // 1 reflow
// Make multiple changes
element.style.width = '100px';
element.style.height = '100px';
element.style.padding = '20px';
// No reflows while display: none
element.style.display = 'block'; // 1 reflow
```

## Usage
- **When to optimize**: 
  - Animations and transitions
  - Scrolling performance
  - Large DOM manipulations
  - Data tables with many rows
  - Infinite scroll implementations
  - Real-time updates (dashboards, charts)
  
- **Real-world example**:
  ```javascript
  // Virtual scrolling to minimize reflows
  class VirtualList {
    render(visibleItems) {
      // Only render visible items
      const fragment = document.createDocumentFragment();
      visibleItems.forEach(item => {
        fragment.appendChild(this.createItem(item));
      });
      
      // Single DOM update
      this.container.innerHTML = '';
      this.container.appendChild(fragment);
    }
  }
  
  // Debounce resize to avoid excessive reflows
  let resizeTimeout;
  window.addEventListener('resize', () => {
    clearTimeout(resizeTimeout);
    resizeTimeout = setTimeout(() => {
      updateLayout(); // Single reflow after resize stops
    }, 150);
  });
  
  // Use requestAnimationFrame for smooth animations
  function animate() {
    // Batch all reads
    const positions = elements.map(el => el.getBoundingClientRect());
    
    // Batch all writes
    elements.forEach((el, i) => {
      el.style.transform = `translateX(${positions[i].x + 10}px)`;
    });
    
    requestAnimationFrame(animate);
  }
  
  // Optimize table rendering
  function updateTable(data) {
    // Hide table during updates
    table.style.display = 'none';
    
    // Make all changes
    data.forEach(row => {
      const tr = createRow(row);
      tbody.appendChild(tr);
    });
    
    // Show table - single reflow
    table.style.display = 'table';
  }
  
  // Use CSS containment
  element.style.contain = 'layout style paint';
  // Tells browser this element's changes won't affect outside elements
  ```

- **Best practices**:
  - Batch DOM reads together, then batch writes together
  - Use CSS classes instead of inline styles for multiple changes
  - Use `transform` and `opacity` for animations (GPU accelerated)
  - Use `DocumentFragment` for multiple element insertions
  - Minimize layout thrashing (alternating read/write operations)
  - Use `requestAnimationFrame` for visual updates
  - Consider CSS `will-change` for frequently animated elements
  - Use `contain` property to limit reflow scope

## FAQ / Interview Questions

**Q: What's the difference between reflow and repaint?**
A: 
- **Reflow** (Layout): Recalculates element positions and dimensions. Triggered by geometry changes (width, height, position, DOM manipulation). Very expensive.
- **Repaint** (Redraw): Redraws pixels without layout changes. Triggered by visual changes (color, background, visibility). Less expensive.
- **Key difference**: Reflow changes layout → always causes repaint. Repaint can happen without reflow.

**Q: What causes reflow and repaint?**
A: 
**Reflow triggers**:
- Changing width, height, padding, margin, border
- Adding/removing DOM elements
- Changing font size or font family
- Window resize
- Reading layout properties (offsetWidth, scrollTop, getComputedStyle)

**Repaint triggers**:
- Changing color, background-color
- Changing visibility or opacity
- Changing outline or box-shadow
- Any property that affects appearance but not layout

**Q: What is layout thrashing and how do you prevent it?**
A: **Layout thrashing** (or forced synchronous layout) occurs when you repeatedly read layout properties and immediately write styles, forcing the browser to recalculate layout multiple times:
```javascript
// ❌ Layout thrashing
elements.forEach(el => {
  const height = el.offsetHeight; // Forces reflow
  el.style.height = height + 10 + 'px'; // Invalidates layout
  // Next iteration forces another reflow!
});

// ✅ Prevention: Batch reads, then writes
const heights = elements.map(el => el.offsetHeight); // All reads
elements.forEach((el, i) => {
  el.style.height = heights[i] + 10 + 'px'; // All writes
});
```

**Q: How can you optimize animations to avoid reflows?**
A: Best practices for performant animations:
1. **Use `transform` and `opacity`** - GPU accelerated, don't trigger reflow
2. **Avoid animating layout properties** (width, height, top, left)
3. **Use `will-change`** to hint browser about upcoming changes
4. **Use `position: absolute/fixed`** to take element out of document flow
5. **Use `requestAnimationFrame`** for smooth 60fps animations

```javascript
// ❌ Bad: Causes reflow
element.style.left = x + 'px';

// ✅ Good: Only repaint (GPU accelerated)
element.style.transform = `translateX(${x}px)`;
element.style.willChange = 'transform';
```

**Q: How do you measure and debug reflow/repaint issues?**
A: Tools and techniques:
1. **Chrome DevTools Performance tab**:
   - Record performance
   - Look for purple "Layout" and green "Paint" bars
   - Long/frequent bars indicate performance issues

2. **Rendering tab** (Chrome DevTools → More tools → Rendering):
   - Enable "Paint flashing" - shows repainted areas
   - Enable "Layout Shift Regions" - shows reflow areas

3. **Performance.mark() API**:
```javascript
performance.mark('start');
// Code that might cause reflow
element.style.width = '100px';
performance.mark('end');
performance.measure('reflow', 'start', 'end');
console.log(performance.getEntriesByName('reflow'));
```

4. **Watch for warnings**: Modern browsers log forced reflow warnings in console

## References
- [Google Web Fundamentals - Rendering Performance](https://developers.google.com/web/fundamentals/performance/rendering)
- [MDN Web Docs - Reflow](https://developer.mozilla.org/en-US/docs/Glossary/Reflow)
- [Paul Irish - What forces layout/reflow](https://gist.github.com/paulirish/5d52fb081b3570c81e3a)
- [CSS Triggers - What triggers layout, paint, composite](https://csstriggers.com/)

---
*See also: [Performance Optimization](../Performance/), [Cumulative Layout Shift](../Performance/CumulativeLayoutShift.md), [Browser Rendering](../Browser/)*
