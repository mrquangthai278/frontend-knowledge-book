# SOLID Design Principles

## Definition / Concept

**SOLID** is an acronym for five fundamental design principles that guide developers in writing maintainable, scalable, and flexible code architecture. These principles, introduced by Robert C. Martin, help minimize dependencies between modules, improve code readability, and facilitate easier testing and refactoring across all programming paradigms. SOLID principles apply universally to object-oriented design and are essential for building robust applications that can adapt to changing requirements without widespread code modifications. Each principle addresses a specific aspect of software design to reduce technical debt and improve long-term code quality.

- **Single Responsibility Principle (SRP)**: Each module or class should have one reason to change
- **Open/Closed Principle (OCP)**: Software entities should be open for extension, closed for modification
- **Liskov Substitution Principle (LSP)**: Derived classes must be substitutable for their base classes
- **Interface Segregation Principle (ISP)**: Clients should not depend on interfaces they don't use
- **Dependency Inversion Principle (DIP)**: Depend on abstractions, not on concrete implementations

## Visual Representation

```
┌─────────────────────────────────────────────────────┐
│              SOLID Design Principles               │
├─────────────────────────────────────────────────────┤
│                                                     │
│  S - Single Responsibility Principle                │
│      One class = One responsibility                 │
│                                                     │
│  O - Open/Closed Principle                         │
│      Open for extension, Closed for modification   │
│                                                     │
│  L - Liskov Substitution Principle                 │
│      Subtypes must be substitutable for base types │
│                                                     │
│  I - Interface Segregation Principle               │
│      Specific interfaces > General interfaces      │
│                                                     │
│  D - Dependency Inversion Principle                │
│      Depend on abstractions, not concretions       │
│                                                     │
└─────────────────────────────────────────────────────┘

Dependencies Flow:
┌──────────────┐
│ Abstractions │  ← High-level modules depend on
└──────────────┘
        ▲
        │ ← Low-level modules depend on
┌──────────────┐
│ Concretions  │
└──────────────┘
```

## Example

### Single Responsibility Principle (SRP)
```javascript
// ❌ BAD: UserManager handles multiple responsibilities
class UserManager {
  constructor(databaseUrl) {
    this.databaseUrl = databaseUrl;
  }

  createUser(userData) {
    // Database logic mixed with business logic
    const connection = this.openConnection(this.databaseUrl);
    const user = { id: Math.random(), ...userData };
    connection.insert('users', user);

    // Email sending logic - not a user management concern
    this.sendWelcomeEmail(user.email);

    // Logging - not a user management concern
    console.log(`User created: ${user.id}`);
    return user;
  }

  openConnection(url) { /* ... */ }
  sendWelcomeEmail(email) { /* ... */ }
}

// ✅ GOOD: Separate concerns into focused classes
class UserRepository {
  constructor(databaseUrl) {
    this.databaseUrl = databaseUrl;
  }

  createUser(userData) {
    const user = { id: Math.random(), ...userData };
    const connection = this.openConnection(this.databaseUrl);
    connection.insert('users', user);
    return user;
  }

  openConnection(url) { /* ... */ }
}

class EmailService {
  sendWelcomeEmail(email) {
    // Only responsible for email logic
    console.log(`Sending welcome email to ${email}`);
  }
}

class UserLogger {
  logUserCreation(userId) {
    // Only responsible for logging
    console.log(`User created: ${userId}`);
  }
}

// Usage
const userRepo = new UserRepository('db://localhost');
const emailService = new EmailService();
const logger = new UserLogger();

const newUser = userRepo.createUser({ name: 'John', email: 'john@example.com' });
emailService.sendWelcomeEmail(newUser.email);
logger.logUserCreation(newUser.id);
```

