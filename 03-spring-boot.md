# Spring Boot Interview Preparation

### Q1. What problem does Spring Boot solve?
<details><summary>Answer</summary>

Spring Boot reduces application setup and configuration by providing opinionated defaults, auto-configuration, starter dependencies, embedded servers and production-oriented features.

It does not replace Spring; it builds on Spring.
</details>

### Q2. How does Spring Boot auto-configuration work?
<details><summary>Answer</summary>

Boot evaluates configuration conditions based on the classpath, existing beans, properties and other conditions, then registers appropriate beans.

Typical conditions include:
- class present
- bean missing
- property enabled
- application type

Understanding conditions is essential when debugging unexpected/missing beans.
</details>

### Q3. What are Spring Boot starters?
<details><summary>Answer</summary>

Starters are curated dependency descriptors that bring a compatible group of libraries for a capability, reducing manual dependency management.

Examples include web, data JPA, validation and actuator starters.
</details>

### Q4. How do profiles work?
<details><summary>Answer</summary>

Profiles allow environment-specific configuration and beans.

Typical examples are `dev`, `test`, `staging` and `prod`.

Avoid embedding environment-specific secrets in source code. Use external secret/configuration mechanisms.
</details>

### Q5. How would you externalize configuration?
<details><summary>Answer</summary>

Use application properties/YAML, environment variables, command-line arguments and external configuration systems as appropriate.

For secrets, use a secret-management solution rather than committing passwords/API keys into Git.
</details>

### Q6. What is Actuator?
<details><summary>Answer</summary>

Spring Boot Actuator exposes operational endpoints for health, metrics, environment and other management information.

Production deployments should expose only appropriate endpoints and protect sensitive management data.
</details>

### Q7. Liveness vs readiness?
<details><summary>Answer</summary>

Liveness answers whether the application process should be restarted.

Readiness answers whether the application should receive traffic.

Confusing them can cause restart loops or traffic being sent to an instance that is not ready.
</details>

### Q8. How would you design exception handling in a REST API?
<details><summary>Answer</summary>

Use centralized exception handling, commonly with `@RestControllerAdvice`.

Return consistent error contracts containing useful information such as code, message, timestamp/correlation identifier where appropriate.

Do not leak stack traces or internal implementation details to clients.
</details>

### Q9. How do you validate REST request payloads?
<details><summary>Answer</summary>

Use Jakarta Bean Validation annotations such as `@NotNull`, `@Size` and `@Valid`.

Validation belongs at the API boundary, while business invariants must still be enforced in the service/domain layer.
</details>

### Q10. How do you prevent duplicate POST processing?
<details><summary>Answer</summary>

Use idempotency keys or business-level unique constraints depending on the operation.

A robust solution usually combines:
- idempotency key
- persistent request/result state
- database uniqueness
- safe retry semantics

An in-memory flag is not sufficient in a multi-instance deployment.
</details>

### Q11. How do you troubleshoot a slow Spring Boot API?
<details><summary>Answer</summary>

Measure first:
- request latency percentiles
- DB query time
- downstream calls
- thread pools
- connection pools
- GC
- CPU/memory
- serialization
- cache hit rate

Use tracing/profiling and database execution plans to identify the actual bottleneck.
</details>

### Q12. How do you handle graceful shutdown?
<details><summary>Answer</summary>

The application should stop accepting new work while allowing in-flight work enough time to finish.

In Kubernetes, this works with termination lifecycle, readiness behavior, termination grace period and application shutdown configuration.

Long-running tasks need explicit handling.
</details>

## Quick Revision Checklist

Auto-configuration → starters → profiles → configuration precedence → Actuator → health → readiness/liveness → REST errors → validation → idempotency → graceful shutdown → observability.
