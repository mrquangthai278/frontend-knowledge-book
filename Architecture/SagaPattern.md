# Saga Pattern

## Definition / Concept

The **Saga Pattern** is an architectural pattern for managing distributed transactions and long-running processes across multiple services in microservices architectures. It breaks down a complex business transaction into a series of local transactions, each executed by a separate service, and uses **compensating transactions** to maintain data consistency when failures occur. Unlike traditional two-phase commit (2PC), sagas avoid distributed locks and improve availability by managing eventual consistency through orchestration or choreography approaches. The pattern is essential for handling business processes that span multiple bounded contexts while maintaining system resilience.

- **Distributed Transaction Management**: Coordinates multiple microservices without distributed locks
- **Long-Running Process Support**: Handles business processes that take minutes, hours, or days
- **Compensating Transactions**: Enables rollback through inverse operations rather than traditional rollback
- **Eventual Consistency**: Maintains data integrity while allowing temporary inconsistencies
- **Failure Isolation**: Prevents cascading failures across the distributed system

## Visual Representation

### Orchestration Approach
```
┌─────────────────────────────────────────────────────┐
│  Saga Orchestrator (State Machine)                  │
│  • Orders Service                                   │
│  • Manages workflow state                           │
│  • Triggers compensations on failure                │
└──────────┬──────────────────────────────────────────┘
           │
    ┌──────┼──────────────────┬──────────────┐
    │      │                  │              │
    ▼      ▼                  ▼              ▼
┌─────┐┌─────────┐      ┌──────────┐  ┌─────────┐
│ Order│ Payment │      │ Inventory│  │ Shipping│
│Svc  ││Service  │      │Service   │  │Service  │
└─────┘└─────────┘      └──────────┘  └─────────┘
    1. Create order
    2. Reserve payment
    3. Reserve inventory
    4. Schedule delivery
    [If step 3 fails: compensate 1,2]
```

### Choreography Approach
```
┌──────────┐        ┌──────────┐        ┌──────────┐        ┌──────────┐
│   Order  │        │ Payment  │        │Inventory │        │ Shipping │
│ Service  │        │ Service  │        │ Service  │        │ Service  │
└────┬─────┘        └────┬─────┘        └────┬─────┘        └────┬─────┘
     │                   │                    │                   │
     │ 1. Order Created  │                    │                   │
     ├──Event Broadcast──►                    │                   │
     │                   │                    │                   │
     │                   │ 2. Payment Done    │                   │
     │                   ├──Event Broadcast──►                    │
     │                   │                    │                   │
     │                   │                    │ 3. Reserved       │
     │                   │                    ├──Event Broadcast──►
     │                   │                    │                   │
     │                   │                    │ 4. Shipped        │
     │◄──────Event Flow (Event Bus)──────────────────────────────┤
     │
     [All services listen and react independently]
```

## Example

