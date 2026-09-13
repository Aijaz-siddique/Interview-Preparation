# System Design — High-Level Design (HLD) — Interview Questions (Experienced)

<details>
<summary>1. What is the difference between High-Level Design (HLD) and Low-Level Design (LLD)?</summary>

HLD focuses on the overall architecture of a system — major components, how they communicate, data flow, technology choices, and how the system scales/handles failure — without diving into class structures or algorithms. LLD focuses on the detailed internal design of individual components — class diagrams, database schemas, API contracts, design patterns used within a module.
</details>

<details>
<summary>2. What are the key non-functional requirements you should clarify before designing any system?</summary>

Scale (requests per second, data volume, number of users), latency requirements, availability requirements (uptime SLA), consistency requirements (strong vs eventual), read/write ratio, and cost/budget constraints — these drive nearly every subsequent architectural decision, so gathering them first is standard practice before proposing a design.
</details>

<details>
<summary>3. What is the difference between vertical scaling and horizontal scaling?</summary>

**Vertical scaling** (scale up) — adding more resources (CPU, RAM, disk) to a single machine; simple, but has a hard physical/cost ceiling and creates a single point of failure. **Horizontal scaling** (scale out) — adding more machines and distributing load across them; more complex (requires load balancing, data partitioning, and dealing with distributed-systems problems) but scales much further and improves fault tolerance.
</details>

<details>
<summary>4. What is a load balancer, and what algorithms do load balancers commonly use?</summary>

A load balancer distributes incoming traffic across multiple backend servers to avoid overloading any single one and to improve availability. Common algorithms: **Round Robin** (cycle through servers evenly), **Least Connections** (send to the server with fewest active connections), **Weighted Round Robin** (accounts for servers with different capacities), **IP Hash** (route based on client IP for session stickiness), **Least Response Time**.
</details>

<details>
<summary>5. What is the difference between Layer 4 (L4) and Layer 7 (L7) load balancing?</summary>

**L4** load balancing operates at the transport layer (TCP/UDP) — routes based on IP/port without inspecting the actual content of the request, faster but less flexible. **L7** load balancing operates at the application layer (HTTP) — can route based on URL path, headers, or cookies, enabling more intelligent routing (e.g., sending `/api/*` to one service and `/static/*` to another) at the cost of slightly more processing overhead.
</details>

<details>
<summary>6. What is a reverse proxy, and how does it differ from a forward proxy?</summary>

A **forward proxy** sits in front of clients, forwarding their requests to the internet on their behalf (hides the client from the server — e.g., a corporate proxy). A **reverse proxy** sits in front of servers, forwarding client requests to the appropriate backend (hides the server infrastructure from the client) — commonly used for load balancing, SSL termination, caching, and request routing (e.g., Nginx, HAProxy).
</details>

<details>
<summary>7. What is caching, and what are the common caching strategies (cache-aside, write-through, write-back)?</summary>

Caching stores frequently accessed data in fast storage (memory) to reduce load on slower backing stores and improve latency. **Cache-aside (lazy loading)** — application checks cache first, on a miss reads from DB and populates cache. **Write-through** — writes go to the cache and DB simultaneously, keeping them always in sync at the cost of write latency. **Write-back (write-behind)** — writes go to the cache first and are asynchronously flushed to the DB later, faster writes but risk of data loss if the cache fails before flushing.
</details>

<details>
<summary>8. What is cache invalidation, and why is it considered one of the hardest problems in computer science?</summary>

Cache invalidation is the process of removing or updating stale cached data when the underlying source changes. It's hard because you need to balance staleness (serving outdated data) against overhead (invalidating/refreshing too aggressively negates the caching benefit), and correctly tracking all the places a piece of data might be cached across a distributed system is genuinely difficult to get right consistently.
</details>

<details>
<summary>9. What is a CDN (Content Delivery Network), and what problem does it solve?</summary>

A geographically distributed network of proxy servers that cache and serve content (static assets, video, sometimes dynamic content) from a location physically closer to the end user — reduces latency, offloads traffic from origin servers, and improves resilience against origin outages/traffic spikes.
</details>

<details>
<summary>10. What is the CAP theorem?</summary>

