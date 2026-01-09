# Memory Heap

## Definition / Concept

The **heap** is a region of memory in JavaScript where **objects and arrays** are stored dynamically at runtime. Unlike the **call stack** which stores primitive values and function calls, the heap allows for flexible memory allocation of complex data structures. When you create an object or array, JavaScript allocates space in the heap and stores a **reference** to that memory location on the stack. The **garbage collector** automatically frees heap memory when objects are no longer referenced, making memory management largely automatic for developers.

- **Heap stores objects, arrays, and complex data structures** - any non-primitive values
- **References to heap memory are stored on the stack** - the stack holds pointers, not the actual data
- **Garbage collection manages heap memory** - unreferenced objects are automatically cleaned up
- **Heap is slower than stack** - accessing heap memory involves pointer dereferencing

## Visual Representation

```
┌─────────────────────────────────────┐
│         CALL STACK                  │
│                                     │
│  primitiveNum: 42                   │
│  objRef: 0x1A2B ──────────┐         │
│  arrRef: 0x3C4D ────────┐ │         │
└─────────────────────────┼─┼─────────┘
                          │ │
        ┌─────────────────┘ │
        │                   │
┌───────▼───────────────────▼─────────┐
│           HEAP MEMORY               │
│                                     │
│  0x1A2B: {                          │
│    name: "John",                    │
│    age: 30                          │
│  }                                  │
│                                     │
│  0x3C4D: [1, 2, 3, 4]              │
└─────────────────────────────────────┘
```

## Example

```javascript
// Primitives stored on stack, objects stored on heap
const num = 42; // Stack: num = 42
const str = "hello"; // Stack: str points to string in heap

const user = { name: "Alice", age: 25 }; // Stack: user reference, Heap: object data
const arr = [1, 2, 3]; // Stack: arr reference, Heap: array data

// Reference copy - both point to same heap memory
const user2 = user;
user2.age = 26;
console.log(user.age); // 26 - both reference same object
```

```javascript
// Memory allocation and garbage collection
function createUsers() {
  const users = [];
  for (let i = 0; i < 1000; i++) {
    users.push({
      id: i,
      name: `User ${i}`,
      email: `user${i}@example.com`
    }); // Each object allocated in heap
  }
  return users;
}

const allUsers = createUsers(); // Heap stores 1000 objects
// allUsers still references the array, heap memory kept alive
```

## Usage

- **When to use**: Understanding how JavaScript manages memory helps you write more efficient code; avoiding memory leaks; understanding reference vs value copying
- **Real-world example**: When you create objects in loops or event handlers, each object gets heap memory. If you don't remove event listeners, the associated closures keep heap memory alive, causing memory leaks
- **Best practices**: Be aware of what gets stored where; remove event listeners and timers when no longer needed; avoid creating unnecessary object references; use developer tools to profile heap memory usage

## FAQ / Interview Questions

**Q: What is the heap and how does it differ from the stack?**
A: The heap is memory used for dynamic allocation of objects and arrays, while the stack stores primitives and function calls. Stack is faster and automatically cleared when functions return. Heap requires garbage collection to free unused memory. Objects and arrays go to the heap; primitives go to the stack (though object references themselves are on the stack).

**Q: Why do objects require heap memory while primitives use the stack?**
A: Primitives have fixed, known sizes (numbers, booleans, strings) so they can be stored directly on the stack. Objects and arrays have variable sizes determined at runtime, so they need flexible heap allocation. The stack stores a reference (pointer) to the heap location.

**Q: What happens when you assign an object to another variable?**
A: You're copying the reference, not the object itself. Both variables point to the same heap memory:
```javascript
const obj1 = { count: 0 };
const obj2 = obj1; // obj2 points to same heap location
obj2.count = 5;
console.log(obj1.count); // 5 - same object
```
To copy the object data itself, use `const obj2 = { ...obj1 }` or `Object.assign()`.

**Q: How does garbage collection work in JavaScript?**
A: The garbage collector automatically identifies objects that are no longer reachable from the root (global scope, active functions). When an object has no references, it's marked for garbage collection and its heap memory is freed. Most modern engines use "mark and sweep" algorithm. This is why removing event listeners matters - they create references that prevent garbage collection.

**Q: Can you have memory leaks in JavaScript?**
A: Yes, despite automatic garbage collection. Memory leaks occur when objects remain referenced but are no longer needed. Common causes:
- Event listeners that aren't removed
- Timers (setInterval) never cleared
- Closures capturing large objects
- Global variables accumulating data
```javascript
// Memory leak: listener never removed
element.addEventListener('click', () => {
  largeObject.doSomething(); // Listener keeps largeObject in memory
});

// Better: remove listener when done
const handler = () => largeObject.doSomething();
element.addEventListener('click', handler);
element.removeEventListener('click', handler);
```

## References
- [MDN Web Docs - Memory Management](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Memory_Management)
- [JavaScript.info - Garbage Collection](https://javascript.info/garbage-collection)
- [Chrome DevTools - Memory Issues](https://developer.chrome.com/docs/devtools/memory-issues/)

---
*See also: [Memory Leak](./MemoryLeak.md), [Closure](./Closure.md), [Scope Chain](./ScopeChain.md)*
