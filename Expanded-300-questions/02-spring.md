# Spring Interview Preparation — Experienced Level

Focus: IoC, DI, bean lifecycle, AOP, transactions, MVC, concurrency and production design.

## IoC, DI & Beans

### Q1. What is IoC?
<details><summary>Answer</summary>

Inversion of Control means object creation and lifecycle are controlled by a container/framework rather than each class constructing its dependencies.

</details>

### Q2. Why is constructor injection preferred?
<details><summary>Answer</summary>

It makes required dependencies explicit, supports immutability, improves testability and exposes circular dependencies early.

</details>

### Q3. Explain Spring bean lifecycle.
<details><summary>Answer</summary>

Definition discovery → instantiation → dependency injection → aware callbacks/post-processors → initialization callbacks → use → destruction callbacks. Exact hooks depend on configuration.

</details>

### Q4. Singleton bean vs singleton design pattern?
<details><summary>Answer</summary>

Spring singleton means one bean instance per ApplicationContext. It is not a JVM-wide singleton pattern.

</details>

### Q5. Prototype scope?
<details><summary>Answer</summary>

A prototype bean is created when requested from the container, with Spring generally not managing the full destruction lifecycle after handing it to the caller.

</details>

### Q6. Request/session scopes?
<details><summary>Answer</summary>

These scopes associate bean instances with an HTTP request or session. They require an appropriate web-aware context.

</details>

### Q7. What is @Configuration?
<details><summary>Answer</summary>

It marks a configuration class whose bean methods are managed by Spring. Full configuration classes can be enhanced so inter-bean method calls respect container semantics.

</details>

### Q8. What is @Bean?
<details><summary>Answer</summary>

It declares an object to be registered as a Spring bean, useful especially for third-party classes or explicit configuration.

</details>

### Q9. Component vs Service vs Repository?
<details><summary>Answer</summary>

They are stereotype annotations expressing roles. Service and repository provide semantic intent; repository also participates in exception translation in supported Spring data stacks.

</details>

### Q10. How does component scanning work?
<details><summary>Answer</summary>

Spring scans configured packages for stereotype/component classes and registers bean definitions. Incorrect base packages are a common reason for missing beans.

</details>

## AOP & Proxies

### Q11. What is AOP?
<details><summary>Answer</summary>

Aspect-oriented programming separates cross-cutting concerns such as transactions, security and metrics from business logic.

</details>

### Q12. Advice vs pointcut?
<details><summary>Answer</summary>

Advice is the action executed around a join point. A pointcut selects where the advice applies.

</details>

### Q13. JDK proxy vs class-based proxy?
<details><summary>Answer</summary>

JDK dynamic proxies work through interfaces. Class-based proxies subclass concrete classes. The exact mechanism depends on configuration and framework version.

</details>

### Q14. Why does self-invocation break proxy-based advice?
<details><summary>Answer</summary>

Calling another method through `this` bypasses the proxy, so proxy interceptors such as transactions may not run.

</details>

### Q15. Can private methods be advised by proxy AOP?
<details><summary>Answer</summary>

Not through normal proxy interception because calls to private methods cannot be overridden/intercepted by the proxy mechanism.

</details>

### Q16. AOP vs Filter vs Interceptor?
<details><summary>Answer</summary>

Filter operates at servlet/container level, MVC interceptor around request/controller processing, and AOP around Spring-managed method execution.

</details>

## Transactions

### Q17. What does @Transactional do?
<details><summary>Answer</summary>

It applies transaction interception around a method according to configured transaction manager and propagation/isolation rules.

</details>

### Q18. Explain REQUIRED vs REQUIRES_NEW.
<details><summary>Answer</summary>

REQUIRED joins an existing transaction or creates one. REQUIRES_NEW suspends an existing transaction and starts an independent one.

</details>

### Q19. What is transaction rollback behavior?
<details><summary>Answer</summary>

By default, Spring commonly rolls back for unchecked exceptions; checked exception behavior can be configured. Always make rollback policy explicit for important business flows.

</details>

