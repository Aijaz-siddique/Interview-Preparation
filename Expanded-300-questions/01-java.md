# Java Interview Preparation — Experienced Level

Designed for senior/lead interviews: internals, concurrency, performance, design and production troubleshooting.

## JVM, Memory & Language Internals

### Q1. Explain JDK, JRE and JVM.
<details><summary>Answer</summary>

JVM executes bytecode; the JRE conceptually contains the JVM plus runtime libraries; the JDK adds development tools such as the compiler and diagnostic utilities. Modern Java distributions commonly package the runtime and development tooling together, but the conceptual distinction remains useful.

</details>

### Q2. Explain the Java class-loading process.
<details><summary>Answer</summary>

Class loading generally involves loading, linking and initialization. Linking includes verification, preparation and resolution. Classes are loaded through class loaders and initialization executes static initialization when required.

</details>

### Q3. What is parent delegation in class loading?
<details><summary>Answer</summary>

A class loader normally asks its parent to load a class before trying itself. This prevents application classes from accidentally replacing core platform classes and provides predictable class identity.

</details>

### Q4. What causes ClassNotFoundException vs NoClassDefFoundError?
<details><summary>Answer</summary>

ClassNotFoundException commonly occurs when code explicitly requests a class that cannot be found. NoClassDefFoundError means the JVM cannot load a class definition that was expected to be available, often because a dependency is missing or initialization previously failed.

</details>

### Q5. Heap vs stack in Java?
<details><summary>Answer</summary>

The heap stores objects and is managed by GC. Each thread has stack frames containing local variables, operand-stack state and call information. Stack memory is thread-local; heap objects are generally shared.

</details>

### Q6. What is metaspace?
<details><summary>Answer</summary>

Metaspace stores JVM class metadata outside the traditional Java heap. Excessive class generation or class-loader leaks can cause metaspace growth.

</details>

### Q7. What is escape analysis?
<details><summary>Answer</summary>

The JVM can analyze whether an object escapes a method/thread. If it does not, JIT optimizations may eliminate allocations or synchronization in suitable cases. Do not assume every small object is stack allocated.

</details>

### Q8. What is JIT compilation?
<details><summary>Answer</summary>

The JVM initially interprets or compiles bytecode and profiles execution. Hot code can be compiled into optimized native machine code by the JIT, with speculative optimizations that may later be deoptimized.

</details>

### Q9. What is the difference between strong, soft, weak and phantom references?
<details><summary>Answer</summary>

Strong references keep objects alive. Soft references may be cleared under memory pressure. Weak references do not prevent collection. Phantom references are used with reference queues for advanced lifecycle/cleanup patterns.

</details>

### Q10. Why is finalization discouraged?
<details><summary>Answer</summary>

Finalization is unpredictable and unsuitable for deterministic resource management. Prefer try-with-resources, explicit lifecycle APIs and cleaners only for specialized safety-net cases.

</details>

### Q11. What is the String pool?
<details><summary>Answer</summary>

The JVM maintains a pool of canonical String instances. String literals are pooled, and `intern()` can request canonicalization. Because String is immutable, sharing is safe.

</details>

### Q12. String vs StringBuilder vs StringBuffer?
<details><summary>Answer</summary>

String is immutable. StringBuilder is mutable and generally preferred for single-threaded construction. StringBuffer provides synchronized methods and is usually unnecessary when external synchronization is already controlled.

</details>

### Q13. What are records?
<details><summary>Answer</summary>

Records are concise classes for transparent data carriers with final components, generated accessors, equality/hashCode and a canonical constructor. They are useful for immutable DTO/value-like data.

</details>

### Q14. What are sealed classes?
<details><summary>Answer</summary>

Sealed classes restrict which classes can extend or implement a type. They make a hierarchy explicit and can improve domain modeling and exhaustiveness reasoning.

</details>

### Q15. What is pattern matching useful for?
<details><summary>Answer</summary>

Pattern matching can combine type checks with binding/extraction, reducing boilerplate and making branching logic clearer. It is particularly useful when handling well-defined polymorphic or sealed hierarchies.

</details>

## Collections & Generics

