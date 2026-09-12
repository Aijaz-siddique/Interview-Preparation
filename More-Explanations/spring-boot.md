# Spring Boot — Interview Questions (Experienced)

<details>
<summary>1. What is Spring Boot, and what problem does it solve over plain Spring?</summary>

Spring Boot is an opinionated framework built on top of Spring that eliminates most manual configuration through auto-configuration, starter dependencies, and embedded servers. Plain Spring required extensive XML/Java configuration for even basic setups (DataSource, DispatcherServlet, view resolvers); Boot lets you get a production-ready app running with minimal setup, following convention-over-configuration.
</details>

<details>
<summary>2. What is a Spring Boot "starter"? Name a few common ones.</summary>

A starter is a curated dependency descriptor that pulls in a coherent set of libraries for a specific purpose, with compatible versions already resolved — you don't manage individual library versions yourself. Examples: `spring-boot-starter-web` (MVC + embedded Tomcat), `spring-boot-starter-data-jpa` (Hibernate + Spring Data), `spring-boot-starter-security`, `spring-boot-starter-test` (JUnit, Mockito, AssertJ).
</details>

<details>
<summary>3. What is `@SpringBootApplication` actually composed of?</summary>

It's a meta-annotation combining three: `@Configuration` (marks it as a source of bean definitions), `@EnableAutoConfiguration` (triggers Boot's auto-configuration mechanism), and `@ComponentScan` (scans the current package and sub-packages for components) — one annotation replacing what would otherwise be three.
</details>

<details>
<summary>4. What is `SpringApplication.run()` doing under the hood?</summary>

