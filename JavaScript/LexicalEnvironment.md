# Lexical Environment

## Definition / Concept

A **lexical environment** is an abstract specification used by JavaScript engines to manage how variables are stored and accessed within a program. It represents the environment in which code is executing and includes all the **local variables**, **parent references**, and the **this** binding. Every function and block creates its own lexical environment, which is determined by the code's **syntactic structure** (where it's written in the code), not where it's called. This is the mechanism that enables closures and the scope chain to function in JavaScript.

- **Lexical environments are created for each scope** - functions, blocks, and global scope each have their own environment
- **They define variable scope and accessibility** - variables are resolved by walking up the scope chain through parent lexical environments
- **They are static and determined at write-time** - the parent environment is determined by where the code is written, not where it's called from

## Visual Representation

```
┌────────────────────────────────────────┐
│  Global Lexical Environment            │
│  { globalVar, functionA, functionB }   │
│                                        │
│  ┌────────────────────────────────────┐│
│  │ Function A's Lexical Environment   ││
│  │ { paramA, localVarA }              ││
│  │ Parent: Global Lexical Environment ││
│  │                                    ││
│  │  ┌──────────────────────────────┐ ││
│  │  │ Inner Function's Lex. Env.   │ ││
│  │  │ { paramInner }               │ ││
│  │  │ Parent: Function A's Lex. Env││ ││
│  │  └──────────────────────────────┘ ││
│  └────────────────────────────────────┘│
│                                        │
│  ┌────────────────────────────────────┐│
│  │ Function B's Lexical Environment   ││
│  │ { paramB, localVarB }              ││
│  │ Parent: Global Lexical Environment ││
│  └────────────────────────────────────┘│
└────────────────────────────────────────┘

Variables are resolved by walking up the chain
until the variable is found or global scope is reached
```

## Example

```javascript
// Global lexical environment
let globalVar = 'global';

function outer(x) {
  // outer's lexical environment: { x, inner, localVar }
  let localVar = 'local';

  function inner(y) {
    // inner's lexical environment: { y }
    // Can access: y, x, localVar, globalVar (via scope chain)
    console.log(y, x, localVar, globalVar);
  }

  inner(100); // 100, 5, 'local', 'global'
  return inner;
}

const returnedInner = outer(5);
returnedInner(200); // 200, 5, 'local', 'global'
// inner still has access to outer's lexical environment (closure)
```

```javascript
// Block lexical environment (let/const)
function demonstrateBlocks() {
  let funcVar = 'function scope';

  if (true) {
    // Block creates its own lexical environment
    let blockVar = 'block scope';
    const constVar = 'const block scope';
    console.log(funcVar, blockVar); // Accessible
  }

  // blockVar and constVar are NOT accessible here
  console.log(funcVar); // Still accessible
}
```

## Usage

- **When to use**: Understanding how variable scoping works, debugging closure-related issues, predicting variable accessibility, understanding why certain variables are available in certain contexts
- **Real-world example**: Debugging why a variable is undefined in a callback; understanding why an event handler can access variables from its enclosing function
- **Best practices**: Think about lexical environments when writing nested functions; understand that scope is determined by code structure, not execution order; use let/const to create block-level lexical environments instead of relying on function scope

## FAQ / Interview Questions

**Q: What is a lexical environment and how does it relate to scope?**
A: A lexical environment is JavaScript's internal mechanism for managing variable storage and access. It directly implements **scope** in the language. Each function, block, and the global scope has its own lexical environment containing their local variables. The scope chain is formed by linking lexical environments through parent references.

**Q: How does lexical environment enable closures?**
A: When a function is created, it captures a reference to its **lexical environment** (the environment in which it was defined). Even after the outer function returns, the inner function maintains this reference, keeping variables from the outer scope accessible. This is why closures can "remember" variables from their creation context.

**Q: What's the difference between lexical environment and execution context?**
A: A **lexical environment** is a static, code-structure-based concept determined at write-time—it defines which variables are accessible based on where code is written. An **execution context** is created at runtime when code executes and includes the lexical environment plus additional information like `this` binding and outer scope references.

**Q: How do let and const create their own lexical environments?**
A: Both `let` and `const` are **block-scoped**, meaning each block (if, for, while, etc.) creates its own lexical environment. This is different from `var`, which is function-scoped. When the block executes, a new lexical environment is created as a child of the enclosing environment:
```javascript
function example() {
  let x = 1; // function lexical environment

  if (true) {
    let x = 2; // new block lexical environment shadows outer x
    console.log(x); // 2
  }

  console.log(x); // 1 (outer x is still accessible)
}
```

**Q: Can you explain how variable resolution works in lexical environments?**
A: When code references a variable, JavaScript searches for it in the **current lexical environment** first. If not found, it looks in the **parent lexical environment**, then the parent's parent, and so on, until it reaches the global lexical environment. This chain is called the **scope chain**. If the variable isn't found anywhere, a ReferenceError is thrown.

## References
- [MDN Web Docs - Lexical scoping](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Closures#lexical_scoping)
- [ECMAScript Specification - Lexical Environments](https://tc39.es/ecma262/#sec-lexical-environments)
- [JavaScript.info - Variable Scope](https://javascript.info/closure)
- [You Don't Know JS - Scope & Closures](https://github.com/getify/You-Dont-Know-JS/tree/2nd-ed/scope-closures)

---
*See also: [Scope Chain](./ScopeChain.md), [Closure](./Closure.md), [Hoisting](./Hoisting.md), [Variables](./Variables.md)*
