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

### Implicit Globals

```javascript
// Normal mode - allows implicit globals
implicitGlobal = 'allowed';  // Creates global variable

// Strict mode - requires declaration
'use strict';
implicitGlobal = 'not allowed';  // ✗ ReferenceError
```

### this in Functions

```javascript
// Normal mode
function normal() { console.log(this); }
normal();  // window/global

// Strict mode
'use strict';
function strict() { console.log(this); }
strict();  // undefined
```

### Delete Variables

```javascript
// Normal mode
var x = 1;
delete x;  // ✓ Works

// Strict mode
'use strict';
var y = 1;
delete y;  // ✗ SyntaxError
```

### Function Parameters

```javascript
'use strict';

// Duplicate parameter names forbidden
// function test(a, a, b) { }  // ✗ SyntaxError
```

### call() Behavior

```javascript
'use strict';

function test() { console.log(this); }

test.call(null);       // null (not window)
test.call(undefined);  // undefined
test.call(5);          // 5 (not boxed to Number)
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