It creates and refreshes an `ApplicationContext`, registers a `CommandLineRunner`/`ApplicationRunner` execution phase, sets up property sources and profiles, configures logging, starts the embedded web server (if it's a web application), and publishes application lifecycle events (`ApplicationStartingEvent`, `ApplicationReadyEvent`, etc.) that listeners can hook into.
</details>

<details>
<summary>5. How does Spring Boot auto-configuration actually decide what to configure?</summary>

Auto-configuration classes (registered via `META-INF/spring/org.springframework.boot.autoconfigure.AutoConfiguration.imports` in Boot 2.7+/3.x, or the older `spring.factories`) are conditionally applied using annotations like `@ConditionalOnClass` (is a library on the classpath?), `@ConditionalOnMissingBean` (has the user already defined their own?), `@ConditionalOnProperty` (is a specific property set?). This lets Boot configure sensible defaults while backing off gracefully if you've customized something yourself.
</details>

<details>
<summary>6. How would you see exactly which auto-configurations were applied vs excluded in your app?</summary>

Run with `--debug` (or set `debug=true` in properties) — Boot prints an auto-configuration report at startup listing "Positive matches" (applied) and "Negative matches" (skipped, with the reason), extremely useful for debugging "why isn't my bean being created" issues.
</details>

<details>
<summary>7. What is the difference between `@ConditionalOnBean` and `@ConditionalOnMissingBean`?</summary>

`@ConditionalOnBean` — a configuration only applies if a specified bean **already exists** in the context. `@ConditionalOnMissingBean` — applies only if that bean does **not** exist yet — the standard way auto-configuration provides a default that backs off if the user supplies their own bean of the same type.
</details>

<details>
<summary>8. What embedded servers does Spring Boot support, and how do you switch between them?</summary>

Tomcat (default with `spring-boot-starter-web`), Jetty, and Undertow. Switch by excluding the default Tomcat dependency and adding the alternative starter, e.g., exclude `spring-boot-starter-tomcat` from `spring-boot-starter-web` and add `spring-boot-starter-jetty`.
</details>

<details>
<summary>9. What is the difference between an executable ("fat") jar and a traditional WAR in Spring Boot?</summary>

A fat jar bundles the app, all dependencies, and an embedded server into one self-contained artifact runnable via `java -jar` — no external app server needed. Boot can still produce a deployable WAR (by extending `SpringBootServletInitializer`) for legacy environments requiring deployment into an existing external servlet container, but the jar-with-embedded-server model is the default and recommended approach.
</details>

<details>
<summary>10. What is the layered jar feature in Spring Boot, and why does it matter for Docker builds?</summary>

Boot can package the fat jar into layers (dependencies, resources, application classes) with the least-frequently-changing layers (third-party dependencies) first. This lets Docker cache those layers across builds, so a code change only invalidates and rebuilds the small "application" layer — dramatically speeding up image builds and reducing pushed image size deltas.
</details>

<details>
<summary>11. What is Spring Boot's default logging setup, and how do you switch to Log4j2?</summary>

Boot uses Logback by default via the `spring-boot-starter-logging` dependency (transitively pulled in by most starters). To switch to Log4j2, exclude `spring-boot-starter-logging` and add `spring-boot-starter-log4j2` — Boot auto-detects and configures whichever logging implementation is on the classpath.
</details>

<details>
<summary>12. How do you configure different log levels per package in Spring Boot?</summary>

Via properties: `logging.level.com.myapp.service=DEBUG`, `logging.level.org.springframework.web=INFO` — or a full `logback-spring.xml`/`log4j2-spring.xml` for more advanced configuration (appenders, rolling policies, MDC patterns).
</details>

<details>
<summary>13. What is `application.yml` profile-specific configuration, and how does the multi-document YAML syntax work?</summary>

You can define profile-specific overrides within a single file using `---` separators and `spring.config.activate.on-profile: prod`, instead of maintaining fully separate `application-prod.yml` files — useful for keeping related configuration visually grouped together.
</details>

<details>
<summary>14. What is the difference between `spring.config.import` and `@PropertySource`?</summary>

`spring.config.import` (Boot 2.4+) is the modern way to import additional configuration files/sources (including from Config Server, Vault, or additional local files) as part of Boot's own config-loading lifecycle, respecting profile activation. `@PropertySource` is the older Spring-core mechanism, loaded earlier and with less integration into Boot's layered property resolution.
</details>

<details>
<summary>15. What is relaxed binding in Spring Boot configuration properties?</summary>

Boot tolerantly matches property names across different naming conventions — `my-app.some-property`, `my_app.some_property`, `myApp.someProperty`, and `MYAPP_SOMEPROPERTY` (env var style) can all bind to the same `@ConfigurationProperties` field, making configuration portable across `.properties`, `.yml`, and environment variables (important for containerized/cloud deployments).
</details>

<details>
<summary>16. What is `@ConfigurationPropertiesScan`?</summary>

Enables component-scanning specifically for `@ConfigurationProperties` classes, so you don't need to separately register each one with `@EnableConfigurationProperties` or as a `@Bean` — Boot discovers and binds them automatically if they're annotated and within scan scope.
</details>

<details>
<summary>17. How do you validate `@ConfigurationProperties` at startup?</summary>

Annotate the properties class with `@Validated` and use standard Bean Validation annotations (`@NotNull`, `@Min`, `@Pattern`) on its fields — if binding produces an invalid object, Boot fails fast at startup with a clear `ConfigurationPropertiesBindException` rather than allowing the app to start with bad config and fail confusingly later.
</details>

<details>
<summary>18. What is the Spring Boot Actuator, and what are its most commonly used endpoints?</summary>

A production-readiness module exposing operational insight into a running application via HTTP/JMX endpoints: `/actuator/health` (up/down status, with configurable indicators), `/actuator/metrics` (JVM, HTTP, custom metrics), `/actuator/info` (build/git metadata), `/actuator/env` (active configuration), `/actuator/loggers` (view/change log levels at runtime), `/actuator/threaddump` and `/actuator/heapdump` for diagnostics.
</details>

<details>
<summary>19. How do you secure Actuator endpoints in production?</summary>

By default, only `/health` and `/info` are exposed over HTTP unless you explicitly widen `management.endpoints.web.exposure.include`. Beyond that, apply Spring Security rules specifically to `/actuator/**` (often requiring an admin role), run actuator on a **separate management port** (`management.server.port`) that isn't publicly reachable, and avoid exposing sensitive endpoints (`env`, `heapdump`, `shutdown`) externally at all.
</details>

<details>
<summary>20. What is a custom `HealthIndicator`, and when would you write one?</summary>

A bean implementing `HealthIndicator` that reports the health of a specific dependency (a downstream API, a message queue connection, disk space threshold) — contributes to the overall `/actuator/health` aggregate status. Write one whenever the default DB/disk-space indicators don't cover a critical dependency your app actually needs to function correctly.
</details>

<details>
<summary>21. What is the difference between Actuator's "health groups" for liveness vs readiness?</summary>

Liveness answers "is the app in a state where restarting it would help?" (e.g., not deadlocked). Readiness answers "can it currently serve traffic?" (e.g., DB connection pool initialized, caches warmed). Kubernetes uses these differently: a failing liveness probe restarts the pod; a failing readiness probe just removes it from the service's load-balanced endpoints without restarting.
</details>

<details>
<summary>22. What is Micrometer, and how does it relate to Boot's metrics?</summary>

Micrometer is a vendor-neutral instrumentation facade (like SLF4J, but for metrics) that Actuator uses internally — your code (or Boot's own auto-instrumentation of HTTP requests, JVM stats, DataSource pools) records metrics through Micrometer's API, and you plug in a registry (Prometheus, Datadog, CloudWatch) to actually export them, without changing instrumentation code when you switch monitoring backends.
</details>

<details>
<summary>23. How would you add a custom business metric (e.g., "orders processed") in a Spring Boot app?</summary>

Inject a `MeterRegistry` and register a counter/gauge/timer:

```java
Counter ordersProcessed = Counter.builder("orders.processed")
    .description("Total orders processed")
    .register(meterRegistry);
ordersProcessed.increment();
```
Automatically becomes scrapeable via `/actuator/prometheus` if the Prometheus registry dependency is present.
</details>

<details>
<summary>24. What is distributed tracing, and how does Spring Boot 3 integrate it (Micrometer Tracing)?</summary>

Distributed tracing tracks a single logical request as it flows across multiple services, correlating spans via a shared trace ID — essential for debugging latency/errors in a microservices architecture. Spring Boot 3 replaced the older Spring Cloud Sleuth with **Micrometer Tracing** (a similar facade pattern to metrics), which can export to Zipkin, OpenTelemetry, or other backends.
</details>

<details>
<summary>25. What is the difference between Spring Boot 2 and Spring Boot 3 at a high level?</summary>

Boot 3 requires Java 17+ as a baseline, migrated from `javax.*` to `jakarta.*` namespace (following Jakarta EE's own rename), replaced Spring Cloud Sleuth with Micrometer Tracing, added native/GraalVM image support as a first-class feature, and dropped support for older Spring Framework 5.x in favor of Spring Framework 6.
</details>

<details>
<summary>26. What is GraalVM native image support in Spring Boot 3, and what are its trade-offs?</summary>

Compiles a Spring Boot application ahead-of-time into a standalone native executable (no JVM needed at runtime), giving near-instant startup and lower memory footprint — valuable for serverless/scale-to-zero workloads. Trade-offs: longer, more resource-intensive build times, some reflection-heavy libraries need explicit hints/configuration to work correctly, and debugging/observability tooling is less mature than on the standard JVM.
</details>

<details>
<summary>27. What is the difference between `spring-boot-starter-tomcat` and running Tomcat as a standalone server?</summary>

`spring-boot-starter-tomcat` embeds a Tomcat instance **inside** your application's process — it starts and stops with your app, and its configuration lives in your app's properties, rather than being a separately managed, shared infrastructure component that multiple WARs might be deployed into.
</details>

<details>
<summary>28. How do you configure the embedded Tomcat's thread pool / connection settings in Boot?</summary>

Via properties like `server.tomcat.threads.max`, `server.tomcat.threads.min-spare`, `server.tomcat.accept-count`, `server.tomcat.connection-timeout` — tuning these matters for handling high concurrent load without exhausting resources or queueing requests excessively.
</details>

<details>
<summary>29. What is the difference between `server.port=0` and a fixed port in Spring Boot, and when is a random port useful?</summary>

`server.port=0` tells the embedded server to bind to any available free port at startup — commonly used in integration tests (`@SpringBootTest(webEnvironment = RANDOM_PORT)`) to avoid port conflicts when running multiple test suites in parallel or on shared CI machines.
</details>

<details>
<summary>30. What is `@SpringBootTest`'s `webEnvironment` attribute, and what are its options?</summary>

Controls how (or whether) a web environment is started for the test: `MOCK` (default — a mock servlet environment, no real server), `RANDOM_PORT` (a real embedded server on a random free port, good for full HTTP integration tests), `DEFINED_PORT` (real server on the configured port), `NONE` (no web environment at all, for non-web context tests).
</details>

<details>
<summary>31. What is `@AutoConfigureMockMvc`, and how is `MockMvc` used?</summary>

Configures a `MockMvc` instance for testing the web layer without starting a real HTTP server — requests are dispatched directly through Spring's `DispatcherServlet` in-memory, making tests fast while still exercising real request-mapping, validation, and serialization logic.

```java
mockMvc.perform(get("/users/1"))
    .andExpect(status().isOk())
    .andExpect(jsonPath("$.name").value("Alice"));
```
</details>

<details>
<summary>32. What is `@DataJpaTest`, and what does it configure/exclude?</summary>

Configures only JPA-related components (repositories, `EntityManager`, an in-memory embedded database by default) while excluding unrelated beans (controllers, services) — each test runs transactionally and rolls back at the end by default, keeping tests fast and isolated from each other.
</details>

<details>
<summary>33. How do you make `@DataJpaTest` use a real database (e.g., via Testcontainers) instead of the default in-memory H2?</summary>

Add `@AutoConfigureTestDatabase(replace = AutoConfigureTestDatabase.Replace.NONE)` to prevent Boot from swapping in an embedded DB, then configure a Testcontainers-backed `DataSource` (often via `@ServiceConnection` in newer Boot versions, or manually via `@DynamicPropertySource`) — important because H2's SQL dialect quirks can hide real Postgres/MySQL-specific bugs.
</details>

<details>
<summary>34. What is `@TestConfiguration`, and how does it differ from `@Configuration`?</summary>

`@TestConfiguration` marks a configuration class intended only for use in tests — it's excluded from component scanning of the main application by default (so it won't accidentally get picked up in production), and is typically used to supply test-specific bean overrides or additional test-only beans.
</details>

<details>
<summary>35. What is `@Sql` in Spring Boot testing?</summary>

Lets you run SQL scripts before/after a test method to set up or tear down test data declaratively, instead of doing it programmatically in test code: `@Sql("/test-data.sql")`.
</details>

<details>
<summary>36. What is the purpose of `spring.test.database.replace` and profiles specifically for testing?</summary>

Controls whether Boot substitutes an embedded database automatically for `@DataJpaTest`-style tests. A separate `test` (or `it`) Spring profile with its own `application-test.yml` is a common pattern to isolate test-specific configuration (test DB URLs, mock external service endpoints) from dev/prod configuration entirely.
</details>

<details>
<summary>37. How would you write a test that verifies Boot's auto-configuration behaves correctly with a specific property set (or absent)?</summary>

Use `ApplicationContextRunner` — a lightweight way to spin up an isolated context with specific properties/configurations applied, and assert on the resulting bean registrations, without bootstrapping a full application:

```java
new ApplicationContextRunner()
    .withPropertyValues("feature.enabled=true")
    .withUserConfiguration(MyAutoConfiguration.class)
    .run(context -> assertThat(context).hasSingleBean(MyService.class));
```
</details>

<details>
<summary>38. What is the difference between integration tests and "slice" tests in terms of what they actually verify?</summary>

Slice tests (`@WebMvcTest`, `@DataJpaTest`, `@JsonTest`) verify a specific architectural layer in isolation with a partial context, catching layer-specific bugs quickly. Full integration tests (`@SpringBootTest`) verify the layers actually wire together correctly end-to-end, catching configuration/wiring bugs slice tests can't — both have their place; slice tests for speed and focus, full integration tests for confidence before release.
</details>

<details>
<summary>39. What is Boot's `spring-boot-starter-validation`, and what does it add beyond core Spring?</summary>

Brings in Hibernate Validator (the reference implementation of Jakarta Bean Validation), enabling `@Valid`/`@Validated` annotation-driven validation on request bodies, method parameters, and configuration properties — this isn't included by default in `spring-boot-starter-web` since Boot 2.3+, so it must be added explicitly if you need bean validation.
</details>

<details>
<summary>40. What is the recommended way to handle validation errors globally in a Spring Boot REST API?</summary>

A `@RestControllerAdvice` with an `@ExceptionHandler(MethodArgumentNotValidException.class)` that extracts field errors from the exception's `BindingResult` and formats them into a consistent structured error response (field name, rejected value, message) rather than letting the default Spring error page/response leak through.
</details>

<details>
<summary>41. What is Spring Boot's default error handling behavior (`/error` endpoint / `BasicErrorController`)?</summary>

Boot auto-configures a `BasicErrorController` mapped to `/error` that produces a generic JSON error response (timestamp, status, error, message, path) for any unhandled exception, as a fallback when you haven't defined your own global exception handling — customizable via `ErrorAttributes` or fully replaceable with your own controller.
</details>

<details>
<summary>42. How would you customize the default whitelabel error page in Spring Boot?</summary>

Either disable it (`server.error.whitelabel.enabled=false`) and provide your own `/error` view/controller, implement a custom `ErrorController`, or override `ErrorAttributes` to customize exactly what fields appear in the default JSON error response.
</details>

<details>
<summary>43. What is the purpose of `@RestControllerAdvice(basePackages = ...)`?</summary>

Scopes a global exception handler to only apply to controllers within specific packages, rather than every controller in the application — useful in larger, modular applications where different modules want different error-handling behavior.
</details>

<details>
<summary>44. What is Boot's `spring.mvc.problemdetails.enabled` property (Boot 3+), and what is RFC 7807?</summary>

Enables Spring's built-in support for **Problem Details** (RFC 7807), a standardized JSON error response format (`type`, `title`, `status`, `detail`, `instance`) — instead of every team inventing its own bespoke error JSON shape, giving API consumers a predictable, machine-parseable error contract out of the box.
</details>

<details>
<summary>45. What is the purpose of the `spring-boot-maven-plugin` (or Gradle equivalent), and what does `repackage` do?</summary>

Packages the application into an executable fat jar/war, and `repackage` specifically takes the plain jar produced by the standard `mvn package` and restructures it, embedding all dependencies and adding a bootstrap launcher class so `java -jar app.jar` works directly.
</details>

<details>
<summary>46. What is the difference between `mvn spring-boot:run` and running the packaged jar directly?</summary>

`spring-boot:run` runs the application directly from compiled classes/source without packaging a jar first — faster for local development iteration. Running the packaged jar (`java -jar app.jar`) executes exactly what would be deployed, useful for final verification before release.
</details>

<details>
<summary>47. What is Spring Boot's build-info feature, and why is it useful for the `/actuator/info` endpoint?</summary>

The `spring-boot-maven-plugin`'s `build-info` goal generates a `build-info.properties` file (version, build time, artifact name) baked into the jar at build time, automatically surfaced by the `/actuator/info` endpoint — useful for confirming exactly which build/version is running in a given environment during an incident.
</details>

<details>
<summary>48. How do you inject the active Git commit hash into `/actuator/info`?</summary>

Add the `git-commit-id-maven-plugin` (or Gradle equivalent), which generates a `git.properties` file at build time containing commit hash, branch, and build timestamp — Actuator's `info` endpoint automatically picks this up if `management.info.git.mode=full` (or default) is set.
</details>

<details>
<summary>49. What is the purpose of environment-specific Docker images and Boot's `spring-boot-starter-actuator` when running in Kubernetes?</summary>

Actuator's health/readiness/liveness endpoints integrate directly with Kubernetes probe configuration, letting the orchestrator make informed decisions about restarting unhealthy pods or removing not-yet-ready pods from service traffic — without Actuator, you'd need custom scripts/logic to approximate the same signal.
</details>

<details>
<summary>50. What is the difference between `spring.lifecycle.timeout-per-shutdown-phase` and simply killing the process?</summary>

The former enables **graceful shutdown**, giving in-flight requests a bounded window to complete (and stopping new request acceptance) before the JVM actually terminates — abruptly killing the process instead drops in-flight connections and can leave partial writes/inconsistent state, especially problematic during rolling deployments.
</details>

<details>
<summary>51. What is `@EnableConfigurationProperties`, and when do you need it explicitly?</summary>

Registers a specific `@ConfigurationProperties` class as a Spring bean when it's not being picked up via component scanning (e.g., when writing a reusable auto-configuration library, where you don't want to rely on the consumer's component scan configuration).
</details>

<details>
<summary>52. What is a custom Spring Boot starter, and what are the two modules typically involved in building one?</summary>

A reusable library packaging auto-configuration for a specific concern, distributable across multiple projects. Typically split into two modules: an `autoconfigure` module (contains the actual `@Configuration` classes and conditional logic) and a `starter` module (a thin POM that just pulls in the autoconfigure module plus any required runtime dependencies) — this separation lets consumers depend on just the starter for a clean dependency graph.
</details>

<details>
<summary>53. What is the purpose of `@AutoConfiguration` (Boot 2.7+) versus the older plain `@Configuration` for auto-config classes?</summary>

`@AutoConfiguration` is a more specific, self-documenting annotation (a specialization of `@Configuration`) introduced to make auto-configuration classes explicitly distinguishable, and it supports `before`/`after` attributes to control ordering relative to other auto-configurations more declaratively than the older `@AutoConfigureBefore`/`@AutoConfigureAfter` combo.
</details>

<details>
<summary>54. What is the significance of the ordering of auto-configuration classes, and how do you control it?</summary>

Order matters because some auto-configurations depend on beans registered by others (e.g., a caching auto-configuration might need a `RedisConnectionFactory` bean to already exist). Control via `@AutoConfigureBefore`/`@AutoConfigureAfter`/`@AutoConfigureOrder`, ensuring dependent configuration classes process in the correct sequence.
</details>

<details>
<summary>55. What is Spring Boot's `DevTools` LiveReload, and what are its limitations?</summary>

Automatically triggers a browser refresh when it detects a classpath change and application restart, speeding up the dev feedback loop for front-end-adjacent work (Thymeleaf templates, static resources). Limitations: doesn't work well with all IDEs' auto-compile settings out of the box, and adds negligible but real value for pure backend/API-only development compared to template-driven apps.
</details>

<details>
<summary>56. What is the difference between a "restart" and a "reload" in Spring Boot DevTools?</summary>

A **restart** (DevTools' actual mechanism) reloads the application context using two classloaders — a rarely-changing "base" classloader for libraries and a "restart" classloader for your own application classes — so only your code needs to be reloaded, making restarts noticeably faster than a full cold start. It's not a true hot-reload (in-memory state is lost), unlike JRebel-style tools.
</details>

<details>
<summary>57. What is Spring Boot's `spring.main.lazy-initialization` property, and what's the trade-off?</summary>

Set to `true`, it makes **all** beans lazy by default (only created when first needed) rather than eagerly at startup — can significantly reduce startup time for large applications. Trade-off: startup-time configuration errors are deferred until a bean is actually requested (surfacing later, possibly in production, rather than immediately at boot), and the first request touching a lazily-initialized bean pays its creation cost.
</details>

<details>
<summary>58. What is a Spring Boot "banner," and can you customize it?</summary>

The ASCII art logo printed to the console/logs at startup. Fully customizable via a `banner.txt` file on the classpath (supporting placeholders like `${spring-boot.version}`), or disabled entirely with `spring.main.banner-mode=off`.
</details>

<details>
<summary>59. What is the significance of `spring-boot-configuration-processor`?</summary>

An annotation processor that generates `META-INF/spring-configuration-metadata.json` at compile time, describing your custom `@ConfigurationProperties` — this is what powers IDE autocomplete and inline documentation for your own custom properties in `application.yml`, just like it works for Boot's built-in properties.
</details>

<details>
<summary>60. What is the difference between `@ConditionalOnWebApplication` and `@ConditionalOnNotWebApplication`?</summary>

Used within auto-configuration to register beans only when the application context is (or isn't) a web application context — e.g., a health check server component shouldn't be registered in a purely batch/CLI application context.
</details>

<details>
<summary>61. How do you run a Spring Boot application as a batch job that exits after completion rather than staying up as a web server?</summary>

Set `spring.main.web-application-type=none` to prevent an embedded server from starting, and call `System.exit(SpringApplication.exit(context, exitCodeGenerator))` in a `CommandLineRunner` after the batch work completes, ensuring a proper exit code is returned to the calling shell/scheduler.
</details>

<details>
<summary>62. What is the purpose of `ExitCodeGenerator` in Spring Boot?</summary>

Lets you control the JVM's process exit code based on application outcome (e.g., return a non-zero code on batch job failure) — important for CI/CD pipelines and job schedulers that key off exit codes to determine success/failure.
</details>

<details>
<summary>63. What is a `SpringApplicationBuilder`, and when would you use it over `SpringApplication.run()` directly?</summary>

A fluent builder API for more advanced startup scenarios — e.g., configuring a parent-child application context hierarchy, or programmatically composing multiple sources of configuration — more flexible than the simple static `run()` method for non-trivial bootstrapping needs.
</details>

<details>
<summary>64. What is the purpose of the `ApplicationReadyEvent` versus `ApplicationStartedEvent`?</summary>

`ApplicationStartedEvent` fires once the context is refreshed and `CommandLineRunner`/`ApplicationRunner`s haven't run yet. `ApplicationReadyEvent` fires after those runners have completed — signaling the application is fully ready to serve its purpose, often the better hook for final startup logging/notifications ("app is up") or triggering downstream registration (e.g., announcing readiness to a service registry).
</details>

<details>
<summary>65. What is the difference between `spring-boot-starter-json` and manually configuring Jackson?</summary>

`spring-boot-starter-json` (transitively included in `spring-boot-starter-web`) auto-configures a sensible default `ObjectMapper` bean with reasonable settings (e.g., ignoring unknown properties can be toggled, Java 8 date/time module registered automatically) — you can further customize it via a `Jackson2ObjectMapperBuilderCustomizer` bean rather than replacing the whole auto-configuration.
</details>

<details>
<summary>66. How would you configure Spring Boot to ignore unknown JSON fields during deserialization?</summary>

`spring.jackson.deserialization.fail-on-unknown-properties=false` (this is actually the default behavior in Boot already) — or annotate the specific DTO with `@JsonIgnoreProperties(ignoreUnknown = true)` for more targeted control.
</details>

<details>
<summary>67. What is the difference between `spring.jackson.date-format` and using `@JsonFormat` on a field?</summary>

`spring.jackson.date-format` sets a **global** default date format applied across the whole `ObjectMapper`. `@JsonFormat(pattern = "yyyy-MM-dd")` on a specific field overrides that default just for that one field — useful when most dates should use ISO format but a specific legacy field needs a different pattern.
</details>

<details>
<summary>68. What is Boot's default handling of `LocalDate`/`LocalDateTime` in JSON, and what module enables it?</summary>

The `jackson-datatype-jsr310` module (auto-included by Boot's starter) enables proper serialization of Java 8 time types as ISO-8601 strings by default, rather than Jackson's default (and awkward) numeric timestamp array representation.
</details>

<details>
<summary>69. What is the purpose of `@JsonComponent` in Spring Boot?</summary>

A convenience annotation for registering a custom Jackson `JsonSerializer`/`JsonDeserializer` as a Spring bean, automatically wired into the auto-configured `ObjectMapper` — simpler than manually building and registering a Jackson `Module`.
</details>

<details>
<summary>70. What is the difference between `spring-boot-starter-data-jpa` and `spring-boot-starter-data-jdbc`?</summary>

`spring-boot-starter-data-jpa` brings in full JPA/Hibernate with its object-relational mapping, lazy loading, dirty checking, and caching machinery — powerful but with a learning curve and potential for subtle performance surprises. `spring-boot-starter-data-jdbc` is a simpler, more explicit, SQL-close abstraction with no lazy loading or implicit dirty checking — better fit when you want predictable, straightforward SQL execution without the ORM complexity.
</details>

<details>
<summary>71. What is Boot's automatic `DataSource` configuration based on classpath detection?</summary>

If Boot finds an embedded database driver (H2, HSQLDB, Derby) on the classpath and no explicit `spring.datasource.url` is configured, it auto-configures an in-memory embedded `DataSource` automatically — convenient for local dev/tests, but must be explicitly configured with real connection details for any persistent environment.
</details>

<details>
<summary>72. What is the purpose of `spring.datasource.hikari.*` properties, and name a few important ones.</summary>

Configure the default HikariCP connection pool: `maximum-pool-size` (max concurrent connections), `minimum-idle` (idle connections kept ready), `connection-timeout` (max wait for a connection before failing), `idle-timeout`, `max-lifetime` (forces connection recycling to avoid stale connections). Undersizing/oversizing the pool relative to actual DB capacity and application concurrency is a very common production tuning issue.
</details>

<details>
<summary>73. What is a common symptom of connection pool exhaustion, and how would you diagnose it?</summary>

Requests hang or time out waiting to acquire a connection (`HikariPool-1 - Connection is not available, request timed out`), often under load spikes or when connections are being held too long (e.g., a slow query, or a connection leak from not closing a resource properly). Diagnose via Actuator's `/actuator/metrics/hikaricp.connections.*` metrics, thread dumps showing threads blocked waiting on the pool, or HikariCP's leak-detection threshold setting.
</details>

<details>
<summary>74. What is the difference between `spring.jpa.open-in-view` being `true` (default) vs `false`, and why is it controversial?</summary>

**Open Session in View** (`true`, Boot's default) keeps the persistence session open for the entire HTTP request, including view rendering — convenient because lazy associations can still be loaded in the view layer without `LazyInitializationException`, but it's controversial because it hides N+1 query problems, ties up a DB connection for the whole request duration (hurting throughput under load), and blurs the separation between the web and persistence layers. Many experienced teams explicitly disable it (`false`) and fetch everything needed within the service layer/transaction boundary.
</details>

<details>
<summary>75. What is Boot's `@Transactional` interaction with `@DataJpaTest`'s automatic rollback?</summary>

`@DataJpaTest` wraps each test method in a transaction that's rolled back automatically at the end, so test data never persists between tests — this is convenient for isolation but means you can't use it to verify actual commit behavior; for that, you'd need `@Commit` (rare) or a full integration test outside the auto-rollback wrapper.
</details>

<details>
<summary>76. What is Spring Boot's support for multiple `DataSource`s, and what extra configuration does it require?</summary>

Boot's auto-configuration assumes a single primary `DataSource` by default. For multiple datasources, you must manually define each `DataSource`/`EntityManagerFactory`/`TransactionManager` bean explicitly, mark one `@Primary`, and typically split entities/repositories into separate packages scanned by separate `@EnableJpaRepositories` configurations pointing at the correct `EntityManagerFactory` for each.
</details>

<details>
<summary>77. What is the purpose of `spring-boot-starter-cache`, and what caching providers does it support?</summary>

Enables Spring's caching abstraction (`@Cacheable`, `@CacheEvict`) with `@EnableCaching`. Supports pluggable backing providers: a simple in-memory `ConcurrentHashMap` (default, dev-only), Caffeine (high-performance local cache), Redis/EhCache (distributed/shared caching) — auto-detected based on what's on the classpath.
</details>

<details>
<summary>78. What is the difference between a local (in-JVM) cache and a distributed cache (like Redis) in a Boot microservice, and when would you need the latter?</summary>

A local cache (Caffeine) is fast but scoped to a single instance — in a horizontally-scaled deployment, each instance has its own independent cache, potentially serving stale/inconsistent data across instances. A distributed cache (Redis) is shared across all instances, ensuring consistency, at the cost of network latency per cache access — necessary whenever cache correctness/consistency across instances matters more than raw local speed.
</details>

<details>
<summary>79. What is Spring Boot's `spring-boot-starter-amqp`/`spring-boot-starter-kafka`, and what do they auto-configure?</summary>

Auto-configure a `RabbitTemplate`/`KafkaTemplate` and underlying connection/producer factories based on properties (`spring.rabbitmq.*`, `spring.kafka.*`), plus enable `@RabbitListener`/`@KafkaListener` annotation processing — removing the need to manually wire up connection factories and listener containers.
</details>

<details>
<summary>80. What is `spring.kafka.consumer.auto-offset-reset`, and what does `earliest` vs `latest` mean?</summary>

Controls what a Kafka consumer does when it has no previously committed offset for a partition (e.g., a brand-new consumer group). `earliest` — starts reading from the very beginning of the partition (reprocesses all historical messages). `latest` — starts reading only new messages published after the consumer connects, skipping history.
</details>

<details>
<summary>81. What is idempotent message consumption, and why does it matter in Kafka/RabbitMQ-integrated Boot services?</summary>

Message brokers commonly offer "at-least-once" delivery, meaning a message can be redelivered (e.g., after a consumer crash before committing an offset/ack). Idempotent consumption ensures processing the same message twice has no harmful side effect (e.g., checking if an order ID was already processed before creating it again) — critical since assuming exactly-once delivery without designing for it leads to duplicate-processing bugs.
</details>

<details>
<summary>82. What is a dead-letter queue (DLQ), and how do you configure one in a Spring Boot messaging consumer?</summary>

A separate queue/topic where messages that repeatedly fail processing are routed instead of being retried indefinitely (or silently dropped), letting you inspect/reprocess them later without blocking the main queue. Configured via broker-specific settings — e.g., RabbitMQ's `x-dead-letter-exchange` queue argument, or Kafka's `DefaultErrorHandler` with a `DeadLetterPublishingRecoverer` in Spring Kafka.
</details>

<details>
<summary>83. What is the purpose of `@KafkaListener`'s `groupId`, and how does consumer group assignment affect scaling?</summary>

Consumers sharing the same `groupId` split partition consumption among themselves — each partition is consumed by exactly one consumer in the group at a time, letting you scale message processing horizontally by adding more consumer instances (up to the number of partitions). Different `groupId`s each independently receive a full copy of every message (pub/sub style).
</details>

<details>
<summary>84. What is the difference between `spring-boot-starter-oauth2-client` and `spring-boot-starter-oauth2-resource-server`?</summary>

`oauth2-client` is used when your Boot app is the **client**, needing to authenticate users via an external identity provider (e.g., "Login with Google") and obtain tokens on their behalf. `oauth2-resource-server` is used when your Boot app **hosts a protected API** and needs to validate incoming access tokens (JWTs) presented by clients — different roles in the OAuth2 flow, often both used together in more complex setups.
</details>

<details>
<summary>85. What does `spring.security.oauth2.resourceserver.jwt.issuer-uri` configure, and what does Boot do with it automatically?</summary>

Points to the identity provider's issuer URL; Boot automatically fetches the provider's public signing keys (via its well-known OpenID configuration/JWKS endpoint) to validate incoming JWT signatures, without you manually managing key rotation or fetching keys yourself.
</details>

<details>
<summary>86. What is CSRF protection's default state in a Spring Boot Security app, and when should you disable it?</summary>

Enabled by default for any state-changing HTTP method when using Spring Security. Should generally remain enabled for browser-based, cookie/session-authenticated apps. Reasonable to disable for stateless, token-based (JWT-in-header) REST APIs consumed by non-browser clients, where there's no ambient session cookie for CSRF to exploit.
</details>

<details>
<summary>87. What is Boot's default password encoding behavior, and why shouldn't you use `NoOpPasswordEncoder`?</summary>

Boot's `spring-boot-starter-security` doesn't silently pick an encoder for you at the application level — you must explicitly configure a `PasswordEncoder` bean, and `BCryptPasswordEncoder` is the standard recommendation. `NoOpPasswordEncoder` stores/compares passwords in **plain text** — acceptable only for the most throwaway prototyping, never for anything resembling real credentials.
</details>

<details>
<summary>88. What is the significance of `SecurityFilterChain` bean configuration replacing `WebSecurityConfigurerAdapter` in modern Spring Security/Boot versions?</summary>

`WebSecurityConfigurerAdapter` (extend-and-override style) was deprecated in favor of a component-based approach — you now declare one or more `SecurityFilterChain` `@Bean`s directly, which is more composable (supports multiple independent filter chains for different URL patterns) and aligns better with Spring's general move toward explicit bean-based configuration over inheritance-based configuration classes.
</details>

<details>
<summary>89. What is the purpose of `spring-boot-starter-actuator`'s `management.endpoints.web.base-path` property?</summary>

Changes the base path actuator endpoints are exposed under (default `/actuator`) — useful for avoiding path collisions with your own API, or for standardizing a company-wide convention for where operational endpoints live across all services.
</details>

<details>
<summary>90. What is the purpose of running Actuator on a separate management port in production?</summary>

Isolates operational/monitoring endpoints from the public-facing application port, so they can be restricted at the network level (firewall/security group rules) without affecting how the main application traffic is exposed — reduces the attack surface for sensitive diagnostic endpoints.
</details>

<details>
<summary>91. What is the difference between `spring.output.ansi.enabled` and why might you disable colored console output?</summary>

Controls whether Boot's console log output uses ANSI color codes for readability in a terminal. Disable it (`NEVER`) when logs are captured by a log aggregator/file that doesn't render ANSI codes, where raw escape characters would just clutter the stored log text.
</details>

<details>
<summary>92. What is Boot's structured/JSON logging support (Boot 3.4+), and why is it useful in containerized environments?</summary>

Allows configuring log output directly as structured JSON (`logging.structured.format.console=ecs` or similar) without hand-rolling a Logback JSON encoder configuration — critical in Kubernetes/cloud environments where logs are typically shipped to a centralized log aggregator (ELK, Loki) that indexes structured fields far more effectively than parsing plain-text log lines.
</details>

<details>
<summary>93. What is a correlation ID / trace ID in logs, and how does Spring Boot help propagate it across a request?</summary>

A unique identifier attached to a request that's included in every log line generated while processing it (and ideally propagated to downstream service calls), letting you filter a log aggregator for all logs related to one specific request across multiple services. Micrometer Tracing auto-populates the MDC (Mapped Diagnostic Context) with trace/span IDs, which Logback's pattern layout can include automatically in every log line.
</details>

<details>
<summary>94. What is Spring Boot's `@Retryable` combined with an exponential backoff — how would you configure it?</summary>

```java
@Retryable(retryFor = TransientException.class, maxAttempts = 4,
    backoff = @Backoff(delay = 500, multiplier = 2))
public void callDownstream() { ... }
```
Each retry waits progressively longer (500ms, 1s, 2s...) rather than hammering a struggling downstream service at a constant rate — reduces load on a recovering service compared to fixed-interval retries.
</details>

<details>
<summary>95. What is Spring Boot's typical approach to feature flags, and how might `@ConditionalOnProperty` be (mis)used for this?</summary>

Simple boolean feature toggles can lean on `@ConditionalOnProperty` to conditionally register beans/behavior, but this only works well for coarse, deploy-time toggles (requires a restart to change). Real dynamic feature-flagging (toggled at runtime without redeploying) typically needs a dedicated feature-flag service/library (e.g., Unleash, LaunchDarkly, or a simple database-backed flag table checked at request time) rather than Spring's static conditional configuration.
</details>

<details>
<summary>96. What is the purpose of `@ConfigurationProperties`-based typed configuration versus scattering `@Value` across many classes, at scale?</summary>

Centralizes related configuration into a single, strongly-typed, IDE-autocomplete-friendly, validatable structure — much easier to maintain, test, and document than dozens of scattered `@Value("${...}")` annotations spread across unrelated classes with no clear grouping or single source of truth for what configuration a module actually needs.
</details>

<details>
<summary>97. What is the significance of immutable `@ConfigurationProperties` (using constructor binding) versus mutable (setter-based) binding?</summary>

Constructor binding (using a `record` or a class with a single constructor, Boot 2.2+) produces genuinely immutable configuration objects, preventing accidental runtime mutation and working cleanly with `final` fields — generally preferred over the older setter-based mutable style for its safety and compatibility with Java records.
</details>

<details>
<summary>98. What is the difference between Boot's `RestTemplateBuilder` customizers and `WebClient.Builder` customizers for cross-cutting HTTP client concerns (e.g., adding a common header to every outgoing call)?</summary>

Both support a customizer bean pattern — implementing `RestTemplateCustomizer`/a `WebClientCustomizer` — that Boot automatically applies to every auto-configured builder instance, letting you centralize concerns like default timeouts, common headers, or logging interceptors without repeating that setup at every call site that builds a client.
</details>

<details>
<summary>99. What is Boot's `spring.http.client.*` (Boot 3.x consolidated client properties), and why was this consolidation introduced?</summary>

Boot 3.x introduced a more unified way to configure common HTTP client settings (connect/read timeouts, redirect handling) that apply consistently across `RestClient`, `RestTemplate`, and `WebClient`, reducing the previous inconsistency where each client type had its own separate, differently-named configuration properties.
</details>

<details>
<summary>100. What is `RestClient` (introduced in Spring Framework 6.1 / Boot 3.2), and how does it compare to `RestTemplate` and `WebClient`?</summary>

`RestClient` is a modern, synchronous HTTP client with a fluent, `WebClient`-like builder API but without the reactive/non-blocking machinery — effectively a spiritual successor to `RestTemplate` for teams that want a clean, modern synchronous API without adopting the full reactive stack.
</details>

<details>
<summary>101. What is the purpose of `@ImportRuntimeHints` in Spring Boot 3, related to native image support?</summary>

Lets you programmatically register reflection, resource, and proxy hints needed for GraalVM's ahead-of-time compilation to correctly handle dynamic behavior (reflection, dynamic proxies) that the native-image build tool can't automatically detect through static analysis alone.
</details>

<details>
<summary>102. What is Ahead-of-Time (AOT) processing in Spring Boot 3, independent of full native-image compilation?</summary>

Boot 3 can perform AOT processing even for regular JVM deployments — pre-computing bean definitions and proxy classes at build time rather than through reflection-heavy processing at every startup, contributing to faster JVM startup as well, not just benefiting native-image builds.
</details>

<details>
<summary>103. What is Boot's checkpoint/restore support (CRaC) for fast startup, and what problem does it solve?</summary>

Coordinated Restore at Checkpoint (CRaC) lets a JVM snapshot its running state (after full warm-up/JIT optimization) to disk, and later "restore" from that checkpoint near-instantly — addressing the classic JVM cold-start latency problem for serverless/autoscaling workloads without the constraints of full native-image compilation.
</details>

<details>
<summary>104. What is the difference between Boot's build-time property placeholders (`@..@` in Maven) and runtime configuration properties?</summary>

Build-time placeholders (`${project.version}` resolved via Maven resource filtering into `application.properties` at build time) bake a fixed value into the artifact at compile time. Runtime configuration properties are resolved when the application actually starts, and can differ per environment/deployment without rebuilding the artifact — mixing the two purposes carelessly is a common source of "why doesn't changing this property do anything" confusion.
</details>

<details>
<summary>105. What is Spring Boot's `spring.profiles.group` feature?</summary>

Lets you define a profile that automatically activates a set of other profiles together — e.g., activating a `production` group profile that internally activates `prod-db`, `prod-security`, and `prod-logging` profiles, avoiding the need to list every individual profile explicitly at deployment time.
</details>

<details>
<summary>106. What is the difference between a Spring Boot "fat jar" and a Docker multi-stage build for the same application?</summary>

The fat jar is Boot's own packaging mechanism (all dependencies + embedded server in one artifact). A Docker multi-stage build is a separate, complementary concern — using one build stage (with a full JDK) to compile/package the fat jar, then copying just the resulting jar into a lean final stage (with only a JRE), producing a smaller, more secure final container image without build tools bundled into the runtime image.
</details>

<details>
<summary>107. What is the purpose of Cloud Native Buildpacks support (`spring-boot:build-image`) in Spring Boot?</summary>

Builds an OCI-compliant container image directly from your Spring Boot application without writing a `Dockerfile` at all — Buildpacks automatically detect the appropriate JDK, apply security best practices, and produce a layered image, standardizing image builds across many services/teams without each maintaining their own bespoke Dockerfile.
</details>

<details>
<summary>108. What is the difference between `ApplicationContext.close()` being called and the JVM simply exiting?</summary>

`close()` triggers Spring's orderly shutdown sequence — publishing a `ContextClosedEvent`, invoking `@PreDestroy` callbacks and `DisposableBean.destroy()` on singleton beans, releasing resources like connection pools cleanly. A raw JVM exit (`kill -9`, or letting the process just terminate) skips all of that, potentially leaving resources (open connections, unflushed buffers) in an inconsistent state.
</details>

<details>
<summary>109. What is the significance of registering a `Runtime.addShutdownHook()` versus relying purely on Spring's own lifecycle callbacks?</summary>

Spring's own `SpringApplication` already registers a shutdown hook internally by default that triggers `close()` on SIGTERM — you rarely need to add your own unless you have cleanup logic that must run even if the Spring context fails to fully initialize in the first place (a scenario Spring's own lifecycle callbacks can't cover, since they require a working context).
</details>

<details>
<summary>110. What is a practical checklist an experienced developer follows before deploying a Spring Boot service to production for the first time?</summary>

Externalize all environment-specific config (no hardcoded URLs/secrets); configure connection pool sizes appropriate to expected load and downstream DB limits; enable and secure Actuator health/readiness endpoints wired to orchestration probes; set sensible JVM memory flags (`-Xmx`) matching container resource limits; enable structured logging with correlation IDs; configure graceful shutdown; verify DB migrations run via Flyway/Liquibase rather than `ddl-auto=update`; add circuit breakers/timeouts around downstream calls; and load-test to validate the thread pool/connection pool sizing under realistic concurrency.
</details>

<details>
<summary>111. What is the difference between vertical and horizontal auto-scaling considerations for a stateless Spring Boot service in a container orchestrator?</summary>

Horizontal auto-scaling (adding more pod replicas) works cleanly for stateless Boot services since any instance can serve any request — the main consideration is ensuring shared resources (DB connection pool limits, external API rate limits) scale sensibly as instance count grows. Vertical scaling (bigger container resource limits) helps with per-request CPU/memory-intensive work but doesn't help with raw request throughput the way adding replicas does.
</details>

<details>
<summary>112. What is the purpose of setting explicit `-Xmx`/container memory limits, and what happens if a Boot app's JVM heap isn't sized relative to its container's memory limit?</summary>

If the JVM's heap can grow larger than the container's memory limit (especially relevant with older JVMs not properly container-aware), the container runtime's OOM killer can abruptly terminate the process when the container exceeds its cgroup memory limit — modern JVMs (Java 10+) are container-aware by default and size the heap as a percentage of the container's limit, but explicit, deliberate tuning is still recommended for predictable, well-understood behavior in production.
</details>

<details>
<summary>113. What is the difference between synchronous request-per-thread scaling limits (Spring MVC) versus using virtual threads (Java 21+) in a high-throughput Boot application?</summary>

Traditional MVC's thread-per-request model is bounded by the platform thread pool size — a large number of concurrent, slow (I/O-bound) requests can exhaust the pool, queueing further requests. Enabling virtual threads (`spring.threads.virtual.enabled=true` in Boot 3.2+) lets each request run on a cheap virtual thread, allowing far higher concurrency for I/O-bound workloads while keeping the simple, blocking-style programming model — without needing to rewrite the application in a reactive style.
</details>

<details>
<summary>114. What is the significance of `spring.threads.virtual.enabled` and what prerequisites does it have?</summary>

Requires Java 21+ (virtual threads are a JDK feature) and Spring Boot 3.2+. When enabled, the embedded Tomcat's request-handling executor and `@Async`/`TaskExecutor` beans switch to using virtual threads — but any blocking calls in the chain (JDBC drivers, synchronized blocks pinning the carrier thread) still need consideration, since a pinned virtual thread doesn't get the full scalability benefit.
</details>

<details>
<summary>115. What is thread pinning in the context of virtual threads, and why does it matter for a Spring Boot app adopting them?</summary>

Pinning occurs when a virtual thread can't be unmounted from its carrier (platform) thread during a blocking operation — historically caused by `synchronized` blocks (improved in newer JDK versions) or certain native method calls. A pinned virtual thread blocks its carrier thread just like a regular platform thread would, undermining the scalability benefit virtual threads are meant to provide for that specific code path.
</details>

<details>
<summary>116. What is a common anti-pattern when combining `@Transactional` with virtual threads or reactive programming?</summary>

`@Transactional` relies on a `ThreadLocal`-based (or similar) persistence context tied to the executing thread — combining it carelessly with reactive code (where execution can hop across different threads) breaks the transaction boundary silently. Virtual threads are generally safer here since each request still runs on one logical (virtual) thread throughout, but blocking JDBC calls within a transactional virtual-thread method should still be reviewed for pinning behavior under heavy load.
</details>

<details>
<summary>117. What is the purpose of Boot's `@Timed` annotation (Micrometer) for method-level metrics?</summary>

Automatically records execution time and invocation count for an annotated method as a Micrometer timer metric, without manually wrapping the method body in timing code — requires `@EnableAspectJAutoProxy` and the appropriate Micrometer aspect bean registered.
</details>

<details>
<summary>118. What is the difference between application-level metrics (custom business counters) and infrastructure-level metrics (JVM heap, GC pauses) that Boot exposes by default?</summary>

Infrastructure metrics (auto-exposed by Actuator/Micrometer: heap usage, GC pause time, thread count, HTTP request latency histograms) tell you about the health of the runtime and framework layer. Application-level metrics (a custom `orders.processed` counter) tell you about business-relevant outcomes — both are necessary for a complete observability picture, since infra metrics alone can't tell you if the business logic itself is behaving correctly.
</details>

<details>
<summary>119. What is the purpose of `management.metrics.tags.*` (or `MeterFilter`) for adding common tags/dimensions to all metrics?</summary>

Lets you attach consistent dimensional labels (e.g., `application=order-service`, `region=us-east`) to every emitted metric automatically, which is essential for distinguishing/filtering metrics correctly once multiple service instances/environments are all reporting to the same monitoring backend.
</details>

<details>
<summary>120. What is the risk of high-cardinality tags in Micrometer metrics, and how might a Spring Boot developer accidentally introduce one?</summary>

Tagging a metric with a value that has effectively unbounded distinct values (e.g., accidentally tagging an HTTP metric with a raw user ID or a full URL including path variables) explodes the number of unique time-series a monitoring backend must store and index — often silently overwhelming or crashing the metrics backend. Boot's default HTTP metrics deliberately use URI **templates** (`/users/{id}`) rather than literal paths specifically to avoid this pitfall; custom metrics need the same discipline.
</details>

<details>
<summary>121. What is the difference between `@Profile`-based bean swapping and dependency-injection-based strategy pattern for handling environment differences, and which is generally preferred at scale?</summary>

`@Profile` swaps entire bean implementations based on the active environment, which works but can become unwieldy and implicit as the number of environment-specific variations grows. A more explicit strategy-pattern approach (configuration-driven bean selection via `@ConditionalOnProperty` or an explicit factory) tends to scale better and stays more discoverable/testable than an ever-growing web of profile-gated beans.
</details>

<details>
<summary>122. What is Boot's `spring-boot-starter-actuator`'s `/actuator/shutdown` endpoint, and why is it disabled by default?</summary>

Allows gracefully shutting down the application via an HTTP call — disabled (`management.endpoint.shutdown.enabled=false` by default) because exposing an unauthenticated (or even authenticated but network-reachable) "kill the application" endpoint is an obvious and severe security/availability risk if not extremely carefully locked down.
</details>

<details>
<summary>123. What is Boot's default behavior for exception logging when an unhandled exception propagates out of a controller method, and how would you customize log level/format for it?</summary>

By default, Boot logs unhandled exceptions at `ERROR` level with a full stack trace via the default error-handling machinery. Customize behavior by implementing your own `@ExceptionHandler`/`@ControllerAdvice` and explicitly controlling what gets logged (and at what level) versus what's returned to the client — important to avoid leaking internal stack traces/details in API responses while still capturing full diagnostic detail server-side.
</details>

<details>
<summary>124. What is a good strategy for managing secrets (DB passwords, API keys) in a Spring Boot application, rather than hardcoding them in `application.yml`?</summary>

Never commit secrets to source control. Use environment variables injected at deploy time, a dedicated secrets manager (AWS Secrets Manager, HashiCorp Vault, GCP Secret Manager) integrated via Spring Cloud Vault or a custom `PropertySource`, or Kubernetes Secrets mounted as environment variables/files — with `application.yml` referencing placeholders (`${DB_PASSWORD}`) rather than literal values.
</details>

<details>
<summary>125. What is the purpose of `@ConfigurationProperties`-bound sensitive fields being masked in `/actuator/env` output?</summary>

Boot automatically sanitizes (masks with `******`) property values whose keys match common sensitive patterns (`password`, `secret`, `token`, `key`) when exposed via `/actuator/env`, preventing accidental credential leakage through an operational diagnostics endpoint — configurable via `management.endpoint.env.keys-to-sanitize` for custom sensitive property names.
</details>

<details>
<summary>126. What is the significance of dependency version alignment (`dependencyManagement`) in a multi-module Spring Boot project, and what problem does the Boot BOM solve?</summary>

Without centralized version management, different modules in a multi-module project could pull in mismatched, incompatible versions of shared libraries (e.g., different Jackson versions), causing subtle runtime failures. The Spring Boot BOM curates a tested, compatible version set for the whole Spring ecosystem plus common third-party libraries, so you generally don't specify explicit versions for dependencies it manages, avoiding this class of bug entirely.
</details>

<details>
<summary>127. What is the risk of overriding a Boot-managed dependency version manually, and when is it justified?</summary>

Overriding can introduce incompatibilities the Boot team has already tested against and avoided (e.g., a newer library version with a breaking API change Boot's auto-configuration doesn't account for). Justified when a specific CVE/security patch needs to be applied ahead of the next Boot release, but should be done deliberately, tested thoroughly, and reverted once Boot's own managed version catches up.
</details>

<details>
<summary>128. What is the purpose of running `mvn dependency:tree` (or Gradle's `dependencies` task) when debugging a Spring Boot dependency conflict?</summary>

Reveals the full transitive dependency graph and which version of a library actually "wins" (nearest-wins resolution in Maven) when multiple dependencies pull in different versions of the same library — essential for diagnosing `NoSuchMethodError`/`ClassNotFoundException` issues caused by an unexpected version actually ending up on the classpath.
</details>

<details>
<summary>129. What is the difference between compile-time annotation processing (Lombok, MapStruct) and Spring's own runtime reflection-based processing, in terms of build/runtime trade-offs?</summary>

Compile-time processors generate actual source/bytecode during the build (e.g., Lombok generating getters, MapStruct generating mapper implementations) — zero runtime reflection overhead, but requires correct build tool/IDE annotation-processing configuration. Spring's own bean wiring/AOP proxying happens via reflection/bytecode generation at **runtime**, which is more flexible (adapts to runtime conditions) but has a startup-time and per-call performance cost compared to purely compile-time-generated code.
</details>

<details>
<summary>130. What is the significance of Lombok's `@Data`/`@Builder` combined with JPA entities, and what pitfalls should you watch for?</summary>

Convenient for reducing boilerplate, but `@Data`'s auto-generated `equals()`/`hashCode()` based on **all** fields (including the ID, and potentially lazy-loaded associations) can cause subtle bugs with JPA entities — e.g., `hashCode()` changing after an entity gets its ID assigned post-persist, breaking `HashSet` behavior. Best practice: use `@EqualsAndHashCode(onlyExplicitlyIncluded = true)` scoped to just a stable business key, or avoid deriving equality from mutable/lazy fields entirely.
</details>

<details>
<summary>131. What is the purpose of a Boot application's `.gitignore` typically excluding `target/`, `build/`, and `application-local.yml`?</summary>

Build output directories are regenerated artifacts that shouldn't be version-controlled (bloats the repo, causes merge noise). `application-local.yml`-style files often hold developer-specific overrides (local DB credentials, personal API keys for testing) that shouldn't be shared/committed, keeping personal environment config out of the shared codebase.
</details>

<details>
<summary>132. What is the difference between a Spring Boot "smoke test" and a full regression test suite in a CI/CD pipeline context?</summary>

A smoke test is a small, fast, high-level check ("does the application start and respond to a basic health check?") run early in a pipeline to fail fast on gross breakages before investing time in a full, slower regression suite (comprehensive unit/integration tests) — a common pattern for keeping CI feedback loops fast for the common case while still running thorough validation before merge/deploy.
</details>

<details>
<summary>133. What is the purpose of contract testing (e.g., Spring Cloud Contract) between microservices, versus relying solely on integration tests?</summary>

Contract testing verifies that a service's API still matches the expectations of its consumers (defined as shared "contracts") **without** needing to spin up the actual consumer/producer together in every test run — catching breaking API changes early, independently, and faster than full end-to-end integration tests across the whole distributed system.
</details>

<details>
<summary>134. What is the difference between blue-green deployment and canary deployment, and how does Spring Boot's Actuator support either strategy?</summary>

**Blue-green** — two full production environments (blue = current, green = new); traffic switches entirely from one to the other once the new version is verified healthy. **Canary** — a small percentage of traffic is gradually shifted to the new version while monitoring for regressions, before rolling out fully. Actuator's health/readiness endpoints support both by letting the orchestrator/load balancer accurately determine which instances are healthy and ready to receive traffic during the transition, but the actual traffic-shifting logic lives in the deployment/orchestration layer (Kubernetes, a service mesh, or a load balancer), not in Boot itself.
</details>

<details>
<summary>135. What is the purpose of feature-complete integration tests running against a Testcontainers-managed real database in CI, versus only running them locally?</summary>

Ensures the exact same DB-dependent behavior is verified consistently and reproducibly in the CI pipeline (catching environment-specific "works on my machine" issues) rather than relying on developers to remember to run integration tests locally before pushing — critical for catching real dialect/behavior differences that an in-memory substitute database wouldn't reveal.
</details>

<details>
<summary>136. What is the significance of pinning exact Testcontainers image versions (rather than `latest`) in a Spring Boot test suite?</summary>

Using `latest` risks a test suite silently breaking (or silently passing against different behavior) whenever the underlying image is updated upstream, outside of your control — pinning specific, known-good versions keeps test runs deterministic and reproducible across time and different CI runs.
</details>

<details>
<summary>137. What is the difference between `spring.jpa.properties.hibernate.jdbc.batch_size` and per-repository `saveAll()` batching behavior?</summary>

Setting `hibernate.jdbc.batch_size` lets Hibernate group multiple `INSERT`/`UPDATE` statements into fewer round trips to the database — but this optimization only actually kicks in correctly when entity IDs use a batching-compatible generation strategy (e.g., not `IDENTITY` in some databases, which forces per-row inserts to retrieve generated keys) — a common gotcha where developers expect `saveAll()` to be automatically batched but it silently isn't, due to ID generation strategy mismatch.
</details>

<details>
<summary>138. What is the purpose of `spring.jpa.properties.hibernate.generate_statistics=true`, and when would you enable it?</summary>

Enables detailed Hibernate performance statistics logging (query counts, cache hit/miss ratios, entity load counts) — useful for diagnosing N+1 query problems or verifying second-level cache effectiveness during performance investigation, but adds overhead and noisy logging, so it's typically enabled temporarily for diagnosis rather than left on permanently in production.
</details>

<details>
<summary>139. What is the difference between `@Repository`'s exception translation and letting a raw `SQLException`/JDBC exception propagate?</summary>

Spring's `@Repository` (via a `PersistenceExceptionTranslationPostProcessor`) automatically converts low-level, vendor-specific persistence exceptions into Spring's unified, unchecked `DataAccessException` hierarchy — letting calling code catch a consistent, meaningful exception type (e.g., `DataIntegrityViolationException`) regardless of which underlying database/driver is actually in use, rather than coupling error-handling logic to vendor-specific SQL error codes.
</details>

<details>
<summary>140. What is the significance of `spring-boot-starter-validation`'s `@Valid` cascading to nested objects, and what annotation is required to enable it?</summary>

By default, `@Valid` only validates the top-level object's own annotated fields — for validation to cascade into a nested object field, that nested field itself must also be annotated with `@Valid` (not just the top-level parameter), otherwise constraint annotations on the nested object's fields are silently skipped.
</details>

<details>
<summary>141. What is a common mistake when combining `@Valid` request body validation with an update (`PUT`/`PATCH`) endpoint that receives a partial payload?</summary>

Applying the same DTO/validation group as a full "create" payload to a partial update can incorrectly reject valid partial updates (e.g., rejecting a payload missing a field that's required on create but legitimately omitted on a partial update) — addressed with validation groups (`@Validated(OnUpdate.class)`) or separate DTOs per operation, rather than reusing one validation-annotated class for fundamentally different request shapes.
</details>

<details>
<summary>142. What is the purpose of a `@RestControllerAdvice`'s `@ExceptionHandler` ordering when multiple handlers could match the same exception type?</summary>

Spring picks the most **specific** matching exception handler method based on the exception class hierarchy (a handler for the exact thrown exception type wins over a handler for a more general superclass) — but when true ambiguity/overlap exists across multiple `@ControllerAdvice` beans, `@Order` controls which advice bean's handler takes precedence.
</details>

<details>
<summary>143. What is the difference between throwing a custom business exception from a service layer versus returning an `Optional`/result-wrapper type for expected "not found" scenarios?</summary>

Exceptions are appropriate for genuinely exceptional, unexpected conditions; using them for routine, expected outcomes (like "user simply wasn't found") incurs unnecessary stack-trace-generation overhead and can make control flow harder to follow. Many teams prefer returning `Optional<User>` (or a dedicated result type) from repository/service lookups for the common "might not exist" case, reserving actual exceptions for truly exceptional failures (a downstream service being unreachable, a data integrity violation).
</details>

<details>
<summary>144. What is the significance of designing idempotent, retry-safe REST endpoints particularly for `POST` operations that create resources, and how might you achieve this in Spring Boot?</summary>

A `POST` retried after a network timeout (client unsure if the original request succeeded) risks creating duplicate resources unless explicitly designed to prevent it — commonly solved with an **idempotency key** pattern: the client sends a unique key header, and the server checks/stores it (often in Redis with a TTL) to detect and short-circuit duplicate submissions of the same logical request.
</details>

<details>
<summary>145. What is the purpose of a well-designed `@ExceptionHandler` returning a `Retry-After` header for rate-limited (`429`) responses?</summary>

Communicates to well-behaved clients exactly how long to wait before retrying, rather than leaving them to guess/retry immediately (worsening the overload condition) — a small but meaningful detail for building genuinely resilient, well-mannered distributed systems rather than just returning a bare status code.
</details>

<details>
<summary>146. What is a reasonable approach to API documentation for a Spring Boot REST service, and what does `springdoc-openapi` provide?</summary>

`springdoc-openapi` auto-generates an OpenAPI (Swagger) specification directly from your controllers/DTOs and their annotations at runtime, and can serve an interactive Swagger UI — keeping documentation automatically in sync with the actual code rather than relying on a separately maintained (and easily outdated) hand-written spec document.
</details>

<details>
<summary>147. What is the significance of annotating DTOs with OpenAPI-specific annotations (`@Schema`, `@Operation`) versus relying purely on auto-inferred documentation?</summary>

Auto-inferred documentation captures the structural shape (field names/types) accurately, but meaningful descriptions, examples, and edge-case documentation (what does a `404` actually mean here, what are valid enum values) require deliberate annotation — pure auto-generation alone rarely produces genuinely useful documentation for external API consumers without this additional annotation effort.
</details>

<details>
<summary>148. What is the purpose of API contract-first (design-first) development using an OpenAPI spec to generate Spring Boot interfaces, versus code-first (generating the spec from annotated code)?</summary>

Contract-first lets the API shape be agreed upon and reviewed (often across teams/consumers) **before** implementation begins, with server-side interfaces/stubs generated from the spec that the implementation must satisfy — reduces the risk of the implementation and documentation drifting apart, and enables consumer teams to start building against a mock/stub before the real implementation is even finished, compared to code-first's inherent risk of the spec always trailing slightly behind the actual code.
</details>

<details>
<summary>149. What is a good way to think about the trade-off between "convention over configuration" (Boot's philosophy) and explicit, verbose configuration, especially for a team scaling up a codebase?</summary>

Boot's defaults are excellent for getting started and for genuinely standard use cases, reducing boilerplate significantly — but as a system's requirements diverge from the common case (custom connection pooling behavior, non-standard security requirements, multiple datasources), leaning too heavily on "magic" auto-configuration without understanding what it's actually doing underneath can make debugging much harder for the team; experienced developers use the `--debug` auto-configuration report and read the relevant auto-configuration source when defaults don't behave as expected, rather than treating Boot as an opaque black box.
</details>

<details>
<summary>150. If asked to explain to a junior developer "what actually happens between `java -jar app.jar` and your first API response," what would you walk through?</summary>

The JVM starts and loads the fat jar's bootstrap launcher class, which sets up a classloader that can read nested jars; `SpringApplication.run()` is invoked, creating an `ApplicationContext`; Boot's auto-configuration mechanism scans the classpath and conditionally registers beans (DataSource, DispatcherServlet, etc.) alongside your own `@Component`-annotated beans discovered via component scanning; the embedded Tomcat (or other) server starts and binds to a port; once the context is fully refreshed, `ApplicationReadyEvent` fires and any `CommandLineRunner`s execute; the server begins accepting connections — an incoming HTTP request is received by the embedded server, dispatched through `DispatcherServlet` to the matching `@RequestMapping` controller method (possibly passing through Filters and `HandlerInterceptor`s first), the method executes (potentially involving `@Transactional` proxies, service/repository layers, and a real database round trip), and the return value is serialized (typically to JSON via Jackson) into the HTTP response sent back to the client.
</details>
