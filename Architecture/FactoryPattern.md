# Factory Pattern

## Definition / Concept
The **Factory Pattern** is a creational design pattern that provides an interface for creating objects without specifying their exact class. Instead of using `new` directly, a factory function creates and returns objects based on input parameters. This pattern promotes **loose coupling**, enables **polymorphism**, and simplifies object creation when the creation logic is complex.

- Creates objects without exposing creation logic
- Returns different object types based on input
- Promotes **DRY principle** - centralizes object creation
- Enables easy object creation without `new` keyword

## Visual Representation
```
Factory Pattern Flow:

Client Code
     ↓
  Factory Function (createUser, createShape, etc.)
     ↓
  Decision Logic (if/switch based on type)
     ↓
  ┌─────┬─────┬─────┐
  ↓     ↓     ↓     ↓
Admin  User  Guest  ...
Object Object Object

Instead of:
const admin = new Admin();  ← Direct instantiation
const user = new User();

Use Factory:
const admin = createUser('admin');  ← Abstracted creation
const user = createUser('regular');
```

## Example
```javascript
// Basic Factory Pattern
function createUser(name, role) {
  return {
    name,
    role,
    permissions: role === 'admin' ? ['read', 'write', 'delete'] : ['read'],
    
    hasPermission(permission) {
      return this.permissions.includes(permission);
    }
  };
}

// Usage
const admin = createUser('John', 'admin');
const guest = createUser('Jane', 'guest');

console.log(admin.hasPermission('delete')); // true
console.log(guest.hasPermission('delete')); // false

// Factory with different object types
function createShape(type, dimensions) {
  switch (type) {
    case 'circle':
      return {
        type: 'circle',
        radius: dimensions.radius,
        area() {
          return Math.PI * this.radius ** 2;
        },
        perimeter() {
          return 2 * Math.PI * this.radius;
        }
      };
      
    case 'rectangle':
      return {
        type: 'rectangle',
        width: dimensions.width,
        height: dimensions.height,
        area() {
          return this.width * this.height;
        },
        perimeter() {
          return 2 * (this.width + this.height);
        }
      };
      
    case 'triangle':
      return {
        type: 'triangle',
        base: dimensions.base,
        height: dimensions.height,
        area() {
          return 0.5 * this.base * this.height;
        }
      };
      
    default:
      throw new Error(`Unknown shape type: ${type}`);
  }
}

// Usage
const circle = createShape('circle', { radius: 5 });
const rectangle = createShape('rectangle', { width: 10, height: 5 });

console.log(circle.area()); // 78.54
console.log(rectangle.area()); // 50

// Factory with classes (Class Factory)
class Dog {
  constructor(name) {
    this.name = name;
  }
  
  speak() {
    return `${this.name} barks!`;
  }
}

class Cat {
  constructor(name) {
    this.name = name;
  }
  
  speak() {
    return `${this.name} meows!`;
  }
}

class Bird {
  constructor(name) {
    this.name = name;
  }
  
  speak() {
    return `${this.name} chirps!`;
  }
}

// Animal Factory
function createAnimal(type, name) {
  const animals = {
    dog: Dog,
    cat: Cat,
    bird: Bird
  };
  
  const AnimalClass = animals[type];
  
  if (!AnimalClass) {
    throw new Error(`Unknown animal type: ${type}`);
  }
  
  return new AnimalClass(name);
}

// Usage
const dog = createAnimal('dog', 'Rex');
const cat = createAnimal('cat', 'Whiskers');

console.log(dog.speak()); // Rex barks!
console.log(cat.speak()); // Whiskers meows!

// Factory with validation and defaults
function createProduct(config) {
  // Default values
  const defaults = {
    name: 'Unnamed Product',
    price: 0,
    category: 'general',
    inStock: true
  };
  
  // Merge with defaults
  const product = { ...defaults, ...config };
  
  // Validation
  if (product.price < 0) {
    throw new Error('Price cannot be negative');
  }
  
  // Add methods
  product.getDisplayPrice = function() {
    return `$${this.price.toFixed(2)}`;
  };
  
  product.applyDiscount = function(percentage) {
    this.price *= (1 - percentage / 100);
    return this;
  };
  
  return product;
}

// Usage
const laptop = createProduct({
  name: 'Laptop',
  price: 999,
  category: 'electronics'
});

console.log(laptop.getDisplayPrice()); // $999.00
laptop.applyDiscount(10);
console.log(laptop.getDisplayPrice()); // $899.10

// Factory for HTTP clients
function createHttpClient(baseURL, options = {}) {
  const defaultOptions = {
    timeout: 5000,
    headers: {
      'Content-Type': 'application/json'
    }
  };
  
  const config = { ...defaultOptions, ...options };
  
  return {
    async get(endpoint) {
      const response = await fetch(baseURL + endpoint, {
        method: 'GET',
        headers: config.headers,
        signal: AbortSignal.timeout(config.timeout)
      });
      return response.json();
    },
    
    async post(endpoint, data) {
      const response = await fetch(baseURL + endpoint, {
        method: 'POST',
        headers: config.headers,
        body: JSON.stringify(data),
        signal: AbortSignal.timeout(config.timeout)
      });
      return response.json();
    },
    
    setHeader(key, value) {
      config.headers[key] = value;
    }
  };
}

// Usage
const apiClient = createHttpClient('https://api.example.com', {
  timeout: 10000
});

apiClient.setHeader('Authorization', 'Bearer token');
const users = await apiClient.get('/users');
```

