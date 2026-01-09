# Event Loop

## Definition / Concept

The **Event Loop** is JavaScript's mechanism for handling asynchronous operations and keeping the program responsive. It continuously checks the **call stack** for code to execute, and when the stack is empty, it pulls callbacks from the **callback queue** (also called the task queue). This allows JavaScript, despite being single-threaded, to handle multiple asynchronous operations like timers, network requests, and DOM events without blocking.

- JavaScript is **single-threaded** — only one line of code executes at a time
- The Event Loop manages the order of code execution between synchronous and asynchronous tasks
- It prevents the browser from freezing by allowing long operations to be deferred

## Visual Representation

```
┌─────────────────────────────────────────┐
│         Call Stack (Execution)          │
│  - Synchronous code runs here           │
│  - Current function being executed      │
└─────────────────────────────────────────┘
         ▲                    │
         │                    ▼
         │          ┌──────────────────┐
         │          │ Event Loop       │
         │          │ (Continuously    │
         │          │  checks & polls) │
         │          └──────────────────┘
         │                    │
         └────────────────────┼───────────┐
                              ▼           ▼
                    ┌──────────────┐  ┌──────────┐
                    │ Callback Q.  │  │  Micr..  │
                    │ (setTimeout) │  │  (Promise)
                    └──────────────┘  └──────────┘
```

## Example

```javascript
// Example: Understanding the Event Loop order

console.log('1. Start');

setTimeout(() => {
  console.log('2. setTimeout callback (Callback Queue)');
}, 0);

Promise.resolve()
  .then(() => {
    console.log('3. Promise callback (Microtask Queue)');
  });

console.log('4. End');

// Output:
// 1. Start
// 4. End
// 3. Promise callback (Microtask Queue)
// 2. setTimeout callback (Callback Queue)
```

**Why this order?**
1. Synchronous code (1, 4) executes first
2. **Microtasks** (Promises, `queueMicrotask()`) execute next
3. **Macrotasks** (setTimeout, setInterval) execute last

## Usage

- **When to use**: Understanding the Event Loop helps you write performant code and debug timing issues
- **Real-world example**: Fetching data from an API, handling button clicks, or deferring heavy computations without freezing the UI
- **Best practices**:
  - Use `async/await` and **Promises** instead of callbacks for cleaner async code
  - Understand **microtasks** vs **macrotasks** to predict execution order
  - Avoid blocking the main thread with long synchronous operations
  - Use `requestAnimationFrame()` for animations to sync with browser rendering

## FAQ / Interview Questions

**Q: What is the Event Loop and why does JavaScript need it?**
A: The Event Loop is JavaScript's mechanism for handling asynchronous code in a single-threaded environment. Without it, the browser would freeze while waiting for network requests, timers, or other async operations. The Event Loop allows these operations to be deferred while keeping the UI responsive.

**Q: What's the difference between the Callback Queue and Microtask Queue?**
A: The **Microtask Queue** (used by Promises and `queueMicrotask()`) has higher priority and executes before the **Callback Queue** (used by `setTimeout`, `setInterval`). All microtasks must complete before the next macrotask begins, ensuring Promises always execute before timers.

**Q: Can you explain the order of execution in this example?**
A:
```javascript
console.log('A');
setTimeout(() => console.log('B'), 0);
Promise.resolve().then(() => console.log('C'));
console.log('D');
// Output: A, D, C, B
```
Synchronous code (A, D) executes first, then microtasks (C), then macrotasks (B).

**Q: Why does `setTimeout(..., 0)` not execute immediately?**
A: Even with a 0 delay, `setTimeout` is a **macrotask** that goes into the Callback Queue. The Event Loop only processes macrotasks after the call stack is empty and all microtasks are complete. So `setTimeout(..., 0)` is deferred until after synchronous code and Promises finish.

**Q: How does `requestAnimationFrame()` fit into the Event Loop?**
A: `requestAnimationFrame()` is scheduled before the browser renders the next frame. It executes after microtasks but before the next macrotask, making it ideal for animations. It runs roughly 60 times per second (on a 60Hz monitor) and ensures smooth visual updates without layout thrashing.

## References

- [MDN Web Docs - The JavaScript Event Loop](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Event_loop)
- [JavaScript.info - Event Loop](https://javascript.info/event-loop)
- [Web APIs - setTimeout and the Event Loop](https://developer.mozilla.org/en-US/docs/Web/API/setTimeout)
- [MDN - Microtasks and Macrotasks](https://developer.mozilla.org/en-US/docs/Web/API/HTML_DOM_API/Microtask)

---

*See also: [Callbacks](./Callbacks.md), [Promises](./Promises.md), [Async/Await](./AsyncAwait.md), [Hoisting](./Hoisting.md)*
