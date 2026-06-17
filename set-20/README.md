# Set 20

| S.No. | Question                                                                                                                                                                                                   |
| ----- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 1.    | [What are vector clocks and how do they help resolve distributed conflicts?](#question-1-what-are-vector-clocks-and-how-do-they-help-resolve-distributed-conflicts)                                        |
| 2.    | [How do you design APIs resilient to network partitions?](#question-2-how-do-you-design-apis-resilient-to-network-partitions)                                                                              |
| 3.    | [What is split-brain failure in distributed systems?](#question-3-what-is-split-brain-failure-in-distributed-systems)                                                                                      |
| 4.    | [How do you ensure time synchronization consistency across services?](#question-4-how-do-you-ensure-time-synchronization-consistency-across-services)                                                      |
| 5.    | [What observability data should every Express.js request capture?](#question-5-what-observability-data-should-every-expressjs-request-capture)                                                             |
| 6.    | [How do sampling strategies work in distributed tracing systems?](#question-6-how-do-sampling-strategies-work-in-distributed-tracing-systems)                                                              |
| 7.    | [What is the difference between metrics, logs, and traces?](#question-7-what-is-the-difference-between-metrics-logs-and-traces)                                                                            |
| 8.    | [How would you implement anomaly detection for backend APIs?](#question-8-how-would-you-implement-anomaly-detection-for-backend-apis)                                                                      |
| 9.    | [What are chaos engineering principles for backend services?](#question-9-what-are-chaos-engineering-principles-for-backend-services)                                                                      |
| 10.   | [How do you safely run chaos experiments in production systems?](#question-10-how-do-you-safely-run-chaos-experiments-in-production-systems)                                                               |
| 11.   | [Design a scalable backend for processing real-time stock market updates](#question-11-design-a-scalable-backend-for-processing-real-time-stock-market-updates)                                            |
| 12.   | [How would you architect a globally distributed API authentication platform?](#question-12-how-would-you-architect-a-globally-distributed-api-authentication-platform)                                     |
| 13.   | [Design a backend system for reliable push notification delivery at internet scale](#question-13-design-a-backend-system-for-reliable-push-notification-delivery-at-internet-scale)                        |
| 14.   | [How would you build a low-latency recommendation API using Express.js?](#question-14-how-would-you-build-a-low-latency-recommendation-api-using-expressjs)                                                |
| 15.   | [Design an API traffic shaping system capable of handling unpredictable spikes](#question-15-design-an-api-traffic-shaping-system-capable-of-handling-unpredictable-spikes)                                |
| 16.   | [How would you architect a backend capable of serving billions of image requests daily?](#question-16-how-would-you-architect-a-backend-capable-of-serving-billions-of-image-requests-daily)               |
| 17.   | [Design a globally distributed audit logging platform using Express.js microservices](#question-17-design-a-globally-distributed-audit-logging-platform-using-expressjs-microservices)                     |
| 18.   | [How would you build a resilient backend for a large-scale ride-sharing platform?](#question-18-how-would-you-build-a-resilient-backend-for-a-large-scale-ride-sharing-platform)                           |
| 19.   | [Design a distributed backend for processing IoT device telemetry in real time](#question-19-design-a-distributed-backend-for-processing-iot-device-telemetry-in-real-time)                                |
| 20.   | [How would you redesign a backend suffering from cascading database failures under heavy load?](#question-20-how-would-you-redesign-a-backend-suffering-from-cascading-database-failures-under-heavy-load) |

## Question 1. What are vector clocks and how do they help resolve distributed conflicts?

## Direct Answer

**Vector clocks** are a logical clock mechanism used in distributed systems to **track causal relationships between events**. They help determine whether two updates are causally related or occurred concurrently, making them useful for **detecting and resolving distributed conflicts** in replicated databases, distributed storage systems, and eventually consistent systems.

Unlike a single timestamp or Lamport clock, vector clocks can identify **concurrent updates**, which is essential when multiple nodes modify the same data independently.

---

# Why Are Vector Clocks Needed?

In distributed systems:

- Multiple nodes process requests independently.
- There is no perfectly synchronized global clock.
- Network delays and partitions may cause replicas to receive updates in different orders.

Consider two users editing the same document while disconnected.

```
Replica A: Update X
Replica B: Update Y
```

When synchronization occurs later:

- Which update is newer?
- Should one overwrite the other?
- Did one depend on the other?

Simple timestamps often cannot answer these questions accurately.

---

# What Is a Vector Clock?

A vector clock is a **vector (list) of counters**, with one counter for each participating node.

Example:

```
Node A
Node B
Node C
```

Vector clock:

```
[A:2, B:5, C:1]
```

Each counter records how many events from that node have been observed.

---

# How Vector Clocks Work

Suppose we have three nodes.

Initial state:

```
A = [0,0,0]
B = [0,0,0]
C = [0,0,0]
```

---

## Step 1: Node A updates data

Node A increments its own counter.

```
A

[1,0,0]
```

---

## Step 2: A sends update to B

B receives:

```
Incoming:

[1,0,0]
```

B merges clocks:

Take maximum of every component.

```
max(
[0,0,0],
[1,0,0]
)

=

[1,0,0]
```

Then B performs another update.

```
[1,1,0]
```

---

## Step 3: Independent update on C

C modifies data separately.

```
[0,0,1]
```

Now we have:

```
Version 1

[1,1,0]

Version 2

[0,0,1]
```

These two versions are **concurrent** because neither vector dominates the other.

---

# Comparing Vector Clocks

Suppose:

```
V1 = [2,1,0]

V2 = [3,2,0]
```

Compare every component.

```
2 ≤ 3

1 ≤ 2

0 ≤ 0
```

Since every component is less than or equal, and at least one is smaller:

```
V1 happened before V2
```

Meaning:

```
V1 → V2
```

No conflict exists.

---

Now consider:

```
V1 = [2,1,4]

V2 = [3,2,2]
```

Comparison:

```
2 < 3

1 < 2

4 > 2
```

Neither vector is entirely less than or greater than the other.

Therefore:

```
Concurrent updates
```

This indicates a conflict.

---

# Conflict Detection

Imagine two replicas:

Replica A:

```
User changes email

Clock

[3,1]
```

Replica B:

```
User changes phone

Clock

[2,4]
```

Comparison:

```
3 > 2

1 < 4
```

Neither dominates.

Result:

```
Conflict detected
```

The system now knows both updates occurred independently.

---

# Conflict Resolution Strategies

Once concurrency is detected, the application chooses how to resolve it.

### 1. Last Write Wins (LWW)

Choose the newest timestamp.

Simple but may lose data.

---

### 2. Merge Both Changes

Example:

```
Email updated

Phone updated
```

Merged result:

```
Updated email

Updated phone
```

No information is lost.

---

### 3. Application-Level Merge

For example:

```
Shopping cart

Replica A:
Apple

Replica B:
Orange
```

Merge:

```
Apple
Orange
```

---

### 4. User Resolution

Collaborative editing tools may ask users to choose between conflicting changes.

---

# Example in Distributed Databases

Suppose two replicas store:

```
Profile

{
  name: "Alice"
}
```

Replica A updates:

```
name = "Alice Smith"

Clock

[2,0]
```

Replica B updates:

```
name = "Alice Johnson"

Clock

[0,3]
```

During synchronization:

```
[2,0]

vs

[0,3]
```

Neither dominates.

Conflict is detected.

Possible actions:

- Keep both versions
- Merge manually
- Ask the user
- Apply custom business logic

---

# Vector Clocks vs Lamport Clocks

| Feature                 | Lamport Clock | Vector Clock |
| ----------------------- | ------------- | ------------ |
| Captures event ordering | Yes           | Yes          |
| Detects concurrency     | No            | Yes          |
| Tracks causality        | Partial       | Complete     |
| Storage overhead        | Low           | Higher       |
| Conflict detection      | Limited       | Excellent    |

Lamport clocks only provide a partial ordering. If two events have timestamps 5 and 6, they cannot reliably indicate whether one caused the other. Vector clocks explicitly encode causal history.

---

# Advantages

- Detects concurrent updates accurately.
- Preserves causal relationships.
- Enables safe conflict detection.
- Well suited for eventually consistent systems.
- Prevents accidental overwriting of independent changes.

---

# Limitations

- Vector size grows with the number of participating nodes.
- Metadata overhead increases in large clusters.
- Dynamic membership (nodes joining or leaving) complicates management.
- Comparing vectors is more expensive than comparing scalar timestamps.

Because of these scalability concerns, some modern systems use alternatives such as **version vectors**, **dotted version vectors**, or **Hybrid Logical Clocks (HLCs)**.

---

# Real-World Uses

Vector clocks (or closely related version-vector techniques) have been used in systems such as:

- Distributed key-value stores for replica synchronization.
- Eventually consistent databases to detect conflicting writes.
- Distributed file synchronization systems.
- Collaborative editing and synchronization platforms.
- Conflict-aware replication protocols.

---

# Interview Summary

> **Vector clocks are logical clocks that maintain a counter for each node in a distributed system, allowing the system to determine causal relationships between events. By comparing vector clocks, a system can distinguish whether one update happened before another or whether they occurred concurrently. This capability is crucial for detecting distributed conflicts in eventually consistent systems, after which conflicts can be resolved using strategies such as merges, application-specific logic, last-write-wins, or user intervention. Compared to Lamport clocks, vector clocks provide more complete causality information but at the cost of additional metadata and scalability challenges.**

## Question 2. How do you design APIs resilient to network partitions?

## Question 3. What is split-brain failure in distributed systems?

## Question 4. How do you ensure time synchronization consistency across services?

## Question 5. What observability data should every Express.js request capture?

## Question 6. How do sampling strategies work in distributed tracing systems?

## Question 7. What is the difference between metrics, logs, and traces?

## Question 8. How would you implement anomaly detection for backend APIs?

## Question 9. What are chaos engineering principles for backend services?

## Question 10. How do you safely run chaos experiments in production systems?

## Question 11. Design a scalable backend for processing real-time stock market updates

## Question 12. How would you architect a globally distributed API authentication platform?

## Question 13. Design a backend system for reliable push notification delivery at internet scale

## Question 14. How would you build a low-latency recommendation API using Express.js?

## Question 15. Design an API traffic shaping system capable of handling unpredictable spikes

## Question 16. How would you architect a backend capable of serving billions of image requests daily?

## Question 17. Design a globally distributed audit logging platform using Express.js microservices

## Question 18. How would you build a resilient backend for a large-scale ride-sharing platform?

## Question 19. Design a distributed backend for processing IoT device telemetry in real time

## Question 20. How would you redesign a backend suffering from cascading database failures under heavy load?