### Open/Closed Principle (OCP)
```javascript
// ❌ BAD: Adding new payment methods requires modifying PaymentProcessor
class PaymentProcessor {
  processPayment(amount, paymentMethod) {
    if (paymentMethod === 'credit-card') {
      return this.processCreditCard(amount);
    } else if (paymentMethod === 'paypal') {
      return this.processPayPal(amount);
    } else if (paymentMethod === 'crypto') {
      return this.processCrypto(amount);
    }
    throw new Error('Unknown payment method');
  }

  processCreditCard(amount) { /* ... */ }
  processPayPal(amount) { /* ... */ }
  processCrypto(amount) { /* ... */ }
}

// ✅ GOOD: Use polymorphism - open for extension, closed for modification
class PaymentMethod {
  process(amount) {
    throw new Error('Must implement process method');
  }
}

class CreditCardPayment extends PaymentMethod {
  process(amount) {
    console.log(`Processing credit card payment: $${amount}`);
    return { success: true, amount, method: 'credit-card' };
  }
}

class PayPalPayment extends PaymentMethod {
  process(amount) {
    console.log(`Processing PayPal payment: $${amount}`);
    return { success: true, amount, method: 'paypal' };
  }
}

class CryptoPayment extends PaymentMethod {
  process(amount) {
    console.log(`Processing crypto payment: $${amount}`);
    return { success: true, amount, method: 'crypto' };
  }
}

class PaymentProcessor {
  processPayment(amount, paymentMethod) {
    if (!(paymentMethod instanceof PaymentMethod)) {
      throw new Error('Invalid payment method');
    }
    return paymentMethod.process(amount);
  }
}

// Usage - add new payment methods without modifying PaymentProcessor
const processor = new PaymentProcessor();
processor.processPayment(100, new CreditCardPayment());
processor.processPayment(50, new PayPalPayment());
processor.processPayment(200, new CryptoPayment());
```

### Liskov Substitution Principle (LSP)
```javascript
// ❌ BAD: Square violates LSP by changing Rectangle's behavior
class Rectangle {
  constructor(width, height) {
    this.width = width;
    this.height = height;
  }

  setWidth(width) {
    this.width = width;
  }

  setHeight(height) {
    this.height = height;
  }

  getArea() {
    return this.width * this.height;
  }
}

class Square extends Rectangle {
  setWidth(width) {
    // Violates LSP: changes expected behavior
    this.width = width;
    this.height = width; // Forces height = width
  }

  setHeight(height) {
    // Violates LSP: changes expected behavior
    this.width = height;
    this.height = height; // Forces width = height
  }
}

// This breaks LSP - we can't substitute Square for Rectangle
function testRectangle(rect) {
  rect.setWidth(5);
  rect.setHeight(10);
  console.log(rect.getArea()); // Expected: 50, Square gives: 100
}

// ✅ GOOD: Use proper inheritance hierarchy
class Shape {
  getArea() {
    throw new Error('Must implement getArea');
  }
}

class Rectangle extends Shape {
  constructor(width, height) {
    super();
    this.width = width;
    this.height = height;
  }

  setWidth(width) {
    this.width = width;
  }

  setHeight(height) {
    this.height = height;
  }

  getArea() {
    return this.width * this.height;
  }
}

class Square extends Shape {
  constructor(sideLength) {
    super();
    this.sideLength = sideLength;
  }

  setSideLength(length) {
    this.sideLength = length;
  }

  getArea() {
    return this.sideLength * this.sideLength;
  }
}

// Now Square properly substitutes for Shape
function calculateArea(shape) {
  return shape.getArea();
}

const rect = new Rectangle(5, 10);
const square = new Square(7);

console.log(calculateArea(rect));   // 50
console.log(calculateArea(square)); // 49
```

### Interface Segregation Principle (ISP)
```javascript
// ❌ BAD: Fat interface forces unnecessary implementations
class Worker {
  work() {
    throw new Error('Must implement work');
  }

  eat() {
    throw new Error('Must implement eat');
  }

  sleep() {
    throw new Error('Must implement sleep');
  }
}

class Robot extends Worker {
  work() {
    console.log('Robot working...');
  }

  eat() {
    // Robots don't eat - forced to implement
    throw new Error('Robots do not eat');
  }

  sleep() {
    // Robots don't sleep - forced to implement
    throw new Error('Robots do not sleep');
  }
}

// ✅ GOOD: Segregate interfaces by responsibility
class Workable {
  work() {
    throw new Error('Must implement work');
  }
}

class Eatable {
  eat() {
    throw new Error('Must implement eat');
  }
}

class Sleepable {
  sleep() {
    throw new Error('Must implement sleep');
  }
}

class Human extends Workable {
  work() {
    console.log('Human working...');
  }

  eat() {
    console.log('Human eating...');
  }

  sleep() {
    console.log('Human sleeping...');
  }
}

class Robot extends Workable {
  work() {
    console.log('Robot working...');
  }
  // No need to implement eat() or sleep()
}

// Usage
const human = new Human();
const robot = new Robot();

human.work();  // Human working...
robot.work();  // Robot working...
```

