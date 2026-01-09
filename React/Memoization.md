# Memoization in React

## Definition / Concept
**Memoization** in React is an optimization technique that caches computation results or component renders to avoid redundant work when inputs haven't changed. React provides three primary tools for memoization: **React.memo** (component-level), **useMemo** (computation-level), and **useCallback** (function reference-level). Together, they prevent unnecessary re-renders and recalculations, improving performance in applications with expensive operations or deeply nested component trees.

- Caches results to avoid redundant computations
- Prevents unnecessary re-renders using reference equality
- Consists of React.memo, useMemo, and useCallback working together
- Essential for optimizing performance-critical applications
- Requires careful dependency management to avoid bugs

## Visual Representation
```
React Memoization Strategy:

1. PARENT COMPONENT
   └─ Re-renders (state change)

2. CALLBACK OPTIMIZATION (useCallback)
   └─ handleClick = useCallback(() => {...}, [deps])
   └─ Function reference stays same if deps unchanged

3. DERIVED VALUE OPTIMIZATION (useMemo)
   └─ derivedValue = useMemo(() => {...}, [deps])
   └─ Computation result cached if deps unchanged

4. CHILD COMPONENT OPTIMIZATION (React.memo)
   └─ const Child = React.memo(({ callback, value }) => {...})
   └─ Child skips re-render if callback and value refs unchanged

RESULT: Parent updates → Optimized callbacks/values → Child doesn't re-render
```

## Example
```javascript
import { useState, useCallback, useMemo } from 'react';

// Memoized child component
const ProductCard = React.memo(({ product, onAddToCart }) => {
  console.log(`Rendering: ${product.name}`);
  return (
    <div>
      <h3>{product.name}</h3>
      <p>${product.price}</p>
      <button onClick={() => onAddToCart(product.id)}>Add to Cart</button>
    </div>
  );
});

// Parent component using all three memoization techniques
const ProductList = ({ products }) => {
  const [cart, setCart] = useState([]);
  const [sortBy, setSortBy] = useState('name');

  // useCallback: Memoize the handler so child doesn't re-render
  const handleAddToCart = useCallback((productId) => {
    setCart([...cart, productId]);
  }, [cart]); // Function recreated only when cart changes

  // useMemo: Memoize the sorted list
  const sortedProducts = useMemo(() => {
    console.log('Sorting products...');
    const sorted = [...products].sort((a, b) => {
      if (sortBy === 'name') return a.name.localeCompare(b.name);
      if (sortBy === 'price') return a.price - b.price;
      return 0;
    });
    return sorted;
  }, [products, sortBy]); // Sorting only happens when products or sortBy change

  return (
    <div>
      <select value={sortBy} onChange={(e) => setSortBy(e.target.value)}>
        <option value="name">Sort by Name</option>
        <option value="price">Sort by Price</option>
      </select>

      {sortedProducts.map(product => (
        <ProductCard
          key={product.id}
          product={product}
          onAddToCart={handleAddToCart}
        />
      ))}

      <p>Cart: {cart.length} items</p>
    </div>
  );
};

// Usage
const App = () => {
  const products = [
    { id: 1, name: 'Laptop', price: 999 },
    { id: 2, name: 'Mouse', price: 29 },
    { id: 3, name: 'Keyboard', price: 79 }
  ];

  return <ProductList products={products} />;
};
```

## Usage
- **When to use**: Optimizing performance bottlenecks, preventing unnecessary re-renders in large component trees, expensive computations, or stable object/function references needed downstream
- **Real-world example**: An e-commerce site with thousands of products - use memoization to prevent re-filtering/sorting and child component re-renders on every interaction
- **Best practices**:
  - Profile first - identify actual bottlenecks before memoizing
  - Use all three tools together for maximum benefit (React.memo + useCallback + useMemo)
  - Dependencies must be accurate and complete
  - Avoid memoizing simple, fast operations
  - Consider the memoization overhead - sometimes it costs more than the benefit
  - Use ESLint rules to catch dependency array mistakes

## FAQ / Interview Questions

**Q: What are the three main memoization tools in React?**
A: React.memo (memoizes components), useMemo (memoizes computation results), and useCallback (memoizes function references). They work together - useCallback and useMemo create stable values/references that React.memo uses to skip re-renders.

**Q: When should I use memoization?**
A: Use memoization when you've identified a performance bottleneck through profiling. Common cases include expensive computations, large lists, or deeply nested components with callbacks. Don't memoize prematurely without evidence of a problem.

**Q: How do React.memo, useMemo, and useCallback work together?**
A: Parent uses useCallback to keep handler reference stable and useMemo to keep data stable. These stable references are passed to React.memo children. React.memo compares these props - if references haven't changed, the child skips re-rendering.

**Q: What happens with incorrect dependencies?**
A: Incorrect dependencies cause stale closures or missed optimizations. Missing a dependency means values don't update when they should (bugs). Extra dependencies cause unnecessary recalculations. Dependencies must match all values used inside the hook.

**Q: Is memoization always good?**
A: No. Memoization has overhead - storing values, comparing dependencies, and managing memory. For simple operations or components that rarely re-render, memoization might hurt performance. Profile first to confirm it helps.

## References
- [React Performance Optimization](https://react.dev/learn/render-and-commit)
- [React.memo Documentation](https://react.dev/reference/react/memo)
- [useMemo and useCallback](https://kentcdodds.com/blog/usememo-and-usecallback)
- [Before You memo](https://react.dev/reference/react/memo#skipping-re-rendering-with-memo)

---
*See also: [React.memo](./Memo.md) | [useMemo](./useMemo.md) | [useCallback](./useCallback.md) | [Hooks](./Hooks.md)*
