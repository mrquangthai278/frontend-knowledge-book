# Strict Mode

## Definition / Concept
**Strict mode** is a restricted variant of JavaScript that enforces stricter parsing and error-handling rules. Enabled with the `'use strict'` directive, it makes several changes to normal semantics: variables must be declared, `this` is undefined in functions (not the global object), deleting variables throws errors, and deprecated features are forbidden. Strict mode catches common mistakes, prevents "unsafe" actions, and improves performance because the engine can optimize code more aggressively. It's the default behavior in ES6 modules and classes.

- Enforces stricter syntax and error handling
- Prevents implicit global variable creation
- Changes `this` behavior in functions
- Disables dangerous features
- Improves code performance and safety

## Visual Representation

```
Normal Mode vs Strict Mode:

NORMAL MODE:
┌──────────────────────────────┐
│ var x = 1;                   │
│ function test() {            │
│   this.name = 'Alice';       │
│   y = 5;  ✓ Creates global   │
│   delete x;  ✓ Allowed       │
│ }                            │
└──────────────────────────────┘

STRICT MODE:
┌──────────────────────────────┐
│ 'use strict';                 │
│ var x = 1;                   │
│ function test() {            │
│   this = undefined;          │
│   y = 5;  ✗ ReferenceError   │
│   delete x;  ✗ SyntaxError   │
│ }                            │
└──────────────────────────────┘
```

## Example

```javascript
// ============================================
// 1. Global Strict Mode
// ============================================

'use strict';

function strictFunction() {
  console.log('This entire file is in strict mode');
}

// ============================================
// 2. Function-Level Strict Mode
// ============================================

// Global scope: normal mode
var globalVar = 'global';

function normalFunction() {
  // This function is in normal mode
  implicitGlobal = 'allowed';
  console.log(implicitGlobal);  // 'allowed'
}

function strictFunction() {
  'use strict';
  // This function is in strict mode
  implicitVar = 'not allowed';  // ReferenceError: implicitVar is not defined
}

normalFunction();
// strictFunction();  // Would throw error

// ============================================
// 3. Variables Must Be Declared
// ============================================

'use strict';

// Normal mode allows implicit globals:
// x = 1;  // Creates window.x in browsers

// Strict mode requires declaration:
var y = 1;      // ✓ Correct
let z = 2;      // ✓ Correct
const a = 3;    // ✓ Correct
// b = 4;         // ✗ ReferenceError

// ============================================
// 4. this in Functions
// ============================================

function normalThis() {
  console.log(this);  // window (or global)
}

function strictThis() {
  'use strict';
  console.log(this);  // undefined
}

function strictMethod() {
  'use strict';

  const obj = {
    name: 'Object',
    method() {
      console.log(this);  // obj (method context still works)
    }
  };

  obj.method();
}

normalThis();
strictThis();
strictMethod();

// ============================================
// 5. Cannot Delete Variables
// ============================================

'use strict';

var x = 1;
let y = 2;

// delete x;  // ✗ SyntaxError: variable x cannot be deleted
// delete y;  // ✗ SyntaxError: variable y cannot be deleted

// Can only delete properties:
const obj = { prop: 1 };
delete obj.prop;  // ✓ Correct

// ============================================
// 6. Function Parameter Restrictions
// ============================================

'use strict';

// Duplicate parameter names not allowed:
// function test(a, a, b) {  // ✗ SyntaxError
//   console.log(a);
// }

// ============================================
// 7. Octal Literals Forbidden
// ============================================

'use strict';

// var x = 010;  // ✗ SyntaxError
var x = 0o10;   // ✓ Correct (ES6 octal syntax)

// ============================================
// 8. eval() Is Safer
// ============================================

'use strict';

// eval() doesn't create variables in surrounding scope
eval("var x = 1;");
console.log(typeof x);  // 'undefined' (in strict mode)
                        // 'number' (in normal mode)

// ============================================
// 9. arguments Object Restrictions
// ============================================

'use strict';

function test(a, b) {
  // arguments is not bound to parameters
  arguments[0] = 999;
  console.log(a);  // Still 1, not affected by arguments
}

test(1, 2);

// ============================================
// 10. with Statement Forbidden
// ============================================

'use strict';

// with statement is not allowed:
// const obj = { x: 1 };
// with (obj) {  // ✗ SyntaxError
//   console.log(x);
// }

// ============================================
// 11. call(), apply(), bind() Behavior
// ============================================

'use strict';

function normalCall() {
  console.log(this);  // window/global
}

function strictCall() {
  console.log(this);  // whatever context is passed
}

normalCall.call(null);     // window/global
strictCall.call(null);     // null
strictCall.call(undefined); // undefined
strictCall.call(5);        // 5 (not boxed to Number)

// ============================================
// 12. ES6 Features in Strict Mode
// ============================================

'use strict';

// Classes are always in strict mode
class MyClass {
  constructor(name) {
    this.name = name;
  }

  method() {
    // this is strictly bound
    console.log(this.name);
  }
}

// Arrow functions inherit strict mode from context
const obj = {
  name: 'Object',
  method: () => {
    console.log(this);  // undefined (strict mode inherited)
  }
};

// ============================================
// 13. Comparison: Normal vs Strict
// ============================================

// NORMAL MODE
function normalExample() {
  x = 1;                    // ✓ Creates global variable
  function inner() {
    console.log(this);      // window
  }
  delete x;                 // ✓ Works
  with (Math) {             // ✓ Allowed
    console.log(sqrt(16));
  }
}

// STRICT MODE
function strictExample() {
  'use strict';
  // x = 1;                 // ✗ ReferenceError
  // function inner() {
  //   console.log(this);   // undefined
  // }
  // delete x;              // ✗ SyntaxError
  // with (Math) { }        // ✗ SyntaxError
}

// ============================================
// 14. Transitioning to Strict Mode
// ============================================

// Best practice: use in modules or classes
// No need to manually add 'use strict'

// Module (automatically strict):
// export const myFunction = () => { ... };

// Class (automatically strict):
// class MyClass {
//   method() { ... }
// }

// Modern build tools and bundlers default to strict mode
```

