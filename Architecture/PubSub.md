# Pub/Sub Pattern (Publish/Subscribe)

## Definition / Concept
The **Pub/Sub Pattern** (Publish/Subscribe) is a messaging pattern where senders (publishers) don't send messages directly to receivers (subscribers). Instead, messages are sent to an intermediary **event bus** or **message broker** that distributes them to interested subscribers based on event types. This pattern provides **complete decoupling** between publishers and subscribers, making it more flexible than the Observer pattern.

- **Publishers** emit events without knowing who receives them
- **Subscribers** listen to events without knowing who sends them
- **Event Bus** (broker) manages event distribution
- Provides **complete decoupling** and **loose coupling**

## Visual Representation
```
Pub/Sub Pattern Structure:

Publisher A ────┐
                 ├──→ Event Bus (Broker) ──→ Subscriber 1
Publisher B ────┘         ↓                   Subscriber 2
                     [Event Queue]            Subscriber 3

Publishers don't know Subscribers
Subscribers don't know Publishers
Communication through Event Bus

vs Observer Pattern:
Observer:  Subject ──direct──→ Observers
Pub/Sub:   Publisher ──Bus──→ Subscribers

Event Flow:
1. Subscriber subscribes to event type
2. Publisher publishes event to bus
3. Bus finds matching subscribers
4. Bus delivers event to subscribers
```

## Example
```javascript
// Simple Event Bus (Pub/Sub)
class EventBus {
  constructor() {
    this.events = {};
  }
  
  subscribe(eventName, callback) {
    if (!this.events[eventName]) {
      this.events[eventName] = [];
    }
    
    this.events[eventName].push(callback);
    
    // Return unsubscribe function
    return () => {
      this.events[eventName] = this.events[eventName].filter(
        cb => cb !== callback
      );
    };
  }
  
  publish(eventName, data) {
    const callbacks = this.events[eventName];
    
    if (!callbacks || callbacks.length === 0) {
      return;
    }
    
    callbacks.forEach(callback => {
      callback(data);
    });
  }
  
  unsubscribe(eventName, callback) {
    if (!this.events[eventName]) return;
    
    this.events[eventName] = this.events[eventName].filter(
      cb => cb !== callback
    );
  }
  
  clear(eventName) {
    if (eventName) {
      delete this.events[eventName];
    } else {
      this.events = {};
    }
  }
}

// Usage
const eventBus = new EventBus();

// Subscriber 1
const unsubscribe1 = eventBus.subscribe('user:login', (user) => {
  console.log('Subscriber 1: User logged in:', user.name);
});

// Subscriber 2
eventBus.subscribe('user:login', (user) => {
  console.log('Subscriber 2: Track login for:', user.name);
});

// Publisher
eventBus.publish('user:login', { name: 'John', id: 1 });
// Subscriber 1: User logged in: John
// Subscriber 2: Track login for: John

unsubscribe1(); // Unsubscribe first subscriber
eventBus.publish('user:login', { name: 'Jane', id: 2 });
// Subscriber 2: Track login for: Jane

// Enhanced Event Bus with wildcards and once
class AdvancedEventBus {
  constructor() {
    this.events = new Map();
  }
  
  on(eventName, callback) {
    if (!this.events.has(eventName)) {
      this.events.set(eventName, []);
    }
    
    this.events.get(eventName).push({ callback, once: false });
    
    return () => this.off(eventName, callback);
  }
  
  once(eventName, callback) {
    if (!this.events.has(eventName)) {
      this.events.set(eventName, []);
    }
    
    this.events.get(eventName).push({ callback, once: true });
  }
  
  off(eventName, callback) {
    if (!this.events.has(eventName)) return;
    
    const handlers = this.events.get(eventName);
    this.events.set(
      eventName,
      handlers.filter(handler => handler.callback !== callback)
    );
  }
  
  emit(eventName, data) {
    if (!this.events.has(eventName)) return;
    
    const handlers = this.events.get(eventName);
    const oneTimeHandlers = [];
    
    handlers.forEach(handler => {
      handler.callback(data);
      
      if (handler.once) {
        oneTimeHandlers.push(handler);
      }
    });
    
    // Remove one-time handlers
    if (oneTimeHandlers.length > 0) {
      this.events.set(
        eventName,
        handlers.filter(h => !oneTimeHandlers.includes(h))
      );
    }
  }
  
  clear() {
    this.events.clear();
  }
}

// Usage
const bus = new AdvancedEventBus();

bus.on('data:received', (data) => {
  console.log('Always handle:', data);
});

bus.once('data:received', (data) => {
  console.log('Handle only once:', data);
});

bus.emit('data:received', 'First');
// Always handle: First
// Handle only once: First

bus.emit('data:received', 'Second');
// Always handle: Second
// (once handler already removed)

// Real-world: Chat Application
class ChatApp {
  constructor() {
    this.eventBus = new EventBus();
  }
  
  // Publishers
  sendMessage(message) {
    this.eventBus.publish('message:sent', {
      text: message,
      timestamp: new Date(),
      sender: 'currentUser'
    });
  }
  
  userJoined(username) {
    this.eventBus.publish('user:joined', { username });
  }
  
  userLeft(username) {
    this.eventBus.publish('user:left', { username });
  }
  
  // Subscriber setup
  onMessageSent(callback) {
    return this.eventBus.subscribe('message:sent', callback);
  }
  
  onUserJoined(callback) {
    return this.eventBus.subscribe('user:joined', callback);
  }
  
  onUserLeft(callback) {
    return this.eventBus.subscribe('user:left', callback);
  }
}

// Usage
const chat = new ChatApp();

// UI Component subscribes
chat.onMessageSent((data) => {
  console.log(`[${data.timestamp.toLocaleTimeString()}] ${data.sender}: ${data.text}`);
});

// Analytics subscribes
chat.onMessageSent((data) => {
  console.log('Analytics: Message sent');
});

// Notification subscribes
chat.onUserJoined((data) => {
  console.log(`🔔 ${data.username} joined the chat`);
});

// Publisher sends events
chat.sendMessage('Hello everyone!');
chat.userJoined('Alice');

// Global Event Bus (Singleton)
const globalEventBus = new EventBus();

// Module A publishes
function moduleA() {
  globalEventBus.publish('app:ready', { version: '1.0.0' });
}

// Module B subscribes
function moduleB() {
  globalEventBus.subscribe('app:ready', (data) => {
    console.log('App is ready, version:', data.version);
  });
}

// No direct dependency between modules!

// Event Bus with async support
class AsyncEventBus {
  constructor() {
    this.events = new Map();
  }
  
  on(eventName, callback) {
    if (!this.events.has(eventName)) {
      this.events.set(eventName, []);
    }
    this.events.get(eventName).push(callback);
  }
  
  async emit(eventName, data) {
    if (!this.events.has(eventName)) return;
    
    const handlers = this.events.get(eventName);
    
    // Execute handlers in parallel
    await Promise.all(
      handlers.map(handler => 
        Promise.resolve(handler(data))
      )
    );
  }
  
  async emitSequential(eventName, data) {
    if (!this.events.has(eventName)) return;
    
    const handlers = this.events.get(eventName);
    
    // Execute handlers sequentially
    for (const handler of handlers) {
      await Promise.resolve(handler(data));
    }
  }
}

// Usage with async
const asyncBus = new AsyncEventBus();

asyncBus.on('data:save', async (data) => {
  await saveToDatabase(data);
  console.log('Saved to database');
});

asyncBus.on('data:save', async (data) => {
  await sendToAnalytics(data);
  console.log('Sent to analytics');
});

await asyncBus.emit('data:save', { user: 'John' });
// Both handlers run in parallel

// Event Bus with priority
class PriorityEventBus {
  constructor() {
    this.events = new Map();
  }
  
  on(eventName, callback, priority = 0) {
    if (!this.events.has(eventName)) {
      this.events.set(eventName, []);
    }
    
    this.events.get(eventName).push({ callback, priority });
    
    // Sort by priority (higher first)
    this.events.get(eventName).sort((a, b) => b.priority - a.priority);
  }
  
  emit(eventName, data) {
    if (!this.events.has(eventName)) return;
    
    const handlers = this.events.get(eventName);
    handlers.forEach(handler => handler.callback(data));
  }
}

// Usage
const priorityBus = new PriorityEventBus();

priorityBus.on('init', () => console.log('Normal priority'), 0);
priorityBus.on('init', () => console.log('High priority'), 10);
priorityBus.on('init', () => console.log('Low priority'), -10);

priorityBus.emit('init');
// High priority
// Normal priority
// Low priority
```