### Orchestration Pattern - Order Saga
```javascript
// Saga Orchestrator - manages the order processing workflow
class OrderSagaOrchestrator {
  constructor(paymentService, inventoryService, shippingService, orderRepository) {
    this.paymentService = paymentService;
    this.inventoryService = inventoryService;
    this.shippingService = shippingService;
    this.orderRepository = orderRepository;
  }

  /**
   * Execute the order saga with automatic compensations on failure
   * Handles distributed transaction across multiple services
   */
  async executeOrderSaga(orderId, orderDetails) {
    const saga = {
      orderId,
      status: 'PENDING',
      completedSteps: [],
      compensations: []
    };

    try {
      // Step 1: Reserve payment
      console.log(`[Saga ${orderId}] Attempting payment reservation...`);
      const paymentReservation = await this.paymentService.reservePayment(
        orderId,
        orderDetails.amount
      );
      saga.completedSteps.push('paymentReserved');
      saga.compensations.push(() =>
        this.paymentService.cancelPayment(paymentReservation.transactionId)
      );

      // Step 2: Reserve inventory
      console.log(`[Saga ${orderId}] Attempting inventory reservation...`);
      const inventoryReservation = await this.inventoryService.reserveItems(
        orderId,
        orderDetails.items
      );
      saga.completedSteps.push('inventoryReserved');
      saga.compensations.push(() =>
        this.inventoryService.releaseReservation(inventoryReservation.reservationId)
      );

      // Step 3: Schedule shipment
      console.log(`[Saga ${orderId}] Scheduling shipment...`);
      const shipmentDetails = await this.shippingService.scheduleShipment(
        orderId,
        orderDetails.shippingAddress,
        inventoryReservation
      );
      saga.completedSteps.push('shipmentScheduled');
      saga.compensations.push(() =>
        this.shippingService.cancelShipment(shipmentDetails.shipmentId)
      );

      // All steps completed successfully
      saga.status = 'COMPLETED';
      console.log(`[Saga ${orderId}] Order processing completed successfully`);

      await this.orderRepository.updateOrder(orderId, { status: 'CONFIRMED' });
      return { success: true, saga };

    } catch (error) {
      // Execute compensating transactions in reverse order
      console.error(`[Saga ${orderId}] Error at step, executing compensations...`, error.message);
      saga.status = 'COMPENSATING';

      // Reverse order execution: last step compensated first
      for (let i = saga.compensations.length - 1; i >= 0; i--) {
        try {
          console.log(`[Saga ${orderId}] Executing compensation ${i + 1}/${saga.compensations.length}`);
          await saga.compensations[i]();
        } catch (compensationError) {
          console.error(`[Saga ${orderId}] Compensation failed:`, compensationError.message);
          // Log for manual intervention
          await this.orderRepository.logCompensationFailure(orderId, compensationError);
        }
      }

      saga.status = 'COMPENSATED';
      await this.orderRepository.updateOrder(orderId, { status: 'CANCELLED' });
      return { success: false, saga, error: error.message };
    }
  }
}

// Service implementations
class PaymentService {
  async reservePayment(orderId, amount) {
    // Simulated payment service
    if (amount <= 0) throw new Error('Invalid payment amount');
    return { transactionId: `txn_${orderId}_${Date.now()}`, amount };
  }

  async cancelPayment(transactionId) {
    console.log(`  → Reversing payment transaction ${transactionId}`);
    // Refund logic
    return { refundId: `ref_${transactionId}` };
  }
}

class InventoryService {
  async reserveItems(orderId, items) {
    // Simulated inventory reservation
    const missingItem = items.find(item => item.quantity > 100);
    if (missingItem) throw new Error(`Insufficient inventory for ${missingItem.sku}`);
    return { reservationId: `res_${orderId}_${Date.now()}`, items };
  }

  async releaseReservation(reservationId) {
    console.log(`  → Releasing inventory reservation ${reservationId}`);
    return { released: true };
  }
}

class ShippingService {
  async scheduleShipment(orderId, address, inventoryReservation) {
    // Validate address
    if (!address.zipcode) throw new Error('Invalid shipping address');
    return { shipmentId: `ship_${orderId}_${Date.now()}`, address };
  }

  async cancelShipment(shipmentId) {
    console.log(`  → Cancelling shipment ${shipmentId}`);
    return { cancelled: true };
  }
}

// Usage example
async function demonstrateOrderSaga() {
  const paymentService = new PaymentService();
  const inventoryService = new InventoryService();
  const shippingService = new ShippingService();

  // Mock repository
  const orderRepository = {
    updateOrder: async (orderId, data) => console.log(`Order ${orderId} updated:`, data),
    logCompensationFailure: async (orderId, error) => console.error(`Compensation logged for ${orderId}:`, error)
  };

  const orchestrator = new OrderSagaOrchestrator(
    paymentService,
    inventoryService,
    shippingService,
    orderRepository
  );

  // Successful order
  console.log('\n=== Successful Order Saga ===');
  const successResult = await orchestrator.executeOrderSaga('ORD001', {
    amount: 99.99,
    items: [{ sku: 'ITEM123', quantity: 2 }],
    shippingAddress: { zipcode: '10001' }
  });
  console.log('Result:', successResult);

  // Failed order triggers compensation
  console.log('\n=== Failed Order Saga with Compensation ===');
  const failureResult = await orchestrator.executeOrderSaga('ORD002', {
    amount: 49.99,
    items: [{ sku: 'ITEM999', quantity: 150 }], // Exceeds inventory
    shippingAddress: { zipcode: '10002' }
  });
  console.log('Result:', failureResult);
}
```

