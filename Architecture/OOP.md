# Object-Oriented Programming (OOP)

## Definition / Concept

**Object-Oriented Programming (OOP)** is a programming paradigm that organizes code around **objects** and **classes**, which bundle data (properties) and behavior (methods) together. OOP enables you to create reusable, modular code through **encapsulation**, **inheritance**, and **polymorphism**. It's a foundational concept in frontend architecture, providing structure for building scalable applications regardless of language.

- Objects encapsulate data and methods, promoting modularity
- Classes define blueprints for creating multiple instances
- Inheritance allows code reuse through class hierarchies
- Polymorphism enables flexible, extensible code through method overriding

## Visual Representation

```
Class (Blueprint)
    ↓
[Constructor] → Creates instances
    ↓
Object Instance
├── Properties (data)
└── Methods (behavior)
    ↓
Inheritance Chain
    ↓
Parent Class → Child Class
```

## Example

```javascript
// ES6 Class Syntax
class Animal {
  constructor(name, species) {
    this.name = name;
    this.species = species;
  }

  speak() {
    console.log(`${this.name} makes a sound`);
  }

  getInfo() {
    return `${this.name} is a ${this.species}`;
  }
}

// Inheritance
class Dog extends Animal {
  constructor(name, breed) {
    super(name, 'Dog'); // Call parent constructor
    this.breed = breed;
  }

  speak() {
    console.log(`${this.name} barks!`); // Override parent method
  }

  getBreedInfo() {
    return `${this.getInfo()} of breed ${this.breed}`;
  }
}

// Usage
const dog = new Dog('Rex', 'Golden Retriever');
dog.speak(); // Rex barks!
console.log(dog.getBreedInfo()); // Rex is a Dog of breed Golden Retriever
```

## Usage

- **When to use**: Building complex applications with multiple entities that share behavior, creating reusable components, organizing large codebases, designing scalable system architecture
- **Real-world example**: A UI component library where Button, Modal, and Dialog all extend a base Component class with common lifecycle methods
- **Best practices**:
  - Keep classes focused on single responsibility
  - Use composition over inheritance when appropriate
  - Encapsulate private data with `#` private fields
  - Avoid deep inheritance hierarchies (prefer composition)
  - Apply SOLID principles to OOP design

## FAQ / Interview Questions

**Q: What are the four pillars of OOP?**
A: The four pillars are:
1. **Encapsulation** - Bundling data and methods, hiding internal details
2. **Abstraction** - Exposing only necessary interfaces, hiding complexity
3. **Inheritance** - Creating new classes from existing ones, promoting code reuse
4. **Polymorphism** - Objects of different types responding to the same method call differently

**Q: When should you use OOP in frontend architecture?**
A: Use OOP when:
- Building component-based frameworks (React class components, Angular services)
- Creating reusable UI patterns with shared behavior
- Managing complex state and business logic
- Organizing code into logical, maintainable modules
- However, functional programming with closures is often preferred in modern JavaScript

**Q: What's the difference between composition and inheritance? When should you use each?**
A: **Inheritance** creates "is-a" relationships (Dog is-a Animal). **Composition** creates "has-a" relationships (Car has-a Engine). Composition is often preferred because it's more flexible and avoids tight coupling. Use inheritance for true hierarchies, composition for flexible feature mixing.

**Q: How does OOP relate to design patterns?**
A: Many design patterns (Singleton, Factory, Observer, Decorator) rely on OOP principles. OOP provides the mechanisms to implement these patterns effectively, while design patterns show best practices for solving common problems with OOP.

**Q: What does the `super` keyword do?**
A: `super` calls methods or accesses properties from the parent class. It's required in a child class constructor before accessing `this`, and can be used to call parent methods explicitly:
```javascript
class Child extends Parent {
  constructor() {
    super(); // Call parent constructor
  }

  method() {
    super.method(); // Call parent method
  }
}
```

## References
- [MDN Web Docs - Object-Oriented JavaScript](https://developer.mozilla.org/en-US/docs/Learn/JavaScript/Objects)
- [MDN - Classes](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Classes)
- [SOLID Principles Guide](https://en.wikipedia.org/wiki/SOLID)
- [Design Patterns and OOP](https://refactoring.guru/design-patterns)

---
*See also: [Design Patterns](./DesignPatterns.md), [SOLID Principles](./SOLIDPrinciples.md)*