## Usage
- **When to use**: 
  - Microservices communication
  - Decoupled module communication
  - Event-driven architectures
  - Real-time applications (chat, notifications)
  - Plugin systems
  - Cross-component communication without direct dependencies
  
- **Real-world example**:
  ```javascript
  // E-commerce Order System
  class OrderSystem {
    constructor() {
      this.eventBus = new EventBus();
    }
    
    // Order Service publishes
    placeOrder(order) {
      console.log('Order placed:', order.id);
      
      this.eventBus.publish('order:placed', order);
    }
    
    // Inventory Service subscribes
    setupInventoryService() {
      this.eventBus.subscribe('order:placed', (order) => {
        console.log('Inventory: Reserving items for order', order.id);
        this.reserveItems(order.items);
      });
    }
    
    // Payment Service subscribes
    setupPaymentService() {
      this.eventBus.subscribe('order:placed', (order) => {
        console.log('Payment: Processing payment for order', order.id);
        this.processPayment(order.total);
      });
    }
    
    // Email Service subscribes
    setupEmailService() {
      this.eventBus.subscribe('order:placed', (order) => {
        console.log('Email: Sending confirmation to', order.customer);
        this.sendConfirmationEmail(order);
      });
    }
    
    // Analytics Service subscribes
    setupAnalyticsService() {
      this.eventBus.subscribe('order:placed', (order) => {
        console.log('Analytics: Tracking order', order.id);
        this.trackOrder(order);
      });
    }
  }
  
  // Application State Management
  class StateManager {
    constructor() {
      this.eventBus = new EventBus();
      this.state = {};
    }
    
    dispatch(action) {
      this.eventBus.publish('action:dispatched', action);
      
      // Update state
      this.state = this.reducer(this.state, action);
      
      this.eventBus.publish('state:changed', this.state);
    }
    
    subscribe(callback) {
      return this.eventBus.subscribe('state:changed', callback);
    }
    
    subscribeToActions(callback) {
      return this.eventBus.subscribe('action:dispatched', callback);
    }
  }
  
  // Logger middleware
  const stateManager = new StateManager();
  stateManager.subscribeToActions((action) => {
    console.log('Action:', action.type, action.payload);
  });
  
  // UI components subscribe
  stateManager.subscribe((state) => {
    console.log('State updated:', state);
    updateUI(state);
  });
  ```