## Usage
- **When to use**: 
  - Object creation is complex or requires configuration
  - Need to create different types of objects based on input
  - Want to decouple object creation from usage
  - Need centralized object creation logic
  - Creating similar objects with variations
  
- **Real-world example**:
  ```javascript
  // Notification Factory
  function createNotification(type, message, options = {}) {
    const base = {
      id: Date.now(),
      message,
      timestamp: new Date(),
      read: false,
      ...options
    };
    
    switch (type) {
      case 'success':
        return {
          ...base,
          type: 'success',
          icon: '✓',
          color: 'green',
          duration: 3000
        };
        
      case 'error':
        return {
          ...base,
          type: 'error',
          icon: '✗',
          color: 'red',
          duration: 5000,
          dismissible: true
        };
        
      case 'warning':
        return {
          ...base,
          type: 'warning',
          icon: '⚠',
          color: 'orange',
          duration: 4000
        };
        
      case 'info':
        return {
          ...base,
          type: 'info',
          icon: 'ℹ',
          color: 'blue',
          duration: 3000
        };
        
      default:
        throw new Error(`Unknown notification type: ${type}`);
    }
  }
  
  // Usage
  const successNotif = createNotification('success', 'Profile updated!');
  const errorNotif = createNotification('error', 'Failed to save');
  
  // Button Factory
  function createButton(variant, text, onClick) {
    const base = {
      text,
      onClick,
      disabled: false,
      className: 'btn'
    };
    
    const variants = {
      primary: { ...base, className: 'btn btn-primary', color: 'blue' },
      secondary: { ...base, className: 'btn btn-secondary', color: 'gray' },
      danger: { ...base, className: 'btn btn-danger', color: 'red' },
      success: { ...base, className: 'btn btn-success', color: 'green' }
    };
    
    return variants[variant] || base;
  }
  
  // Form Field Factory
  function createFormField(type, name, config = {}) {
    const baseField = {
      name,
      value: '',
      error: null,
      touched: false,
      
      validate() {
        if (this.required && !this.value) {
          this.error = 'This field is required';
          return false;
        }
        this.error = null;
        return true;
      },
      
      ...config
    };
    
    switch (type) {
      case 'email':
        return {
          ...baseField,
          type: 'email',
          validate() {
            if (this.required && !this.value) {
              this.error = 'Email is required';
              return false;
            }
            const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
            if (this.value && !emailRegex.test(this.value)) {
              this.error = 'Invalid email format';
              return false;
            }
            this.error = null;
            return true;
          }
        };
        
      case 'password':
        return {
          ...baseField,
          type: 'password',
          minLength: 8,
          validate() {
            if (this.required && !this.value) {
              this.error = 'Password is required';
              return false;
            }
            if (this.value.length < this.minLength) {
              this.error = `Password must be at least ${this.minLength} characters`;
              return false;
            }
            this.error = null;
            return true;
          }
        };
        
      default:
        return { ...baseField, type };
    }
  }
  
  // Usage
  const emailField = createFormField('email', 'userEmail', { required: true });
  const passwordField = createFormField('password', 'userPassword', { 
    required: true, 
    minLength: 10 
  });
  ```

