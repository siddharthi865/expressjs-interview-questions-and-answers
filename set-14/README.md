# Set 14

| S.No. | Question                                                                                                                                                                                 |
| ----- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 1.    | [How would you design a plugin architecture for Express.js applications?](#question-1-how-would-you-design-a-plugin-architecture-for-expressjs-applications)                             |
| 2.    | [How do middleware composition patterns improve maintainability?](#question-2-how-do-middleware-composition-patterns-improve-maintainability)                                            |
| 3.    | [What are the trade-offs between Express.js and highly opinionated backend frameworks?](#question-3-what-are-the-trade-offs-between-expressjs-and-highly-opinionated-backend-frameworks) |
| 4.    | [How would you benchmark Express.js API performance?](#question-4-how-would-you-benchmark-expressjs-api-performance)                                                                     |
| 5.    | [What metrics are most important for backend API monitoring?](#question-5-what-metrics-are-most-important-for-backend-api-monitoring)                                                    |
| 6.    | [How do you analyze slow API endpoints in production?](#question-6-how-do-you-analyze-slow-api-endpoints-in-production)                                                                  |
| 7.    | [What are flame graphs and how are they useful in Node.js performance tuning?](#question-7-what-are-flame-graphs-and-how-are-they-useful-in-nodejs-performance-tuning)                   |
| 8.    | [How would you detect memory fragmentation issues in Node.js applications?](#question-8-how-would-you-detect-memory-fragmentation-issues-in-nodejs-applications)                         |
| 9.    | [How do connection timeouts impact Express.js scalability?](#question-9-how-do-connection-timeouts-impact-expressjs-scalability)                                                         |
| 10.   | [How do keep-alive connections improve backend performance?](#question-10-how-do-keep-alive-connections-improve-backend-performance)                                                     |
| 11.   | [What is head-of-line blocking and how can it affect APIs?](#question-11-what-is-head-of-line-blocking-and-how-can-it-affect-apis)                                                       |
| 12.   | [How do HTTP/2 features improve Express.js applications?](#question-12-how-do-http2-features-improve-expressjs-applications)                                                             |
| 13.   | [What challenges arise when migrating from HTTP/1.1 to HTTP/2?](#question-13-what-challenges-arise-when-migrating-from-http11-to-http2)                                                  |
| 14.   | [How do you implement streaming responses in Express.js?](#question-14-how-do-you-implement-streaming-responses-in-expressjs)                                                            |
| 15.   | [What are the advantages of chunked transfer encoding?](#question-15-what-are-the-advantages-of-chunked-transfer-encoding)                                                               |
| 16.   | [How do you handle large CSV or video streaming in Express.js?](#question-16-how-do-you-handle-large-csv-or-video-streaming-in-expressjs)                                                |
| 17.   | [How would you design a resilient file upload service?](#question-17-how-would-you-design-a-resilient-file-upload-service)                                                               |
| 18.   | [How do distributed locks work in backend systems?](#question-18-how-do-distributed-locks-work-in-backend-systems)                                                                       |
| 19.   | [How would you prevent race conditions in distributed Express.js APIs?](#question-19-how-would-you-prevent-race-conditions-in-distributed-expressjs-apis)                                |
| 20.   | [What are the challenges of distributed caching invalidation?](#question-20-what-are-the-challenges-of-distributed-caching-invalidation)                                                 |

## Question 1. How would you design a plugin architecture for Express.js applications?

## Direct Answer

A plugin architecture in Express.js is a modular design pattern where features are packaged as independent plugins that can register routes, middleware, services, configuration, event listeners, and startup logic with the main application. The core application provides a stable API, while plugins extend functionality without modifying the core codebase.

This approach improves maintainability, scalability, testability, and allows features to be added or removed with minimal changes.

---

# What is a Plugin Architecture?

A plugin architecture separates the **application core** from **feature modules**.

Instead of placing everything inside a single Express application:

```
app.js
 ├── Users
 ├── Orders
 ├── Payments
 ├── Analytics
 ├── Admin
```

Each feature becomes an independent plugin:

```
app.js
plugins/
    users/
    payments/
    analytics/
    admin/
```

Each plugin exposes a standard interface that the application can load dynamically.

---

# Typical Responsibilities of a Plugin

A plugin may register:

- Express routes
- Middleware
- Services
- Database models
- Scheduled jobs
- Event listeners
- Configuration
- Dependency injection registrations
- Startup/shutdown hooks

For example:

```
Payments Plugin

✓ Routes
✓ Authentication middleware
✓ Payment service
✓ Webhook endpoint
✓ Event handlers
✓ Config validation
```

---

# Basic Plugin Interface

A common pattern is exporting a function.

```javascript
// plugins/users/index.js

module.exports = function (app) {
  app.get("/users", (req, res) => {
    res.json(["Alice", "Bob"]);
  });
};
```

Main application:

```javascript
const express = require("express");
const app = express();

require("./plugins/users")(app);

app.listen(3000);
```

---

# A More Structured Plugin

Instead of exporting only a function, export metadata.

```javascript
module.exports = {
  name: "users",

  register(app) {
    app.get("/users", (req, res) => {
      res.send("Users");
    });
  },
};
```

Loader:

```javascript
const plugin = require("./plugins/users");

plugin.register(app);
```

This allows adding:

- version
- dependencies
- configuration
- lifecycle hooks

Example:

```javascript
module.exports = {
  name: "payments",
  version: "1.0.0",
  dependencies: ["users"],

  register(app) {
    // register routes
  },
};
```

---

# Plugin Folder Structure

```
plugins/
    users/
        index.js
        routes.js
        service.js
        middleware.js
        config.js

    payments/
        index.js
        routes.js
        service.js
        webhook.js
```

Each plugin remains self-contained.

---

# Registering Routes

A plugin should own its router.

```javascript
const express = require("express");
const router = express.Router();

router.get("/", (req, res) => {
  res.send("Users");
});

module.exports = router;
```

Plugin:

```javascript
const router = require("./routes");

module.exports = {
  register(app) {
    app.use("/users", router);
  },
};
```

---

# Plugin Loader

Instead of manually importing every plugin:

```javascript
const fs = require("fs");
const path = require("path");

const pluginDir = path.join(__dirname, "plugins");

fs.readdirSync(pluginDir).forEach((folder) => {
  const plugin = require(path.join(pluginDir, folder));

  plugin.register(app);
});
```

Now adding a plugin only requires creating a new folder.

---

# Passing a Context Object

Instead of passing only `app`, pass a context.

```javascript
plugin.register({
  app,
  config,
  logger,
  db,
});
```

Plugin:

```javascript
module.exports = {
  register({ app, logger }) {
    app.get("/health", (req, res) => {
      logger.info("Health endpoint called");
      res.send("OK");
    });
  },
};
```

This reduces tight coupling.

---

# Dependency Injection

Plugins should avoid creating global resources.

Bad:

```javascript
const db = new Database();
```

Better:

```javascript
module.exports = {
  register({ db }) {},
};
```

The application manages shared dependencies.

---

# Plugin Configuration

Each plugin can define its own configuration.

```javascript
module.exports = {
  name: "payments",

  defaults: {
    currency: "USD",
    timeout: 5000,
  },

  register({ app, config }) {},
};
```

Application:

```javascript
const config = {
  payments: {
    currency: "EUR",
  },
};
```

---

# Plugin Lifecycle Hooks

A mature plugin system often supports lifecycle methods.

```javascript
module.exports = {
  async init(context) {},

  register(context) {},

  async destroy(context) {},
};
```

Example:

```
Application Start

↓

Load plugins

↓

Initialize database

↓

Register routes

↓

Start server

↓

Shutdown

↓

Destroy plugins
```

---

# Handling Plugin Dependencies

Sometimes one plugin depends on another.

Example:

```
Payments
      │
      ▼
Users
      │
      ▼
Authentication
```

Plugin:

```javascript
module.exports = {
  name: "payments",
  dependencies: ["users", "auth"],
};
```

The loader can:

- validate dependencies
- sort plugins
- detect circular dependencies
- fail early if required plugins are missing

---

# Plugin Isolation

Plugins should avoid modifying global application state unnecessarily.

Avoid:

```javascript
app.locals.user = ...
```

Prefer:

```javascript
context.services.userService;
```

Encapsulate:

- routes
- services
- middleware
- configuration

---

# Middleware Registration

Plugins may register middleware.

```javascript
register({ app }) {
    app.use(require("./middleware/logger"));
}
```

Or plugin-specific middleware:

```javascript
app.use("/payments", paymentMiddleware);
```

---

# Event-Based Communication

Instead of plugins calling each other directly, use an event bus.

```javascript
eventBus.emit("user.created", user);
```

Another plugin:

```javascript
eventBus.on("user.created", sendWelcomeEmail);
```

Advantages:

- loose coupling
- extensibility
- easier testing

---

# Auto Discovery

Plugins can be discovered automatically.

```
plugins/
    auth/
    users/
    payments/
    notifications/
```

Loader:

```
Read directory

↓

Import plugin

↓

Validate interface

↓

Load configuration

↓

Register

↓

Initialize
```

This is common in large systems.

---

# Version Compatibility

Plugins should specify supported application versions.

```javascript
module.exports = {
  engine: "^2.0.0",
};
```

The loader checks compatibility before registration.

---

# Error Handling

Plugin loading should be isolated.

```javascript
try {
  plugin.register(context);
} catch (err) {
  logger.error(err);
}
```

For critical plugins, stop startup.

For optional plugins, continue while logging the failure.

---

# Testing Plugins

Each plugin can be tested independently.

```javascript
const request = require("supertest");

describe("Users Plugin", () => {
  it("returns users", async () => {});
});
```

The core application doesn't need to be fully started.

---

# Advantages

- Modular architecture
- Easy feature addition/removal
- Better separation of concerns
- Independent testing
- Improved maintainability
- Dynamic feature loading
- Easier code ownership across teams
- Better scalability for large applications

---

# Potential Challenges

- Managing plugin dependencies and load order
- Version compatibility between plugins and the core application
- Defining a stable plugin API to avoid breaking changes
- Debugging interactions across many plugins
- Performance overhead if too many plugins initialize synchronously
- Preventing plugins from mutating shared state unexpectedly

---

# Best Practices

- Define a clear plugin contract (e.g., `register`, `init`, `destroy`).
- Pass shared resources (logger, database, configuration, cache) through a context object instead of relying on globals.
- Keep plugins self-contained, owning their routes, middleware, services, and configuration.
- Use an Express `Router` within each plugin rather than registering routes directly on the main app.
- Validate configuration and dependencies during startup and fail fast for required plugins.
- Support lifecycle hooks for initialization and graceful shutdown.
- Prefer event-driven communication or dependency injection over direct plugin-to-plugin calls to reduce coupling.
- Document the plugin API and use semantic versioning to maintain compatibility as the application evolves.

---

# Interview Summary

> "In Express.js, I would design a plugin architecture by defining a standard plugin interface with lifecycle methods like `register`, `init`, and `destroy`. Each plugin would encapsulate its own router, middleware, services, configuration, and event handlers, while the application provides shared dependencies such as the Express app, logger, database, and configuration through a context object. A plugin loader would dynamically discover plugins, validate dependencies and compatibility, and register them in the correct order. This modular design improves scalability, maintainability, testing, and allows features to be enabled, disabled, or extended without modifying the application's core."

## Question 2. How do middleware composition patterns improve maintainability?

## Question 3. What are the trade-offs between Express.js and highly opinionated backend frameworks?

## Question 4. How would you benchmark Express.js API performance?

## Question 5. What metrics are most important for backend API monitoring?

## Question 6. How do you analyze slow API endpoints in production?

## Question 7. What are flame graphs and how are they useful in Node.js performance tuning?

## Question 8. How would you detect memory fragmentation issues in Node.js applications?

## Question 9. How do connection timeouts impact Express.js scalability?

## Question 10. How do keep-alive connections improve backend performance?

## Question 11. What is head-of-line blocking and how can it affect APIs?

## Question 12. How do HTTP/2 features improve Express.js applications?

## Question 13. What challenges arise when migrating from HTTP/1.1 to HTTP/2?

## Question 14. How do you implement streaming responses in Express.js?

## Question 15. What are the advantages of chunked transfer encoding?

## Question 16. How do you handle large CSV or video streaming in Express.js?

## Question 17. How would you design a resilient file upload service?

## Question 18. How do distributed locks work in backend systems?

## Question 19. How would you prevent race conditions in distributed Express.js APIs?

## Question 20. What are the challenges of distributed caching invalidation?
