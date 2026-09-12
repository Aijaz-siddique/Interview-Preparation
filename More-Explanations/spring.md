# Spring Framework — Interview Questions (Experienced)

<details>
<summary>1. What is Inversion of Control (IoC) and Dependency Injection (DI)?</summary>

**IoC** is a principle where control of object creation and lifecycle is handed over to a framework/container instead of the application code creating objects directly (`new`).

**DI** is the mechanism Spring uses to implement IoC — dependencies are "injected" into a class from outside, rather than the class constructing them itself. This decouples classes from their dependencies' concrete implementations, making code easier to test (mock dependencies) and maintain.
</details>

<details>
<summary>2. What are the different types of Dependency Injection in Spring?</summary>

- **Constructor injection** (recommended) — dependencies passed via constructor; allows `final` fields, guarantees the object is fully initialized, works well with immutability, and makes circular dependencies fail fast at startup.
- **Setter injection** — dependencies set via setter methods; useful for optional dependencies.
- **Field injection** (`@Autowired` directly on a field) — concise but discouraged: hides dependencies, makes unit testing harder without reflection, and allows circular dependencies to slip through undetected until runtime.

```java
@Service
public class OrderService {
    private final PaymentService paymentService;

    public OrderService(PaymentService paymentService) { // constructor injection
        this.paymentService = paymentService;
    }
}
```
</details>

<details>
<summary>3. What is the Spring IoC Container? Difference between `BeanFactory` and `ApplicationContext`?</summary>

The **IoC Container** is responsible for instantiating, configuring, and managing the lifecycle of beans.

- **`BeanFactory`**: the basic container — lazy initialization, minimal features.
- **`ApplicationContext`**: a superset of `BeanFactory` — eager initialization of singletons by default, supports internationalization, event publishing, AOP integration, and annotation-driven configuration. Almost always what you use in practice (`AnnotationConfigApplicationContext`, or auto-configured in Spring Boot).
</details>

<details>
<summary>4. Explain Spring Bean scopes.</summary>

- **`singleton`** (default) — one shared instance per Spring container.
- **`prototype`** — a new instance every time the bean is requested.
- **`request`** — one instance per HTTP request (web-aware contexts only).
- **`session`** — one instance per HTTP session.
- **`application`** — one instance per `ServletContext`.
- **`websocket`** — one instance per WebSocket session.

```java
@Scope("prototype")
@Component
public class ShoppingCart { }
```
</details>

<details>
<summary>5. Explain the Spring Bean lifecycle.</summary>

