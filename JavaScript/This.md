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

```javascript
// ============================================
// 1. Global Scope
// ============================================

console.log(this);  // In browser: window
                    // In Node.js: global
                    // In module: {}

// ============================================
// 2. Method Call (this = object)
// ============================================

const person = {
  name: 'Alice',
  greet() {
    console.log(`Hello, I'm ${this.name}`);
  }
};

person.greet();  // 'Hello, I'm Alice' (this = person)

// ============================================
// 3. Function Call (this = global/undefined in strict mode)
// ============================================

function sayName() {
  console.log(this.name);
}

sayName();  // undefined (this = window/global)

// In strict mode:
function strictFunc() {
  'use strict';
  console.log(this);  // undefined
}
strictFunc();

// ============================================
// 4. Constructor Function (this = new instance)
// ============================================

function Person(name) {
  this.name = name;
  this.greet = function() {
    console.log(`Hi, I'm ${this.name}`);
  };
}

const alice = new Person('Alice');
alice.greet();  // 'Hi, I'm Alice' (this = alice)

// ============================================
// 5. Method Assignment Problem
// ============================================

const obj = {
  name: 'Object',
  method() {
    console.log(this.name);
  }
};

obj.method();        // 'Object' (called on obj)

const fn = obj.method;
fn();                // undefined (called standalone, this = window)

// Solution: Use bind
const boundFn = obj.method.bind(obj);
boundFn();           // 'Object' (this always = obj)

// ============================================
// 6. call(), apply(), bind() - Explicit this
// ============================================

function introduce(greeting) {
  console.log(`${greeting}, I'm ${this.name}`);
}

const person1 = { name: 'Alice' };
const person2 = { name: 'Bob' };

// call: invoke immediately, pass args individually
introduce.call(person1, 'Hello');      // 'Hello, I'm Alice'

// apply: invoke immediately, pass args as array
introduce.apply(person2, ['Hi']);      // 'Hi, I'm Bob'

// bind: return new function with fixed this
const greetAlice = introduce.bind(person1);
greetAlice('Hey');                     // 'Hey, I'm Alice'

// ============================================
// 7. Arrow Functions (inherit this)
// ============================================

const obj2 = {
  name: 'Object',
  regularMethod() {
    console.log('Regular:', this.name);  // 'Object'

    const arrowFunc = () => {
      console.log('Arrow:', this.name);   // 'Object' (inherited)
    };
    arrowFunc();
  },
  wrongMethod: () => {
    console.log('Arrow method:', this.name);  // undefined (this from outer scope)
  }
};

obj2.regularMethod();   // Regular: Object, Arrow: Object
obj2.wrongMethod();     // Arrow method: undefined

// ============================================
// 8. Event Handlers
// ============================================

class Button {
  constructor() {
    this.text = 'Click me';
  }

  // Wrong: this = button element
  setupWrong() {
    document.getElementById('btn').addEventListener('click', function() {
      console.log(this.text);  // undefined (this = button element)
    });
  }

  // Correct: this = Button instance (arrow function)
  setupCorrect() {
    document.getElementById('btn').addEventListener('click', () => {
      console.log(this.text);  // 'Click me' (this inherited from class)
    });
  }

  // Also correct: use bind
  setupBind() {
    document.getElementById('btn').addEventListener('click', function() {
      console.log(this.text);  // 'Click me'
    }.bind(this));
  }
}

// ============================================
// 9. Class Methods
// ============================================

class Counter {
  constructor(initialValue = 0) {
    this.value = initialValue;
  }

  increment() {
    this.value++;
    console.log(this.value);
  }

  // Bound method (auto-bound in classes)
  getCount = () => {
    return this.value;
  };
}

const counter = new Counter(5);
counter.increment();      // 6 (this = counter)

// But if you extract a method:
const increment = counter.increment;
// increment();           // Error: can't read property 'value' of undefined

// Solution: bind in constructor
class BetterCounter {
  constructor(initialValue = 0) {
    this.value = initialValue;
    this.increment = this.increment.bind(this);
  }

  increment() {
    this.value++;
  }
}

// ============================================
// 10. this in Callbacks
// ============================================

class DataFetcher {
  constructor() {
    this.data = [];
  }

  // Problem: this is lost in callback
  fetchWrong() {
    setTimeout(function() {
      console.log(this.data);  // undefined
    }, 1000);
  }

  // Solution 1: arrow function
  fetchCorrect1() {
    setTimeout(() => {
      console.log(this.data);  // [] (inherited)
    }, 1000);
  }

  // Solution 2: bind
  fetchCorrect2() {
    setTimeout(function() {
      console.log(this.data);  // []
    }.bind(this), 1000);
  }

  // Solution 3: store this reference
  fetchCorrect3() {
    const self = this;
    setTimeout(function() {
      console.log(self.data);  // []
    }, 1000);
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