### Q16. How does HashMap work internally?
<details><summary>Answer</summary>

HashMap uses buckets selected from a key hash. Collisions are handled within buckets; modern JDK implementations can treeify heavily collided buckets under suitable conditions. Resizing changes bucket distribution. Average lookup is O(1), but depends on hash quality.

</details>

### Q17. Why must equals() and hashCode() agree?
<details><summary>Answer</summary>

If two objects are equal, they must have the same hash code. Hash-based collections use the hash to locate a bucket and equals to resolve equality. Breaking the contract can make logically present keys impossible to find.

</details>

### Q18. Why are mutable HashMap keys dangerous?
<details><summary>Answer</summary>

If fields participating in hashCode/equals change after insertion, the object can effectively move from the collection's logical perspective while remaining in the old bucket. Lookups may then fail.

</details>

### Q19. HashMap vs Hashtable vs ConcurrentHashMap?
<details><summary>Answer</summary>

HashMap is unsynchronized. Hashtable is legacy synchronized collection. ConcurrentHashMap is designed for concurrent access with better scalability and atomic compound operations.

</details>

### Q20. How does ConcurrentHashMap achieve concurrency?
<details><summary>Answer</summary>

Modern implementations use fine-grained synchronization/CAS and volatile state rather than one global lock for all operations. Exact internals vary by JDK, so focus on concurrency guarantees rather than memorizing implementation details.

</details>

### Q21. ArrayList vs LinkedList?
<details><summary>Answer</summary>

ArrayList provides fast indexed access and excellent locality but middle insertion/removal is O(n). LinkedList has node-based insertion/removal once positioned but poor locality and O(n) indexed access. ArrayList is usually the default.

</details>

### Q22. HashSet vs TreeSet vs LinkedHashSet?
<details><summary>Answer</summary>

HashSet offers hash-based average O(1) operations without ordering. LinkedHashSet preserves insertion order. TreeSet maintains sorted order with O(log n) operations.

</details>

### Q23. Comparable vs Comparator?
<details><summary>Answer</summary>

Comparable defines a type's natural ordering. Comparator defines external/custom ordering. Comparator is useful when a type needs multiple valid orderings.

</details>

### Q24. What is type erasure?
<details><summary>Answer</summary>

Java generics are largely implemented through type erasure, so generic type information is not generally available as runtime parameterized type information. This explains restrictions such as inability to instantiate `new T()` directly.

</details>

### Q25. Why can't you create a generic array directly?
<details><summary>Answer</summary>

Arrays are reified while generic type parameters are erased. Creating `new T[]` would create runtime type-safety problems. Use collections or carefully designed array/reflection techniques.

</details>

### Q26. PECS in generics?
<details><summary>Answer</summary>

Producer Extends, Consumer Super. Use `? extends T` when reading values as T from a producer; use `? super T` when writing T into a consumer.

</details>

### Q27. What is fail-fast iteration?
<details><summary>Answer</summary>

Many collection iterators detect structural modification and may throw ConcurrentModificationException. It is a bug-detection mechanism, not a synchronization guarantee.

</details>

## Concurrency & Async

### Q28. Explain the Java Memory Model.
<details><summary>Answer</summary>

The JMM defines visibility and ordering guarantees between threads. Synchronization mechanisms establish happens-before relationships. Correct concurrent code must reason about those guarantees rather than hardware assumptions.

</details>

### Q29. What is happens-before?
<details><summary>Answer</summary>

It is a formal ordering relationship: if action A happens-before B, B is guaranteed to observe the effects relevant under the memory model. Lock release/acquisition, volatile writes/reads and thread lifecycle operations can establish happens-before edges.

</details>

### Q30. volatile vs synchronized?
<details><summary>Answer</summary>

volatile provides visibility and ordering for a variable but does not make compound operations such as increment atomic. synchronized provides mutual exclusion plus memory visibility.

</details>

### Q31. What is CAS?
<details><summary>Answer</summary>

Compare-and-set atomically updates a value only if it still equals an expected value. CAS is the basis for many lock-free/concurrent structures but can suffer contention and ABA-related issues.

</details>

### Q32. What is the ABA problem?
<details><summary>Answer</summary>