1. Container instantiates the bean.
2. Dependencies are injected (constructor/setter/field).
3. `BeanNameAware`, `BeanFactoryAware`, `ApplicationContextAware` callbacks (if implemented) run.
4. `@PostConstruct` / `InitializingBean.afterPropertiesSet()` / custom `init-method` runs.
5. Bean is ready to use.
6. On container shutdown: `@PreDestroy` / `DisposableBean.destroy()` / custom `destroy-method` runs (singleton scope only — prototype beans aren't managed after creation).
</details>

<details>
<summary>6. What is AOP (Aspect-Oriented Programming) in Spring? What are advice types?</summary>

AOP lets you modularize cross-cutting concerns (logging, security, transactions) that would otherwise be scattered across many classes, by defining them once as **aspects** applied via **pointcuts**.

Advice types:
- **`@Before`** — runs before the method.
- **`@After`** — runs after (regardless of outcome).
- **`@AfterReturning`** — runs after successful return.
- **`@AfterThrowing`** — runs if an exception is thrown.
- **`@Around`** — wraps the method call; can control whether it proceeds at all, and modify the return value.

Spring AOP is proxy-based (JDK dynamic proxies for interfaces, CGLIB for classes) and only applies to Spring-managed beans, and only intercepts calls that go **through the proxy** (internal self-invocation within the same class bypasses AOP).
</details>

<details>
<summary>7. What is `@Transactional` and how does it work under the hood?</summary>

`@Transactional` declaratively wraps a method in a database transaction — commits on success, rolls back on a `RuntimeException` (unchecked) by default.

Under the hood, Spring creates a **proxy** around the bean. Calling the annotated method actually calls the proxy, which starts a transaction, invokes the real method, then commits/rolls back based on the outcome.

**Common gotcha**: self-invocation. If method A calls method B on `this` within the same class, and B is `@Transactional`, the proxy is bypassed — B runs in A's existing transaction context (or none at all), not a new one, because the call never goes through the Spring proxy.
</details>

<details>
<summary>8. Explain transaction propagation levels (`REQUIRED`, `REQUIRES_NEW`, etc.).</summary>

- **`REQUIRED`** (default) — join the existing transaction if one exists, else create a new one.
- **`REQUIRES_NEW`** — always suspend any existing transaction and start a new, independent one.
- **`NESTED`** — starts a nested transaction (savepoint) within the existing one if present; a rollback in the nested transaction doesn't necessarily roll back the outer one.
- **`SUPPORTS`** — join if a transaction exists, else run non-transactionally.
- **`MANDATORY`** — must run within an existing transaction; throws an exception if none exists.
- **`NOT_SUPPORTED`** — suspends any existing transaction and runs non-transactionally.
- **`NEVER`** — throws an exception if a transaction exists.
</details>

<details>
<summary>9. What's the difference between `@Component`, `@Service`, `@Repository`, and `@Controller`?</summary>

All are specializations of `@Component` and are functionally similar for bean registration/scanning purposes — the distinction is mostly **semantic**, aiding readability and enabling extra behavior:

- **`@Component`** — generic stereotype for any Spring-managed bean.
- **`@Service`** — marks a service-layer (business logic) bean.
- **`@Repository`** — marks a data-access-layer bean; Spring also applies **exception translation** here, converting persistence-specific exceptions (e.g., JDBC `SQLException`) into Spring's unified `DataAccessException` hierarchy.
- **`@Controller`** — marks a web controller (MVC); combined with `@ResponseBody` (or as `@RestController`) for REST APIs.
</details>

<details>
<summary>10. How does Spring resolve bean autowiring by type vs by name? What happens with multiple candidates?</summary>

`@Autowired` resolves by **type** first. If multiple beans of the same type exist, Spring tries to match **by field/parameter name** against bean names. If still ambiguous, it throws `NoUniqueBeanDefinitionException` unless you disambiguate with:

- **`@Qualifier("beanName")`** — explicitly pick a bean by name.
- **`@Primary`** — mark one candidate as the default choice when multiple exist.

```java
@Autowired
@Qualifier("paypalPaymentService")
private PaymentService paymentService;
```
</details>

<details>
<summary>11. What is a circular dependency in Spring, and how do you resolve it?</summary>

A circular dependency occurs when bean A depends on bean B, and B (directly or transitively) depends on A. Spring can resolve this for **setter/field injection** singletons using a three-level cache (early bean references), but it **cannot** resolve circular dependencies for **constructor injection** — it fails fast at startup with a `BeanCurrentlyInCreationException` (since Spring 5.x/Boot 2.6, this even applies more strictly by default).

Resolutions: redesign to remove the cycle (often signals a design smell — extract shared logic into a third bean), use `@Lazy` on one of the dependencies, or switch to setter injection as a last resort.
</details>

<details>
<summary>12. Explain `@Configuration` and `@Bean`. What does `@Configuration`'s CGLIB proxying do?</summary>

- **`@Bean`** — declares a method whose return value is registered as a Spring bean.
- **`@Configuration`** — marks a class as a source of bean definitions.

Spring proxies `@Configuration` classes with CGLIB so that calling one `@Bean` method from another **within the same class** returns the same singleton instance from the container, rather than creating a new object each time:

```java
@Configuration
public class AppConfig {
    @Bean
    public DataSource dataSource() { return new DataSource(); }

    @Bean
    public UserRepository userRepository() {
        return new UserRepository(dataSource()); // returns the SAME dataSource singleton
    }
}
```

(Using `@Configuration(proxyBeanMethods = false)` disables this — appropriate when you don't rely on inter-bean method calls, for a startup performance gain.)
</details>

<details>
<summary>13. What is the difference between Spring MVC and Spring WebFlux?</summary>

- **Spring MVC** — synchronous, blocking, thread-per-request model built on the Servlet API. Simple mental model, works well for typical CRUD/backend services.
- **Spring WebFlux** — reactive, non-blocking, built on Project Reactor (`Mono`/`Flux`) and Netty (or other reactive servers). Designed for high-concurrency, I/O-bound workloads where blocking threads would be wasteful (e.g., many slow downstream calls). Steeper learning curve, and the whole call chain (including DB drivers) needs to be non-blocking to get the benefit.

Choose WebFlux when you genuinely need to handle very high concurrency with limited threads; otherwise MVC (or now, MVC + virtual threads on Java 21+) is usually simpler and sufficient.
</details>

<details>
<summary>14. What is Spring Security and how does the filter chain work at a high level?</summary>

Spring Security handles authentication (who are you) and authorization (what can you do) via a chain of **servlet filters** that intercept every request before it reaches your controller.

Key filters (order matters): `SecurityContextPersistenceFilter` (loads the security context), authentication filters (e.g., `UsernamePasswordAuthenticationFilter`, JWT filter), `ExceptionTranslationFilter` (handles auth exceptions), `FilterSecurityInterceptor` (authorization decision). Each filter can short-circuit the chain (e.g., reject unauthenticated requests) or pass control to the next filter.
</details>

<details>
<summary>15. What is the difference between `@RequestParam`, `@PathVariable`, and `@RequestBody`?</summary>

- **`@RequestParam`** — binds a query parameter or form field: `/users?id=5` → `@RequestParam Long id`.
- **`@PathVariable`** — binds a URI template segment: `/users/{id}` → `@PathVariable Long id`.
- **`@RequestBody`** — binds the entire HTTP request body (typically JSON) to a Java object, deserialized via `HttpMessageConverter` (Jackson by default).
</details>

<details>
<summary>16. What is the Spring `Environment` abstraction?</summary>

`Environment` provides unified access to properties (from `application.properties`, system properties, environment variables, command-line args) and active profiles, allowing your code to query configuration regardless of where it originally came from.
</details>

<details>
<summary>17. What are Spring Profiles and how do you use them?</summary>

Profiles let you register beans conditionally based on the active environment (`dev`, `test`, `prod`). Mark a bean with `@Profile("dev")`, and activate it via `spring.profiles.active=dev` (property) or `-Dspring.profiles.active=dev` (JVM arg). Useful for swapping implementations (e.g., in-memory DB for dev, real DB for prod) without code changes.
</details>

<details>
<summary>18. What is `@Value` used for, and how do you provide a default?</summary>

Injects a value from a property source directly into a field/parameter: `@Value("${server.port}")`. Provide a default with the colon syntax: `@Value("${server.port:8080}")` — used if the property is absent.
</details>

<details>
<summary>19. What is `@ConfigurationProperties` and how does it differ from `@Value`?</summary>

`@ConfigurationProperties` binds a whole group of related properties to a strongly-typed POJO (supports nested objects, lists, validation via `@Validated`), rather than injecting one property at a time like `@Value`. Preferred for structured configuration blocks.

```java
@ConfigurationProperties(prefix = "app.mail")
public class MailProperties {
    private String host;
    private int port;
}
```
</details>

<details>
<summary>20. What is component scanning, and how does `@ComponentScan` work?</summary>

Component scanning automatically discovers and registers classes annotated with `@Component` (and its specializations) as beans, without explicit `@Bean` declarations for each one. `@ComponentScan` (implicitly included in `@SpringBootApplication`) specifies the base package(s) to scan.
</details>

<details>
<summary>21. What is the difference between XML-based, annotation-based, and Java-based Spring configuration?</summary>

**XML config** — legacy, verbose, defines beans in `applicationContext.xml`. **Annotation-based** — beans marked with stereotypes (`@Component`, `@Service`) and auto-discovered via scanning. **Java-based (`@Configuration` + `@Bean`)** — beans defined explicitly in Java code, type-safe and refactor-friendly. Modern Spring/Spring Boot apps almost exclusively use annotation + Java config.
</details>

<details>
<summary>22. What is `FactoryBean` in Spring?</summary>

An interface for beans that act as factories for other beans — instead of the object itself being registered directly, `getObject()` returns the actual bean instance to be used, useful for complex object creation logic that can't be expressed as a simple constructor call (common in library integration code).
</details>

<details>
<summary>23. What is the difference between `@PostConstruct` and a constructor?</summary>

The constructor runs during instantiation, potentially before all dependencies are injected (for setter/field injection). `@PostConstruct` runs **after** the bean is fully constructed and all dependencies are injected, making it the right place for initialization logic that depends on those injected values.
</details>

<details>
<summary>24. What is a `BeanPostProcessor`?</summary>

A hook into the bean lifecycle allowing custom logic to run before (`postProcessBeforeInitialization`) and after (`postProcessAfterInitialization`) a bean's initialization callbacks — this is how Spring implements features like `@Autowired` processing and AOP proxy creation internally.
</details>

<details>
<summary>25. What is a `BeanFactoryPostProcessor`, and how is it different from `BeanPostProcessor`?</summary>

`BeanFactoryPostProcessor` operates on **bean definitions** (metadata) before any beans are actually instantiated — e.g., `PropertySourcesPlaceholderConfigurer` resolves `${...}` placeholders in bean definitions. `BeanPostProcessor` operates on actual **bean instances** after they're created.
</details>

<details>
<summary>26. What is `@Lazy` and when would you use it?</summary>

Delays bean creation until it's first needed, instead of at container startup (the default for singletons). Useful for breaking certain circular dependency issues, reducing startup time for rarely-used heavy beans, or when a bean's dependencies aren't ready yet at startup.
</details>

<details>
<summary>27. What is the difference between `@Autowired` and `@Inject`?</summary>

`@Autowired` is Spring-specific. `@Inject` is from the standard `javax.inject`/`jakarta.inject` (JSR-330) API, supported by Spring for portability across DI frameworks. Functionally similar, but `@Autowired` has Spring-specific extras like `required = false`.
</details>

<details>
<summary>28. Can you inject a `List` or `Map` of all beans of a certain type in Spring?</summary>

Yes — `@Autowired List<PaymentStrategy> strategies;` injects all beans implementing `PaymentStrategy`, and `@Autowired Map<String, PaymentStrategy> strategies;` injects them keyed by bean name. Useful for strategy-pattern-style dispatch without a big `if/else` chain.
</details>

<details>
<summary>29. What is `ApplicationEvent` and `ApplicationListener`? How do you publish custom events?</summary>

Spring's built-in observer pattern implementation for decoupled, in-process event communication. Publish via `ApplicationEventPublisher.publishEvent(event)`; listen via `@EventListener` (or implementing `ApplicationListener<T>`). By default listeners run synchronously in the publisher's thread unless marked `@Async`.
</details>

<details>
<summary>30. What is `@Async` in Spring, and what do you need to enable it?</summary>

Runs a method in a separate thread (from a configured `TaskExecutor`) instead of blocking the caller. Requires `@EnableAsync` on a configuration class. The method must be called from **outside the class** (proxy-based, like `@Transactional`) and typically returns `void`, `Future<T>`, or `CompletableFuture<T>`.
</details>

<details>
<summary>31. What is Spring's `@Cacheable`, `@CacheEvict`, and `@CachePut`?</summary>

Declarative caching support: `@Cacheable` — caches a method's return value, skipping the method body on subsequent calls with the same key. `@CachePut` — always executes the method but updates the cache with the result. `@CacheEvict` — removes an entry (or clears the cache) — typically used on update/delete operations to keep the cache consistent.
</details>

<details>
<summary>32. What is the difference between `@RestController` and `@Controller`?</summary>

`@Controller` returns view names to be resolved (server-side rendered pages, e.g., Thymeleaf). `@RestController` = `@Controller` + `@ResponseBody` on every method — the return value is serialized directly into the HTTP response body (typically JSON), making it the standard choice for REST APIs.
</details>

<details>
<summary>33. What is `HandlerInterceptor` in Spring MVC?</summary>

Lets you hook into the request lifecycle at three points — `preHandle` (before controller execution, can short-circuit), `postHandle` (after controller, before view rendering), `afterCompletion` (after the full response is sent). Used for cross-cutting web concerns like logging, auth checks, or timing — similar in spirit to a filter but with more context about the specific handler being invoked.
</details>

<details>
<summary>34. What is the difference between a Servlet `Filter` and a Spring `HandlerInterceptor`?</summary>

A `Filter` operates at the raw Servlet container level, before the request even reaches Spring's `DispatcherServlet` — framework-agnostic, can wrap/modify the request or response. `HandlerInterceptor` is Spring-MVC-specific, operating within the `DispatcherServlet`'s processing, with awareness of which controller method will handle the request.
</details>

<details>
<summary>35. What is `@ControllerAdvice` / `@ExceptionHandler` used for?</summary>

`@ExceptionHandler` on a method handles specific exceptions thrown within that controller. `@ControllerAdvice` (or `@RestControllerAdvice`) makes exception handlers **global**, applying across all controllers — the standard way to implement centralized error handling and consistent error response formats in a REST API.

```java
@RestControllerAdvice
public class GlobalExceptionHandler {
    @ExceptionHandler(ResourceNotFoundException.class)
    public ResponseEntity<ErrorResponse> handleNotFound(ResourceNotFoundException ex) {
        return ResponseEntity.status(404).body(new ErrorResponse(ex.getMessage()));
    }
}
```
</details>

<details>
<summary>36. What is the `DispatcherServlet` and what is its role?</summary>

The front controller in Spring MVC — a single servlet that receives all incoming HTTP requests, delegates to the appropriate handler (controller method) via `HandlerMapping`, invokes it, and resolves the result (view or response body) — the central coordination point of the whole MVC request-processing flow.
</details>

<details>
<summary>37. What is content negotiation in Spring MVC?</summary>

The process of determining the response format (JSON, XML, etc.) based on the client's `Accept` header, URL suffix, or request parameter — Spring MVC's `ContentNegotiationManager` picks the appropriate `HttpMessageConverter` to serialize the response accordingly.
</details>

<details>
<summary>38. What is `ResponseEntity` and why use it over just returning an object?</summary>

`ResponseEntity<T>` gives full control over the HTTP response — status code, headers, and body — rather than always returning `200 OK` with just the serialized object. Standard practice for building well-formed REST APIs (`201 Created` with a `Location` header, `404 Not Found`, etc.).
</details>

<details>
<summary>39. What is a Spring `Validator` and how does bean validation (`@Valid`) work?</summary>

`@Valid` on a `@RequestBody` parameter triggers Bean Validation (JSR-380/Jakarta Validation) annotations on the DTO (`@NotNull`, `@Size`, `@Email`) — Spring validates the object automatically and throws `MethodArgumentNotValidException` on failure, typically handled via `@ExceptionHandler` to return a structured 400 response. Custom `Validator` implementations can be plugged in for complex, cross-field validation logic.
</details>

<details>
<summary>40. What is Spring Data JPA, and what problem does it solve?</summary>

An abstraction over JPA that eliminates most boilerplate DAO code — you declare a repository interface (e.g., extending `JpaRepository<Entity, ID>`) and Spring generates the implementation at runtime, including CRUD methods and query derivation from method names (`findByLastNameAndAge`).
</details>

<details>
<summary>41. What is the difference between `CrudRepository`, `PagingAndSortingRepository`, and `JpaRepository`?</summary>

`CrudRepository` — basic CRUD operations. `PagingAndSortingRepository` extends it with pagination/sorting support. `JpaRepository` extends both and adds JPA-specific features (batch operations, flushing). Each is a superset of the previous — `JpaRepository` is what's typically used in practice.
</details>

<details>
<summary>42. What is derived query methods in Spring Data JPA? Give an example.</summary>

Spring parses a repository method's name and generates the query automatically, no implementation needed: `List<User> findByEmailAndStatus(String email, Status status);` translates to a `WHERE email = ? AND status = ?` query. Keeps simple queries declarative and boilerplate-free.
</details>

<details>
<summary>43. What is `@Query` used for in Spring Data JPA?</summary>

Lets you write an explicit JPQL or native SQL query when derived method names would be too complex or unreadable:

```java
@Query("SELECT u FROM User u WHERE u.status = :status")
List<User> findActiveUsers(@Param("status") Status status);
```
</details>

<details>
<summary>44. What is the N+1 query problem, and how do you fix it in JPA/Hibernate?</summary>

Occurs when fetching a list of entities triggers 1 query for the list, then N additional queries — one per entity — to lazily fetch a related association. Fixes: use `JOIN FETCH` in a JPQL query, `@EntityGraph` to specify eager fetch paths for a specific query, or batch fetching (`hibernate.default_batch_fetch_size`).
</details>

<details>
<summary>45. What is the difference between `FetchType.LAZY` and `FetchType.EAGER`?</summary>

`LAZY` — the association is only loaded from the DB when actually accessed (can cause `LazyInitializationException` if accessed outside an active persistence context). `EAGER` — loaded immediately along with the owning entity, which can cause performance issues (over-fetching) if not needed. `LAZY` is generally the safer default, applied explicitly where needed.
</details>

<details>
<summary>46. What is the Hibernate first-level and second-level cache?</summary>

**First-level cache** — scoped to a single `Session`/`EntityManager`, always enabled, ensures identical entity lookups within one transaction return the same object without hitting the DB twice. **Second-level cache** — optional, shared across sessions (e.g., via Ehcache/Redis), caches entities across transactions/requests, must be explicitly configured and used carefully to avoid stale-data issues.
</details>

<details>
<summary>47. What is optimistic vs pessimistic locking in JPA?</summary>

**Optimistic locking** — assumes conflicts are rare; uses a `@Version` field checked at commit time, throwing `OptimisticLockException` if the record changed since it was read. **Pessimistic locking** — acquires a DB-level lock (`SELECT ... FOR UPDATE`) upfront, blocking other transactions from modifying the row until released — safer under high contention but reduces concurrency.
</details>

<details>
<summary>48. What is the difference between `save()`, `saveAndFlush()`, and `flush()` in Spring Data JPA?</summary>

`save()` persists/merges the entity, but the actual SQL `INSERT`/`UPDATE` might be deferred until the persistence context flushes (e.g., at transaction commit or the next query). `flush()` forces pending changes to be synchronized with the database immediately. `saveAndFlush()` does both in one call — useful when you need the DB state updated before continuing (e.g., before a native query in the same transaction).
</details>

<details>
<summary>49. What is `EntityManager` and `PersistenceContext`?</summary>

`EntityManager` is the JPA interface for managing entity persistence operations (`persist`, `merge`, `find`, `remove`). The `PersistenceContext` is the first-level cache/set of managed entities associated with an `EntityManager` — entities within it are tracked for changes (dirty checking) and synchronized to the DB on flush.
</details>

<details>
<summary>50. What is dirty checking in Hibernate?</summary>

Hibernate automatically detects changes made to managed (attached) entities within a transaction by comparing their current state to a snapshot taken when they were loaded, and generates the necessary `UPDATE` SQL at flush time — you don't need to explicitly call `save()` again after modifying a managed entity's fields.
</details>

<details>
<summary>51. What is the Spring `RestTemplate` vs `WebClient`?</summary>

`RestTemplate` — the older, synchronous/blocking HTTP client, now in maintenance mode (not actively enhanced). `WebClient` (from Spring WebFlux) — the modern, non-blocking, reactive HTTP client, usable in both reactive and traditional MVC applications (can be used synchronously via `.block()` if needed), the recommended choice for new code.
</details>

<details>
<summary>52. What is Spring Cloud, and name a few of its components.</summary>

A set of tools built on Spring Boot for common distributed-systems/microservices patterns. Components: **Eureka** (service discovery), **Spring Cloud Gateway** (API gateway/routing), **Config Server** (centralized externalized configuration), **Resilience4j/Hystrix** (circuit breakers), **Sleuth/Micrometer Tracing** (distributed tracing), **OpenFeign** (declarative REST clients).
</details>

<details>
<summary>53. What is a Circuit Breaker pattern, and how is it implemented in Spring (Resilience4j)?</summary>

Prevents a failing downstream service from being repeatedly hammered by requests, which would waste resources and worsen cascading failures. The circuit "opens" after a failure threshold is crossed (calls fail fast without even trying the downstream service), then transitions to "half-open" to test recovery before fully "closing" again. Implemented declaratively with `@CircuitBreaker(name = "...", fallbackMethod = "...")` from Resilience4j.
</details>

<details>
<summary>54. What is service discovery, and how does Eureka work at a high level?</summary>

Service discovery lets services find each other's network locations dynamically instead of hardcoding hostnames/ports (which change constantly in cloud/container environments). Services register themselves with a Eureka server on startup (with periodic heartbeats), and clients query Eureka to resolve a logical service name to a current, healthy instance address.
</details>

<details>
<summary>55. What is Spring Cloud Config, and why use externalized configuration?</summary>

A centralized configuration server that serves environment-specific properties to multiple microservices from a single source (often a Git repo), instead of bundling config inside each service's jar. Enables changing configuration without rebuilding/redeploying services, and keeps configuration consistent and auditable across a fleet of services.
</details>

<details>
<summary>56. What is idempotency, and why does it matter for REST APIs?</summary>

An idempotent operation produces the same end result no matter how many times it's performed. `GET`, `PUT`, and `DELETE` are expected to be idempotent by HTTP convention; `POST` typically is not. Matters heavily for retry logic in distributed systems — safely retrying a network-timed-out request requires the operation to be idempotent, or you risk duplicate side effects (e.g., double-charging a payment).
</details>

<details>
<summary>57. How would you design a REST API for pagination in Spring?</summary>

Use `Pageable` as a controller method parameter (Spring Data auto-binds `page`, `size`, `sort` query params) and return a `Page<T>` (includes content, total elements, total pages) or map it to a custom paginated response DTO to avoid leaking internal Spring Data types in your public API contract.
</details>

<details>
<summary>58. What is HATEOAS, and does Spring support it?</summary>

Hypermedia As The Engine Of Application State — REST responses include links to related/available actions, letting clients navigate the API dynamically rather than hardcoding URLs. Spring HATEOAS provides `EntityModel`/`CollectionModel`/`Link` builders to add this hypermedia layer to your responses.
</details>

<details>
<summary>59. What is CORS, and how do you configure it in Spring?</summary>

Cross-Origin Resource Sharing — a browser security mechanism restricting web pages from making requests to a different origin than the one that served the page, unless the server explicitly allows it via response headers. Configure globally via a `WebMvcConfigurer` bean's `addCorsMappings()`, or per-controller/method with `@CrossOrigin`.
</details>

<details>
<summary>60. What is the difference between authentication and authorization in Spring Security?</summary>

**Authentication** — verifying *who* the user is (login, validating credentials/tokens). **Authorization** — determining *what* an authenticated user is allowed to do (role/permission checks, e.g., `@PreAuthorize("hasRole('ADMIN')")`).
</details>

<details>
<summary>61. What is JWT, and how is it typically used with Spring Security?</summary>

A JSON Web Token is a compact, self-contained, digitally signed token carrying claims (user identity, roles, expiry). Typical flow: client authenticates once, server issues a JWT; the client sends it in the `Authorization: Bearer <token>` header on subsequent requests; a custom filter validates the signature/expiry and populates the `SecurityContext`, avoiding server-side session storage (stateless auth).
</details>

<details>
<summary>62. What is `SecurityContextHolder`?</summary>

A thread-local holder for the current `SecurityContext`, which contains the `Authentication` object representing the currently authenticated user — accessible anywhere in the call stack of a request via `SecurityContextHolder.getContext().getAuthentication()` without passing it explicitly through method parameters.
</details>

<details>
<summary>63. What is method-level security in Spring, and how do you enable it?</summary>

Lets you apply authorization checks directly on service/controller methods using annotations like `@PreAuthorize`, `@PostAuthorize`, `@Secured`. Enabled via `@EnableMethodSecurity` (or the older `@EnableGlobalMethodSecurity`) on a configuration class.

```java
@PreAuthorize("hasRole('ADMIN') or #userId == authentication.principal.id")
public void deleteUser(Long userId) { ... }
```
</details>

<details>
<summary>64. What is CSRF, and when do you need CSRF protection?</summary>

Cross-Site Request Forgery — an attack that tricks an authenticated user's browser into submitting an unwanted request (using their existing session cookie) to your server. Relevant for **cookie/session-based** authentication in browser apps; typically **not needed** for stateless, token-based APIs (JWT in headers) since there's no ambient session cookie for a malicious site to exploit.
</details>

<details>
<summary>65. What is the difference between `PasswordEncoder` implementations like `BCryptPasswordEncoder`?</summary>

`PasswordEncoder` hashes passwords one-way for secure storage. `BCryptPasswordEncoder` is the standard recommendation — it's intentionally slow (configurable cost factor) to resist brute-force attacks, and automatically salts each hash, so identical passwords produce different hashes.
</details>

<details>
<summary>66. What is OAuth2, and what roles do Authorization Server, Resource Server, and Client play?</summary>

OAuth2 is a delegated-authorization framework. The **Authorization Server** authenticates the user and issues access tokens. The **Resource Server** hosts protected APIs and validates incoming tokens. The **Client** (your application) requests tokens on the user's behalf and uses them to call the Resource Server — this separation lets a single identity provider serve many independent client apps and APIs.
</details>

<details>
<summary>67. What is the difference between Spring Security's `UserDetailsService` and `AuthenticationProvider`?</summary>

`UserDetailsService` loads user-specific data (username, password hash, roles) typically from a database, given a username. `AuthenticationProvider` performs the actual authentication logic — it uses a `UserDetailsService` (in the common `DaoAuthenticationProvider` implementation) plus a `PasswordEncoder` to verify credentials and produce an authenticated `Authentication` object.
</details>

<details>
<summary>68. What is a Spring `Filter` chain order issue, and how do you control filter ordering?</summary>

Filters must run in a specific order for correct behavior (e.g., an authentication filter must run before an authorization check). Control ordering via `FilterRegistrationBean.setOrder()`, `@Order` annotation, or — in Spring Security specifically — by explicitly positioning custom filters relative to built-in ones (`addFilterBefore`, `addFilterAfter`) in the `SecurityFilterChain` configuration.
</details>

<details>
<summary>69. What is the difference between `@RequestMapping`, `@GetMapping`, `@PostMapping`, etc.?</summary>

`@GetMapping`, `@PostMapping`, `@PutMapping`, `@DeleteMapping`, `@PatchMapping` are specialized, more readable shortcuts for `@RequestMapping(method = RequestMethod.GET/POST/...)` — functionally equivalent, just more concise and self-documenting.
</details>

<details>
<summary>70. What is content type `application/json` handling — how does Spring convert Java objects to/from JSON automatically?</summary>

Via `HttpMessageConverter`s registered in Spring MVC — by default, `MappingJackson2HttpMessageConverter` (backed by the Jackson library) handles JSON serialization/deserialization automatically for `@RequestBody`/`@ResponseBody`, as long as Jackson is on the classpath (which it is by default in `spring-boot-starter-web`).
</details>

<details>
<summary>71. What is the Spring TestContext framework, and what does `@SpringBootTest` do?</summary>

The TestContext framework manages loading and caching the Spring `ApplicationContext` across test classes for integration testing. `@SpringBootTest` boots the full (or a sliced) application context for integration tests, letting you autowire real beans and test end-to-end behavior rather than isolated units.
</details>

<details>
<summary>72. What is the difference between `@Mock`, `@MockBean`, and `@Spy`?</summary>

`@Mock` (Mockito) — creates a plain mock object, used in pure unit tests without a Spring context. `@MockBean` (Spring Boot Test) — creates a Mockito mock **and registers it in the Spring context**, replacing the real bean — used in Spring-context-based tests (e.g., `@WebMvcTest`). `@Spy` — wraps a real object, allowing you to stub only specific methods while others call through to the real implementation.
</details>

<details>
<summary>73. What are Spring Boot test slices (`@WebMvcTest`, `@DataJpaTest`)?</summary>

Test slices load only a relevant subset of the application context instead of the whole application, making tests faster and more focused. `@WebMvcTest` loads only the web layer (controllers, filters, `@ControllerAdvice`) with mocked service dependencies. `@DataJpaTest` loads only JPA-related components with an in-memory test database, ideal for testing repository queries in isolation.
</details>

<details>
<summary>74. What is `TestRestTemplate`/`WebTestClient` used for?</summary>

Both are used for end-to-end integration testing of a running application over HTTP. `TestRestTemplate` — synchronous, for traditional MVC apps. `WebTestClient` — the reactive equivalent, also usable to test WebFlux apps or to make fluent, chainable assertions on REST responses even for MVC apps.
</details>

<details>
<summary>75. What is Testcontainers and why use it with Spring Boot integration tests?</summary>

A library that spins up real, disposable Docker containers (a real Postgres, Kafka, Redis, etc.) for integration tests, giving you far higher fidelity than mocks or in-memory substitutes (like H2 pretending to be Postgres) — catching real DB-dialect-specific bugs before production.
</details>

<details>
<summary>76. What is the difference between unit testing and integration testing in a typical Spring application?</summary>

**Unit tests** test a single class in isolation, mocking all its dependencies — fast, no Spring context needed. **Integration tests** verify multiple components working together (e.g., controller → service → repository → real/test database), typically with a (partial or full) Spring context loaded, slower but catching wiring/configuration issues unit tests can't.
</details>

<details>
<summary>77. What is `Mockito.when().thenReturn()` vs `doReturn().when()`? When would you use the latter?</summary>

`when(mock.method()).thenReturn(value)` is the standard, readable form. `doReturn(value).when(mock).method()` is needed when mocking `void` methods, or when stubbing a `spy()` where calling the real method inside `when()` could throw or have side effects — `doReturn` avoids actually invoking the real method during stub setup.
</details>

<details>
<summary>78. What is `ArgumentCaptor` in Mockito, and when would you use it?</summary>

Captures the actual arguments passed to a mock's method call during a test, letting you assert on their content — useful when you need to verify not just *that* a method was called, but *what* was passed to it (e.g., verifying the exact object saved to a repository).
</details>

<details>
<summary>79. What is Spring Boot's `@ConditionalOnProperty`, `@ConditionalOnMissingBean`, etc.?</summary>

Part of Spring's conditional bean registration mechanism (heavily used in Boot's auto-configuration). `@ConditionalOnProperty` — register a bean only if a specific property has a certain value. `@ConditionalOnMissingBean` — register a bean only if no other bean of that type already exists, letting users override auto-configured defaults simply by defining their own bean.
</details>

<details>
<summary>80. How does Spring resolve ambiguity when multiple `@Configuration` classes define beans with the same name?</summary>

The bean defined **last** to be processed generally wins (overrides earlier ones) if bean overriding is enabled — though Spring Boot disables bean definition overriding by default since 2.1, throwing a `BeanDefinitionOverrideException` instead, forcing you to explicitly opt back in (`spring.main.allow-bean-definition-overriding=true`) or resolve the conflict deliberately.
</details>

<details>
<summary>81. What is the difference between `ApplicationContext.getBean()` and dependency injection — when (if ever) should you use `getBean()` directly?</summary>

`getBean()` is a programmatic, service-locator-style lookup — generally discouraged since it hides dependencies and couples code to the Spring API directly. Legitimate uses are rare: dynamic bean selection at runtime based on data not known at compile time, or within framework/infrastructure code itself.
</details>

<details>
<summary>82. What is the `@Order` annotation used for?</summary>

Controls the ordering of beans when Spring injects multiple candidates as a collection (e.g., a `List` of filters, interceptors, or AOP advice) — lower values run/appear earlier. Also affects the execution order of multiple `@EventListener`s for the same event.
</details>

<details>
<summary>83. What is Spring's `TaskScheduler` and `@Scheduled`?</summary>

`@Scheduled` (with `@EnableScheduling`) lets you run a method periodically — fixed rate, fixed delay, or cron expression — without a separate scheduling library. Backed internally by a `TaskScheduler`, which you can customize (e.g., configure the thread pool size) to avoid all scheduled tasks contending for a single default thread.
</details>

<details>
<summary>84. What is the actuator module in Spring Boot, and what does it provide?</summary>

`spring-boot-starter-actuator` exposes production-ready operational endpoints out of the box: `/actuator/health` (liveness/readiness), `/actuator/metrics` (application/JVM metrics), `/actuator/info`, `/actuator/env`, and more — essential for monitoring and integrating with tools like Prometheus/Grafana in production.
</details>

<details>
<summary>85. What is `Micrometer`, and how does it relate to Spring Boot Actuator?</summary>

Micrometer is a vendor-neutral application metrics facade (similar in spirit to SLF4J for logging) — Spring Boot Actuator uses it internally to collect metrics, which can then be exported to various monitoring backends (Prometheus, Datadog, New Relic) simply by adding the corresponding Micrometer registry dependency, without changing your instrumentation code.
</details>

<details>
<summary>86. What is the difference between horizontal and vertical scaling, and how does Spring Boot's statelessness support horizontal scaling?</summary>

**Vertical scaling** — adding more resources (CPU/RAM) to a single instance. **Horizontal scaling** — running more instances behind a load balancer. Stateless Spring Boot services (no server-side session state stored in memory, e.g., using JWTs instead of HTTP sessions) can be scaled horizontally trivially — any instance can handle any request.
</details>

<details>
<summary>87. What is graceful shutdown, and how do you configure it in Spring Boot?</summary>

Allows in-flight requests to complete before the application shuts down (instead of abruptly killing connections), important during deployments/rolling restarts. Configure with `server.shutdown=graceful` and a `spring.lifecycle.timeout-per-shutdown-phase` to bound how long it waits.
</details>

<details>
<summary>88. What is the difference between `@PreDestroy` and a JVM shutdown hook?</summary>

`@PreDestroy` is a Spring bean-lifecycle callback, invoked when the Spring container itself shuts down that specific bean (singleton scope). A JVM shutdown hook (`Runtime.addShutdownHook()`) is a lower-level mechanism that runs on JVM termination regardless of any framework, useful for cleanup logic that must run even if the Spring context never fully starts.
</details>

<details>
<summary>89. What is connection pooling, and what's the default connection pool in Spring Boot?</summary>

Connection pooling reuses a limited set of database connections across requests instead of opening/closing a new one for every query, which is expensive. Spring Boot uses **HikariCP** by default — configured via `spring.datasource.hikari.*` properties (max pool size, connection timeout, etc.).
</details>

<details>
<summary>90. What is database migration, and how do Flyway/Liquibase fit into a Spring Boot project?</summary>

Database migration tools version-control schema changes as scripts, applied incrementally and tracked in a metadata table — ensuring all environments (dev, staging, prod) evolve their schema consistently and reproducibly. Flyway uses versioned SQL/Java migration files; Liquibase uses XML/YAML/JSON changelogs. Both auto-run on Spring Boot startup if included as a dependency, before the application accepts traffic.
</details>

<details>
<summary>91. What is a DTO, and why not just expose JPA entities directly in a REST API?</summary>

A Data Transfer Object is a plain object shaped specifically for API input/output, decoupled from the internal persistence model. Exposing entities directly risks: leaking internal DB structure/lazy-loading issues (serializing an uninitialized lazy collection can throw or trigger unwanted queries), tightly coupling your API contract to your schema (any DB change breaks the API), and over/under-exposing fields.
</details>

<details>
<summary>92. What is the Repository pattern, and how does Spring Data JPA relate to it?</summary>

The Repository pattern abstracts data access behind a collection-like interface, hiding persistence details from the business logic layer. Spring Data JPA implements this pattern by auto-generating the implementation of your repository interfaces at runtime via dynamic proxies, so you only declare the contract.
</details>

<details>
<summary>93. What is the Unit of Work pattern, and how does JPA's persistence context relate to it?</summary>

Unit of Work tracks a set of changes made during a business transaction and commits them together as a single atomic operation. JPA's persistence context does exactly this — it tracks all changes to managed entities and flushes them as a coordinated batch of SQL statements at transaction commit, rather than issuing separate immediate writes per change.
</details>

<details>
<summary>94. What is the difference between a monolith and microservices, and what does Spring Boot/Cloud offer for each?</summary>

A **monolith** is a single deployable unit containing all application functionality — simpler to develop/deploy/test initially, but scaling and team autonomy become harder as it grows. **Microservices** split functionality into independently deployable services — better scalability and team autonomy, at the cost of added operational complexity (network calls, distributed data consistency, service discovery). Spring Boot is well-suited for building individual services quickly; Spring Cloud adds the cross-cutting concerns (discovery, config, resilience, gateway) needed to operate them as a coherent system.
</details>

<details>
<summary>95. What is the Saga pattern for distributed transactions, and why can't you just use a normal DB transaction across microservices?</summary>

A traditional ACID transaction can't span multiple independent databases/services. The **Saga pattern** breaks a distributed transaction into a sequence of local transactions, each with a corresponding **compensating action** to undo it if a later step fails — achieving eventual consistency across services instead of atomicity, implemented either via choreography (services react to each other's events) or orchestration (a central coordinator directs the steps).
</details>

