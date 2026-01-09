# Scope Chain

## Definition / Concept
The **scope chain** is the mechanism JavaScript uses to look up variables in **nested scopes**. When a variable is referenced, JavaScript searches the current scope first, then progressively moves outward through parent scopes until it finds the variable or reaches the global scope. Each function creates a new scope that has access to all variables in its parent scopes, forming a "chain" of scope lookups. Understanding scope chain is fundamental to understanding **closures**, **hoisting**, and **variable shadowing**.

- JavaScript searches from inner to outer scope
- Each function creates a new scope level
- Variables in outer scopes are accessible to inner functions
- The global scope is the final level in the chain

## Visual Representation

```
Global Scope
├── variable: x = 1
│
└── Function outer()
    ├── variable: y = 2
    │
    └── Function inner()
        ├── variable: z = 3
        │
        └── console.log(z)  ← Scope Chain: z → inner scope → outer scope → global

Scope Chain Lookup:
inner() searches: z (found in inner scope) ✓
inner() searches: y (not in inner, checks outer) ✓
inner() searches: x (not in inner/outer, checks global) ✓
inner() searches: unknown (not found anywhere) → ReferenceError
```

## Example

```javascript
// Global scope
const globalVar = 'global';

function outer() {
  // Outer function scope
  const outerVar = 'outer';

  function inner() {
    // Inner function scope
    const innerVar = 'inner';

    // Scope chain access: inner → outer → global
    console.log(innerVar);    // 'inner' (found in inner scope)
    console.log(outerVar);    // 'outer' (found in outer scope)
    console.log(globalVar);   // 'global' (found in global scope)
  }

  inner();
}

outer();

// ============================================
// Scope chain with variable shadowing
// ============================================

const x = 'global x';

function shadowExample() {
  const x = 'function x';  // Shadows global x

  function nestedFn() {
    const x = 'nested x';  // Shadows outer x

    console.log(x);        // 'nested x' (stops at inner scope)
  }

  nestedFn();
  console.log(x);          // 'function x' (outer scope)
}

shadowExample();
console.log(x);            // 'global x' (global scope)

// ============================================
// Scope chain with closures
// ============================================

function createCounter() {
  let count = 0;  // Captured in scope chain

  return function() {
    count++;
    return count;
  };
}

const counter = createCounter();
console.log(counter());  // 1 (closure accesses count from outer scope)
console.log(counter());  // 2 (count persists across calls)
console.log(counter());  // 3
```

## Usage

- **When to use**: Understanding variable access patterns, debugging variable conflicts, understanding closures
- **Real-world example**: Event handlers and callbacks that need access to surrounding function variables
- **Best practices**:
  - Minimize variable shadowing to avoid confusion
  - Use block scope (`let`, `const`) to create explicit scope boundaries
  - Be aware of closure memory implications when capturing variables
  - Avoid creating unnecessary nested functions
  - Use meaningful variable names to prevent conflicts
  - Understand that each function call creates a new scope instance

## FAQ / Interview Questions

**Q: What is the scope chain and how does it work?**
A: The scope chain is JavaScript's mechanism for resolving variable references. When a variable is accessed, JavaScript searches for it in the current scope. If not found, it looks in the parent scope, then the parent's parent, and so on until reaching the global scope. If the variable isn't found anywhere, a ReferenceError is thrown. This creates a chain of scopes from inner to outer.

**Q: What's the difference between scope chain and lexical scoping?**
A: **Lexical scoping** (or static scoping) determines which scope a variable belongs to based on where it's written in the code. **Scope chain** is the runtime mechanism that traverses these scopes to find variables. Lexical scoping defines the scope hierarchy, while scope chain implements the lookup process. JavaScript uses lexical scoping, so a function's accessible variables are determined by its position in the source code, not where it's called from.

**Q: Can inner functions access variables from outer functions?**
A: Yes, inner functions have access to all variables in outer scopes through the scope chain. This is called a **closure**:
```javascript
function outer(x) {
  return function inner() {
    return x * 2;  // inner accesses x from outer scope
  };
}
const fn = outer(5);
console.log(fn());  // 10
```
The inner function maintains a reference to its outer scope even after outer() returns.

**Q: What happens with variable shadowing in the scope chain?**
A: When a variable is declared in an inner scope with the same name as an outer scope variable, the inner variable **shadows** (hides) the outer one. The scope chain stops at the first matching variable name:
```javascript
const x = 'global';
function fn() {
  const x = 'local';  // Shadows global x
  console.log(x);     // 'local'
}
```
The inner `x` is found first and the lookup stops, so global `x` is inaccessible. This can lead to bugs if not intentional.

**Q: How does the scope chain relate to memory and garbage collection?**
A: Variables in the scope chain are kept in memory as long as any function referencing them exists. In closures, captured variables can't be garbage collected even after the outer function returns. This can consume memory if not managed carefully:
```javascript
function createLargeObject() {
  const largeArray = new Array(1000000);
  return function() {
    return largeArray.length;  // Holds reference, prevents GC
  };
}
```
Understanding scope chain helps you manage memory efficiently and avoid unintended memory leaks.

## References
- [MDN Web Docs - Scope](https://developer.mozilla.org/en-US/docs/Glossary/Scope)
- [MDN Web Docs - Closures](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Closures)
- [JavaScript.info - Variable Scope, Closure](https://javascript.info/closure)
- [Eloquent JavaScript - Scoping](https://eloquentjavascript.net/03_functions.html#h_lnOqqDKMKc)

---
*See also: [Closure](Closure.md), [Scope](Scope.md), [Hoisting](Hoisting.md), [This](This.md)*