### Q20. What is readOnly=true?
<details><summary>Answer</summary>

It communicates that a transaction is intended for reads and may allow optimizations. It should not be treated as an absolute prohibition on writes across all database implementations.

</details>

### Q21. What is isolation?
<details><summary>Answer</summary>

Isolation controls visibility of concurrent transaction changes and affects anomalies such as dirty, non-repeatable and phantom reads.

</details>

### Q22. What is optimistic locking?
<details><summary>Answer</summary>

Transactions detect conflicting updates using a version/timestamp rather than holding a database lock for the entire business operation.

</details>

### Q23. What is pessimistic locking?
<details><summary>Answer</summary>

The database locks rows/resources to prevent conflicting concurrent operations. It can reduce concurrency and must be used carefully to avoid lock contention/deadlocks.

</details>

### Q24. Why is transaction boundary important?
<details><summary>Answer</summary>

The boundary defines atomicity and consistency. Too broad increases lock duration and resource usage; too narrow can leave partially completed business operations.

</details>

## Spring MVC & Web

### Q25. Explain a Spring MVC request flow.
<details><summary>Answer</summary>

A request reaches the servlet layer, DispatcherServlet selects a handler, argument resolution/validation occurs, controller executes, and response conversion/exception handling produces the HTTP response.

</details>

### Q26. Controller vs RestController?
<details><summary>Answer</summary>

RestController combines Controller semantics with response-body behavior, making returned values typically serialized directly.

</details>

### Q27. How does validation work?
<details><summary>Answer</summary>

Jakarta Bean Validation annotations define constraints; `@Valid`/`@Validated` triggers validation at supported boundaries. Business invariants still belong in the domain/service layer.

</details>

### Q28. How should API errors be centralized?
<details><summary>Answer</summary>

Use controller advice/exception handlers to map exceptions to a stable error contract. Avoid leaking internal stack traces.

</details>

### Q29. Filter vs interceptor use cases?
<details><summary>Answer</summary>

Filters are ideal for low-level request/response concerns such as correlation IDs or security integration. Interceptors are useful for MVC-level concerns around controller execution.

</details>

### Q30. How do you handle file uploads safely?
<details><summary>Answer</summary>

Validate size/type/content, use streaming where possible, avoid trusting filenames, store outside executable paths, scan where required, and apply authorization and resource limits.

</details>

## Advanced Spring

### Q31. How do conditional beans work?
<details><summary>Answer</summary>

Spring can create configuration/beans only when conditions such as class presence, properties or missing beans are satisfied.

</details>

### Q32. What is BeanPostProcessor?
<details><summary>Answer</summary>

It can modify or wrap beans before/after initialization. Many framework features use post-processors to create proxies or apply behavior.

</details>

### Q33. What is ApplicationContext?
<details><summary>Answer</summary>

It extends basic bean-factory capabilities with configuration, events, resource loading, internationalization and application lifecycle support.

</details>

### Q34. ApplicationEvent vs direct method call?
<details><summary>Answer</summary>

Events decouple publishers from listeners and can be useful for in-process domain/application events. They add indirection and should not be confused with durable distributed messaging.

</details>

### Q35. What is @Async?
<details><summary>Answer</summary>

It can execute methods asynchronously through a configured executor. Thread pools, exception handling and context propagation must be designed explicitly.

</details>

### Q36. What is Spring scheduling?
<details><summary>Answer</summary>

Scheduling triggers methods based on fixed rates, delays or cron-like schedules. In multi-instance deployments, ensure jobs are not unintentionally executed concurrently on every instance.

</details>

### Q37. How do you test Spring components?
<details><summary>Answer</summary>

Use focused unit tests for business logic and targeted integration tests for framework/database behavior. Avoid loading the entire application context for every test.

</details>

### Q38. How do you avoid overusing Spring?
<details><summary>Answer</summary>

Keep business logic framework-light where possible. Framework annotations should express infrastructure boundaries rather than replace sound domain design.

</details>

## Question Count

**38 experienced-level questions** in this file.

## Quick Revision Checklist