<details>
<summary>96. What is event-driven architecture, and how does Spring integrate with message brokers like Kafka/RabbitMQ?</summary>

Services communicate by publishing/consuming events asynchronously through a message broker rather than direct synchronous calls, improving decoupling and resilience to downstream slowness/failure. Spring provides `spring-kafka` and `spring-amqp`(RabbitMQ) with annotation-driven listeners (`@KafkaListener`, `@RabbitListener`) and a `KafkaTemplate`/`RabbitTemplate` for publishing, abstracting away much of the low-level client API boilerplate.
</details>

<details>
<summary>97. What is the difference between synchronous and asynchronous inter-service communication, and their trade-offs?</summary>

**Synchronous** (REST/gRPC call-and-wait) — simple to reason about, but couples the caller's availability/latency to the callee's, and failures cascade if not handled carefully (circuit breakers help). **Asynchronous** (message queue/event) — better decoupling and resilience (the consumer can be down temporarily without losing messages), but adds complexity: eventual consistency, message ordering, duplicate delivery handling.
</details>

<details>
<summary>98. What is API versioning, and what strategies exist for versioning a Spring REST API?</summary>

Versioning lets you evolve an API without breaking existing clients. Common strategies: **URI versioning** (`/api/v1/users`), **request parameter** (`?version=1`), **custom header** (`X-API-Version: 1`), or **media type/content negotiation** (`Accept: application/vnd.company.v1+json`). URI versioning is simplest and most common in practice despite being the least "RESTful" in theory.
</details>