## Usage

- **When to use**: Always use strict mode in modern code; it's the default in modules and classes
- **Real-world example**: All modern frameworks (React, Vue, Angular) use strict mode by default; modern build tools (Webpack, Vite) output strict mode
- **Best practices**:
  - Add `'use strict';` at the top of files in non-module code
  - Use ES6 modules (automatically strict)
  - Use classes (automatically strict)
  - Don't mix strict and non-strict functions in the same file
  - Enable strict mode for all new code
  - Use a linter (ESLint) to enforce strict mode
  - Modern bundlers handle strict mode automatically
  - Test code thoroughly when transitioning from normal to strict mode

## FAQ / Interview Questions

**Q: What is strict mode and why should I use it?**
A: Strict mode is a restricted variant of JavaScript that enforces stricter rules. You should use it because:
- **Catches mistakes**: Prevents implicit global variables, unsafe operations
- **Improves performance**: Engines optimize strict code better
- **Prevents errors**: Makes code more predictable and safer
- **Modern standard**: Default in ES6 modules and classes
- **Future-proof**: Prepares code for future JavaScript versions
Most modern code automatically uses strict mode through modules and classes.

**Q: What are the main differences between normal and strict mode?**
A: Key differences:
1. **Implicit globals**: Normal allows `x = 1` to create globals; strict throws error
2. **`this` in functions**: Normal: window/global; strict: undefined
3. **Delete operations**: Normal allows deleting variables; strict throws error
4. **Duplicate parameters**: Normal allows `function(a, a)`; strict throws error
5. **eval() scope**: Normal: can create variables in surrounding scope; strict: isolated scope
6. **Octal literals**: Normal allows `010`; strict requires `0o10`

**Q: Do I need to add 'use strict' if I use ES6 modules?**
A: No, ES6 modules are **automatically in strict mode**. Same with ES6 classes. If you're using modern JavaScript with modules and classes, you don't need to manually add `'use strict'`. However, if you're writing function-level strict mode or using non-module code, add it explicitly at the top.

**Q: Why does "this" become undefined in strict mode?**
A: In normal mode, `this` defaults to the global object (window/global) when a function is called without context. Strict mode prevents this auto-boxing because:
- It's often a source of bugs
- Explicit context is clearer and safer
- Event handlers still work (they set `this` to the element)
- Methods still work (they set `this` to the object)
Only plain function calls get `undefined`, which catches accidental globals.

**Q: Can I mix strict and non-strict code?**
A: Avoid mixing strict and non-strict code in the same file. You can have:
- Strict module + strict imports = safe
- Strict class + strict methods = safe
- Strict function + normal outer scope = confusing (don't do this)

Best practice is to make entire files strict by using modules or adding `'use strict'` at the top. Modern build tools handle this automatically.

**Q: How do I enable strict mode in my project?**
A: Depending on your setup:
- **ES6 modules**: Automatically strict (no action needed)
- **Classes**: Automatically strict (no action needed)
- **Regular scripts**: Add `'use strict';` at the top of each file
- **Build tools**: Most transpilers (Babel, Webpack) default to strict mode
- **Linters**: Use ESLint to enforce strict mode and catch violations
- **Node.js**: Use `'use strict';` or package with `"type": "module"`

## References
- [MDN Web Docs - Strict mode](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Strict_mode)
- [JavaScript.info - Strict mode](https://javascript.info/strict-mode)
- [ECMAScript 2015 - Strict Mode Code](https://www.ecma-international.org/publications-and-standards/standards/ecma-262/#sec-strict-mode-of-ecmascript)
- [Web.dev - JavaScript modules](https://web.dev/modules/)

---
*See also: [Scope](Scope.md), [This](This.md), [Hoisting](Hoisting.md), [Variable Declaration](VariableDeclaration.md)*
