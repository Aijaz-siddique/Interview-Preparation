# System Design Interview Preparation — HLD + LLD

Senior/lead interview revision covering requirements, scalability, distributed systems, data, messaging, resilience, APIs, security and classic designs.

## Interview Approach & Estimation

### Q1. How do you start an HLD interview?
<details><summary>Answer</summary>

Clarify functional requirements, non-functional requirements, users, scale, latency, availability, consistency and constraints before drawing the architecture.

</details>

### Q2. How do you estimate RPS?
<details><summary>Answer</summary>

Convert requests/day to average requests/sec and apply a peak multiplier. State assumptions explicitly and use them to justify capacity.

</details>

### Q3. How do you estimate storage?
<details><summary>Answer</summary>

Estimate events/records per day × average size × retention, then add indexes/replication overhead where relevant.

</details>

### Q4. What is an SLO?
<details><summary>Answer</summary>

A target for service behavior such as availability or latency. SLOs make reliability measurable and guide engineering trade-offs.

</details>

### Q5. RTO vs RPO?
<details><summary>Answer</summary>

RTO is how quickly service/data must be restored. RPO is the maximum acceptable data loss measured in time.

</details>

### Q6. What questions should you ask about consistency?
<details><summary>Answer</summary>

Which operations require read-your-writes/strong consistency? Which data can be eventually consistent? What stale-data window is acceptable?

</details>

## Distributed Systems

### Q7. CAP theorem?
<details><summary>Answer</summary>

When a network partition occurs, a distributed system cannot simultaneously guarantee both strong consistency and availability for every operation. Real designs choose behavior appropriate to the business.

</details>

### Q8. Strong vs eventual consistency?
<details><summary>Answer</summary>

Strong consistency gives stronger read guarantees; eventual consistency allows temporary divergence and convergence later. The choice depends on business invariants and latency/availability needs.

</details>

### Q9. What is quorum?
<details><summary>Answer</summary>

A quorum requires enough replicas to participate in reads/writes so overlapping quorums can provide consistency properties. Exact guarantees depend on the system and protocol.

</details>

### Q10. What is leader election?
<details><summary>Answer</summary>

Nodes select a leader to coordinate certain operations. The design must handle leader failure, stale leaders and split-brain concerns.

</details>

### Q11. What is split brain?
<details><summary>Answer</summary>

Two nodes/groups believe they are authoritative simultaneously, potentially causing conflicting writes. Fencing, quorum and robust consensus mechanisms mitigate it.

</details>

### Q12. What is idempotency?
<details><summary>Answer</summary>

Repeating an operation yields the same intended business outcome. It is essential because distributed retries and duplicate messages are normal.

</details>

### Q13. Exactly-once processing—is it real?
<details><summary>Answer</summary>

Exactly-once end-to-end business effects are difficult. Systems often provide at-least-once delivery plus idempotent processing, transactional boundaries or deduplication to achieve effectively-once business behavior.

</details>

## Scalability & Resilience

### Q14. Vertical vs horizontal scaling?
<details><summary>Answer</summary>

Vertical adds resources to a machine; horizontal adds instances. Horizontal scaling improves capacity/resilience but requires distributed-state and coordination strategies.

</details>

### Q15. What is a load balancer?
<details><summary>Answer</summary>

It distributes requests among healthy backend instances and can provide health checks, TLS termination, routing and connection management.

</details>

### Q16. What is backpressure?
<details><summary>Answer</summary>

It limits producer rate/concurrency based on consumer capacity, preventing unbounded queues and resource exhaustion.

</details>

### Q17. What is a circuit breaker?
<details><summary>Answer</summary>

It stops repeated calls to an unhealthy dependency, allowing recovery and protecting callers from cascading failures.

</details>

### Q18. Bulkhead pattern?
<details><summary>Answer</summary>

Isolate resources such as thread pools or connection pools so failure/overload in one dependency does not exhaust the entire service.

</details>

### Q19. What is load shedding?
<details><summary>Answer</summary>