### Dependency Inversion Principle (DIP)
```javascript
// ❌ BAD: High-level module depends on low-level modules
class MySQLDatabase {
  save(data) {
    console.log(`Saving to MySQL: ${JSON.stringify(data)}`);
  }
}

class UserService {
  constructor() {
    this.database = new MySQLDatabase(); // Tightly coupled
  }

  saveUser(userData) {
    this.database.save(userData);
  }
}

// Changing database requires modifying UserService
const service = new UserService();
service.saveUser({ name: 'John' });

// ✅ GOOD: Depend on abstractions, not concretions
class Database {
  save(data) {
    throw new Error('Must implement save');
  }
}

class MySQLDatabase extends Database {
  save(data) {
    console.log(`Saving to MySQL: ${JSON.stringify(data)}`);
  }
}

class MongoDatabase extends Database {
  save(data) {
    console.log(`Saving to MongoDB: ${JSON.stringify(data)}`);
  }
}

class UserService {
  constructor(database) {
    // Depends on abstraction (Database), not concrete implementation
    this.database = database;
  }

  saveUser(userData) {
    this.database.save(userData);
  }
}

// Can easily swap database implementations
const mysqlService = new UserService(new MySQLDatabase());
const mongoService = new UserService(new MongoDatabase());

mysqlService.saveUser({ name: 'John' });
mongoService.saveUser({ name: 'Jane' });
```

## Usage

- **When to use**: SOLID principles are essential when building scalable applications, especially in team environments where multiple developers maintain code. Apply them when designing class hierarchies, creating service layers, building frameworks, or refactoring legacy code.
- **Real-world example**: A large e-commerce platform benefits from SOLID by keeping payment processing, inventory management, and order fulfillment as separate, testable concerns. When PayPal changes their API, only the PayPal payment module changes. New payment methods can be added without touching existing code.
- **Best practices**:
  - Start with SRP as the foundation - ensure each class has one clear responsibility
  - Use composition over inheritance to maintain LSP
  - Create specific interfaces rather than broad ones (ISP)
  - Always inject dependencies rather than instantiating them internally (DIP)
  - Refactor gradually - don't try to apply all principles at once
  - Use automated tests to verify substitutability and behavior contracts

## FAQ / Interview Questions

**Q: What are the five SOLID principles and why should we care about them?**
A: SOLID is an acronym for Single Responsibility, Open/Closed, Liskov Substitution, Interface Segregation, and Dependency Inversion principles. These principles help developers write code that is easier to maintain, test, extend, and refactor. They reduce coupling between modules, minimize the impact of changes, and create more flexible architecture that adapts to evolving requirements without widespread code modifications.

**Q: Explain the Single Responsibility Principle with a real-world example.**
A: SRP states that a class should have only one reason to change. For example, a User class should handle user data properties, but email notifications, database persistence, and logging should be delegated to separate classes (EmailService, UserRepository, Logger). If you need to change how emails are sent, only the EmailService changes, not the User class. This isolation makes code easier to test and modify.

**Q: How does the Open/Closed Principle relate to the Dependency Inversion Principle?**
A: Both principles encourage us to depend on abstractions rather than concrete implementations. OCP says we should extend functionality by creating new implementations of abstract interfaces, not by modifying existing code. DIP reinforces this by stating that high-level modules should depend on abstractions that define contracts, allowing low-level modules to be swapped without affecting the high-level code. Together, they create extensible systems that remain closed for modification.

**Q: What is the Liskov Substitution Principle and when might it be violated?**
A: LSP states that derived classes must be safely substitutable for their base classes without breaking the application. It's violated when a subclass changes the contract or behavior expected from its parent. For example, if a Square extends Rectangle but forces width to equal height, code expecting a Rectangle would break. The solution is to use proper inheritance hierarchies where Square and Rectangle both extend Shape, each implementing getArea() according to their own logic.

**Q: How would you refactor a class that violates Interface Segregation Principle?**
A: If a class implements an interface with methods it doesn't need, split the interface into smaller, more specific ones. For example, instead of having a single Worker interface with work(), eat(), and sleep() methods, create separate Workable, Eatable, and Sleepable interfaces. Classes then implement only the interfaces relevant to them. This prevents classes from being forced to implement unnecessary methods and makes code clearer about what responsibilities each component has.

## References
- [SOLID Principles by Robert C. Martin](https://en.wikipedia.org/wiki/SOLID)
- [SOLID Design Principles - Stack Overflow](https://stackoverflow.com/questions/tagged/solid-principles)
- [Microsoft Architecture Guide - SOLID Principles](https://docs.microsoft.com/en-us/dotnet/architecture/modern-web-apps-azure/architectural-principles)
- [Clean Code: A Handbook of Agile Software Craftsmanship by Robert C. Martin](https://www.oreilly.com/library/view/clean-code-a/9780136083238/)

---
*See also: [Design Patterns](DesignPatterns.md), [Architecture Best Practices](ArchitectureBestPractices.md)*