<details>
<summary>99. What is rate limiting, and how might you implement it in a Spring Boot API?</summary>

Restricts how many requests a client can make in a given time window, protecting the service from abuse/overload. Implementations range from a simple `Bucket4j`-based token-bucket filter/interceptor within the app, to delegating it entirely to an API gateway (Spring Cloud Gateway with a `RequestRateLimiter` filter backed by Redis) sitting in front of the services.
</details>

<details>
<summary>100. What is `@RestControllerAdvice`'s role in building a consistent error response contract across a Spring Boot API?</summary>

It centralizes exception-to-HTTP-response mapping logic in one place, ensuring every error (validation failure, not-found, unexpected exception) is transformed into a consistent JSON structure (e.g., `{timestamp, status, error, message, path}`) rather than each controller handling errors inconsistently or leaking raw stack traces to clients.
</details>

<details>
<summary>101. What is the difference between `spring-boot-starter-web` and `spring-boot-starter-webflux`?</summary>

`spring-boot-starter-web` brings in Spring MVC on an embedded Servlet container (Tomcat by default) — the traditional, blocking stack. `spring-boot-starter-webflux` brings in the reactive stack on Netty by default — non-blocking, built around Project Reactor. You generally pick one or the other; mixing both on the classpath requires explicit configuration to avoid ambiguity about which stack to auto-configure.
</details>

