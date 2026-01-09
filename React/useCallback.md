# useCallback

## Definition / Concept
**useCallback** is a React hook that memoizes a function reference, returning the same function instance between renders unless dependencies change. Unlike regular functions that are recreated on every render, useCallback keeps the same function reference when dependencies haven't updated. This is essential for optimizing components that rely on function reference equality, such as event handlers passed to memoized children.

- Returns the same function reference when dependencies are unchanged
- Prevents child components from unnecessary re-renders when using React.memo
- Essential for dependency arrays in useEffect
- Commonly paired with React.memo to prevent parent re-renders from triggering child renders

## Visual Representation
```
Without useCallback:
Render 1: Create handleClick function A
Render 2: Create handleClick function B (different reference!)
Render 3: Create handleClick function C (different reference!)
→ Child component re-renders every time, even with React.memo

With useCallback:
Render 1: Create handleClick function A → cache it
Render 2: Return same function A (dependencies unchanged)
Render 3: Return same function A (dependencies unchanged)
Render 4: Create new handleClick function B (dependency changed)
→ Child component only re-renders when function actually changes
```

## Example
```javascript
import { useCallback, useState } from 'react';

// Example 1: Memoize handler for child component
const Parent = () => {
  const [count, setCount] = useState(0);

  const handleClick = useCallback(() => {
    setCount(count + 1);
  }, [count]); // Same function reference unless count changes

  return <MemoButton onClick={handleClick} />;
};

// Example 2: useCallback with multiple dependencies
const SearchComponent = ({ onSearch }) => {
  const [query, setQuery] = useState('');

  const handleSearch = useCallback(() => {
    onSearch(query);
  }, [query, onSearch]); // Include all dependencies

  return (
    <div>
      <input value={query} onChange={e => setQuery(e.target.value)} />
      <button onClick={handleSearch}>Search</button>
    </div>
  );
};

// Example 3: useCallback with React.memo
const MemoButton = React.memo(({ onClick }) => (
  <button onClick={onClick}>Click me</button>
));

const Counter = () => {
  const [count, setCount] = useState(0);
  const increment = useCallback(() => setCount(count + 1), [count]);

  return <MemoButton onClick={increment} />; // Only re-renders if increment reference changes
};
```

## Usage
- **When to use**: Passing callbacks to memoized child components, including callbacks in useEffect dependency arrays, optimizing performance in lists or frequently re-rendering parents
- **Real-world example**: A search input that passes a memoized callback to a memoized list component, so the list doesn't re-filter on every keystroke
- **Best practices**:
  - Always include all dependencies in the dependency array (use ESLint rule to catch this)
  - Only memoize callbacks that are passed to memoized components or useEffect
  - Don't memoize every callback - only those that impact performance
  - useCallback is a micro-optimization; profile first to confirm it helps
  - Pair useCallback with React.memo for maximum benefit

## FAQ / Interview Questions

**Q: What does useCallback do?**
A: useCallback returns the same function reference between renders unless dependencies change. This prevents unnecessary function recreations and is useful for passing callbacks to optimized child components.

**Q: Why is function reference important in React?**
A: React.memo and dependency arrays compare references. If a function is recreated on every render, it's a different reference, causing memoized components to re-render or effects to re-run unnecessarily, defeating optimization.

**Q: What's the difference between useCallback and useMemo?**
A: useCallback memoizes the function itself - `useCallback(fn, deps)` is equivalent to `useMemo(() => fn, deps)`. useMemo memoizes the return value of a function. Use useCallback for functions, useMemo for values.

**Q: When should I use useCallback?**
A: Use useCallback when passing callbacks to React.memo components or including callbacks in useEffect dependency arrays. For callbacks only used locally, useCallback usually isn't necessary.

**Q: What happens if I miss a dependency?**
A: If you miss a dependency, the function won't update when it should, causing stale closures. For example, a counter callback might use an old count value. Always include all values from the outer scope in the dependency array.

## References
- [React Documentation - useCallback](https://react.dev/reference/react/useCallback)
- [useCallback vs useMemo](https://kentcdodds.com/blog/usememo-and-usecallback)
- [React Hooks Rules](https://react.dev/reference/rules/rules-of-hooks)

---
*See also: [useMemo](./useMemo.md) | [React.memo](./Memo.md) | [Hooks](./Hooks.md)*