### Choreography Pattern - Event-Driven Saga
```javascript
// Event-driven saga using pub/sub pattern
class EventBus {
  constructor() {
    this.subscribers = {};
  }

  subscribe(eventType, handler) {
    if (!this.subscribers[eventType]) {
      this.subscribers[eventType] = [];
    }
    this.subscribers[eventType].push(handler);
  }

  async publish(event) {
    console.log(`[Event] Publishing: ${event.type}`);
    const handlers = this.subscribers[event.type] || [];
    await Promise.all(handlers.map(handler => handler(event)));
  }
}

// Each service reacts to events independently
class OrderService {
  constructor(eventBus, orderRepository) {
    this.eventBus = eventBus;
    this.orderRepository = orderRepository;

    // Subscribe to events
    this.eventBus.subscribe('OrderRequested', this.handleOrderRequested.bind(this));
    this.eventBus.subscribe('PaymentCompleted', this.handlePaymentCompleted.bind(this));
    this.eventBus.subscribe('PaymentFailed', this.handlePaymentFailed.bind(this));
  }

  async handleOrderRequested(event) {
    console.log(`[Order Service] Creating order ${event.orderId}`);
    await this.orderRepository.create({ id: event.orderId, status: 'PENDING' });

    // Trigger payment
    await this.eventBus.publish({
      type: 'ProcessPaymentRequested',
      orderId: event.orderId,
      amount: event.amount
    });
  }

  async handlePaymentCompleted(event) {
    console.log(`[Order Service] Payment confirmed for ${event.orderId}`);
    await this.orderRepository.update(event.orderId, { status: 'PAYMENT_CONFIRMED' });

    // Request inventory reservation
    await this.eventBus.publish({
      type: 'ReserveInventoryRequested',
      orderId: event.orderId,
      items: event.items
    });
  }

  async handlePaymentFailed(event) {
    console.log(`[Order Service] Payment failed for ${event.orderId}`);
    await this.orderRepository.update(event.orderId, { status: 'CANCELLED' });
  }
}

class PaymentService {
  constructor(eventBus) {
    this.eventBus = eventBus;
    this.eventBus.subscribe('ProcessPaymentRequested', this.processPayment.bind(this));
  }

  async processPayment(event) {
    try {
      console.log(`[Payment Service] Processing payment for ${event.orderId}`);
      // Simulate payment processing
      if (Math.random() > 0.1) { // 90% success rate
        await this.eventBus.publish({
          type: 'PaymentCompleted',
          orderId: event.orderId,
          transactionId: `txn_${Date.now()}`,
          amount: event.amount
        });
      } else {
        throw new Error('Payment declined');
      }
    } catch (error) {
      await this.eventBus.publish({
        type: 'PaymentFailed',
        orderId: event.orderId,
        reason: error.message
      });
    }
  }
}

class InventoryService {
  constructor(eventBus) {
    this.eventBus = eventBus;
    this.eventBus.subscribe('ReserveInventoryRequested', this.reserveInventory.bind(this));
  }

  async reserveInventory(event) {
    try {
      console.log(`[Inventory Service] Reserving items for ${event.orderId}`);
      // Simulate inventory check
      if (event.items.some(item => item.quantity > 100)) {
        throw new Error('Insufficient inventory');
      }

      await this.eventBus.publish({
        type: 'InventoryReserved',
        orderId: event.orderId,
        reservationId: `res_${Date.now()}`
      });
    } catch (error) {
      await this.eventBus.publish({
        type: 'InventoryReservationFailed',
        orderId: event.orderId,
        reason: error.message
      });
    }
  }
}

// Usage example
async function demonstrateChoreographySaga() {
  const eventBus = new EventBus();
  const orderRepository = {
    create: async (order) => console.log(`  → Order created: ${JSON.stringify(order)}`),
    update: async (id, data) => console.log(`  → Order updated: ${JSON.stringify(data)}`)
  };

  const orderService = new OrderService(eventBus, orderRepository);
  const paymentService = new PaymentService(eventBus);
  const inventoryService = new InventoryService(eventBus);

  console.log('\n=== Choreography-Based Saga ===');
  await eventBus.publish({
    type: 'OrderRequested',
    orderId: 'ORD003',
    amount: 149.99,
    items: [{ sku: 'ITEM001', quantity: 3 }]
  });

  // Allow async operations to complete
  await new Promise(resolve => setTimeout(resolve, 100));
}
```

## Usage

### When to Use

- **Microservices Architecture**: Managing business transactions that span multiple independent services without relying on distributed transactions
- **Long-Running Processes**: Handling workflows that take significant time (order processing, batch operations, approval flows)
- **Systems Requiring High Availability**: When traditional 2PC would create bottlenecks and availability issues
- **Event-Driven Architectures**: When services need loose coupling through event-based communication
- **Financial Transactions**: Processing multi-step transactions (payment, inventory, shipping) with automatic rollback capabilities
- **IoT and Asynchronous Workflows**: Coordinating operations across distributed components with variable latency

### Real-World Example

**E-Commerce Order Processing**:
1. Customer submits order → Order Service creates order record
2. Payment gateway validates and reserves funds → Payment Service publishes `PaymentReserved` event
3. Warehouse checks inventory and marks items as reserved → Inventory Service publishes `ItemsReserved` event
4. Shipping partner creates shipment label and schedules pickup → Shipping Service publishes `ShipmentScheduled` event
5. If any step fails (e.g., payment declined), compensating transactions automatically:
   - Release inventory reservation
   - Cancel payment hold
   - Notify warehouse of cancellation
   - Update order status to failed

### Best Practices

1. **Idempotency**: Ensure all saga steps and compensations can be executed multiple times safely
   ```javascript
   // Good: Idempotent compensation
   async cancelPayment(transactionId) {
     const existing = await this.db.find({ transactionId, type: 'refund' });
     if (existing) return existing; // Already compensated
     return await this.createRefund(transactionId);
   }
   ```

