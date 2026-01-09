# Currying

## Definition / Concept
**Currying** is a functional programming technique where a function with multiple arguments is transformed into a sequence of functions, each taking a single argument. Instead of calling `f(a, b, c)`, currying allows you to call it as `f(a)(b)(c)`. This enables **partial application** of functions, making code more reusable and composable.

- Transforms multi-argument functions into chained single-argument functions
- Enables **partial application** - creating specialized functions by pre-filling some arguments
- Improves code reusability and functional composition
- Named after mathematician Haskell Curry

## Visual Representation
```
Regular Function:
add(2, 3) → 5

Curried Function:
add(2) → (returns function)
add(2)(3) → 5

Currying Process:
f(a, b, c) → curry(f) → f(a) → f(b) → f(c)
```

## Example
```javascript
// Regular function
function add(a, b, c) {
  return a + b + c;
}
add(1, 2, 3); // 6

// Curried version
function curriedAdd(a) {
  return function(b) {
    return function(c) {
      return a + b + c;
    };
  };
}
curriedAdd(1)(2)(3); // 6

// Using arrow functions (cleaner syntax)
const curriedAddArrow = a => b => c => a + b + c;
curriedAddArrow(1)(2)(3); // 6

// Practical use: partial application
const add5 = curriedAddArrow(5);
const add5And10 = add5(10);
add5And10(3); // 18

// Generic curry function
function curry(fn) {
  return function curried(...args) {
    if (args.length >= fn.length) {
      return fn.apply(this, args);
    } else {
      return function(...nextArgs) {
        return curried.apply(this, args.concat(nextArgs));
      };
    }
  };
}

// Usage
const multiply = (a, b, c) => a * b * c;
const curriedMultiply = curry(multiply);
curriedMultiply(2)(3)(4); // 24
curriedMultiply(2, 3)(4); // 24
curriedMultiply(2)(3, 4); // 24
```

## Usage
- **When to use**: 
  - Creating specialized functions from generic ones
  - Building reusable utility functions
  - Functional composition and pipelines
  - Event handlers with pre-configured parameters
  
- **Real-world example**:
  ```javascript
  // Logging with different levels
  const log = level => message => timestamp =>
    console.log(`[${timestamp}] ${level}: ${message}`);
  
  const errorLog = log('ERROR');
  const infoLog = log('INFO');
  
  errorLog('Database connection failed')(new Date().toISOString());
  infoLog('User logged in')(new Date().toISOString());
  
  // Discount calculator
  const applyDiscount = discount => price => price * (1 - discount);
  const apply10Percent = applyDiscount(0.1);
  const apply20Percent = applyDiscount(0.2);
  
  apply10Percent(100); // 90
  apply20Percent(100); // 80
  ```

- **Best practices**:
  - Use currying for frequently reused function variations
  - Combine with composition for powerful functional patterns
  - Keep curried functions pure (no side effects)
  - Use arrow functions for cleaner syntax
  - Consider library implementations (lodash.curry, ramda.curry) for production

## FAQ / Interview Questions

**Q: What is currying and how does it differ from partial application?**
A: **Currying** transforms a function with multiple arguments into a sequence of single-argument functions. **Partial application** is applying some arguments to a function and returning a new function expecting the remaining arguments. Currying always produces unary functions (one argument at a time), while partial application can accept any number of arguments at each step.

**Q: What are the advantages of currying?**
A: Currying provides several benefits:
- **Reusability**: Create specialized functions from generic ones
- **Composition**: Easier to compose functions in functional pipelines
- **Readability**: More declarative and expressive code
- **Testing**: Smaller, focused functions are easier to test
- **Lazy evaluation**: Defer execution until all arguments are provided

**Q: Can you write a generic curry function?**
A: Yes, here's a practical implementation:
```javascript
function curry(fn) {
  return function curried(...args) {
    if (args.length >= fn.length) {
      return fn.apply(this, args);
    }
    return (...nextArgs) => curried(...args, ...nextArgs);
  };
}
```
This checks if enough arguments are provided (`fn.length`). If yes, executes the function; if not, returns a function waiting for more arguments.

**Q: What's a practical use case for currying in React or modern JavaScript?**
A: Common use cases include:
```javascript
// Event handlers with pre-configured data
const handleClick = id => event => {
  event.preventDefault();
  fetchData(id);
};

<button onClick={handleClick(userId)}>Click</button>

// API request builders
const apiRequest = method => endpoint => data =>
  fetch(endpoint, { method, body: JSON.stringify(data) });

const post = apiRequest('POST');
const postToUsers = post('/api/users');
postToUsers({ name: 'John' });
```

**Q: What are the downsides of currying?**
A: Potential drawbacks include:
- **Performance overhead**: Each curried function creates a closure, consuming memory
- **Debugging difficulty**: Stack traces can be harder to read with nested functions
- **Learning curve**: Requires understanding of functional programming concepts
- **Readability**: Overuse can make code harder to understand for those unfamiliar with the pattern
- **Not always necessary**: Simple functions may not benefit from currying

## References
- [MDN Web Docs - Closures](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Closures)
- [JavaScript.info - Currying](https://javascript.info/currying-partials)
- [Lodash - curry](https://lodash.com/docs/#curry)
- [Functional Programming in JavaScript](https://eloquentjavascript.net/)

---
*See also: [Closure](Closure.md), [Higher Order Functions](../JavaScript/), [Functional Programming](../JavaScript/)*
