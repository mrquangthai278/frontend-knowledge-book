# Closure

## Definition / Concept

A **closure** is a function that has access to variables from its outer scope, even after the outer function has returned. In JavaScript, closures are created every time a function is created. They combine a function with the variables it needs from its surrounding scope, enabling **data privacy** and **encapsulation**. Closures are fundamental to understanding JavaScript's scoping mechanism and are essential for callbacks, higher-order functions, and asynchronous programming.

- **Functions can access variables from their outer scope** - even after the outer function completes execution
- **Closures enable data privacy** - variables can be protected from external access
- **Every function creates a closure** - they automatically capture their surrounding context

## Visual Representation

```
┌─────────────────────────────────────┐
│  Outer Function (Scope)             │
│  ┌─────────────────────────────────┐│
│  │  Variable: count = 0            ││
│  │  ┌──────────────────────────────┐│
│  │  │  Inner Function (Closure)    ││
│  │  │  - Has access to count       ││
│  │  │  - Remembers count value     ││
│  │  └──────────────────────────────┘│
│  └─────────────────────────────────┘│
└─────────────────────────────────────┘

When Inner Function returns, it still has
access to count variable through closure
```

## Example

```javascript
// Simple closure example
function createCounter() {
  let count = 0; // Private variable captured by closure

  return function() {
    count++;
    return count;
  };
}

const counter = createCounter();
console.log(counter()); // 1
console.log(counter()); // 2
console.log(counter()); // 3
```

```javascript
// Closure with multiple functions
function setupButtons(buttonIds) {
  buttonIds.forEach(id => {
    const btn = document.getElementById(id);
    btn.addEventListener('click', function() {
      console.log(`Button ${id} clicked`); // 'id' is captured in closure
    });
  });
}

setupButtons(['btn1', 'btn2', 'btn3']);
```

## Usage

- **When to use**: Creating private variables, managing state in callbacks, implementing module patterns, and handling asynchronous operations
- **Real-world example**: Event listeners that need to remember context (element ID, user data, etc.); factory functions that create multiple instances with independent state
- **Best practices**: Be aware of memory implications when closures capture large objects; use closures intentionally for encapsulation rather than relying on global variables

## FAQ / Interview Questions

**Q: What is a closure and why is it important?**
A: A closure is a function that has access to variables from its outer scope even after the outer function has returned. It's important because:
- It enables **data privacy and encapsulation** - variables can be hidden from external access
- It's the foundation for **callbacks and event handlers** - handlers can remember the context they were created in
- It's crucial for **async code** - understanding closures helps work with promises and async/await

**Q: Can you give a practical example of a closure?**
A: Event handlers commonly use closures. When you attach an event listener inside a loop, the closure captures the loop variable:
```javascript
for (let i = 0; i < 3; i++) {
  setTimeout(() => console.log(i), 1000); // Closure captures 'i'
}
```
Each callback closes over its own value of `i` and logs it correctly (0, 1, 2).

**Q: What's the difference between var and let in closure contexts?**
A: With `var`, all iterations share the same variable, so all callbacks see the final value. With `let`, each iteration gets its own block scope:
```javascript
// var: all closures share same i, prints 3 three times
for (var i = 0; i < 3; i++) {
  setTimeout(() => console.log(i), 0); // 3, 3, 3
}

// let: each closure captures different i, prints 0, 1, 2
for (let i = 0; i < 3; i++) {
  setTimeout(() => console.log(i), 0); // 0, 1, 2
}
```

**Q: How does closure relate to memory management?**
A: Closures hold references to outer variables, keeping them in memory. If a closure captures a large object and isn't garbage collected, the object remains in memory. This can cause memory leaks if closures are created in loops or event handlers without proper cleanup. Always remove event listeners when they're no longer needed.

**Q: Can you explain a module pattern using closures?**
A: The module pattern uses closures to create private variables and expose only public methods:
```javascript
const calculator = (function() {
  let result = 0; // Private variable

  return {
    add(x) { result += x; return result; },
    subtract(x) { result -= x; return result; },
    getResult() { return result; }
  };
})();

calculator.add(5); // Can't directly access 'result', must use methods
```

## References
- [MDN Web Docs - Closures](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Closures)
- [JavaScript.info - Variable Scope, Closure](https://javascript.info/closure)
- [You Don't Know JS - Scope & Closures](https://github.com/getify/You-Dont-Know-JS/tree/2nd-ed/scope-closures)

---
*See also: [Scope Chain](./ScopeChain.md), [Hoisting](./Hoisting.md), [This](./This.md)*