A thread observes value A, another thread changes A→B→A, and the first thread cannot tell the value changed. Version stamps or other designs can distinguish the state transitions.

</details>

### Q33. CountDownLatch vs CyclicBarrier vs Semaphore?
<details><summary>Answer</summary>

CountDownLatch waits for a count to reach zero and is generally one-shot. CyclicBarrier lets a group repeatedly meet at a barrier. Semaphore controls permits for concurrent access to a resource.

</details>

### Q34. ExecutorService vs ForkJoinPool?
<details><summary>Answer</summary>

ExecutorService is a general task-execution abstraction. ForkJoinPool is optimized for recursive/parallel tasks using work stealing. Common async frameworks may also use ForkJoinPool defaults, so choose explicitly when workload isolation matters.

</details>

### Q35. How do you size a thread pool?
<details><summary>Answer</summary>

For CPU-bound work, start near available processors and measure. For I/O-bound work, more concurrency can be useful, but downstream capacity, queueing, memory and connection pools become constraints. Use bounded pools and load testing.

</details>

### Q36. What is CompletableFuture?
<details><summary>Answer</summary>

It represents an asynchronously completed result and supports composition, transformation, combination and exception handling. Avoid blocking its worker threads unnecessarily.

</details>

### Q37. thenApply vs thenCompose?
<details><summary>Answer</summary>

thenApply transforms a result into another value. thenCompose flattens an asynchronous operation that itself returns a CompletionStage, avoiding nested futures.

</details>

### Q38. What causes deadlock?
<details><summary>Answer</summary>

A deadlock can occur when threads wait cyclically for locks. Prevent it through consistent lock ordering, reduced lock scope, timed acquisition and careful ownership design.

</details>

### Q39. What is starvation?
<details><summary>Answer</summary>

Starvation occurs when a thread repeatedly fails to obtain CPU or required resources. Causes include unfair locks, overloaded pools and priority misuse.

</details>

### Q40. What is livelock?
<details><summary>Answer</summary>

Threads remain active but keep responding to each other without making progress, often because of overly polite retry/release behavior.

</details>

### Q41. How do virtual threads help?
<details><summary>Answer</summary>

Virtual threads make large numbers of blocking-style tasks cheaper than platform threads. They are useful for I/O-heavy workloads but do not increase CPU capacity or remove downstream connection limits.

</details>

## Streams, Functional Java & Exceptions

### Q42. map vs flatMap?
<details><summary>Answer</summary>

map transforms each element to one value. flatMap transforms each element to a stream and flattens the results into one stream.

</details>

### Q43. Intermediate vs terminal stream operations?
<details><summary>Answer</summary>

Intermediate operations build a lazy pipeline; terminal operations trigger evaluation. Examples include filter/map as intermediate and collect/count/reduce as terminal.

</details>

### Q44. Why can parallel streams be dangerous in services?
<details><summary>Answer</summary>

They use shared execution resources and can introduce contention, unexpected blocking and poor performance. Parallelism should be measured and isolated, especially in server applications.

</details>

### Q45. reduce vs collect?
<details><summary>Answer</summary>

reduce combines values into a single result using an associative reduction. collect is designed for mutable result containers and has collector-specific parallel semantics.

</details>

### Q46. orElse vs orElseGet?
<details><summary>Answer</summary>

orElse evaluates its argument eagerly even when the Optional contains a value. orElseGet evaluates a supplier lazily only when needed.

</details>

### Q47. When should Optional be avoided?
<details><summary>Answer</summary>

Avoid using Optional everywhere, especially as entity fields or method parameters without a clear contract. It is most useful for expressing potentially absent return values.

</details>

### Q48. Checked vs unchecked exceptions?
<details><summary>Answer</summary>

Checked exceptions force declaration/handling and can be appropriate for recoverable conditions. Unchecked exceptions are often used for programming errors and application-layer failures. The key is meaningful recovery semantics.

</details>

### Q49. What is try-with-resources?
<details><summary>Answer</summary>

It automatically closes AutoCloseable resources even when an exception occurs. It also preserves suppressed exceptions from close operations.

</details>

### Q50. Why should exceptions not be swallowed?
<details><summary>Answer</summary>

