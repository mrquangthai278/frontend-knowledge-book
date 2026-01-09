# Variables in JavaScript

## Definition / Concept

**Variables** are named containers that store values in JavaScript. They allow you to save data and reference it by name throughout your code. JavaScript provides three keywords to declare variables: `var`, `let`, and `const`. Each has different **scope** rules and **hoisting** behavior. `const` should be your default choice for immutability, `let` when you need to reassign, and `var` should be avoided in modern JavaScript. Variables can hold any data type and their type can change at runtime.

- **`const`** declares block-scoped variables that cannot be reassigned
- **`let`** declares block-scoped variables that can be reassigned
- **`var`** declares function-scoped variables (avoid in modern code)
- **Block scope** limits variable access to the nearest enclosing block
- **Reassignment vs. Mutation** — `const` prevents reassignment but allows object mutations

## Visual Representation

```
VARIABLE DECLARATION COMPARISON
═══════════════════════════════════════════════════════════

┌──────────────────────────────────────────────────────────┐
│                      const vs let vs var                 │
├──────────┬─────────────┬──────────────┬─────────────────┤
│ Feature  │    const    │     let      │       var       │
├──────────┼─────────────┼──────────────┼─────────────────┤
│ Scope    │ Block       │ Block        │ Function        │
│ Reassign │ ✗ No        │ ✓ Yes        │ ✓ Yes           │
│ Hoist    │ TDZ*        │ TDZ*         │ undefined       │
│ Redecl   │ ✗ No        │ ✗ No         │ ✓ Yes (bad)     │
│ Modern   │ ✓ Preferred │ ✓ Use when   │ ✗ Avoid         │
│          │             │ needed       │                 │
└──────────┴─────────────┴──────────────┴─────────────────┘
           * TDZ = Temporal Dead Zone

VARIABLE SCOPE LEVELS
═══════════════════════════════════════════════════════════

Global Scope (top level)
│
├─ Function Scope (inside function)
│  ├─ Block Scope (inside if, loop, etc.)
│  │  ├─ let x = 1  (accessible only here)
│  │  └─ const y = 2 (accessible only here)
│  │
│  └─ var z = 3  (accessible to entire function)
│
└─ Outside function (cannot access let/const from function)

CONST VS LET IN PRACTICE
═══════════════════════════════════════════════════════════

const obj = { name: "Alice" };
obj = {};  // ✗ Error: reassignment blocked

obj.name = "Bob"; // ✓ OK: mutation allowed (not reassignment)
obj.age = 30;     // ✓ OK: adding properties allowed

const arr = [1, 2, 3];
arr = [4, 5, 6]; // ✗ Error: reassignment blocked
arr[0] = 99;     // ✓ OK: mutation of array contents allowed
arr.push(4);     // ✓ OK: array methods work fine
```

## Example

### const, let, and var

```javascript
// const - prevents reassignment (use by default)
const maxAttempts = 3;
maxAttempts = 5; // ✗ TypeError: Assignment to constant variable

// let - allows reassignment
let score = 0;
score = 10; // ✓ OK

// var - function-scoped (avoid in modern code)
var oldStyle = "not recommended";
var oldStyle = "redeclared"; // ✓ Allowed, but confusing
```

### Block Scope vs Function Scope

```javascript
// let/const - block-scoped (inside if/loop)
if (true) {
  const blockVar = "only in this block";
  console.log(blockVar); // ✓ Works
}
console.log(blockVar); // ✗ ReferenceError

// var - function-scoped (entire function)
if (true) {
  var funcVar = "entire function scope";
}
console.log(funcVar); // ✓ Works
```

### Loop Variables

```javascript
// let - each iteration has its own i
for (let i = 0; i < 3; i++) {
  setTimeout(() => console.log(i), 100);  // 0, 1, 2
}

// var - all iterations share same i
for (var j = 0; j < 3; j++) {
  setTimeout(() => console.log(j), 100);  // 3, 3, 3
}
```

### Initialization and Temporal Dead Zone

```javascript
// const and let have Temporal Dead Zone
function tdz() {
  console.log(x); // ✗ ReferenceError: Cannot access 'x' before initialization
  const x = 5;
}

// var gets hoisted with undefined
function hoisting() {
  console.log(y); // undefined (not an error)
  var y = 10;
}

// Variables can be declared without initialization
let uninitialized;
console.log(uninitialized); // undefined

const mustInit = 5; // const requires initialization

// Declare without initialization, then assign
let name;
name = "John"; // ✓ OK
```

### Const with Objects and Arrays

