# Set 9

| S.No. | Question                                                                                                                                                                                                  |
| ----- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 1.    | [How would you implement zero-downtime deployments for Express.js services?](#question-1-how-would-you-implement-zero-downtime-deployments-for-expressjs-services)                                        |
| 2.    | [What deployment challenges arise with stateful Express.js applications?](#question-2-what-deployment-challenges-arise-with-stateful-expressjs-applications)                                              |
| 3.    | [How do sticky sessions work with load balancers?](#question-3-how-do-sticky-sessions-work-with-load-balancers)                                                                                           |
| 4.    | [How do you horizontally scale WebSocket connections in Express.js systems?](#question-4-how-do-you-horizontally-scale-websocket-connections-in-expressjs-systems)                                        |
| 5.    | [What is the impact of blocking I/O on Node.js application performance?](#question-5-what-is-the-impact-of-blocking-io-on-nodejs-application-performance)                                                 |
| 6.    | [How do you profile CPU bottlenecks in Express.js applications?](#question-6-how-do-you-profile-cpu-bottlenecks-in-expressjs-applications)                                                                |
| 7.    | [How do you identify and debug event emitter memory leaks?](#question-7-how-do-you-identify-and-debug-event-emitter-memory-leaks)                                                                         |
| 8.    | [What tools would you use to monitor memory usage in production Node.js servers?](#question-8-what-tools-would-you-use-to-monitor-memory-usage-in-production-nodejs-servers)                              |
| 9.    | [How does garbage collection affect high-performance Express.js APIs?](#question-9-how-does-garbage-collection-affect-high-performance-expressjs-apis)                                                    |
| 10.   | [How do you tune Node.js memory limits for large-scale applications?](#question-10-how-do-you-tune-nodejs-memory-limits-for-large-scale-applications)                                                     |
| 11.   | [What are the advantages and disadvantages of monorepo architecture for Express.js services?](#question-11-what-are-the-advantages-and-disadvantages-of-monorepo-architecture-for-expressjs-services)     |
| 12.   | [How do you implement blue-green deployments for Express.js applications?](#question-12-how-do-you-implement-blue-green-deployments-for-expressjs-applications)                                           |
| 13.   | [How would you design a centralized authentication service shared by multiple Express.js APIs?](#question-13-how-would-you-design-a-centralized-authentication-service-shared-by-multiple-expressjs-apis) |
| 14.   | [How do API contracts help prevent breaking changes between services?](#question-14-how-do-api-contracts-help-prevent-breaking-changes-between-services)                                                  |
| 15.   | [What strategies can reduce cold start times in serverless Express.js deployments?](#question-15-what-strategies-can-reduce-cold-start-times-in-serverless-expressjs-deployments)                         |
| 16.   | [How do you secure internal service-to-service communication?](#question-16-how-do-you-secure-internal-service-to-service-communication)                                                                  |
| 17.   | [How would you implement mutual TLS authentication between microservices?](#question-17-how-would-you-implement-mutual-tls-authentication-between-microservices)                                          |
| 18.   | [What are the challenges of eventual consistency in distributed Express.js systems?](#question-18-what-are-the-challenges-of-eventual-consistency-in-distributed-expressjs-systems)                       |
| 19.   | [How do you implement saga patterns for distributed transactions?](#question-19-how-do-you-implement-saga-patterns-for-distributed-transactions)                                                          |
| 20.   | [How would you design retry-safe APIs for unreliable networks?](#question-20-how-would-you-design-retry-safe-apis-for-unreliable-networks)                                                                |

## Question 1. How would you implement zero-downtime deployments for Express.js services?

## Direct Answer

Zero-downtime deployments in Express.js are achieved by **starting a new version of the application before stopping the old one**, ensuring that no incoming requests are dropped. This is typically implemented using **graceful shutdown**, **load balancers or reverse proxies**, **process managers (PM2)**, **container orchestration (Kubernetes)**, or **blue-green/canary deployment strategies**.

The key is to:

1. Stop accepting new requests.
2. Allow existing requests to complete.
3. Close database and external connections.
4. Terminate the old process only after cleanup is complete.

---

# Detailed Explanation

A normal deployment often looks like this:

```
Stop old server
↓
Deploy new code
↓
Start new server
```

During the gap between stopping and starting, users receive errors (502, 503, connection refused).

A zero-downtime deployment instead works like:

```
Start new server
↓
Health check passes
↓
Route traffic to new server
↓
Gracefully stop old server
```

Users continue to access the application without interruption.

---

# 1. Graceful Shutdown (Most Important)

An Express application should never terminate immediately when receiving `SIGTERM` or `SIGINT`.

Instead:

- stop accepting new requests
- finish current requests
- close database connections
- close Redis/message queues
- exit cleanly

Example:

```javascript
const express = require("express");

const app = express();

const server = app.listen(3000, () => {
  console.log("Server started");
});

process.on("SIGTERM", () => {
  console.log("SIGTERM received");

  server.close(() => {
    console.log("HTTP server closed");

    // Close DB connections here

    process.exit(0);
  });

  // Force shutdown if hanging
  setTimeout(() => {
    process.exit(1);
  }, 30000);
});
```

`server.close()`:

- stops accepting new connections
- allows active requests to finish

This is the foundation of graceful deployment.

---

# 2. Track Active Connections

Sometimes clients keep connections open for a long time.

Track sockets:

```javascript
const sockets = new Set();

server.on("connection", (socket) => {
  sockets.add(socket);

  socket.on("close", () => {
    sockets.delete(socket);
  });
});

process.on("SIGTERM", () => {
  server.close(() => {
    process.exit(0);
  });

  sockets.forEach((socket) => socket.end());

  setTimeout(() => {
    sockets.forEach((socket) => socket.destroy());
  }, 5000);
});
```

This prevents deployments from hanging indefinitely.

---

# 3. Use Health Checks

Before routing traffic to a new instance, ensure it is ready.

Example:

```javascript
app.get("/health", (req, res) => {
  res.json({
    status: "ok",
  });
});
```

A deployment platform or load balancer should only send traffic after this endpoint reports healthy.

Readiness checks should verify dependencies like the database or cache if your application cannot serve requests without them.

---

# 4. Load Balancer Strategy

A load balancer sits in front of multiple Express instances.

```
                Users
                   |
             Load Balancer
            /      |      \
         App V1  App V1  App V2
```

Deployment steps:

1. Start App V2.
2. Wait until healthy.
3. Begin routing traffic to V2.
4. Remove V1 from rotation.
5. Gracefully shut down V1.

No requests are interrupted.

Common load balancers include:

- NGINX
- HAProxy
- Cloud load balancers provided by major cloud platforms

---

# 5. PM2 Zero-Downtime Reload

If using PM2 in cluster mode:

```bash
pm2 start app.js -i max
```

Reload without downtime:

```bash
pm2 reload all
```

PM2:

- starts new workers
- waits until they're ready
- shuts down old workers

Traffic continues uninterrupted.

---

# 6. Node.js Cluster Mode

Node's built-in clustering allows multiple worker processes.

```
          Master
        /    |    \
     Worker Worker Worker
```

Rolling restart:

- start a replacement worker
- wait until ready
- stop one old worker
- repeat

Users never notice the deployment.

---

# 7. Kubernetes Rolling Updates

In containerized environments, rolling updates are the standard approach.

Typical process:

```
Old Pods: 4

↓

Old Pods: 3
New Pods: 1

↓

Old Pods: 2
New Pods: 2

↓

Old Pods: 1
New Pods: 3

↓

Old Pods: 0
New Pods: 4
```

Important Kubernetes features include:

- readiness probes
- liveness probes
- rolling updates
- configurable surge and unavailable limits
- graceful termination periods

Kubernetes sends `SIGTERM`, giving Express time to shut down gracefully.

---

# 8. Blue-Green Deployment

Maintain two identical environments.

```
Blue  → Current Production

Green → New Version
```

Deploy to Green:

```
Blue (Live)

↓

Green deployed

↓

Switch Load Balancer

↓

Blue removed
```

Advantages:

- instant rollback
- nearly zero downtime
- safer deployments

Disadvantages:

- double infrastructure cost during deployment.

---

# 9. Canary Deployment

Instead of switching all users at once:

```
Version 1 → 95%

Version 2 → 5%
```

Then:

```
90 / 10

↓

50 / 50

↓

100% New Version
```

Benefits:

- detect bugs early
- reduce deployment risk
- easy rollback if issues arise

---

# 10. Database Migration Strategy

Database changes often cause downtime.

Prefer backward-compatible migrations:

Instead of:

```
Deploy code
Drop column
```

Use:

```
Add new column
↓

Deploy new app

↓

Migrate data

↓

Remove old column later
```

This allows old and new application versions to run simultaneously during the rollout.

---

# 11. Session Handling

If sessions are stored in memory:

```
Old Server
↓

Deployment

↓

Sessions lost
```

Instead, use a shared session store such as:

- Redis
- a database-backed session store

This ensures sessions survive rolling deployments and work across multiple instances.

---

# 12. Handle Long-Lived Connections

Applications using:

- WebSockets
- Server-Sent Events (SSE)
- long polling

need special handling.

Options include:

- stop accepting new connections
- allow existing connections to drain
- notify clients to reconnect
- use sticky sessions if required

Abruptly terminating these connections can disrupt users.

---

# Best Practices

- Implement graceful shutdown with `SIGTERM`.
- Use `server.close()` to stop accepting new requests while allowing in-flight requests to finish.
- Configure health/readiness endpoints so new instances receive traffic only when fully initialized.
- Run multiple application instances behind a load balancer.
- Use rolling, blue-green, or canary deployment strategies depending on your infrastructure and risk tolerance.
- Externalize state (sessions, caches) so instances remain stateless.
- Design database migrations to be backward compatible.
- Set reasonable shutdown timeouts and log deployment lifecycle events for observability.

---

# Common Pitfalls

- Calling `process.exit()` immediately, which terminates active requests.
- Forgetting to close database, cache, or message queue connections.
- Using in-memory sessions in a multi-instance deployment.
- Ignoring long-lived WebSocket or SSE connections during shutdown.
- Routing traffic to new instances before they pass readiness checks.
- Performing breaking database schema changes before all application instances have been updated.

---

# Interview Summary

A senior Express.js application achieves zero-downtime deployments by combining **graceful shutdown**, **health/readiness checks**, **multiple application instances behind a load balancer**, and **rolling, blue-green, or canary deployment strategies**. The application should stop accepting new requests on `SIGTERM`, allow existing requests to complete, clean up resources, and only then exit. Stateless application design, external session storage, and backward-compatible database migrations are essential to ensure deployments remain seamless and users experience no interruption.

## Question 2. What deployment challenges arise with stateful Express.js applications?

## Question 3. How do sticky sessions work with load balancers?

## Question 4. How do you horizontally scale WebSocket connections in Express.js systems?

## Question 5. What is the impact of blocking I/O on Node.js application performance?

## Question 6. How do you profile CPU bottlenecks in Express.js applications?

## Question 7. How do you identify and debug event emitter memory leaks?

## Question 8. What tools would you use to monitor memory usage in production Node.js servers?

## Question 9. How does garbage collection affect high-performance Express.js APIs?

## Question 10. How do you tune Node.js memory limits for large-scale applications?

## Question 11. What are the advantages and disadvantages of monorepo architecture for Express.js services?

## Question 12. How do you implement blue-green deployments for Express.js applications?

## Question 13. How would you design a centralized authentication service shared by multiple Express.js APIs?

## Question 14. How do API contracts help prevent breaking changes between services?

## Question 15. What strategies can reduce cold start times in serverless Express.js deployments?

## Question 16. How do you secure internal service-to-service communication?

## Question 17. How would you implement mutual TLS authentication between microservices?

## Question 18. What are the challenges of eventual consistency in distributed Express.js systems?

## Question 19. How do you implement saga patterns for distributed transactions?

## Question 20. How would you design retry-safe APIs for unreliable networks?
