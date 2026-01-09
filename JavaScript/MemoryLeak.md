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

```javascript
// ❌ Bad: Memory Leak - Event listener never removed
const button = document.getElementById('myButton');

function handleClick() {
  console.log('Button clicked');
}

button.addEventListener('click', handleClick);
// If button is later removed from DOM, the listener still holds a reference
// Memory is not freed even though the button is gone

// ✅ Good: Properly remove event listeners
button.removeEventListener('click', handleClick);
```

```javascript
// ❌ Bad: Forgotten timer
let counter = 0;

function startCounter() {
  setInterval(() => {
    counter++;
    console.log(counter);
  }, 1000); // This interval runs forever, never cleared
}

startCounter();

// ✅ Good: Store and clear the timer
let timerId;

function startCounter() {
  timerId = setInterval(() => {
    counter++;
    console.log(counter);
  }, 1000);
}

function stopCounter() {
  clearInterval(timerId); // Properly cleanup
}
```

```javascript
// ❌ Bad: Detached DOM nodes still referenced
const element = document.getElementById('node');
const cache = [element]; // Cache holds reference

element.remove(); // Removed from DOM but still in memory via cache

// ✅ Good: Clear references when removing DOM nodes
element.remove();
cache.length = 0; // Or cache = null; to release reference
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