Intentionally reject or degrade lower-priority work during overload to preserve critical functionality and system stability.

</details>

### Q20. How do retries cause outages?
<details><summary>Answer</summary>

Synchronized retries multiply traffic against an already failing dependency. Use bounded attempts, backoff, jitter and circuit breaking.

</details>

### Q21. How do you design for AZ failure?
<details><summary>Answer</summary>

Run stateless compute across zones, use resilient data services/replicas, health-aware routing and test failure scenarios.

</details>

## Caching

### Q22. When should you cache?
<details><summary>Answer</summary>

Cache data that is frequently read, expensive to compute/fetch and tolerant of the chosen staleness.

</details>

### Q23. Cache-aside?
<details><summary>Answer</summary>

Application reads cache first; on miss it loads the source, then populates cache. Writes generally update the source and invalidate/update cache.

</details>

### Q24. Write-through vs write-back?
<details><summary>Answer</summary>

Write-through updates cache and source together from the application's perspective. Write-back writes cache first and persists later, improving write latency but increasing durability/complexity risk.

</details>

### Q25. Cache stampede?
<details><summary>Answer</summary>

Many requests miss simultaneously and overload the source. Use TTL jitter, request coalescing, locking, early refresh or stale-while-revalidate.

</details>

### Q26. Hot key problem?
<details><summary>Answer</summary>

One extremely popular key concentrates load on one cache shard/node. Replication, local caching or key distribution strategies can reduce pressure.

</details>

## Messaging & Events

### Q27. Queue vs pub/sub?
<details><summary>Answer</summary>

A queue distributes work among consumers; pub/sub lets multiple subscribers independently receive an event.

</details>

### Q28. Kafka partitioning?
<details><summary>Answer</summary>

Partitions provide ordering within a partition and enable parallel consumption. Keys influence partition placement and therefore ordering scope.

</details>

### Q29. What determines Kafka consumer throughput?
<details><summary>Answer</summary>

Partition count, consumer parallelism, batch size, processing time, broker performance and downstream capacity.

</details>

### Q30. How do you handle poison messages?
<details><summary>Answer</summary>

Limit retries, route permanently failing messages to a DLQ/quarantine, preserve diagnostic context and provide replay tooling.

</details>

### Q31. Outbox pattern?
<details><summary>Answer</summary>

Write the business change and an event/outbox record in the same database transaction, then publish the outbox asynchronously. This avoids the dual-write inconsistency.

</details>

### Q32. Event-driven vs synchronous architecture?
<details><summary>Answer</summary>

Events decouple producers/consumers and improve buffering/scaling, but add eventual consistency and operational complexity. Synchronous calls are simpler when immediate responses and tight coupling are acceptable.

</details>

## Data & APIs

### Q33. SQL vs NoSQL decision?
<details><summary>Answer</summary>

Start with access patterns, relationships, transaction requirements, consistency and scale. Do not choose NoSQL merely because the system is 'large'.

</details>

### Q34. Sharding?
<details><summary>Answer</summary>

Partition data across nodes using a shard key. A good key balances load and supports common access patterns while avoiding hot shards.

</details>

### Q35. Read replicas?
<details><summary>Answer</summary>

Replicas scale reads and provide resilience, but introduce replication lag and require a strategy for read-after-write behavior.

</details>

### Q36. Optimistic vs pessimistic concurrency?
<details><summary>Answer</summary>

Optimistic detects conflicts during commit/update; pessimistic locks resources earlier. Optimistic often scales better when conflicts are rare.

</details>

### Q37. Cursor vs offset pagination?
<details><summary>Answer</summary>

Offset is easy but can become slow and unstable at large offsets. Cursor/keyset pagination is efficient and stable for large ordered datasets.

</details>

### Q38. How should APIs handle retries?
<details><summary>Answer</summary>

Use idempotent semantics or idempotency keys, stable error contracts and explicit timeout/retry behavior.

</details>

## Classic Design Problems

### Q39. Design a URL shortener.
<details><summary>Answer</summary>