<details>
<summary>102. What is `Mono` and `Flux` in Reactor (used by Spring WebFlux)?</summary>

`Mono<T>` represents an asynchronous stream of **0 or 1** element. `Flux<T>` represents an asynchronous stream of **0 to N** elements. Both are lazy — nothing happens until something subscribes — and support the same rich set of composable, non-blocking operators (`map`, `flatMap`, `filter`) as Java Streams, but for asynchronous data.
</details>

<details>
<summary>103. What is backpressure in reactive programming?</summary>

A mechanism allowing a slow consumer to signal to a fast producer how much data it can currently handle, preventing the consumer from being overwhelmed (buffer overflow, OOM). Reactive Streams (which Reactor implements) build backpressure into the core specification via the `request(n)` signal from subscriber to publisher.
</details>

<details>
<summary>104. What is the difference between `subscribe()` and `block()` in Reactor?</summary>

`subscribe()` triggers execution **asynchronously**, non-blocking — the calling thread continues immediately. `block()` triggers execution and **blocks** the calling thread until a result is available, essentially bridging back to imperative/blocking code — defeats the purpose of reactive programming if used pervasively, and should generally be avoided except at specific integration boundaries (e.g., tests, or bridging legacy blocking code).
</details>

<details>
<summary>105. What is a common mistake developers make when adopting Spring WebFlux from Spring MVC?</summary>