- **Best practices**:
  - Use factories for objects with complex creation logic
  - Provide sensible defaults
  - Include validation in factory
  - Make factory functions pure when possible
  - Use descriptive factory names (createX, makeX, buildX)
  - Return frozen objects if immutability is needed
  - Document what types of objects the factory can create

## FAQ / Interview Questions

**Q: What is the Factory Pattern and when should you use it?**
A: The **Factory Pattern** is a creational pattern that creates objects without exposing creation logic. Use it when:
- Object creation is complex
- Need to create different types based on input
- Want to centralize creation logic
- Need to decouple creation from usage

```javascript
// Instead of this:
const user = new User(name, email);
const admin = new Admin(name, email, permissions);

// Use factory:
const user = createUser('regular', { name, email });
const admin = createUser('admin', { name, email, permissions });
```

**Q: What's the difference between Factory Pattern and Constructor?**
A:
**Constructor**:
- Uses `new` keyword
- Tightly coupled to specific class
- Less flexible for variations

**Factory**:
- Just a function call
- Can return different types
- More flexible and testable

```javascript
// Constructor
class User {
  constructor(name) {
    this.name = name;
  }
}
const user = new User('John');

// Factory
function createUser(name) {
  return { name };
}
const user = createUser('John');
```

**Q: How does Factory Pattern promote loose coupling?**
A: Factory decouples object creation from usage:

```javascript
// Tight coupling - client knows exact class
const shape = new Rectangle(10, 5);

// Loose coupling - client only knows factory
const shape = createShape('rectangle', { width: 10, height: 5 });

// Now changing shape creation doesn't affect client code
// Can add validation, caching, pooling, etc. in factory
function createShape(type, dimensions) {
  // Can add logging, caching, validation
  // Client code doesn't need to change
  return type === 'rectangle' 
    ? { ...dimensions, area: () => dimensions.width * dimensions.height }
    : { ...dimensions };
}
```

**Q: Can you show an example of Factory Pattern with dependency injection?**
A:
```javascript
// Factory with dependencies
function createUserService(database, logger) {
  return {
    async createUser(userData) {
      logger.info('Creating user:', userData);
      
      const user = await database.insert('users', userData);
      
      logger.info('User created:', user.id);
      return user;
    },
    
    async getUser(id) {
      logger.info('Fetching user:', id);
      return database.findById('users', id);
    }
  };
}

// Usage with different dependencies
const prodService = createUserService(pgDatabase, prodLogger);
const testService = createUserService(mockDatabase, mockLogger);
```

**Q: What are the advantages and disadvantages of Factory Pattern?**
A:
**Advantages**:
- **Flexibility**: Easy to add new types
- **Encapsulation**: Hides creation complexity
- **DRY**: Centralized creation logic
- **Testability**: Easy to mock/stub
- **No `new` required**: Simpler syntax

**Disadvantages**:
- Can become complex with many types
- May be overkill for simple objects
- Need to maintain factory as types change
- Less obvious than direct instantiation

## References
- [Refactoring Guru - Factory Pattern](https://refactoring.guru/design-patterns/factory-method)
- [JavaScript Design Patterns - Factory](https://www.patterns.dev/posts/factory-pattern/)
- [MDN Web Docs - Object creation](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Object_initializer)
- [Learning JavaScript Design Patterns](https://addyosmani.com/resources/essentialjsdesignpatterns/book/)

---
*See also: [Module Pattern](ModulePattern.md), [Singleton](Singleton.md), [Builder Pattern](../JavaScript/)*
