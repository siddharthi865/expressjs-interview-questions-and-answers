# Set 19

| S.No. | Question                                                                                                                                                                           |
| ----- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 1.    | [How does Express.js interact with the Node.js event loop internally?](#question-1-how-does-expressjs-interact-with-the-nodejs-event-loop-internally)                              |
| 2.    | [What are microtasks and macrotasks, and how can they affect Express.js APIs?](#question-2-what-are-microtasks-and-macrotasks-and-how-can-they-affect-expressjs-apis)              |
| 3.    | [How can blocking synchronous code freeze an Express.js server?](#question-3-how-can-blocking-synchronous-code-freeze-an-expressjs-server)                                         |
| 4.    | [What are the trade-offs between Express.js and Fastify in high-performance APIs?](#question-4-what-are-the-trade-offs-between-expressjs-and-fastify-in-high-performance-apis)     |
| 5.    | [How would you benchmark different Node.js frameworks objectively?](#question-5-how-would-you-benchmark-different-nodejs-frameworks-objectively)                                   |
| 6.    | [How do you tune garbage collection for latency-sensitive applications?](#question-6-how-do-you-tune-garbage-collection-for-latency-sensitive-applications)                        |
| 7.    | [What is event loop starvation and how can it occur in Express.js?](#question-7-what-is-event-loop-starvation-and-how-can-it-occur-in-expressjs)                                   |
| 8.    | [How do long-running synchronous loops impact concurrent requests?](#question-8-how-do-long-running-synchronous-loops-impact-concurrent-requests)                                  |
| 9.    | [How do AsyncLocalStorage and request context propagation work?](#question-9-how-do-asynclocalstorage-and-request-context-propagation-work)                                        |
| 10.   | [What are the challenges of maintaining transaction boundaries across async calls?](#question-10-what-are-the-challenges-of-maintaining-transaction-boundaries-across-async-calls) |
| 11.   | [How do you implement streaming uploads directly to cloud storage?](#question-11-how-do-you-implement-streaming-uploads-directly-to-cloud-storage)                                 |
| 12.   | [What are signed POST policies for cloud uploads?](#question-12-what-are-signed-post-policies-for-cloud-uploads)                                                                   |
| 13.   | [How would you secure file processing pipelines against malicious uploads?](#question-13-how-would-you-secure-file-processing-pipelines-against-malicious-uploads)                 |
| 14.   | [How do antivirus scanning workflows integrate with Express.js upload systems?](#question-14-how-do-antivirus-scanning-workflows-integrate-with-expressjs-upload-systems)          |
| 15.   | [What are the trade-offs between synchronous APIs and event-driven architectures?](#question-15-what-are-the-trade-offs-between-synchronous-apis-and-event-driven-architectures)   |
| 16.   | [How do CQRS patterns apply to Express.js applications?](#question-16-how-do-cqrs-patterns-apply-to-expressjs-applications)                                                        |
| 17.   | [What are read replicas and how do they improve scalability?](#question-17-what-are-read-replicas-and-how-do-they-improve-scalability)                                             |
| 18.   | [How do you route read and write queries efficiently in APIs?](#question-18-how-do-you-route-read-and-write-queries-efficiently-in-apis)                                           |
| 19.   | [What are quorum-based systems and when are they useful?](#question-19-what-are-quorum-based-systems-and-when-are-they-useful)                                                     |
| 20.   | [How would you implement distributed consensus-related workflows?](#question-20-how-would-you-implement-distributed-consensus-related-workflows)                                   |

## Question 1. How does Express.js interact with the Node.js event loop internally?

## Direct Answer

Express.js does **not implement its own event loop**. Instead, it runs **on top of Node.js**, leveraging Node's **single-threaded, event-driven architecture**. When an HTTP request arrives, Node.js receives it through its HTTP server, invokes the Express application as a request handler, and Express processes the request by executing its middleware and routing stack synchronously until it encounters asynchronous operations. During asynchronous work (such as database queries or file I/O), control returns to the Node.js event loop, allowing it to process other requests. Once the async operation completes, the callback, Promise, or `async/await` continuation is queued and eventually resumes Express's request handling.

---

# Detailed Explanation

## 1. High-Level Architecture

The request flow looks like this:

```text
Client Request
      │
      ▼
Node.js HTTP Server
      │
      ▼
Event Loop receives request
      │
      ▼
Express Application (request handler)
      │
      ▼
Middleware Stack
      │
      ▼
Route Handler
      │
      ▼
Async Operations (DB, File, API)
      │
      ▼
Node.js Event Loop
      │
      ▼
Callback / Promise Resolution
      │
      ▼
Remaining Express Middleware
      │
      ▼
Response Sent
```

---

# 2. Express Is Just a Request Handler

When you create an Express application:

```javascript
const express = require("express");
const app = express();

app.listen(3000);
```

Internally, Express does something conceptually similar to:

```javascript
const http = require("http");

const server = http.createServer(app);

server.listen(3000);
```

Notice that:

```javascript
app;
```

is actually a function:

```javascript
(req, res) => {
  // Express handles request
};
```

Node's HTTP server invokes this function every time a request arrives.

---

# 3. Request Lifecycle

Suppose you have:

```javascript
app.use((req, res, next) => {
  console.log("Middleware 1");
  next();
});

app.get("/users", (req, res) => {
  res.send("Users");
});
```

When a request arrives:

1. Node accepts the TCP connection.
2. HTTP parser creates `req` and `res`.
3. Event loop dispatches the request.
4. Express receives `(req, res)`.
5. Middleware executes.
6. Route executes.
7. Response is written.
8. Connection returns to Node.

Everything above happens synchronously unless async work is introduced.

---

# 4. What Happens During Async Operations?

Consider:

```javascript
app.get("/users", async (req, res) => {
  const users = await db.getUsers();

  res.json(users);
});
```

Execution proceeds like this:

```text
Request arrives
      │
      ▼
Express starts handler
      │
      ▼
await db.getUsers()
      │
      ▼
Database query starts
      │
      ▼
Handler pauses
      │
      ▼
Control returns to Event Loop
      │
      ▼
Other requests continue executing
      │
      ▼
Database finishes
      │
      ▼
Promise resolves
      │
      ▼
Continuation scheduled
      │
      ▼
Handler resumes
      │
      ▼
res.json(users)
```

This non-blocking behavior allows a single Node.js process to efficiently serve many concurrent requests.

---

# 5. Middleware Execution and the Event Loop

Express maintains a middleware stack.

Example:

```javascript
app.use(auth);
app.use(logger);
app.use(validate);

app.get("/profile", handler);
```

Internally it behaves conceptually like:

```javascript
auth(req, res, () => {
  logger(req, res, () => {
    validate(req, res, () => {
      handler(req, res);
    });
  });
});
```

Each middleware calls:

```javascript
next();
```

to hand control to the next middleware.

If middleware performs asynchronous work:

```javascript
app.use(async (req, res, next) => {
  const user = await getUser();

  req.user = user;

  next();
});
```

then:

- middleware pauses
- event loop continues processing other events
- Promise resolves
- middleware resumes
- `next()` continues the chain

---

# 6. Event Loop Never Waits for I/O

Imagine three requests:

```text
Request A → DB query
Request B → Static file
Request C → API call
```

Timeline:

```text
Time

A starts
A waits for DB

B starts
B completes

C starts
C waits for API

DB completes
A resumes

API completes
C resumes
```

The JavaScript thread is not blocked while waiting for external resources.

---

# 7. What Actually Blocks the Event Loop?

Express itself is lightweight and generally doesn't block the event loop. Problems arise when your route handlers perform CPU-intensive synchronous work, such as:

```javascript
app.get("/report", (req, res) => {
  // Bad
  while (true) {}
});
```

or

```javascript
const crypto = require("crypto");

app.get("/hash", (req, res) => {
  for (let i = 0; i < 500000; i++) {
    crypto.createHash("sha512").update("data").digest("hex");
  }

  res.send("Done");
});
```

During such synchronous computation:

- no other requests are processed
- timers don't fire
- sockets aren't serviced
- the event loop is blocked

---

# 8. Express Benefits from Non-Blocking APIs

Good example:

```javascript
app.get("/file", (req, res) => {
  fs.readFile("large.txt", (err, data) => {
    res.send(data);
  });
});
```

While the file is being read:

- Node delegates the work (typically to the libuv thread pool for file system operations).
- The event loop continues handling other requests.
- Once the read completes, the callback is queued and executed.

---

# 9. Promise-Based Flow

Modern Express commonly uses `async/await`:

```javascript
app.get("/products", async (req, res) => {
  const products = await Product.find();

  res.json(products);
});
```

Internally:

```text
Handler starts
      │
      ▼
Promise created
      │
      ▼
Handler suspended
      │
      ▼
Event loop continues
      │
      ▼
Promise fulfilled
      │
      ▼
Microtask queue
      │
      ▼
Handler resumes
```

`await` doesn't block the event loop; it pauses only the current async function.

---

# 10. Role of libuv

Node.js relies on **libuv** to provide the event loop and asynchronous I/O capabilities.

When Express code performs operations like:

- File system access (`fs`)
- DNS lookups
- Some cryptographic operations
- Compression

libuv may use its internal thread pool or operating system facilities to perform the work asynchronously. Network sockets are generally handled through non-blocking OS mechanisms rather than the thread pool.

Express simply consumes these asynchronous APIs—it does not manage threads or the event loop itself.

---

# 11. Error Handling and the Event Loop

With Express 5, rejected Promises or exceptions thrown inside `async` route handlers are automatically forwarded to the error-handling middleware.

```javascript
app.get("/users", async (req, res) => {
  const users = await db.getUsers(); // If rejected
  res.json(users);
});

app.use((err, req, res, next) => {
  res.status(500).json({
    error: err.message,
  });
});
```

The Promise rejection is detected after the asynchronous operation completes, and Express routes the error through its middleware chain without blocking the event loop.

---

# Common Pitfalls

- **Blocking the event loop** with CPU-intensive synchronous code.
- Using synchronous APIs like `fs.readFileSync()` or `crypto.pbkdf2Sync()` inside request handlers.
- Forgetting to `await` asynchronous operations, which can lead to responses being sent before work completes.
- Omitting `next()` (or ending the response) in middleware, causing requests to hang.
- Performing long-running computations in route handlers instead of offloading them to worker threads, child processes, or external services.

---

# Best Practices

- Prefer asynchronous APIs (`fs.promises`, database drivers, HTTP clients).
- Keep middleware lightweight and avoid expensive synchronous operations.
- Use `async/await` for readable asynchronous code.
- Offload CPU-intensive work to **Worker Threads**, background jobs, or separate services.
- Stream large files instead of loading them entirely into memory.
- Monitor event loop lag in production to detect blocking operations.

---

# Interview Summary

> **Express.js doesn't have its own event loop—it relies entirely on Node.js's event-driven architecture. Node's HTTP server invokes the Express application for each incoming request, and Express processes it through its middleware and routing stack. When asynchronous operations such as database queries or file I/O occur, execution yields back to the Node.js event loop, allowing other requests to be handled concurrently. Once the asynchronous task completes, the associated callback or Promise continuation resumes the Express request flow. This non-blocking model enables Express applications to efficiently handle many concurrent I/O-bound requests, provided developers avoid blocking the event loop with CPU-intensive synchronous code.**

## Question 2. What are microtasks and macrotasks, and how can they affect Express.js APIs?

## Question 3. How can blocking synchronous code freeze an Express.js server?

## Question 4. What are the trade-offs between Express.js and Fastify in high-performance APIs?

## Question 5. How would you benchmark different Node.js frameworks objectively?

## Question 6. How do you tune garbage collection for latency-sensitive applications?

## Question 7. What is event loop starvation and how can it occur in Express.js?

## Question 8. How do long-running synchronous loops impact concurrent requests?

## Question 9. How do AsyncLocalStorage and request context propagation work?

## Question 10. What are the challenges of maintaining transaction boundaries across async calls?

## Question 11. How do you implement streaming uploads directly to cloud storage?

## Question 12. What are signed POST policies for cloud uploads?

## Question 13. How would you secure file processing pipelines against malicious uploads?

## Question 14. How do antivirus scanning workflows integrate with Express.js upload systems?

## Question 15. What are the trade-offs between synchronous APIs and event-driven architectures?

## Question 16. How do CQRS patterns apply to Express.js applications?

## Question 17. What are read replicas and how do they improve scalability?

## Question 18. How do you route read and write queries efficiently in APIs?

## Question 19. What are quorum-based systems and when are they useful?

## Question 20. How would you implement distributed consensus-related workflows?
