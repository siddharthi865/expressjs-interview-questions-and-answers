# Set 10

| S.No. | Question                                                                                                                                                                                                                                         |
| ----- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| 1.    | [How do you implement request deduplication in APIs?](#question-1-how-do-you-implement-request-deduplication-in-apis)                                                                                                                            |
| 2.    | [What are dead-letter queues and why are they important?](#question-2-what-are-dead-letter-queues-and-why-are-they-important)                                                                                                                    |
| 3.    | [How would you implement health checks in Express.js services?](#question-3-how-would-you-implement-health-checks-in-expressjs-services)                                                                                                         |
| 4.    | [What is the difference between liveness probes and readiness probes?](#question-4-what-is-the-difference-between-liveness-probes-and-readiness-probes)                                                                                          |
| 5.    | [How do you protect APIs from brute-force login attacks?](#question-5-how-do-you-protect-apis-from-brute-force-login-attacks)                                                                                                                    |
| 6.    | [How do you securely store passwords in backend systems?](#question-6-how-do-you-securely-store-passwords-in-backend-systems)                                                                                                                    |
| 7.    | [What are timing attacks and how can they affect authentication systems?](#question-7-what-are-timing-attacks-and-how-can-they-affect-authentication-systems)                                                                                    |
| 8.    | [How would you implement API key management securely?](#question-8-how-would-you-implement-api-key-management-securely)                                                                                                                          |
| 9.    | [How do you design multi-region Express.js deployments with failover support?](#question-9-how-do-you-design-multi-region-expressjs-deployments-with-failover-support)                                                                           |
| 10.   | [What are the trade-offs between synchronous and asynchronous inter-service communication?](#question-10-what-are-the-trade-offs-between-synchronous-and-asynchronous-inter-service-communication)                                               |
| 11.   | [Design a distributed notification system using Express.js](#question-11-design-a-distributed-notification-system-using-expressjs)                                                                                                               |
| 12.   | [How would you architect a real-time collaborative editing backend using Express.js?](#question-12-how-would-you-architect-a-real-time-collaborative-editing-backend-using-expressjs)                                                            |
| 13.   | [Design an API platform capable of serving thousands of third-party developers securely](#question-13-design-an-api-platform-capable-of-serving-thousands-of-third-party-developers-securely)                                                    |
| 14.   | [How would you build a centralized logging pipeline for hundreds of Express.js services?](#question-14-how-would-you-build-a-centralized-logging-pipeline-for-hundreds-of-expressjs-services)                                                    |
| 15.   | [Design a distributed session management system for globally scaled applications](#question-15-design-a-distributed-session-management-system-for-globally-scaled-applications)                                                                  |
| 16.   | [How would you architect an Express.js backend for high-frequency financial transactions?](#question-16-how-would-you-architect-an-expressjs-backend-for-high-frequency-financial-transactions)                                                  |
| 17.   | [How would you detect and mitigate cascading failures in Express.js microservices?](#question-17-how-would-you-detect-and-mitigate-cascading-failures-in-expressjs-microservices)                                                                |
| 18.   | [Design a globally distributed API caching layer for low-latency responses](#question-18-design-a-globally-distributed-api-caching-layer-for-low-latency-responses)                                                                              |
| 19.   | [How would you design a canary deployment strategy for critical Express.js services?](#question-19-how-would-you-design-a-canary-deployment-strategy-for-critical-expressjs-services)                                                            |
| 20.   | [Explain how you would perform root-cause analysis for a large-scale production outage in an Express.js ecosystem](#question-20-explain-how-you-would-perform-root-cause-analysis-for-a-large-scale-production-outage-in-an-expressjs-ecosystem) |

## Question 1. How do you implement request deduplication in APIs?

### Direct Answer

**Request deduplication** is a technique that prevents the same API request from being processed multiple times. It's commonly used to handle **duplicate client retries, network failures, double-clicks, webhook retries, and distributed systems**.

The most common implementation is **idempotency keys**, where the client sends a unique key with each request, and the server stores the result of the first successful execution. Any subsequent request with the same key returns the previously stored response instead of executing the operation again.

---

# Detailed Explanation

Duplicate requests can happen because of:

- User clicking the submit button multiple times
- Mobile applications retrying requests after timeouts
- Load balancers retrying failed requests
- Payment providers sending webhook events multiple times
- Network failures causing clients to retry

Without deduplication, these can result in:

- Duplicate orders
- Multiple payments
- Duplicate emails
- Repeated database writes

---

# Common Approaches

## 1. Idempotency Keys (Recommended)

This is the industry-standard approach for APIs that create resources.

The client generates a unique UUID:

```http
POST /payments

Idempotency-Key: 8d2abcf7-1298-4f2a-a8fd-f2d39f
```

Server flow:

```
Receive request
      │
      ▼
Check if key exists
      │
 ┌────┴────┐
 │         │
Exists   Doesn't exist
 │         │
Return    Process request
stored        │
response      ▼
         Store response
```

Example Express middleware:

```javascript
const store = new Map();

async function idempotency(req, res, next) {
  const key = req.headers["idempotency-key"];

  if (!key) {
    return res.status(400).json({
      error: "Idempotency-Key required",
    });
  }

  if (store.has(key)) {
    return res.json(store.get(key));
  }

  const originalJson = res.json.bind(res);

  res.json = (body) => {
    store.set(key, body);
    return originalJson(body);
  };

  next();
}
```

Usage:

```javascript
app.post("/orders", idempotency, async (req, res) => {
  const order = await createOrder(req.body);
  res.json(order);
});
```

---

# 2. Database Unique Constraints

Sometimes the database itself prevents duplicates.

Example:

```sql
CREATE UNIQUE INDEX unique_payment
ON payments(transaction_id);
```

Then:

```javascript
try {
  await Payment.create(data);
} catch (err) {
  if (err.code === "23505") {
    // duplicate
  }
}
```

This is very reliable because the database enforces uniqueness.

---

# 3. Request Hashing

Instead of requiring a client-generated key, hash the request body.

Example:

```
SHA256(
    userId +
    amount +
    productId
)
```

If the same hash already exists:

```
Return previous response
```

Pros:

- No client changes

Cons:

- Different requests may accidentally hash to the same logical operation.
- Not suitable if identical payloads should legitimately create separate resources.

---

# 4. Distributed Cache (Redis)

For horizontally scaled applications, in-memory storage won't work because requests may reach different servers.

Use a shared cache like Redis.

```
Client
   │
   ▼
Express
   │
   ▼
Redis
   │
Check key
   │
 ┌─┴────────┐
 │          │
Found     Missing
 │          │
Return    Process
Cached     Request
Response
```

Example:

```javascript
const result = await redis.get(key);

if (result) {
  return res.json(JSON.parse(result));
}

const order = await createOrder();

await redis.set(key, JSON.stringify(order), {
  EX: 3600,
});

res.json(order);
```

---

# 5. Locking While Processing

A duplicate request may arrive before the first one finishes.

Example:

```
Request A
      │
      ▼
Processing...

Request B
      │
      ▼
Should NOT start another payment
```

Use Redis locks:

```
SET payment123 LOCK NX EX 30
```

If lock exists:

- Return `409 Conflict`
- Return `202 Accepted`
- Wait for the first request to finish
- Or return the stored result when available

This prevents concurrent execution of the same operation.

---

# Where Deduplication is Essential

- Payment APIs
- Order creation
- Inventory management
- Ticket booking
- Financial transactions
- Webhook processing
- Email sending
- Background job scheduling

---

# HTTP Idempotency

Some HTTP methods are naturally idempotent:

| Method | Idempotent?   | Notes                                    |
| ------ | ------------- | ---------------------------------------- |
| GET    | ✅ Yes        | Read-only                                |
| PUT    | ✅ Yes        | Replaces a resource                      |
| DELETE | ✅ Yes        | Repeated deletes have the same end state |
| POST   | ❌ No         | Usually creates a new resource           |
| PATCH  | ❌ Usually No | Partial updates may not be repeatable    |

Since `POST` is typically non-idempotent, APIs often implement **idempotency keys** to safely handle retries.

---

# Best Practices

- Use **idempotency keys** for all critical `POST` endpoints.
- Store keys in a **shared datastore** (such as Redis or a database) instead of process memory for distributed deployments.
- Associate the key with the **authenticated user or client** to prevent cross-user collisions.
- Store the **response, status code, and request fingerprint** (e.g., method, path, body hash). If the same key is reused with a different payload, reject it (commonly with `409 Conflict` or `422 Unprocessable Entity`).
- Set an appropriate **TTL (Time-To-Live)** on stored keys (e.g., 24 hours for payment operations) to avoid unbounded storage growth.
- Use **distributed locks** or atomic operations (e.g., Redis `SET NX`) to prevent race conditions when duplicate requests arrive simultaneously.
- Back deduplication with **database constraints** where business rules require guaranteed uniqueness (e.g., unique transaction IDs or order references).
- Log duplicate requests for monitoring and troubleshooting.

---

# Common Pitfalls

- Using an in-memory `Map` in production, which fails in multi-instance deployments and loses state on restarts.
- Storing only the idempotency key without validating that the request payload matches the original request.
- Not handling concurrent requests, allowing two identical requests to execute before the first result is stored.
- Keeping idempotency records forever, leading to unnecessary storage growth.
- Relying solely on application logic without enforcing critical uniqueness constraints at the database level.

---

# Interview Tip

In an interview, emphasize that **idempotency keys are the preferred solution for request deduplication in modern REST APIs**, especially for payment and order creation endpoints. Mention that in production Express.js applications, the keys should be stored in a **shared datastore like Redis or a database**, combined with **atomic locking and database uniqueness constraints** to safely handle retries, concurrent requests, and horizontally scaled deployments.

## Question 2. What are dead-letter queues and why are they important?

## Question 3. How would you implement health checks in Express.js services?

## Question 4. What is the difference between liveness probes and readiness probes?

## Question 5. How do you protect APIs from brute-force login attacks?

## Question 6. How do you securely store passwords in backend systems?

## Question 7. What are timing attacks and how can they affect authentication systems?

## Question 8. How would you implement API key management securely?

## Question 9. How do you design multi-region Express.js deployments with failover support?

## Question 10. What are the trade-offs between synchronous and asynchronous inter-service communication?

## Question 11. Design a distributed notification system using Express.js

## Question 12. How would you architect a real-time collaborative editing backend using Express.js?

## Question 13. Design an API platform capable of serving thousands of third-party developers securely

## Question 14. How would you build a centralized logging pipeline for hundreds of Express.js services?

## Question 15. Design a distributed session management system for globally scaled applications

## Question 16. How would you architect an Express.js backend for high-frequency financial transactions?

## Question 17. How would you detect and mitigate cascading failures in Express.js microservices?

## Question 18. Design a globally distributed API caching layer for low-latency responses

## Question 19. How would you design a canary deployment strategy for critical Express.js services?

## Question 20. Explain how you would perform root-cause analysis for a large-scale production outage in an Express.js ecosystem
