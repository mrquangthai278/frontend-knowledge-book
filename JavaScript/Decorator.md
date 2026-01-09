# Decorator

## Definition / Concept

A **decorator** is a higher-order function that wraps another function or object to modify its behavior without changing its original code. Decorators enhance functionality by adding features like **logging**, **validation**, **caching**, or **access control**. They follow the **Decorator Design Pattern** and are fundamental to functional programming, enabling **separation of concerns** and **code reusability**.

- Wraps functions or objects to add new functionality
- Preserves the original function's interface and behavior
- Enables **composition** of multiple behaviors
- Core concept for **middleware**, **higher-order functions**, and **aspect-oriented programming**

## Visual Representation

```
Original Function:
┌─────────────────┐
│  calculateSum   │
└─────────────────┘

With Decorator (Logging):
┌──────────────────────────────────────┐
│  Logger Decorator                    │
│  ┌──────────────────────────────┐   │
│  │  calculateSum (original)     │   │
│  └──────────────────────────────┘   │
│  + Logs input/output                 │
└──────────────────────────────────────┘

Multiple Decorators (Composition):
┌─────────────────────────────────────────────┐
│  Auth Decorator                             │
│  ┌─────────────────────────────────────┐   │
│  │  Logger Decorator                   │   │
│  │  ┌──────────────────────────────┐   │   │
│  │  │  calculateSum (original)     │   │   │
│  │  └──────────────────────────────┘   │   │
│  └─────────────────────────────────────┘   │
└─────────────────────────────────────────────┘
```

## Example

```javascript
// Basic decorator - adding logging
function logDecorator(fn) {
  return function(...args) {
    console.log(`Calling ${fn.name} with:`, args);
    const result = fn(...args);
    console.log(`Result:`, result);
    return result;
  };
}

function multiply(a, b) {
  return a * b;
}

const loggedMultiply = logDecorator(multiply);
loggedMultiply(5, 3);
// Calling multiply with: [5, 3]
// Result: 15

// Decorator for caching/memoization
function memoizeDecorator(fn) {
  const cache = new Map();
  return function(...args) {
    const key = JSON.stringify(args);
    if (cache.has(key)) {
      console.log("Returning cached result");
      return cache.get(key);
    }
    const result = fn(...args);
    cache.set(key, result);
    return result;
  };
}

function expensiveCalculation(n) {
  console.log("Computing...");
  return n * n;
}

const memoized = memoizeDecorator(expensiveCalculation);
memoized(5);  // Computing... → 25
memoized(5);  // Returning cached result → 25

// Decorator for authentication/authorization
function requireAuthDecorator(fn) {
  return function(user, ...args) {
    if (!user || !user.isAuthenticated) {
      throw new Error("User must be authenticated");
    }
    return fn(user, ...args);
  };
}

function deleteUserData(user, userId) {
  console.log(`Deleting data for user ${userId}`);
  return true;
}

const secureDelete = requireAuthDecorator(deleteUserData);
secureDelete({ isAuthenticated: true }, 123);  // Works!
// secureDelete({ isAuthenticated: false }, 123);  // Error!

// Composing decorators
function compose(...decorators) {
  return (fn) => decorators.reduceRight((f, dec) => dec(f), fn);
}

const enhancedMultiply = compose(
  logDecorator,
  memoizeDecorator
)(multiply);

enhancedMultiply(5, 3);  // Logs input, computes, caches result
enhancedMultiply(5, 3);  // Logs input, returns cached result
```

## Usage

- **When to use**: Adding cross-cutting concerns like logging, validation, caching, authentication, or error handling without modifying original code
- **Real-world example**: Express middleware functions, Redux action enhancers, logger wrappers for API calls, permission checks for sensitive operations
- **Best practices**:
  - Keep decorators **focused on a single responsibility**
  - Preserve the original function's signature when possible
  - Use **composition** to combine multiple decorators
  - Document what the decorator does clearly
  - Consider performance impact of nested decorators
  - Return a function with the same name (when using `fn.name`) for debugging

## FAQ / Interview Questions

**Q: What's the difference between a decorator and middleware?**
A: **Middleware** and **decorators** are similar concepts. Middleware typically refers to functions that process requests/responses in a chain (like Express middleware), while decorators are a broader pattern for wrapping functions. In practice, middleware is often implemented using the decorator pattern. The main difference is context: middleware is framework-specific, decorators are a general pattern.

**Q: Can you chain/compose multiple decorators?**
A: Yes! You can apply decorators sequentially:
```javascript
const fn = multiplyDecorator(logDecorator(cacheDecorator(original)));
```
Or use a compose function:
```javascript
const fn = compose(multiplyDecorator, logDecorator, cacheDecorator)(original);
```
Order matters—decorators wrap from innermost to outermost.

**Q: How is this different from just modifying the function directly?**
A: Modifying the original function violates the **Open/Closed Principle** and makes code harder to maintain. Decorators keep the original function unchanged, making it **reusable** with different enhancements, and allow you to **combine multiple behaviors** without creating complex interdependencies.

**Q: What's the performance impact of using decorators?**
A: Each decorator adds a function call overhead. Multiple nested decorators can create a performance cost. For performance-critical code, consider: batching operations, memoizing results, or using decorators selectively. Profile your code to measure actual impact rather than assuming.

**Q: Is this the same as TypeScript's `@decorator` syntax?**
A: No, but related. TypeScript decorators (`@decorator`) are a syntactic sugar built on the decorator pattern. JavaScript's stage-3 decorator proposal (currently not widely supported) provides syntax support. The concept of wrapping functions to modify behavior is the same, but TypeScript decorators work with classes and methods, while this pattern applies to any function.

## References
- [MDN - Higher-Order Functions](https://developer.mozilla.org/en-US/docs/Glossary/Higher-order_function)
- [JavaScript Design Patterns - Decorator Pattern](https://www.patterns.dev/posts/decorator-pattern/)
- [Refactoring Guru - Decorator Pattern](https://refactoring.guru/design-patterns/decorator)
- [You Don't Know JS - Closures and Scope](https://github.com/getify/You-Dont-Know-JS)

---

*See also: [Higher-Order Functions](./HigherOrderFunctions.md), [Closures](./Closure.md), [Composition](./Composition.md), [Callbacks](./Callbacks.md)*
