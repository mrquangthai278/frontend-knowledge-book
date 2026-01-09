# React.memo

## Definition / Concept
**React.memo** is a higher-order component that memoizes a functional component, preventing unnecessary re-renders when props haven't changed. It performs a **shallow comparison** of props and only re-renders if props actually differ. This optimization is useful for expensive child components that receive the same props frequently.

- Wraps functional components to skip re-renders when props are equal
- Uses shallow comparison by default (comparing prop references)
- Can accept a custom comparison function for advanced control
- Helps prevent performance bottlenecks in deeply nested component trees

## Visual Representation
```
Before React.memo:
Parent re-renders → Child re-renders (even with same props)
Parent re-renders → Child re-renders
Parent re-renders → Child re-renders

After React.memo:
Parent re-renders → Child checks props → Props same? Skip render
Parent re-renders → Child checks props → Props same? Skip render
Parent re-renders → Child checks props → Props different? Re-render
```

## Example
```javascript
// Without React.memo - child re-renders on every parent update
const UserProfile = ({ name, age }) => {
  console.log('UserProfile rendered');
  return <div>{name} - {age}</div>;
};

// With React.memo - child only re-renders if props change
const MemoizedUserProfile = React.memo(({ name, age }) => {
  console.log('MemoizedUserProfile rendered');
  return <div>{name} - {age}</div>;
});

// Parent component
const App = () => {
  const [count, setCount] = useState(0);

  return (
    <div>
      <button onClick={() => setCount(count + 1)}>Increment: {count}</button>
      {/* Re-renders every time parent updates */}
      <UserProfile name="Alice" age={30} />

      {/* Only re-renders if name or age props change */}
      <MemoizedUserProfile name="Alice" age={30} />
    </div>
  );
};

// Custom comparison function - advanced control
const CustomMemoComponent = React.memo(
  ({ data, onUpdate }) => <div onClick={onUpdate}>{data.value}</div>,
  (prevProps, nextProps) => {
    // Return true if props are equal (DON'T re-render)
    // Return false if props differ (DO re-render)
    return prevProps.data.value === nextProps.data.value;
  }
);
```

## Usage
- **When to use**: Memoize expensive components that receive the same props frequently, child components in large lists, or components with heavy render logic
- **Real-world example**: A product card component in an e-commerce site that receives the same product ID repeatedly as parent state changes
- **Best practices**:
  - Only use when you've identified performance issues (profile first)
  - Avoid memoizing components with primitive props (they're always created fresh)
  - Be cautious with object/function props - they need memoization too (useMemo/useCallback) or they'll defeat React.memo
  - Don't over-optimize prematurely; React.memo has its own performance cost

## FAQ / Interview Questions

**Q: What does React.memo do?**
A: React.memo is a higher-order component that memoizes a functional component. It prevents unnecessary re-renders by comparing props before rendering. If props are the same as the previous render, the component skips re-rendering and returns the cached result.

**Q: How does React.memo compare props?**
A: By default, React.memo uses shallow comparison, meaning it checks if each prop reference is the same using `===`. Objects and functions are compared by reference, not by value. This is why you need useCallback and useMemo when passing such props.

**Q: What's the difference between React.memo and useMemo?**
A: React.memo memoizes entire components based on prop changes. useMemo memoizes the result of expensive computations. Use React.memo for component optimization, useMemo for expensive calculations within components.

**Q: Can I provide a custom comparison function?**
A: Yes, React.memo's second argument accepts a custom comparison function. Return `true` if props are equal (skip render), `false` if they differ (render). This gives you fine-grained control over when to skip re-renders.

**Q: When shouldn't I use React.memo?**
A: Avoid React.memo when components have simple render logic, primitive props that always change, or when props include objects/functions that aren't memoized. In these cases, the memoization overhead outweighs the benefit.

## References
- [React Documentation - React.memo](https://react.dev/reference/react/memo)
- [React Performance Optimization](https://react.dev/learn/render-and-commit)
- [When to useMemo and useCallback](https://kentcdodds.com/blog/usememo-and-usecallback)

---
*See also: [useMemo](./useMemo.md) | [useCallback](./useCallback.md) | [Performance](../Performance/)*