```javascript
// const prevents reassignment, allows mutation
const user = { name: "John", age: 30 };

user = {};  // ✗ Error: reassignment fails
user.name = "Jane";  // ✓ OK: mutation allowed
user.email = "jane@example.com";  // ✓ OK: add property

const colors = ["red", "green"];
colors = ["blue"];  // ✗ Error: reassignment fails
colors[0] = "orange";  // ✓ OK: mutation allowed
colors.push("yellow");  // ✓ OK: array methods work

// For truly immutable objects, use Object.freeze()
const frozen = Object.freeze({ name: "Alice" });
frozen.name = "Bob";  // ✗ Fails silently (or throws in strict mode)
```

### Variable Naming

```javascript
const MAX_RETRIES = 3;           // Constants: UPPER_SNAKE_CASE
const userName = "John";         // Regular: camelCase
const isActive = true;           // Boolean: is/has/can/should prefix

for (let i = 0; i < 10; i++) {}  // ✓ OK: use i in loops only
const x = 5;                     // ✗ Avoid: unclear names
const totalPrice = 5;            // ✓ Clear and descriptive
```

## Usage

### When to Use This Knowledge

- **Writing modern JavaScript**: Choosing between const/let/var properly
- **Preventing bugs**: Understanding scope and hoisting avoids common mistakes
- **Code review**: Identifying scope-related issues in other code
- **Performance**: Proper scoping helps garbage collection
- **Security**: Block scoping prevents unintended variable sharing

### Real-world Example

```javascript
// User login state management
class UserManager {
  constructor() {
    const users = [];       // const: doesn't reassign
    let currentUser = null; // let: reassign on login/logout
    const MAX_USERS = 100;  // const: never changes

    this.login = (username) => {
      const user = users.find(u => u.name === username);
      if (user) currentUser = user;  // Reassign when needed
      return !!user;
    };

    this.logout = () => { currentUser = null; };
    this.getUser = () => currentUser;
  }
}
```

### Best Practices

- **Use `const` by default** — it prevents accidental reassignment
- **Use `let` when reassignment is needed** — clearly signals intent
- **Never use `var`** in modern JavaScript — use const/let instead
- **Declare variables close to where they're used** — improves readability
- **Use descriptive names** — avoid single letters (except in loops)
- **Use const for objects/arrays** that won't be reassigned, even if their contents change
- **Declare variables in the smallest scope** where they're needed
- **Use UPPER_CASE for constants** that never change at runtime

## FAQ / Interview Questions

**Q: What's the difference between `const`, `let`, and `var`?**
A: `const` declares block-scoped variables that cannot be reassigned (though objects/arrays can be mutated). `let` declares block-scoped variables that can be reassigned. `var` declares function-scoped variables and should be avoided. All are hoisted, but `var` is initialized with `undefined` while `const`/`let` are not initialized (Temporal Dead Zone).

**Q: Can you reassign a `const` variable?**
A: No, reassigning a `const` variable throws a TypeError. However, you can mutate the contents of a `const` object or array—for example, `const obj = {}; obj.x = 1;` works fine. `const` prevents reassignment, not mutation.

**Q: What's the difference between block scope and function scope?**
A: Block scope (used by `let` and `const`) limits a variable to the nearest enclosing block (if statement, loop, function, etc.). Function scope (used by `var`) limits a variable to the entire function. Block scope is safer and prevents accidental variable sharing across different blocks.

**Q: Why should we avoid using `var` in modern JavaScript?**
A: `var` has confusing behavior: it's function-scoped (not block-scoped), allows re-declaration in the same scope, and has unexpected hoisting. These quirks cause bugs. `let` and `const` are safer with block scoping, no re-declaration, and clearer semantics. Using them prevents entire categories of bugs.

**Q: What is the Temporal Dead Zone (TDZ)?**
A: The Temporal Dead Zone is the period from when a scope starts until a `let` or `const` declaration is reached. Accessing the variable during TDZ throws a ReferenceError, even though the variable is hoisted. This makes TDZ safer than `var`'s `undefined` initialization because it forces you to declare before using.

## References

- [MDN - var, let, and const](https://developer.mozilla.org/en-US/docs/Learn/JavaScript/First_steps/Variables)
- [MDN - Scope](https://developer.mozilla.org/en-US/docs/Glossary/Scope)
- [MDN - Hoisting](https://developer.mozilla.org/en-US/docs/Glossary/Hoisting)
- [JavaScript.info - Variables](https://javascript.info/variables)

---

*See also: [Hoisting](./Hoisting.md), [Temporal Dead Zone](./TemporalDeadZone.md), [Scope Chain](./ScopeChain.md), [Typeof](./Typeof.md)*
