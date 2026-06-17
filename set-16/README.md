# Set 16

| S.No. | Question                                                                                                                                                                                  |
| ----- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 1.    | [What is the purpose of `app.all()` in Express.js?](#question-1-what-is-the-purpose-of-appall-in-expressjs)                                                                               |
| 2.    | [How do you handle DELETE requests in Express.js?](#question-2-how-do-you-handle-delete-requests-in-expressjs)                                                                            |
| 3.    | [What is the difference between `res.end()` and `res.send()`?](#question-3-what-is-the-difference-between-resend-and-ressend)                                                             |
| 4.    | [How can you return XML responses from Express.js?](#question-4-how-can-you-return-xml-responses-from-expressjs)                                                                          |
| 5.    | [What is the purpose of route wildcards like `*` in Express.js?](#question-5-what-is-the-purpose-of-route-wildcards-like--in-expressjs)                                                   |
| 6.    | [How do you define regex-based routes in Express.js?](#question-6-how-do-you-define-regex-based-routes-in-expressjs)                                                                      |
| 7.    | [What is the difference between absolute and relative paths in Express.js routing?](#question-7-what-is-the-difference-between-absolute-and-relative-paths-in-expressjs-routing)          |
| 8.    | [How do you mount routers on specific paths?](#question-8-how-do-you-mount-routers-on-specific-paths)                                                                                     |
| 9.    | [What is the role of callback functions in route handlers?](#question-9-what-is-the-role-of-callback-functions-in-route-handlers)                                                         |
| 10.   | [How do you send HTTP status code 201 in Express.js?](#question-10-how-do-you-send-http-status-code-201-in-expressjs)                                                                     |
| 11.   | [What is the significance of HTTP status code 204?](#question-11-what-is-the-significance-of-http-status-code-204)                                                                        |
| 12.   | [How do you create reusable route modules in Express.js?](#question-12-how-do-you-create-reusable-route-modules-in-expressjs)                                                             |
| 13.   | [What are middleware execution chains?](#question-13-what-are-middleware-execution-chains)                                                                                                |
| 14.   | [How do you debug route matching issues in Express.js?](#question-14-how-do-you-debug-route-matching-issues-in-expressjs)                                                                 |
| 15.   | [What is the difference between JSON APIs and rendered web applications in Express.js?](#question-15-what-is-the-difference-between-json-apis-and-rendered-web-applications-in-expressjs) |
| 16.   | [How do you configure Express.js to work with frontend applications?](#question-16-how-do-you-configure-expressjs-to-work-with-frontend-applications)                                     |
| 17.   | [What are RESTful naming conventions for API endpoints?](#question-17-what-are-restful-naming-conventions-for-api-endpoints)                                                              |
| 18.   | [How do you access uploaded file metadata in Express.js?](#question-18-how-do-you-access-uploaded-file-metadata-in-expressjs)                                                             |
| 19.   | [What is the purpose of body parsing middleware?](#question-19-what-is-the-purpose-of-body-parsing-middleware)                                                                            |
| 20.   | [How do you create environment-specific configuration files?](#question-20-how-do-you-create-environment-specific-configuration-files)                                                    |

## Question 1. What is the purpose of `app.all()` in Express.js?

## Direct Answer

`app.all()` in Express.js is used to define a route handler that matches **all HTTP methods** (GET, POST, PUT, PATCH, DELETE, OPTIONS, HEAD, etc.) for a specific route. It is commonly used to execute shared logic—such as authentication, logging, validation, or middleware—before handling requests, regardless of the HTTP method.

---

# Detailed Explanation

Normally, Express provides method-specific routing:

```javascript
app.get("/users", (req, res) => {
  res.send("GET request");
});

app.post("/users", (req, res) => {
  res.send("POST request");
});
```

These handlers respond only to their respective HTTP methods.

`app.all()` is different because it listens to **every HTTP method** for the specified path.

```javascript
app.all("/users", (req, res) => {
  res.send(`Received a ${req.method} request`);
});
```

Requests like:

- GET /users
- POST /users
- PUT /users
- DELETE /users

will all execute the same handler.

---

# Syntax

```javascript
app.all(path, callback);
```

or with multiple middleware:

```javascript
app.all(path, middleware1, middleware2, handler);
```

---

# Common Use Cases

## 1. Authentication for All Methods

A very common interview example.

```javascript
function authenticate(req, res, next) {
  if (!req.headers.authorization) {
    return res.status(401).json({
      message: "Unauthorized",
    });
  }

  next();
}

app.all("/admin/*", authenticate);

app.get("/admin/users", (req, res) => {
  res.send("User List");
});

app.post("/admin/users", (req, res) => {
  res.send("User Created");
});
```

Every request under `/admin/*` must pass authentication regardless of whether it's GET, POST, PUT, or DELETE.

---

## 2. Logging Requests

```javascript
app.all("*", (req, res, next) => {
  console.log(`${req.method} ${req.originalUrl}`);
  next();
});
```

This logs every incoming request.

Although today, global middleware is more commonly registered with:

```javascript
app.use((req, res, next) => {
  console.log(`${req.method} ${req.originalUrl}`);
  next();
});
```

---

## 3. Shared Preprocessing

```javascript
app.all("/products/:id", (req, res, next) => {
  console.log("Checking product...");
  next();
});

app.get("/products/:id", (req, res) => {
  res.send("Product Details");
});

app.put("/products/:id", (req, res) => {
  res.send("Product Updated");
});
```

The `app.all()` middleware runs first, then control passes to the method-specific handler via `next()`.

---

## 4. Route-Level Middleware

```javascript
app.all("/api/*", rateLimiter, authenticate, (req, res, next) => {
  next();
});
```

Now every API endpoint under `/api` automatically uses:

- Rate limiting
- Authentication

before reaching its actual route handler.

---

# How `next()` Works with `app.all()`

Typically, `app.all()` is used as middleware.

```javascript
app.all("/users", (req, res, next) => {
  console.log("Common logic");
  next();
});

app.get("/users", (req, res) => {
  res.send("GET Users");
});
```

Flow:

```
GET /users
     │
     ▼
app.all()
     │
 next()
     │
     ▼
app.get()
     │
     ▼
Response
```

Without `next()`, Express will not continue to the next matching route.

---

# `app.all()` vs `app.use()`

| Feature                         | `app.all()`                     | `app.use()`                     |
| ------------------------------- | ------------------------------- | ------------------------------- |
| Matches all HTTP methods        | ✅                              | ✅                              |
| Supports route paths            | ✅                              | ✅                              |
| Executes only on matching route | ✅                              | ✅                              |
| Intended for route handling     | ✅                              | Primarily middleware            |
| Requires `next()` to continue   | Yes, unless it sends a response | Yes, unless it sends a response |
| Can terminate the request       | ✅                              | ✅                              |

### `app.use()`

```javascript
app.use("/api", authenticate);
```

Runs for every request under `/api`.

### `app.all()`

```javascript
app.all("/api/users", authenticate);
```

Runs only for that specific route (or matching pattern) across all HTTP methods.

In practice, `app.use()` is generally preferred for reusable middleware, while `app.all()` is useful when you want middleware or a handler tied to a specific route regardless of the HTTP method.

---

# `app.all()` vs Method-Specific Routes

```javascript
app.all("/users", handler);
```

Matches:

- GET
- POST
- PUT
- PATCH
- DELETE
- OPTIONS
- HEAD

Whereas:

```javascript
app.get("/users", handler);
```

Only matches:

- GET

---

# Common Pitfalls

### 1. Forgetting `next()`

```javascript
app.all("/users", (req, res) => {
  console.log("Executed");
});
```

If no response is sent and `next()` is not called, the request will hang.

Correct:

```javascript
app.all("/users", (req, res, next) => {
  console.log("Executed");
  next();
});
```

---

### 2. Accidentally Handling Every Method

```javascript
app.all("/users", (req, res) => {
  res.send("Hello");
});
```

Now POST, DELETE, PUT, etc., all receive the same response, which may not be what you intended. Use method-specific routes when different HTTP methods require different behavior.

---

### 3. Using `app.all()` Instead of `app.use()`

For application-wide concerns like:

- Authentication
- CORS
- Compression
- Body parsing
- Logging

prefer:

```javascript
app.use(...)
```

rather than:

```javascript
app.all('*', ...)
```

because `app.use()` is the conventional and more idiomatic middleware mechanism in Express.

---

# Best Practices

- Use `app.all()` for **route-specific logic** that should apply to every HTTP method.
- Use `next()` when the request should continue to subsequent handlers.
- Use `app.use()` for global or reusable middleware across many routes.
- Prefer method-specific routes (`app.get()`, `app.post()`, etc.) when different HTTP methods have distinct behaviors.
- Keep `app.all()` handlers focused on cross-cutting concerns (such as authentication or validation) rather than implementing all business logic in a single handler.

---

# Interview Summary

> **`app.all()` registers a route handler that matches all HTTP methods for a given path. It is commonly used to apply shared route-specific logic—such as authentication, logging, validation, or preprocessing—before delegating to method-specific handlers using `next()`. While `app.all()` is useful for logic tied to a particular route, `app.use()` is generally the preferred choice for global or reusable middleware.**

## Question 2. How do you handle DELETE requests in Express.js?

## Question 3. What is the difference between `res.end()` and `res.send()`?

## Question 4. How can you return XML responses from Express.js?

## Question 5. What is the purpose of route wildcards like `*` in Express.js?

## Question 6. How do you define regex-based routes in Express.js?

## Question 7. What is the difference between absolute and relative paths in Express.js routing?

## Question 8. How do you mount routers on specific paths?

## Question 9. What is the role of callback functions in route handlers?

## Question 10. How do you send HTTP status code 201 in Express.js?

## Question 11. What is the significance of HTTP status code 204?

## Question 12. How do you create reusable route modules in Express.js?

## Question 13. What are middleware execution chains?

## Question 14. How do you debug route matching issues in Express.js?

## Question 15. What is the difference between JSON APIs and rendered web applications in Express.js?

## Question 16. How do you configure Express.js to work with frontend applications?

## Question 17. What are RESTful naming conventions for API endpoints?

## Question 18. How do you access uploaded file metadata in Express.js?

## Question 19. What is the purpose of body parsing middleware?

## Question 20. How do you create environment-specific configuration files?