- **Best practices**:
  - Use descriptive event names (namespace:action format)
  - Always unsubscribe to prevent memory leaks
  - Document available events and their data structure
  - Avoid tight coupling through event naming
  - Consider using TypeScript for event type safety
  - Implement error handling in subscribers
  - Use singleton event bus for global communication
  - Consider event versioning for evolving systems

## FAQ / Interview Questions

**Q: What is the Pub/Sub pattern and how does it differ from Observer pattern?**
A: **Pub/Sub** uses an event bus to completely decouple publishers from subscribers:

**Observer Pattern**:
```javascript
// Subject knows observers
subject.subscribe(observer);
subject.notify(data); // Directly notifies observers
```

**Pub/Sub Pattern**:
```javascript
// Publishers/subscribers don't know each other
eventBus.subscribe('event', callback);
eventBus.publish('event', data); // Bus routes to subscribers
```

**Key differences**:
- Observer: Direct reference, tighter coupling
- Pub/Sub: Event bus mediator, complete decoupling
- Observer: Synchronous by default
- Pub/Sub: Can be async, supports filtering

**Q: What are the advantages and disadvantages of Pub/Sub?**
A:
**Advantages**:
- **Complete decoupling**: Publishers/subscribers independent
- **Scalability**: Easy to add new subscribers
- **Flexibility**: Dynamic subscription/unsubscription
- **Loose coupling**: Easier to maintain and test
- **Parallel processing**: Multiple subscribers handle same event

**Disadvantages**:
- **Debugging difficulty**: Hard to trace event flow
- **No guaranteed delivery**: Subscriber might miss events
- **Memory leaks**: Forgot to unsubscribe
- **Performance**: Event bus overhead
- **Complexity**: Indirect communication harder to understand

**Q: How do you prevent memory leaks in Pub/Sub?**
A: Key strategies:

```javascript
// 1. Always unsubscribe
const unsubscribe = eventBus.subscribe('event', handler);
// When done:
unsubscribe();

// 2. Cleanup in lifecycle
class Component {
  componentDidMount() {
    this.unsubscribe = eventBus.subscribe('event', this.handler);
  }
  
  componentWillUnmount() {
    this.unsubscribe(); // Critical!
  }
}

// 3. Use once for one-time subscriptions
eventBus.once('init', handler); // Auto-unsubscribes

// 4. Clear all subscriptions when appropriate
eventBus.clear('event');
```

**Q: How would you implement async event handling in Pub/Sub?**
A:
```javascript
class AsyncEventBus {
  constructor() {
    this.events = new Map();
  }
  
  on(event, handler) {
    if (!this.events.has(event)) {
      this.events.set(event, []);
    }
    this.events.get(event).push(handler);
  }
  
  async emit(event, data) {
    const handlers = this.events.get(event) || [];
    
    // Parallel execution
    await Promise.all(
      handlers.map(h => Promise.resolve(h(data)))
    );
  }
  
  async emitSequential(event, data) {
    const handlers = this.events.get(event) || [];
    
    // Sequential execution
    for (const handler of handlers) {
      await Promise.resolve(handler(data));
    }
  }
}
```

**Q: What are real-world examples of Pub/Sub?**
A:
- **Redux**: `store.subscribe()`, actions dispatched
- **Node.js EventEmitter**: `on()`, `emit()`
- **WebSockets**: Server broadcasts to all clients
- **Message queues**: RabbitMQ, Kafka, Redis Pub/Sub
- **Browser**: `addEventListener()`, `dispatchEvent()`
- **Vue.js**: Event bus for component communication
- **Microservices**: Service-to-service messaging

```javascript
// Node.js EventEmitter
const EventEmitter = require('events');
const emitter = new EventEmitter();

emitter.on('data', (data) => console.log(data));
emitter.emit('data', { message: 'Hello' });
```

## References
- [MDN Web Docs - EventTarget](https://developer.mozilla.org/en-US/docs/Web/API/EventTarget)
- [Node.js - EventEmitter](https://nodejs.org/api/events.html)
- [Pub/Sub Pattern Explained](https://www.patterns.dev/posts/publish-subscribe-pattern/)
- [Learning JavaScript Design Patterns](https://addyosmani.com/resources/essentialjsdesignpatterns/book/)

---
*See also: [Observer Pattern](Observer.md), [Event Delegation](EventDelegation.md), [Event Loop](EventLoop.md)*
