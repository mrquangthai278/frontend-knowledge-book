# Module Pattern

## Definition / Concept
The **Module Pattern** is a design pattern that encapsulates private and public members using closures, providing a way to create **self-contained** modules with private state and methods. It uses an **IIFE** (Immediately Invoked Function Expression) to create a private scope, exposing only the public API. This pattern enables **data privacy**, **namespace management**, and **code organization** in JavaScript.

- Encapsulates private data using **closures**
- Exposes only selected public methods and properties
- Prevents global namespace pollution
- Provides **information hiding** and **data privacy**

## Visual Representation
```
Module Pattern Structure:

┌─────────────────────────────────────┐
│   IIFE (Immediately Invoked)        │
│  ┌───────────────────────────────┐  │
│  │  Private Scope (Closure)      │  │
│  │  • privateVar                 │  │
│  │  • privateMethod()            │  │
│  │                               │  │
│  │  Return Public API:           │  │
│  │  {                            │  │
│  │    publicMethod: fn,          │  │
│  │    publicProp: value          │  │
│  │  }                            │  │
│  └───────────────────────────────┘  │
└─────────────────────────────────────┘
         ↓
   Only public API accessible
   Private members hidden
```

## Example
```javascript
// Basic Module Pattern
const Calculator = (function() {
  // Private variables
  let result = 0;
  
  // Private method
  function logOperation(operation, value) {
    console.log(`${operation}: ${value}, Result: ${result}`);
  }
  
  // Public API
  return {
    add: function(value) {
      result += value;
      logOperation('Add', value);
      return this;
    },
    
    subtract: function(value) {
      result -= value;
      logOperation('Subtract', value);
      return this;
    },
    
    getResult: function() {
      return result;
    },
    
    reset: function() {
      result = 0;
      return this;
    }
  };
})();

// Usage
Calculator.add(10).add(5).subtract(3);
console.log(Calculator.getResult()); // 12
console.log(Calculator.result); // undefined (private)

// Module with configuration
const UserModule = (function() {
  // Private data
  const users = [];
  let nextId = 1;
  
  // Private methods
  function validateUser(user) {
    return user.name && user.email;
  }
  
  function findById(id) {
    return users.find(u => u.id === id);
  }
  
  // Public API
  return {
    create: function(name, email) {
      const user = { id: nextId++, name, email };
      
      if (!validateUser(user)) {
        throw new Error('Invalid user data');
      }
      
      users.push(user);
      return user;
    },
    
    getAll: function() {
      return [...users]; // Return copy, not reference
    },
    
    getById: function(id) {
      const user = findById(id);
      return user ? { ...user } : null;
    },
    
    update: function(id, data) {
      const user = findById(id);
      if (!user) return false;
      
      Object.assign(user, data);
      return true;
    },
    
    delete: function(id) {
      const index = users.findIndex(u => u.id === id);
      if (index === -1) return false;
      
      users.splice(index, 1);
      return true;
    },
    
    count: function() {
      return users.length;
    }
  };
})();

// Usage
const user1 = UserModule.create('John', 'john@example.com');
const user2 = UserModule.create('Jane', 'jane@example.com');
console.log(UserModule.count()); // 2

// Revealing Module Pattern (variation)
const CounterModule = (function() {
  // Private
  let count = 0;
  
  function increment() {
    count++;
  }
  
  function decrement() {
    count--;
  }
  
  function getCount() {
    return count;
  }
  
  function reset() {
    count = 0;
  }
  
  // Reveal public methods
  return {
    increment,
    decrement,
    getCount,
    reset
  };
})();

// Module with dependencies
const AppModule = (function($, _) {
  // Private
  const config = {
    apiUrl: 'https://api.example.com',
    timeout: 5000
  };
  
  function fetchData(endpoint) {
    return $.ajax({
      url: config.apiUrl + endpoint,
      timeout: config.timeout
    });
  }
  
  // Public
  return {
    init: function(options) {
      _.extend(config, options);
    },
    
    loadUsers: function() {
      return fetchData('/users');
    },
    
    loadPosts: function() {
      return fetchData('/posts');
    }
  };
})(jQuery, _); // Inject dependencies

// ES6 Module (modern approach)
// userModule.js
const users = [];
let nextId = 1;

function validateUser(user) {
  return user.name && user.email;
}

export function createUser(name, email) {
  const user = { id: nextId++, name, email };
  if (!validateUser(user)) {
    throw new Error('Invalid user data');
  }
  users.push(user);
  return user;
}

export function getAllUsers() {
  return [...users];
}

export function getUserById(id) {
  const user = users.find(u => u.id === id);
  return user ? { ...user } : null;
}

// Import
// import { createUser, getAllUsers } from './userModule.js';
```

## Usage
- **When to use**: 
  - Need data privacy and encapsulation
  - Organizing code into logical units
  - Preventing global namespace pollution
  - Creating libraries and utilities
  - Legacy projects (pre-ES6 modules)
  
