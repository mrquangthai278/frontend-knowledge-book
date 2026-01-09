# Higher-Order Functions

## Definition / Concept
**Higher-order functions** are functions that either take one or more functions as arguments, return a function as a result, or both. They are a fundamental concept in **functional programming** and enable powerful abstractions like `map`, `filter`, `reduce`, and function composition. Higher-order functions treat functions as **first-class citizens**, allowing them to be passed around like any other value.

- Accept functions as parameters (callbacks)
- Return functions as output
- Enable functional composition and reusable abstractions
- Built-in array methods like `map`, `filter`, `reduce` are higher-order functions

## Visual Representation
```
Regular Function:
add(2, 3) → 5

Higher-Order Function (accepts function):
processArray([1, 2, 3], double) → [2, 4, 6]
                        ↑
                   function parameter

Higher-Order Function (returns function):
createMultiplier(5) → function(x) { return x * 5 }
                      ↓
                   returned function
```

## Example
```javascript
// Higher-order function that accepts a function
function repeat(n, action) {
  for (let i = 0; i < n; i++) {
    action(i);
  }
}

repeat(3, console.log); // 0, 1, 2

// Higher-order function that returns a function
function greaterThan(n) {
  return function(m) {
    return m > n;
  };
}

const greaterThan10 = greaterThan(10);
greaterThan10(11); // true
greaterThan10(5);  // false

// Built-in higher-order functions
const numbers = [1, 2, 3, 4, 5];

// map - transforms each element
const doubled = numbers.map(n => n * 2);
// [2, 4, 6, 8, 10]

// filter - selects elements matching condition
const evens = numbers.filter(n => n % 2 === 0);
// [2, 4]

// reduce - accumulates values
const sum = numbers.reduce((acc, n) => acc + n, 0);
// 15

// Combining higher-order functions
const result = numbers
  .filter(n => n > 2)
  .map(n => n * 2)
  .reduce((acc, n) => acc + n, 0);
// (3 + 4 + 5) * 2 = 24

// Custom higher-order function
function withLogging(fn) {
  return function(...args) {
    console.log(`Calling with args:`, args);
    const result = fn(...args);
    console.log(`Result:`, result);
    return result;
  };
}

const add = (a, b) => a + b;
const addWithLogging = withLogging(add);
addWithLogging(2, 3);
// Calling with args: [2, 3]
// Result: 5
// Returns: 5
```

## Usage
- **When to use**: 
  - Array transformations (map, filter, reduce)
  - Function composition and pipelines
  - Creating reusable abstractions
  - Implementing middleware patterns
  - Event handlers and callbacks
  - Decorating functions with additional behavior
  
- **Real-world example**:
  ```javascript
  // API middleware pattern
  function withAuth(apiCall) {
    return async function(...args) {
      const token = getAuthToken();
      if (!token) throw new Error('Not authenticated');
      return apiCall(...args, token);
    };
  }
  
  const fetchUserData = withAuth(async (userId, token) => {
    // API call implementation with token
  });
  
  // Debounce - delays function execution
  function debounce(fn, delay) {
    let timeoutId;
    return function(...args) {
      clearTimeout(timeoutId);
      timeoutId = setTimeout(() => fn(...args), delay);
    };
  }
  
  const handleSearch = debounce((query) => {
    console.log('Searching for:', query);
  }, 300);
  
  // Data processing pipeline
  const users = [
    { name: 'John', age: 25, active: true },
    { name: 'Jane', age: 30, active: false },
    { name: 'Bob', age: 35, active: true }
  ];
  
  const activeUserNames = users
    .filter(user => user.active)
    .map(user => user.name)
    .sort();
  // ['Bob', 'John']
  ```

- **Best practices**:
  - Use built-in array methods (map, filter, reduce) instead of loops
  - Keep higher-order functions pure when possible
  - Combine multiple operations in a pipeline for readability
  - Use descriptive names for callback functions
  - Avoid deep nesting - extract callbacks to named functions

## FAQ / Interview Questions

**Q: What is a higher-order function?**
A: A **higher-order function** is a function that either:
1. Takes one or more functions as arguments, OR
2. Returns a function as its result
Common examples include `map`, `filter`, `reduce`, `setTimeout`, and event listeners. They enable functional programming patterns and code reusability by treating functions as first-class values.

**Q: What's the difference between map, filter, and reduce?**
A: 
- **map**: Transforms each element and returns a new array of the same length
  ```javascript
  [1, 2, 3].map(n => n * 2) // [2, 4, 6]
  ```
- **filter**: Selects elements matching a condition, returns subset
  ```javascript
  [1, 2, 3].filter(n => n > 1) // [2, 3]
  ```
- **reduce**: Accumulates values into a single result
  ```javascript
  [1, 2, 3].reduce((sum, n) => sum + n, 0) // 6
  ```

**Q: Can you explain how reduce works with an example?**
A: `reduce` takes a callback function and an initial value. The callback receives an accumulator and current value, returning the updated accumulator:
```javascript
const numbers = [1, 2, 3, 4];
const sum = numbers.reduce((accumulator, current) => {
  return accumulator + current;
}, 0); // 0 is initial value
// Iteration 1: acc=0, current=1 → returns 1
// Iteration 2: acc=1, current=2 → returns 3
// Iteration 3: acc=3, current=3 → returns 6
// Iteration 4: acc=6, current=4 → returns 10
```

**Q: What are the advantages of using higher-order functions?**
A: Key advantages include:
- **Code reusability**: Extract common patterns into reusable functions
- **Abstraction**: Hide implementation details, focus on what not how
- **Composition**: Chain operations to build complex logic from simple parts
- **Declarative code**: Express intent clearly (what to do vs how to do it)
- **Immutability**: Array methods return new arrays, preserving originals
- **Testability**: Small, focused functions are easier to test

**Q: How would you implement your own map function?**
A: Here's a simple implementation:
```javascript
function myMap(array, callback) {
  const result = [];
  for (let i = 0; i < array.length; i++) {
    result.push(callback(array[i], i, array));
  }
  return result;
}

// Usage
myMap([1, 2, 3], n => n * 2); // [2, 4, 6]
```
This iterates through the array, applies the callback to each element with its index and the original array, and collects results in a new array.

## References
- [MDN Web Docs - Higher-order functions](https://developer.mozilla.org/en-US/docs/Glossary/First-class_Function)
- [MDN Web Docs - Array methods](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array)
- [Eloquent JavaScript - Higher-Order Functions](https://eloquentjavascript.net/05_higher_order.html)
- [JavaScript.info - Array methods](https://javascript.info/array-methods)

---
*See also: [Closure](Closure.md), [Currying](Currying.md), [Functional Programming](../JavaScript/)*
