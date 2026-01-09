# Observer Pattern

## Definition / Concept
The **Observer Pattern** is a behavioral design pattern where an object (the **subject**) maintains a list of dependents (the **observers**) and automatically notifies them of any state changes. This pattern establishes a **one-to-many** relationship between objects, allowing multiple observers to listen to and react to events from a single subject. It's the foundation of event-driven programming and reactive systems.

- **Subject** maintains list of observers and notifies them
- **Observers** subscribe to subject and receive updates
- Implements **one-to-many** dependency between objects
- Promotes **loose coupling** between subject and observers

## Visual Representation
```
Observer Pattern Structure:

        Subject (Observable)
        ┌──────────────────┐
        │ - observers[]    │
        │ + subscribe()    │
        │ + unsubscribe()  │
        │ + notify()       │
        └────────┬─────────┘
                 │
        ┌────────┴────────┐
        │                 │
        ▼                 ▼
   Observer A        Observer B
   ┌────────┐        ┌────────┐
   │update()│        │update()│
   └────────┘        └────────┘

Flow:
1. Observers subscribe to Subject
2. Subject state changes
3. Subject notifies all observers
4. Observers react to notification

vs Pub/Sub:
Observer: Subject knows observers (direct reference)
Pub/Sub: Publishers/Subscribers don't know each other (Event Bus)
```