Mixing blocking calls (a blocking JDBC driver, `Thread.sleep()`, a blocking HTTP client) inside a reactive pipeline — this defeats the entire purpose of WebFlux's small, fixed-size event-loop thread pool, since a blocked event-loop thread can't process any other requests, potentially stalling the whole application under load. The whole call chain needs to be non-blocking to actually benefit.
</details>

<details>
<summary>106. What is Spring's `@Transactional(readOnly = true)` used for?</summary>

Hints to the underlying persistence provider/driver that a transaction won't perform writes, enabling potential optimizations (e.g., Hibernate can skip dirty-checking overhead, and some databases/drivers can route read-only transactions to a read replica). It doesn't strictly enforce immutability at the JPA level by itself, but signals intent and enables performance optimizations.
</details>

<details>
<summary>107. What is the difference between `@Transactional` at the class level vs the method level?</summary>

Class-level `@Transactional` applies as the default to all public methods in the class; a method-level `@Transactional` on a specific method overrides the class-level settings just for that method. Fine-grained control (e.g., different propagation/isolation per method) requires method-level annotations.
</details>

<details>
<summary>108. What is transaction isolation, and what are the standard isolation levels?</summary>

Isolation controls how visible one transaction's uncommitted/committed changes are to concurrently running transactions. Standard levels (increasing strictness): **READ UNCOMMITTED** (dirty reads possible), **READ COMMITTED** (default in most DBs — no dirty reads, but non-repeatable reads possible), **REPEATABLE READ** (no non-repeatable reads, but phantom reads possible), **SERIALIZABLE** (fully isolated, as if transactions ran one at a time — safest but slowest).
</details>

<details>
<summary>109. What is a "dirty read", "non-repeatable read", and "phantom read"?</summary>

**Dirty read** — reading another transaction's uncommitted changes, which might later be rolled back. **Non-repeatable read** — re-reading the same row within a transaction and getting a different value because another transaction committed a change in between. **Phantom read** — re-running the same query within a transaction and getting a different *set* of rows because another transaction inserted/deleted matching rows in between.
</details>

<details>
<summary>110. What is the difference between `@Modifying` and a normal `@Query` in Spring Data JPA?</summary>

`@Query` alone is assumed to be a `SELECT`. `@Modifying` must be added alongside `@Query` for `UPDATE`/`DELETE` (bulk) JPQL queries, signaling that the query changes data rather than reading it — Spring Data enforces this to avoid accidentally running a destructive query without acknowledgment.
</details>

<details>
<summary>111. What is projection in Spring Data JPA?</summary>

Fetching only a subset of an entity's fields rather than the whole entity, improving query performance for read-heavy views. Implemented via interface-based projections (Spring proxies an interface with matching getter names) or class-based (DTO) projections using a constructor expression in `@Query`.
</details>

<details>
<summary>112. What is the difference between `@Embeddable`/`@Embedded` and a `@OneToOne` relationship in JPA?</summary>

`@Embeddable`/`@Embedded` models a value object whose fields are stored **inline** in the owning entity's table (no separate table, no identity of its own — e.g., an `Address` embedded in a `User`). `@OneToOne` models a genuine separate entity with its own identity/table, related via a foreign key.
</details>

<details>
<summary>113. What is `CascadeType` in JPA relationships, and what does `CascadeType.ALL` do?</summary>

Determines whether operations performed on a parent entity (persist, merge, remove, refresh, detach) automatically propagate to its related child entities. `CascadeType.ALL` propagates every operation — convenient for true parent-owns-child (composition) relationships (e.g., an `Order` and its `OrderItems`), but dangerous if applied to relationships where the child has independent lifecycle/is shared.
</details>

