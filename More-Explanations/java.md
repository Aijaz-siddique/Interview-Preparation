# Java — Interview Questions (Experienced)

> Click a question to expand the answer.

<details>
<summary>1. What is the difference between JDK, JRE, and JVM?</summary>

- **JVM (Java Virtual Machine)**: the runtime engine that executes bytecode. It's platform-specific but makes Java "write once, run anywhere" possible.
- **JRE (Java Runtime Environment)**: JVM + core libraries needed to *run* Java applications. No compiler.
- **JDK (Java Development Kit)**: JRE + development tools (`javac`, debugger, etc.) needed to *build* Java applications.

`JDK ⊃ JRE ⊃ JVM`
</details>

<details>
<summary>2. Explain the Java Memory Model — Heap vs Stack.</summary>

- **Stack**: one per thread. Stores method call frames, local variables, and references. LIFO. Cleaned up automatically when a method returns. `StackOverflowError` if too deep (e.g., infinite recursion).
- **Heap**: shared across all threads. Stores all objects and class-level (static) data lives in Metaspace (since Java 8, replacing PermGen). Managed by the Garbage Collector. `OutOfMemoryError: Java heap space` if it fills up.
- Heap is further divided into **Young Generation** (Eden + two Survivor spaces) and **Old Generation** — most objects die young, which is why generational GC is efficient.
</details>

<details>
<summary>3. How does Garbage Collection work? Name a few GC algorithms.</summary>

GC reclaims memory occupied by objects with no live references, using a **mark-and-sweep** approach at its core (mark reachable objects from GC roots, sweep the rest).

Common collectors in the JVM:
- **Serial GC** — single-threaded, good for small heaps/single-core.
- **Parallel GC** — multi-threaded, throughput-focused (default pre-Java 9).
- **CMS (Concurrent Mark Sweep)** — low-pause, deprecated since Java 9, removed in Java 14.
- **G1 (Garbage First)** — default since Java 9; divides heap into regions, balances throughput and pause time.
- **ZGC / Shenandoah** — ultra-low-pause collectors (sub-millisecond) for very large heaps, available in modern JDKs.
</details>

<details>
<summary>4. What's the difference between `==` and `.equals()`?</summary>

- `==` compares references (memory addresses) for objects, and values for primitives.
- `.equals()` compares logical/content equality — but only if the class overrides it (default `Object.equals()` also just does reference comparison).
- For `String`, `Integer` (within cache range -128 to 127), etc., always use `.equals()` unless you specifically want reference identity.
</details>

<details>
<summary>5. Why should you override `hashCode()` when you override `equals()`?</summary>

The `hashCode()`/`equals()` contract requires: if two objects are equal per `.equals()`, they **must** have the same `hashCode()`. Hash-based collections (`HashMap`, `HashSet`) use `hashCode()` to locate the bucket first, then `.equals()` to confirm the match within that bucket. Breaking this contract causes silent bugs — e.g., `map.get(key)` returning `null` even though an "equal" key was inserted, because it landed in a different bucket.
</details>

<details>
<summary>6. Explain `String` immutability. Why is `String` immutable in Java?</summary>

Once created, a `String`'s internal character array cannot be changed — every "modifying" method (`concat`, `replace`, `substring`) returns a new `String`.

Reasons:
- **String pool / interning**: identical literals can safely share the same object since it can never change underneath another reference.
- **Thread safety**: immutable objects are inherently thread-safe — no synchronization needed.
- **Security**: strings are used for class names, file paths, network connections — mutability would let e.g. security-sensitive class-loading. checks be bypassed.
- **Hashcode caching**: since content never changes, `hashCode()` can be computed once and cached, making `String` a fast `HashMap` key.
</details>

<details>
<summary>7. What are functional interfaces and lambda expressions? Give examples.</summary>

A **functional interface** has exactly one abstract method (may have default/static methods). Annotated with `@FunctionalInterface` (optional but recommended).

Built-in ones from `java.util.function`:
- `Function<T, R>` — takes T, returns R
- `Predicate<T>` — takes T, returns boolean
- `Consumer<T>` — takes T, returns nothing
- `Supplier<T>` — takes nothing, returns T

```java
Function<Integer, Integer> square = x -> x * x;
Predicate<String> isEmpty = String::isEmpty;
Consumer<String> print = System.out::println;
Supplier<Double> random = Math::random;
```

A **lambda expression** is syntactic sugar for an anonymous implementation of a functional interface.
</details>

<details>
<summary>8. Explain the Stream API — intermediate vs terminal operations.</summary>

Streams process collections declaratively (map/filter/reduce style) instead of imperative loops.

- **Intermediate operations** (lazy, return a new stream): `map()`, `filter()`, `sorted()`, `distinct()`, `limit()`, `flatMap()`.
- **Terminal operations** (trigger execution, produce a result): `collect()`, `forEach()`, `reduce()`, `count()`, `anyMatch()`.

```java
List<String> result = list.stream()
    .filter(x -> x.length() > 3)
    .map(String::toUpperCase)
    .sorted()
    .collect(Collectors.toList());
```

Streams are lazy — nothing executes until a terminal operation is called, and a stream can only be consumed once.
</details>

<details>
<summary>9. What is the difference between `Comparable` and `Comparator`?</summary>

- `Comparable<T>` — implemented by the class itself; defines the class's **natural ordering** via `compareTo()`. Only one natural order per class.
- `Comparator<T>` — a separate object defining a **custom ordering** via `compare(a, b)`. You can have many comparators for the same class, passed to `sort()` methods.

```java
class Employee implements Comparable<Employee> {
    public int compareTo(Employee o) { return Integer.compare(this.age, o.age); }
}
Comparator<Employee> bySalary = Comparator.comparing(Employee::getSalary);
```
</details>

<details>
<summary>10. Explain checked vs unchecked exceptions. When would you create a custom exception?</summary>

- **Checked exceptions** (extend `Exception`, not `RuntimeException`): must be declared (`throws`) or caught at compile time — e.g., `IOException`, `SQLException`. Used for recoverable conditions the caller is expected to handle.
- **Unchecked exceptions** (extend `RuntimeException`): not enforced at compile time — e.g., `NullPointerException`, `IllegalArgumentException`. Used for programming errors.

Create a **custom exception** when you need a domain-specific, meaningful error type (e.g., `InsufficientBalanceException`) that callers can catch specifically, often carrying extra context (error codes, metadata) beyond a generic exception.
</details>

<details>
<summary>11. What is the `volatile` keyword? How is it different from `synchronized`?</summary>

- `volatile` guarantees **visibility** — writes to a volatile variable by one thread are immediately visible to other threads (no caching in CPU registers/thread-local caches). It does **not** guarantee atomicity for compound operations (`i++` is still not thread-safe even if `i` is volatile).
- `synchronized` guarantees both **visibility and atomicity** — only one thread can execute the synchronized block/method at a time, and changes made within are visible to subsequent threads.

Use `volatile` for simple flags (e.g., a `boolean running` checked by multiple threads); use `synchronized` (or `java.util.concurrent` utilities) for compound state changes.
</details>

<details>
<summary>12. Explain the `synchronized` keyword and object-level vs class-level locking.</summary>

`synchronized` ensures mutual exclusion using an intrinsic lock (monitor).

- **Instance method / block on `this`**: locks on the object instance — different instances don't block each other.
- **Static method / block on `ClassName.class`**: locks on the `Class` object — shared across *all* instances, since there's one `Class` object per class.

```java
synchronized void instanceMethod() { }          // locks on 'this'
static synchronized void staticMethod() { }      // locks on MyClass.class
synchronized(lockObject) { }                      // locks on an explicit object
```
</details>

<details>
<summary>13. What is the difference between `ExecutorService`, `Future`, and `CompletableFuture`?</summary>