API → ID/code generation → persistent mapping → cache → redirect. Discuss uniqueness, read-heavy scaling, expiration, abuse, analytics and multi-region trade-offs.

</details>

### Q40. Design a notification system.
<details><summary>Answer</summary>

API → durable queue → channel workers → providers. Add preferences, templates, retries, DLQ, idempotency, rate limits, provider failover and delivery tracking.

</details>

### Q41. Design a rate limiter.
<details><summary>Answer</summary>

Choose fixed/sliding window, token bucket or leaky bucket. For distributed limits, use an atomic shared store and define fail-open/closed behavior.

</details>

### Q42. Design an e-commerce order system.
<details><summary>Answer</summary>

Separate catalog, cart, order, payment, inventory and fulfillment boundaries. Use synchronous validation where necessary and events/outbox for asynchronous propagation.

</details>

### Q43. Design a payment system.
<details><summary>Answer</summary>

Model explicit payment states, idempotency keys, provider adapters, reconciliation, durable audit trails and ambiguous outcome handling. Never assume a timeout means payment failed.

</details>

### Q44. Design a file upload service.
<details><summary>Answer</summary>

Use direct object-storage uploads with pre-signed URLs where appropriate, metadata service, virus/content validation, size limits, asynchronous processing and lifecycle policies.

</details>

### Q45. Design a job scheduler.
<details><summary>Answer</summary>

Persist jobs, lease work, coordinate workers, enforce idempotency, handle retries/misfires and use durable execution metadata. Avoid relying on in-memory timers alone.

</details>

### Q46. Design a distributed lock.
<details><summary>Answer</summary>

Use a system with well-defined lease/ownership semantics and fencing where stale holders could corrupt state. Always define timeout and failure behavior.

</details>

## LLD & Object Design

### Q47. What is SOLID?
<details><summary>Answer</summary>

Single Responsibility, Open/Closed, Liskov Substitution, Interface Segregation and Dependency Inversion. Apply them to reduce coupling and improve changeability—not to create needless abstractions.

</details>

### Q48. Strategy vs Factory?
<details><summary>Answer</summary>

Strategy encapsulates interchangeable behavior; Factory encapsulates object creation. A factory can select a strategy.

</details>

### Q49. Adapter vs Facade?
<details><summary>Answer</summary>

Adapter converts one interface to another. Facade provides a simplified interface over a subsystem.

</details>

### Q50. Decorator vs Proxy?
<details><summary>Answer</summary>

Decorator adds behavior while preserving an interface; Proxy controls access to another object. They can look similar structurally but serve different intent.

</details>

### Q51. Design a payment provider abstraction.
<details><summary>Answer</summary>

Define a provider interface, isolate provider-specific DTOs/adapters, centralize state transitions/idempotency and use a factory/strategy to select the provider.

</details>

### Q52. How do you make LLD thread-safe?
<details><summary>Answer</summary>

Prefer immutable objects and confinement. For shared mutable state, protect invariants using locks/atomics/concurrent collections and keep critical sections small.

</details>

## Security & Observability

### Q53. Authentication vs authorization?
<details><summary>Answer</summary>

Authentication establishes identity; authorization determines permitted actions.

</details>

### Q54. How do you protect APIs?
<details><summary>Answer</summary>

TLS, authentication, authorization, validation, rate limiting, input/output controls, secure secrets and audit logging. Never trust frontend authorization.

</details>

### Q55. What should you log?
<details><summary>Answer</summary>

Log structured business/technical events with correlation IDs and enough context for diagnosis, while avoiding secrets, tokens and sensitive data.

</details>

### Q56. What are the three pillars of observability?
<details><summary>Answer</summary>

Logs, metrics and traces provide complementary views. Modern observability also emphasizes profiles, events and high-quality context.

</details>

### Q57. How do you debug a latency spike?
<details><summary>Answer</summary>

Break latency into queueing, application CPU, GC, database, cache and downstream spans; compare percentiles and saturation against baseline and recent changes.

</details>

## Question Count

**57 experienced-level questions** in this file.

## Quick Revision Checklist