## Example
```javascript
// Basic Observer Pattern
class Subject {
  constructor() {
    this.observers = [];
  }
  
  subscribe(observer) {
    this.observers.push(observer);
  }
  
  unsubscribe(observer) {
    this.observers = this.observers.filter(obs => obs !== observer);
  }
  
  notify(data) {
    this.observers.forEach(observer => observer.update(data));
  }
}

class Observer {
  constructor(name) {
    this.name = name;
  }
  
  update(data) {
    console.log(`${this.name} received:`, data);
  }
}

// Usage
const subject = new Subject();

const observer1 = new Observer('Observer 1');
const observer2 = new Observer('Observer 2');

subject.subscribe(observer1);
subject.subscribe(observer2);

subject.notify('Hello Observers!');
// Observer 1 received: Hello Observers!
// Observer 2 received: Hello Observers!

subject.unsubscribe(observer1);
subject.notify('Second message');
// Observer 2 received: Second message

// Real-world example: Stock Price Monitor
class Stock {
  constructor(symbol, price) {
    this.symbol = symbol;
    this.price = price;
    this.observers = [];
  }
  
  subscribe(observer) {
    this.observers.push(observer);
    console.log(`${observer.name} subscribed to ${this.symbol}`);
  }
  
  unsubscribe(observer) {
    this.observers = this.observers.filter(obs => obs !== observer);
    console.log(`${observer.name} unsubscribed from ${this.symbol}`);
  }
  
  setPrice(newPrice) {
    const oldPrice = this.price;
    this.price = newPrice;
    
    this.notify({
      symbol: this.symbol,
      oldPrice,
      newPrice,
      change: newPrice - oldPrice,
      changePercent: ((newPrice - oldPrice) / oldPrice * 100).toFixed(2)
    });
  }
  
  notify(data) {
    this.observers.forEach(observer => observer.update(data));
  }
}

class Investor {
  constructor(name) {
    this.name = name;
  }
  
  update(data) {
    console.log(`${this.name} notified: ${data.symbol} changed from $${data.oldPrice} to $${data.newPrice} (${data.changePercent}%)`);
    
    if (data.change > 0) {
      console.log(`  → ${this.name}: Good news!`);
    } else {
      console.log(`  → ${this.name}: Time to sell?`);
    }
  }
}

// Usage
const apple = new Stock('AAPL', 150);

const investor1 = new Investor('John');
const investor2 = new Investor('Jane');

apple.subscribe(investor1);
apple.subscribe(investor2);

apple.setPrice(155);
// John notified: AAPL changed from $150 to $155 (3.33%)
//   → John: Good news!
// Jane notified: AAPL changed from $150 to $155 (3.33%)
//   → Jane: Good news!

apple.setPrice(148);
// Both investors notified about price drop

// Newsletter Subscription System
class Newsletter {
  constructor(name) {
    this.name = name;
    this.subscribers = [];
  }
  
  subscribe(subscriber) {
    if (!this.subscribers.includes(subscriber)) {
      this.subscribers.push(subscriber);
      subscriber.onSubscribe(this.name);
    }
  }
  
  unsubscribe(subscriber) {
    this.subscribers = this.subscribers.filter(sub => sub !== subscriber);
    subscriber.onUnsubscribe(this.name);
  }
  
  publish(article) {
    console.log(`\nPublishing article: "${article.title}"`);
    this.subscribers.forEach(subscriber => {
      subscriber.receive(this.name, article);
    });
  }
}

class Subscriber {
  constructor(name, email) {
    this.name = name;
    this.email = email;
  }
  
  receive(newsletter, article) {
    console.log(`📧 ${this.email}: New article from ${newsletter} - "${article.title}"`);
  }
  
  onSubscribe(newsletter) {
    console.log(`✓ ${this.name} subscribed to ${newsletter}`);
  }
  
  onUnsubscribe(newsletter) {
    console.log(`✗ ${this.name} unsubscribed from ${newsletter}`);
  }
}

// Usage
const techNews = new Newsletter('Tech News Daily');

const sub1 = new Subscriber('Alice', 'alice@example.com');
const sub2 = new Subscriber('Bob', 'bob@example.com');

techNews.subscribe(sub1);
techNews.subscribe(sub2);

techNews.publish({ title: 'New JavaScript Features in 2026' });

// Event-driven UI Component
class Button {
  constructor(id) {
    this.id = id;
    this.clickHandlers = [];
  }
  
  onClick(handler) {
    this.clickHandlers.push(handler);
  }
  
  offClick(handler) {
    this.clickHandlers = this.clickHandlers.filter(h => h !== handler);
  }
  
  click() {
    console.log(`Button ${this.id} clicked`);
    this.clickHandlers.forEach(handler => handler(this.id));
  }
}

// Usage
const submitButton = new Button('submit');

const logHandler = (id) => console.log(`Logged: Button ${id} clicked`);
const analyticsHandler = (id) => console.log(`Analytics: Track button ${id}`);
const validationHandler = (id) => console.log(`Validate form before submit`);

submitButton.onClick(logHandler);
submitButton.onClick(analyticsHandler);
submitButton.onClick(validationHandler);

submitButton.click();
// Button submit clicked
// Logged: Button submit clicked
// Analytics: Track button submit
// Validate form before submit

// Observable Data Store
class Store {
  constructor(initialState = {}) {
    this.state = initialState;
    this.observers = [];
  }
  
  getState() {
    return { ...this.state };
  }
  
  setState(newState) {
    const oldState = this.state;
    this.state = { ...this.state, ...newState };
    
    this.notify({
      oldState,
      newState: this.state,
      changes: newState
    });
  }
  
  subscribe(observer) {
    this.observers.push(observer);
    
    // Return unsubscribe function
    return () => {
      this.observers = this.observers.filter(obs => obs !== observer);
    };
  }
  
  notify(data) {
    this.observers.forEach(observer => observer(data));
  }
}

// Usage
const store = new Store({ count: 0, user: null });

const unsubscribe1 = store.subscribe((data) => {
  console.log('Observer 1 - State changed:', data.changes);
});

const unsubscribe2 = store.subscribe((data) => {
  console.log('Observer 2 - New state:', data.newState);
});

store.setState({ count: 1 });
// Observer 1 - State changed: { count: 1 }
// Observer 2 - New state: { count: 1, user: null }

store.setState({ user: { name: 'John' } });

unsubscribe1(); // Stop observing
store.setState({ count: 2 }); // Only Observer 2 notified
```

## Usage
- **When to use**: 
  - Event handling systems
  - Data binding (MVC/MVVM patterns)
  - Real-time notifications
  - Reactive programming
  - UI components responding to data changes
  
- **Real-world example**:
  ```javascript
  // React-like State Management
  class Component {
    constructor() {
      this.state = {};
      this.observers = [];
    }
    
    setState(newState) {
      this.state = { ...this.state, ...newState };
      this.forceUpdate();
    }
    
    forceUpdate() {
      this.observers.forEach(observer => observer());
    }
    
    subscribe(observer) {
      this.observers.push(observer);
      return () => {
        this.observers = this.observers.filter(o => o !== observer);
      };
    }
  }
  
  // Form Validation Observer
  class Form {
    constructor() {
      this.fields = {};
      this.validators = [];
      this.errors = {};
    }
    
    addValidator(validator) {
      this.validators.push(validator);
    }
    
    setField(name, value) {
      this.fields[name] = value;
      this.validate();
    }
    
    validate() {
      this.errors = {};
      
      this.validators.forEach(validator => {
        const error = validator.validate(this.fields);
        if (error) {
          this.errors = { ...this.errors, ...error };
        }
      });
      
      this.notifyValidators();
    }
    
    notifyValidators() {
      this.validators.forEach(validator => {
        validator.onValidate(this.errors);
      });
    }
  }
  
  class EmailValidator {
    validate(fields) {
      const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
      if (fields.email && !emailRegex.test(fields.email)) {
        return { email: 'Invalid email format' };
      }
      return null;
    }
    
    onValidate(errors) {
      if (errors.email) {
        console.log('Email error:', errors.email);
      }
    }
  }
  ```

