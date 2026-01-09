# Singleton Pattern

## Definition / Concept
The **Singleton Pattern** is a creational design pattern that ensures a class has only **one instance** and provides a global point of access to it. The pattern restricts instantiation to a single object, making it useful for managing shared resources like database connections, configuration objects, or caches. In JavaScript, singletons can be implemented using closures, modules, or ES6 classes.

- Ensures only **one instance** exists globally
- Provides **global access point** to that instance
- Lazy initialization - created only when needed
- Useful for shared resources (config, cache, connection pools)

## Visual Representation
```
Singleton Pattern Flow:

First Call:
Client → getInstance() → Create Instance → Return Instance
                              ↓
                         Store in closure/static

Subsequent Calls:
Client → getInstance() → Check if exists → Return Same Instance
                              ↓
                         (No new creation)

Multiple Clients:
Client A ──┐
           ├──→ getInstance() → Same Instance
Client B ──┘

Regular Pattern (No Singleton):
Client A → new Class() → Instance A
Client B → new Class() → Instance B (different)
```

## Example
```javascript
// Basic Singleton using IIFE and closure
const Singleton = (function() {
  let instance;
  
  function createInstance() {
    return {
      name: 'Singleton Instance',
      random: Math.random(),
      
      doSomething() {
        console.log('Doing something...');
      }
    };
  }
  
  return {
    getInstance: function() {
      if (!instance) {
        instance = createInstance();
      }
      return instance;
    }
  };
})();

// Usage
const instance1 = Singleton.getInstance();
const instance2 = Singleton.getInstance();

console.log(instance1 === instance2); // true (same instance)
console.log(instance1.random === instance2.random); // true

// ES6 Class Singleton
class Database {
  constructor() {
    if (Database.instance) {
      return Database.instance;
    }
    
    this.connection = null;
    this.connected = false;
    
    Database.instance = this;
  }
  
  connect() {
    if (!this.connected) {
      this.connection = 'Connected to DB';
      this.connected = true;
      console.log('Database connected');
    }
    return this.connection;
  }
  
  query(sql) {
    if (!this.connected) {
      throw new Error('Not connected to database');
    }
    console.log(`Executing: ${sql}`);
    return [];
  }
  
  disconnect() {
    this.connection = null;
    this.connected = false;
    console.log('Database disconnected');
  }
}

// Usage
const db1 = new Database();
const db2 = new Database();

console.log(db1 === db2); // true
db1.connect();
db2.query('SELECT * FROM users'); // Works (same connection)

// Singleton with static method (cleaner)
class Config {
  static instance = null;
  
  constructor() {
    if (Config.instance) {
      return Config.instance;
    }
    
    this.settings = {
      apiUrl: 'https://api.example.com',
      timeout: 5000,
      debug: false
    };
    
    Config.instance = this;
  }
  
  static getInstance() {
    if (!Config.instance) {
      Config.instance = new Config();
    }
    return Config.instance;
  }
  
  get(key) {
    return this.settings[key];
  }
  
  set(key, value) {
    this.settings[key] = value;
  }
}

// Usage
const config1 = Config.getInstance();
const config2 = Config.getInstance();

config1.set('debug', true);
console.log(config2.get('debug')); // true (same instance)

// Singleton for Cache
class Cache {
  static instance = null;
  
  constructor() {
    if (Cache.instance) {
      return Cache.instance;
    }
    
    this.cache = new Map();
    Cache.instance = this;
  }
  
  static getInstance() {
    if (!Cache.instance) {
      Cache.instance = new Cache();
    }
    return Cache.instance;
  }
  
  set(key, value, ttl = 60000) {
    const expiry = Date.now() + ttl;
    this.cache.set(key, { value, expiry });
  }
  
  get(key) {
    const item = this.cache.get(key);
    
    if (!item) return null;
    
    if (Date.now() > item.expiry) {
      this.cache.delete(key);
      return null;
    }
    
    return item.value;
  }
  
  has(key) {
    return this.get(key) !== null;
  }
  
  clear() {
    this.cache.clear();
  }
  
  size() {
    return this.cache.size;
  }
}

// Usage
const cache1 = Cache.getInstance();
const cache2 = Cache.getInstance();

cache1.set('user', { id: 1, name: 'John' }, 10000);
console.log(cache2.get('user')); // { id: 1, name: 'John' }

// Module Singleton (ES6 modules are singletons by default)
// logger.js
class Logger {
  constructor() {
    this.logs = [];
  }
  
  log(message, level = 'INFO') {
    const entry = {
      message,
      level,
      timestamp: new Date()
    };
    this.logs.push(entry);
    console.log(`[${level}] ${message}`);
  }
  
  info(message) {
    this.log(message, 'INFO');
  }
  
  error(message) {
    this.log(message, 'ERROR');
  }
  
  warn(message) {
    this.log(message, 'WARN');
  }
  
  getLogs() {
    return [...this.logs];
  }
  
  clear() {
    this.logs = [];
  }
}

// Export single instance
export default new Logger();

// Usage in other files
// import logger from './logger.js';
// logger.info('Application started');
// logger.error('Something went wrong');

// Singleton with lazy initialization
class ExpensiveResource {
  static instance = null;
  
  constructor() {
    if (ExpensiveResource.instance) {
      return ExpensiveResource.instance;
    }
    
    console.log('Creating expensive resource...');
    this.data = this.loadExpensiveData();
    
    ExpensiveResource.instance = this;
  }
  
  loadExpensiveData() {
    // Simulate expensive operation
    return { loaded: true, timestamp: Date.now() };
  }
  
  static getInstance() {
    if (!ExpensiveResource.instance) {
      ExpensiveResource.instance = new ExpensiveResource();
    }
    return ExpensiveResource.instance;
  }
}

// Resource is created only when first accessed
const resource1 = ExpensiveResource.getInstance(); // "Creating expensive resource..."
const resource2 = ExpensiveResource.getInstance(); // No log (reuses instance)
```

