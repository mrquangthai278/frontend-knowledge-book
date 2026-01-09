# Memory Leak

## Definition / Concept

A **Memory Leak** occurs when a program retains references to objects that are no longer needed, preventing the **garbage collector** from freeing that memory. In JavaScript, this commonly happens when event listeners are not removed, circular references are created, or detached DOM nodes are still referenced. Memory leaks accumulate over time, causing the application to consume more memory and eventually become slow or crash.

- Memory leaks occur when objects are **no longer needed but still referenced**
- The garbage collector cannot free memory for objects with active references
- In long-running applications (SPAs, Node.js servers), memory leaks compound and degrade performance
- **Common causes**: unremoved event listeners, forgotten timers, circular references, detached DOM nodes

## Visual Representation

```
┌─────────────────────────────────────┐
│   Memory Allocation Over Time        │
│                                       │
│  ▓▓▓▓                                │
│  ▓▓▓▓▓▓▓                             │
│  ▓▓▓▓▓▓▓▓▓▓                          │  ← Memory grows (leak)
│  ▓▓▓▓▓▓▓▓▓▓▓▓▓                       │  ← Not released
│  ▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓                    │
│  ▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓                  │
│  ▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓                │  ← Eventually crashes
│  ▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓              │
│  ──────────────────────────────────── │
│  Time →                               │
└─────────────────────────────────────┘

Without leak (healthy):
│  ▓▓                                   │
│  ▓▓  ▓▓                               │
│  ▓▓  ▓▓  ▓▓                           │
│  ────────────── (stable, GC works)    │
```

## Example

### Unremoved Event Listeners

```javascript
// ❌ Bad: Listener never removed
const button = document.getElementById('btn');
button.addEventListener('click', () => {
  console.log('Clicked');
});
// Button removed from DOM but listener still holds reference

// ✅ Good: Remove listener when done
const handler = () => console.log('Clicked');
button.addEventListener('click', handler);
button.removeEventListener('click', handler);  // Cleanup
```

### Forgotten Timers

```javascript
// ❌ Bad: Timer runs forever
setInterval(() => {
  console.log('tick');
}, 1000);  // Never cleared

// ✅ Good: Store and clear timer
const timerId = setInterval(() => {
  console.log('tick');
}, 1000);

clearInterval(timerId);  // Cleanup when done
```

### Detached DOM Nodes

```javascript
// ❌ Bad: DOM node still referenced after removal
const element = document.getElementById('node');
const cache = [element];
element.remove();  // Still in memory via cache

// ✅ Good: Clear references
element.remove();
cache.length = 0;  // Or cache = null
```

## Usage

- **When to use**: Identifying and fixing memory leaks is critical for maintaining performant long-running applications
- **Real-world example**: In a single-page application (SPA), navigating between pages without cleaning up event listeners causes memory to grow indefinitely
- **Best practices**:
  - **Always remove event listeners** when components are destroyed
  - **Clear timers** with `clearInterval()` and `clearTimeout()`
  - **Use weak references** (`WeakMap`, `WeakSet`) for caches that shouldn't prevent garbage collection
  - **Monitor memory usage** with DevTools to detect leaks early
  - **Clean up in lifecycle methods** (React: `useEffect` cleanup, Vue: `beforeUnmount`)
  - **Avoid circular references** where objects reference each other

## FAQ / Interview Questions

**Q: What is a memory leak and why is it a problem?**
A: A memory leak occurs when objects are no longer needed but remain in memory because they're still referenced. This is problematic because:
- Memory usage grows over time, eventually exhausting available RAM
- Applications become slow as garbage collection becomes more expensive
- Long-running applications (servers, SPAs) may crash
- User experience degrades with slower performance

**Q: What are the most common causes of memory leaks in JavaScript?**
A: The most common causes are:
1. **Unremoved event listeners** — Not calling `removeEventListener()`
2. **Forgotten timers** — `setInterval()` or `setTimeout()` that are never cleared
3. **Detached DOM nodes** — DOM elements removed from the document but still referenced
4. **Global variables** — Accidentally creating global references that accumulate
5. **Circular references** — Objects referencing each other (though modern GC handles this)
6. **Cached data** — Arrays or objects that keep growing without bounds

**Q: How do you detect memory leaks in a JavaScript application?**
A: Use browser DevTools:
1. Open **Chrome DevTools** → **Memory** tab
2. Take a **heap snapshot** before and after an action
3. Trigger the action multiple times (e.g., navigate pages multiple times)
4. Compare snapshots to see if memory grows
5. Look for objects that **increase in count** but should have been garbage collected
6. Use the **Allocation timeline** to track memory growth over time

**Q: How would you fix a memory leak caused by event listeners?**
A: Store the handler function and remove it when done:
```javascript
const handler = () => { /* ... */ };
element.addEventListener('click', handler);
// Later:
element.removeEventListener('click', handler);
```
Or use `once: true` for one-time listeners:
```javascript
element.addEventListener('click', handler, { once: true });
```

**Q: What are WeakMap and WeakSet, and how do they help prevent memory leaks?**
A: `WeakMap` and `WeakSet` hold weak references to objects, allowing them to be garbage collected even if they're in the collection. They're useful for caches:
```javascript
// ❌ Strong reference - prevents GC
const cache = new Map();
const obj = { id: 1 };
cache.set('key', obj);
obj = null; // obj still in memory via cache

// ✅ Weak reference - allows GC
const weakCache = new WeakMap();
weakCache.set(obj, 'data');
obj = null; // obj can now be garbage collected
```

## References

- [MDN Web Docs - Memory Management](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Memory_Management)
- [Chrome DevTools - Find Memory Problems](https://developer.chrome.com/docs/devtools/memory-problems/)
- [JavaScript.info - Garbage Collection](https://javascript.info/garbage-collection)
- [WeakMap and WeakSet - MDN](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/WeakMap)

---

*See also: [Garbage Collection](./GarbageCollection.md), [Event Loop](./EventLoop.md), [Closures](./Closure.md), [This](./This.md)*
