# useMemo

## Definition / Concept
**useMemo** is a React hook that memoizes expensive computations and only recalculates them when dependencies change. It caches the result of a function and returns the same reference until dependencies update, preventing unnecessary recalculations. This is useful for expensive calculations, filtering large arrays, or creating stable object references needed by other hooks.

- Caches the result of expensive computations
- Only recalculates when dependencies change
- Returns the same object reference between renders (useful for derived objects)
- Helps optimize performance when combined with React.memo or other hooks

## Visual Representation
```
Without useMemo:
Render 1: Calculate expensive result → Return result
Render 2: Calculate expensive result again → Return result
Render 3: Calculate expensive result again → Return result

With useMemo:
Render 1: Calculate expensive result → Cache it → Return result
Render 2: Dependencies unchanged → Return cached result (skip calculation)
Render 3: Dependencies changed → Calculate new result → Cache it → Return result
```

## Example
```javascript
import { useMemo, useState } from 'react';

// Example 1: Memoizing expensive calculations
const FilteredList = ({ items, searchTerm }) => {
  // Without useMemo - filters on every render
  // const filtered = items.filter(item =>
  //   item.name.toLowerCase().includes(searchTerm.toLowerCase())
  // );

  // With useMemo - only filters when items or searchTerm change
  const filtered = useMemo(() => {
    console.log('Filtering items...');
    return items.filter(item =>
      item.name.toLowerCase().includes(searchTerm.toLowerCase())
    );
  }, [items, searchTerm]);

  return <ul>{filtered.map(item => <li key={item.id}>{item.name}</li>)}</ul>;
};

// Example 2: Memoizing object references
const UserProfile = ({ userId }) => {
  const [theme, setTheme] = useState('light');

  // Without useMemo - creates new object every render
  // const userConfig = { userId, preferences: { theme } };

  // With useMemo - same object reference when userId/theme unchanged
  const userConfig = useMemo(() => ({
    userId,
    preferences: { theme }
  }), [userId, theme]);

  // userConfig object reference stays same, so child won't re-render
  return <ChildComponent config={userConfig} />;
};

// Example 3: Computing derived state
const Dashboard = ({ data }) => {
  const stats = useMemo(() => {
    console.log('Computing statistics...');
    return {
      total: data.reduce((sum, item) => sum + item.value, 0),
      average: data.reduce((sum, item) => sum + item.value, 0) / data.length,
      max: Math.max(...data.map(item => item.value))
    };
  }, [data]);

  return <div>Total: {stats.total}, Avg: {stats.average}</div>;
};
```

## Usage
- **When to use**: Expensive calculations, filtering large datasets, creating stable object/array references needed by React.memo or dependency arrays, or computations that happen frequently
- **Real-world example**: A table component that filters, sorts, and paginates thousands of items based on user input
- **Best practices**:
  - Profile first - only use useMemo when you've identified a performance bottleneck
  - Dependencies must be accurate - missing or incorrect dependencies cause bugs
  - Use for actually expensive operations (simple calculations don't need memoization)
  - Don't over-memoize - storing values in memory has its own cost
  - Include all values used inside useMemo in the dependency array

## FAQ / Interview Questions

**Q: What does useMemo do?**
A: useMemo memoizes the result of an expensive computation. It runs the function only when dependencies change, otherwise returns the cached result. This prevents unnecessary recalculations on every render.

**Q: When should I use useMemo?**
A: Use useMemo when you have expensive calculations that don't need to run every render. Common cases include filtering/sorting large arrays, complex transformations, or creating stable object references for optimization purposes.

**Q: What's the difference between useMemo and useCallback?**
A: useMemo memoizes the return value of a function. useCallback memoizes the function itself. Use useMemo for computations, useCallback for function references (callbacks, event handlers).

**Q: What happens if I don't include a dependency in the array?**
A: If a dependency is missing, useMemo won't recalculate when it changes, returning stale data. This causes bugs where the component uses outdated values. Always include every value used inside useMemo in the dependency array.

**Q: Is useMemo always beneficial?**
A: No. useMemo has overhead for storing and comparing dependencies. For simple, fast calculations, useMemo might actually slow things down. Only use it for genuinely expensive operations.

## References
- [React Documentation - useMemo](https://react.dev/reference/react/useMemo)
- [When to useMemo](https://kentcdodds.com/blog/usememo-and-usecallback)
- [React Performance Optimization](https://react.dev/learn/render-and-commit)

---
*See also: [useCallback](./useCallback.md) | [React.memo](./Memo.md) | [Hooks](./Hooks.md)*
