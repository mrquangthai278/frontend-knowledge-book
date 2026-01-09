# This

## Definition / Concept
**`this`** is a special keyword in JavaScript that refers to the **execution context** of a function. The value of `this` is determined by **how** a function is called, not where it's defined (dynamic binding). In the global scope, `this` refers to the global object (window in browsers, global in Node.js). In object methods, `this` refers to the object. In arrow functions, `this` is inherited from the enclosing scope. Understanding `this` is crucial for writing correct object-oriented code, event handlers, and callbacks.

- `this` refers to the execution context/object
- Determined by how a function is called, not where it's defined
- Different values in different calling contexts
- Can be controlled with call(), apply(), or bind()
- Arrow functions inherit `this` from enclosing scope

## Visual Representation

```
Function Call Context Determines 'this':

1. Method Call:          2. Function Call:       3. Constructor:
   obj.method()             function()              new Class()
        │                        │                        │
        ├─→ this = obj      ├─→ this = window/global  ├─→ this = new instance
        │                        │                        │
   console.log(this)      console.log(this)      console.log(this)

4. Event Handler:        5. call/apply/bind:     6. Arrow Function:
   element.onclick           fn.call(obj)            const f = () => this
        │                        │                        │
   ├─→ this = element    ├─→ this = obj          ├─→ this = outer scope
        │                        │                        │
```

## Example

### Method Call

```javascript
const person = {
  name: 'Alice',
  greet() {
    console.log(this.name);  // this = person
  }
};
person.greet();  // 'Alice'
```

### Global Scope

```javascript
console.log(this);  // window (browser) or global (Node.js)
function show() { console.log(this); }  // undefined in strict mode
```

### Constructor

```javascript
function Person(name) {
  this.name = name;  // this = new instance
}
const alice = new Person('Alice');
```

### call(), apply(), bind()

```javascript
function greet(greeting) { console.log(`${greeting}, ${this.name}`); }

greet.call({ name: 'Alice' }, 'Hello');        // call: invoke now
greet.apply({ name: 'Bob' }, ['Hi']);          // apply: args as array
const boundGreet = greet.bind({ name: 'Eve' }); // bind: return new function
```

### Arrow Functions (inherit this)

```javascript
const obj = {
  name: 'Object',
  method() {
    const arrow = () => console.log(this.name);  // inherited
    arrow();  // 'Object'
  }
};
```

### Method Lost in Assignment

```javascript
const obj = {
  value: 42,
  getValue() { return this.value; }
};

const fn = obj.getValue;
fn();                         // undefined (this = window)
obj.getValue.bind(obj)();     // 42 (bind fixes this)
```

### Event Handlers

```javascript
// Wrong: this = element
element.addEventListener('click', function() {
  this.style.background = 'red';
});

// Correct: this = class instance (arrow inherits)
element.addEventListener('click', () => {
  this.style.background = 'red';  // inherited from outer scope
});
```

### Callbacks

```javascript
class DataFetcher {
  constructor() { this.data = []; }

  // Problem: this is lost
  fetch() {
    setTimeout(function() {
      console.log(this.data);  // undefined
    }, 1000);
  }

  // Solutions:
  fetchArrow() {
    setTimeout(() => { console.log(this.data); }, 1000);  // inherited
  }

  fetchBind() {
    setTimeout(function() { console.log(this.data); }.bind(this), 1000);
  }
}
```

## Usage

- **When to use**: Writing object methods, constructors, event handlers, and callbacks that need access to object context
- **Real-world example**: React class components use `this` to access state and methods; jQuery uses `this` in event handlers to reference the clicked element
- **Best practices**:
  - Prefer arrow functions in modern code (cleaner `this` binding)
  - Use `bind()` for methods that will be passed as callbacks
  - Avoid using `this` with arrow functions when you need dynamic context
  - Be explicit about `this` context with `call()`, `apply()`, or `bind()`
  - Document `this` behavior in comments for complex code
  - Consider using classes with arrow function methods for auto-binding
  - Remember: method calls get `this` automatically, extracted methods don't

## FAQ / Interview Questions

**Q: What does "this" refer to in JavaScript?**
A: `this` is a special keyword that refers to the **execution context** of a function—the object that a method is called on. The value of `this` is determined **dynamically at runtime** based on how the function is called, not where it's defined. For example, when `obj.method()` is called, `this` inside `method()` refers to `obj`.

**Q: Why does "this" change value based on how a function is called?**
A: JavaScript uses **dynamic binding** for `this`. This allows the same function to work with different objects. For example:
```javascript
const greet = function() { console.log(this.name); };
const alice = { name: 'Alice' };
const bob = { name: 'Bob' };

greet.call(alice);  // 'Alice' (this = alice)
greet.call(bob);    // 'Bob' (this = bob)
```
The same function produces different results based on the calling context, which is powerful but requires careful understanding.

**Q: What's the difference between call(), apply(), and bind()?**
A: All three allow you to explicitly set `this`:
- **call()**: Invokes immediately, pass arguments individually: `fn.call(obj, arg1, arg2)`
- **apply()**: Invokes immediately, pass arguments as array: `fn.apply(obj, [arg1, arg2])`
- **bind()**: Returns new function with fixed `this`, doesn't invoke: `const newFn = fn.bind(obj)`

Use `call()` or `apply()` to invoke with specific context. Use `bind()` to create a function with permanently fixed context (useful for callbacks).

**Q: How do arrow functions handle "this" differently?**
A: Arrow functions **don't have their own `this`**—they inherit `this` from the enclosing scope (lexical binding). This prevents the common problem of losing `this` in callbacks:
```javascript
const obj = {
  name: 'Object',
  method() {
    setTimeout(() => {
      console.log(this.name);  // 'Object' (inherited from method)
    }, 100);
  }
};
```
This is why arrow functions are preferred in modern code, especially for callbacks and event handlers.

**Q: Why do event handlers use "this" to reference the element?**
A: When an event handler is called, JavaScript automatically sets `this` to the element that triggered the event. This is useful for accessing element properties:
```javascript
element.addEventListener('click', function() {
  this.style.background = 'red';  // this = clicked element
});
```
With arrow functions, `this` is inherited instead, so you need to use the `event.target` or parameter reference.

## References
- [MDN Web Docs - this](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/this)
- [MDN Web Docs - Function.prototype.bind()](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Function/bind)
- [JavaScript.info - Object methods, "this"](https://javascript.info/object-methods)
- [Eloquent JavaScript - The Secret Life of Objects](https://eloquentjavascript.net/06_object.html)

---
*See also: [Closure](Closure.md), [Scope Chain](ScopeChain.md), [Arrow Functions](ArrowFunctions.md), [Prototype](Prototype.md)*
