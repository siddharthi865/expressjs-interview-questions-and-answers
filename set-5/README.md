# Set 5

| S.No. | Question                                                                                                                                                                                                                      |
| ----- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 1.    | [How do you monitor event loop lag in Node.js applications?](#question-1-how-do-you-monitor-event-loop-lag-in-nodejs-applications)                                                                                            |
| 2.    | [How do you optimize Express.js applications for low latency under heavy traffic?](#question-2-how-do-you-optimize-expressjs-applications-for-low-latency-under-heavy-traffic)                                                |
| 3.    | [How would you implement circuit breakers in Express.js microservices?](#question-3-how-would-you-implement-circuit-breakers-in-expressjs-microservices)                                                                      |
| 4.    | [How do retries and exponential backoff improve system resilience?](#question-4-how-do-retries-and-exponential-backoff-improve-system-resilience)                                                                             |
| 5.    | [How do you test middleware in Express.js applications?](#question-5-how-do-you-test-middleware-in-expressjs-applications)                                                                                                    |
| 6.    | [How would you implement integration testing for Express.js APIs?](#question-6-how-would-you-implement-integration-testing-for-expressjs-apis)                                                                                |
| 7.    | [What are the differences between unit tests, integration tests, and end-to-end tests in backend systems?](#question-7-what-are-the-differences-between-unit-tests-integration-tests-and-end-to-end-tests-in-backend-systems) |
| 8.    | [How do you mock external services while testing Express.js APIs?](#question-8-how-do-you-mock-external-services-while-testing-expressjs-apis)                                                                                |
| 9.    | [How do you implement structured error responses across all APIs?](#question-9-how-do-you-implement-structured-error-responses-across-all-apis)                                                                               |
| 10.   | [How would you handle multi-tenant architecture in Express.js applications?](#question-10-how-would-you-handle-multi-tenant-architecture-in-expressjs-applications)                                                           |
| 11.   | [Design a globally scalable URL shortener backend using Express.js](#question-11-design-a-globally-scalable-url-shortener-backend-using-expressjs)                                                                            |
| 12.   | [Design a highly available authentication service using Express.js and JWTs](#question-12-design-a-highly-available-authentication-service-using-expressjs-and-jwts)                                                          |
| 13.   | [How would you scale an Express.js API to handle millions of requests per minute?](#question-13-how-would-you-scale-an-expressjs-api-to-handle-millions-of-requests-per-minute)                                               |
| 14.   | [Design a rate-limiting system for a multi-region Express.js deployment](#question-14-design-a-rate-limiting-system-for-a-multi-region-expressjs-deployment)                                                                  |
| 15.   | [How would you reduce p99 latency in a heavily loaded Express.js service?](#question-15-how-would-you-reduce-p99-latency-in-a-heavily-loaded-expressjs-service)                                                               |
| 16.   | [How would you debug intermittent memory spikes in a production Express.js application?](#question-16-how-would-you-debug-intermittent-memory-spikes-in-a-production-expressjs-application)                                   |
| 17.   | [Design a resilient Express.js backend for real-time chat applications](#question-17-design-a-resilient-expressjs-backend-for-real-time-chat-applications)                                                                    |
| 18.   | [How would you implement distributed tracing across multiple Express.js microservices?](#question-18-how-would-you-implement-distributed-tracing-across-multiple-expressjs-microservices)                                     |
| 19.   | [How would you migrate a monolithic Express.js application into microservices incrementally?](#question-19-how-would-you-migrate-a-monolithic-expressjs-application-into-microservices-incrementally)                         |
| 20.   | [Explain how you would architect a fault-tolerant Express.js backend for a global e-commerce platform](#question-20-explain-how-you-would-architect-a-fault-tolerant-expressjs-backend-for-a-global-e-commerce-platform)      |

## Question 1. How do you monitor event loop lag in Node.js applications?

**Short Answer:**
You can monitor Node.js event loop lag using built-in `perf_hooks`, external monitoring tools (like Prometheus, Datadog, New Relic), or custom interval-based timers that measure delay in scheduled execution. The most accurate native approach is using `perf_hooks.monitorEventLoopDelay()`.

---

# 📌 Detailed Interview Explanation

## 🧠 What is Event Loop Lag?

Event loop lag (or latency) is the delay between when the event loop _should_ execute a callback and when it actually executes it. High lag indicates:

- CPU saturation
- Blocking synchronous code
- Heavy GC pressure
- Poorly optimized I/O handling

In production, it’s a key metric for Node.js performance health.

---

# 🚀 1. Using `perf_hooks.monitorEventLoopDelay()` (Recommended)

Node.js provides a built-in, high-resolution API for measuring event loop delay.

### Example:

```js
const { monitorEventLoopDelay } = require("perf_hooks");

const h = monitorEventLoopDelay({ resolution: 20 });
h.enable();

// Log every 5 seconds
setInterval(() => {
  console.log({
    min: h.min / 1e6, // ms
    max: h.max / 1e6,
    mean: h.mean / 1e6,
    p99: h.percentile(99) / 1e6,
  });

  h.reset();
}, 5000);
```

---

## 📊 Why this is best:

- Measures actual scheduling delay inside event loop
- High-resolution histogram-based metrics
- No external dependencies
- Suitable for production monitoring

---

## ⚠️ Common Pitfall:

- Forgetting to call `h.enable()` → returns zeros
- Not resetting histogram → misleading cumulative stats

---

# 🧪 2. Simple Custom Lag Detection (SetTimeout Drift)

This is a classic interview approach.

### Example:

```js
const interval = 100;

setInterval(() => {
  const start = Date.now();

  setTimeout(() => {
    const lag = Date.now() - start - interval;
    console.log(`Event loop lag: ${lag}ms`);
  }, interval);
}, interval);
```

---

## ⚠️ Limitations:

- Less accurate than `perf_hooks`
- Sensitive to system scheduling noise
- Not ideal for production metrics

---

# 📡 3. Production Monitoring Tools

In real-world systems, event loop lag is often exported as a metric:

## Popular tools:

- **Prometheus + prom-client**
- **Datadog APM**
- **New Relic**
- **Elastic APM**

### Example with Prometheus:

```js
const client = require("prom-client");
const { monitorEventLoopDelay } = require("perf_hooks");

const histogram = new client.Histogram({
  name: "event_loop_lag_ms",
  help: "Event loop lag in ms",
});

const h = monitorEventLoopDelay();
h.enable();

setInterval(() => {
  histogram.observe(h.mean / 1e6);
  h.reset();
}, 5000);
```

---

# ⚙️ 4. Node.js Internal Signals (Indirect Indicators)

While not direct lag measurement, these help detect symptoms:

- CPU usage (`process.cpuUsage()`)
- Garbage collection logs (`--trace-gc`)
- libuv threadpool saturation
- Response latency (HTTP middleware timing)

---

# 🚨 Common Causes of Event Loop Lag

Interviewers often expect this follow-up:

### 1. Blocking synchronous code

```js
while (true) {} // 💥 blocks event loop
```

### 2. Heavy JSON parsing or computation

### 3. Large synchronous loops

### 4. CPU-bound crypto operations

### 5. GC pressure from memory leaks

---

# 🧠 Best Practices (Senior-Level Answer)

- Always prefer `perf_hooks.monitorEventLoopDelay()` in production
- Export metrics to a monitoring system (Prometheus/APM)
- Set alert thresholds (e.g., p99 > 50ms)
- Correlate lag with CPU/memory metrics
- Break heavy CPU work into worker threads (`worker_threads` module)

---

# 🆚 Comparison Summary

| Method                             | Accuracy   | Production Ready | Notes              |
| ---------------------------------- | ---------- | ---------------- | ------------------ |
| `perf_hooks.monitorEventLoopDelay` | ⭐⭐⭐⭐⭐ | ✅ Yes           | Best option        |
| setTimeout drift                   | ⭐⭐       | ⚠️ Limited       | Educational/demo   |
| APM tools                          | ⭐⭐⭐⭐⭐ | ✅ Yes           | Best observability |
| CPU/memory heuristics              | ⭐⭐       | ⚠️ Indirect      | Supportive only    |

## Question 2. How do you optimize Express.js applications for low latency under heavy traffic?

## Question 3. How would you implement circuit breakers in Express.js microservices?

## Question 4. How do retries and exponential backoff improve system resilience?

## Question 5. How do you test middleware in Express.js applications?

## Question 6. How would you implement integration testing for Express.js APIs?

## Question 7. What are the differences between unit tests, integration tests, and end-to-end tests in backend systems?

## Question 8. How do you mock external services while testing Express.js APIs?

## Question 9. How do you implement structured error responses across all APIs?

## Question 10. How would you handle multi-tenant architecture in Express.js applications?

## Question 11. Design a globally scalable URL shortener backend using Express.js

## Question 12. Design a highly available authentication service using Express.js and JWTs

## Question 13. How would you scale an Express.js API to handle millions of requests per minute?

## Question 14. Design a rate-limiting system for a multi-region Express.js deployment

## Question 15. How would you reduce p99 latency in a heavily loaded Express.js service?

## Question 16. How would you debug intermittent memory spikes in a production Express.js application?

## Question 17. Design a resilient Express.js backend for real-time chat applications

## Question 18. How would you implement distributed tracing across multiple Express.js microservices?

## Question 19. How would you migrate a monolithic Express.js application into microservices incrementally?

## Question 20. Explain how you would architect a fault-tolerant Express.js backend for a global e-commerce platform