## Usage
- **When to use**: 
  - Managing shared resources (database, cache, configuration)
  - Coordinating actions across the system
  - Logging services
  - Thread pools or connection pools
  - Managing global state (use cautiously)
  
- **Real-world example**:
  ```javascript
  // API Client Singleton
  class ApiClient {
    static instance = null;
    
    constructor() {
      if (ApiClient.instance) {
        return ApiClient.instance;
      }
      
      this.baseURL = process.env.API_URL;
      this.token = null;
      this.requestQueue = [];
      
      ApiClient.instance = this;
    }
    
    static getInstance() {
      if (!ApiClient.instance) {
        ApiClient.instance = new ApiClient();
      }
      return ApiClient.instance;
    }
    
    setToken(token) {
      this.token = token;
    }
    
    async request(endpoint, options = {}) {
      const headers = {
        'Content-Type': 'application/json',
        ...(this.token && { Authorization: `Bearer ${this.token}` }),
        ...options.headers
      };
      
      const response = await fetch(this.baseURL + endpoint, {
        ...options,
        headers
      });
      
      return response.json();
    }
    
    get(endpoint) {
      return this.request(endpoint, { method: 'GET' });
    }
    
    post(endpoint, data) {
      return this.request(endpoint, {
        method: 'POST',
        body: JSON.stringify(data)
      });
    }
  }
  
  // Usage across application
  // fileA.js
  const api = ApiClient.getInstance();
  api.setToken('abc123');
  
  // fileB.js
  const api = ApiClient.getInstance(); // Same instance, has token
  const users = await api.get('/users');
  
  // Application State Manager
  class StateManager {
    static instance = null;
    
    constructor() {
      if (StateManager.instance) {
        return StateManager.instance;
      }
      
      this.state = {};
      this.listeners = new Map();
      
      StateManager.instance = this;
    }
    
    static getInstance() {
      if (!StateManager.instance) {
        StateManager.instance = new StateManager();
      }
      return StateManager.instance;
    }
    
    setState(key, value) {
      this.state[key] = value;
      this.notify(key, value);
    }
    
    getState(key) {
      return this.state[key];
    }
    
    subscribe(key, callback) {
      if (!this.listeners.has(key)) {
        this.listeners.set(key, []);
      }
      this.listeners.get(key).push(callback);
      
      // Return unsubscribe function
      return () => {
        const callbacks = this.listeners.get(key);
        const index = callbacks.indexOf(callback);
        if (index > -1) {
          callbacks.splice(index, 1);
        }
      };
    }
    
    notify(key, value) {
      const callbacks = this.listeners.get(key) || [];
      callbacks.forEach(callback => callback(value));
    }
  }
  
  // Usage
  const state = StateManager.getInstance();
  
  state.subscribe('user', (user) => {
    console.log('User updated:', user);
  });
  
  state.setState('user', { id: 1, name: 'John' });
  ```

