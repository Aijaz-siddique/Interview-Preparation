# System Design Interview Preparation — HLD + LLD

## HLD

### Q1. How do you start a system-design interview?
<details><summary>Answer</summary>

Do not immediately draw boxes.

Start with:
1. functional requirements
2. non-functional requirements
3. users/actors
4. scale estimates
5. latency/availability goals
6. consistency requirements
7. constraints

Then define APIs, data model and architecture.
</details>

### Q2. How do you estimate capacity?
<details><summary>Answer</summary>

Estimate:
- users
- requests/day
- average and peak RPS
- read/write ratio
- payload size
- storage/day
- retention
- bandwidth

For example, 10 million requests/day is roughly 116 requests/sec on average. Peak traffic may be several times higher.

Use estimates to justify architecture instead of choosing technologies arbitrarily.
</details>

### Q3. Scale up vs scale out?
<details><summary>Answer</summary>

Scale up means adding CPU/RAM to a machine. Scale out means adding instances.

Scale out usually improves horizontal resilience and capacity, but introduces distributed-system complexity such as load balancing, shared state, coordination and consistency.
</details>

### Q4. When would you use a cache?
<details><summary>Answer</summary>

Use caching when repeated reads are expensive and data can tolerate the chosen staleness.

Consider:
- cache-aside
- TTL
- invalidation
- hot keys
- stampede protection
- cache size
- eviction
- consistency

Caching is not free; stale data and operational complexity are trade-offs.
</details>

### Q5. SQL vs NoSQL?
<details><summary>Answer</summary>

SQL is usually a strong default when relationships, transactions and flexible querying matter.

NoSQL can be appropriate for specific access patterns requiring high scale, flexible schemas or specialized distribution models.

Choose from business access patterns, consistency, scale and operational requirements—not from fashion.
</details>

### Q6. When would you introduce Kafka/message queues?
<details><summary>Answer</summary>

Use asynchronous messaging when you need decoupling, buffering, independent scaling, event-driven workflows or reliable asynchronous processing.

Questions to answer:
- ordering
- delivery semantics
- retries
- DLQ
- duplicate events
- consumer lag
- replay
- partitioning
</details>

### Q7. Explain idempotency in distributed systems.
<details><summary>Answer</summary>

An operation is idempotent when repeating it produces the same intended business result.

This matters because retries are unavoidable.

Examples:
- PUT with a deterministic resource state
- payment request with an idempotency key
- unique database constraint for a business operation
</details>

### Q8. How do you design for high availability?
<details><summary>Answer</summary>

Remove single points of failure:
- multiple application instances
- load balancing
- multi-zone deployment
- replicated databases
- durable messaging
- health checks
- automated failover
- backups and disaster recovery

Define the availability target first; architecture follows the requirement.
</details>

### Q9. Strong vs eventual consistency?
<details><summary>Answer</summary>

Strong consistency means reads observe the latest committed state according to the system's consistency model.

Eventual consistency allows temporary divergence with convergence later.

Use strong consistency for invariants such as certain financial operations; eventual consistency can be excellent for feeds, search indexes and derived views.
</details>

### Q10. What is a circuit breaker?
<details><summary>Answer</summary>

A circuit breaker stops repeatedly calling a failing dependency.

Typical states:
- closed
- open
- half-open

It protects the caller from cascading failures. It should be combined with timeouts, bounded retries and appropriate fallback behavior.
</details>

### Q11. How do you prevent cascading failures?
<details><summary>Answer</summary>

Use:
- timeouts
- bounded retries
- exponential backoff/jitter
- circuit breakers
- bulkheads
- rate limits
- queues
- load shedding
- connection pool limits

A retry storm can be worse than the original outage.
</details>

### Q12. Design a URL shortener.
<details><summary>Answer</summary>

Core flow:
Client → API → ID/code generator → persistent mapping → cache → redirect.

Discuss:
- unique short code generation
- collision handling
- read-heavy caching
- TTL/expiry
- abuse controls
- analytics
- multi-region considerations
- availability vs consistency

A good candidate also estimates read/write traffic and storage.
</details>

### Q13. Design a notification system.
<details><summary>Answer</summary>

Separate request acceptance from delivery:

API → notification service → durable queue → channel workers (email/SMS/push) → provider.

Add:
- templates
- preferences
- retry policy
- DLQ
- idempotency
- rate limits
- provider failover
- delivery status
- observability
</details>

### Q14. Design a rate limiter.
<details><summary>Answer</summary>

Common algorithms:
- fixed window
- sliding window
- token bucket
- leaky bucket

For distributed systems, state may live in Redis or another distributed store.

Consider atomicity, clock behavior, hot keys, per-user vs global limits and fail-open/fail-closed behavior.
</details>

## LLD

### Q15. What SOLID principles do you use in production?
<details><summary>Answer</summary>

- **S**ingle Responsibility
- **O**pen/Closed
- **L**iskov Substitution
- **I**nterface Segregation
- **D**ependency Inversion

Don't treat SOLID as rules to create more classes. Use it to reduce coupling and improve changeability/testability.
</details>

### Q16. Strategy vs Factory pattern?
<details><summary>Answer</summary>

Strategy encapsulates interchangeable behavior.

Factory encapsulates object creation.

They are often combined: a factory selects the appropriate strategy implementation.
</details>

### Q17. Design an extensible payment system.
<details><summary>Answer</summary>

Use abstractions around payment providers, e.g. `PaymentProvider`.

Concrete providers implement the interface. A strategy/factory selects the provider.

Keep business orchestration independent from provider-specific APIs.

Handle idempotency, status transitions, retries and reconciliation explicitly.
</details>

### Q18. How do you design thread-safe classes?
<details><summary>Answer</summary>

Prefer immutability. If mutable shared state is necessary, protect invariants with synchronized/lock/atomic mechanisms.

Document thread-safety assumptions and avoid exposing internal mutable collections.
</details>

### Q19. What makes an API well designed?
<details><summary>Answer</summary>

Consider:
- resource modeling
- HTTP semantics
- status codes
- validation
- pagination
- filtering
- versioning
- idempotency
- consistent errors
- authentication/authorization
- backward compatibility
- observability

Good APIs are designed for evolution, not only today's consumers.
</details>

### Q20. What should you discuss when reviewing a design?
<details><summary>Answer</summary>

Ask:
- Where is the bottleneck?
- What happens when a dependency fails?
- What happens during retries?
- What is the consistency model?
- What happens at 10x traffic?
- Where is state stored?
- How is data recovered?
- How is the system observed?
- What is the security boundary?
- What are the operational costs?

## Last-Minute HLD Checklist

Requirements → scale → APIs → data → architecture → cache → queue → consistency → availability → failure → security → observability → bottlenecks → trade-offs.

## Last-Minute LLD Checklist

SOLID → interfaces → composition → patterns → extensibility → immutability → concurrency → testability.