- **Best practices**:
  - Return unsubscribe function from subscribe
  - Avoid memory leaks by unsubscribing when done
  - Use weak references if available
  - Consider using event names/types for filtering
  - Implement error handling in observers
  - Document what data observers will receive
  - Consider async notifications if needed
  - Prevent circular updates

## FAQ / Interview Questions

**Q: What is the Observer Pattern and how does it work?**
A: The **Observer Pattern** establishes a one-to-many dependency where a subject maintains observers and notifies them of changes:

```javascript
class Subject {
  constructor() {
    this.observers = [];
  }
  
  subscribe(observer) {
    this.observers.push(observer);
  }
  
  notify(data) {
    this.observers.forEach(obs => obs.update(data));
  }
}

// Subject knows its observers directly
```

Benefits: Loose coupling, dynamic relationships, broadcast communication.

**Q: What's the difference between Observer Pattern and Pub/Sub Pattern?**
A:
**Observer Pattern**:
- Subject knows observers directly
- Tighter coupling
- Synchronous by default
- Subject manages subscriptions

**Pub/Sub Pattern**:
- Publishers/subscribers don't know each other
- Event bus/broker in between
- Looser coupling
- More flexible, can be async

```javascript
// Observer: Direct reference
subject.subscribe(observer);

// Pub/Sub: Through event bus
eventBus.on('event', handler);
eventBus.emit('event', data);
```

**Q: How do you prevent memory leaks with observers?**
A: Key strategies:

1. **Always unsubscribe**:
```javascript
const unsubscribe = subject.subscribe(observer);
// Later...
unsubscribe();
```

2. **Cleanup in lifecycle methods**:
```javascript
class Component {
  componentDidMount() {
    this.unsubscribe = store.subscribe(this.handleUpdate);
  }
  
  componentWillUnmount() {
    this.unsubscribe();
  }
}
```

3. **Use WeakMap/WeakSet** for automatic cleanup
4. **Remove references** when objects are destroyed

**Q: How is the Observer Pattern used in modern frameworks?**
A:
- **React**: `useState`, `useEffect` (observer-like subscriptions)
- **Vue**: Reactive data system (observers on data properties)
- **RxJS**: Observable streams
- **Redux**: `store.subscribe(listener)`
- **MobX**: Observable state with automatic reactions

```javascript
// Redux example
const unsubscribe = store.subscribe(() => {
  console.log('State changed:', store.getState());
});

store.dispatch({ type: 'INCREMENT' }); // Notifies subscribers
```

**Q: What are the advantages and disadvantages of Observer Pattern?**
A:
**Advantages**:
- **Loose coupling**: Subject doesn't need to know observer details
- **Dynamic**: Add/remove observers at runtime
- **Broadcast**: One change notifies many observers
- **Reusable**: Observers can be reused with different subjects

**Disadvantages**:
- **Memory leaks**: Forgot to unsubscribe
- **Performance**: Many observers = many notifications
- **Unexpected updates**: Hard to track cause of changes
- **Order dependency**: Observer execution order matters sometimes
- **Debugging**: Complex notification chains

## References
- [Refactoring Guru - Observer Pattern](https://refactoring.guru/design-patterns/observer)
- [JavaScript Design Patterns - Observer](https://www.patterns.dev/posts/observer-pattern/)
- [MDN Web Docs - EventTarget](https://developer.mozilla.org/en-US/docs/Web/API/EventTarget)
- [Learning JavaScript Design Patterns](https://addyosmani.com/resources/essentialjsdesignpatterns/book/)

---
*See also: [Pub/Sub Pattern](PubSub.md), [Event Delegation](EventDelegation.md), [Event Loop](EventLoop.md)*