- **`ExecutorService`**: manages a thread pool; submit tasks via `submit()`/`execute()` instead of manually creating `Thread`s.
- **`Future<T>`**: represents the result of an async computation. `.get()` blocks until done. Limited — no easy way to chain or combine results without blocking.
- **`CompletableFuture<T>`**: a more powerful, non-blocking `Future` that supports chaining (`thenApply`, `thenCompose`), combining multiple futures (`thenCombine`, `allOf`), and exception handling (`exceptionally`, `handle`) — the modern way to write async pipelines in Java.

```java
CompletableFuture.supplyAsync(() -> fetchUser())
    .thenApply(user -> user.getName())
    .thenAccept(System.out::println);
```
</details>

<details>
<summary>14. What happens when two threads call a `synchronized` method on the same object concurrently?</summary>

Only one thread acquires the intrinsic lock (monitor) and executes the method; the other thread blocks until the lock is released (method completes or throws). The JVM guarantees mutual exclusion and a happens-before relationship — changes made by the first thread are visible to the second once it acquires the lock.
</details>

<details>
<summary>15. Explain `HashMap` internals — how does it handle collisions? What changed in Java 8?</summary>

`HashMap` stores entries in an array of buckets, indexed by `hash(key) % capacity`.

- **Collisions** (two keys hashing to the same bucket) are handled by chaining — each bucket holds a linked list of entries.
- **Java 8 change**: if a bucket's linked list grows beyond a threshold (8 entries, with table capacity ≥ 64), it's converted to a **red-black tree**, improving worst-case lookup from O(n) to O(log n) for that bucket — protects against hash-flooding attacks and pathological collision cases.
- Resizing (rehashing) happens when size exceeds `capacity * loadFactor` (default load factor 0.75), doubling the capacity.
</details>

<details>
<summary>16. What's the difference between `ArrayList` and `LinkedList`? When would you choose one over the other?</summary>

- `ArrayList`: backed by a dynamic array. O(1) random access (`get(i)`), O(n) insert/delete in the middle (shifting), O(1) amortized append.
- `LinkedList`: doubly-linked list. O(n) random access, O(1) insert/delete at a known node (head/tail or via iterator), higher per-element memory overhead (node pointers).

**Choose `ArrayList`** for most cases — it's more cache-friendly and has less overhead. **Choose `LinkedList`** only when you do frequent insertions/deletions at the head/middle via an iterator and rarely need random access (or need `Deque` semantics — though `ArrayDeque` is usually better even there).
</details>

<details>
<summary>17. What is the diamond problem in Java, and how do interfaces with default methods solve/avoid it?</summary>

The "diamond problem" arises when a class inherits from two sources that both provide the same method — which one wins? Java avoids ambiguity for **state** by disallowing multiple class inheritance entirely.

For **default methods** in interfaces (Java 8+), if a class implements two interfaces with the same default method signature, the compiler forces you to explicitly override and resolve it — often by calling `InterfaceName.super.methodName()` to pick one, preventing silent ambiguity.
</details>

<details>
<summary>18. What is the difference between `abstract class` and `interface`?</summary>

| | `abstract class` | `interface` |
|---|---|---|
| Multiple inheritance | No (single class inheritance) | Yes (implement many) |
| Fields | Can have instance state | Only `public static final` constants |
| Constructors | Yes | No |
| Method bodies | Yes (abstract + concrete) | Yes, via `default`/`static` methods (Java 8+) |
| Access modifiers | Any | Implicitly `public` (for abstract methods) |

Use an **abstract class** when subclasses share common state/behavior ("is-a" with shared implementation). Use an **interface** to define a contract/capability that unrelated classes can implement ("can-do").
</details>

<details>
<summary>19. Explain the difference between method overloading and overriding.</summary>

- **Overloading**: same method name, different parameter list, **same class** (or with inheritance) — resolved at **compile time** (static/early binding).
- **Overriding**: subclass provides a new implementation for a method with the **same signature** as in the parent — resolved at **runtime** (dynamic/late binding) via virtual method dispatch.

Overloading is about *having multiple versions* of a method; overriding is about *replacing the behavior* inherited from a parent.
</details>

<details>
<summary>20. What are the SOLID principles? Give a one-line description of each.</summary>

