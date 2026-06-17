# Set 24

| S.No. | Question                                                                                                                                                                         |
| ----- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 1.    | [How would you build a custom Express.js middleware engine from scratch?](#question-1-how-would-you-build-a-custom-expressjs-middleware-engine-from-scratch)                     |
| 2.    | [What are the internals of Express.js request and response prototypes?](#question-2-what-are-the-internals-of-expressjs-request-and-response-prototypes)                         |
| 3.    | [How does Express.js extend Node.js core request and response objects?](#question-3-how-does-expressjs-extend-nodejs-core-request-and-response-objects)                          |
| 4.    | [What are hidden performance costs of excessive middleware layers?](#question-4-what-are-hidden-performance-costs-of-excessive-middleware-layers)                                |
| 5.    | [How do V8 optimizations affect Express.js runtime performance?](#question-5-how-do-v8-optimizations-affect-expressjs-runtime-performance)                                       |
| 6.    | [What coding patterns can deoptimize V8 in backend applications?](#question-6-what-coding-patterns-can-deoptimize-v8-in-backend-applications)                                    |
| 7.    | [How do hidden classes in V8 influence Express.js object performance?](#question-7-how-do-hidden-classes-in-v8-influence-expressjs-object-performance)                           |
| 8.    | [What are memory retention paths and how do they cause leaks?](#question-8-what-are-memory-retention-paths-and-how-do-they-cause-leaks)                                          |
| 9.    | [How would you investigate heap growth over time in production?](#question-9-how-would-you-investigate-heap-growth-over-time-in-production)                                      |
| 10.   | [How do CPU-bound cryptographic operations affect Node.js scalability?](#question-10-how-do-cpu-bound-cryptographic-operations-affect-nodejs-scalability)                        |
| 11.   | [How do you offload expensive processing from the event loop safely?](#question-11-how-do-you-offload-expensive-processing-from-the-event-loop-safely)                           |
| 12.   | [What are zero-copy buffers and why are they useful in Node.js?](#question-12-what-are-zero-copy-buffers-and-why-are-they-useful-in-nodejs)                                      |
| 13.   | [How do TCP connection states impact backend API reliability?](#question-13-how-do-tcp-connection-states-impact-backend-api-reliability)                                         |
| 14.   | [What is socket exhaustion and how can it affect Express.js services?](#question-14-what-is-socket-exhaustion-and-how-can-it-affect-expressjs-services)                          |
| 15.   | [How do connection pools interact with Kubernetes autoscaling?](#question-15-how-do-connection-pools-interact-with-kubernetes-autoscaling)                                       |
| 16.   | [What are thundering herd problems in distributed systems?](#question-16-what-are-thundering-herd-problems-in-distributed-systems)                                               |
| 17.   | [How do token bucket and leaky bucket algorithms differ?](#question-17-how-do-token-bucket-and-leaky-bucket-algorithms-differ)                                                   |
| 18.   | [How would you implement distributed rate limiting with eventual consistency?](#question-18-how-would-you-implement-distributed-rate-limiting-with-eventual-consistency)         |
| 19.   | [What are the trade-offs between centralized and decentralized service discovery?](#question-19-what-are-the-trade-offs-between-centralized-and-decentralized-service-discovery) |
| 20.   | [How do sidecar proxies integrate with Express.js microservices?](#question-20-how-do-sidecar-proxies-integrate-with-expressjs-microservices)                                    |

## Question 1. How would you build a custom Express.js middleware engine from scratch?

## Direct Answer

A custom Express.js middleware engine can be built by maintaining a **stack (array) of middleware functions** and executing them sequentially using a `next()` function. Each middleware receives `(req, res, next)` and decides whether to process the request, modify it, terminate the response, or pass control to the next middleware. This is essentially the core idea behind how Express internally manages middleware.

---

# Detailed Explanation

Express.js is fundamentally a **middleware execution engine**. Every request passes through a chain of middleware before reaching the final route handler or error handler.

A simplified middleware engine requires:

- A middleware registration mechanism (`use()`)
- A middleware stack
- A request dispatcher
- A `next()` function
- Error propagation
- Route matching (optional but useful)

---

# Step 1: Create a Middleware Stack

Store middleware functions in an array.

```javascript
class MiniExpress {
  constructor() {
    this.middlewares = [];
  }

  use(fn) {
    this.middlewares.push(fn);
  }
}
```

Every call to:

```javascript
app.use(logger);
app.use(auth);
app.use(router);
```

adds another function into this array.

Internally it becomes:

```javascript
[logger, auth, router];
```

---

# Step 2: Execute Middleware Sequentially

Now implement a dispatcher.

```javascript
class MiniExpress {
  constructor() {
    this.middlewares = [];
  }

  use(fn) {
    this.middlewares.push(fn);
  }

  handle(req, res) {
    let index = 0;

    const next = () => {
      const middleware = this.middlewares[index++];

      if (!middleware) {
        return;
      }

      middleware(req, res, next);
    };

    next();
  }
}
```

Execution flow:

```
Request
   ↓
Middleware 1
   ↓ next()
Middleware 2
   ↓ next()
Middleware 3
   ↓ next()
End
```

---

# Step 3: Example Middleware

```javascript
const app = new MiniExpress();

app.use((req, res, next) => {
  console.log("Logger");
  next();
});

app.use((req, res, next) => {
  req.user = { name: "Alice" };
  next();
});

app.use((req, res) => {
  res.end(`Hello ${req.user.name}`);
});
```

Execution order:

```
Logger
↓
Attach user
↓
Send response
```

---

# Step 4: Support Async Middleware

Modern middleware is usually asynchronous.

```javascript
handle(req, res) {
    let index = 0;

    const next = async (err) => {
        const middleware = this.middlewares[index++];

        if (!middleware) return;

        await middleware(req, res, next);
    };

    next();
}
```

Example:

```javascript
app.use(async (req, res, next) => {
  await new Promise((resolve) => setTimeout(resolve, 100));

  console.log("Database loaded");

  next();
});
```

---

# Step 5: Error Handling

Express distinguishes normal middleware from error middleware.

Normal middleware:

```javascript
(req, res, next);
```

Error middleware:

```javascript
(err, req, res, next);
```

Dispatcher:

```javascript
const next = (err) => {
  const middleware = this.middlewares[index++];

  if (!middleware) return;

  if (err) {
    if (middleware.length === 4) {
      return middleware(err, req, res, next);
    }

    return next(err);
  }

  if (middleware.length < 4) {
    return middleware(req, res, next);
  }

  next();
};
```

Usage:

```javascript
app.use((req, res, next) => {
  next(new Error("Something failed"));
});

app.use((err, req, res, next) => {
  res.statusCode = 500;
  res.end(err.message);
});
```

---

# Step 6: Route Matching

Instead of storing only middleware:

```javascript
[fn1, fn2, fn3];
```

Store metadata.

```javascript
[
  {
    path: "/",
    method: "USE",
    handler: fn,
  },
  {
    path: "/users",
    method: "GET",
    handler: fn,
  },
];
```

Dispatcher:

```javascript
if (layer.path === req.url && layer.method === req.method) {
  layer.handler(req, res, next);
}
```

This is roughly how Express layers are matched.

---

# Step 7: Prefix Matching

Support:

```javascript
app.use("/api", middleware);
```

Logic:

```javascript
if (req.url.startsWith(layer.path)) {
  layer.handler(req, res, next);
}
```

Example:

```
/api/users
/api/products
/api/orders
```

All execute the middleware.

---

# Step 8: Add HTTP Methods

Implement helpers:

```javascript
get(path, handler) {
    this.middlewares.push({
        method: "GET",
        path,
        handler
    });
}

post(path, handler) {
    this.middlewares.push({
        method: "POST",
        path,
        handler
    });
}
```

Usage:

```javascript
app.get("/users", (req, res) => {
  res.end("Users");
});
```

---

# Step 9: Nested Routers

A router is simply another middleware engine.

```javascript
const router = new MiniExpress();

router.get("/users", handler);

app.use("/api", router.handle.bind(router));
```

Request:

```
GET /api/users
```

Flow:

```
App
 ↓
Router
 ↓
Route
```

This mirrors Express's router composition.

---

# Step 10: Middleware Flow

```
Incoming Request
        │
        ▼
+----------------+
| Middleware 1   |
+----------------+
        │ next()
        ▼
+----------------+
| Middleware 2   |
+----------------+
        │ next()
        ▼
+----------------+
| Middleware 3   |
+----------------+
        │ next()
        ▼
+----------------+
| Route Handler  |
+----------------+
        │
        ▼
   Response Sent
```

If an error occurs:

```
Request
   │
   ▼
Middleware
   │
next(error)
   ▼
Error Middleware
   │
Response
```

---

# How Express.js Does It Internally

Express maintains a stack of **Layer** objects. Each layer typically contains:

- Path
- HTTP method (for routes)
- Handler function
- Matching logic (exact path, prefix, or pattern)
- Whether it's a route or generic middleware

For every incoming request, Express iterates through the stack:

1. Checks whether the current layer matches the request path and method.
2. Invokes the handler if it matches.
3. Provides a `next()` callback to continue to the next matching layer.
4. Skips non-matching layers.
5. Switches to error-handling middleware when `next(err)` is called.
6. Ends with a 404 response if no route handles the request.

This linear traversal of a middleware stack is one of the reasons Express remains simple, extensible, and easy to reason about.

---

# Common Pitfalls

- **Forgetting to call `next()`** causes the request to hang unless the middleware sends a response.
- **Calling `next()` after sending a response** can result in "headers already sent" errors if later middleware also writes to the response.
- **Calling `next()` multiple times** may execute downstream middleware unexpectedly.
- **Not handling rejected promises** in asynchronous middleware can lead to unhandled errors. In modern Express (v5), rejected promises from middleware and route handlers are automatically forwarded to error-handling middleware; in Express 4, they typically require explicit forwarding (for example, `next(err)`).

---

# Best Practices

- Keep each middleware focused on a single responsibility (logging, authentication, validation, etc.).
- Ensure every code path either sends a response or calls `next()` exactly once.
- Place global middleware (logging, security, body parsing) before route handlers.
- Register error-handling middleware after all other middleware and routes.
- Design middleware to be reusable and composable rather than tightly coupled to specific routes.

---

# Interview Summary

A custom Express.js middleware engine is built around a **middleware stack** and a **dispatcher** that invokes each middleware in order using a `next()` callback. By extending this core with path and method matching, asynchronous support, nested routers, and dedicated error-handling middleware, you can recreate the fundamental execution model that powers Express.js itself.

## Question 2. What are the internals of Express.js request and response prototypes?

## Question 3. How does Express.js extend Node.js core request and response objects?

## Question 4. What are hidden performance costs of excessive middleware layers?

## Question 5. How do V8 optimizations affect Express.js runtime performance?

## Question 6. What coding patterns can deoptimize V8 in backend applications?

## Question 7. How do hidden classes in V8 influence Express.js object performance?

## Question 8. What are memory retention paths and how do they cause leaks?

## Question 9. How would you investigate heap growth over time in production?

## Question 10. How do CPU-bound cryptographic operations affect Node.js scalability?

## Question 11. How do you offload expensive processing from the event loop safely?

## Question 12. What are zero-copy buffers and why are they useful in Node.js?

## Question 13. How do TCP connection states impact backend API reliability?

## Question 14. What is socket exhaustion and how can it affect Express.js services?

## Question 15. How do connection pools interact with Kubernetes autoscaling?

## Question 16. What are thundering herd problems in distributed systems?

## Question 17. How do token bucket and leaky bucket algorithms differ?

## Question 18. How would you implement distributed rate limiting with eventual consistency?

## Question 19. What are the trade-offs between centralized and decentralized service discovery?

## Question 20. How do sidecar proxies integrate with Express.js microservices?
