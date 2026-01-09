# Temporal Dead Zone (TDZ)

## Definition / Concept

The **Temporal Dead Zone (TDZ)** is the region of code where a variable declared with `let` or `const` is hoisted but not yet initialized. During this period, accessing the variable throws a `ReferenceError`. The TDZ exists from the start of the block scope until the declaration statement is executed. Understanding TDZ is crucial for writing predictable JavaScript code and avoiding subtle bugs.

- **TDZ starts** when the scope begins
- **TDZ ends** when the declaration statement is reached
- **Accessing variables during TDZ** throws `ReferenceError`
- **`var` does NOT have TDZ** — it's initialized to `undefined`
- **Function parameters** can also create TDZ scenarios

## Visual Representation

```
TEMPORAL DEAD ZONE TIMELINE
═══════════════════════════════════════════════════════════

Block Scope Created
        ↓
┌───────────────────────────────────────┐
│ TDZ ZONE - Variable Inaccessible      │
│                                       │
│ console.log(x);  ❌ ReferenceError    │
│                                       │
│ let x;  ← TDZ ENDS HERE               │
│ x = 10;                               │
│                                       │
│ console.log(x);  ✅ Works (value: 10) │
└───────────────────────────────────────┘

COMPARISON: var vs let/const
═══════════════════════════════════════════════════════════

var x:
    ↓
┌─────────────────────┐
│ Hoisted to undefined│
│ Accessible: YES     │
└─────────────────────┘

let x / const x:
    ↓
┌─────────────────────┐
│ Hoisted but NOT     │
│ initialized (TDZ)   │
│ Accessible: NO ❌   │
└─────────────────────┘
```

## Example

### Basic TDZ Example

```javascript
console.log(x);  // ReferenceError: Cannot access 'x' before initialization
let x = 5;       // TDZ ends here at declaration
```

### TDZ with Block Scope

```javascript
{
  console.log(x);  // ReferenceError (hoisted but in TDZ)
  let x = 10;      // TDZ ends here
}
```

### TDZ Shadowing Issue

```javascript
let x = "outer";
{
  console.log(x);  // ReferenceError! (not outer x, but hoisted inner x in TDZ)
  let x = "inner"; // Shadows the outer x
}
```

### TDZ with Default Parameters

```javascript
const value = 10;
function fn(a = value) {
  // value is accessible here (not in TDZ)
  console.log(a);
}
```

### TDZ with const

```javascript
console.log(PI);    // ReferenceError (TDZ)
const PI = 3.14159; // TDZ ends here at declaration
```

## Usage

### When to Use This Knowledge

- **Avoid ReferenceError bugs**: Understand why accessing a variable throws an error
- **Debug variable access issues**: Know that TDZ is creating the "Cannot access before initialization" error
- **Write safer code**: TDZ forces you to declare before using, preventing bugs
- **Understand scope properly**: TDZ helps clarify how block scope works
- **Interview preparation**: TDZ is a popular advanced JavaScript interview question

### Real-world Example

```javascript
// React hook - avoiding TDZ issues
function UserProfile({ userId }) {
  const id = userId || 1;  // Declare before use

  useEffect(() => {
    const fetchUser = async () => {
      const response = await fetch(`/api/users/${id}`);
      return response.json();
    };
    fetchUser();
  }, [id]);
}
```

### Best Practices

- **Declare variables before using them**: Simple and clear
- **Use `const` by default**: Prevents reassignment issues
- **Avoid accessing variables in TDZ**: Leads to ReferenceError
- **Understand block scope**: TDZ is block-scoped, not function-scoped
- **Use linters**: ESLint catches TDZ violations automatically
- **Think linearly**: Read code top-to-bottom and declare before use

## FAQ / Interview Questions

**Q: What is the Temporal Dead Zone?**
A: The Temporal Dead Zone (TDZ) is the region where a variable is hoisted but not initialized. Any attempt to access `let` or `const` variables during TDZ throws a `ReferenceError: Cannot access [variable] before initialization`. The TDZ starts at the beginning of the block scope and ends when the declaration statement is reached.

**Q: Why does `let x` cause a ReferenceError while `var x` returns undefined?**
A: With `var`, the variable is hoisted AND initialized to `undefined`, so it's accessible before the assignment. With `let` and `const`, the variable is hoisted but NOT initialized, keeping it in the Temporal Dead Zone. This design prevents accidental use of uninitialized variables.

**Q: Can you explain this code snippet?**
```javascript
let x = "outer";
{
  console.log(x);  // ReferenceError
  let x = "inner";
}
```
A: The `let x = "inner"` declaration creates a new variable in the block scope. JavaScript hoists this `x` to the top of the block, putting it in TDZ. When `console.log(x)` executes, `x` is still in TDZ (hoisted but not initialized), causing a ReferenceError. It doesn't fall back to the outer `x`.

**Q: Does `const` have a Temporal Dead Zone?**
A: Yes, `const` also has a Temporal Dead Zone, just like `let`. The TDZ exists until the `const` declaration statement is reached. Additionally, `const` must be initialized at declaration — you cannot declare it without assigning a value.

**Q: How can TDZ be helpful in writing better code?**
A: TDZ forces you to declare variables before using them, preventing bugs from accidental use of uninitialized variables. It makes code more predictable and easier to reason about. It also encourages using `const` and `let` instead of `var`, which have better scoping semantics.

## References

- [MDN - Temporal Dead Zone](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/let#Temporal_Dead_Zone)
- [MDN - let Statement](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/let)
- [MDN - const Statement](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/const)
- [JavaScript.info - Hoisting and TDZ](https://javascript.info/closure)

---

*See also: [Hoisting](./Hoisting.md), [Scope](./Scope.md), [var vs let vs const](./VarLetConst.md), [Block Scope](./BlockScope.md)*