- **S — Single Responsibility**: a class should have only one reason to change.
- **O — Open/Closed**: open for extension, closed for modification (extend behavior via new code, not editing existing code).
- **L — Liskov Substitution**: subtypes must be substitutable for their base types without breaking correctness.
- **I — Interface Segregation**: prefer many small, specific interfaces over one fat interface.
- **D — Dependency Inversion**: depend on abstractions, not concrete implementations (high-level modules shouldn't depend on low-level details).
</details>

<details>
<summary>21. What is the difference between `final`, `finally`, and `finalize()`?</summary>

- `final` — a keyword/modifier: a `final` variable can't be reassigned, a `final` method can't be overridden, a `final` class can't be extended.
- `finally` — a block that always executes after a `try`/`catch`, whether or not an exception occurred (used for cleanup, e.g., closing resources).
- `finalize()` — a deprecated `Object` method the GC used to call before reclaiming an object; unreliable timing, deprecated since Java 9, removed in later versions. Use `try-with-resources` / `AutoCloseable` instead.
</details>

<details>
<summary>22. What are Java records (Java 14+)? Why use them?</summary>

Records are a concise way to declare immutable data-carrier classes. The compiler auto-generates the constructor, `equals()`, `hashCode()`, `toString()`, and accessor methods (named after fields, not `getX()`).

```java
record Point(int x, int y) {}
// equivalent to a full class with final fields, constructor, equals/hashCode/toString, x(), y()
```

Ideal for DTOs, value objects, and API response models where you want immutability and boilerplate-free equality without dragging in Lombok.
</details>

<details>
<summary>23. Explain String pool and how `new String("abc")` differs from `"abc"`.</summary>

The **String pool** (part of the heap since Java 7) caches literal strings — identical literals reuse the same object.

- `String a = "abc";` — reuses an existing pooled object if `"abc"` was already interned.
- `String b = new String("abc");` — always creates a **new** object on the heap, separate from the pool, even if `"abc"` is already pooled.
- `a == b` → `false` (different references). `a.equals(b)` → `true` (same content).
- `b.intern()` returns the pooled reference, making `a == b.intern()` → `true`.
</details>

<details>
<summary>24. What is the difference between `throw` and `throws`?</summary>

- `throw` — used inside a method body to actually **raise** an exception instance: `throw new IllegalArgumentException("bad input");`
- `throws` — used in a method **signature** to declare that the method might propagate a checked exception to its caller: `void readFile() throws IOException { ... }`
</details>

<details>
<summary>25. What's new/important in recent Java versions (11, 17, 21) an experienced dev should know?</summary>

- **Java 11 (LTS)**: `var` for local type inference, new `String` methods (`isBlank`, `strip`, `repeat`), HTTP Client API.
- **Java 17 (LTS)**: **Sealed classes** (restrict which classes can extend/implement a type), **Pattern matching for `instanceof`**, **Records** stabilized, improved `switch` expressions.
- **Java 21 (LTS)**: **Virtual threads** (Project Loom — lightweight threads for massive concurrency without the OS-thread overhead), **Pattern matching for `switch`**, **Record patterns** (destructuring records in `switch`/`instanceof`), sequenced collections.

Virtual threads in particular are a big deal for backend systems — they let you write simple blocking-style code that scales to hundreds of thousands of concurrent connections, competing with reactive programming for I/O-bound workloads without the complexity.
</details>

<details>
<summary>26. What is the difference between `List`, `Set`, and `Map`?</summary>

`List` is an ordered, index-based collection allowing duplicates (`ArrayList`, `LinkedList`). `Set` stores unique elements with no guaranteed order unless you use `LinkedHashSet` (insertion order) or `TreeSet` (sorted). `Map` stores key-value pairs with unique keys — it's not part of the `Collection` interface hierarchy at all.
</details>

<details>
<summary>27. Difference between `HashSet`, `LinkedHashSet`, and `TreeSet`?</summary>

`HashSet` — no ordering guarantee, O(1) average operations. `LinkedHashSet` — maintains insertion order, slight overhead over `HashSet`. `TreeSet` — sorted order (natural or via comparator), O(log n) operations, backed by a red-black tree.
</details>

<details>
<summary>28. Difference between `HashMap`, `LinkedHashMap`, and `TreeMap`?</summary>

`HashMap` — no ordering, O(1) average. `LinkedHashMap` — preserves insertion order (or access order if configured), useful for LRU caches. `TreeMap` — sorted by key, O(log n), supports range/navigation queries (`floorKey`, `ceilingKey`).
</details>

<details>
<summary>29. Is `HashMap` thread-safe? What are the alternatives?</summary>

No — concurrent modification can corrupt internal structure or cause infinite loops in older JDKs. Alternatives: `Collections.synchronizedMap()` (coarse locking, contention-prone), or preferably `ConcurrentHashMap` (segment/bucket-level locking, much better concurrent throughput).
</details>

<details>
<summary>30. How does `ConcurrentHashMap` achieve thread safety without locking the whole map?</summary>

Pre-Java 8 it used lock striping (segments). Since Java 8, it uses CAS (compare-and-swap) operations for most updates and synchronizes only on the specific bin (bucket) being modified, allowing many threads to read/write different parts of the map concurrently.
</details>

<details>
<summary>31. What is `CopyOnWriteArrayList` and when would you use it?</summary>

A thread-safe `List` variant where every mutation (`add`, `remove`) creates a new copy of the underlying array. Reads never block and never throw `ConcurrentModificationException`. Best for read-heavy, write-rare scenarios (e.g., a list of event listeners) since writes are expensive (O(n) copy each time).
</details>

<details>
<summary>32. What is `ConcurrentModificationException` and how do you avoid it?</summary>

Thrown when a collection is structurally modified while being iterated with a fail-fast iterator (most standard collections). Avoid it by using an explicit `Iterator.remove()`, `CopyOnWriteArrayList`, `ConcurrentHashMap`, collecting items to remove/add separately, or `removeIf()`.
</details>

<details>
<summary>33. What's the difference between `Iterator` and `ListIterator`?</summary>

`Iterator` supports forward-only traversal and `remove()`. `ListIterator` (only for `List`) supports bidirectional traversal (`hasPrevious`/`previous`), index access, and in-place `set()`/`add()` during iteration.
</details>

<details>
<summary>34. What is `fail-fast` vs `fail-safe` iteration?</summary>

Fail-fast iterators (e.g., `ArrayList`, `HashMap`) throw `ConcurrentModificationException` if the collection is structurally modified during iteration, detected via a modCount check. Fail-safe iterators (e.g., `CopyOnWriteArrayList`, `ConcurrentHashMap`) iterate over a snapshot or tolerate concurrent changes without throwing, though they may not reflect the very latest state.
</details>

<details>
<summary>35. What is the difference between `Collection` and `Collections`?</summary>

`Collection` is the root interface of the collections hierarchy (`List`, `Set`, `Queue`). `Collections` is a utility class with static helper methods (`sort`, `reverse`, `unmodifiableList`, `synchronizedList`, etc.).
</details>

<details>
<summary>36. What is an immutable collection and how do you create one?</summary>

A collection whose contents can't be changed after creation. Create via `List.of(...)`, `Map.of(...)`, `Set.of(...)` (Java 9+, truly immutable — throws `UnsupportedOperationException` on mutation), or `Collections.unmodifiableList(list)` (a read-only view — the underlying list can still change).
</details>

<details>
<summary>37. Explain `Optional` and why it was introduced.</summary>

`Optional<T>` is a container that may or may not hold a non-null value, used as a return type to explicitly signal "this might be absent" instead of returning `null` and risking `NullPointerException`. Use `.isPresent()`/`.map()`/`.orElse()`/`.orElseThrow()` instead of manual null checks. Avoid using it as a field type or method parameter — it's designed for return types.
</details>

<details>
<summary>38. What is the difference between `Array` and `ArrayList`?</summary>

Arrays have fixed size, can hold primitives directly, and are slightly faster. `ArrayList` is resizable, only holds objects (autoboxing for primitives), and offers a rich API (`add`, `remove`, `contains`).
</details>

<details>
<summary>39. What is `Deque` and how does it differ from `Queue`?</summary>

`Deque` (double-ended queue) supports insertion/removal at both ends; it's a superset of `Queue` (FIFO) and can also act as a `Stack` (LIFO) via `push`/`pop`. `ArrayDeque` is the preferred general-purpose implementation for both use cases.
</details>

<details>
<summary>40. What happens internally when a `HashMap` resizes?</summary>

When `size > capacity * loadFactor`, the internal array doubles in size, and every entry is rehashed and redistributed across the new, larger array of buckets — an O(n) operation. Frequent resizing can hurt performance, so specifying an initial capacity upfront helps when the size is roughly known.
</details>

<details>
<summary>41. What is load factor in `HashMap`?</summary>

The threshold (default 0.75) at which the map resizes. A lower load factor means more space used but fewer collisions/faster lookups; a higher load factor means less memory but more collisions.
</details>

<details>
<summary>42. Can you use a custom object as a `HashMap` key? What must you ensure?</summary>

Yes, but you must correctly override `equals()` and `hashCode()` consistently, and ideally make the key **immutable** — if the fields used in `hashCode()` change after insertion, the entry becomes unfindable (it's now in the "wrong" bucket for its current hash).
</details>

<details>
<summary>43. What is the difference between `PriorityQueue` and a regular `Queue`?</summary>

A regular `Queue` is FIFO. `PriorityQueue` orders elements by natural ordering or a comparator, always dequeuing the smallest (or highest priority) element next — internally backed by a binary heap array, not full ordering, so iteration order is not sorted.
</details>

<details>
<summary>44. What is `EnumMap` and `EnumSet`, and why are they more efficient than regular `HashMap`/`HashSet` for enums?</summary>

Both use the enum's ordinal value as an internal array index instead of hashing, making them faster and more memory-efficient than generic hash-based collections when keys/elements are all from the same enum type.
</details>

<details>
<summary>45. What's the difference between `Vector`, `Stack`, and their modern replacements?</summary>

`Vector` and `Stack` are legacy, synchronized (thread-safe but slow due to lock contention) collection classes from Java 1.0. Modern replacements: `ArrayList` (non-synchronized `Vector`), `ArrayDeque` (faster `Stack`), or `Collections.synchronizedList()`/`CopyOnWriteArrayList` if thread safety is genuinely needed.
</details>

<details>
<summary>46. What is a thread, and how do you create one in Java?</summary>

A thread is an independent path of execution within a process. Create one by extending `Thread` and overriding `run()`, or (preferred) implementing `Runnable`/`Callable` and passing it to a `Thread` or `ExecutorService` — preferred because it doesn't burn your one shot at class inheritance and separates the task from the execution mechanism.
</details>

<details>
<summary>47. What is the difference between `Runnable` and `Callable`?</summary>

`Runnable.run()` returns nothing and can't throw checked exceptions. `Callable<V>.call()` returns a value and can throw checked exceptions — used with `ExecutorService.submit()` to get a `Future<V>`.
</details>

<details>
<summary>48. Explain thread states in Java.</summary>

`NEW` (created, not started) → `RUNNABLE` (running or ready to run) → `BLOCKED` (waiting for a monitor lock) / `WAITING` (waiting indefinitely, e.g., `Object.wait()`) / `TIMED_WAITING` (waiting with a timeout, e.g., `sleep()`) → `TERMINATED` (finished).
</details>

<details>
<summary>49. What is a deadlock? How can you prevent it?</summary>

A deadlock occurs when two or more threads are each waiting for a lock held by the other, so none can proceed. Prevention: always acquire locks in a **consistent global order**, use timed lock attempts (`tryLock` with timeout), minimize lock scope, or use higher-level concurrency utilities that avoid manual locking altogether.
</details>

<details>
<summary>50. What is a race condition? Give an example.</summary>

A race condition occurs when the outcome depends on the unpredictable timing/interleaving of multiple threads accessing shared state. Classic example: two threads both reading a shared counter's value, incrementing it locally, and writing back — one increment can be lost because the read-modify-write isn't atomic.
</details>

<details>
<summary>51. What is the `java.util.concurrent.atomic` package used for?</summary>

Provides classes like `AtomicInteger`, `AtomicLong`, `AtomicReference` that support lock-free, thread-safe operations (`incrementAndGet`, `compareAndSet`) using CAS instructions at the hardware level — faster than `synchronized` for simple counters/flags under contention.
</details>

<details>
<summary>52. What is a `ReentrantLock` and how does it differ from `synchronized`?</summary>

`ReentrantLock` is an explicit lock offering more control: `tryLock()` (non-blocking attempt), timed acquisition, interruptible lock waits, and fairness policies (FIFO lock granting). Unlike `synchronized`, you must manually `unlock()` in a `finally` block — forgetting this causes a permanent deadlock, so it requires more discipline.
</details>

<details>
<summary>53. What is `CountDownLatch` and when would you use it?</summary>

A synchronization aid that lets one or more threads wait until a set of operations in other threads completes. Initialized with a count; each completing thread calls `countDown()`; waiting threads call `await()` and unblock once the count hits zero. One-time use only (can't reset). Common for "wait for N services to start before proceeding."
</details>

<details>
<summary>54. What is `CyclicBarrier` and how is it different from `CountDownLatch`?</summary>

`CyclicBarrier` makes a fixed number of threads wait for each other at a common barrier point before all proceeding together — and unlike `CountDownLatch`, it's **reusable** across multiple phases/cycles. Often used to synchronize parallel computation phases.
</details>

<details>
<summary>55. What is a `Semaphore` in Java concurrency?</summary>

A counter-based lock that controls access to a limited resource pool — threads `acquire()` a permit (blocking if none available) and `release()` it when done. Useful for limiting concurrent access, e.g., capping simultaneous database connections.
</details>

<details>
<summary>56. What is thread pooling and what pool types does `Executors` provide?</summary>

Thread pooling reuses a fixed set of worker threads instead of creating/destroying threads per task, reducing overhead. `Executors` factory methods: `newFixedThreadPool(n)`, `newCachedThreadPool()` (grows/shrinks as needed), `newSingleThreadExecutor()`, `newScheduledThreadPool()` (delayed/periodic tasks). In production, prefer constructing `ThreadPoolExecutor` directly for explicit control over queue size and rejection policy, since some `Executors` factories can lead to unbounded resource use.
</details>

<details>
<summary>57. What are the core parameters of `ThreadPoolExecutor`?</summary>

`corePoolSize` (threads kept alive even when idle), `maximumPoolSize` (upper bound), `keepAliveTime` (how long idle non-core threads survive), `workQueue` (where pending tasks wait), and `RejectedExecutionHandler` (policy when the pool and queue are both full — e.g., `AbortPolicy`, `CallerRunsPolicy`).
</details>

<details>
<summary>58. What is a virtual thread (Java 21+) and how is it different from a platform thread?</summary>

A virtual thread is a lightweight, JVM-managed thread that doesn't map 1:1 to an OS thread — thousands or millions can run on a small pool of OS "carrier" threads. When a virtual thread blocks on I/O, it's "unmounted" from its carrier thread, freeing it for other virtual threads. This makes simple blocking-style code scale to massive concurrency without the reactive-programming complexity.
</details>

<details>
<summary>59. What is the happens-before relationship in the Java Memory Model?</summary>

A guarantee that if action A happens-before action B, then A's effects (memory writes) are visible to B. Established by things like: a thread's actions before `Thread.start()` happen-before actions in the started thread; a `synchronized` block's actions happen-before a subsequent thread acquiring the same lock; a `volatile` write happens-before a subsequent `volatile` read of the same variable.
</details>

<details>
<summary>60. What is thread starvation and thread livelock?</summary>

**Starvation**: a thread is perpetually denied access to a resource because other threads (often higher priority or greedier) keep getting it first. **Livelock**: threads keep changing state in response to each other without making actual progress — e.g., two people repeatedly stepping aside for each other in a hallway.
</details>

<details>
<summary>61. What is the difference between `wait()`/`notify()` and `Condition` (from `Lock`)?</summary>

`wait()`/`notify()`/`notifyAll()` are `Object` methods used inside `synchronized` blocks for basic thread coordination. `Condition` (from `java.util.concurrent.locks`) is the more flexible equivalent for explicit `Lock`s — allows multiple independent condition queues per lock (`lock.newCondition()`), giving finer-grained signaling than the single implicit monitor per object.
</details>

<details>
<summary>62. Why should `wait()` always be called in a loop, not an `if`?</summary>

Because of spurious wakeups (a thread can wake up without an actual `notify()`) and because multiple threads might be waiting on the same condition — after waking, the condition that was signaled might no longer hold by the time this thread actually gets to run. A `while(!condition) wait();` loop re-checks the condition before proceeding.
</details>

<details>
<summary>63. What is the Fork/Join framework?</summary>

A framework (`ForkJoinPool`) designed for divide-and-conquer parallel tasks — a large task is recursively split ("forked") into smaller subtasks, executed in parallel, and results are combined ("joined"). Uses **work-stealing**: idle threads steal tasks from busy threads' queues to balance load. Powers parallel streams (`.parallelStream()`) under the hood.
</details>

<details>
<summary>64. What is `ThreadLocal` and when would you use it?</summary>

`ThreadLocal<T>` gives each thread its own independent copy of a variable, avoiding shared-state synchronization. Common uses: per-request context (e.g., current user, transaction, or locale in a web app), `SimpleDateFormat` instances (not thread-safe). Must be cleaned up (`.remove()`) in thread-pool environments to avoid memory leaks/stale data across reused threads.
</details>

<details>
<summary>65. What is the difference between parallelism and concurrency?</summary>

**Concurrency** is about structuring a program to handle multiple tasks that can make progress independently (may or may not run at the exact same instant — e.g., time-slicing on one core). **Parallelism** is about actually executing multiple tasks simultaneously on multiple cores. Concurrency is a design concept; parallelism is a runtime execution property.
</details>

<details>
<summary>66. What is a producer-consumer problem and how would you implement it in Java?</summary>

A classic concurrency pattern: producer threads generate data and consumer threads process it, coordinated via a shared, bounded buffer. Simplest Java implementation uses `BlockingQueue` (e.g., `ArrayBlockingQueue`) — `put()` blocks when full, `take()` blocks when empty, handling all the synchronization internally.
</details>

<details>
<summary>67. What is `BlockingQueue` and name a few implementations.</summary>

A thread-safe queue that blocks on `put()` when full and `take()` when empty, ideal for producer-consumer pipelines. Implementations: `ArrayBlockingQueue` (fixed-size, array-backed), `LinkedBlockingQueue` (optionally unbounded, linked-list-backed), `PriorityBlockingQueue` (priority-ordered), `SynchronousQueue` (zero capacity — a direct hand-off between threads).
</details>

<details>
<summary>68. Can you interrupt a running thread in Java? How?</summary>

Yes, via `thread.interrupt()` — this sets an internal interrupt flag. It doesn't forcibly stop the thread; the thread must cooperatively check `Thread.isInterrupted()` (or catch `InterruptedException` from a blocking call like `sleep()`/`wait()`) and decide how to respond, typically by cleaning up and exiting.
</details>

<details>
<summary>69. Why is stopping a thread with `Thread.stop()` deprecated/dangerous?</summary>

`stop()` terminates the thread immediately, releasing all its locks without any cleanup — this can leave shared objects in an inconsistent, corrupted state visible to other threads. It's deprecated; use cooperative cancellation via interrupt flags or a shared `volatile boolean` instead.
</details>

<details>
<summary>70. How do parallel streams work, and when should you avoid them?</summary>

`.parallelStream()` splits the source data and processes chunks across multiple threads using the common `ForkJoinPool`. Avoid them for: I/O-bound work (they don't help — the pool is sized for CPU-bound work), small datasets (overhead outweighs benefit), or when you're already inside another parallel/async context (can cause pool starvation since all parallel streams share one common pool by default).
</details>

<details>
<summary>71. What is the JIT (Just-In-Time) compiler?</summary>

The JVM initially interprets bytecode, then the JIT compiler identifies "hot" code paths (frequently executed methods/loops) and compiles them directly to native machine code at runtime, dramatically improving performance for long-running applications compared to pure interpretation.
</details>

<details>
<summary>72. What is class loading and the classloader hierarchy?</summary>

Class loading is the process of locating, reading, and initializing `.class` bytecode into the JVM at runtime. Hierarchy: **Bootstrap** classloader (loads core JDK classes) → **Platform/Extension** classloader → **Application/System** classloader (loads your app's classpath). Uses a **delegation model** — a classloader asks its parent first before trying to load a class itself, preventing core classes from being overridden.
</details>

<details>
<summary>73. What are the phases of class loading?</summary>

**Loading** (find and read the `.class` bytecode) → **Linking**, which is **Verification** (bytecode is valid/safe) + **Preparation** (allocate memory for static fields, default values) + **Resolution** (resolve symbolic references) → **Initialization** (run static initializers and static blocks).
</details>

<details>
<summary>74. What is Metaspace and how is it different from PermGen?</summary>

Metaspace (Java 8+) stores class metadata and replaces the old PermGen. Unlike PermGen (a fixed-size region of the heap prone to `OutOfMemoryError: PermGen space`), Metaspace lives in **native memory** and grows dynamically by default, largely eliminating that class of OOM error (though it can still exhaust native memory if unbounded).
</details>

<details>
<summary>75. What is escape analysis in the JVM?</summary>

A JIT optimization technique that determines whether an object's reference "escapes" the method/thread it was created in. If it doesn't escape, the JVM can optimize aggressively — allocate it on the stack instead of the heap (reducing GC pressure), or even eliminate the allocation entirely (scalar replacement).
</details>

<details>
<summary>76. What causes an `OutOfMemoryError` and how would you diagnose one in production?</summary>

Common causes: memory leaks (objects unintentionally retained, e.g., via static collections, unclosed resources, or `ThreadLocal` misuse), genuinely insufficient heap size for the workload, or too many threads (`OutOfMemoryError: unable to create new native thread`). Diagnosis: heap dump analysis (`jmap`, Eclipse MAT, VisualVM) to find retained object graphs, GC logs to observe memory trends over time, and profilers to trace allocation hotspots.
</details>

<details>
<summary>77. What is a memory leak in Java, given that it has garbage collection?</summary>

A memory leak happens when objects are no longer needed by the application logic but remain **reachable** from GC roots (so the GC can't reclaim them) — e.g., objects added to a static `List` and never removed, unclosed resources holding native memory, or listeners/caches that are never unregistered/evicted.
</details>

<details>
<summary>78. What are the different types of references in Java (strong, weak, soft, phantom)?</summary>

**Strong** — normal references; prevent GC as long as reachable. **Weak** (`WeakReference`) — collected as soon as no strong references exist, even if memory isn't tight (used in `WeakHashMap` for caches keyed by objects that shouldn't be kept alive). **Soft** (`SoftReference`) — collected only when the JVM is under memory pressure, good for memory-sensitive caches. **Phantom** (`PhantomReference`) — used for post-mortem cleanup tracking after an object is finalized, paired with a `ReferenceQueue`.
</details>

<details>
<summary>79. What is the difference between stack overflow and heap overflow errors?</summary>

`StackOverflowError` — the call stack exceeds its limit, typically from unbounded/too-deep recursion. `OutOfMemoryError: Java heap space` — the heap can't allocate a new object because it's full and the GC couldn't free enough space.
</details>

<details>
<summary>80. What JVM tuning flags should an experienced developer know?</summary>

`-Xms`/`-Xmx` (initial/max heap size), `-XX:+UseG1GC` (select GC algorithm), `-XX:MaxGCPauseMillis` (target pause time hint for G1), `-Xss` (thread stack size), `-XX:+HeapDumpOnOutOfMemoryError` (auto-capture a heap dump on OOM for later analysis).
</details>

<details>
<summary>81. What is bytecode, and why is it central to Java's portability?</summary>

Bytecode is the intermediate, platform-independent instruction set that `javac` compiles Java source into (`.class` files). Any JVM, on any OS/hardware, can execute the same bytecode — this is the "write once, run anywhere" promise, since only the JVM implementation (not the compiled program) needs to be platform-specific.
</details>

<details>
<summary>82. What is reflection in Java? What are its downsides?</summary>

Reflection (`java.lang.reflect`) lets code inspect and manipulate classes, methods, and fields at runtime — even private ones — without knowing them at compile time. Powers frameworks like Spring (dependency injection) and Jackson (serialization). Downsides: significant performance overhead versus direct calls, breaks encapsulation, bypasses compile-time type safety, and can be restricted by the module system (Java 9+ strong encapsulation).
</details>

<details>
<summary>83. What are annotations, and how are they processed?</summary>

Annotations are metadata attached to code elements (classes, methods, fields) that don't directly affect execution by themselves — they're read by tools/frameworks via reflection at runtime (`RUNTIME` retention, e.g., `@Autowired`) or by the compiler at build time (`SOURCE`/`CLASS` retention, e.g., annotation processors generating code like Lombok or MapStruct).
</details>

<details>
<summary>84. What is the Service Provider Interface (SPI) pattern in Java?</summary>

A mechanism (`java.util.ServiceLoader`) for discovering and loading implementations of an interface at runtime without hard-coding the implementation class — implementations register themselves via a file under `META-INF/services/`. Used by JDBC drivers, logging facades, and many plugin architectures.
</details>

<details>
<summary>85. What is the Java Platform Module System (JPMS, Java 9+)?</summary>

Introduced with `module-info.java`, it lets you explicitly declare a module's dependencies (`requires`) and what it exposes (`exports`), enabling **strong encapsulation** — internal packages aren't accessible outside the module even via reflection by default (unless `opens` is declared). Aimed at improving maintainability of large codebases and reducing the JDK's own footprint.
</details>

<details>
<summary>86. What are Java Generics and why were they introduced?</summary>

Generics let you parameterize types (`List<String>`, `Map<K, V>`), enabling compile-time type checking and eliminating the need for manual casting that pre-Java-5 code required (`Object` based collections). They catch type errors at compile time instead of `ClassCastException` at runtime.
</details>

<details>
<summary>87. What is type erasure in Java generics?</summary>

At compile time, generic type information is checked and then **erased** — replaced with `Object` (or the bound type) in the compiled bytecode, with casts inserted automatically. This means generic type information isn't available at runtime (`List<String>` and `List<Integer>` have the same runtime class), which is why you can't do `new T[10]` or `instanceof List<String>` directly.
</details>

<details>
<summary>88. What are bounded type parameters? Give an example.</summary>

Restrict a generic type to a specific supertype: `<T extends Comparable<T>>` means T must implement `Comparable`. This lets you call methods of the bound (e.g., `compareTo()`) within the generic code, which wouldn't otherwise be available on an unbounded `T`.
</details>

<details>
<summary>89. Explain wildcards: `? extends T` vs `? super T`.</summary>

**`? extends T`** (upper bounded) — the list holds T or a subtype; you can safely **read** T from it, but can't safely add to it (producer). **`? super T`** (lower bounded) — the list holds T or a supertype; you can safely **write** T to it, but reads only guarantee `Object` (consumer). Mnemonic: **PECS** — Producer Extends, Consumer Super.
</details>

<details>
<summary>90. Why can't you create a generic array in Java (`new T[10]`)?</summary>

Because of type erasure, the JVM doesn't know T's actual type at runtime, so it can't verify array store safety (arrays are covariant and check element types on every write — `ArrayStoreException`). Generics rely on compile-time checking with no runtime type info, so this combination isn't allowed directly; use `List<T>` or unchecked casts with `@SuppressWarnings` if truly necessary.
</details>

<details>
<summary>91. What is a raw type in generics, and why should you avoid it?</summary>

Using a generic class without specifying its type parameter (e.g., `List list = new ArrayList();` instead of `List<String>`). It bypasses compile-time type checking entirely, reintroducing the risk of `ClassCastException` at runtime — kept only for backward compatibility with pre-Java-5 code.
</details>

<details>
<summary>92. Can generic methods have their own type parameters independent of the class?</summary>

Yes — a static or instance method can declare its own type parameter(s), useful in utility methods: `public static <T> List<T> singletonList(T item) { ... }` — this works even in a non-generic class.
</details>

<details>
<summary>93. What is the difference between `List<Object>` and `List<?>`?</summary>

`List<Object>` can hold any type of object and you can add anything to it. `List<?>` (unbounded wildcard) represents a list of *some* unknown specific type — you can read elements as `Object`, but you cannot add anything (except `null`) since the compiler doesn't know the actual element type.
</details>

<details>
<summary>94. How do generics interact with method overloading?</summary>

Due to type erasure, you can't overload two methods that only differ by generic type parameter after erasure — e.g., `void process(List<String> list)` and `void process(List<Integer> list)` in the same class won't compile; they'd have the same erased signature `process(List)`.
</details>

<details>
<summary>95. What is a self-bounded generic type (recursive generic), and where is it used?</summary>

A pattern like `class Enum<E extends Enum<E>>` where a type parameter is bounded by a generic type involving itself — enables type-safe fluent builders and comparisons where a subtype's methods return the subtype itself. `Comparable<T>`'s typical usage (`class MyClass implements Comparable<MyClass>`) is a simpler, common instance of this idea.
</details>

<details>
<summary>96. What is exception chaining, and why is it useful?</summary>

Wrapping a lower-level exception inside a higher-level, more meaningful one while preserving the original as the "cause" (`new ServiceException("failed", originalException)`), accessible via `getCause()`. Preserves the full stack trace/root cause for debugging while presenting a cleaner abstraction to callers.
</details>

<details>
<summary>97. What is try-with-resources, and what interface must a resource implement?</summary>

A `try` block that automatically closes resources when it exits (normally or via exception), eliminating manual `finally` cleanup. Resources must implement `AutoCloseable` (or `Closeable`, a sub-interface). Multiple resources close in reverse order of declaration.

```java
try (BufferedReader br = new BufferedReader(new FileReader("file.txt"))) {
    return br.readLine();
} // br.close() called automatically
```
</details>

<details>
<summary>98. What is a suppressed exception in try-with-resources?</summary>

If an exception is thrown both from the try block body **and** from a resource's `close()` method during cleanup, the body's exception is the one propagated, and the `close()` exception is attached to it as a "suppressed" exception, retrievable via `getSuppressed()` — so neither error is silently lost.
</details>

<details>
<summary>99. What is the difference between `Error` and `Exception`?</summary>

`Exception` represents conditions an application might reasonably want to catch and handle. `Error` represents serious, typically unrecoverable JVM-level problems (`OutOfMemoryError`, `StackOverflowError`) that applications generally shouldn't try to catch/handle — they signal something has gone fundamentally wrong with the runtime environment.
</details>

<details>
<summary>100. What are some best practices for exception handling in production code?</summary>

Catch specific exceptions, not broad `Exception`/`Throwable`, unless at a top-level boundary handler; never swallow exceptions silently (empty catch blocks); include context in exception messages; use custom exceptions for domain errors; log at the point where you have the most context, don't log-and-rethrow the same exception repeatedly at every layer; and clean up resources deterministically with try-with-resources.
</details>

<details>
<summary>101. What is the Singleton design pattern, and how do you implement a thread-safe one in Java?</summary>

Ensures a class has only one instance globally. Thread-safe implementations: **eager initialization** (instance created at class load — simplest, no lazy benefit), **double-checked locking** with a `volatile` field, or (cleanest) an **enum singleton**, which is inherently thread-safe and serialization-safe by the JVM's guarantees:

```java
enum Singleton { INSTANCE; void doWork() { } }
```
</details>

<details>
<summary>102. What is the Factory pattern, and how does it differ from Abstract Factory?</summary>

**Factory Method** — a method that creates and returns objects of a type, letting subclasses decide which concrete class to instantiate. **Abstract Factory** — a factory of factories: an interface for creating families of related objects without specifying their concrete classes, useful when a system needs to be independent of how its products are created/composed.
</details>

<details>
<summary>103. What is the Builder pattern, and when would you use it?</summary>

Constructs complex objects step by step, avoiding "telescoping constructors" (many overloaded constructors for optional parameters). Useful for objects with many optional fields — often paired with method chaining for a fluent API.

```java
User user = User.builder().name("Alice").age(30).build();
```
</details>

<details>
<summary>104. What is the Observer pattern? Where is it used in the JDK/Spring?</summary>

Defines a one-to-many dependency: when a subject's state changes, all registered observers are notified automatically. JDK example: `PropertyChangeListener`. Spring example: `ApplicationEventPublisher` and `@EventListener` for application events.
</details>

<details>
<summary>105. What is the Strategy pattern?</summary>

Defines a family of interchangeable algorithms behind a common interface, letting the algorithm vary independently from the client using it. Example: a `Comparator` passed into `Collections.sort()` — the sorting logic (client) stays the same regardless of which comparison strategy is injected.
</details>

<details>
<summary>106. What is the Decorator pattern? Give a JDK example.</summary>

Wraps an object to add behavior dynamically without altering its class or affecting other instances of the same class. JDK example: `java.io` streams — `new BufferedReader(new FileReader("file.txt"))` layers buffering behavior onto a basic file reader.
</details>

<details>
<summary>107. What is the Adapter pattern?</summary>

Converts one interface into another that a client expects, letting incompatible interfaces work together without modifying either side's source code. Example: wrapping a legacy payment API's methods behind your application's standard `PaymentGateway` interface.
</details>

<details>
<summary>108. What is the Proxy pattern, and how does Spring use it?</summary>

A proxy object controls access to a real object, adding behavior transparently (lazy loading, access control, logging) around it. Spring uses dynamic proxies (JDK proxies for interfaces, CGLIB for classes) extensively for `@Transactional`, AOP, and `@Async` — the "bean" you inject is often actually a proxy wrapping the real object.
</details>

<details>
<summary>109. What is the Template Method pattern?</summary>

Defines the skeleton of an algorithm in a base class method, deferring specific steps to subclasses via overridable methods, without letting subclasses change the algorithm's overall structure. Example: `JdbcTemplate` in Spring — it handles the boilerplate (connection, statement, cleanup) while you supply the specific query/row-mapping logic.
</details>

<details>
<summary>110. What is the difference between the Facade and Adapter patterns?</summary>

**Facade** simplifies a complex subsystem by providing a single, simpler interface over it (same underlying interface family, just simplified). **Adapter** converts one specific interface into a different, incompatible one that a client expects — it's about compatibility, not simplification.
</details>

<details>
<summary>111. What is the Command pattern?</summary>

Encapsulates a request/action as an object, letting you parameterize clients with different requests, queue them, log them, or support undo/redo. Common in GUI actions, job queues, and transactional/undoable operations.
</details>

<details>
<summary>112. What is the Chain of Responsibility pattern? Where might you see it in Java frameworks?</summary>

Passes a request along a chain of handlers, each deciding whether to process it or pass it to the next. Seen in servlet **filter chains** and Spring Security's filter chain — each filter can act on the request and/or delegate to the next.
</details>

<details>
<summary>113. What is dependency injection as a design pattern (independent of Spring)?</summary>

A specific form of the broader Inversion of Control principle where an object's dependencies are supplied externally (by a constructor, setter, or framework) rather than the object creating them itself — decoupling classes from concrete implementations and making unit testing straightforward via mocks/stubs.
</details>

<details>
<summary>114. What's the difference between composition and inheritance? Why is "favor composition over inheritance" common advice?</summary>

Inheritance ("is-a") creates tight coupling to a parent's implementation and can break with changes to the base class (the fragile base class problem); it's also limited to single inheritance in Java. Composition ("has-a") builds behavior by combining smaller objects, offering more flexibility, easier testing, and avoiding deep, brittle class hierarchies.
</details>

<details>
<summary>115. What is the difference between a design pattern and an architectural pattern?</summary>

A **design pattern** solves a recurring problem at the class/object level within a codebase (Singleton, Factory, Observer). An **architectural pattern** addresses the overall structure of a system at a much larger scale (Layered architecture, Microservices, Event-Driven architecture, MVC).
</details>

<details>
<summary>116. What is polymorphism, and what are its two types in Java?</summary>

Polymorphism lets objects of different types be treated through a common interface. **Compile-time (static)** polymorphism — method overloading, resolved at compile time. **Runtime (dynamic)** polymorphism — method overriding, resolved via virtual dispatch based on the actual object type at runtime.
</details>

<details>
<summary>117. What is encapsulation, and how is it achieved in Java?</summary>

Bundling data (fields) and behavior (methods) together while restricting direct external access to internal state — achieved via `private` fields with public getter/setter methods (or, more idiomatically, exposing only well-designed behavior methods rather than raw getters/setters everywhere).
</details>

<details>
<summary>118. What is the difference between IS-A and HAS-A relationships?</summary>

**IS-A** — inheritance relationship (`Dog extends Animal`, a Dog *is an* Animal). **HAS-A** — composition/association relationship (`Car has an Engine`) — the Car class holds a reference to an Engine object rather than extending it.
</details>

<details>
<summary>119. Can a constructor be private? Why would you do that?</summary>

Yes — used to prevent direct instantiation from outside the class, forcing object creation through a static factory method (Factory pattern) or to enforce a Singleton.
</details>

<details>
<summary>120. What is a static nested class vs an inner (non-static) class?</summary>

A **static nested class** doesn't hold an implicit reference to an instance of the outer class — it behaves like a top-level class, just namespaced inside another. An **inner class** holds an implicit reference to its enclosing instance and can access its instance members directly, but requires an outer instance to be created (`outer.new Inner()`).
</details>

<details>
<summary>121. What is an anonymous inner class? When would you use one over a lambda?</summary>

A class defined and instantiated in a single expression without a name, typically implementing an interface or extending a class inline. Use lambdas for functional interfaces (single abstract method) for conciseness; use anonymous classes when you need to implement an interface with multiple methods, maintain instance state across calls, or extend a concrete class.
</details>

<details>
<summary>122. What is method hiding vs method overriding for static methods?</summary>

Static methods can't be truly overridden (no dynamic dispatch) — if a subclass declares a static method with the same signature as a superclass's static method, it **hides** it rather than overrides it. Which method runs is determined by the **reference type** at compile time, not the actual object type at runtime, unlike instance method overriding.
</details>

<details>
<summary>123. What is covariant return type in method overriding?</summary>

Since Java 5, an overriding method can return a subtype of the return type declared in the overridden (parent) method, rather than requiring an exact match — useful for returning more specific types in subclasses without breaking the override contract.
</details>

<details>
<summary>124. What is the `super` keyword used for?</summary>

Refers to the immediate parent class — used to call the parent's constructor (`super(...)`, must be the first statement), access a parent's overridden method (`super.methodName()`), or access a parent's field shadowed by the subclass.
</details>

<details>
<summary>125. What is object cloning in Java, and what's the difference between shallow and deep copy?</summary>

Cloning creates a copy of an object via `Object.clone()` (requires implementing the `Cloneable` marker interface). **Shallow copy** copies field values directly — if a field is a reference type, both the original and the copy point to the *same* underlying object. **Deep copy** recursively copies referenced objects too, so the copy is fully independent. `clone()` is widely considered clunky/error-prone in practice; copy constructors or factory methods are often preferred.
</details>

<details>
<summary>126. What is serialization in Java? What does `Serializable` do?</summary>

Serialization converts an object's state into a byte stream (for storage or network transmission), and deserialization reconstructs it. A class must implement the `Serializable` marker interface (no methods to implement) to opt in — the JVM handles the mechanics via reflection.
</details>

<details>
<summary>127. What is `serialVersionUID` and why is it important?</summary>

A version identifier for a serializable class. If not declared explicitly, the JVM computes one based on class details — if the class changes even slightly (adding a field) and you deserialize old data with a new class version without a matching `serialVersionUID`, you get an `InvalidClassException`. Declaring it explicitly gives you control over compatibility across versions.
</details>

<details>
<summary>128. How do you exclude a field from serialization?</summary>

Mark it `transient` — its value is skipped during serialization and reset to the default value (`0`, `null`, `false`) upon deserialization. Commonly used for sensitive data (passwords), derived/cacheable fields, or non-serializable resources like open file handles.
</details>

<details>
<summary>129. What is the difference between `Serializable` and `Externalizable`?</summary>

`Serializable` uses default JVM-driven reflection-based serialization. `Externalizable` gives you full manual control by implementing `writeExternal()`/`readExternal()` yourself — more code, but better performance and control over the exact byte format, useful for versioning or performance-critical serialization.
</details>

<details>
<summary>130. What are the main classes in Java I/O, and what's the difference between byte streams and character streams?</summary>

**Byte streams** (`InputStream`/`OutputStream` and subclasses like `FileInputStream`) handle raw binary data. **Character streams** (`Reader`/`Writer` and subclasses like `FileReader`, `BufferedReader`) handle text, automatically managing character encoding — always prefer character streams for text data to avoid encoding bugs.
</details>

<details>
<summary>131. What is the difference between `BufferedReader` and `Scanner`?</summary>

`BufferedReader` is faster for reading raw lines of text (buffers input to reduce I/O calls) but has minimal parsing capability. `Scanner` is more convenient for parsing tokens/primitives (`nextInt()`, `nextLine()`) using regex internally, but is noticeably slower for large inputs due to that overhead.
</details>

<details>
<summary>132. What is NIO (New I/O), and how does it differ from classic I/O?</summary>

`java.nio` introduced non-blocking, buffer-and-channel-based I/O (as opposed to classic stream-based blocking I/O), along with `Selector`s that let a single thread monitor multiple channels for readiness — enabling scalable I/O multiplexing for servers handling many connections with fewer threads. `java.nio.file` (NIO.2, Java 7+) also modernized file-system operations (`Files`, `Path`).
</details>

<details>
<summary>133. What is the difference between `Files.readAllLines()` and streaming a file with `Files.lines()`?</summary>

`Files.readAllLines()` loads the **entire** file into memory as a `List<String>` — fine for small files. `Files.lines()` returns a lazily-evaluated `Stream<String>`, reading line by line, suitable for large files where loading everything into memory would be wasteful or infeasible — but the stream must be closed (try-with-resources) since it holds an open file handle.
</details>

<details>
<summary>134. What is the difference between `Path` and `File`?</summary>

`File` (legacy, `java.io`) represents a file/directory path with a limited, sometimes inconsistent API (e.g., silent failures returning `false` instead of exceptions). `Path` (`java.nio.file`, Java 7+) is the modern replacement, offering richer operations, symbolic link support, and better exception-based error handling via the `Files` utility class.
</details>

<details>
<summary>135. How would you copy a large file efficiently in Java?</summary>

Use `Files.copy(source, target, StandardCopyOption.REPLACE_EXISTING)` — it lets the JVM/OS use efficient native mechanisms internally. For very large files or maximum control, use `FileChannel.transferTo()`/`transferFrom()`, which can leverage zero-copy OS-level transfer, avoiding unnecessary data copying between kernel and user space.
</details>

<details>
<summary>136. What is the `switch` expression (Java 14+) and how does it differ from a traditional `switch` statement?</summary>

`switch` expressions use `->` syntax, return a value directly, don't fall through between cases by default, and can use `yield` for multi-statement branches — eliminating the classic missing-`break` bug and reducing boilerplate.

```java
String result = switch (day) {
    case MONDAY, FRIDAY -> "Busy";
    case SATURDAY, SUNDAY -> "Relax";
    default -> "Normal";
};
```
</details>

<details>
<summary>137. What are sealed classes/interfaces (Java 17+)?</summary>

`sealed` restricts which classes/interfaces are allowed to extend/implement a type, declared explicitly via `permits`. Combined with pattern matching in `switch`, the compiler can verify exhaustiveness (all subtypes are handled) — giving algebraic-data-type-like safety in Java.

```java
sealed interface Shape permits Circle, Square {}
```
</details>

<details>
<summary>138. What is pattern matching for `instanceof` (Java 16+)?</summary>

Combines the type check and cast into one step, eliminating boilerplate: `if (obj instanceof String s) { System.out.println(s.length()); }` — `s` is automatically cast and scoped to the `if` block (and beyond, if the compiler can prove flow reachability).
</details>

<details>
<summary>139. What are text blocks (Java 15+)?</summary>

Multi-line string literals using triple quotes (`"""`) that preserve formatting without needing explicit `\n` and escaped quotes — much cleaner for embedding JSON, SQL, or HTML snippets directly in code.

```java
String json = """
    {"name": "Alice"}
    """;
```
</details>

<details>
<summary>140. What are local variable type inference (`var`, Java 10+) rules and best practices?</summary>

`var` lets the compiler infer a local variable's type from its initializer — it's still statically typed (not like JavaScript's dynamic typing), just less verbose. Best practice: use it when the type is obvious from the right-hand side (`var list = new ArrayList<String>();`) — avoid it when it would obscure meaning (`var result = process();` gives no clue what `result` is).
</details>

<details>
<summary>141. What is the difference between `Stream.of()`, `Arrays.stream()`, and `Collection.stream()`?</summary>

All produce a `Stream`, just from different sources: `Stream.of(a, b, c)` from individual varargs elements, `Arrays.stream(array)` from an existing array (with primitive-specialized overloads like `IntStream`), and `collection.stream()` as a default method on any `Collection`.
</details>

<details>
<summary>142. What is `Collectors.groupingBy()` and give a practical example.</summary>

Groups stream elements into a `Map` keyed by a classifier function, useful for SQL-`GROUP BY`-style aggregation:

```java
Map<Department, List<Employee>> byDept = employees.stream()
    .collect(Collectors.groupingBy(Employee::getDepartment));
```
Can be combined with a downstream collector, e.g., `Collectors.groupingBy(Employee::getDepartment, Collectors.counting())`.
</details>

<details>
<summary>143. What is the difference between `map()` and `flatMap()` in streams?</summary>

`map()` transforms each element 1-to-1 (`Stream<T>` → `Stream<R>`). `flatMap()` transforms each element into a stream and then **flattens** all those streams into a single stream — used when each input maps to *multiple* output elements (e.g., a `Stream<List<String>>` flattened into a single `Stream<String>`).
</details>

<details>
<summary>144. What is a Method Reference in Java, and what are its four types?</summary>

Shorthand syntax for a lambda that just calls an existing method: `Type::method`. Four forms: static method (`Math::abs`), instance method on a particular object (`str::length`), instance method on an arbitrary object of a type (`String::toUpperCase`), and constructor reference (`ArrayList::new`).
</details>

<details>
<summary>145. What is the difference between `Predicate.and()`, `or()`, and `negate()`?</summary>

Default methods on `Predicate` that let you compose multiple predicates without writing new lambda logic: `p1.and(p2)` (both must be true), `p1.or(p2)` (either true), `p1.negate()` (inverts the result) — enabling readable, declarative filtering chains.
</details>

<details>
<summary>146. What is the difference between `reduce()` and `collect()` in streams?</summary>

`reduce()` combines stream elements into a single result using an associative accumulator function, best suited for immutable accumulation (summing numbers, concatenating). `collect()` is designed for mutable reduction — accumulating into a mutable container (`List`, `Map`, `StringBuilder`) via a supplier/accumulator/combiner, which is generally more efficient for building collections.
</details>

<details>
<summary>147. What is lazy evaluation in streams, and why does it matter?</summary>

Intermediate stream operations (`map`, `filter`) aren't executed until a terminal operation is invoked — the stream builds a pipeline of operations and processes elements one at a time through the whole pipeline, rather than materializing intermediate collections at each step. This allows short-circuiting (e.g., `findFirst()` stops as soon as a match is found) and avoids unnecessary work.
</details>

<details>
<summary>148. What is the difference between checked exceptions and lambdas — why can't lambdas easily throw checked exceptions?</summary>

Functional interfaces like `Function<T, R>` don't declare `throws` in their abstract method signature, so a lambda implementing them can't throw a checked exception directly without wrapping it (e.g., rethrow as an unchecked exception, or use a custom functional interface that declares `throws Exception`). This is a common practical annoyance when mixing streams with checked-exception-throwing APIs (like I/O).
</details>

<details>
<summary>149. What is the `Comparator.comparing().thenComparing()` chaining pattern?</summary>

Builds a multi-level sort comparator declaratively: sort primarily by one key, and for ties, fall back to a secondary key, and so on.

```java
list.sort(Comparator.comparing(Employee::getDept)
    .thenComparing(Employee::getSalary, Comparator.reverseOrder()));
```
</details>

<details>
<summary>150. What are some common Java performance pitfalls an experienced developer should avoid?</summary>

String concatenation with `+` in loops (use `StringBuilder`), autoboxing in tight loops (unnecessary `Integer`/`int` conversions), catching exceptions for normal control flow (expensive stack trace generation), not closing resources (leaks), using `ArrayList.contains()`/`remove()` in a loop over large lists (O(n) each call — consider `HashSet` instead), and premature/unverified optimization without profiling first.
</details>