Swallowing exceptions destroys failure information and can make the system appear successful while state is incorrect. Log or translate failures at an appropriate boundary and preserve context.

</details>

### Q51. How would you design an exception hierarchy?
<details><summary>Answer</summary>

Create domain-specific exceptions where callers need different recovery behavior. Avoid dozens of classes with no semantic value. Map them to stable API error codes at the boundary.

</details>

## Performance & Production Debugging

### Q52. How would you investigate high CPU in a Java service?
<details><summary>Answer</summary>

Correlate CPU with traffic and deployments, inspect thread CPU, thread dumps, profiles, GC, hot methods, retries and downstream failures. Identify the hot path before changing capacity.

</details>

### Q53. How would you investigate high memory?
<details><summary>Answer</summary>

Check heap/non-heap usage, GC behavior, allocation rate, heap dumps, retained objects, caches, queues and native memory. Look for leaks and unbounded collections.

</details>

### Q54. What is a memory leak in Java?
<details><summary>Answer</summary>

An object leak occurs when objects are no longer logically needed but remain reachable, preventing GC. Common causes include static collections, caches without eviction, listeners and ThreadLocal misuse.

</details>

### Q55. What is a thread dump useful for?
<details><summary>Answer</summary>

It shows thread states and stack traces. It helps identify blocked threads, deadlocks, hot loops and pool exhaustion.

</details>

### Q56. What metrics matter for a Java API?
<details><summary>Answer</summary>

Track throughput, p50/p95/p99 latency, error rate, saturation, CPU/memory, GC pauses, thread pools, DB pool utilization, downstream latency and queue depth.

</details>

### Q57. How do you find an N+1 problem?
<details><summary>Answer</summary>

Trace or profile database calls per request. If loading N parent records triggers N additional queries, use joins/fetch strategies, batching or dedicated queries appropriate to the access pattern.

</details>

### Q58. Why can increasing a thread pool make performance worse?
<details><summary>Answer</summary>

More threads increase context switching, queue contention and downstream pressure. If the bottleneck is a DB or remote service, more concurrency can amplify latency and failures.

</details>

### Q59. What is backpressure?
<details><summary>Answer</summary>

Backpressure prevents producers from overwhelming consumers by bounding queues/concurrency or signaling demand. Without it, queues and memory can grow until the system fails.

</details>

### Q60. What is graceful degradation?
<details><summary>Answer</summary>

When dependencies fail or load spikes, the system intentionally provides reduced functionality rather than failing completely—for example serving cached data or disabling non-critical features.

</details>

## Modern Java Design

### Q61. Why prefer composition over inheritance?
<details><summary>Answer</summary>

Composition keeps behavior dependencies explicit and avoids rigid class hierarchies. Inheritance is valuable when there is a genuine substitutable relationship.

</details>

### Q62. What is dependency inversion?
<details><summary>Answer</summary>

High-level policy should depend on abstractions rather than concrete low-level implementations. Dependency injection is one practical mechanism for applying this principle.

</details>

### Q63. How do you design an immutable class?
<details><summary>Answer</summary>

Make state private/final, initialize completely in constructors/factories, avoid mutators, defensively copy mutable inputs/outputs and ensure referenced objects are appropriately immutable.

</details>

### Q64. What is defensive copying?
<details><summary>Answer</summary>

Create copies of mutable objects when accepting or exposing them so callers cannot mutate internal state unexpectedly.

</details>

### Q65. What makes a class thread-safe?
<details><summary>Answer</summary>

Its invariants remain correct under concurrent access. Immutability is simplest; otherwise use proper synchronization, confinement or concurrent structures.

</details>

### Q66. What are common singleton pitfalls?
<details><summary>Answer</summary>

Global mutable state, hidden dependencies, test isolation problems and unsafe publication. Framework-managed singleton scope is usually preferable to hand-written singleton patterns in applications.

</details>

## Question Count

**66 experienced-level questions** in this file.

## Quick Revision Checklist

- JVM/class loading
- Memory/GC/JIT
- Collections/generics
- Concurrency/async
- Streams/Optional/exceptions
- Performance/debugging
- Modern Java/design

