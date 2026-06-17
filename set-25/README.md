# Set 25

| S.No. | Question                                                                                                                                                                                                                                                                |
| ----- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 1.    | [What are service mesh architectures and why are they used?](#question-1-what-are-service-mesh-architectures-and-why-are-they-used)                                                                                                                                     |
| 2.    | [How does mutual TLS work inside a service mesh?](#question-2-how-does-mutual-tls-work-inside-a-service-mesh)                                                                                                                                                           |
| 3.    | [How do you design APIs resilient to cloud region outages?](#question-3-how-do-you-design-apis-resilient-to-cloud-region-outages)                                                                                                                                       |
| 4.    | [What are active-active and active-passive deployment strategies?](#question-4-what-are-active-active-and-active-passive-deployment-strategies)                                                                                                                         |
| 5.    | [How do CAP theorem trade-offs influence backend API design?](#question-5-how-do-cap-theorem-trade-offs-influence-backend-api-design)                                                                                                                                   |
| 6.    | [What are quorum reads and quorum writes?](#question-6-what-are-quorum-reads-and-quorum-writes)                                                                                                                                                                         |
| 7.    | [How do distributed tracing systems propagate context headers?](#question-7-how-do-distributed-tracing-systems-propagate-context-headers)                                                                                                                               |
| 8.    | [What are tail latencies and why are they critical in large-scale APIs?](#question-8-what-are-tail-latencies-and-why-are-they-critical-in-large-scale-apis)                                                                                                             |
| 9.    | [How would you debug intermittent high-latency spikes in Express.js services?](#question-9-how-would-you-debug-intermittent-high-latency-spikes-in-expressjs-services)                                                                                                  |
| 10.   | [What are common causes of cascading retries in distributed systems?](#question-10-what-are-common-causes-of-cascading-retries-in-distributed-systems)                                                                                                                  |
| 11.   | [Design a backend for globally distributed real-time document collaboration](#question-11-design-a-backend-for-globally-distributed-real-time-document-collaboration)                                                                                                   |
| 12.   | [How would you architect a high-throughput fraud detection API platform?](#question-12-how-would-you-architect-a-high-throughput-fraud-detection-api-platform)                                                                                                          |
| 13.   | [Design a resilient API orchestration layer for thousands of microservices](#question-13-design-a-resilient-api-orchestration-layer-for-thousands-of-microservices)                                                                                                     |
| 14.   | [How would you build a globally distributed API analytics platform with near real-time dashboards?](#question-14-how-would-you-build-a-globally-distributed-api-analytics-platform-with-near-real-time-dashboards)                                                      |
| 15.   | [Design a backend capable of processing millions of concurrent WebSocket connections](#question-15-design-a-backend-capable-of-processing-millions-of-concurrent-websocket-connections)                                                                                 |
| 16.   | [How would you architect a low-latency multiplayer gaming backend using Express.js services?](#question-16-how-would-you-architect-a-low-latency-multiplayer-gaming-backend-using-expressjs-services)                                                                   |
| 17.   | [Design a backend system for globally synchronized live sports score updates](#question-17-design-a-backend-system-for-globally-synchronized-live-sports-score-updates)                                                                                                 |
| 18.   | [How would you build a scalable API platform for third-party developer ecosystems?](#question-18-how-would-you-build-a-scalable-api-platform-for-third-party-developer-ecosystems)                                                                                      |
| 19.   | [Design a resilient distributed backend for a worldwide food delivery application](#question-19-design-a-resilient-distributed-backend-for-a-worldwide-food-delivery-application)                                                                                       |
| 20.   | [How would you architect an Express.js-based platform capable of surviving full regional cloud outages with minimal downtime?](#question-20-how-would-you-architect-an-expressjs-based-platform-capable-of-surviving-full-regional-cloud-outages-with-minimal-downtime) |

## Question 1. What are service mesh architectures and why are they used?

## Direct Answer

A **service mesh** is an infrastructure layer that manages **service-to-service communication** in a microservices architecture. Instead of implementing networking features like service discovery, load balancing, retries, authentication, encryption, and observability inside each service, a service mesh handles these concerns transparently through **sidecar proxies** or similar data-plane components.

Service meshes are used to **improve reliability, security, observability, and traffic management** while allowing developers to focus on business logic rather than networking code.

---

# Detailed Explanation

As applications grow from a few services to hundreds of microservices, communication between services becomes increasingly complex.

Without a service mesh, every service may need to implement:

- Authentication
- TLS encryption
- Retries
- Timeouts
- Circuit breakers
- Metrics collection
- Distributed tracing
- Load balancing

This results in duplicated code and inconsistent implementations.

A service mesh centralizes these networking responsibilities.

---

# High-Level Architecture

A service mesh typically consists of two major components:

```
                Control Plane
        +-------------------------+
        | Configuration           |
        | Policies                |
        | Certificates            |
        | Service Discovery       |
        +------------+------------+
                     |
        -------------------------------
                     |
             Data Plane (Sidecars)

 Client ---> Proxy ---> Service A
                  |
                  |
             Proxy ---> Service B
                  |
             Proxy ---> Service C
```

The application communicates through a local proxy rather than directly with other services.

---

# Core Components

## 1. Data Plane

The data plane consists of proxies running alongside every service.

Usually implemented as:

- Envoy Proxy
- Linkerd Proxy

Responsibilities:

- Forward requests
- Retry failed requests
- Load balancing
- TLS encryption
- Metrics
- Logging
- Traffic routing

Example:

```
User Request

↓

Express Service A

↓

Envoy Sidecar

↓

Envoy Sidecar

↓

Express Service B
```

The Express application doesn't need to know anything about networking policies.

---

## 2. Control Plane

The control plane configures all proxies.

Responsibilities include:

- Policy management
- Service discovery
- Certificate management
- Traffic rules
- Authorization
- Configuration updates

The control plane never handles application traffic directly.

---

# Why Service Meshes Are Used

## 1. Traffic Management

Instead of writing routing logic in code:

```
Service A

↓

70% → v1

30% → v2
```

You configure:

- Canary deployments
- Blue-green deployments
- A/B testing
- Shadow traffic

without changing application code.

---

## 2. Load Balancing

Instead of:

```javascript
const instances = ["server1", "server2", "server3"];
```

The proxy automatically performs:

- Round robin
- Least connections
- Weighted balancing
- Locality-aware routing

---

## 3. Automatic Retries

Without a service mesh:

```javascript
try {
  await axios.get(url);
} catch {
  retry();
}
```

With a service mesh:

```
Retry:
  attempts: 3
```

Configured once for every service.

---

## 4. Timeouts

Instead of every service implementing:

```javascript
axios.get(url, {
  timeout: 3000,
});
```

The mesh enforces:

```
Timeout: 3 seconds
```

centrally.

---

## 5. Circuit Breaking

If a downstream service fails repeatedly:

```
Service A

↓

Service B (down)

↓

Stop sending traffic temporarily
```

The proxy opens the circuit automatically.

Benefits:

- Prevents cascading failures
- Protects healthy services

---

## 6. Mutual TLS (mTLS)

One of the biggest advantages.

Every service communicates over encrypted channels.

```
Service A

⇄ TLS

Service B
```

The mesh automatically:

- Generates certificates
- Rotates certificates
- Verifies identities
- Encrypts traffic

Developers don't write TLS code.

---

## 7. Authentication and Authorization

Policies like:

```
Only payment-service
may call billing-service
```

can be enforced centrally.

---

## 8. Observability

Every request is automatically tracked.

Collected data includes:

- Latency
- Request rate
- Error rate
- Traces
- Logs

No instrumentation is required for basic network metrics.

---

# Example Request Flow

```
User

↓

API Gateway

↓

Proxy A

↓

Express Service A

↓

Proxy A

↓

Proxy B

↓

Express Service B

↓

Database
```

Every request passes through proxies.

---

# Common Features

| Feature             | Purpose                         |
| ------------------- | ------------------------------- |
| Service Discovery   | Locate services dynamically     |
| Load Balancing      | Distribute traffic              |
| Retries             | Recover from transient failures |
| Timeouts            | Prevent hanging requests        |
| Circuit Breaker     | Prevent cascading failures      |
| mTLS                | Encrypt service communication   |
| Metrics             | Monitor service health          |
| Distributed Tracing | Track request flow              |
| Rate Limiting       | Protect services                |
| Traffic Splitting   | Canary deployments              |

---

# Popular Service Mesh Implementations

Some widely used service meshes include:

- Istio
- Linkerd
- Consul Connect
- Kuma
- Open Service Mesh

Most modern service meshes use **Envoy** as the data-plane proxy.

---

# Service Mesh vs API Gateway

| API Gateway                | Service Mesh                    |
| -------------------------- | ------------------------------- |
| North-South traffic        | East-West traffic               |
| Client → Service           | Service → Service               |
| Authentication for clients | Authentication between services |
| Public entry point         | Internal communication          |
| Rate limiting              | Retries, mTLS, routing          |
| Usually one gateway        | Proxy beside every service      |

In many architectures, both are used together:

- An API gateway handles external client requests.
- A service mesh manages internal service-to-service communication.

---

# Advantages

- Centralized traffic management
- Automatic mTLS encryption
- Consistent retries and timeouts
- Better observability
- Simplified application code
- Easier canary and blue-green deployments
- Improved reliability
- Uniform security policies

---

# Challenges

- Additional infrastructure complexity
- Increased CPU and memory usage due to sidecar proxies
- Slight increase in network latency because traffic passes through proxies
- More operational overhead for configuration and maintenance
- Can be excessive for small applications with only a few services

---

# Best Practices

- Adopt a service mesh when managing many microservices with complex communication patterns.
- Keep business logic inside services; avoid embedding networking concerns in application code.
- Enable mTLS by default for secure service-to-service communication.
- Define retries, timeouts, and circuit breakers carefully to avoid retry storms.
- Monitor proxy metrics and distributed traces to detect latency and failures.
- Start with a minimal set of mesh features and introduce advanced traffic management as operational needs grow.

---

# Interview Summary

> **A service mesh is an infrastructure layer that manages service-to-service communication in microservices. It uses sidecar proxies controlled by a centralized control plane to provide features such as load balancing, service discovery, retries, circuit breaking, mTLS, traffic routing, and observability without requiring these capabilities to be implemented in application code. This improves security, reliability, and operational consistency while allowing developers to focus on business logic.**

## Question 2. How does mutual TLS work inside a service mesh?

## Question 3. How do you design APIs resilient to cloud region outages?

## Question 4. What are active-active and active-passive deployment strategies?

## Question 5. How do CAP theorem trade-offs influence backend API design?

## Question 6. What are quorum reads and quorum writes?

## Question 7. How do distributed tracing systems propagate context headers?

## Question 8. What are tail latencies and why are they critical in large-scale APIs?

## Question 9. How would you debug intermittent high-latency spikes in Express.js services?

## Question 10. What are common causes of cascading retries in distributed systems?

## Question 11. Design a backend for globally distributed real-time document collaboration

## Question 12. How would you architect a high-throughput fraud detection API platform?

## Question 13. Design a resilient API orchestration layer for thousands of microservices

## Question 14. How would you build a globally distributed API analytics platform with near real-time dashboards?

## Question 15. Design a backend capable of processing millions of concurrent WebSocket connections

## Question 16. How would you architect a low-latency multiplayer gaming backend using Express.js services?

## Question 17. Design a backend system for globally synchronized live sports score updates

## Question 18. How would you build a scalable API platform for third-party developer ecosystems?

## Question 19. Design a resilient distributed backend for a worldwide food delivery application

## Question 20. How would you architect an Express.js-based platform capable of surviving full regional cloud outages with minimal downtime?