- **Best practices**:
  - Consider if singleton is really needed (often indicates design issue)
  - Use ES6 modules for natural singletons
  - Make singletons thread-safe if needed
  - Avoid using singletons for mutable global state
  - Consider dependency injection instead
  - Document that class is a singleton
  - Make constructor private when possible (TypeScript)
  - Be careful with testing (singletons can cause test pollution)

## FAQ / Interview Questions

**Q: What is the Singleton Pattern and when should you use it?**
A: The **Singleton Pattern** ensures a class has only one instance globally. Use it for:
- Shared resources: database connections, cache, config
- Coordinating system-wide actions
- Managing global state (cautiously)

```javascript
class Singleton {
  static instance = null;
  
  static getInstance() {
    if (!Singleton.instance) {
      Singleton.instance = new Singleton();
    }
    return Singleton.instance;
  }
}

const a = Singleton.getInstance();
const b = Singleton.getInstance();
console.log(a === b); // true
```

**Q: What are the advantages and disadvantages of Singleton?**
A:
**Advantages**:
- Controlled access to single instance
- Reduced namespace pollution
- Lazy initialization
- Easy to maintain shared state

**Disadvantages**:
- **Global state** (hard to test, debug)
- **Hidden dependencies** (not obvious what depends on it)
- **Tight coupling** (violates dependency injection)
- **Testing difficulties** (shared state between tests)
- **Concurrency issues** (if not thread-safe)

Many consider Singleton an **anti-pattern** in modern JavaScript.

**Q: How is a Singleton different from a static class?**
A:
**Singleton**:
- Has single instance
- Can implement interfaces
- Can be passed as parameter
- Can be lazy initialized

**Static Class** (all static methods):
- No instances at all
- Cannot implement interfaces
- Cannot be passed around
- Always loaded

```javascript
// Singleton
class Singleton {
  static instance = null;
  static getInstance() { /* ... */ }
  instanceMethod() { /* ... */ }
}

// Static class
class StaticClass {
  static method1() { /* ... */ }
  static method2() { /* ... */ }
}
```

**Q: Why are ES6 modules natural singletons?**
A: ES6 modules are evaluated only once and cached:

```javascript
// config.js
class Config {
  constructor() {
    this.settings = {};
  }
}

export default new Config(); // Single instance

// fileA.js
import config from './config.js';
config.settings.debug = true;

// fileB.js
import config from './config.js'; // Same instance!
console.log(config.settings.debug); // true
```

Each import gets the same cached instance - natural singleton without boilerplate.

**Q: How would you test code that uses singletons?**
A: Testing singletons is challenging. Solutions:

1. **Reset singleton between tests**:
```javascript
class Singleton {
  static reset() {
    Singleton.instance = null;
  }
}

afterEach(() => {
  Singleton.reset();
});
```

2. **Use dependency injection** (better):
```javascript
// Instead of this:
class UserService {
  constructor() {
    this.db = Database.getInstance(); // Hard to test
  }
}

// Do this:
class UserService {
  constructor(db) {
    this.db = db; // Inject dependency
  }
}

// Test with mock
const mockDb = { query: jest.fn() };
const service = new UserService(mockDb);
```

3. **Use factory/builder for tests**

## References
- [Refactoring Guru - Singleton Pattern](https://refactoring.guru/design-patterns/singleton)
- [JavaScript Design Patterns - Singleton](https://www.patterns.dev/posts/singleton-pattern/)
- [MDN Web Docs - ES6 Modules](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Modules)
- [Why Singletons are Controversial](https://stackoverflow.com/questions/137975/what-is-so-bad-about-singletons)

---
*See also: [Module Pattern](ModulePattern.md), [Factory Pattern](FactoryPattern.md), [Dependency Injection](../JavaScript/)*
