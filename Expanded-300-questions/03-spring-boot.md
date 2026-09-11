# Spring Boot Interview Preparation — Experienced Level

Focus: internals, production APIs, observability, reliability, data, testing and deployment.

## Boot Fundamentals

### Q1. What does Spring Boot add to Spring?
<details><summary>Answer</summary>

Opinionated auto-configuration, starters, embedded runtime support, externalized configuration and operational features reduce setup and convention-heavy configuration.

</details>

### Q2. How does auto-configuration work?
<details><summary>Answer</summary>

Boot evaluates conditions based on classpath, existing beans, properties and application type, then registers appropriate infrastructure beans.

</details>

### Q3. What is a starter?
<details><summary>Answer</summary>

A curated dependency descriptor that brings a compatible group of libraries for a capability, reducing manual dependency selection.

</details>

### Q4. What is configuration property binding?
<details><summary>Answer</summary>

External configuration can be bound to typed objects, allowing validated, structured application settings rather than scattering string lookups throughout code.

</details>

### Q5. How do profiles work?
<details><summary>Answer</summary>

Profiles activate environment-specific configuration and beans. They should not be used as a substitute for proper secret/configuration management.

</details>

### Q6. What is configuration precedence?
<details><summary>Answer</summary>

Spring Boot combines multiple configuration sources with defined precedence. Understand which source wins rather than relying on an assumed order; document production overrides.

</details>

## Production APIs

### Q7. How do you design REST error responses?
<details><summary>Answer</summary>

Use a stable error schema with machine-readable code, human-readable message and optional correlation/field details. Keep internal stack traces out of client responses.

</details>

### Q8. How do you prevent duplicate requests?
<details><summary>Answer</summary>

Use idempotency keys or business uniqueness constraints, persist processing state when necessary, and make retries safe across multiple instances.

</details>

### Q9. How do you implement pagination?
<details><summary>Answer</summary>

Offset pagination is simple but can become expensive/unstable at large offsets. Cursor/keyset pagination is often better for large or frequently changing datasets.

</details>

### Q10. How do you version APIs?
<details><summary>Answer</summary>

Prefer additive/backward-compatible evolution where possible. Use explicit versioning when contract breaks are unavoidable and define deprecation timelines.

</details>

### Q11. What is graceful shutdown?
<details><summary>Answer</summary>

Stop accepting new work, allow in-flight operations to finish within a bounded grace period, and coordinate readiness/termination with the orchestrator.

</details>

## Actuator & Observability

### Q12. What is Spring Boot Actuator?
<details><summary>Answer</summary>

It exposes operational endpoints for health, metrics and management information. Production exposure must be secured and minimized.

</details>

### Q13. Liveness vs readiness?
<details><summary>Answer</summary>

Liveness determines whether a process should be restarted; readiness determines whether it should receive traffic.

</details>

### Q14. What should a health check test?
<details><summary>Answer</summary>

Readiness should verify critical dependencies only to the extent necessary to safely serve traffic. Avoid checks that create cascading failures or make the service flap.

</details>

### Q15. What is Micrometer?
<details><summary>Answer</summary>

It provides a metrics instrumentation facade that integrates with monitoring systems and supports application/infrastructure metrics.

</details>

### Q16. What is distributed tracing?
<details><summary>Answer</summary>

Tracing follows a request across services using trace/span context. It helps identify latency and failure across distributed call chains.

</details>

### Q17. Structured logging vs plain text?
<details><summary>Answer</summary>

Structured logs encode fields such as timestamp, service, request ID and error code, making search and aggregation reliable.

</details>

## Data & Reliability

### Q18. How do you configure a database connection pool?
<details><summary>Answer</summary>

Set pool size based on database capacity, workload and latency; configure timeouts and monitor active/idle/pending connections. Bigger pools are not automatically faster.

</details>

### Q19. How do you handle downstream timeouts?
<details><summary>Answer</summary>

Set bounded connect/read/call timeouts. Combine with limited retries, backoff/jitter, circuit breaking and fallback where appropriate.

</details>

### Q20. How should retries be designed?
<details><summary>Answer</summary>

Retry only transient/idempotent operations, use exponential backoff with jitter and a strict attempt/time budget. Avoid retry storms.

</details>

### Q21. How do you handle secrets?
<details><summary>Answer</summary>

Keep secrets outside source control and container images. Use a managed secret mechanism, least privilege and rotation.

</details>

### Q22. How do you secure actuator endpoints?
<details><summary>Answer</summary>

Expose only necessary endpoints, authenticate/authorize management access and isolate sensitive information from public traffic.

</details>

## Testing & Deployment

### Q23. Unit vs integration vs contract tests?
<details><summary>Answer</summary>

Unit tests isolate logic. Integration tests validate component interactions. Contract tests validate producer/consumer API expectations without requiring every dependency in every test.

</details>

### Q24. How do you test Kafka/database integrations?
<details><summary>Answer</summary>

Use realistic integration environments or test containers where practical, verify transactional/idempotency behavior and avoid mocks that hide serialization or query problems.

</details>

### Q25. Blue/green vs canary?
<details><summary>Answer</summary>

Blue/green switches traffic between two environments. Canary gradually sends a small percentage to the new version and observes health before increasing exposure.

</details>

### Q26. How do you make database migrations safe?
<details><summary>Answer</summary>

Use backward-compatible expand/contract changes: add new schema first, deploy compatible application code, migrate data, then remove old structures after consumers are upgraded.

</details>

## Question Count

**26 experienced-level questions** in this file.

## Quick Revision Checklist

