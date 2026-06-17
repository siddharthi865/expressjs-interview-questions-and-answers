# Set 13

| S.No. | Question                                                                                                                                                                    |
| ----- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 1.    | [How do you handle concurrent updates to the same resource?](#question-1-how-do-you-handle-concurrent-updates-to-the-same-resource)                                         |
| 2.    | [What are the trade-offs between REST and GraphQL when using Express.js?](#question-2-what-are-the-trade-offs-between-rest-and-graphql-when-using-expressjs)                |
| 3.    | [How would you integrate GraphQL into an Express.js application?](#question-3-how-would-you-integrate-graphql-into-an-expressjs-application)                                |
| 4.    | [What are the performance implications of GraphQL resolvers?](#question-4-what-are-the-performance-implications-of-graphql-resolvers)                                       |
| 5.    | [How do you secure GraphQL endpoints in Express.js?](#question-5-how-do-you-secure-graphql-endpoints-in-expressjs)                                                          |
| 6.    | [What is request payload size limiting and why is it important?](#question-6-what-is-request-payload-size-limiting-and-why-is-it-important)                                 |
| 7.    | [How do you prevent denial-of-service attacks caused by large payloads?](#question-7-how-do-you-prevent-denial-of-service-attacks-caused-by-large-payloads)                 |
| 8.    | [What is API gateway authentication delegation?](#question-8-what-is-api-gateway-authentication-delegation)                                                                 |
| 9.    | [How do reverse proxies interact with Express.js applications?](#question-9-how-do-reverse-proxies-interact-with-expressjs-applications)                                    |
| 10.   | [What is the purpose of `trust proxy` in Express.js?](#question-10-what-is-the-purpose-of-trust-proxy-in-expressjs)                                                         |
| 11.   | [How do you retrieve the real client IP behind a proxy?](#question-11-how-do-you-retrieve-the-real-client-ip-behind-a-proxy)                                                |
| 12.   | [What are signed URLs and where are they useful?](#question-12-what-are-signed-urls-and-where-are-they-useful)                                                              |
| 13.   | [How do you implement temporary access links for private resources?](#question-13-how-do-you-implement-temporary-access-links-for-private-resources)                        |
| 14.   | [What are webhook endpoints and how do you secure them?](#question-14-what-are-webhook-endpoints-and-how-do-you-secure-them)                                                |
| 15.   | [How do you validate webhook signatures?](#question-15-how-do-you-validate-webhook-signatures)                                                                              |
| 16.   | [What are common challenges with third-party API integrations?](#question-16-what-are-common-challenges-with-third-party-api-integrations)                                  |
| 17.   | [How do you safely retry failed external API requests?](#question-17-how-do-you-safely-retry-failed-external-api-requests)                                                  |
| 18.   | [What is connection pooling and why does it matter for Express.js applications?](#question-18-what-is-connection-pooling-and-why-does-it-matter-for-expressjs-applications) |
| 19.   | [How do database connection leaks affect backend systems?](#question-19-how-do-database-connection-leaks-affect-backend-systems)                                            |
| 20.   | [What are the risks of excessive logging in production APIs?](#question-20-what-are-the-risks-of-excessive-logging-in-production-apis)                                      |

## Question 1. How do you handle concurrent updates to the same resource?

## Direct Answer

Handling concurrent updates to the same resource involves ensuring that multiple clients don't accidentally overwrite each other's changes. The most common strategies are:

- **Optimistic locking (recommended for most web APIs)** using version numbers or timestamps.
- **Pessimistic locking** by locking the resource during updates.
- **Database transactions** for atomic operations.
- **Conditional requests** using HTTP `ETag` and `If-Match` headers.
- **Atomic database operations** for counters and simple updates.
- **Conflict detection and appropriate HTTP status codes** such as **409 Conflict** or **412 Precondition Failed**.

In Express.js, concurrency control is typically implemented at the **database layer**, while Express handles request validation, error handling, and HTTP responses.

---

# Detailed Explanation

Express.js itself is stateless and does not provide built-in concurrency control. Since multiple requests can reach the server simultaneously, the application must rely on the database or storage system to prevent race conditions.

Example scenario:

```
Initial balance = 1000

User A reads balance = 1000
User B reads balance = 1000

User A withdraws 300
User B withdraws 500

Without concurrency control:
Final balance may incorrectly become 500 or 700
instead of 200.
```

---

# 1. Optimistic Locking (Most Common)

Optimistic locking assumes conflicts are rare.

Each record contains a **version** (or `updatedAt`) field.

Example record:

```json
{
  "id": 1,
  "title": "Node Guide",
  "version": 5
}
```

Client receives:

```json
{
  "id": 1,
  "title": "Node Guide",
  "version": 5
}
```

Update request:

```http
PUT /articles/1
```

```json
{
  "title": "Updated Guide",
  "version": 5
}
```

Database query:

```sql
UPDATE articles
SET title = ?, version = version + 1
WHERE id = ? AND version = 5;
```

If:

```
affectedRows == 0
```

then another client already updated the record.

Return:

```http
409 Conflict
```

or

```http
412 Precondition Failed
```

depending on the API design.

### Express Example

```javascript
app.put("/articles/:id", async (req, res) => {
  const { version, title } = req.body;

  const updated = await Article.update(
    {
      title,
      version: version + 1,
    },
    {
      where: {
        id: req.params.id,
        version,
      },
    },
  );

  if (updated[0] === 0) {
    return res.status(409).json({
      error: "Resource was modified by another request",
    });
  }

  res.json({
    message: "Updated successfully",
  });
});
```

---

# 2. HTTP Conditional Requests (ETag)

REST APIs often use **ETags** to prevent lost updates.

Client fetches:

```http
GET /users/10
```

Response:

```http
ETag: "abc123"
```

Client updates:

```http
PUT /users/10
If-Match: "abc123"
```

Server compares the supplied ETag with the current version.

If they differ:

```http
412 Precondition Failed
```

This is a standard HTTP approach for optimistic concurrency control.

---

# 3. Database Transactions

Transactions ensure multiple operations succeed or fail together.

Example:

```javascript
await sequelize.transaction(async (t) => {
  // Read
  // Update inventory
  // Create order
  // Update payment
});
```

Benefits:

- Atomic operations
- Prevents partial updates
- Maintains data consistency

---

# 4. Pessimistic Locking

Instead of detecting conflicts later, lock the row before updating.

SQL example:

```sql
SELECT * FROM accounts
WHERE id = 1
FOR UPDATE;
```

Other transactions must wait until the lock is released.

Useful when:

- Banking
- Inventory systems
- Seat booking
- Financial transactions

Downside:

- Reduced concurrency
- Potential deadlocks
- Longer wait times

---

# 5. Atomic Database Operations

Many race conditions can be avoided with atomic updates.

Instead of:

```javascript
balance = balance - 100;
```

Use:

```sql
UPDATE accounts
SET balance = balance - 100
WHERE id = 1;
```

Or with MongoDB:

```javascript
await Account.updateOne({ _id: id }, { $inc: { balance: -100 } });
```

Atomic operations eliminate the need for separate read and write steps.

---

# 6. Conflict Detection

If concurrent modifications occur:

Return meaningful responses:

```http
409 Conflict
```

Example:

```json
{
  "error": "Resource has been modified by another client."
}
```

Or:

```http
412 Precondition Failed
```

when using conditional requests (`If-Match`/ETag).

This allows the client to fetch the latest version and retry if appropriate.

---

# Common Real-World Examples

### Inventory

Only one customer should purchase the last item.

Solution:

- Transaction
- Row lock
- Atomic stock decrement

---

### Bank Account

Two withdrawals occur simultaneously.

Solution:

- Transaction
- Row locking
- Serializable isolation level when necessary

---

### Collaborative Editing

Two users edit the same document.

Solution:

- Version field
- ETag
- Merge strategy if appropriate

---

### Profile Updates

Two browser tabs edit the same profile.

Solution:

- Optimistic locking
- Version number
- Return conflict if stale data is submitted

---

# Best Practices

- Use **optimistic locking** for most REST APIs because conflicts are usually infrequent and it scales well.
- Perform concurrency control in the **database layer**, not in Express memory, especially if your application runs on multiple processes or servers.
- Use **database transactions** for operations involving multiple related writes.
- Prefer **atomic database operations** (such as increment/decrement statements) whenever possible.
- Support **HTTP conditional requests** with `ETag` and `If-Match` for standards-compliant REST APIs.
- Return appropriate status codes (`409 Conflict` or `412 Precondition Failed`) so clients can detect and resolve update conflicts.
- Keep transactions as short as possible to minimize lock contention and improve scalability.

---

# Common Pitfalls

- Reading a value, modifying it in application code, and writing it back without checking for concurrent changes (the "lost update" problem).
- Relying on in-memory locks in an Express application deployed across multiple instances; these do not work reliably in distributed environments.
- Holding database locks for too long, which increases contention and the risk of deadlocks.
- Ignoring version or ETag validation, allowing stale clients to overwrite newer data.
- Assuming Node.js's single-threaded event loop prevents concurrent updates—it does not. Multiple requests can interleave, and concurrency issues primarily arise when accessing shared resources like databases.

---

# Interview Tip

A strong senior-level answer is:

> "Express.js doesn't manage concurrent updates itself; concurrency control belongs primarily in the persistence layer. For most REST APIs, I'd use optimistic locking with a version field or HTTP ETags and `If-Match` headers to prevent lost updates. For critical operations like banking or inventory, I'd use database transactions with row-level locking (`SELECT ... FOR UPDATE`) or atomic update operations. If a conflict occurs, I'd return `409 Conflict` or `412 Precondition Failed` so the client can retrieve the latest resource and retry safely."

## Question 2. What are the trade-offs between REST and GraphQL when using Express.js?

## Question 3. How would you integrate GraphQL into an Express.js application?

## Question 4. What are the performance implications of GraphQL resolvers?

## Question 5. How do you secure GraphQL endpoints in Express.js?

## Question 6. What is request payload size limiting and why is it important?

## Question 7. How do you prevent denial-of-service attacks caused by large payloads?

## Question 8. What is API gateway authentication delegation?

## Question 9. How do reverse proxies interact with Express.js applications?

## Question 10. What is the purpose of `trust proxy` in Express.js?

## Question 11. How do you retrieve the real client IP behind a proxy?

## Question 12. What are signed URLs and where are they useful?

## Question 13. How do you implement temporary access links for private resources?

## Question 14. What are webhook endpoints and how do you secure them?

## Question 15. How do you validate webhook signatures?

## Question 16. What are common challenges with third-party API integrations?

## Question 17. How do you safely retry failed external API requests?

## Question 18. What is connection pooling and why does it matter for Express.js applications?

## Question 19. How do database connection leaks affect backend systems?

## Question 20. What are the risks of excessive logging in production APIs?
