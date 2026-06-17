# Set 15

| S.No. | Question                                                                                                                                                                                                                             |
| ----- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| 1.    | [How do you design APIs for eventual consistency?](#question-1-how-do-you-design-apis-for-eventual-consistency)                                                                                                                      |
| 2.    | [What are compensating transactions in distributed systems?](#question-2-what-are-compensating-transactions-in-distributed-systems)                                                                                                  |
| 3.    | [How do message brokers improve scalability in Express.js architectures?](#question-3-how-do-message-brokers-improve-scalability-in-expressjs-architectures)                                                                         |
| 4.    | [What are the trade-offs between Kafka and RabbitMQ for backend systems?](#question-4-what-are-the-trade-offs-between-kafka-and-rabbitmq-for-backend-systems)                                                                        |
| 5.    | [How do you ensure exactly-once processing semantics in distributed systems?](#question-5-how-do-you-ensure-exactly-once-processing-semantics-in-distributed-systems)                                                                |
| 6.    | [How do you detect duplicate event consumption in message-driven architectures?](#question-6-how-do-you-detect-duplicate-event-consumption-in-message-driven-architectures)                                                          |
| 7.    | [What are common anti-patterns in microservices built with Express.js?](#question-7-what-are-common-anti-patterns-in-microservices-built-with-expressjs)                                                                             |
| 8.    | [How do you avoid tight coupling between backend services?](#question-8-how-do-you-avoid-tight-coupling-between-backend-services)                                                                                                    |
| 9.    | [What are the challenges of schema evolution in APIs?](#question-9-what-are-the-challenges-of-schema-evolution-in-apis)                                                                                                              |
| 10.   | [How do blue-green and rolling deployments differ operationally?](#question-10-how-do-blue-green-and-rolling-deployments-differ-operationally)                                                                                       |
| 11.   | [Design a globally scalable media upload backend using Express.js](#question-11-design-a-globally-scalable-media-upload-backend-using-expressjs)                                                                                     |
| 12.   | [Design a distributed API rate-limiting system with tenant isolation](#question-12-design-a-distributed-api-rate-limiting-system-with-tenant-isolation)                                                                              |
| 13.   | [How would you architect a backend system for processing billions of analytics events daily?](#question-13-how-would-you-architect-a-backend-system-for-processing-billions-of-analytics-events-daily)                               |
| 14.   | [Design a resilient webhook delivery platform using Express.js](#question-14-design-a-resilient-webhook-delivery-platform-using-expressjs)                                                                                           |
| 15.   | [How would you build a multi-region Express.js backend with automatic failover?](#question-15-how-would-you-build-a-multi-region-expressjs-backend-with-automatic-failover)                                                          |
| 16.   | [Design a highly scalable API monitoring and alerting platform](#question-16-design-a-highly-scalable-api-monitoring-and-alerting-platform)                                                                                          |
| 17.   | [How would you build a backend system supporting real-time multiplayer game sessions?](#question-17-how-would-you-build-a-backend-system-supporting-real-time-multiplayer-game-sessions)                                             |
| 18.   | [Design a distributed search indexing pipeline using Express.js microservices](#question-18-design-a-distributed-search-indexing-pipeline-using-expressjs-microservices)                                                             |
| 19.   | [How would you architect a large-scale backend handling flash-sale traffic spikes?](#question-19-how-would-you-architect-a-large-scale-backend-handling-flash-sale-traffic-spikes)                                                   |
| 20.   | [Explain how you would redesign a failing monolithic Express.js backend into a resilient distributed system](#question-20-explain-how-you-would-redesign-a-failing-monolithic-expressjs-backend-into-a-resilient-distributed-system) |

## Question 1. How do you design APIs for eventual consistency?

## Direct Answer

Designing APIs for **eventual consistency** means accepting that different parts of the system may not be updated immediately, while ensuring they **converge to the correct state over time**. In Express.js, this typically involves asynchronous processing with message queues, background workers, idempotent operations, status endpoints, event-driven communication, and clear API contracts that communicate when data is pending versus finalized.

---

# Designing APIs for Eventual Consistency

In distributed systems, it's often impossible or undesirable to keep every service perfectly synchronized. Instead of enforcing immediate consistency through distributed transactions, services update independently and synchronize asynchronously.

Typical architecture:

```
Client
   │
   ▼
Express API
   │
   ├── Save request
   ├── Return Accepted (202)
   └── Publish Event
            │
      Message Queue
            │
     Background Workers
            │
 ┌──────────┴──────────┐
 ▼                     ▼
Inventory Service   Billing Service
        │                 │
        └──────► Eventually Consistent
```

Examples include:

- Order processing
- Payment systems
- Inventory synchronization
- Email sending
- Notifications
- Analytics pipelines

---

# 1. Return Immediately Instead of Waiting

Instead of blocking until every service completes, accept the request and process it asynchronously.

```javascript
app.post("/orders", async (req, res) => {
  const order = await Order.create({
    ...req.body,
    status: "PENDING",
  });

  await queue.publish("order.created", order);

  res.status(202).json({
    orderId: order.id,
    status: "PENDING",
  });
});
```

Benefits:

- Faster responses
- Better scalability
- Loose coupling
- Higher availability

---

# 2. Use HTTP 202 Accepted

When processing will continue in the background, return:

```
HTTP/1.1 202 Accepted
```

Example response:

```json
{
  "orderId": "123",
  "status": "PENDING",
  "message": "Order is being processed."
}
```

This tells clients:

- Request accepted
- Work has not completed
- Final state will be available later

---

# 3. Expose Resource Status

Allow clients to query progress.

```http
GET /orders/123
```

Example response:

```json
{
  "id": "123",
  "status": "PROCESSING"
}
```

Possible states:

```
PENDING
PROCESSING
PAID
COMPLETED
FAILED
CANCELLED
```

Avoid returning ambiguous responses like:

```json
{
  "success": true
}
```

Instead, expose the actual lifecycle state.

---

# 4. Publish Domain Events

After changing state, publish events.

```
Order Created
↓

Inventory Reserved
↓

Payment Completed
↓

Shipment Created
↓

Email Sent
```

Example:

```javascript
await queue.publish("order.created", {
  orderId: order.id,
});
```

Consumers update their own databases independently.

This avoids tightly coupling services.

---

# 5. Make Operations Idempotent

Duplicate messages are common in distributed systems.

Workers must safely handle retries.

Bad:

```
Receive event twice

↓

Charge customer twice
```

Good:

```
Receive event twice

↓

Detect duplicate

↓

Ignore second event
```

Example:

```javascript
if (await alreadyProcessed(event.id)) {
  return;
}

await processOrder(event);
```

---

# 6. Support Retry Mechanisms

Background processing should retry temporary failures.

Example flow:

```
Worker
   │
Failure
   │
Retry
   │
Retry
   │
Success
```

Use:

- exponential backoff
- retry limits
- dead-letter queues
- logging

---

# 7. Design Clear State Transitions

Avoid arbitrary status changes.

Instead define a state machine.

```
PENDING
    │
    ▼
PROCESSING
   │      │
   ▼      ▼
SUCCESS FAILED
```

This prevents invalid transitions like:

```
FAILED
↓

PROCESSING
```

unless explicitly allowed.

---

# 8. Allow Clients to Poll

Simple approach:

```
POST /orders

↓

202 Accepted

↓

GET /orders/123

↓

Completed
```

Useful when:

- Mobile apps
- Third-party integrations
- Simpler architectures

---

# 9. Push Updates When Possible

Instead of polling, use:

- WebSockets
- Server-Sent Events (SSE)
- Webhooks (for external systems)

Example:

```
Client
   │
POST /orders
   │
202 Accepted
   │
WebSocket Event
   │
Order Completed
```

This reduces unnecessary polling.

---

# 10. Use the Outbox Pattern

A common challenge is ensuring the database update and event publication stay synchronized.

Problem:

```
Save Order

↓

Server crashes

↓

Event never published
```

Solution:

```
Transaction

Save Order
Save Event (Outbox)

Commit

↓

Background Worker

↓

Publish Event

↓

Mark Event Sent
```

Benefits:

- No lost events
- Reliable delivery
- Consistent state between the database and message broker

---

# 11. Handle Read-After-Write Consistency

Immediately after a write, a client may not see the latest data.

Example:

```
POST /orders

↓

GET /orders

↓

Order missing
```

Possible approaches:

- Return the created resource directly in the POST response.
- Include a processing status so clients know synchronization is pending.
- Use the primary database for critical read-after-write scenarios instead of a lagging replica.
- Document any expected propagation delays.

---

# 12. Handle Failures Gracefully

If part of the workflow fails:

```
Order Created

↓

Inventory Failed
```

Possible outcomes:

```
FAILED
```

or

```
COMPENSATING
↓

Refund

↓

Cancelled
```

Rather than leaving resources in an unknown state, expose meaningful statuses and trigger compensating actions when appropriate.

---

# 13. Use Correlation IDs

Track an entire workflow with a shared identifier.

Example:

```
X-Correlation-ID: abc123
```

Every service logs:

```
API
↓

Queue
↓

Payment

↓

Inventory

↓

Shipping
```

This greatly simplifies debugging and tracing distributed requests.

---

# Express.js Example

```javascript
app.post("/orders", async (req, res) => {
  const order = await Order.create({
    status: "PENDING",
  });

  await queue.publish("order.created", {
    orderId: order.id,
  });

  res.status(202).json({
    orderId: order.id,
    status: "PENDING",
  });
});

app.get("/orders/:id", async (req, res) => {
  const order = await Order.findByPk(req.params.id);

  if (!order) {
    return res.sendStatus(404);
  }

  res.json(order);
});
```

---

# Common Pitfalls

- Returning `200 OK` when processing hasn't finished.
- Expecting immediate consistency across distributed services.
- Not making consumers idempotent, leading to duplicate side effects.
- Omitting retry policies or dead-letter queues, causing transient failures to become permanent.
- Not exposing processing status, leaving clients uncertain about request progress.
- Publishing events outside a reliable mechanism like the Outbox Pattern, which can lead to lost events.
- Ignoring correlation IDs, making distributed debugging difficult.
- Failing to document consistency guarantees and expected propagation delays.

---

# Best Practices

- Return **`202 Accepted`** for asynchronous operations.
- Model resources with explicit lifecycle states (e.g., `PENDING`, `PROCESSING`, `COMPLETED`, `FAILED`).
- Process work asynchronously using queues and background workers.
- Publish domain events to decouple services.
- Ensure all consumers are **idempotent**.
- Implement retries with exponential backoff and dead-letter queues.
- Use the **Outbox Pattern** for reliable event publication.
- Provide polling endpoints or push-based updates (WebSockets, SSE, or webhooks).
- Include correlation IDs for end-to-end tracing.
- Clearly document eventual consistency behavior, status transitions, and expected delays so API consumers know what guarantees to expect.

---

## Interview Summary

In a senior-level interview, emphasize that **eventual consistency prioritizes availability and scalability over immediate synchronization**. A well-designed Express.js API accepts requests quickly (often with **`202 Accepted`**), processes work asynchronously through queues, exposes resource status, uses idempotent operations and reliable event publishing (such as the Outbox Pattern), supports retries and compensating actions, and provides clear mechanisms for clients to track progress. These practices enable resilient, scalable distributed systems without relying on costly distributed transactions.

## Question 2. What are compensating transactions in distributed systems?

## Question 3. How do message brokers improve scalability in Express.js architectures?

## Question 4. What are the trade-offs between Kafka and RabbitMQ for backend systems?

## Question 5. How do you ensure exactly-once processing semantics in distributed systems?

## Question 6. How do you detect duplicate event consumption in message-driven architectures?

## Question 7. What are common anti-patterns in microservices built with Express.js?

## Question 8. How do you avoid tight coupling between backend services?

## Question 9. What are the challenges of schema evolution in APIs?

## Question 10. How do blue-green and rolling deployments differ operationally?

## Question 11. Design a globally scalable media upload backend using Express.js

## Question 12. Design a distributed API rate-limiting system with tenant isolation

## Question 13. How would you architect a backend system for processing billions of analytics events daily?

## Question 14. Design a resilient webhook delivery platform using Express.js

## Question 15. How would you build a multi-region Express.js backend with automatic failover?

## Question 16. Design a highly scalable API monitoring and alerting platform

## Question 17. How would you build a backend system supporting real-time multiplayer game sessions?

## Question 18. Design a distributed search indexing pipeline using Express.js microservices

## Question 19. How would you architect a large-scale backend handling flash-sale traffic spikes?

## Question 20. Explain how you would redesign a failing monolithic Express.js backend into a resilient distributed system
