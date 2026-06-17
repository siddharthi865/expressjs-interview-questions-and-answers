# Set 12

| S.No. | Question                                                                                                                                                                   |
| ----- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 1.    | [How do you expose API endpoints for health checks?](#question-1-how-do-you-expose-api-endpoints-for-health-checks)                                                        |
| 2.    | [What is the difference between development and production modes in Express.js?](#question-2-what-is-the-difference-between-development-and-production-modes-in-expressjs) |
| 3.    | [What are common folder structures for beginner Express.js projects?](#question-3-what-are-common-folder-structures-for-beginner-expressjs-projects)                       |
| 4.    | [How do you make an Express.js server listen on a custom port?](#question-4-how-do-you-make-an-expressjs-server-listen-on-a-custom-port)                                   |
| 5.    | [What are query strings and how are they used in APIs?](#question-5-what-are-query-strings-and-how-are-they-used-in-apis)                                                  |
| 6.    | [How do middleware short-circuit request execution in Express.js?](#question-6-how-do-middleware-short-circuit-request-execution-in-expressjs)                             |
| 7.    | [What is conditional middleware execution?](#question-7-what-is-conditional-middleware-execution)                                                                          |
| 8.    | [How would you build reusable authentication guards in Express.js?](#question-8-how-would-you-build-reusable-authentication-guards-in-expressjs)                           |
| 9.    | [How do you create parameter middleware using `router.param()`?](#question-9-how-do-you-create-parameter-middleware-using-routerparam)                                     |
| 10.   | [What are common mistakes developers make with async middleware?](#question-10-what-are-common-mistakes-developers-make-with-async-middleware)                             |
| 11.   | [How do you prevent duplicate middleware execution?](#question-11-how-do-you-prevent-duplicate-middleware-execution)                                                       |
| 12.   | [How would you design reusable response formatter utilities?](#question-12-how-would-you-design-reusable-response-formatter-utilities)                                     |
| 13.   | [What are DTOs and why are they useful in backend applications?](#question-13-what-are-dtos-and-why-are-they-useful-in-backend-applications)                               |
| 14.   | [How do you enforce consistent API response schemas?](#question-14-how-do-you-enforce-consistent-api-response-schemas)                                                     |
| 15.   | [What is the purpose of API contracts between frontend and backend teams?](#question-15-what-is-the-purpose-of-api-contracts-between-frontend-and-backend-teams)           |
| 16.   | [How do you implement partial updates using HTTP PATCH?](#question-16-how-do-you-implement-partial-updates-using-http-patch)                                               |
| 17.   | [What are idempotent HTTP methods?](#question-17-what-are-idempotent-http-methods)                                                                                         |
| 18.   | [Why should PUT requests generally be idempotent?](#question-18-why-should-put-requests-generally-be-idempotent)                                                           |
| 19.   | [How do you implement optimistic concurrency control in REST APIs?](#question-19-how-do-you-implement-optimistic-concurrency-control-in-rest-apis)                         |
| 20.   | [What are ETags and how do they help APIs?](#question-20-what-are-etags-and-how-do-they-help-apis)                                                                         |

## Question 1. How do you expose API endpoints for health checks?

## Direct Answer

Health check endpoints in Express.js are lightweight API routes that allow load balancers, orchestration platforms (such as Kubernetes), monitoring systems, and uptime tools to determine whether an application is running and ready to serve traffic.

The most common endpoints are:

- **`/health`** – General application health.
- **`/live`** or **`/health/live`** – Liveness check (is the process alive?).
- **`/ready`** or **`/health/ready`** – Readiness check (can the application handle requests?).

These endpoints should be fast, reliable, and return appropriate HTTP status codes.

---

# Detailed Explanation

Health check endpoints are an important part of production-ready Express.js applications. They help:

- Kubernetes decide whether to restart a container.
- Load balancers determine if an instance should receive traffic.
- Monitoring systems detect outages.
- CI/CD pipelines verify deployments.
- Developers quickly inspect application status.

---

# Basic Health Check Endpoint

The simplest implementation only verifies that the server is running.

```javascript
const express = require("express");

const app = express();

app.get("/health", (req, res) => {
  res.status(200).json({
    status: "UP",
    timestamp: new Date().toISOString(),
  });
});

app.listen(3000);
```

Response:

```json
{
  "status": "UP",
  "timestamp": "2026-06-18T08:30:00.000Z"
}
```

This confirms that:

- Express is running.
- The HTTP server is accepting requests.

---

# Liveness vs Readiness

Modern deployments distinguish between two different health checks.

## 1. Liveness Probe

Question:

> Is the application process alive?

It should **not** check external dependencies.

Example:

```javascript
app.get("/health/live", (req, res) => {
  res.sendStatus(200);
});
```

If this endpoint fails:

- Kubernetes restarts the container.

---

## 2. Readiness Probe

Question:

> Can this application currently serve requests?

Here you can verify critical dependencies such as:

- Database
- Redis
- Message queue
- External APIs (only if essential)

Example:

```javascript
app.get("/health/ready", async (req, res) => {
  try {
    await db.ping();

    res.json({
      status: "READY",
    });
  } catch (err) {
    res.status(503).json({
      status: "NOT_READY",
    });
  }
});
```

If readiness fails:

- The application remains running.
- Traffic is temporarily stopped until recovery.

---

# Returning Dependency Status

A more informative endpoint reports the health of important services.

```javascript
app.get("/health", async (req, res) => {
  const health = {
    uptime: process.uptime(),
    timestamp: new Date().toISOString(),
    database: "UP",
    redis: "UP",
  };

  try {
    await db.ping();
  } catch {
    health.database = "DOWN";
  }

  try {
    await redis.ping();
  } catch {
    health.redis = "DOWN";
  }

  const healthy = health.database === "UP" && health.redis === "UP";

  res.status(healthy ? 200 : 503).json(health);
});
```

Example response:

```json
{
  "uptime": 15420,
  "timestamp": "2026-06-18T08:30:00Z",
  "database": "UP",
  "redis": "DOWN"
}
```

---

# Appropriate HTTP Status Codes

| Status                        | Meaning                                              |
| ----------------------------- | ---------------------------------------------------- |
| **200 OK**                    | Healthy                                              |
| **204 No Content**            | Healthy with no response body                        |
| **503 Service Unavailable**   | Unhealthy or not ready                               |
| **500 Internal Server Error** | Unexpected failure while performing the health check |

Monitoring tools primarily rely on the HTTP status code.

---

# Useful Information to Include

A health endpoint may include:

```javascript
{
  status: "UP",
  uptime: process.uptime(),
  timestamp: new Date().toISOString(),
  version: process.env.APP_VERSION,
  environment: process.env.NODE_ENV,
  memory: process.memoryUsage()
}
```

Useful fields include:

- uptime
- version
- environment
- memory usage
- dependency status
- build number
- git commit hash

Avoid including sensitive information.

---

# Keep Health Checks Lightweight

Health endpoints should:

- Return within a few milliseconds.
- Avoid expensive database queries.
- Avoid long-running computations.
- Avoid blocking the event loop.
- Avoid calling non-essential third-party services.

Good:

```javascript
await db.ping();
```

Bad:

```javascript
await db.users.find({});
```

---

# Security Considerations

Health endpoints should **not** expose:

- Database passwords
- API keys
- Internal IP addresses
- Stack traces
- Configuration secrets

Bad:

```json
{
  "databasePassword": "secret123"
}
```

Good:

```json
{
  "database": "UP"
}
```

---

# Organizing Health Routes

A common project structure is:

```
routes/
    health.js

controllers/
    healthController.js

services/
    healthService.js
```

Example:

```javascript
// routes/health.js
const router = require("express").Router();

router.get("/", healthController.health);
router.get("/live", healthController.live);
router.get("/ready", healthController.ready);

module.exports = router;
```

---

# Integration with Kubernetes

Typical configuration:

```yaml
livenessProbe:
  httpGet:
    path: /health/live
    port: 3000

readinessProbe:
  httpGet:
    path: /health/ready
    port: 3000
```

Behavior:

- Liveness failure → restart the container.
- Readiness failure → remove the instance from service until it recovers.

---

# Best Practices

- Expose separate endpoints for liveness and readiness.
- Keep health checks lightweight and fast.
- Return `200` for healthy and `503` for unhealthy dependencies.
- Include useful metadata such as uptime, version, and timestamps.
- Avoid exposing secrets or internal implementation details.
- Use readiness checks for critical dependencies only.
- Ensure health checks are reliable and do not introduce significant load.

---

# Common Pitfalls

- Performing heavy database queries instead of lightweight connectivity checks.
- Checking every external dependency, causing slow or unreliable responses.
- Returning `200 OK` even when critical services are unavailable.
- Exposing sensitive configuration or debug information.
- Using a single endpoint for both liveness and readiness without considering their different purposes.

---

# Interview Tip

In an interview, emphasize that **production-grade Express.js applications typically expose separate liveness and readiness endpoints**. Liveness verifies that the process is running, while readiness verifies that the application can serve requests by checking essential dependencies (such as the database or cache). Health endpoints should be fast, lightweight, secure, and return appropriate HTTP status codes (`200` for healthy and `503` for unhealthy) so they integrate effectively with load balancers, monitoring systems, and orchestration platforms like Kubernetes.

## Question 2. What is the difference between development and production modes in Express.js?

## Question 3. What are common folder structures for beginner Express.js projects?

## Question 4. How do you make an Express.js server listen on a custom port?

## Question 5. What are query strings and how are they used in APIs?

## Question 6. How do middleware short-circuit request execution in Express.js?

## Question 7. What is conditional middleware execution?

## Question 8. How would you build reusable authentication guards in Express.js?

## Question 9. How do you create parameter middleware using `router.param()`?

## Question 10. What are common mistakes developers make with async middleware?

## Question 11. How do you prevent duplicate middleware execution?

## Question 12. How would you design reusable response formatter utilities?

## Question 13. What are DTOs and why are they useful in backend applications?

## Question 14. How do you enforce consistent API response schemas?

## Question 15. What is the purpose of API contracts between frontend and backend teams?

## Question 16. How do you implement partial updates using HTTP PATCH?

## Question 17. What are idempotent HTTP methods?

## Question 18. Why should PUT requests generally be idempotent?

## Question 19. How do you implement optimistic concurrency control in REST APIs?

## Question 20. What are ETags and how do they help APIs?
