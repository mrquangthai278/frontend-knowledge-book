# Generator

## Definition / Concept

A **generator** is a special function that can pause and resume execution, returning an **iterator** object. Generators are declared with the `function*` syntax and use the `yield` keyword to produce a sequence of values over time. Unlike regular functions that run to completion, generators allow you to **generate values on-demand**, enabling **lazy evaluation**, **infinite sequences**, and **cleaner async code**. They are fundamental to modern JavaScript's iteration protocol.

- Pauses execution with `yield` and resumes when requested
- Returns an **iterator** object that follows the iteration protocol
- Enables **lazy evaluation** and memory-efficient processing
- Foundation for **async/await** and **Promise handling**
- Simplifies complex **state management** and **sequential operations**

## Visual Representation

```
Regular Function:
┌──────────────────────────────────────────┐
│  function multiply(a, b)                 │
│  ├─ Execute all code                     │
│  ├─ Return single value                  │
│  └─ Done (can't resume)                  │
└──────────────────────────────────────────┘

Generator Function:
┌──────────────────────────────────────────┐
│  function* generator()                   │
│  ├─ Pause at yield #1 (return value 1)   │
│  ├─ Resume: Pause at yield #2 (return 2)│
│  ├─ Resume: Pause at yield #3 (return 3)│
│  ├─ Resume: Return final value           │
│  └─ Done (exhausted)                     │
└──────────────────────────────────────────┘

Iterator Protocol:
next() → { value: 1, done: false }
next() → { value: 2, done: false }
next() → { value: 3, done: false }
next() → { value: undefined, done: true }
```

## Example

### Basic Generator

```javascript
function* simpleGenerator() {
  yield 1;
  yield 2;
  yield 3;
}

const gen = simpleGenerator();
console.log(gen.next());  // { value: 1, done: false }
console.log(gen.next());  // { value: 2, done: false }
```

### Using for...of

```javascript
function* countToN(n) {
  for (let i = 1; i <= n; i++) {
    yield i;
  }
}

for (const num of countToN(3)) {
  console.log(num);  // 1, 2, 3
}
```

### Two-Way Communication

```javascript
function* dialogue() {
  const name = yield "What is your name?";
  const age = yield `Hello ${name}, what is your age?`;
  yield `${name} is ${age} years old!`;
}

const conv = dialogue();
conv.next().value;           // "What is your name?"
conv.next("Alice").value;    // "Hello Alice, what is your age?"
conv.next(25).value;         // "Alice is 25 years old!"
```

### Lazy Evaluation

```javascript
function* infiniteSequence() {
  let id = 0;
  while (true) yield id++;  // Generates values on demand
}

const ids = infiniteSequence();
console.log(ids.next().value);  // 0 (memory efficient)
```

### Generator Delegation

```javascript
function* delegating() {
  yield* [1, 2, 3];       // Delegate to array
  yield* countToN(2);     // Delegate to another generator
}
```

## Usage

- **When to use**: Processing large datasets lazily, creating infinite sequences, implementing async patterns, simplifying iterators, managing complex state machines
- **Real-world example**: Paginating through API results without loading all data, streaming file processing, implementing range/filter operations, Redux sagas for side effects, state machine workflows
- **Best practices**:
  - Use `for...of` loop for simpler iteration instead of manual `next()` calls
  - Use `yield*` to delegate to other generators
  - Remember that generators are **lazy**—values are computed on demand
  - Combine with Promise for async operations (though async/await is often clearer)
  - Document what each `yield` produces for readability
  - Use generators for **memory-efficient** processing of large datasets

## FAQ / Interview Questions

**Q: What's the difference between a generator and a regular function?**
A: Regular functions execute completely and return once. Generators pause at each `yield`, returning control until `next()` is called again. Regular functions return a value; generators return an **iterator object** that produces multiple values over time. This allows generators to be **memory efficient** and **lazy**.

**Q: How does `yield` differ from `return`?**
A: `return` exits the function permanently and returns a value. `yield` pauses the generator and produces a value, but the function can resume from that exact point when `next()` is called again. A generator can have multiple `yield` statements but typically only one final `return`.

**Q: Can you pass values into a generator?**
A: Yes! You can pass values via `next(value)`. The value becomes the result of the `yield` expression:
```javascript
function* gen() {
  const x = yield "pause";  // x will be whatever we pass to next()
  console.log(x);
}
const g = gen();
g.next();      // Pauses at yield
g.next(42);    // x becomes 42, logs 42
```

**Q: What's `yield*` (yield with asterisk)?**
A: `yield*` **delegates** to another generator or iterable. It yields all values from the delegated generator:
```javascript
function* gen1() { yield 1; yield 2; }
function* gen2() { yield* gen1(); yield 3; }
// Equivalent to manually yielding each value from gen1
```

**Q: Are generators used for async code?**
A: Generators were used with **Promises** as a pattern before `async/await` existed. However, `async/await` is now the standard for async code. Generators are still useful for sync iteration, lazy evaluation, and complex control flow, but for async operations, `async/await` is clearer and more readable.

## References
- [MDN - Generators](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Iterators_and_Generators)
- [MDN - function*](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/function*)
- [JavaScript.info - Generators](https://javascript.info/generators)
- [Exploring ES6 - Generators](https://exploringjs.com/es6/ch_generators.html)

---

*See also: [Iterator](./Iterator.md), [Async/Await](./AsyncAwait.md), [Promises](./Promises.md), [Closures](./Closure.md)*