<details>
<summary>114. What is `orphanRemoval = true` in a JPA `@OneToMany` mapping?</summary>

Automatically deletes a child entity from the database when it's removed from the parent's collection (and the association is no longer referenced), even if `remove()` wasn't explicitly called on the child — useful for strict parent-owns-child relationships where an orphaned child has no reason to exist.
</details>

<details>
<summary>115. What is the difference between `merge()` and `persist()` in JPA?</summary>

`persist()` makes a **new, transient** entity managed, scheduling an `INSERT`. `merge()` takes a **detached** entity (possibly already existing in the DB) and copies its state onto a managed instance (fetching it first if needed), scheduling an `UPDATE` — used to reattach an entity that was modified outside the current persistence context (e.g., received from an HTTP request).
</details>

<details>
<summary>116. What is bean validation group validation (`@Validated` with groups)?</summary>

Allows applying different sets of validation constraints depending on context — e.g., a `Create` group might require an ID to be null, while an `Update` group requires it to be present. Constraints are tagged with `groups = {Create.class}`, and `@Validated(Create.class)` triggers validation for just that group.
</details>

<details>
<summary>117. What is the purpose of `@JsonIgnore`, `@JsonProperty`, and `@JsonInclude` (Jackson annotations)?</summary>

`@JsonIgnore` excludes a field from JSON serialization/deserialization entirely (e.g., a password field). `@JsonProperty` renames a field in the JSON representation, or marks a field for read-only/write-only access. `@JsonInclude(JsonInclude.Include.NON_NULL)` omits null fields from the output JSON, keeping payloads cleaner.
</details>

<details>
<summary>118. What is circular reference handling in Jackson serialization, and how do you fix it?</summary>

Bidirectional entity relationships (e.g., `Order` referencing `Customer`, and `Customer` holding a list of `Order`s) cause infinite recursion during JSON serialization (`StackOverflowError`). Fix with `@JsonManagedReference`/`@JsonBackReference` (breaks the cycle by ignoring one direction), `@JsonIdentityInfo` (replaces repeated references with an ID), or — the cleaner fix in practice — use separate DTOs that don't have the cyclic structure at all.
</details>

<details>
<summary>119. What is the difference between `@RequestMapping(produces=...)` and `@RequestMapping(consumes=...)`?</summary>

`consumes` restricts which request `Content-Type`s the endpoint accepts (e.g., only handle `application/json` bodies). `produces` restricts which response `Content-Type`s the endpoint can return, used for content negotiation with the client's `Accept` header.
</details>

<details>
<summary>120. What is the Spring Boot "fat jar," and how does it differ from a traditional WAR deployment?</summary>

A fat (executable) jar bundles the application code, all its dependencies, and an embedded servlet container (Tomcat/Jetty/Netty) into a single, self-contained, runnable artifact (`java -jar app.jar`) — no external application server needed. Traditional WAR deployment requires a pre-installed, separately managed application server (e.g., a standalone Tomcat instance) that the WAR is deployed into.
</details>

<details>
<summary>121. What is Spring Boot auto-configuration, and how does it decide what to configure?</summary>

Auto-configuration automatically registers sensible-default beans based on what's present on the classpath and existing bean definitions, using `@Conditional`-family annotations (`@ConditionalOnClass`, `@ConditionalOnMissingBean`) inside `spring-boot-autoconfigure`'s configuration classes. For example, having an H2 driver and `spring-boot-starter-data-jpa` on the classpath auto-configures an embedded `DataSource` without you writing any config, as long as you haven't defined your own conflicting `DataSource` bean.
</details>

<details>
<summary>122. How would you exclude a specific auto-configuration class in Spring Boot?</summary>

`@SpringBootApplication(exclude = {DataSourceAutoConfiguration.class})`, or via the `spring.autoconfigure.exclude` property — useful when you want to fully customize a piece of infrastructure Boot would otherwise configure for you.
</details>

<details>
<summary>123. What is the order of property source precedence in Spring Boot?</summary>

From highest to lowest priority (roughly): command-line arguments → `SPRING_APPLICATION_JSON` env var → `application-{profile}.properties`/`.yml` → `application.properties`/`.yml` → `@PropertySource` on `@Configuration` classes → default properties. A value from a higher-priority source overrides the same key defined in a lower one.
</details>

<details>
<summary>124. What is the difference between `.properties` and `.yml` configuration files in Spring Boot?</summary>

Functionally equivalent — both are property sources parsed into the same key-value structure internally. `.yml` supports nested hierarchical structure more naturally and is generally more readable for deeply nested config, but doesn't support multiple identical top-level documents concatenated as cleanly as `.properties` with profile-specific files, and is more whitespace-sensitive (indentation errors are a common pitfall).
</details>

<details>
<summary>125. What is `spring-boot-devtools`, and what does it provide?</summary>

A development-time convenience dependency providing automatic application restart on classpath changes, LiveReload browser integration, and sensible dev-only default property overrides (e.g., disabling template caching) — automatically excluded from production packaging (fat jar) so it never ships to prod.
</details>

<details>
<summary>126. What is `CommandLineRunner` and `ApplicationRunner` in Spring Boot?</summary>

Both let you run code once, right after the Spring application context has fully started and before the application starts serving requests — useful for startup tasks (seeding data, warming caches). `CommandLineRunner.run(String... args)` gets raw command-line args; `ApplicationRunner.run(ApplicationArguments args)` gets a parsed representation (named options vs positional args).
</details>

<details>
<summary>127. What is the Spring Boot health check contract, and how do readiness and liveness probes differ (in a Kubernetes context)?</summary>

