# Pure Function

## Definition / Concept
A **pure function** is a function that always returns the same output for the same input and has no **side effects**. It depends only on its input parameters and doesn't modify external state, making it predictable, testable, and ideal for **functional programming**. Pure functions are the foundation of reliable, maintainable code.

- Always returns the same result for the same arguments (**deterministic**)
- No **side effects** - doesn't modify external variables, DOM, or perform I/O
- Doesn't depend on external state or mutable data
- Results are predictable and easy to test

## Visual Representation
```
Pure Function:
Input → [Pure Function] → Output
  2, 3      add(a, b)        5
  2, 3      add(a, b)        5  (always same result)

Impure Function:
Input → [Impure Function] → Output + Side Effects
  2, 3      addAndLog        5 + console.log (side effect)
                            ↓
                         External State Modified
```

## Example
```javascript
// ✅ Pure function - deterministic, no side effects
function add(a, b) {
  return a + b;
}
add(2, 3); // Always returns 5
add(2, 3); // Always returns 5

// ✅ Pure function - array operations
function double(numbers) {
  return numbers.map(n => n * 2);
}
double([1, 2, 3]); // [2, 4, 6]

// ❌ Impure - modifies external state
let total = 0;
function addToTotal(value) {
  total += value; // Side effect!
  return total;
}

// ❌ Impure - depends on external state
function addToTotal2(value) {
  return total + value; // Depends on external variable
}

// ❌ Impure - produces side effects
function addAndLog(a, b) {
  console.log('Adding numbers'); // Side effect!
  return a + b;
}

// ❌ Impure - non-deterministic
function random() {
  return Math.random(); // Different output each time
}

// ❌ Impure - mutates input
function addItem(array, item) {
  array.push(item); // Mutates original array!
  return array;
}

// ✅ Pure version - returns new array
function addItemPure(array, item) {
  return [...array, item]; // No mutation
}

// ✅ Pure function - complex calculation
function calculateDiscount(price, discountPercent) {
  return price * (1 - discountPercent / 100);
}
calculateDiscount(100, 10); // Always 90
```

## Usage
- **When to use**: 
  - Data transformations and calculations
  - Utility functions and helpers
  - Redux reducers and state management
  - Unit testing scenarios
  - Concurrent/parallel operations
  - Memoization and caching
  
- **Real-world example**:
  ```javascript
  // Pure reducer in Redux
  function cartReducer(state = [], action) {
    switch (action.type) {
      case 'ADD_ITEM':
        return [...state, action.item]; // Returns new state
      case 'REMOVE_ITEM':
        return state.filter(item => item.id !== action.id);
      default:
        return state;
    }
  }
  
  // Pure data transformation
  function formatUsers(users) {
    return users.map(user => ({
      id: user.id,
      fullName: `${user.firstName} ${user.lastName}`,
      email: user.email.toLowerCase()
    }));
  }
  
  // Pure validation
  function isValidEmail(email) {
    const regex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
    return regex.test(email);
  }
  
  // Impure to Pure conversion
  // ❌ Impure - modifies DOM
  function updatePrice(price) {
    document.getElementById('price').textContent = price;
  }
  
  // ✅ Pure - returns formatted value
  function formatPrice(price) {
    return `$${price.toFixed(2)}`;
  }
  // Use in component: element.textContent = formatPrice(price)
  ```

- **Best practices**:
  - Prefer pure functions for business logic and calculations
  - Isolate side effects to specific parts of your application
  - Use pure functions in reducers (Redux, useReducer)
  - Return new objects/arrays instead of mutating
  - Make pure functions the default, impure when necessary
  - Document when functions must be impure

## FAQ / Interview Questions

**Q: What is a pure function and why is it important?**
A: A **pure function** is a function that:
1. Always returns the same output for the same input (deterministic)
2. Has no side effects (doesn't modify external state)

It's important because pure functions are:
- **Predictable**: Easy to reason about and debug
- **Testable**: No need to mock external dependencies
- **Cacheable**: Results can be memoized for performance
- **Parallelizable**: Safe to run concurrently
- **Composable**: Easy to combine with other functions

**Q: What are side effects in programming?**
A: **Side effects** are any observable changes outside the function's scope:
- Modifying global variables or external state
- Mutating input parameters
- Making HTTP requests or API calls
- Writing to console, files, or database
- Modifying the DOM
- Getting current time or random numbers
Side effects make functions impure and harder to test and predict.

**Q: How do you convert an impure function to a pure one?**
A: Example conversion:
```javascript
// ❌ Impure - mutates input and uses external state
let tax = 0.1;
function calculateTotal(cart) {
  cart.total = cart.items.reduce((sum, item) => sum + item.price, 0);
  cart.total += cart.total * tax;
  return cart;
}

// ✅ Pure - no mutation, tax passed as parameter
function calculateTotalPure(cart, taxRate) {
  const subtotal = cart.items.reduce((sum, item) => sum + item.price, 0);
  const total = subtotal + (subtotal * taxRate);
  return { ...cart, total };
}
```
Key changes: pass dependencies as parameters, return new objects, don't mutate inputs.

**Q: Can you have side effects in real applications?**
A: Yes, side effects are necessary for useful applications (API calls, DOM updates, logging). The strategy is to:
- **Isolate** side effects in specific layers (e.g., API service, event handlers)
- Keep **business logic pure**
- Use patterns like Redux where reducers are pure but effects are handled separately
- In React, use pure components and isolate side effects in `useEffect`
```javascript
// Pure business logic
const calculatePrice = (items) => items.reduce((sum, i) => sum + i.price, 0);

// Side effects isolated in effect
useEffect(() => {
  const price = calculatePrice(items); // Pure
  updatePriceDisplay(price); // Impure (DOM manipulation)
}, [items]);
```

**Q: What's the difference between a pure function and a deterministic function?**
A: All **pure functions** are deterministic, but not all **deterministic functions** are pure:
- **Deterministic**: Always returns same output for same input
- **Pure**: Deterministic AND no side effects

Example:
```javascript
// Deterministic but impure (has side effect)
function addAndLog(a, b) {
  console.log('Adding'); // Side effect!
  return a + b; // Deterministic result
}

// Pure (deterministic + no side effects)
function add(a, b) {
  return a + b;
}
```

## References
- [MDN Web Docs - Functions](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Functions)
- [Functional Programming Principles](https://eloquentjavascript.net/)
- [Redux - Pure Functions](https://redux.js.org/understanding/thinking-in-redux/three-principles)
- [Eric Elliott - Master the JavaScript Interview](https://medium.com/javascript-scene/master-the-javascript-interview-what-is-a-pure-function-d1c076bec976)

---
*See also: [Closure](Closure.md), [Higher-Order Functions](HigherOrderFunctions.md), [Immutability](../JavaScript/)*