States that a distributed data store can only guarantee **two** of three properties simultaneously during a network partition: **Consistency** (every read receives the most recent write or an error), **Availability** (every request receives a non-error response, without guaranteeing it's the most recent data), and **Partition tolerance** (the system continues operating despite network failures between nodes). Since network partitions are unavoidable in real distributed systems, the practical choice is really between consistency and availability during a partition (CP vs AP).
</details>

<details>
<summary>11. What is the difference between strong consistency and eventual consistency?</summary>

**Strong consistency** — after a write completes, every subsequent read (from any node) immediately reflects that write. **Eventual consistency** — after a write, reads might temporarily return stale data, but the system guarantees that if no new writes occur, all replicas will eventually converge to the same value. Eventual consistency trades immediate correctness for higher availability and lower latency, appropriate for many use cases (social media likes/views) but not others (financial balances).
</details>

<details>
<summary>12. What is the PACELC theorem, and how does it extend CAP?</summary>

PACELC extends CAP by addressing behavior even when there's **no** partition: "if Partitioned, choose between Availability and Consistency; Else (normal operation), choose between Latency and Consistency." It highlights that even without a network partition, there's still a fundamental trade-off between consistency and latency in distributed systems, which CAP alone doesn't capture.
</details>

<details>
<summary>13. What is database sharding, and what are common sharding strategies?</summary>

Sharding splits a large dataset across multiple database instances (shards), each holding a subset of the data, to scale beyond what a single database server can handle. Strategies: **Range-based** (shard by a value range, e.g., user IDs 1-1M on shard A), **Hash-based** (shard by a hash of the key, spreading data more evenly), **Geographic/directory-based** (shard by region or a lookup table mapping keys to shards).
</details>

<details>
<summary>14. What are the challenges introduced by database sharding?</summary>

Cross-shard queries/joins become expensive or impossible without application-level aggregation; rebalancing data when adding/removing shards is complex and can require significant data movement; maintaining referential integrity/transactions across shards is much harder; and "hot shards" (uneven load distribution) can still occur if the sharding key isn't chosen carefully.
</details>

<details>
<summary>15. What is consistent hashing, and what problem does it solve over simple modulo-based hashing?</summary>

Simple modulo hashing (`hash(key) % N`) requires remapping almost all keys when the number of nodes `N` changes, causing massive data movement. Consistent hashing arranges nodes and keys on a conceptual ring, so adding/removing a node only affects the keys immediately adjacent to it on the ring — dramatically reducing the amount of data that needs to move during scaling events.
</details>

<details>
<summary>16. What is a virtual node in consistent hashing, and why is it used?</summary>

Instead of placing each physical node once on the hash ring, each physical node is represented by many virtual nodes spread across the ring — this improves load distribution evenness (avoiding one physical node getting a disproportionately large or small range) and makes rebalancing smoother when nodes are added/removed.
</details>

<details>
<summary>17. What is database replication, and what's the difference between master-slave (primary-replica) and master-master (multi-primary) replication?</summary>

Replication maintains copies of data across multiple database nodes for redundancy and read scalability. **Primary-replica** — all writes go to a single primary, which propagates changes to read-only replicas; simple, avoids write conflicts, but the primary is a single point of failure/bottleneck for writes. **Multi-primary** — multiple nodes can accept writes, offering better write availability/geographic distribution, but requires conflict resolution when the same data is written differently on different primaries concurrently.
</details>

<details>
<summary>18. What is the difference between synchronous and asynchronous replication?</summary>

**Synchronous** — a write is only acknowledged as successful once it's confirmed on the replica(s) too, guaranteeing replicas are always up to date but adding write latency and reducing availability if a replica is slow/unreachable. **Asynchronous** — a write is acknowledged immediately after the primary commits, with replication happening in the background; faster writes but replicas can lag, and a primary failure before replication completes can lose the most recent writes.
</details>

<details>
<summary>19. What is a message queue, and what problems does it solve in a distributed system?</summary>

A message queue enables asynchronous communication between services — a producer publishes a message without waiting for a consumer to process it, decoupling the two in time and failure domain. Solves problems like: smoothing traffic spikes (buffering bursts for consumers to process at their own pace), decoupling service availability (a consumer being temporarily down doesn't block the producer), and enabling reliable retry/at-least-once delivery semantics.
</details>

<details>
<summary>20. What is the difference between a message queue (point-to-point) and a pub/sub (publish-subscribe) system?</summary>

**Point-to-point (queue)** — each message is consumed by exactly **one** consumer (even if multiple consumers are listening, only one gets each message — good for work distribution/load balancing across workers). **Pub/sub** — each message is delivered to **all** subscribers of a topic (good for broadcasting events to multiple independent interested parties).
</details>

<details>
<summary>21. What is the difference between at-most-once, at-least-once, and exactly-once delivery semantics?</summary>

**At-most-once** — a message might be lost but is never duplicated (fire-and-forget). **At-least-once** — a message is guaranteed to be delivered but might be delivered more than once (requires idempotent consumers to handle duplicates safely). **Exactly-once** — guaranteed delivered exactly one time; the hardest to achieve in a distributed system and often implemented as effectively "at-least-once delivery + idempotent processing" rather than a true distributed exactly-once guarantee.
</details>

<details>
<summary>22. What is idempotency, and why is it critical for reliable distributed system design?</summary>

An idempotent operation produces the same result regardless of how many times it's performed. Critical because network failures/timeouts make retries unavoidable in distributed systems — if an operation isn't idempotent, a retry after an ambiguous failure (did the first attempt actually succeed?) risks duplicate side effects (e.g., double-charging a customer).
</details>

<details>
<summary>23. What is a distributed transaction, and why are they generally avoided in microservices architectures?</summary>

A distributed transaction attempts to atomically commit changes across multiple independent databases/services (traditionally via two-phase commit). Avoided in microservices because it requires tight coupling and synchronous coordination between services (undermining independent deployability/availability), doesn't scale well, and a single slow/unavailable participant blocks the entire transaction — the Saga pattern is generally preferred instead.
</details>

<details>
<summary>24. What is the Saga pattern, and what's the difference between choreography-based and orchestration-based sagas?</summary>

A Saga breaks a distributed transaction into a sequence of local transactions, each with a compensating action to undo it if a later step fails, achieving eventual consistency instead of atomicity. **Choreography** — each service publishes events that trigger the next service's local transaction, no central coordinator (more decoupled, but harder to track overall saga state). **Orchestration** — a central orchestrator explicitly directs each step and handles failures/compensations (easier to understand/monitor, but introduces a central coordinating component).
</details>

<details>
<summary>25. What is a two-phase commit (2PC) protocol, and what are its drawbacks?</summary>

A coordinator asks all participants to "prepare" (vote to commit or abort) in phase one, then instructs all participants to actually commit (or abort) in phase two based on the votes. Drawbacks: it's a **blocking** protocol — if the coordinator crashes after phase one, participants are stuck holding locks indefinitely waiting for a decision; it also doesn't tolerate network partitions well, conflicting with the goals of highly available distributed systems.
</details>

<details>
<summary>26. What is eventual consistency's practical impact on user-facing system design, and how do you communicate/handle it gracefully?</summary>

Users might briefly see stale data after performing an action (e.g., a "like" count not immediately updating everywhere). Handled gracefully via: optimistic UI updates (show the expected change immediately client-side while the backend catches up), read-your-own-writes consistency guarantees (a user always sees their own recent changes even if other users temporarily see stale data), or simply setting appropriate user expectations where the domain tolerates it.
</details>

<details>
<summary>27. What is the difference between a monolithic architecture and a microservices architecture, and what are the trade-offs?</summary>

A **monolith** is a single deployable codebase/unit containing all functionality — simpler to develop, test, and deploy initially, with easy in-process communication, but scaling requires scaling the whole thing together, and it can become hard to maintain/deploy independently as it grows ("big ball of mud" risk). **Microservices** split functionality into independently deployable services — better scalability, team autonomy, and fault isolation, at the cost of operational complexity (network calls, distributed data consistency, service discovery, monitoring across many services).
</details>

<details>
<summary>28. What is an API Gateway, and what responsibilities does it typically handle in a microservices architecture?</summary>

A single entry point that sits in front of multiple backend microservices, handling cross-cutting concerns: request routing, authentication/authorization, rate limiting, request/response transformation, load balancing, and sometimes aggregating responses from multiple services into one client-facing response — sparing individual services from re-implementing these concerns themselves.
</details>

<details>
<summary>29. What is service discovery, and why is it necessary in a dynamic microservices environment?</summary>

In environments where service instances are frequently created/destroyed/rescheduled (auto-scaling, container orchestration), hardcoded IP addresses/hostnames don't work — service discovery (via a registry like Eureka/Consul, or platform-native DNS in Kubernetes) lets services find each other's *current* network location dynamically.
</details>

<details>
<summary>30. What is the difference between client-side service discovery and server-side service discovery?</summary>

**Client-side** — the calling service itself queries the service registry and picks a healthy instance (client needs registry-awareness logic, e.g., Netflix Eureka + Ribbon). **Server-side** — the client makes a request to a well-known load balancer/router (e.g., a Kubernetes Service or an AWS ALB), which internally queries the registry and routes to a healthy instance — the client stays simple and unaware of the discovery mechanism.
</details>

<details>
<summary>31. What is a circuit breaker pattern, and why is it important in a distributed system with many service-to-service calls?</summary>

Prevents a failing/slow downstream service from being repeatedly called (wasting resources, worsening cascading failure) by "opening" the circuit after a failure threshold — subsequent calls fail immediately without even attempting the downstream call, until a cooldown period allows a test ("half-open") call to check if the downstream service has recovered.
</details>

<details>
<summary>32. What is the bulkhead pattern in distributed system resilience design?</summary>

Named after ship compartmentalization — isolates resources (thread pools, connection pools) allocated to different downstream dependencies, so a failure/slowdown in one dependency (exhausting its dedicated resource pool) doesn't starve resources needed to call other, healthy dependencies — containing the "blast radius" of a single dependency's failure.
</details>

<details>
<summary>33. What is rate limiting, and what are common algorithms for implementing it (token bucket, leaky bucket, sliding window)?</summary>

Rate limiting restricts how many requests a client can make in a given time period, protecting a system from overload/abuse. **Token bucket** — tokens are added to a bucket at a fixed rate; a request consumes a token, and is rejected if the bucket is empty (allows bursts up to the bucket size). **Leaky bucket** — requests are processed at a fixed, steady output rate regardless of input burstiness (smooths traffic). **Sliding window** — counts requests within a moving time window, more accurate than fixed windows at preventing burst-at-boundary abuse.
</details>

<details>
<summary>34. What is the difference between horizontal partitioning and vertical partitioning of a database?</summary>

**Horizontal partitioning (sharding)** — splits rows of a table across multiple databases/servers (each shard has the same schema, a subset of rows). **Vertical partitioning** — splits columns/tables across different databases based on functional area (e.g., user profile data in one DB, order data in another) — often naturally emerges from splitting a monolith into microservices, each owning its own subset of the overall data model.
</details>

<details>
<summary>35. What is denormalization, and why might you deliberately denormalize data in a large-scale system despite the data-integrity risks?</summary>

Denormalization duplicates data across tables/documents to avoid expensive joins at read time, trading storage space and write-time consistency effort for significantly faster, simpler reads — commonly done in high-read-throughput systems where join performance at scale becomes a bottleneck, accepting the added complexity of keeping duplicated data in sync on writes.
</details>

<details>
<summary>36. What is the difference between SQL and NoSQL databases, and when would you choose one over the other?</summary>

**SQL (relational)** — structured schema, strong ACID transactional guarantees, powerful joins/querying — good fit for data with complex relationships and strong consistency requirements (financial systems, order management). **NoSQL** — flexible/schema-less (or schema-light), designed for horizontal scalability and high throughput, often trading some consistency/query flexibility for that scale — good fit for very large-scale, simpler-access-pattern data (session storage, activity feeds, IoT time-series data).
</details>

<details>
<summary>37. What are the main categories of NoSQL databases, and give an example use case for each?</summary>

**Key-value** (Redis, DynamoDB) — simple, extremely fast lookups by key, e.g., session storage/caching. **Document** (MongoDB) — flexible, nested JSON-like documents, e.g., a content management system with varying content structures. **Column-family** (Cassandra, HBase) — optimized for very wide tables and high write throughput, e.g., time-series/sensor data. **Graph** (Neo4j) — optimized for traversing relationships, e.g., social networks/recommendation graphs.
</details>

<details>
<summary>38. What is a write-ahead log (WAL), and why is it fundamental to database durability?</summary>

Before a database applies a change to its main data structures, it first writes a record of that intended change to an append-only log on durable storage — if the database crashes before the actual data structure update completes, it can replay the WAL on recovery to ensure no committed write is lost, providing durability without requiring every single change to be immediately, expensively flushed to its final storage location.
</details>

<details>
<summary>39. What is database indexing, and what's the trade-off of adding more indexes to a table?</summary>

An index is a data structure (commonly a B-tree) that lets the database find rows matching a query condition much faster than scanning the entire table. Trade-off: indexes speed up reads but slow down writes (every insert/update/delete must also update all relevant indexes) and consume additional storage — indexes should be added deliberately based on actual query patterns, not indiscriminately.
</details>

<details>
<summary>40. What is a B-tree, and why is it the standard data structure for database indexes?</summary>

A self-balancing tree structure that keeps data sorted and allows searches, insertions, and deletions in logarithmic time, while being optimized for systems that read/write large blocks of data (like disk pages) — its shallow, wide structure minimizes the number of disk reads needed to find a record, which matters far more for disk-backed indexes than pure in-memory algorithmic complexity would suggest.
</details>

<details>
<summary>41. What is the difference between optimistic and pessimistic concurrency control in distributed/database systems?</summary>

**Pessimistic** — acquires a lock before accessing a resource, blocking other transactions from touching it until released; safer under high contention but reduces concurrency. **Optimistic** — assumes conflicts are rare, allows concurrent access, and checks for conflicts only at commit time (e.g., via a version number), retrying/failing if a conflict is detected; better throughput under low contention, but can waste work under high contention (repeated conflict-driven retries).
</details>

<details>
<summary>42. What is a distributed lock, and what challenges arise when implementing one correctly?</summary>

A mechanism ensuring only one process/node across a distributed system can hold a lock on a resource at a time (e.g., to prevent two instances of a scheduled job from running concurrently). Challenges: handling lock holder failure/crash (locks need a timeout/lease so a crashed holder doesn't block forever), avoiding split-brain scenarios (network partition causing two nodes to both believe they hold the lock), and clock skew/timing issues across distributed nodes making lease-expiry timing unreliable in edge cases.
</details>

<details>
<summary>43. What is Redlock, and what controversy surrounds it as a distributed locking algorithm?</summary>

Redlock is an algorithm (proposed for Redis) that acquires a lock across a majority of independent Redis instances to tolerate individual node failures. It's controversial because critics (notably Martin Kleppmann) argue it doesn't provide the strong safety guarantees it claims under certain failure/timing scenarios (clock drift, process pauses), making it potentially unsafe for use cases requiring genuinely strong mutual exclusion guarantees, even though it's often "good enough" for less critical use cases like reducing (not strictly preventing) duplicate work.
</details>

<details>
<summary>44. What is a consensus algorithm, and why do distributed systems need one? Name two examples.</summary>

Consensus algorithms let a group of distributed nodes agree on a single value/decision even in the presence of failures or network issues — essential for maintaining consistency in replicated systems (e.g., electing a leader, agreeing on the order of operations). Examples: **Paxos** (foundational but notoriously difficult to understand/implement correctly), **Raft** (designed specifically to be more understandable, widely used in systems like etcd, Consul).
</details>

<details>
<summary>45. What is leader election, and why is it needed in distributed systems?</summary>

The process of designating one node among a group as the "leader" responsible for coordinating certain operations (e.g., accepting writes in a primary-replica setup, or coordinating a distributed job) — needed to avoid conflicting/uncoordinated actions from multiple nodes, while still allowing the system to automatically recover (elect a new leader) if the current leader fails.
</details>

<details>
<summary>46. What is a heartbeat mechanism, and how is it used for failure detection in distributed systems?</summary>

Nodes periodically send small "I'm still alive" signals to each other (or to a central coordinator); if a node's heartbeat isn't received within an expected timeframe, it's presumed failed/unreachable and appropriate recovery action is taken (e.g., triggering leader re-election, removing it from a load balancer's healthy pool).
</details>

<details>
<summary>47. What is the difference between a stateful and stateless service, and why is statelessness generally preferred for horizontal scalability?</summary>

A **stateless** service doesn't retain client-specific data between requests (any instance can handle any request) — trivially scalable horizontally since load balancers can route requests to any available instance without concern for prior interaction history. A **stateful** service retains session/context between requests, requiring either sticky sessions (routing a client consistently to the same instance) or externalizing that state (e.g., to a shared cache/database) to scale horizontally without breaking functionality.
</details>

<details>
<summary>48. What is a good back-of-the-envelope approach for estimating storage requirements in a system design interview?</summary>

Estimate: (average size of one record/item) × (expected number of records, based on stated scale/growth assumptions) × (replication factor, if relevant) — then sanity-check the resulting order of magnitude (megabytes vs terabytes vs petabytes) to inform decisions like whether a single database can handle it or sharding/distributed storage is needed.
</details>

<details>
<summary>49. What is a good back-of-the-envelope approach for estimating required server/bandwidth capacity?</summary>

Estimate requests per second (RPS) from stated daily active users and average actions per user (accounting for peak traffic being significantly higher than average — often applying a peak-to-average multiplier), then combine with average request/response payload size to estimate bandwidth, and use typical single-server capacity assumptions (e.g., "a server can handle roughly X requests/sec for this workload type") to estimate the number of servers needed.
</details>

<details>
<summary>50. What is the purpose of asking clarifying questions at the start of a system design interview, and what's a good example set of questions for "design a URL shortener"?</summary>

Demonstrates structured thinking and ensures you're solving the actual problem being asked rather than making unfounded assumptions, while also surfacing the non-functional requirements that drive the design. For a URL shortener: expected scale (URLs created/day, read:write ratio — typically very read-heavy), whether custom aliases are needed, whether analytics/click-tracking is required, expected URL lifetime/expiration behavior, and whether it needs to support very high availability (versus tolerating brief downtime).
</details>

<details>
<summary>51. How would you design a URL shortener (like bit.ly) at a high level?</summary>

Core flow: a write request generates a short, unique key (e.g., base62-encoded auto-incrementing ID, or a hash with collision handling) mapped to the long URL, stored in a key-value store (fast lookups by key fit naturally). Reads (redirects) are by far the dominant traffic — heavily cache the key→URL mapping (CDN/edge cache or Redis) since redirects need to be extremely fast and the data is essentially immutable once created. A `301`/`302` redirect response is returned; analytics (click counts) can be tracked asynchronously (e.g., via a message queue) to avoid slowing down the critical redirect path.
</details>

<details>
<summary>52. In a URL shortener design, how would you generate unique short keys, and what are the trade-offs between different approaches?</summary>

**Auto-incrementing counter + base62 encoding** — simple, guaranteed unique, but requires a centralized counter (potential bottleneck/single point of coordination) unless partitioned across ranges per server. **Hashing the long URL (e.g., MD5, truncated)** — no central coordination needed, but requires collision detection/handling (two different URLs hashing to the same short key). **Pre-generated key pool** — a background service generates and stores a pool of available unique keys ahead of time, workers just pop one off — avoids both coordination bottlenecks and collision handling at request time.
</details>

<details>
<summary>53. How would you design a rate limiter as a standalone system (e.g., for an API gateway)?</summary>

Store request counts (or tokens, in a token-bucket approach) per client key in a fast, shared data store (Redis, using atomic increment operations) so the rate limit state is consistent across multiple gateway instances rather than each instance tracking its own independent, inconsistent count. Choose an algorithm (sliding window log/counter for accuracy, token bucket for burst tolerance) based on requirements, and return a `429 Too Many Requests` with a `Retry-After` header when the limit is exceeded.
</details>

<details>
<summary>54. How would you design a distributed unique ID generator (like Twitter's Snowflake)?</summary>

Generate IDs that are unique, roughly time-sortable, and don't require central coordination for every ID — Snowflake-style IDs typically combine: a timestamp (bits for sortability/uniqueness over time), a machine/worker ID (bits identifying which node generated it, avoiding collisions across nodes), and a per-millisecond sequence number (bits allowing multiple unique IDs from the same node within the same millisecond) — packed into a single 64-bit integer, generated locally on each node without needing a round trip to a central service.
</details>

<details>
<summary>55. How would you design a news feed / social media timeline system (like Twitter/Instagram's feed)?</summary>

Two main approaches: **Fan-out on write (push model)** — when a user posts, the post is immediately pushed/written into all their followers' pre-computed feed lists; fast reads (feed is pre-assembled), but expensive/slow writes for users with millions of followers ("celebrity problem"). **Fan-out on read (pull model)** — a feed is assembled dynamically at read time by querying and merging posts from everyone the user follows; cheap writes, but potentially slow/expensive reads, especially for users following many people. Most real systems use a **hybrid**: push model for typical users, pull model (or a special-cased hybrid) for celebrity/high-follower accounts to avoid the fan-out write explosion.
</details>

<details>
<summary>56. How would you design a chat/messaging system (like WhatsApp) at a high level?</summary>

Real-time delivery typically uses persistent connections (WebSockets, or long-polling as a fallback) rather than repeated HTTP polling, with connection state tracked so the system knows which server a given user is currently connected to (often via a lookup service, since a user might connect to any of many gateway servers). Messages are persisted (for offline delivery/history) and delivered directly if the recipient is online, or queued for delivery when they reconnect. For group chats, fan-out logic delivers a single message to multiple recipients. End-to-end encryption, message ordering guarantees, and read-receipt/delivery-status tracking are additional layered concerns.
</details>

<details>
<summary>57. In a chat system design, how do you handle a user being connected to one of many server instances, and how does a message get routed to the right server?</summary>

Maintain a mapping (in a fast shared store like Redis) of `userId → serverId` (which specific server instance currently holds that user's active WebSocket connection) — when a message needs to be delivered, the system looks up the recipient's current server and routes the message there (often via an internal pub/sub mechanism between server instances) rather than assuming the sending server and receiving server are the same instance.
</details>

<details>
<summary>58. How would you design a notification system that needs to send push notifications, emails, and SMS at scale?</summary>

A central notification service exposes an API/consumes events describing "notify user X about Y"; it looks up the user's preferences/contact info and enqueues delivery tasks to channel-specific queues (push/email/SMS), each consumed by dedicated workers integrating with the relevant third-party provider (APNs/FCM for push, an email service, an SMS gateway) — decoupling the triggering event from actual delivery, allowing independent scaling/retry logic per channel, and providing a natural point to implement rate limiting/deduplication/user preference filtering before any actual send occurs.
</details>

<details>
<summary>59. How would you design a system to handle deduplication of notifications (avoiding sending the same notification to a user multiple times)?</summary>

Generate a deterministic idempotency key for each logical notification (e.g., hash of user ID + event type + relevant entity ID), and check/record it in a fast store (Redis with a TTL) before sending — if the key was already recorded recently, skip sending; this handles both legitimate retries (a failed delivery attempt being retried) and duplicate-event scenarios (the same underlying business event being published more than once upstream).
</details>

<details>
<summary>60. How would you design a distributed file storage system (like Google Drive/Dropbox) at a high level?</summary>

Separate metadata (file names, folder structure, permissions, version history — stored in a relational/NoSQL database optimized for that access pattern) from actual file content (stored in a blob store like S3 or a distributed file system, chunked into blocks for efficient large-file handling and deduplication). Uploads/downloads typically go through a dedicated storage service (not the main application server) to avoid bottlenecking it with large binary transfers. Synchronization across a user's multiple devices requires tracking version/change history and resolving conflicts (e.g., a "last write wins" or explicit conflict file creation strategy) when the same file is edited offline on multiple devices.
</details>

<details>
<summary>61. How would you design a video streaming platform (like YouTube/Netflix) at a high level?</summary>

Uploaded video is processed asynchronously (transcoded into multiple resolutions/bitrates and formats) via a pipeline of background workers, then the resulting video segments are stored and distributed via a CDN close to viewers. Streaming uses adaptive bitrate protocols (HLS/DASH) that let a client's player dynamically switch quality based on current network conditions, requesting small segments rather than the whole file at once. Metadata (titles, descriptions, view counts, recommendations) is served from a separate, much smaller and simpler metadata service/database, decoupled from the massive-scale video-serving infrastructure itself.
</details>

<details>
<summary>62. In a video streaming design, why is adaptive bitrate streaming (HLS/DASH) preferred over simply serving one fixed-quality video file?</summary>

Network conditions vary significantly across users and even change during a single viewing session — adaptive bitrate breaks video into short segments encoded at multiple quality levels, letting the player seamlessly switch quality up/down as available bandwidth changes, minimizing buffering/stalls compared to a fixed-quality stream that either wastes bandwidth (too high quality for a poor connection) or looks worse than necessary (too low quality for a good connection).
</details>

<details>
<summary>63. How would you design a ride-sharing system's core matching functionality (like Uber matching riders to nearby drivers)?</summary>

Requires efficient **geospatial querying** — driver locations are continuously updated (via periodic location pings) into a data structure optimized for "find nearby entities" queries, commonly using geohashing (encoding lat/long into a string where nearby locations share string prefixes) or a quad-tree, rather than naively scanning/calculating distance for every driver on every request. When a ride is requested, the system queries for available drivers within an expanding radius, applies matching logic (ETA, driver rating, acceptance likelihood), and offers the ride — with a timeout/re-offer mechanism if a driver doesn't respond promptly.
</details>

<details>
<summary>64. What is geohashing, and why is it useful for location-based system design?</summary>

Geohashing encodes a latitude/longitude pair into a single string, where the encoding has the useful property that geographically nearby locations tend to share a common string prefix — this lets you index and query locations using standard string-prefix-based database indexes/range queries, rather than needing specialized 2D spatial indexing structures, making "find things near this point" queries much simpler to implement on top of common database technology.
</details>

<details>
<summary>65. How would you design a distributed cache (like Redis or Memcached) at a high level?</summary>

Keys are distributed across multiple cache nodes (typically via consistent hashing, to minimize redistribution when nodes are added/removed). Clients (or a proxy layer) need to know how to route a given key to the correct node. Handle node failure by either simply treating a cache miss as acceptable (re-fetch from the source of truth, since it's "just a cache") or by replicating hot data across multiple nodes for higher availability. Eviction policies (LRU, LFU, TTL-based expiration) manage limited memory by deciding what to remove when the cache is full.
</details>

<details>
<summary>66. What is the difference between LRU, LFU, and FIFO cache eviction policies?</summary>

**LRU (Least Recently Used)** — evicts the item that hasn't been accessed for the longest time; good general-purpose default, assumes recently-accessed items are likely to be accessed again soon. **LFU (Least Frequently Used)** — evicts the item with the fewest total accesses; better for workloads with a stable set of consistently "hot" items, but can struggle with items that were frequently accessed long ago but aren't relevant anymore. **FIFO** — evicts the oldest-inserted item regardless of access pattern; simplest, but ignores actual usage patterns entirely.
</details>

<details>
<summary>67. How would you design a search autocomplete/typeahead system (like Google Search suggestions)?</summary>

Precompute and store popular search terms/prefixes in a **trie** data structure (or an inverted-index-based approach), allowing very fast prefix-based lookups; results are typically ranked by historical query frequency, and the whole structure is periodically rebuilt/updated offline from aggregated query logs rather than updated synchronously on every single search, since perfect real-time freshness usually isn't required for this feature. Heavy caching (the same popular prefixes are queried constantly) and serving the top-K results from edge/CDN locations further reduces latency for this famously latency-sensitive feature.
</details>

<details>
<summary>68. How would you design a web crawler at a high level?</summary>

A frontier/queue of URLs to visit (seeded initially, then continuously replenished with newly discovered links); multiple distributed worker processes pull URLs from the frontier, fetch the page, extract links (adding new ones to the frontier) and content (sent for indexing/storage). Key concerns: politeness (rate-limiting requests per domain to avoid overwhelming any single site), avoiding duplicate crawling (a "seen URL" filter, often a Bloom filter for memory efficiency at scale), respecting `robots.txt`, and prioritization (crawling high-value/frequently-changing pages more often than static, low-value ones).
</details>

<details>
<summary>69. What is a Bloom filter, and why is it useful in large-scale distributed system design (e.g., for a web crawler's "seen URLs" check)?</summary>

A Bloom filter is a space-efficient probabilistic data structure that can tell you "definitely not in the set" or "possibly in the set" (with a tunable false-positive rate, but never a false negative) using far less memory than storing the actual full set of items — ideal for a "have I seen this URL/key before" check at massive scale, where occasionally re-crawling a small percentage of already-seen URLs (a false positive) is an acceptable trade-off for the enormous memory savings versus storing every URL exactly.
</details>

<details>
<summary>70. How would you design a distributed logging/monitoring system that aggregates logs from thousands of servers?</summary>

Lightweight agents on each server ship logs (often buffered/batched) to a centralized ingestion pipeline (commonly via a message queue like Kafka to absorb bursty log volume and decouple ingestion rate from processing rate), which feeds into a storage/indexing system optimized for log search (like Elasticsearch) and a visualization layer (Kibana/Grafana). Given the enormous volume, sampling, log-level filtering at the source, and tiered storage (hot/recent logs on fast storage, older logs archived to cheaper cold storage) are important cost/performance considerations.
</details>

<details>
<summary>71. How would you design a payment processing system, and what are the key correctness concerns?</summary>

Correctness (especially idempotency and avoiding double-charging/double-crediting) is paramount above raw performance for this domain. Key elements: an idempotency-key-based API design so retried requests (due to network timeouts) don't create duplicate charges; a ledger-based data model (recording immutable transaction entries rather than just mutating a balance field directly) for auditability and reconciliation; integration with external payment gateways/processors via well-defined state machines (pending → authorized → captured → settled, with failure/refund paths) since payment operations are inherently asynchronous and can fail at multiple stages; and strong consistency guarantees around actual money movement, even if other parts of the broader system tolerate eventual consistency.
</details>

<details>
<summary>72. In a payment system, why is a ledger-based (append-only, double-entry-style) data model often preferred over simply storing and updating a user's "balance" field directly?</summary>

An append-only ledger of individual transactions provides a complete, auditable history that can be independently verified/reconstructed and reconciled against external systems (crucial for financial correctness and regulatory compliance) — directly mutating a single balance field loses this audit trail, makes concurrent-update race conditions harder to reason about safely, and makes it much harder to detect/investigate discrepancies after the fact.
</details>

<details>
<summary>73. How would you design an e-commerce inventory/stock management system that must avoid overselling a limited-stock item during a flash sale?</summary>

Requires careful concurrency control around stock decrement operations — options include: pessimistic locking on the inventory row during checkout (simple, but can create contention/bottleneck under very high concurrent demand for the same item), or an atomic conditional decrement (e.g., `UPDATE inventory SET stock = stock - 1 WHERE stock > 0`, checking the affected row count to know if it actually succeeded) which avoids holding a lock for the duration of the whole checkout flow. For extremely high-contention flash-sale scenarios, some systems use a pre-allocated, in-memory (Redis-backed) counter/queue system that serializes access to a hot inventory count far faster than a traditional database could under that specific contention pattern.
</details>

<details>
<summary>74. How would you design a distributed job scheduler (like a distributed cron)?</summary>

A central scheduler (itself needs to be highly available, often via leader election among multiple scheduler instances to avoid a single point of failure) maintains job definitions and their schedules, and at the appropriate time, dispatches job execution requests to a pool of worker nodes (via a queue) — needs to handle: preventing duplicate execution of the same scheduled job instance (idempotency/locking), tracking job execution status/retries, and graceful handling of a worker crashing mid-job (detecting the failure and re-dispatching, ideally to a different worker).
</details>

<details>
<summary>75. How would you design a distributed counter that needs to handle extremely high write throughput (e.g., counting video views)?</summary>

A single database row being incremented by millions of concurrent writers becomes a severe contention bottleneck. Common approaches: **sharded counters** (split the count across multiple independent counter rows/keys, summed together only when reading the total — writes distribute across shards, reducing contention on any single row), or **approximate/batched counting** (buffer increments locally/in-memory and periodically flush aggregated batches to the persistent store rather than writing on every single event), trading perfect real-time accuracy for dramatically higher achievable write throughput.
</details>

<details>
<summary>76. How would you design a system to detect and prevent duplicate/fraudulent transactions in real time?</summary>

Combines rule-based checks (velocity limits — too many transactions too quickly from one account/device, geographic impossibility — a transaction from two distant locations within an implausibly short time) with machine-learning-based scoring models evaluated in real time (or near-real time) against a request, often requiring low-latency access to recent historical behavior data (via a fast feature store/cache) to make a timely allow/flag/block decision without meaningfully slowing down the legitimate transaction flow.
</details>

<details>
<summary>77. How would you design a distributed rate-limited API for a third-party developer platform with different rate limit tiers per customer?</summary>

Store rate limit configuration (tier/limits) per API key/customer, and track current usage counters in a shared, fast store (Redis) keyed by customer + time window, checked/incremented atomically on every request at the API gateway layer before the request is allowed to proceed to backend services — with clear, standardized rate-limit-related response headers (`X-RateLimit-Remaining`, `Retry-After`) so well-behaved API consumers can self-regulate their request patterns.
</details>

<details>
<summary>78. How would you design a leaderboard system (e.g., a real-time gaming leaderboard with millions of players)?</summary>

A **sorted set** data structure (like Redis's `ZSET`) is a natural fit — provides efficient insertion/update of a player's score and efficient retrieval of "top N" or "rank of player X" queries, both essential leaderboard operations, without needing to sort the entire dataset on every query. For very large scale, sharding the leaderboard (e.g., regional leaderboards aggregated periodically into a global one) can help distribute load, accepting some staleness in the fully global view.
</details>

<details>
<summary>79. How would you design a distributed configuration management system (like what powers feature flags across many services)?</summary>

A central configuration store (often with a simple key-value model) that services can query/subscribe to for configuration changes, propagated either via polling (services periodically re-check) or push notifications (a watch/streaming mechanism, e.g., via etcd/Consul's watch API or a pub/sub channel) so configuration changes take effect promptly across a fleet without requiring a redeploy — with local caching at each service instance so a temporary configuration-store outage doesn't immediately break every dependent service (falling back to last-known-good configuration).
</details>

<details>
<summary>80. How would you design a system for handling large-scale, real-time analytics (like a dashboard showing live metrics across millions of events per second)?</summary>

Raw events are ingested via a high-throughput streaming platform (Kafka), processed by a stream-processing engine (Flink/Kafka Streams/Spark Streaming) that computes aggregations (counts, sums, averages) over sliding/tumbling time windows in near-real time, writing results to a fast-read time-series or analytical database optimized for the dashboard's query patterns — deliberately trading perfect precision (some approximate aggregation techniques, like HyperLogLog for unique counts, are common at this scale) for the throughput and low latency required for a genuinely "live" dashboard experience.
</details>

<details>
<summary>81. What is HyperLogLog, and why is it useful for large-scale unique-count estimation (e.g., "how many unique visitors today")?</summary>

A probabilistic algorithm that estimates the cardinality (count of distinct elements) of a very large set using a small, fixed amount of memory (a few kilobytes, regardless of whether the actual set has thousands or billions of unique elements), with a small, well-understood margin of error — dramatically more memory-efficient than exactly tracking every unique element (e.g., in a hash set) when you only need an approximate count, which is sufficient for most analytics dashboard use cases.
</details>

<details>
<summary>82. How would you approach designing a system for a "trending topics" feature (like Twitter Trends)?</summary>

Requires tracking approximate frequency counts of terms/hashtags over a recent sliding time window (not all-time, since "trending" implies recent surge, not just overall popularity) — typically implemented with a streaming aggregation pipeline computing counts per time bucket, combined with a scoring algorithm that weighs recent velocity/acceleration of mentions (a sudden spike) more heavily than steady-but-unremarkable volume, to surface genuinely "trending" (rapidly rising) topics rather than just perpetually popular ones.
</details>

<details>
<summary>83. How would you design a distributed system to ensure exactly-once processing of financial transactions even if a service crashes mid-processing?</summary>

Combine idempotency keys (so a retried/duplicate request has no additional effect) with transactional outbox patterns (ensuring a state change and any resulting side-effect event are committed atomically together within a single database transaction) and careful use of database-level unique constraints as a final safety net (e.g., a unique constraint on a transaction reference ID preventing accidental duplicate insertion even if application-level idempotency logic somehow fails).
</details>

<details>
<summary>84. What is the transactional outbox pattern, and what problem does it solve?</summary>

Solves the "dual write problem" — when a service needs to both update its own database AND publish an event to a message broker as part of one logical operation, doing these as two separate, non-atomic actions risks inconsistency if a crash happens between them (DB updated but event never published, or vice versa). The outbox pattern writes the event as a row in an "outbox" table within the **same database transaction** as the main business data change, and a separate background process reliably reads from the outbox table and publishes to the broker — guaranteeing the event is eventually published if and only if the original transaction actually committed.
</details>

<details>
<summary>85. How would you design a system for handling large file uploads (e.g., multi-gigabyte video uploads) reliably over an unreliable network?</summary>

Use **chunked/multipart upload** — the large file is split into smaller chunks uploaded independently (potentially in parallel), each chunk acknowledged individually, so a network interruption only requires re-uploading the failed chunk(s) rather than restarting the entire upload from scratch. The server tracks which chunks have been received and reassembles the complete file once all chunks arrive, with typical support for resuming an interrupted upload later using a persisted upload session ID.
</details>

<details>
<summary>86. How would you design a system to handle "eventually consistent" search indexing (e.g., a product catalog whose Elasticsearch index needs to reflect database changes)?</summary>

Rather than synchronously updating the search index on every single database write (coupling write latency/availability to the search infrastructure's health), changes are propagated asynchronously — either via Change Data Capture (CDC) tailing the database's transaction log, or by publishing an explicit "entity changed" event from the application after committing the primary database write — consumed by an indexing service that updates the search index, accepting a brief window of index staleness in exchange for decoupling and resilience.
</details>

<details>
<summary>87. What is Change Data Capture (CDC), and what are its advantages over application-level dual writes for keeping downstream systems in sync?</summary>

CDC captures changes directly from a database's transaction log (e.g., via Debezium reading MySQL's binlog or Postgres's WAL) rather than requiring the application code to explicitly publish an event alongside every write — this guarantees every actual database change is captured (even ones made outside the normal application code path, like a manual data fix) without relying on the application remembering to publish an event correctly every single time, avoiding the dual-write consistency problem entirely at the source.
</details>

<details>
<summary>88. How would you design a global system that needs to serve users with low latency across multiple continents while maintaining data consistency?</summary>

Deploy regional data centers/points of presence close to users for low-latency reads (often via a CDN or regional read replicas), while carefully deciding which data genuinely needs strong global consistency (typically routed to a single "source of truth" region or using a globally consistent database like Google Spanner) versus data that can tolerate eventual consistency/regional independence (often the vast majority of read-heavy content) — this fundamental trade-off between global consistency and low-latency multi-region access is usually the central design tension in truly global system architecture.
</details>

<details>
<summary>89. What is multi-region active-active architecture, and what challenges does it introduce?</summary>

Multiple geographically distributed regions all actively accept both reads and writes (rather than one primary region handling writes with others as read-only replicas) — improves latency (users write to their nearest region) and availability (no single region is a write bottleneck/single point of failure), but introduces significant complexity around conflict resolution (the same data being written differently in two regions concurrently) and requires careful choice of a conflict resolution strategy (last-write-wins, CRDTs, or application-specific merge logic) since true synchronous cross-region consistency would reintroduce the very latency problem multi-region deployment was meant to solve.
</details>

<details>
<summary>90. What is a CRDT (Conflict-free Replicated Data Type), and why is it relevant to multi-region/offline-capable system design?</summary>

A data structure specifically designed so that concurrent, independent updates made on different replicas (without coordination) can always be merged together deterministically into a consistent final result, without conflicts — useful for multi-region active-active systems or offline-first applications where you can't rely on coordinating every write through a single authority, common examples include grow-only counters, and specialized set/map structures with well-defined merge semantics.
</details>

<details>
<summary>91. How would you design a system to handle graceful degradation during a partial outage (e.g., a recommendation service is down, but the core shopping flow should still work)?</summary>

Design non-critical features (recommendations, "customers also bought") to fail gracefully and independently — wrapped in circuit breakers/timeouts with a sensible fallback (show no recommendations, or a cached/generic default, rather than failing the entire page load) — ensuring a failure in a non-essential dependency degrades the experience gracefully rather than cascading into a complete outage of the core, business-critical functionality (browsing/checkout).
</details>

<details>
<summary>92. What is the difference between designing for high availability and designing for disaster recovery, and what metrics (RTO, RPO) are relevant to the latter?</summary>

**High availability** focuses on minimizing downtime during normal operational failures (a single server/component failing) via redundancy within a system's normal operating environment. **Disaster recovery** addresses much larger-scale failure scenarios (an entire data center/region becoming unavailable) via separate backup infrastructure/procedures. **RTO (Recovery Time Objective)** — the maximum acceptable time to restore service after a disaster. **RPO (Recovery Point Objective)** — the maximum acceptable amount of data loss, measured in time (e.g., "we can lose at most 5 minutes of data") — both drive decisions about backup frequency and failover architecture.
</details>

<details>
<summary>93. What is the difference between a hot standby, warm standby, and cold standby disaster recovery strategy?</summary>

**Hot standby** — a fully running, continuously synchronized replica ready to take over near-instantly (lowest RTO, highest cost, since you're running duplicate infrastructure continuously). **Warm standby** — a scaled-down but running replica that needs to be scaled up before fully taking over (moderate RTO/cost trade-off). **Cold standby** — infrastructure/backups exist but aren't actively running, requiring provisioning and data restoration before becoming operational (highest RTO, lowest ongoing cost).
</details>

<details>
<summary>94. How would you design an autocomplete/search-as-you-type system to keep latency very low (sub-100ms) even at large scale?</summary>

Precompute and cache popular prefix results aggressively (since the same short prefixes are queried extremely frequently across all users), serve from geographically distributed edge caches/CDN close to the user to minimize network round-trip time, and keep the actual lookup data structure (trie or similar) small and memory-resident (avoiding disk I/O) — every millisecond matters disproportionately for this specific kind of highly latency-sensitive, interactive feature compared to typical page-load-oriented requests.
</details>

<details>
<summary>95. How would you design a system to handle "at most one" processing for scheduled jobs across multiple redundant scheduler instances (avoiding the same job running twice)?</summary>

Use a distributed lock (with an appropriate expiry/lease) acquired by whichever scheduler instance attempts to trigger a given scheduled job — only the instance that successfully acquires the lock actually proceeds to launch the job, and the others back off, ensuring redundant scheduler instances (needed for scheduler-level high availability) don't inadvertently cause duplicate job execution.
</details>

<details>
<summary>96. What is the difference between horizontal and vertical database partitioning combined with read replicas, in terms of solving different scaling bottlenecks?</summary>

**Read replicas** address read-throughput bottlenecks (many more reads than a single database instance can handle) by distributing read queries across multiple copies of the full dataset. **Sharding (horizontal partitioning)** addresses write-throughput and total-data-volume bottlenecks (a single instance can't handle the total write load or storage size) by splitting the dataset itself across multiple independent database instances — these solve genuinely different bottlenecks and are often combined (each shard itself might also have its own read replicas).
</details>

<details>
<summary>97. What is the "thundering herd" problem, and how might it manifest in a cache-based system design?</summary>

Occurs when a popular cached item expires (or the cache itself restarts/is flushed), and a large number of concurrent requests all simultaneously experience a cache miss and hit the backing database/service at once, potentially overwhelming it — mitigated via techniques like request coalescing (only one request actually goes to the backend, others wait for and share that result), staggered/jittered cache expiration times (avoiding many keys expiring at exactly the same moment), or proactive cache refresh before actual expiration.
</details>

<details>
<summary>98. What is cache stampede protection, and name a specific technique to implement it.</summary>

Prevents the thundering herd scenario specifically at the individual-key level — a common technique is a **mutex/lock on cache-miss**: when a key expires and multiple concurrent requests miss the cache, only the first one acquires a short-lived lock and actually queries the backend/recomputes the value (repopulating the cache), while other concurrent requests either wait briefly for that result or serve a slightly stale cached value in the meantime, rather than all of them redundantly hitting the backend simultaneously.
</details>

<details>
<summary>99. What is the difference between horizontal scaling of stateless application servers versus horizontal scaling of a stateful database, in terms of relative difficulty?</summary>

Scaling stateless application servers horizontally is comparatively straightforward (just add more identical instances behind a load balancer, since there's no data/state to coordinate between them). Scaling a stateful database horizontally is fundamentally harder because data itself must be partitioned/replicated and kept consistent across nodes, introducing all the complexity of sharding, replication lag, and distributed consistency trade-offs that stateless scaling simply doesn't face.
</details>

<details>
<summary>100. What is a good general framework/structure for approaching any system design interview question from start to finish?</summary>

1) Clarify functional and non-functional requirements (scope, scale, consistency/availability needs). 2) Estimate scale (back-of-envelope calculations for storage/traffic). 3) Define the high-level API/interface the system exposes. 4) Sketch the high-level architecture/major components and data flow. 5) Deep-dive into the most interesting/critical components (often where the interviewer wants to probe further — data model, a specific algorithm, a scaling bottleneck). 6) Discuss trade-offs explicitly (why this choice over alternatives), bottlenecks, and how the design evolves/scales further, and address failure modes/edge cases.
</details>

<details>
<summary>101. Why is it important to explicitly discuss trade-offs during a system design interview rather than just presenting one "correct" design?</summary>

Real system design almost always involves genuine trade-offs (consistency vs availability, cost vs performance, simplicity vs flexibility) with no single universally "correct" answer — interviewers are typically evaluating whether a candidate understands and can articulate *why* a particular choice was made and what was given up, demonstrating engineering judgment, rather than just reciting a memorized "correct" architecture for a well-known problem.
</details>

<details>
<summary>102. How would you design a system to handle "seen/unseen" state tracking for millions of users across a large content feed (e.g., "mark all as read")?</summary>

Rather than storing an explicit record for every single (user, item) pair ever seen (which grows unboundedly and expensively), a common approach stores just a single "last seen timestamp/ID" per user — any content with a creation timestamp/ID before that marker is considered "seen," and "mark all as read" simply updates that one marker value, dramatically reducing storage compared to tracking every individual item's read state explicitly.
</details>

<details>
<summary>103. How would you design a distributed system to compute and serve personalized recommendations at scale?</summary>

Split into an offline batch component (periodically, e.g., nightly, computing recommendation models/candidate lists using historical behavioral data at scale — batch/ML infrastructure) and an online serving component (a low-latency service that retrieves precomputed candidates for a given user and applies lightweight real-time re-ranking/filtering based on immediate context) — this separation lets the computationally expensive model training/candidate generation happen without latency constraints, while the actual user-facing request path stays fast by just looking up and lightly adjusting precomputed results.
</details>

<details>
<summary>104. What is the difference between a batch-computed recommendation approach and a real-time/online learning approach, and what are the trade-offs?</summary>

**Batch-computed** — recommendations are precomputed periodically (e.g., nightly) from historical data; simpler and more scalable to compute, but can feel stale (doesn't immediately reflect a user's most recent actions). **Real-time/online** — incorporates very recent user behavior into recommendations with minimal delay; more responsive/relevant, but significantly more complex infrastructure (streaming feature computation, online model updates) and higher operational cost.
</details>

<details>
<summary>105. How would you design a system for handling large-scale, idempotent bulk data import (e.g., a customer uploading a CSV of 1 million records to be processed)?</summary>

Accept the upload and immediately return an acknowledgment with a tracking ID (don't process synchronously within the request-response cycle for something this large), then process asynchronously via a background job (potentially a Spring Batch-style chunked pipeline) that validates, transforms, and loads records — using per-record idempotency keys or upsert semantics so that if the import needs to be retried/resumed after a partial failure, it doesn't create duplicates; provide a status-check endpoint/notification so the client can track progress and eventual completion.
</details>

<details>
<summary>106. What is backpressure in the context of system design, and why is it important when connecting components with different processing speeds?</summary>

Backpressure is a mechanism for a slower downstream consumer to signal to a faster upstream producer to slow down, preventing the consumer from being overwhelmed (unbounded queue growth, memory exhaustion, or dropped data) — essential whenever components in a pipeline have significantly different throughput capacities, whether implemented via reactive streams' built-in backpressure protocol, a bounded queue that blocks the producer when full, or explicit rate-limiting of the producer based on consumer feedback.
</details>

<details>
<summary>107. How would you design a system to safely roll out a risky change (e.g., a new pricing algorithm) to a small percentage of production traffic before a full rollout?</summary>

Implement via feature flagging/canary release infrastructure that can route a configurable percentage of traffic (often based on a consistent hash of user ID, so a given user consistently gets the same experience across requests rather than randomly flip-flopping) to the new logic, combined with careful monitoring/comparison of key metrics between the control and treatment groups before deciding to expand rollout — allowing early detection of problems while limiting the "blast radius" of a potential issue to a small fraction of overall traffic/users.
</details>

<details>
<summary>108. What is the difference between A/B testing infrastructure and a simple feature flag system, and how might they overlap in a system design?</summary>

A simple feature flag is a binary/percentage-based on-off switch for a feature, often used for operational purposes (safe rollout, kill switch). A/B testing infrastructure builds on similar underlying routing mechanics but adds rigorous statistical experiment design (proper randomization, sample size/significance calculation, metric tracking tied to specific experiment variants) specifically to measure the causal impact of a change on business metrics — the underlying traffic-splitting mechanism is often shared/overlapping infrastructure, but the purpose and rigor of analysis differ significantly.
</details>

<details>
<summary>109. How would you design a system to handle time-zone-aware scheduling for a global user base (e.g., "send this notification at 9am local time for each user")?</summary>

Store the user's timezone (or infer/update it based on their activity/location), and store scheduling logic in terms of that user's local time rather than a single global UTC time — the scheduling system needs to periodically evaluate "which users currently have a local time matching their configured send time" across all timezones, which is a meaningfully different (and more complex) query pattern than a single global fixed-UTC-time cron trigger.
</details>

<details>
<summary>110. What is a common pitfall when storing and reasoning about dates/times in a globally distributed system, and how do you avoid it?</summary>

Storing timestamps in local/ambiguous time zones (rather than a canonical format like UTC) leads to subtle, hard-to-debug bugs around daylight saving time transitions, cross-timezone comparisons, and ambiguous/non-existent local times during DST changes — the standard mitigation is to always store and process timestamps internally in UTC, converting to a user's local timezone only at the final display/presentation layer.
</details>

<details>
<summary>111. How would you design a system to support "undo" functionality for a user action (e.g., undo send on an email, or undo delete)?</summary>

Rather than immediately, irreversibly performing the destructive action, introduce a brief delay window (e.g., queue the action for execution a few seconds later, or mark the item as "soft deleted"/pending rather than immediately hard-deleting it) during which the user can cancel — the actual destructive operation only truly executes after the undo window expires without cancellation, at which point it can proceed (or be permanently purged in the case of soft deletes).
</details>

<details>
<summary>112. What is the difference between a soft delete and a hard delete, and what are the trade-offs of each in system design?</summary>

**Soft delete** — marks a record as deleted (a flag/timestamp) without actually removing it from storage; supports undo, audit trails, and referential integrity for other data still referencing it, at the cost of needing to filter out soft-deleted records in every query and accumulating storage over time. **Hard delete** — actually removes the data permanently; simpler queries and reclaims storage, but loses recoverability/audit history and can complicate referential integrity if other records still reference the now-gone entity.
</details>

<details>
<summary>113. How would you design a system that needs to comply with data privacy regulations (like GDPR's "right to be forgotten") while still maintaining necessary audit trails/backups?</summary>

Requires careful data modeling distinguishing genuinely erasable personal data from data that must be retained for legitimate legal/audit purposes (often anonymized/pseudonymized rather than fully retained in identifiable form), a documented, auditable deletion process (including propagating deletion requests to backups/archives and any downstream systems/replicas that also hold copies of the data), and clear data retention policies established upfront — this is as much a data-governance/legal design problem as a technical one, requiring genuine cross-functional design collaboration, not just an engineering afterthought.
</details>

<details>
<summary>114. How would you design a search system that needs to support full-text search with relevance ranking (like a product search feature)?</summary>

Typically built on a dedicated search engine (Elasticsearch/Solr, both built on Lucene) rather than relying on a relational database's basic text-matching capabilities — data is indexed into an inverted index (mapping terms to the documents containing them) enabling fast full-text queries, with relevance scoring (commonly TF-IDF or BM25-based) ranking results by how well they match the query, further tunable with custom boosting rules (e.g., boosting in-stock items, or recency) layered on top of the base relevance algorithm.
</details>

<details>
<summary>115. What is an inverted index, and why is it fundamental to how full-text search engines work?</summary>

An inverted index maps each unique term/word to a list of documents (and often positions within those documents) that contain it — the inverse of the "natural" document-to-words mapping. This structure lets a search engine answer "which documents contain word X" almost instantly via a direct lookup, rather than scanning every document's full text on every single query, which is what makes fast full-text search over large document collections practically feasible.
</details>

<details>
<summary>116. How would you design a system to detect and mitigate a DDoS attack at a high level?</summary>

Layered defense: edge-level traffic filtering/absorption (CDN and DDoS-protection services that can absorb massive volumetric attacks before they even reach origin infrastructure), rate limiting and anomaly detection at the application/API gateway layer (identifying and blocking abnormal traffic patterns from specific sources), and architectural resilience (auto-scaling to absorb legitimate traffic spikes, circuit breakers to protect backend services if attack traffic does get through) — no single layer is sufficient alone; effective DDoS mitigation is inherently a defense-in-depth strategy.
</details>

<details>
<summary>117. What is the difference between designing a system for "read-heavy" versus "write-heavy" workloads, and how does that shift architectural priorities?</summary>

**Read-heavy** systems (most social media/content platforms) prioritize aggressive caching, read replicas, and denormalized read-optimized data models, often tolerating eventual consistency and more complex write-side logic in exchange for very fast, cheap reads. **Write-heavy** systems (analytics/logging/IoT ingestion) prioritize high write throughput (often via append-only log-structured storage, write-optimized databases, and horizontal write partitioning/sharding), sometimes accepting slower or more limited read query capability (e.g., only supporting specific pre-aggregated query patterns) in exchange for sustaining very high ingestion rates.
</details>

<details>
<summary>118. How would you design a distributed system to handle "exactly-once" delivery semantics using Kafka specifically?</summary>

Kafka supports idempotent producers (avoiding duplicate messages from producer-side retries via sequence numbers) and transactional writes (atomically writing to multiple partitions/topics, useful for read-process-write patterns like Kafka Streams), which together can achieve effectively-exactly-once semantics **within the Kafka ecosystem itself** — but true end-to-end exactly-once still requires the final consumer's side effect (e.g., a database write) to also be coordinated with the Kafka offset commit atomically (e.g., writing the offset and the result to the same database transaction) to avoid a gap where a crash after processing but before offset commit causes reprocessing.
</details>

<details>
<summary>119. What is the difference between designing a system around synchronous request-response APIs versus an event-driven architecture, and how do you decide which fits a given interaction?</summary>

Synchronous APIs fit interactions where the caller genuinely needs an immediate result to proceed (e.g., "is this payment authorized, yes or no, right now"). Event-driven architecture fits interactions that are naturally asynchronous or where decoupling matters more than immediate confirmation (e.g., "an order was placed" triggering independent downstream processes like inventory update, shipping notification, and analytics, none of which need to block the original order-placement response) — many real systems use a mix of both, choosing per-interaction based on this genuine need for an immediate synchronous response versus tolerance for asynchronous, eventual processing.
</details>

<details>
<summary>120. How would you design a system to safely handle schema evolution of events flowing through a message queue/event bus over time (e.g., adding a new field to an event type)?</summary>

Use a schema registry (like Confluent Schema Registry for Kafka/Avro) enforcing compatibility rules (e.g., backward compatibility — new schema versions can read data written with old schemas, forward compatibility — old consumers can read data written with new schemas) so producers and consumers can be deployed independently without breaking each other, and design event schemas defensively (new fields optional with sensible defaults, avoiding removing/renaming existing fields) to minimize the blast radius of schema changes across a distributed system with many independent producers/consumers.
</details>

<details>
<summary>121. What is the difference between "fat events" (containing full entity state) and "thin events" (containing just an ID/reference, requiring a follow-up query) in event-driven design, and what are the trade-offs?</summary>

**Fat events** carry the full relevant data in the event itself — consumers don't need to make an additional call back to the producer to get details, reducing coupling and latency, but events can become large and schema changes to the entity ripple more visibly through many consumers. **Thin events** just signal "something changed, here's the ID" — smaller, simpler events, but every consumer now needs to make a follow-up call to fetch current details (adding latency, load on the producer, and a race condition risk if the entity has changed again by the time the consumer fetches it).
</details>

<details>
<summary>122. How would you design a system to handle "read your own writes" consistency in an otherwise eventually-consistent, cache-heavy architecture?</summary>

Common approaches: route a user's own subsequent reads to the primary/source-of-truth database (bypassing read replicas/cache) for a short window after they perform a write, or explicitly update/invalidate the relevant cache entry synchronously as part of the write operation itself (rather than relying purely on lazy cache-aside population) — ensuring at minimum the acting user sees their own change immediately, even if propagation to other users' cached views is still eventually consistent.
</details>

<details>
<summary>123. How would you design a multi-tenant SaaS system's data architecture, and what are the trade-offs between shared-database and separate-database-per-tenant models?</summary>

**Shared database, shared schema** (a `tenant_id` column on shared tables) — simplest and most resource-efficient, but requires rigorous, consistently-enforced tenant isolation in every single query (a missed `WHERE tenant_id = ?` is a severe data-leak bug), and "noisy neighbor" tenants can affect others sharing the same database. **Separate database per tenant** — strongest isolation (both security and performance), simpler to reason about correctness, but far more operationally complex/costly to manage (schema migrations, backups, and monitoring multiplied across potentially thousands of tenant databases).
</details>

<details>
<summary>124. What is the "noisy neighbor" problem in a shared multi-tenant infrastructure, and how might you mitigate it at a system design level?</summary>

One tenant's unusually heavy usage (a large customer running expensive queries, or an unexpected traffic spike) degrades performance/availability for other tenants sharing the same underlying infrastructure. Mitigations: per-tenant rate limiting/resource quotas, dedicated resource pools for premium/large tenants (a tiered infrastructure model), and query/resource governance (timeouts, query complexity limits) to prevent any single tenant's workload from monopolizing shared resources.
</details>

<details>
<summary>125. How would you design a system's data model and architecture to support both real-time transactional operations (OLTP) and complex analytical reporting (OLAP) without one negatively impacting the other?</summary>

Avoid running heavy analytical queries directly against the primary transactional database (they compete for the same resources and can degrade transactional performance) — instead, replicate/ETL data (often via CDC or scheduled batch extraction) into a separate data warehouse/analytical store optimized for OLAP query patterns (often columnar storage, which is much more efficient for the wide aggregation queries typical of reporting compared to row-oriented OLTP storage), keeping the two workloads architecturally separated despite sourcing from the same underlying data.
</details>

<details>
<summary>126. What is the difference between row-oriented and column-oriented (columnar) database storage, and why does it matter for OLTP versus OLAP workloads?</summary>

**Row-oriented** storage keeps all columns of a single row physically together — efficient for OLTP workloads that typically read/write a small number of complete records at a time. **Column-oriented** storage keeps all values of a single column physically together across all rows — much more efficient for OLAP-style queries that aggregate over a few specific columns across millions of rows (e.g., `SUM(revenue) GROUP BY region`), since it only needs to read the relevant columns' data rather than scanning entire rows just to access a couple of fields.
</details>

<details>
<summary>127. How would you design a system to support "point-in-time" data recovery (restoring the system's exact state as of a specific past moment)?</summary>

Requires combining periodic full backups/snapshots with continuous incremental change logs (like a database's write-ahead log/binlog) retained for the desired recovery window — restoring to an arbitrary point in time involves loading the nearest preceding full snapshot and then replaying the incremental log up to (but not past) the exact target timestamp, rather than only being able to restore to the fixed times when full snapshots happened to be taken.
</details>

<details>
<summary>128. What is a good approach to designing a system's API versioning strategy to avoid breaking existing clients as the system evolves?</summary>

Favor backward-compatible, additive changes wherever possible (adding new optional fields rather than removing/renaming existing ones) to avoid needing a version bump at all for most changes; when a genuinely breaking change is unavoidable, introduce a new API version while continuing to support the previous version for a defined deprecation period, with clear communication/monitoring of which clients are still using the older version before it's eventually retired.
</details>

<details>
<summary>129. How would you design a system to gracefully handle a downstream dependency's degraded performance (not full failure, but elevated latency)?</summary>

Aggressive, well-tuned timeouts (so a slow dependency doesn't hold up resources/threads indefinitely) combined with circuit breakers that can trip based on latency degradation (not just outright failures), and fallback behavior (serving cached/default data) specifically for latency-based circuit trips — since a dependency that's technically "up" but very slow can actually be more dangerous to overall system health than one that's cleanly, quickly failing, because it ties up resources across the calling system for longer.
</details>

<details>
<summary>130. What is the significance of designing timeouts thoughtfully across a chain of service-to-service calls (e.g., Service A calls B calls C), rather than using the same timeout value everywhere?</summary>

If every service in a call chain uses the same timeout, a slow response from the deepest service (C) can cause each calling layer (B, then A) to also time out independently and unpredictably, without a coherent overall behavior — a better pattern budgets an overall end-to-end deadline and propagates a shrinking remaining-time budget down the call chain (each service's effective timeout accounts for how much of the overall budget remains), ensuring a coherent, bounded total response time regardless of how many hops are involved.
</details>

<details>
<summary>131. How would you design a system to test/validate a large-scale distributed system's resilience to real-world failure conditions before they occur in production?</summary>

**Chaos engineering** — deliberately, in a controlled way, injecting realistic failures (killing instances, introducing network latency/partitions, exhausting resources) into a system (ideally in a production-like or even actual production environment with appropriate safeguards) to proactively discover weaknesses in resilience mechanisms (circuit breakers, retries, failover) before they're exposed by genuine, unplanned incidents — tools like Chaos Monkey pioneered this practice.
</details>

<details>
<summary>132. What is the significance of designing and regularly testing a system's actual failover/disaster-recovery procedures, rather than just having the underlying infrastructure/architecture theoretically capable of failover?</summary>

Failover mechanisms that are never actually exercised often have subtle, undiscovered bugs or outdated runbooks/configuration that only surface during a real disaster — regularly, deliberately testing actual failover (not just architecture review) is the only reliable way to have genuine confidence the disaster recovery capability will actually work correctly under real, high-pressure incident conditions, rather than discovering it's broken precisely when you most need it to work.
</details>

<details>
<summary>133. How would you design a system's observability strategy (logs, metrics, traces) for a complex microservices architecture?</summary>

**Logs** — structured, correlated (via a shared trace/correlation ID across service boundaries) for detailed, ad-hoc debugging of specific requests. **Metrics** — aggregated numerical time-series (latency percentiles, error rates, throughput) for dashboards and alerting on overall system health trends. **Distributed tracing** — following a single request's full journey across multiple services, showing exactly where time is spent/where failures occur in a multi-hop call chain — the three pillars together (not any single one alone) are generally needed to effectively diagnose issues in a genuinely complex distributed system.
</details>

<details>
<summary>134. What is the significance of tracking latency percentiles (p50, p95, p99) rather than just average latency when monitoring system performance?</summary>

Average latency can mask significant tail latency problems — a system where most requests are fast but a meaningful percentage (say, 1%) are very slow will still show a deceptively good average, while p99 latency directly surfaces that tail experience; since at scale, even a "rare" 1% slow-request rate affects a very large absolute number of real users, tail latency often matters more to overall user experience/business impact than the average does.
</details>

<details>
<summary>135. How would you design a system to handle the "cascading failure" scenario where one overloaded service's slowness causes upstream services to also become overloaded and fail?</summary>

Combine circuit breakers (stopping calls to an already-struggling downstream service, giving it room to recover rather than piling on more load) with load shedding (deliberately rejecting a portion of incoming requests under extreme load, prioritizing serving some requests well over trying and failing to serve all of them) and bulkheads (isolating resource pools so one dependency's overload doesn't exhaust resources needed for calls to healthy dependencies) — cascading failures are fundamentally a resource-exhaustion/feedback-loop problem, and breaking that feedback loop early (rather than letting overload propagate) is the core defensive principle.
</details>

<details>
<summary>136. What is load shedding, and when might a system deliberately choose to reject some requests rather than trying to serve all of them?</summary>

Load shedding deliberately drops or rejects a portion of incoming requests (often the lowest-priority ones, via some prioritization scheme) once the system is approaching overload, in order to preserve enough capacity to reliably serve the remaining requests well — the alternative (trying to serve every single request even under severe overload) often results in the whole system degrading so badly that it effectively fails to serve *any* request acceptably, making deliberate, prioritized shedding a better outcome than uncontrolled collapse.
</details>

<details>
<summary>137. How would you design a system's capacity planning process to anticipate and prepare for a known, large traffic event (e.g., a major sale event like Black Friday)?</summary>

Combine historical traffic analysis/trend extrapolation with deliberate, realistic load testing at the anticipated peak scale well ahead of the actual event, identify and address bottlenecks discovered during load testing, ensure auto-scaling policies/limits are configured appropriately for the expected scale (not just relying on default scaling limits that might be insufficient), and establish a clear incident-response/escalation plan specifically for the event window given the higher stakes and reduced tolerance for extended downtime during a known critical traffic period.
</details>

<details>
<summary>138. What is the significance of designing a system with clear, well-defined service boundaries and ownership, especially as an organization scales?</summary>

Beyond pure technical architecture, service boundaries in a microservices system often map to (and should be deliberately designed around) team boundaries and areas of business ownership (a pattern often summarized as "Conway's Law" — system design tends to mirror organizational communication structure) — poorly aligned boundaries (e.g., a single service requiring coordination across multiple teams for every change) create organizational friction and slow delivery, independent of whether the technical architecture itself is otherwise sound.
</details>

<details>
<summary>139. What is Conway's Law, and why is it relevant to system/microservices architecture design decisions?</summary>

States that organizations design systems that mirror their own communication structure. Relevant because it suggests architectural boundaries (which services exist, how they're divided) should often be deliberately aligned with team boundaries (each service owned end-to-end by one team) to minimize cross-team coordination overhead for typical changes — ignoring this and designing "ideal" technical boundaries that cut across team ownership lines often leads to persistent organizational friction regardless of the technical architecture's merits.
</details>

<details>
<summary>140. How would you approach explaining and justifying a specific technology choice (e.g., "why Kafka over RabbitMQ" or "why Cassandra over Postgres") during a system design interview?</summary>

Ground the justification in the specific requirements of the problem at hand (throughput needs, message ordering/replay requirements, consistency needs, team familiarity/operational maturity with the technology) rather than a generic "X is better than Y" claim — e.g., Kafka's log-based, replayable model and very high throughput fits event-streaming/analytics use cases particularly well, while RabbitMQ's more traditional queue semantics and flexible routing might fit simpler task-queue use cases more naturally; a good answer explicitly ties the choice back to the specific requirements rather than treating it as a universal ranking.
</details>

<details>
<summary>141. What is the significance of explicitly identifying and calling out a system design's potential single points of failure, and proposing mitigations for the most critical ones?</summary>

No real system design is entirely free of single points of failure in some form (even a "fully redundant" system has a shared underlying dependency somewhere) — a strong system design answer proactively identifies the most consequential ones (e.g., "our leader-election coordinator is itself a SPOF unless we also make it redundant") and reasons about whether the residual risk is acceptable given the system's actual availability requirements, rather than either ignoring the question or claiming an unrealistic zero-single-point-of-failure design.
</details>

<details>
<summary>142. How would you design a system's approach to database schema migrations in a live, zero-downtime production environment?</summary>

Use expand-contract (a.k.a. parallel-change) migration strategy — first "expand" the schema additively (add a new column/table alongside the old one, without removing anything), deploy application code that can work with both old and new schema simultaneously, backfill/migrate data, then only once fully transitioned, "contract" by removing the old, now-unused schema elements in a later, separate deployment — avoiding a single atomic cutover that would otherwise require simultaneous, coordinated deployment of schema and application code (which is fragile and hard to roll back safely).
</details>

<details>
<summary>143. What is the significance of designing a system to be able to roll back a bad deployment quickly and safely, and what architectural choices support this?</summary>

Fast, safe rollback capability (rather than only forward-fixing) significantly reduces the blast radius/duration of a bad deployment's impact — supported architecturally by: backward-compatible database migrations (so an older application version can still run correctly against a newer schema during rollback), maintaining previous container images/deployment artifacts readily available for quick redeployment, and feature-flag-gating risky new logic (allowing an instant flag-flip rollback without a full redeploy at all) for particularly risky changes.
</details>

<details>
<summary>144. How would you design a system's approach to handling and communicating planned maintenance windows for a service with strict uptime SLAs?</summary>

Favor architecture that minimizes or eliminates the need for maintenance-related downtime in the first place (rolling deployments, online schema migrations, redundant infrastructure allowing one node to be taken down for maintenance while others continue serving traffic) — when brief unavoidable downtime genuinely can't be avoided (e.g., some hardware-level infrastructure maintenance), clear advance communication to affected stakeholders/customers and choosing genuinely lowest-impact timing windows are the key operational (not just technical) considerations.
</details>

<details>
<summary>145. What is the significance of designing a system's data model with future extensibility in mind, without over-engineering for speculative requirements that may never materialize (YAGNI)?</summary>

There's a genuine tension between designing rigidly around only today's exact known requirements (risking painful, expensive rework when requirements inevitably evolve) and over-engineering excessive flexibility for speculative future needs that may never actually materialize (adding real complexity/cost now for a benefit that might never be realized) — experienced designers generally favor building in flexibility specifically at points of genuine, well-reasoned uncertainty (e.g., "we don't yet know if we'll need multi-currency support, but the cost of designing for it now is low and the cost of retrofitting it later is high") while resisting speculative complexity elsewhere.
</details>

<details>
<summary>146. How would you design a system to support gradual, safe database migration from one database technology to another (e.g., migrating from a monolithic Postgres instance to a sharded architecture) with minimal downtime?</summary>

A common pattern: dual-write (write to both old and new systems simultaneously for a transition period, with careful handling of the dual-write consistency problem discussed earlier), backfill historical data into the new system, run comprehensive validation/comparison between the two systems to build confidence in correctness, then gradually shift read traffic to the new system (often via a feature flag/percentage rollout), and only fully decommission the old system once the new one has been proven reliable under full production load for a sufficient period.
</details>

<details>
<summary>147. What is the significance of clearly articulating assumptions explicitly during a system design interview, rather than silently making them?</summary>

Explicitly stating assumptions ("I'm assuming reads significantly outnumber writes here, roughly 100:1, which is why I'm prioritizing read-side caching") demonstrates structured reasoning and gives the interviewer a clear opportunity to correct a wrong assumption early, before it derails significant subsequent design work built on a faulty premise — silently assuming things risks either building on an incorrect foundation the interviewer never gets a chance to redirect, or appearing to have jumped to a design without clear underlying reasoning.
</details>

<details>
<summary>148. How would you approach a system design question that feels deliberately ambiguous or underspecified (e.g., simply "design Twitter")?</summary>

Treat the ambiguity as an invitation (not an obstacle) to demonstrate structured requirement-gathering — proactively ask clarifying questions to scope the problem to a tractable, well-defined subset (e.g., "should I focus on the core tweet/timeline functionality, or also cover features like direct messages and trending topics?") rather than either freezing up or unilaterally guessing at scope without checking, since real-world system design work virtually always starts from similarly ambiguous, open-ended business requirements that need to be actively scoped and clarified.
</details>

<details>
<summary>149. What is a good way to close out a system design interview response, after walking through the core design?</summary>

Briefly summarize the key trade-offs made and why, proactively mention known limitations or areas that would need further design work given more time/information, and where relevant, briefly discuss how the design might evolve at 10x or 100x the originally discussed scale — demonstrating awareness that the current design is a reasoned point-in-time solution for stated requirements, not a claim of universal perfection.
</details>

<details>
<summary>150. If asked "how would this whole design change if we suddenly needed to support 100x the current scale," what kind of answer demonstrates strong systems thinking?</summary>

A strong answer identifies which specific components of the current design would actually become the bottleneck first at 100x scale (rather than vaguely claiming "we'd need more servers everywhere") — e.g., "our single primary database would become a write bottleneck well before the application servers do, so at that scale we'd need to introduce sharding, and our current synchronous cross-service calls would likely need to shift toward more asynchronous, event-driven patterns to avoid cascading latency amplification across the now much-larger call volume" — demonstrating genuine understanding of where the actual scaling limits of the specific proposed design lie, rather than generic hand-waving about "scaling out."
</details>
