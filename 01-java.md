# Java Interview Preparation

## Core Java & JVM

### Q1. Explain the Java Memory Model (JMM).
<details><summary>Answer</summary>

The JMM defines how threads interact through memory and what guarantees Java provides around visibility, ordering and atomicity.

Key concepts:
- Each thread can have working/local state.
- Shared state lives in heap/main memory.
- `volatile` provides visibility and ordering guarantees, but not compound-operation atomicity.
- `synchronized`, locks and concurrent utilities establish happens-before relationships.
- Correct concurrent programs depend on happens-before rather than assumptions about CPU/cache behavior.

A strong production answer should distinguish **visibility, ordering and atomicity**.
</details>

### Q2. What is the difference between `==`, `equals()` and `hashCode()`?
<details><summary>Answer</summary>

`==` compares primitive values or object references. `equals()` expresses logical equality. `hashCode()` provides a hash used by hash-based collections.

Contract: if `a.equals(b)` is true, `a.hashCode()` must equal `b.hashCode()`.

If you override `equals()`, normally override `hashCode()` too. Violating the contract can cause objects to become effectively unreachable in `HashMap`/`HashSet`.
</details>

### Q3. How does HashMap work internally?
<details><summary>Answer</summary>

A `HashMap` uses an array of buckets. A key's hash is transformed into a bucket index. Collisions are stored within the bucket, using linked structures and, under suitable conditions in modern Java, tree bins.

Average lookup is approximately O(1), but performance depends on a good hash distribution and resizing.

Important interview points:
- Hash/equality contract
- collisions
- resizing/load factor
- mutable keys are dangerous
- not thread-safe for concurrent mutation
</details>

### Q4. HashMap vs ConcurrentHashMap?
<details><summary>Answer</summary>

`HashMap` is not designed for concurrent structural updates. `ConcurrentHashMap` supports concurrent access with substantially better concurrency than synchronizing the whole map.

`ConcurrentHashMap` also provides atomic compound operations such as `computeIfAbsent`.

Use `ConcurrentHashMap` when multiple threads need to safely mutate/read shared map state. Don't simply replace every HashMap with it; understand whether synchronization is actually required.
</details>

### Q5. ArrayList vs LinkedList?
<details><summary>Answer</summary>

`ArrayList` uses a dynamically sized array:
- O(1) indexed access
- efficient iteration/cache locality
- insertion/removal in the middle is O(n)

`LinkedList` provides node-based storage:
- indexed access is O(n)
- insertion/removal can be O(1) once the node is known
- higher memory overhead and poor locality

In most application code, `ArrayList` is the better default.
</details>

### Q6. Explain immutable objects and why String is immutable.
<details><summary>Answer</summary>

An immutable object cannot change after construction.

Benefits:
- thread safety by design
- safe sharing
- predictable hashing
- easier caching
- safer use as map keys

`String` immutability also supports string pooling and prevents surprising changes to values used in security-sensitive or framework operations.
</details>

### Q7. Checked vs unchecked exceptions?
<details><summary>Answer</summary>

Checked exceptions are enforced by the compiler and represent conditions callers may reasonably be expected to handle. Unchecked exceptions extend `RuntimeException`.

The important production question is not "checked or unchecked?" but whether the exception boundary communicates a meaningful recovery strategy.

Avoid swallowing exceptions or using exceptions for normal control flow.
</details>

### Q8. Explain garbage collection at a high level.
<details><summary>Answer</summary>

The JVM identifies objects that are no longer reachable and reclaims their memory. Modern collectors use generational concepts because most objects die young.

GC tuning depends on:
- allocation rate
- heap size
- latency requirements
- pause targets
- object lifetime
- collector choice

Production diagnosis should use GC logs, heap analysis, allocation profiling and application metrics rather than guessing.
</details>

### Q9. What is the difference between `synchronized`, Lock and atomic classes?
<details><summary>Answer</summary>

`synchronized` provides intrinsic locking and happens-before guarantees.

`Lock` APIs provide more control, such as timed acquisition, interruptible locking and multiple condition queues.

Atomic classes use non-blocking atomic operations such as CAS for suitable state transitions.

Choose the simplest mechanism that correctly protects the invariant. Atomic variables are not a universal replacement for locks.
</details>

### Q10. What is a race condition? Give a production example.
<details><summary>Answer</summary>

A race condition occurs when correctness depends on the timing/interleaving of concurrent operations.

Example: two requests read an account balance of 100, both subtract 80, and both write 20. Without appropriate concurrency control, the final state violates the business invariant.

Solutions can include database transactions/locking, optimistic concurrency, distributed locks or redesigning the state transition.
</details>

### Q11. ExecutorService vs creating threads manually?
<details><summary>Answer</summary>

An `ExecutorService` manages task execution separately from task submission and lets applications control concurrency, queueing and lifecycle.

Benefits include:
- bounded concurrency
- thread reuse
- centralized shutdown
- configurable rejection policies
- better observability

Creating an unbounded thread per request can exhaust memory and CPU under load.
</details>

### Q12. What are virtual threads and when would you use them?
<details><summary>Answer</summary>

Virtual threads are lightweight JVM-managed threads designed to make high-concurrency blocking I/O applications easier to write.

They are especially useful for I/O-heavy workloads where many operations spend time waiting.

They don't magically make CPU-bound work faster. You still need to consider database connection pools, downstream capacity and concurrency limits.
</details>

### Q13. Stream API: map vs flatMap?
<details><summary>Answer</summary>

`map` transforms one element into one result.

`flatMap` transforms an element into a stream and flattens nested streams into a single stream.

Example conceptually:
- `List<User>` → `map(User::getName)` → names
- `List<User>` → `flatMap(user -> user.getOrders().stream())` → one stream of all orders

Avoid complex streams when they make business logic harder to read or debug.
</details>

### Q14. What is Optional and what are common mistakes?
<details><summary>Answer</summary>

`Optional` represents a value that may be absent and can make return contracts clearer.

Avoid:
- using Optional as every field type
- calling `get()` blindly
- returning null from an Optional-returning method
- using it merely to make code look functional

Prefer expressive operations such as `map`, `flatMap`, `orElseGet` and `orElseThrow`.
</details>

### Q15. What would you check when a Java service suddenly has high CPU?
<details><summary>Answer</summary>

Start with metrics and correlate CPU with traffic/deployments.

Investigate:
1. thread CPU usage
2. thread dumps
3. hot methods/profiling
4. GC activity
5. request rate/latency
6. recent code/config changes
7. infinite loops or retry storms
8. expensive serialization/regex/queries
9. downstream failures causing busy retries

Do not immediately increase CPU without identifying the cause.
</details>

## Quick Revision Checklist

- JMM / happens-before
- HashMap internals
- equals/hashCode
- Collections complexity
- immutability
- exceptions
- GC
- concurrency
- ExecutorService
- CompletableFuture
- virtual threads
- Streams
- Optional
- profiling and production debugging
