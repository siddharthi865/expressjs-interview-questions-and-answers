# Set 8

| S.No. | Question                                                                                                                                                                                         |
| ----- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| 1.    | [How do you design consistent RESTful endpoints?](#question-1-how-do-you-design-consistent-restful-endpoints)                                                                                    |
| 2.    | [What is HATEOAS and is it commonly used in Express.js APIs?](#question-2-what-is-hateoas-and-is-it-commonly-used-in-expressjs-apis)                                                             |
| 3.    | [How do you implement filtering and sorting in APIs?](#question-3-how-do-you-implement-filtering-and-sorting-in-apis)                                                                            |
| 4.    | [What are the benefits of cursor-based pagination over offset pagination?](#question-4-what-are-the-benefits-of-cursor-based-pagination-over-offset-pagination)                                  |
| 5.    | [How do you prevent over-fetching and under-fetching in REST APIs?](#question-5-how-do-you-prevent-over-fetching-and-under-fetching-in-rest-apis)                                                |
| 6.    | [What is API throttling and how is it different from rate limiting?](#question-6-what-is-api-throttling-and-how-is-it-different-from-rate-limiting)                                              |
| 7.    | [How do you handle long-running tasks in Express.js APIs?](#question-7-how-do-you-handle-long-running-tasks-in-expressjs-apis)                                                                   |
| 8.    | [How would you integrate background job queues with Express.js?](#question-8-how-would-you-integrate-background-job-queues-with-expressjs)                                                       |
| 9.    | [What are the use cases for Redis in Express.js applications?](#question-9-what-are-the-use-cases-for-redis-in-expressjs-applications)                                                           |
| 10.   | [How do you invalidate cache correctly in distributed systems?](#question-10-how-do-you-invalidate-cache-correctly-in-distributed-systems)                                                       |
| 11.   | [What are signed cookies and when should they be used?](#question-11-what-are-signed-cookies-and-when-should-they-be-used)                                                                       |
| 12.   | [How do secure cookies differ from HTTP-only cookies?](#question-12-how-do-secure-cookies-differ-from-http-only-cookies)                                                                         |
| 13.   | [How do you implement refresh token rotation securely?](#question-13-how-do-you-implement-refresh-token-rotation-securely)                                                                       |
| 14.   | [What are common JWT vulnerabilities developers introduce accidentally?](#question-14-what-are-common-jwt-vulnerabilities-developers-introduce-accidentally)                                     |
| 15.   | [How do you handle token expiration in APIs?](#question-15-how-do-you-handle-token-expiration-in-apis)                                                                                           |
| 16.   | [What are replay attacks and how can APIs defend against them?](#question-16-what-are-replay-attacks-and-how-can-apis-defend-against-them)                                                       |
| 17.   | [How would you implement audit logging in Express.js systems?](#question-17-how-would-you-implement-audit-logging-in-expressjs-systems)                                                          |
| 18.   | [How do you safely expose error details in production APIs?](#question-18-how-do-you-safely-expose-error-details-in-production-apis)                                                             |
| 19.   | [What are feature flags and how can they be used in backend systems?](#question-19-what-are-feature-flags-and-how-can-they-be-used-in-backend-systems)                                           |
| 20.   | [How do you manage configuration across development, staging, and production environments?](#question-20-how-do-you-manage-configuration-across-development-staging-and-production-environments) |

## Question 1. How do you design consistent RESTful endpoints?

## Direct Answer

Designing consistent RESTful endpoints means using predictable URL structures, appropriate HTTP methods, standard status codes, consistent naming conventions, and uniform request/response formats. A well-designed REST API should be intuitive so that clients can easily understand and use it without extensive documentation.

---

# Detailed Explanation

Consistency is one of the most important qualities of a REST API. It improves readability, maintainability, scalability, and developer experience.

## 1. Use Nouns, Not Verbs

Endpoints should represent **resources**, while HTTP methods define the action.

### ✅ Good

```http
GET    /users
GET    /users/123
POST   /users
PUT    /users/123
PATCH  /users/123
DELETE /users/123
```

### ❌ Bad

```http
GET    /getUsers
POST   /createUser
POST   /deleteUser
```

The URL identifies the resource, and the HTTP method specifies the operation.

---

## 2. Use Proper HTTP Methods

| Method | Purpose                   | Idempotent |
| ------ | ------------------------- | ---------- |
| GET    | Retrieve data             | ✅         |
| POST   | Create resource           | ❌         |
| PUT    | Replace entire resource   | ✅         |
| PATCH  | Partially update resource | ❌         |
| DELETE | Remove resource           | ✅         |

Example:

```http
GET    /products
POST   /products
GET    /products/25
PATCH  /products/25
DELETE /products/25
```

---

## 3. Use Plural Resource Names

Prefer plural nouns for collections.

```http
/users
/orders
/products
/books
```

Instead of

```http
/user
/order
/product
```

This makes endpoints more predictable.

---

## 4. Use Hierarchical URLs for Relationships

Represent relationships naturally.

```http
/users/15/orders
/users/15/orders/8
```

Instead of

```http
/getOrdersForUser?id=15
```

Example:

```http
GET /users/10/posts
```

Returns all posts created by user 10.

---

## 5. Keep URLs Simple

Avoid unnecessary nesting.

Good:

```http
/orders/5/items
```

Too deep:

```http
/customers/10/orders/5/items/2/reviews/3/comments
```

Generally, avoid nesting more than two or three levels unless it clearly reflects the resource hierarchy.

---

## 6. Use Query Parameters for Filtering

Don't create separate endpoints for every filter.

Good:

```http
GET /products?category=laptop
GET /products?priceMin=1000
GET /products?sort=price
GET /products?page=2&limit=20
```

Not:

```http
GET /products/category/laptop
GET /cheapProducts
```

---

## 7. Standardize Response Format

Example success response:

```json
{
  "success": true,
  "data": {
    "id": 1,
    "name": "John"
  }
}
```

Collection response:

```json
{
  "success": true,
  "count": 2,
  "data": [
    {
      "id": 1,
      "name": "John"
    },
    {
      "id": 2,
      "name": "Jane"
    }
  ]
}
```

Error response:

```json
{
  "success": false,
  "message": "User not found",
  "error": {
    "code": "USER_NOT_FOUND"
  }
}
```

Keeping a consistent response structure simplifies client-side development.

---

## 8. Use Appropriate HTTP Status Codes

| Status | Meaning                 |
| ------ | ----------------------- |
| 200    | Success                 |
| 201    | Resource created        |
| 204    | Success with no content |
| 400    | Bad request             |
| 401    | Unauthorized            |
| 403    | Forbidden               |
| 404    | Resource not found      |
| 409    | Conflict                |
| 422    | Validation failed       |
| 500    | Internal server error   |

Example:

```javascript
app.post("/users", (req, res) => {
  const user = createUser(req.body);

  res.status(201).json(user);
});
```

---

## 9. Version Your API

Common approaches:

```http
/api/v1/users
/api/v2/users
```

This allows you to introduce breaking changes without affecting existing clients.

---

## 10. Keep Naming Consistent

Choose one naming style and use it throughout the API.

Examples:

```http
/users
/userProfiles
```

or

```http
/users
/user-profiles
```

Avoid mixing styles:

```http
/user_profiles
/userProfiles
/user-profiles
```

A common convention is to use lowercase, hyphen-separated paths:

```http
/user-profiles
/order-items
```

---

## 11. Make Endpoints Predictable

Once a pattern is established, apply it consistently.

```http
GET    /users
GET    /products
GET    /orders

POST   /users
POST   /products
POST   /orders

DELETE /users/:id
DELETE /products/:id
DELETE /orders/:id
```

Clients shouldn't have to guess endpoint structures.

---

## 12. Use Middleware for Cross-Cutting Concerns

Avoid repeating logic in each route.

```javascript
app.use(express.json());
app.use(authMiddleware);
app.use(loggerMiddleware);
```

Routes remain focused on business logic:

```javascript
app.get("/users", getUsers);
app.post("/users", createUser);
```

---

## Express.js Example

```javascript
const express = require("express");
const app = express();

app.use(express.json());

app.get("/users", getUsers);
app.get("/users/:id", getUser);

app.post("/users", createUser);

app.put("/users/:id", replaceUser);

app.patch("/users/:id", updateUser);

app.delete("/users/:id", deleteUser);
```

This structure is clean, predictable, and easy to maintain.

---

# Best Practices

- Use resource-oriented URLs (nouns instead of verbs).
- Use the correct HTTP methods for each operation.
- Keep URL naming conventions consistent (prefer lowercase and hyphens).
- Use plural resource names.
- Use query parameters for filtering, sorting, searching, and pagination.
- Return meaningful HTTP status codes.
- Maintain a consistent JSON response and error format.
- Version APIs before introducing breaking changes.
- Validate incoming requests and return clear validation errors.
- Document the API (e.g., with OpenAPI/Swagger) so consumers understand available endpoints and schemas.
- Keep business logic out of route handlers by using controllers, services, and middleware.

---

# Common Pitfalls

- Using verbs in endpoint names (`/createUser`, `/deleteProduct`).
- Returning `200 OK` for every response, including errors.
- Mixing singular and plural resource names.
- Inconsistent response structures across endpoints.
- Embedding actions in URLs instead of using HTTP methods.
- Overly deep nested routes that are difficult to understand.
- Ignoring pagination for collection endpoints, leading to large responses.
- Exposing internal implementation details in URLs or error messages.

---

# Alternative Approaches

| Approach         | Advantages                                  | Disadvantages                                       |
| ---------------- | ------------------------------------------- | --------------------------------------------------- |
| Traditional REST | Standard, widely understood, cache-friendly | May require multiple requests for related resources |
| GraphQL          | Clients request exactly the data they need  | More complex server implementation and caching      |
| RPC-style APIs   | Simple for action-oriented operations       | Less resource-oriented and less RESTful             |

For most Express.js applications, a well-designed REST API with consistent resource naming, proper HTTP methods, standardized responses, and clear versioning provides the best balance of simplicity, maintainability, and interoperability.

---

## Interview Tip

In an interview, emphasize that **consistency is more important than personal preference**. A good REST API uses predictable resource-oriented URLs, correct HTTP methods, standard status codes, consistent request and response formats, validation, pagination, versioning, and centralized middleware. These practices make the API easier to consume, test, document, and maintain as it grows.

## Question 2. What is HATEOAS and is it commonly used in Express.js APIs?

## Question 3. How do you implement filtering and sorting in APIs?

## Question 4. What are the benefits of cursor-based pagination over offset pagination?

## Question 5. How do you prevent over-fetching and under-fetching in REST APIs?

## Question 6. What is API throttling and how is it different from rate limiting?

## Question 7. How do you handle long-running tasks in Express.js APIs?

## Question 8. How would you integrate background job queues with Express.js?

## Question 9. What are the use cases for Redis in Express.js applications?

## Question 10. How do you invalidate cache correctly in distributed systems?

## Question 11. What are signed cookies and when should they be used?

## Question 12. How do secure cookies differ from HTTP-only cookies?

## Question 13. How do you implement refresh token rotation securely?

## Question 14. What are common JWT vulnerabilities developers introduce accidentally?

## Question 15. How do you handle token expiration in APIs?

## Question 16. What are replay attacks and how can APIs defend against them?

## Question 17. How would you implement audit logging in Express.js systems?

## Question 18. How do you safely expose error details in production APIs?

## Question 19. What are feature flags and how can they be used in backend systems?

## Question 20. How do you manage configuration across development, staging, and production environments?