2. **Compensating Transactions**: Design inverse operations that undo changes reliably
   - Refund for payment reservation
   - Release for inventory reservation
   - Cancel for shipment scheduling

3. **Timeout Handling**: Define clear timeouts for each saga step to prevent indefinite waiting
   ```javascript
   const PAYMENT_TIMEOUT = 30000; // 30 seconds
   const result = await Promise.race([
     this.paymentService.reserve(orderId),
     this.timeout(PAYMENT_TIMEOUT)
   ]);
   ```

4. **Monitoring and Logging**: Maintain detailed logs for saga execution state for troubleshooting
   ```javascript
   saga.log = {
     startTime: Date.now(),
     steps: [],
     compensations: []
   };
   ```

5. **Dead Letter Queue**: Route failed compensations to manual intervention queues
   ```javascript
   if (compensationFailed) {
     await this.deadLetterQueue.publish({
       sagaId: orderId,
       failedCompensation: compensationError,
       requiresManualIntervention: true
     });
   }
   ```

6. **Choose Appropriate Pattern**:
   - **Orchestration**: Better for complex workflows, centralized control, easier debugging
   - **Choreography**: Better for loosely coupled systems, scales horizontally, less central bottleneck

## FAQ / Interview Questions

**Q: What is the Saga Pattern and why is it used in microservices?**
A: The Saga Pattern is an architectural approach for managing distributed transactions across microservices without using 2PC (two-phase commit). It breaks complex transactions into a sequence of local transactions, each handled by a separate service. If any step fails, compensating transactions execute in reverse order to maintain consistency. It's preferred in microservices because 2PC creates availability issues and tight coupling, while sagas provide better scalability and resilience through compensating transactions and eventual consistency.

**Q: What's the difference between orchestration and choreography approaches?**
A: **Orchestration** uses a central orchestrator (state machine) that explicitly commands each service when to execute, maintaining workflow logic centrally. It's easier to debug and handle complex workflows but creates a central bottleneck. **Choreography** uses an event bus where services react independently to published events, creating loose coupling and horizontal scalability but making workflows harder to visualize and debug. Choice depends on complexity: orchestration for intricate workflows, choreography for loosely coupled systems.

**Q: How do compensating transactions work and what challenges exist?**
A: Compensating transactions are inverse operations that undo changes made by previous saga steps. For each forward step, a compensation is registered: payment reservation → refund, inventory reservation → release, etc. Challenges include: (1) not all operations have simple inverses, (2) compensations must be idempotent (safe to retry), (3) services must track enough state to support compensation, (4) timing issues when compensations themselves fail. Solutions include dead-letter queues for failed compensations and manual intervention workflows.

**Q: How does Saga Pattern compare to traditional distributed transactions (2PC)?**
A: 2PC achieves **strong consistency** through blocking locks across all services until all agree to commit, but blocks resources and hurts availability. Saga Pattern achieves **eventual consistency** through compensating transactions, allowing services to proceed independently and retry failed steps. 2PC fails catastrophically if a coordinator crashes; sagas handle failures gracefully. However, 2PC is simpler conceptually. Sagas are standard in modern microservices due to better scalability, availability, and resilience despite requiring more complex thinking about compensation.

**Q: What are the key implementation considerations for saga patterns?**
A: Critical considerations include: (1) **Idempotency** - all operations must be safe to execute multiple times, (2) **State Management** - track saga progress to enable recovery from failures, (3) **Timeout Configuration** - define how long to wait before considering a step failed, (4) **Monitoring** - detailed logging for troubleshooting distributed failures, (5) **Monitoring Dead Letters** - manual intervention for failed compensations, (6) **Event Ordering** - ensure causality in choreography-based sagas, (7) **Backwards Compatibility** - handle schema changes across services, (8) **Testing** - simulate failure scenarios and compensation paths thoroughly.

## References

- [Pattern: Saga - Chris Richardson Microservices Patterns](https://microservices.io/patterns/data/saga.html)
- [Saga Pattern Documentation - AWS Microservices](https://docs.aws.amazon.com/prescriptive-guidance/latest/patterns/saga-pattern.html)
- [Event Sourcing Pattern - Greg Young](https://martinfowler.com/eaaDev/EventSourcing.html)
- [Enterprise Integration Patterns - Gregor Hohpe](https://www.enterpriseintegrationpatterns.com/)
- [Compensating Transactions Pattern - Microsoft Azure](https://learn.microsoft.com/en-us/azure/architecture/patterns/compensating-transaction)

---

*See also: [CQRS Pattern](./CQRS.md), [Event Sourcing](./EventSourcing.md), [Distributed Transactions](./DistributedTransactions.md)*