**Liveness** — "is the application running/not deadlocked?" A failing liveness probe causes Kubernetes to restart the pod. **Readiness** — "is the application ready to accept traffic right now?" A failing readiness probe removes the pod from load-balancer rotation without restarting it (e.g., while it's still warming up caches or waiting on a downstream dependency). Spring Boot Actuator exposes both as separate health groups/endpoints (`/actuator/health/liveness`, `/actuator/health/readiness`) when `management.endpoint.health.probes.enabled=true`.
</details>

<details>
<summary>128. What is the difference between `@Import` and `@ComponentScan` in Spring configuration?</summary>

`@ComponentScan` discovers and registers beans automatically based on classpath scanning and stereotype annotations within specified packages. `@Import` explicitly registers one or more specific `@Configuration` classes (or `@Component` classes), giving fine-grained, explicit control rather than relying on package-based discovery — often used to compose modular configuration classes together deliberately.
</details>

<details>
<summary>129. What is a Spring `Converter`/`Formatter`, and where is it used?</summary>

`Converter<S, T>` and `Formatter<T>` let you register custom logic for converting between types — e.g., binding a request parameter `String` directly into a custom enum or value object. Registered via a `WebMvcConfigurer`'s `addFormatters()`, and automatically applied during data binding for `@RequestParam`/`@PathVariable`/form binding.
</details>

<details>
<summary>130. What is Spring's `@Retryable` (Spring Retry), and how does it differ from a circuit breaker?</summary>

`@Retryable` automatically retries a failed method call a configured number of times (with optional backoff) before giving up — good for handling transient failures (a brief network blip). A **circuit breaker** instead stops calling a consistently-failing service altogether for a period, to protect both the caller and the struggling downstream service — the two are often used together (retry a few times, but a circuit breaker prevents endless retries against a truly down service).
</details>

<details>
<summary>131. What is the difference between a `RuntimeException` handled by `@Transactional` rollback and a checked exception?</summary>

By default, `@Transactional` rolls back on unchecked (`RuntimeException`/`Error`) exceptions but **commits** on checked exceptions, since Spring assumes checked exceptions represent expected, recoverable business outcomes rather than failures requiring a rollback. Override this default with `@Transactional(rollbackFor = SomeCheckedException.class)` if you need a checked exception to also trigger a rollback.
</details>

<details>
<summary>132. What is the purpose of `spring.jpa.hibernate.ddl-auto`, and why is `update`/`create` dangerous in production?</summary>

Controls whether Hibernate automatically manages schema changes at startup (`none`, `validate`, `update`, `create`, `create-drop`). `update`/`create` are convenient in local dev but dangerous in production — they can silently and unpredictably alter (or destroy) production schema/data outside of a controlled, reviewed migration process. Production should use `validate` (or `none`) alongside a proper migration tool like Flyway/Liquibase.
</details>

<details>
<summary>133. What is the difference between `@RequestScope`, `@SessionScope`, and `@ApplicationScope` (Spring's web-aware scope annotations)?</summary>

Convenience annotations equivalent to `@Scope("request")`, `@Scope("session")`, `@Scope("application")` — a `@RequestScope` bean is a fresh instance per HTTP request, `@SessionScope` per user session, `@ApplicationScope` a single instance for the whole `ServletContext` lifetime (essentially another singleton, but web-context-bound).
</details>

<details>
<summary>134. What is the difference between Spring's `@Primary` and `@Qualifier` for resolving ambiguous bean injection, and when would you prefer one over the other?</summary>

`@Primary` marks one implementation as the default choice application-wide — convenient when one implementation is clearly the "usual" one and only occasionally overridden. `@Qualifier` requires explicit, per-injection-point disambiguation — better when there's no natural default and you always want the choice to be visible and deliberate at each usage site.
</details>

<details>
<summary>135. If asked to design a resilient, scalable Spring Boot microservice from scratch, what key building blocks would you mention?</summary>

Stateless design (JWT-based auth, no in-memory session state) for horizontal scalability; externalized configuration (Spring Cloud Config or environment variables); service discovery if in a dynamic environment (Eureka/Kubernetes DNS); circuit breakers and retries for downstream calls (Resilience4j); centralized structured logging and distributed tracing (Micrometer Tracing + Zipkin/Jaeger); health/readiness/liveness endpoints (Actuator) wired into orchestration (Kubernetes probes); database migrations via Flyway/Liquibase; and API-level concerns like rate limiting, versioning, and a consistent global error-handling contract.
</details>

<details>
<summary>136. What is the difference between `@Bean` initialization order and `@DependsOn`?</summary>

Spring normally determines bean creation order automatically based on declared dependencies (constructor/field injection). `@DependsOn("beanName")` explicitly forces one bean to be created only after another, for cases where a dependency isn't expressed through direct injection (e.g., a bean that relies on a static initializer or side effect triggered by another bean's creation).
</details>

<details>
<summary>137. What is the difference between `@Primary` and bean name matching when using `@Autowired` on a field named after a bean?</summary>

If multiple candidates exist and none is marked `@Primary`, Spring falls back to matching the **field/parameter name** against bean names as a tiebreaker before giving up with `NoUniqueBeanDefinitionException`. `@Primary` is a stronger, more explicit and refactor-safe signal than relying on this name-matching fallback.
</details>

<details>
<summary>138. What is a Spring `ApplicationContextInitializer`?</summary>

A callback interface allowing you to programmatically customize the `ApplicationContext` **before** bean definitions are loaded/refreshed — useful for setting up property sources or profiles dynamically at startup, registered via `SpringApplication.addInitializers()` or a `spring.factories`/`spring.context.initializer.classes` entry.
</details>

<details>
<summary>139. What is the purpose of `spring.factories` (or the newer `META-INF/spring/*.imports`)?</summary>

A file mechanism that lets libraries register auto-configuration classes, initializers, and listeners to be picked up automatically by Spring Boot's auto-configuration machinery, without the consuming application needing to explicitly declare them — the backbone of how Boot "starters" work.
</details>

<details>
<summary>140. What is the difference between `@Component` scanning conflicts and explicit bean naming?</summary>

If two classes in scanned packages happen to produce the same default bean name (usually derived from the class's simple name), Spring throws a conflict at startup. Resolve by giving one an explicit name: `@Component("customUserService")`.
</details>

<details>
<summary>141. What is Spring's `ObjectProvider<T>`, and when would you use it over plain `@Autowired`?</summary>

`ObjectProvider<T>` is a more flexible injection point that defers bean lookup — lets you handle the case where zero, one, or multiple beans of a type might exist gracefully (`getIfAvailable()`, `getIfUnique()`, `orderedStream()`), instead of `@Autowired` failing hard at context startup when a bean is optionally absent.
</details>

<details>
<summary>142. What is the difference between `@Configuration` classes and `@Component` classes that happen to declare `@Bean` methods?</summary>

Only true `@Configuration` classes get CGLIB-proxied to guarantee singleton semantics for inter-bean method calls (see Q12 above). A plain `@Component` with `@Bean` methods (a "lite" mode) is treated as a simple factory — calling one `@Bean` method from another within it does **not** return the same singleton; it creates a fresh instance each time, which is a common, subtle bug.
</details>

<details>
<summary>143. What is the difference between `RestTemplateBuilder` and directly `new RestTemplate()`?</summary>

`RestTemplateBuilder` is a Spring Boot–provided builder that applies auto-configured, sensible defaults (message converters, error handlers, and any customizers registered in the context) — the recommended way to construct a `RestTemplate` bean in a Boot app, rather than instantiating it manually and losing those integrations.
</details>

<details>
<summary>144. What is Spring's `RetryTemplate` (older Spring Retry API) versus the `@Retryable` annotation?</summary>

`RetryTemplate` is the imperative, programmatic API — you wrap the retryable logic in a callback passed to `execute()`. `@Retryable` is the declarative, annotation-driven equivalent (AOP-proxy-based, like `@Transactional`), generally preferred for its conciseness unless you need highly dynamic, runtime-configured retry policies that don't fit neatly into an annotation.
</details>

<details>
<summary>145. What is the difference between `@Scheduled(fixedRate = ...)` and `@Scheduled(fixedDelay = ...)`?</summary>

`fixedRate` triggers the next execution at a fixed interval from the **start** of the previous execution, regardless of how long it took (executions can overlap or queue up if the task runs longer than the interval). `fixedDelay` waits a fixed interval after the **completion** of the previous execution before starting the next — safer when task duration is variable, since it guarantees no overlap.
</details>

<details>
<summary>146. What is the difference between `@Lookup` method injection and standard constructor injection?</summary>

`@Lookup` lets a singleton bean obtain a **fresh instance of a prototype-scoped bean** on each call, working around the natural limitation that a singleton's dependencies are normally injected only once at creation time — Spring overrides the annotated (often abstract) method at runtime to fetch a new prototype instance from the container each time it's invoked.
</details>

<details>
<summary>147. What is the purpose of Spring's `@Profile("!prod")` (negated profile expressions)?</summary>

Profile expressions support logical operators (`!`, `&`, `|`) — `@Profile("!prod")` registers a bean in every profile **except** `prod`, useful for dev/test-only conveniences (like an in-memory mock service) that should be automatically excluded in production without needing to be explicitly listed for every non-prod profile.
</details>

<details>
<summary>148. What is a common cause of "No qualifying bean of type X found" errors, and how do you debug it?</summary>

Usually caused by: the class not being annotated with a stereotype (`@Component`/`@Service`) or not covered by `@ComponentScan`'s base package, a missing `@Import`/auto-configuration for a library bean, or a typo/mismatch in `@Qualifier`. Debug by enabling `--debug` to print the auto-configuration report, or checking `ApplicationContext.getBeanDefinitionNames()` to confirm what's actually registered.
</details>

<details>
<summary>149. What is the difference between `spring-boot-starter-parent` and importing the `spring-boot-dependencies` BOM directly?</summary>

`spring-boot-starter-parent` is a full **Maven parent POM** — it manages dependency versions **and** sets sensible plugin defaults (compiler settings, packaging config for the fat jar). Importing `spring-boot-dependencies` as a **BOM** (`<dependencyManagement><import>`) only gives you the version management, useful when your project already has its own parent POM and can't also inherit from `spring-boot-starter-parent`.
</details>

<details>
<summary>150. If asked "how would you migrate a legacy monolithic Spring MVC app to microservices," what would you outline?</summary>

Start by identifying bounded contexts/domain boundaries (Domain-Driven Design) rather than splitting arbitrarily; extract one well-isolated module at a time (strangler fig pattern) rather than a risky big-bang rewrite; introduce an API gateway to route traffic during the transition; establish a shared contract/versioning strategy between old and new; replace direct in-process calls between the extracted piece and the monolith with REST/messaging; introduce distributed tracing early since debugging across services gets much harder; and only fully decommission monolith code paths once the extracted service is proven stable in production under real traffic.
</details>
