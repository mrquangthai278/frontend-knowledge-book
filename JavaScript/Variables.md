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

### Declaring Variables with const, let, and var

```javascript
// const - use by default
const maxAttempts = 3;
const userName = "Alice";
const userSettings = { theme: "dark", language: "en" };

// Const prevents reassignment
maxAttempts = 5; // ✗ TypeError: Assignment to constant variable

// But allows mutation (changing object properties)
userSettings.theme = "light"; // ✓ OK
userSettings.notifications = true; // ✓ OK

// let - use when you need to reassign
let score = 0;
score = 10; // ✓ OK
score++; // ✓ OK

let count;
count = 1; // Initialize later

// var - avoid in modern code
var oldStyle = "not recommended";
var oldStyle = "redeclared"; // ✓ Allowed, but confusing

// var is function-scoped, not block-scoped
function example() {
  var x = 1;
  if (true) {
    var x = 2; // overwrites outer x
  }
  console.log(x); // 2
}

// let is block-scoped
function example2() {
  let y = 1;
  if (true) {
    let y = 2; // separate from outer y
  }
  console.log(y); // 1
}
```

### Block Scope vs Function Scope

```javascript
// Block scope with let/const
function blockScopeExample() {
  if (true) {
    const blockVar = "I exist only in this block";
    let blockLet = "Me too";
    console.log(blockVar); // "I exist only in this block"
  }

  console.log(blockVar); // ✗ ReferenceError: blockVar is not defined
  console.log(blockLet); // ✗ ReferenceError: blockLet is not defined
}

// Function scope with var
function functionScopeExample() {
  if (true) {
    var funcVar = "I exist in the entire function";
  }

  console.log(funcVar); // "I exist in the entire function"
}

// Loop variable scoping
for (let i = 0; i < 3; i++) {
  setTimeout(() => {
    console.log(i); // 0, 1, 2 - each iteration has its own i
  }, 100);
}

for (var j = 0; j < 3; j++) {
  setTimeout(() => {
    console.log(j); // 3, 3, 3 - all reference the same j
  }, 100);
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
// Const prevents reassignment, not mutation
const user = {
  firstName: "John",
  lastName: "Doe",
  email: "john@example.com"
};

// Reassignment fails
user = { firstName: "Jane" }; // ✗ Error

// But mutation succeeds
user.firstName = "Jane"; // ✓ OK
user.phone = "555-1234"; // ✓ Add new property

// Same with arrays
const colors = ["red", "green", "blue"];

colors = ["yellow"]; // ✗ Error: reassignment

colors[0] = "orange"; // ✓ OK: mutation
colors.push("purple"); // ✓ OK: mutation
colors.pop(); // ✓ OK: mutation

// For truly immutable objects, use Object.freeze()
const frozenUser = Object.freeze({
  name: "Alice",
  id: 1
});

frozenUser.name = "Bob"; // fails silently in non-strict mode
console.log(frozenUser.name); // still "Alice"
```

### Variable Naming Conventions

```javascript
// Constants (never change)
const MAX_RETRIES = 3;
const API_ENDPOINT = "https://api.example.com";

// Regular variables (camelCase)
const userName = "John";
let userScore = 100;
const userPreferences = { theme: "dark" };

// Boolean variables (prefixed with is, has, can, should)
const isActive = true;
const hasPermission = false;
const canDelete = true;
const shouldUpdate = false;

// Avoid unclear names
let x = 5; // ✗ Unclear
const totalPrice = 5; // ✓ Clear

// Avoid single letters except for loops
for (let i = 0; i < 10; i++) { // ✓ OK in loops
  console.log(i);
}

// Avoid abbreviations
const usr = "John"; // ✗ Unclear
const user = "John"; // ✓ Clear
```

### Variable Reassignment Best Practices

```javascript
// Example: Counter pattern
const counter = {
  value: 0,
  increment() {
    this.value++;
  },
  decrement() {
    this.value--;
  },
  getValue() {
    return this.value;
  }
};

counter.increment(); // ✓ Mutate the object
console.log(counter.getValue()); // 1

// Example: Using let for changing state
let currentUser = null;

function login(user) {
  currentUser = user; // ✓ Reassign is appropriate here
}

function logout() {
  currentUser = null; // ✓ Reassign is appropriate here
}

// Example: Array manipulation
const numbers = [1, 2, 3];
const doubled = numbers.map(n => n * 2); // ✓ Create new array
// instead of trying to reassign: let numbers = ...
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
// User management system
class UserManager {
  constructor() {
    // Instance variables
    const users = []; // Private to constructor
    let loggedInUser = null;
    const MAX_USERS = 100;

    // Public methods
    this.addUser = function(user) {
      if (users.length >= MAX_USERS) {
        throw new Error(`Cannot exceed ${MAX_USERS} users`);
      }
      users.push(user);
    };

    this.login = function(username) {
      const user = users.find(u => u.name === username);
      if (user) {
        loggedInUser = user; // Reassign when needed
        return true;
      }
      return false;
    };

    this.getCurrentUser = function() {
      return loggedInUser;
    };

    this.logout = function() {
      loggedInUser = null;
    };
  }
}

// API request handler
async function fetchUserData(userId) {
  const baseUrl = "https://api.example.com"; // Constant
  let retries = 0; // Will be reassigned
  const maxRetries = 3;

  while (retries < maxRetries) {
    try {
      const response = await fetch(`${baseUrl}/users/${userId}`);
      const userData = await response.json();
      return userData;
    } catch (error) {
      retries++; // Reassignment needed
      if (retries >= maxRetries) {
        throw new Error("Failed to fetch user data after retries");
      }
    }
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