- **Real-world example**:
  ```javascript
  // API Client Module
  const ApiClient = (function() {
    const baseURL = 'https://api.example.com';
    const cache = new Map();
    
    async function request(endpoint, options = {}) {
      const url = baseURL + endpoint;
      const cacheKey = url + JSON.stringify(options);
      
      if (cache.has(cacheKey)) {
        return cache.get(cacheKey);
      }
      
      const response = await fetch(url, options);
      const data = await response.json();
      cache.set(cacheKey, data);
      
      return data;
    }
    
    return {
      get: (endpoint) => request(endpoint, { method: 'GET' }),
      post: (endpoint, body) => request(endpoint, { 
        method: 'POST', 
        body: JSON.stringify(body) 
      }),
      clearCache: () => cache.clear()
    };
  })();
  
  // Shopping Cart Module
  const ShoppingCart = (function() {
    let items = [];
    
    function calculateTotal() {
      return items.reduce((sum, item) => sum + item.price * item.quantity, 0);
    }
    
    function findItem(productId) {
      return items.find(item => item.productId === productId);
    }
    
    return {
      addItem: function(product, quantity = 1) {
        const existing = findItem(product.id);
        
        if (existing) {
          existing.quantity += quantity;
        } else {
          items.push({
            productId: product.id,
            name: product.name,
            price: product.price,
            quantity
          });
        }
      },
      
      removeItem: function(productId) {
        items = items.filter(item => item.productId !== productId);
      },
      
      updateQuantity: function(productId, quantity) {
        const item = findItem(productId);
        if (item) item.quantity = quantity;
      },
      
      getItems: function() {
        return [...items];
      },
      
      getTotal: function() {
        return calculateTotal();
      },
      
      clear: function() {
        items = [];
      }
    };
  })();
  
  // Logger Module
  const Logger = (function() {
    const logs = [];
    const maxLogs = 100;
    
    function addLog(level, message) {
      logs.push({
        level,
        message,
        timestamp: new Date()
      });
      
      if (logs.length > maxLogs) {
        logs.shift();
      }
    }
    
    return {
      info: (msg) => addLog('INFO', msg),
      warn: (msg) => addLog('WARN', msg),
      error: (msg) => addLog('ERROR', msg),
      getLogs: () => [...logs],
      clear: () => logs.length = 0
    };
  })();
  ```

- **Best practices**:
  - Use Revealing Module Pattern for cleaner code
  - Return copies of objects/arrays to prevent mutation
  - Use ES6 modules in modern applications
  - Keep modules focused on single responsibility
  - Document public API clearly
  - Consider using namespacing for related modules
  - Inject dependencies rather than using globals

## FAQ / Interview Questions

**Q: What is the Module Pattern and what problem does it solve?**
A: The **Module Pattern** is a design pattern that uses closures to create private scope and expose only selected public methods. It solves:
- **Privacy**: Encapsulates private data and methods
- **Namespace pollution**: Avoids cluttering global scope
- **Organization**: Groups related functionality together
- **Information hiding**: Exposes only necessary API

```javascript
const Module = (function() {
  let privateVar = 'secret'; // Private
  
  return {
    publicMethod: function() { // Public
      return privateVar;
    }
  };
})();
```

**Q: What's the difference between Module Pattern and Revealing Module Pattern?**
A:
**Module Pattern**: Defines public methods inline in the return object
```javascript
const Module = (function() {
  let count = 0;
  
  return {
    increment: function() { count++; },
    getCount: function() { return count; }
  };
})();
```

**Revealing Module Pattern**: Defines all functions first, then reveals selected ones
```javascript
const Module = (function() {
  let count = 0;
  
  function increment() { count++; }
  function getCount() { return count; }
  
  return { increment, getCount }; // Reveal
})();
```

Revealing pattern is cleaner and makes it clear what's public.

**Q: How does the Module Pattern achieve privacy?**
A: It uses **closures** created by an IIFE:
1. IIFE creates a private scope
2. Variables/functions inside are not accessible outside
3. Returned object has access via closure
4. Only returned members are public

```javascript
const Module = (function() {
  let secret = 'private'; // Closure variable
  
  return {
    getSecret: function() {
      return secret; // Has access via closure
    }
  };
})();

console.log(Module.secret); // undefined
console.log(Module.getSecret()); // 'private'
```

**Q: What are the advantages and disadvantages of the Module Pattern?**
A:
**Advantages**:
- Data privacy and encapsulation
- Prevents namespace pollution
- Better code organization
- Easy to maintain and test public API

**Disadvantages**:
- Cannot easily extend or modify private members
- Difficult to unit test private methods
- Memory overhead (all instances share same methods)
- Less flexible than classes
- ES6 modules are now preferred

**Q: How would you migrate from Module Pattern to ES6 modules?**
A:
```javascript
// Module Pattern (old)
const UserModule = (function() {
  const users = [];
  
  function add(user) {
    users.push(user);
  }
  
  function getAll() {
    return [...users];
  }
  
  return { add, getAll };
})();

// ES6 Module (modern)
// userModule.js
const users = [];

export function add(user) {
  users.push(user);
}

export function getAll() {
  return [...users];
}

// Usage
// import { add, getAll } from './userModule.js';
```

Benefits: Better tooling support, static analysis, tree-shaking, standard syntax.

## References
- [MDN Web Docs - Closures](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Closures)
- [JavaScript Design Patterns](https://addyosmani.com/resources/essentialjsdesignpatterns/book/)
- [Learning JavaScript Design Patterns - Module Pattern](https://www.patterns.dev/posts/module-pattern/)
- [Todd Motto - Mastering the Module Pattern](https://ultimatecourses.com/blog/mastering-the-module-pattern)

---
*See also: [Closure](Closure.md), [Factory Pattern](FactoryPattern.md), [Singleton](Singleton.md)*
