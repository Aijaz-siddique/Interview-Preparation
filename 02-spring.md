# Spring Interview Preparation

### Q1. Explain Spring IoC and Dependency Injection.
<details><summary>Answer</summary>

IoC means object creation/configuration/lifecycle is controlled by the Spring container rather than application classes manually constructing their dependencies.

Dependency Injection supplies dependencies to objects, typically through constructors.

Constructor injection is generally preferred because dependencies are explicit, objects can be immutable, and required dependencies cannot be omitted accidentally.
</details>

### Q2. What is the Spring bean lifecycle?
<details><summary>Answer</summary>

At a high level:
1. Bean definition is discovered.
2. Bean is instantiated.
3. Dependencies are injected.
4. Bean post-processors run.
5. Initialization callbacks run.
6. Bean is available for use.
7. Destruction callbacks run when the context shuts down.

`BeanPostProcessor` is particularly important because Spring features such as proxies rely heavily on post-processing.
</details>

### Q3. Explain singleton scope in Spring.
<details><summary>Answer</summary>

Spring singleton means one bean instance per Spring `ApplicationContext`, not one instance per JVM globally.

Singleton beans must be designed carefully when multiple requests/threads use them. Avoid mutable request-specific state inside singleton beans.
</details>

### Q4. How does `@Autowired` work?
<details><summary>Answer</summary>

Spring resolves dependencies from the application context and injects matching beans.

With multiple candidates, use mechanisms such as `@Qualifier` or `@Primary`.

A single constructor can generally be used without explicitly adding `@Autowired`.
</details>

### Q5. Explain Spring AOP.
<details><summary>Answer</summary>

AOP separates cross-cutting concerns such as transactions, security, logging and metrics from core business logic.

Spring commonly implements AOP using proxies. This creates important limitations, such as self-invocation not necessarily passing through the proxy.

Know the difference between join points, pointcuts, advice and proxies.
</details>

### Q6. Why can `@Transactional` fail on self-invocation?
<details><summary>Answer</summary>

Spring's transaction interceptor is typically applied through a proxy. If a method inside the same object calls another method on `this`, the call bypasses the proxy, so the interceptor may not execute.

Solutions include restructuring the service boundary or invoking through the proxied bean when appropriate.
</details>

### Q7. Explain propagation in Spring transactions.
<details><summary>Answer</summary>

Propagation defines how a transactional method behaves when called from an existing transaction.

Common modes:
- `REQUIRED`: join existing or create new
- `REQUIRES_NEW`: suspend existing and create new
- `SUPPORTS`: use existing if present
- `MANDATORY`: require existing

The correct choice depends on business atomicity and failure semantics.
</details>

### Q8. What is transaction isolation?
<details><summary>Answer</summary>

Isolation controls how concurrent transactions can observe each other's changes.

Common anomalies include:
- dirty reads
- non-repeatable reads
- phantom reads

Higher isolation can improve consistency but may reduce concurrency. The actual behavior also depends on the database engine.
</details>

### Q9. How would you debug a bean creation failure?
<details><summary>Answer</summary>

Read the deepest/root `Caused by`, not just the top-level exception.

Check:
- missing bean
- component scanning
- profile/configuration
- circular dependency
- conditional configuration
- incorrect constructor
- incompatible dependency versions
- environment properties

Spring's condition evaluation information can be valuable in Boot applications.
</details>

### Q10. What is the difference between Filter, Interceptor and AOP?
<details><summary>Answer</summary>

A servlet Filter operates at the servlet/container boundary.

A Spring MVC Interceptor operates around controller request processing.

AOP operates around Spring-managed method execution and is useful for cross-cutting application concerns.

Choose based on the layer where the concern belongs.
</details>

### Q11. How do you handle circular dependencies?
<details><summary>Answer</summary>

First question whether the design itself is wrong. Circular dependencies often indicate tightly coupled responsibilities.

Prefer redesigning boundaries. Constructor injection also makes many cycles obvious early.

Do not solve every cycle by introducing lazy injection without understanding the underlying design.
</details>

### Q12. How would you make a Spring service production-ready?
<details><summary>Answer</summary>

Consider:
- externalized configuration
- health/readiness checks
- metrics and tracing
- structured logging
- timeouts
- retries with backoff
- circuit breaking where appropriate
- database pool limits
- graceful shutdown
- security
- validation
- idempotency
- deployment strategy
- tests

Production readiness is broader than "the endpoint works."
</details>

## Quick Revision Checklist

IoC → DI → bean lifecycle → scopes → AOP → proxies → transactions → propagation → isolation → MVC layers → filters/interceptors → configuration → production readiness.
