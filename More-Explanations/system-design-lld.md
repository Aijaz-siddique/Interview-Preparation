# System Design — Low-Level Design (LLD) — Interview Questions (Experienced)

<details>
<summary>1. What is Low-Level Design, and how does an LLD interview differ from an HLD interview?</summary>

LLD focuses on translating a set of requirements into concrete class structures, interfaces, relationships, and interactions — object-oriented design, design patterns, database schema at the entity level, and detailed API contracts. Where HLD interviews evaluate architectural/scaling judgment across whole systems, LLD interviews evaluate object-oriented design skill, applying SOLID principles, and structuring maintainable, extensible code for a well-scoped feature/system component.
</details>

<details>
<summary>2. What is the general approach/framework for tackling an LLD interview question (e.g., "design a parking lot system")?</summary>

1) Clarify functional requirements and scope. 2) Identify the core entities/objects and their responsibilities (nouns in the requirements often hint at classes). 3) Define relationships between entities (inheritance, composition, association) and draw a rough class diagram. 4) Identify key interfaces/abstractions and where design patterns naturally fit. 5) Walk through the main use cases/flows against your design to validate it. 6) Discuss extensibility — how would the design change if a new requirement were added.
</details>

<details>
<summary>3. What is the Single Responsibility Principle, and how would you identify a violation of it in a class diagram?</summary>

A class should have only one reason to change — one well-defined responsibility. A violation often shows up as a class with an unrelated grab-bag of methods (e.g., a `User` class that also handles password hashing, email sending, and report generation) — a good heuristic is asking "what would cause this class to need to change?" and checking if there's more than one plausible, unrelated answer.
</details>

<details>
<summary>4. How would you design a Parking Lot system? What are the core entities?</summary>

Core entities: `ParkingLot` (manages overall state, floors), `ParkingFloor`, `ParkingSpot` (with a type — compact/large/handicapped/motorcycle — and occupied status), `Vehicle` (with a type, possibly subclassed as `Car`/`Motorcycle`/`Truck`), `Ticket` (issued on entry, tracks entry time/spot), and a `ParkingSpotAssignmentStrategy` (an interface allowing different spot-assignment algorithms — nearest-available, by vehicle-type matching — to be swapped without changing the core `ParkingLot` logic).
</details>

<details>
<summary>5. In a Parking Lot design, how would you handle different vehicle types requiring different spot sizes?</summary>

Model spot types and vehicle types as an enum or class hierarchy, and encode compatibility rules (a motorcycle can park in any spot type, a car needs a compact-or-larger spot, a truck needs a large spot) either via a compatibility-checking method on `ParkingSpot` (`canFit(Vehicle vehicle)`) or a dedicated strategy object — favoring composition/strategy over a large hardcoded `if/else` chain scattered through the allocation logic.
</details>

<details>
<summary>6. How would you design an Elevator System? What are the core entities and key design challenges?</summary>

Core entities: `Elevator` (current floor, direction, state — idle/moving up/moving down), `ElevatorController`/`ElevatorSystem` (manages multiple elevators, dispatches requests), `Request` (represents a floor call, either an external hall call or an internal cabin button press), and a `SchedulingStrategy` (an interface for the algorithm deciding which elevator responds to a given request). Key design challenges: efficiently choosing which elevator to dispatch for a new request (nearest idle elevator, or one already moving in the right direction), and handling the elevator's internal request queue (processing floor requests in an efficient order rather than strictly FIFO).
</details>

<details>
<summary>7. In an Elevator System design, what design pattern fits naturally for handling the "which elevator responds to this request" logic, and why?</summary>

The **Strategy pattern** — encapsulating the elevator-selection algorithm (nearest elevator, least busy elevator, zone-based assignment) behind a common interface lets you swap or add new dispatch algorithms without modifying the core `ElevatorController` logic, and makes each strategy independently testable.
</details>

<details>
<summary>8. How would you design a Library Management System? What are the core entities?</summary>

Core entities: `Book` (catalog-level info — title, author, ISBN), `BookItem` (a specific physical/lendable copy of a `Book`, since a library has multiple copies), `Member`/`Patron`, `Librarian`, `Loan`/`BookLending` (tracks which `BookItem` is loaned to which `Member`, due date), `Reservation` (for holding a currently-unavailable book), and a `Catalog`/`Library` aggregate root coordinating search and lending operations.
</details>

<details>
<summary>9. In a Library Management System, why would you model `Book` and `BookItem` as separate classes rather than one?</summary>

`Book` represents catalog-level metadata shared across all copies (title, author, ISBN) — this data shouldn't be duplicated per physical copy. `BookItem` represents an individual, distinctly trackable physical copy (with its own barcode/condition/availability status) — separating them avoids data duplication/inconsistency (updating a book's title once, rather than on every copy) and correctly models the real-world one-to-many relationship between a catalog entry and its physical copies.
</details>

<details>
<summary>10. How would you design a Vending Machine system? What state-related design pattern fits well?</summary>

Core entities: `VendingMachine`, `Product`/`Item` (with price and inventory count), `Inventory`, and a set of **states** (`IdleState`, `HasMoneyState`, `DispensingState`, `OutOfStockState`) implementing a common `VendingMachineState` interface — the **State pattern** fits naturally here, since the machine's valid operations and transitions genuinely depend on its current state (e.g., you can't select a product before inserting money), and encoding this as explicit state classes (rather than a tangle of boolean flags and conditionals) keeps each state's behavior/transitions clean and independently understandable.
</details>

<details>
<summary>11. What is the State design pattern, and what problem does it solve?</summary>

Lets an object alter its behavior when its internal state changes, appearing as if the object changed its class — implemented by encapsulating state-specific behavior into separate state classes implementing a common interface, with the context object delegating to its current state object rather than using large conditional blocks checking "what state am I in" scattered throughout its methods.
</details>

<details>
<summary>12. How would you design a Movie Ticket Booking System (like BookMyShow)? What is the trickiest concurrency-related design challenge?</summary>

Core entities: `Movie`, `Theater`, `Show` (a specific movie at a specific theater/time), `Seat`, `Booking`. The trickiest challenge is preventing **double-booking** of the same seat by concurrent users — typically solved with either pessimistic locking (lock the seat row during the booking transaction) or an optimistic approach with a temporary "seat hold/lock" mechanism (reserving a seat for a short window, e.g., 5-10 minutes, while a user completes payment, using a version check or a distributed lock to prevent two users from simultaneously holding the same seat).
</details>

<details>
<summary>13. In a Movie Ticket Booking System, how would you design the seat-locking mechanism to avoid seats being held indefinitely by an abandoned booking attempt?</summary>

Attach a time-to-live (TTL) to the seat hold (e.g., stored with an expiry timestamp, or in a store like Redis with a native TTL/expiration feature) — if the user doesn't complete payment within the window, the hold automatically expires and the seat becomes available again for other users, without needing an explicit cleanup action from the abandoning user's session.
</details>

<details>
<summary>14. How would you design a Splitwise-like expense-sharing application? What are the core entities and key algorithmic challenge?</summary>

Core entities: `User`, `Group`, `Expense` (amount, who paid, how it's split among participants — equally, by percentage, or by exact amounts), and a `Balance`/`Ledger` tracking net amounts owed between pairs of users. The key algorithmic challenge is **debt simplification** — minimizing the total number of individual transactions needed to settle all debts within a group (e.g., if A owes B and B owes C the same amount, simplify directly to A owing C), typically solved via a greedy algorithm that repeatedly matches the largest creditor with the largest debtor.
</details>

<details>
<summary>15. In a Splitwise-like system, how would you model different expense-splitting strategies (equal, percentage, exact amount)?</summary>

Use the **Strategy pattern** — define a `SplitStrategy` interface with a method like `calculateSplits(amount, participants)`, and implement `EqualSplitStrategy`, `PercentageSplitStrategy`, `ExactAmountSplitStrategy` — the `Expense` class holds a reference to whichever strategy was chosen at creation time, keeping the splitting-calculation logic cleanly separated and independently extensible (adding a new split type doesn't require modifying existing `Expense` logic).
</details>

<details>
<summary>16. How would you design a Chess Game (or a similar board game)? What are the core entities and design patterns that fit well?</summary>

Core entities: `Board`, `Piece` (an abstract base class, with `King`, `Queen`, `Rook`, `Bishop`, `Knight`, `Pawn` subclasses each implementing their own `getValidMoves()` logic — a natural fit for **polymorphism/Template Method**), `Player`, `Move`, and a `Game`/`GameController` coordinating turns and win/check/checkmate detection. The **Command pattern** fits well for representing moves (enabling undo/redo and move-history tracking), and the **Strategy pattern** can encapsulate different AI difficulty levels if computer opponents are part of the scope.
</details>

<details>
<summary>17. In a Chess Game design, how would you implement "undo move" functionality cleanly?</summary>

Model each move as a `Command` object capturing enough information to both execute and reverse it (the piece moved, its origin/destination, and any captured piece) — maintain a stack (history) of executed `Move` commands; undo simply pops the most recent command and calls its `undo()` method, which restores the board to its exact prior state, including restoring any captured piece.
</details>

<details>
<summary>18. How would you design a Tic-Tac-Toe game with support for different board sizes and win conditions?</summary>

Core entities: `Board` (a configurable N×N grid), `Player`, `Symbol`/`Mark`, and a `WinningStrategy` interface (allowing different win-condition logic — standard N-in-a-row, or a custom variant — to be plugged in without changing the core game loop) — designing the board size and win-length as configurable parameters from the start (rather than hardcoding a 3x3 grid with a hardcoded "3 in a row" check) demonstrates anticipating a common LLD interview follow-up ("now make it work for a 5x5 board with 4-in-a-row to win").
</details>

<details>
<summary>19. How would you design a Logging Framework (like a simplified version of Log4j)? What design patterns are commonly used?</summary>

Core entities: `Logger` (the interface applications interact with), `LogLevel` (enum: DEBUG, INFO, WARN, ERROR), `Appender`/`LogHandler` (an interface for different output destinations — console, file, remote service — a natural fit for the **Strategy pattern**, letting a `Logger` be configured with one or more appenders). The **Chain of Responsibility** pattern fits well for level-based filtering — each handler in a chain checks if it should process a given log level, passing it along if not. The **Singleton** pattern is often used for a global `LoggerFactory`/`LogManager`.
</details>

<details>
<summary>20. How would you design a Notification System (supporting Email, SMS, and Push notifications) with extensibility for adding new channels later?</summary>

Define a `NotificationChannel` interface with a `send(Notification)` method, implemented by `EmailNotification`, `SmsNotification`, `PushNotification` — a **Factory** (or `NotificationFactory`) creates the appropriate channel implementation based on the requested type, and a `NotificationService` orchestrates sending (potentially to multiple channels for one logical notification, via the **Composite** pattern, or the **Decorator** pattern if you need to layer cross-cutting behavior like retry/logging around any channel implementation).
</details>

<details>
<summary>21. What is the Factory Method pattern, and how does it differ from simply using `new` directly, in the context of a Notification System design?</summary>

Factory Method encapsulates the logic of *which concrete class to instantiate* behind a method, so calling code depends only on the abstract `NotificationChannel` interface, not concrete implementation classes — this means adding a new channel (e.g., WhatsApp) later only requires adding a new implementation and updating the factory, without touching any of the calling code that already depends on the abstraction, honoring the Open/Closed Principle.
</details>

<details>
<summary>22. How would you design a Cache (like an LRU Cache) from scratch, including the data structures needed for O(1) get and put operations?</summary>

Combine a **HashMap** (for O(1) key lookup) with a **doubly linked list** (maintaining usage order — most-recently-used at one end, least-recently-used at the other). The HashMap maps keys directly to their corresponding linked-list node, so on a `get()`, you can locate and move a node to the "most recently used" end in O(1) (no traversal needed), and on `put()` when the cache is full, you evict the node at the "least recently used" end in O(1).
</details>

<details>
<summary>23. In an LRU Cache design, why is a plain `LinkedList` combined with a `HashMap` insufficient for true O(1) operations, and what's needed instead?</summary>

A standard `LinkedList`'s `remove(Object)` is O(n) because it must traverse to find the node first — true O(1) removal/reordering requires the HashMap to store direct references to the actual linked-list **node objects** (not just values), so a node can be unlinked/relinked in constant time using its stored reference, without any traversal.
</details>

<details>
<summary>24. How would you design a Rate Limiter as a reusable, embeddable component (rather than the distributed-systems version)?</summary>

Define a `RateLimiter` interface with a method like `allowRequest(clientId)`, implemented by different algorithm-specific classes (`TokenBucketRateLimiter`, `SlidingWindowRateLimiter`) — the **Strategy pattern** again fits naturally, letting the specific rate-limiting algorithm be chosen/swapped independently of how/where the rate limiter is invoked in the broader application.
</details>

<details>
<summary>25. How would you implement a Token Bucket rate limiter's core logic (the `allowRequest` method)?</summary>

Track a `currentTokens` count and a `lastRefillTimestamp` per client; on each request, first calculate and add any tokens accumulated since the last refill (based on elapsed time and the configured refill rate, capped at the bucket's max capacity), then check if at least one token is available — if so, decrement and allow the request; if not, reject it. This lazy, calculate-on-demand refill approach avoids needing a separate background thread continuously ticking to add tokens.
</details>

<details>
<summary>26. How would you design a File System (in-memory, simplified) supporting directories, files, and path-based operations?</summary>

Use the **Composite pattern** — define a common `FileSystemEntry` (or `Node`) interface/abstract class with `File` and `Directory` as concrete implementations; a `Directory` holds a collection of child `FileSystemEntry` objects (which can themselves be files or further nested directories) — this lets client code (e.g., a `getSize()` or `search()` operation) treat individual files and entire directory subtrees uniformly through the same interface, naturally supporting arbitrary nesting depth via recursion.
</details>

<details>
<summary>27. What is the Composite design pattern, and what class of problems does it elegantly solve?</summary>

Lets you compose objects into tree structures and treat individual objects and compositions of objects uniformly through a shared interface — elegant for any part-whole hierarchy (file systems, UI component trees, organizational hierarchies) where you want operations (like "calculate total size" or "render") to work identically whether applied to a single leaf node or an entire nested subtree, without the calling code needing to know which case it's dealing with.
</details>

<details>
<summary>28. How would you design a Meeting Room / Calendar Booking System, and what's the core scheduling-conflict-detection challenge?</summary>

Core entities: `Room`, `Booking`(start time, end time, room, organizer), `User`. The core challenge is efficiently detecting/preventing overlapping bookings for the same room — for a moderate number of bookings, checking a new booking's `[start, end)` interval against all existing bookings for that room for overlap (`existing.start < new.end && new.start < existing.end`) is sufficient; at larger scale, an interval tree or similar data structure indexed by room can make conflict detection more efficient than a linear scan.
</details>

<details>
<summary>29. In a Meeting Room Booking System, how would you extend the design to support recurring meetings (e.g., "every Monday at 10am")?</summary>

Introduce a `RecurrenceRule` (frequency, interval, end condition — e.g., "weekly, every Monday, until a specific end date or after N occurrences") associated with a `Booking`, and generate/expand the concrete individual occurrence instances either eagerly (materializing all occurrences up to some future horizon) or lazily (computing "does an occurrence fall on this date" on demand) — the lazy approach avoids potentially unbounded storage for indefinitely recurring meetings, at the cost of slightly more complex conflict-detection logic that must account for the recurrence pattern rather than just simple stored date ranges.
</details>

<details>
<summary>30. How would you design a Snake and Ladder game, focusing on clean object-oriented structure?</summary>

Core entities: `Board` (size, positions of `Snake`s and `Ladder`s — each mapping a start position to an end position), `Player` (current position), `Dice`, and a `Game`/`GameController` managing turns and win detection. A clean design represents `Snake` and `Ladder` uniformly (both are just "position jumps," differing only in direction — a snake moves you backward, a ladder forward) potentially via a shared `Jump`/`Entity` abstraction, simplifying the board's lookup logic to a single "is there a jump defined at this position" check regardless of whether it's a snake or ladder.
</details>

<details>
<summary>31. How would you design a Restaurant Table Reservation / Food Ordering System covering menu, order, and table management?</summary>

Core entities: `MenuItem`, `Menu`, `Order` (containing `OrderItem`s, a status — placed/preparing/served/paid), `Table`, `Reservation`, `Bill`/`Invoice`. A key design decision is modeling `Order` status as an explicit state machine (via the State pattern or a simple enum with validated transitions) to prevent invalid transitions (e.g., can't mark an order "served" before it's "prepared"), and separating the `Menu`/`MenuItem` catalog data from the transactional `Order`/`OrderItem` data (capturing the price *at the time of order*, since menu prices can change later without retroactively affecting historical orders).
</details>

<details>
<summary>32. How would you design an ATM Machine system, and what are the core entities/states?</summary>

Core entities: `ATM`, `Account`, `Card`, `Transaction` (withdrawal, deposit, balance inquiry), `CashDispenser` (tracking available denominations). The **State pattern** fits well for the ATM's overall operational flow (`IdleState` → `HasCardState` → `AuthenticatedState` → `TransactionState`), and a `CashDispensingStrategy` can encapsulate the algorithm for determining which denominations to dispense for a given withdrawal amount (e.g., greedily using the largest denominations first, subject to available inventory).
</details>

<details>
<summary>33. In an ATM Machine design, how would you handle the denomination-dispensing algorithm, and what edge case must it account for?</summary>

A greedy approach (use as many of the largest available denomination as possible, then the next largest, and so on) works for most standard denomination sets, but must account for the edge case where the ATM has run out of a needed denomination (e.g., no $20 bills left, only $50s and $10s, and the greedy approach with what's available might not be able to make an exact requested amount) — the design should detect and gracefully reject/adjust the requested amount if it's genuinely undispensable given current cash inventory, rather than assuming it's always solvable.
</details>

<details>
<summary>34. How would you design a Hotel Booking System covering room types, availability, and pricing?</summary>

Core entities: `Hotel`, `RoomType` (category with base pricing), `Room` (a specific physical room, of a given `RoomType`), `Reservation` (date range, guest, room), and a `PricingStrategy` (allowing dynamic pricing rules — seasonal rates, demand-based pricing — to be plugged in without hardcoding pricing logic directly into the booking flow). Availability checking for a date range typically requires efficiently querying "which rooms of type X have no overlapping reservation during date range Y," similar in spirit to the meeting-room conflict-detection problem.
</details>

<details>
<summary>35. How would you design an Online Shopping Cart / E-commerce Checkout flow, focusing on the checkout process's state management?</summary>

Model the checkout process as an explicit sequence of states/steps (`CartReviewState` → `ShippingAddressState` → `PaymentState` → `OrderConfirmedState`), each with validated entry conditions (can't reach the payment step without a valid shipping address already set) — this can be implemented via the State pattern for strict enforcement, or more simply via a well-designed `Checkout`/`Order` aggregate with clear invariants enforced in its methods, ensuring the checkout process can't be completed in an invalid or out-of-order sequence.
</details>

<details>
<summary>36. What is the Observer pattern, and how would you apply it in designing a Stock Price Alerting system?</summary>

The Observer pattern lets multiple "observer" objects register interest in a "subject" and be automatically notified when the subject's state changes. In a stock alerting system: a `Stock` (subject) maintains a list of registered `PriceAlertObserver`s (e.g., specific users' alert conditions); whenever the stock's price updates, it notifies all registered observers, each independently checking if its specific alert condition (e.g., "price dropped below $50") is now met and triggering a notification if so — decoupling the stock price update mechanism from the various different alerting conditions that might be registered against it.
</details>

<details>
<summary>37. How would you design a Pub/Sub (Publish-Subscribe) messaging system from scratch as an LLD exercise?</summary>

Core entities: `Topic` (maintains a list of subscribed `Subscriber`s), `Publisher` (publishes `Message`s to a specific `Topic`), `Subscriber` (an interface with an `onMessage(Message)` callback), and a central `MessageBroker`/`PubSubSystem` managing topic registration and routing published messages to all current subscribers of that topic — essentially a generalized, multi-topic application of the Observer pattern, often also requiring consideration of whether delivery is synchronous (in the publishing thread) or asynchronous (via an internal queue/thread pool per topic or subscriber).
</details>

<details>
<summary>38. How would you design a URL Shortener's core encoding/decoding logic as an LLD problem (distinct from the distributed-systems HLD version)?</summary>

Core focus at the LLD level: the actual algorithm for generating a short, unique code — e.g., base62 encoding (using characters `[a-zA-Z0-9]`) of an auto-incrementing numeric ID, implemented via repeated division/modulo by 62 and mapping remainders to characters, producing a compact fixed-or-variable-length code; plus a bidirectional mapping structure (`Map<String, String>` in a simplified in-memory version) supporting both `shorten(longUrl)` and `expand(shortUrl)` operations with appropriate encapsulation (e.g., a dedicated `UrlShortenerService` class rather than exposing the raw map).
</details>

<details>
<summary>39. What is the Decorator pattern, and how would you apply it to design a Pizza/Coffee ordering system with customizable toppings/add-ons?</summary>

The Decorator pattern lets you attach additional behavior/responsibilities to an object dynamically by wrapping it in decorator objects implementing the same interface. For a pizza ordering system: a base `Pizza` interface has a `cost()` method; a base `PlainPizza` implements it with a base cost; each topping (`CheeseTopping`, `MushroomTopping`) is a decorator wrapping a `Pizza` and adding its own cost to the wrapped pizza's cost — letting you compose an arbitrary combination of toppings dynamically (`new MushroomTopping(new CheeseTopping(new PlainPizza()))`) without needing a combinatorial explosion of subclasses for every possible topping combination.
</details>

<details>
<summary>40. Why is the Decorator pattern preferred over subclassing for the "customizable toppings" scenario, specifically?</summary>

Subclassing would require a separate class for every combination of toppings (`CheeseMushroomPizza`, `CheeseOnionPizza`, `CheeseMushroomOnionPizza`...) — an explosion of classes growing combinatorially with the number of toppings. The Decorator pattern instead lets you compose any combination dynamically at runtime by wrapping/nesting decorator objects, needing only one decorator class per topping regardless of how many combinations are actually needed.
</details>

<details>
<summary>41. How would you design a Text Editor's "Find and Replace" and "Undo/Redo" functionality cleanly?</summary>

For undo/redo, the **Command pattern** fits naturally — each edit operation (insert, delete, replace) is encapsulated as a `Command` object with `execute()` and `undo()` methods, maintained in two stacks (an undo stack of executed commands, and a redo stack of undone commands that can be re-applied) — pushing a new command onto the undo stack and clearing the redo stack on any new edit (since redoing past that point is no longer meaningful once a new edit branches off).
</details>

<details>
<summary>42. How would you design a Chess/Tic-Tac-Toe AI opponent using the Strategy pattern for different difficulty levels?</summary>

Define an `AIStrategy` interface with a `chooseMove(Board)` method, implemented by `RandomMoveStrategy` (easy — picks any valid move), `MinimaxStrategy` (harder — evaluates future game states to some search depth), and potentially `MinimaxWithAlphaBetaPruningStrategy` (optimized harder difficulty) — the game engine depends only on the `AIStrategy` interface, letting difficulty be selected/swapped without any change to the core game loop logic.
</details>

<details>
<summary>43. What is the Template Method pattern, and how would you apply it when designing a data-processing pipeline with steps common across different data types (CSV, JSON, XML import)?</summary>

Template Method defines the overall skeleton/sequence of an algorithm in a base class, deferring specific step implementations to subclasses. For a data import pipeline: an abstract `DataImporter` class defines a final `importData()` method calling, in order, `readSource()`, `parseData()` (abstract, implemented differently per format), `validateData()` (shared, common validation logic in the base class), and `saveToDatabase()` (shared) — ensuring the overall process structure and shared steps stay consistent across formats, while only the format-specific parsing logic actually varies between subclasses.
</details>

<details>
<summary>44. How would you design an Inventory Management System handling stock levels across multiple warehouses?</summary>

Core entities: `Product`, `Warehouse`, `InventoryItem` (product + warehouse + quantity), and a `StockTransferService`/`ReplenishmentStrategy` (handling logic for restocking or transferring inventory between warehouses when one location runs low) — key design consideration is ensuring stock-level updates (increment/decrement operations happening concurrently from multiple order-processing threads) are handled with appropriate concurrency control to avoid race conditions leading to incorrect stock counts or overselling.
</details>

<details>
<summary>45. How would you design a Social Media "News Feed" ranking system's core interfaces (LLD-level, not the distributed HLD architecture)?</summary>

Define a `RankingStrategy` interface with a method like `rank(List<Post>, User)` returning a sorted list, implemented by different strategies (`ChronologicalRankingStrategy`, `EngagementBasedRankingStrategy`, `MLModelRankingStrategy`) — the `NewsFeedService` depends only on the abstraction, allowing the actual ranking algorithm to be swapped or A/B tested independently of the rest of the feed-generation/serving logic.
</details>

<details>
<summary>46. What is the Builder pattern, and when would you apply it when designing a class representing a complex object like an `HttpRequest` or a `PizzaOrder` with many optional configuration fields?</summary>

The Builder pattern separates the construction of a complex object from its representation, letting you build up an object step by step through a fluent chain of method calls, avoiding constructors with an unwieldy number of parameters (especially many optional ones, which would otherwise require numerous overloaded "telescoping constructors"). It also allows validating the final object's consistency (e.g., "you can't set both X and Y") in a single `build()` step, rather than allowing an object to exist in a partially-invalid intermediate state.

```java
HttpRequest request = HttpRequest.builder()
    .url("https://api.example.com")
    .method("POST")
    .header("Content-Type", "application/json")
    .build();
```
</details>

<details>
<summary>47. How would you design a Payment Processing module (LLD-level) supporting multiple payment methods (credit card, PayPal, wallet)?</summary>

Define a `PaymentStrategy`/`PaymentMethod` interface with a `pay(amount)` method, implemented by `CreditCardPayment`, `PayPalPayment`, `WalletPayment` — the checkout/order logic depends only on this abstraction, and a `PaymentFactory` (or dependency injection) selects the appropriate concrete implementation based on the user's chosen payment method at checkout time, keeping the core checkout flow entirely decoupled from any specific payment provider's implementation details.
</details>

<details>
<summary>48. What is the Adapter pattern, and how might it apply when integrating a third-party payment gateway's SDK into your own `PaymentMethod` interface hierarchy?</summary>

If a third-party payment SDK exposes a different method signature/interface than your own internal `PaymentMethod` interface expects, an `Adapter` class wraps the third-party SDK and translates calls to/from your internal interface's expected shape — this isolates your core business logic from being directly coupled to (and needing rewrites if) the third-party SDK's specific API changes, keeping the adaptation/translation logic contained in one place.
</details>

<details>
<summary>49. How would you design a Job Scheduler (LLD-level, single-machine, not the distributed HLD version) that supports scheduling tasks to run at specific times or after delays?</summary>

Core structure: a `Task`/`Job` interface (with an `execute()` method), a scheduling data structure (often a **min-heap/priority queue** ordered by next-execution-time, allowing efficient retrieval of "what needs to run next") managed by a `Scheduler` class, and a background thread/executor that repeatedly checks/waits for the next scheduled task's time and dispatches it — for recurring tasks, after execution, the task is recalculated with its next execution time and re-inserted into the priority queue.
</details>

<details>
<summary>50. What is the difference between association, aggregation, and composition in object-oriented design, and how would you illustrate each with an example?</summary>

**Association** — a general relationship where objects are aware of/use each other, but neither owns the other's lifecycle (e.g., a `Student` and a `Course` — related, but independently existing). **Aggregation** — a "has-a" relationship where one object contains references to others, but the contained objects can exist independently (a `Department` has `Employee`s, but employees can exist even if the department is dissolved — a "whole-part" relationship with independent lifecycles). **Composition** — a stronger "has-a" where the contained object's lifecycle is tightly bound to the containing object (a `House` has `Room`s — a room typically doesn't meaningfully exist independent of its house; destroying the house destroys its rooms).
</details>

<details>
<summary>51. How would you design a Bowling Game score-calculation system, and what makes it a genuinely tricky LLD problem despite sounding simple?</summary>

Core entities: `Frame` (holds 1-2 rolls, or 3 for the final frame), `Game` (a sequence of 10 `Frame`s), and scoring logic. It's tricky because bonus scoring for strikes/spares depends on rolls in **subsequent** frames (a strike's score includes the next two rolls, a spare's includes the next one roll) — clean designs often calculate a frame's final score lazily/on-demand once enough subsequent roll data is available, rather than trying to compute each frame's score immediately and independently as rolls come in.
</details>

<details>
<summary>52. How would you design a Traffic Light Control System at an intersection, focusing on state transitions?</summary>

Core entities: `TrafficLight` (per direction, with a `LightState`: Red/Yellow/Green), and an `IntersectionController` coordinating the timing/transitions across all lights at the intersection to ensure mutually exclusive green states for conflicting directions — the State pattern (or a simpler enum-driven state machine with a defined valid-transition table) enforces that lights can only transition through valid sequences (Green → Yellow → Red → Green), preventing an invalid direct Red-to-Green (skipping Yellow) transition from being possible in the code.
</details>

<details>
<summary>53. How would you design a Car Rental System covering vehicle inventory, reservations, and pricing?</summary>

Core entities: `Vehicle` (with a `VehicleType` — economy/SUV/luxury), `RentalStore`/`Branch` (location-specific inventory), `Reservation` (date range, vehicle, customer), and a `PricingStrategy` (accounting for vehicle type, rental duration, and possibly seasonal/demand-based adjustments). Availability checking follows the same date-range-overlap-detection pattern seen in the Meeting Room and Hotel Booking designs — a recurring LLD theme worth recognizing across superficially different problems.
</details>

<details>
<summary>54. What is a common recurring theme/pattern across many different LLD "booking/reservation" style problems (parking, hotel, car rental, meeting rooms)?</summary>

They all fundamentally reduce to the same core sub-problem: managing a pool of limited, interchangeable-within-category resources (spots, rooms, cars) and efficiently checking/preventing overlapping allocation of the same specific resource across a time range or duration — recognizing this shared structure lets you reuse a similar underlying design approach (availability-checking logic, allocation strategy abstraction) across what initially look like quite different problem domains.
</details>

<details>
<summary>55. How would you design a Music Streaming Service's core playlist/queue management (LLD-level)?</summary>

Core entities: `Song`, `Playlist` (an ordered collection of `Song`s), `Queue`/`PlaybackQueue` (the currently-playing sequence, which might differ from a static playlist due to shuffle/repeat modes), and a `PlaybackStrategy` interface encapsulating different playback-order behaviors (`SequentialPlaybackStrategy`, `ShufflePlaybackStrategy`, `RepeatOnePlaybackStrategy`) — letting the play/next/previous logic in the core player remain unchanged regardless of which playback mode is currently active.
</details>

<details>
<summary>56. How would you design a Ride Sharing (LLD-level) matching/pricing logic, distinct from the distributed geospatial-indexing HLD concerns?</summary>

Core entities: `Rider`, `Driver`, `Ride` (with a `RideStatus` state machine: requested → accepted → in-progress → completed/cancelled), `Vehicle`, and a `FareCalculationStrategy` (encapsulating pricing logic — base fare + distance + time + surge multiplier — as a pluggable, independently-testable component separate from the ride-matching/status-management logic).
</details>

<details>
<summary>57. What is the Visitor design pattern, and where might it apply in an LLD problem involving a tree of heterogeneous object types (e.g., different shapes in a drawing application)?</summary>

The Visitor pattern lets you add new operations to a set of existing classes without modifying those classes themselves, by defining a separate `Visitor` interface with a `visit()` method per concrete type, and having each element in the object structure accept a visitor (double dispatch). For a drawing app with `Circle`, `Square`, `Triangle` shapes, a new operation (e.g., "calculate total area," or "export to SVG") can be added as a new `Visitor` implementation without touching the existing shape classes at all — useful specifically when the set of element types is relatively stable, but you anticipate needing to add many new *operations* over time.
</details>

<details>
<summary>58. What is a key trade-off/downside of the Visitor pattern, and when might it be a poor fit despite solving the "add operations without modifying classes" problem?</summary>

The Visitor pattern makes it easy to add new *operations* but hard to add new *element types* (every existing Visitor implementation needs a new `visit()` method added for the new type) — it essentially inverts the typical extensibility trade-off, so it's a poor fit if the set of element types is expected to grow/change frequently, even if the set of operations is relatively stable.
</details>

<details>
<summary>59. How would you design a Library Fine/Penalty calculation system, ensuring the calculation logic is easily extensible for different fine rules?</summary>

Encapsulate fine calculation behind a `FineCalculationStrategy` interface (e.g., a flat daily rate, or a tiered rate that increases the longer a book is overdue, or different rates per book category) — the `Loan`/`BookLending` class delegates to the currently configured strategy to compute an overdue fine, rather than embedding a specific fine formula directly, allowing the library's fine policy to be changed/extended without modifying the core lending logic.
</details>

<details>
<summary>60. How would you design a Shopping Cart's discount/coupon application logic to support multiple, potentially combinable discount types (percentage off, fixed amount off, buy-one-get-one)?</summary>

Define a `DiscountStrategy`/`DiscountRule` interface with an `apply(Cart)` method, implemented per discount type — the **Chain of Responsibility** pattern can then apply multiple applicable discount rules in sequence to a cart (each rule potentially further reducing the total), or a `CompositeDiscount` could combine multiple discounts — while also needing explicit business logic to handle discount-stacking rules (e.g., "coupon codes can't be combined with the loyalty discount") that go beyond what a purely structural pattern alone would enforce.
</details>

<details>
<summary>61. How would you design a class hierarchy for different Employee types (Full-time, Part-time, Contractor) with different salary calculation rules, avoiding a fragile inheritance hierarchy?</summary>

Rather than deep inheritance (`Employee` → `FullTimeEmployee` → ...) which can become rigid if an employee's type/rules can change over time, prefer composition — an `Employee` class holds a reference to a `CompensationStrategy` (`SalariedCompensation`, `HourlyCompensation`, `ContractCompensation`) — this allows an employee's compensation model to be reassigned without needing to reclassify/recreate the underlying `Employee` object itself, and avoids the classic fragile-base-class problems that can emerge from deep, rigid inheritance hierarchies modeling what's really more of a role/behavior difference than a fundamental type difference.
</details>

<details>
<summary>62. What is "favor composition over inheritance," and how does it manifest as a concrete LLD interview evaluation criterion?</summary>

A general design guideline (discussed in the Java section too) that in an LLD interview specifically often manifests as: an interviewer proposing a new requirement mid-interview (e.g., "now an employee can have compensation change dynamically, or even be both salaried AND receive a bonus structure") to see whether the candidate's initial design (if built on rigid inheritance) becomes awkward/requires significant rework, versus a composition-based design that can accommodate the new requirement more gracefully by simply composing in an additional behavior/strategy object.
</details>

<details>
<summary>63. How would you design a generic "Validation Framework" that can apply a configurable set of validation rules to different types of objects?</summary>

Define a generic `ValidationRule<T>` interface with a `validate(T object)` method returning a result (valid/invalid + error message), and a `Validator<T>` that holds a collection of `ValidationRule<T>` instances, running each against a given object and aggregating results — new validation rules can be added by simply implementing the interface and registering the rule, without modifying the core `Validator` class (Open/Closed Principle in action).
</details>

<details>
<summary>64. How would you design a simplified Dependency Injection container from scratch (as an LLD exercise to test understanding of reflection/object graphs)?</summary>

Core structure: a registry mapping interfaces/types to their concrete implementation classes (or pre-built instances), and a `resolve(Class<T>)` method that, given a requested type, looks up its registered implementation, inspects its constructor (via reflection) to determine its own dependencies, recursively resolves each of those dependencies first, and finally instantiates the object with its fully resolved dependencies — a good design also needs to handle/detect circular dependencies (e.g., tracking types currently being resolved on the call stack and throwing an error if a cycle is detected) and decide on instance scope (singleton vs new-instance-per-resolution).
</details>

<details>
<summary>65. How would you design a Task Management/To-Do application supporting task dependencies (task B can't start until task A is complete)?</summary>

Model tasks and their dependencies as a **directed graph** (each `Task` has a list of prerequisite `Task`s it depends on) — determining a valid execution order (or detecting if the dependencies are even satisfiable) is a classic **topological sort** problem; the design should also explicitly detect and reject circular dependencies (which would make topological sort/valid execution order impossible) at the point a new dependency is added, rather than only discovering the problem later when trying to compute an execution order.
</details>

<details>
<summary>66. How would you design a simplified Version Control System's core "commit" and "diff" data model (LLD-level, not the full distributed Git internals)?</summary>

Core entities: `File`, `Commit` (a snapshot reference, parent commit reference(s) — supporting the linked, potentially branching commit-history graph), and a `Repository` managing the current state/HEAD and commit history. Representing each commit as storing only the *changes* (a diff) relative to its parent, versus storing a full snapshot per commit, is a genuine design trade-off (diffs save storage but make reconstructing a specific historical state more computationally expensive, requiring replaying diffs) worth discussing explicitly.
</details>

<details>
<summary>67. How would you design a Rate-limited API Client wrapper (LLD-level) that transparently handles retries with backoff for a flaky third-party API?</summary>

Wrap the actual API call behind a `Decorator`-style class implementing the same client interface, adding retry-with-backoff logic transparently around calls to the underlying real client — this keeps the resilience logic (retry count, backoff calculation, which exceptions are retryable) cleanly separated from both the actual API-calling logic and the business logic that uses the client, and makes the resilience behavior itself independently unit-testable by mocking the underlying wrapped client.
</details>

<details>
<summary>68. How would you design a class structure for a Quiz/Exam application supporting multiple question types (multiple-choice, true/false, short-answer) with different scoring/validation logic per type?</summary>

Define an abstract `Question` base class (or interface) with an abstract `isCorrect(Answer)` method, implemented differently by `MultipleChoiceQuestion`, `TrueFalseQuestion`, `ShortAnswerQuestion` subclasses — a `Quiz`/`Exam` holds a heterogeneous collection of `Question` objects and can score the whole exam uniformly by calling `isCorrect()` polymorphically on each, without needing to know or check each question's specific type via conditional logic.
</details>

<details>
<summary>69. What is the Liskov Substitution Principle, and how might a naive Question-type class hierarchy (from the previous question) accidentally violate it?</summary>

States that objects of a superclass should be replaceable with objects of a subclass without breaking correctness. A violation might occur if, say, a `ShortAnswerQuestion` subclass's `isCorrect()` method has a different, incompatible contract (e.g., it requires an additional parameter not present in the base interface, or behaves unexpectedly/throws for input that the base class's contract implies should be handled) — any code written generically against the base `Question` type should be able to work correctly with any subclass without special-casing, and a hierarchy that requires such special-casing has likely violated LSP somewhere.
</details>

<details>
<summary>70. How would you design a simplified Search/Autocomplete data structure (a Trie) from scratch as an LLD coding exercise?</summary>

A `TrieNode` class holds a map of `Map<Character, TrieNode> children` and a boolean `isEndOfWord` flag; a `Trie` class wraps a root `TrieNode` and exposes `insert(word)` (walking/creating nodes character by character), `search(word)` (exact match check), and `startsWith(prefix)` (prefix existence check) — for an autocomplete feature specifically, you'd typically also augment each node with either a list of complete words reachable from it, or perform a DFS from the prefix's ending node to collect all valid completions on demand.
</details>

<details>
<summary>71. How would you design a class for representing and validating Money/Currency amounts, avoiding common floating-point pitfalls?</summary>

Never use `float`/`double` for monetary values directly (floating-point representation errors cause subtle incorrect calculations) — use a fixed-point/integer-based representation instead (e.g., storing amounts as integer cents/smallest currency unit, or using `BigDecimal` in Java with explicit rounding modes specified), and design a dedicated `Money` value object encapsulating both the amount and currency together, with arithmetic operations that explicitly validate/reject mixing different currencies (preventing a nonsensical "add $5 + €5" operation from silently producing a meaningless result).
</details>

<details>
<summary>72. What is a Value Object, and why is designing `Money` as an immutable value object (rather than a mutable class with getters/setters) generally good practice?</summary>

A Value Object is defined by its attributes rather than a unique identity — two `Money` objects representing "$10" are considered equal regardless of being different object instances. Immutability prevents a whole class of subtle bugs where a `Money` object is accidentally mutated in a shared context (e.g., one part of the code changing an amount that another part still holds a reference to, expecting the original value) — arithmetic operations instead return a **new** `Money` instance rather than mutating the original, which is both safer and matches how numeric/monetary values intuitively behave.
</details>

<details>
<summary>73. How would you design a class structure to represent a Deck of Cards and support common card game operations (shuffle, deal)?</summary>

Core entities: `Card` (with `Suit` and `Rank` enums — an immutable value object, since a specific card's identity doesn't change), `Deck` (a collection of 52 unique `Card`s, with `shuffle()` — typically implementing the Fisher-Yates algorithm — and `dealCard()` methods), and potentially a `Hand` representing a player's currently held cards — designed generically enough (e.g., not hardcoding assumptions specific to one card game) to support building multiple different card games (Poker, Blackjack) on top of the same core `Card`/`Deck` primitives.
</details>

<details>
<summary>74. How would you design a class hierarchy for different types of Bank Accounts (Savings, Checking, Fixed Deposit) with different interest-calculation and withdrawal-rule behaviors?</summary>

An abstract `Account` base class holds common state (balance, account number, owner) and common operations (`deposit()`), while `calculateInterest()` and `withdraw()` (with type-specific rules — e.g., a `SavingsAccount` might limit withdrawals per month, a `FixedDepositAccount` might disallow withdrawal before maturity entirely) are implemented differently per subclass — a good design considers whether these truly vary enough to warrant subclassing (inheritance) versus whether a composed `InterestStrategy`/`WithdrawalPolicy` (composition) would be more flexible if account "types" might need to be reconfigured dynamically.
</details>

<details>
<summary>75. How would you design a simplified Concurrent/Thread-safe Bounded Blocking Queue as an LLD coding exercise (without just using `java.util.concurrent.BlockingQueue` directly)?</summary>

Use an internal fixed-size array/list as the buffer, with `synchronized` methods (or explicit `Lock`/`Condition` objects) for `put()` and `take()` — `put()` should block (via `wait()`, or a `Condition.await()`) if the queue is currently full, and `take()` should block if the queue is currently empty, with each operation notifying (`notifyAll()`/`signalAll()`) waiting threads on the opposite operation after successfully completing, so a `put()` wakes up any threads blocked in `take()` waiting for an item to become available, and vice versa.
</details>

<details>
<summary>76. How would you design a Producer-Consumer system (LLD-level, focusing on the actual synchronization code) using `wait()`/`notify()`?</summary>

A shared bounded buffer (e.g., backed by the queue design from the previous question) with producer threads calling a blocking `put()` and consumer threads calling a blocking `take()` — the key correctness detail is using `wait()` inside a `while` loop re-checking the actual condition (not an `if`), to correctly handle spurious wakeups and the scenario where multiple producers/consumers are waiting and a `notifyAll()` wakes several of them, only one of which can actually proceed given the now-changed buffer state.
</details>

<details>
<summary>77. How would you design a class representing a Matrix and its common operations (addition, multiplication, transpose), with attention to encapsulation and immutability?</summary>

A `Matrix` class internally storing a 2D array, exposing operations (`add()`, `multiply()`, `transpose()`) that validate dimension compatibility explicitly (throwing a clear exception for e.g. mismatched dimensions in multiplication, rather than an obscure `ArrayIndexOutOfBoundsException`) and return a **new** `Matrix` instance rather than mutating the original operands — keeping the internal array representation genuinely encapsulated (not exposing a mutable reference to the raw array directly via a getter) to preserve the class's invariants.
</details>

<details>
<summary>78. How would you design a Feature Flag evaluation system (LLD-level) supporting different targeting rules (percentage rollout, specific user IDs, user attributes)?</summary>

Define a `TargetingRule` interface with an `evaluate(User, Context)` method, implemented by `PercentageRolloutRule` (deterministic based on a consistent hash of user ID), `UserIdAllowListRule`, `AttributeMatchRule` — a `FeatureFlag` holds one or more `TargetingRule`s (potentially combined with AND/OR composite logic) that are evaluated to determine if a given user should see the feature — designed so new targeting rule types can be added without modifying the core flag-evaluation engine.
</details>

<details>
<summary>79. How would you design a class structure for a Blog/CMS Comment system supporting nested/threaded replies?</summary>

A `Comment` class with a self-referential `parentComment` reference (or `parentId`) and a collection of `replies` (child comments) — naturally forming a tree structure per top-level comment; rendering/traversing the nested comment tree is a straightforward recursive operation, and depth-limiting (many UIs cap visual nesting depth) can be handled either at render time or enforced structurally at comment-creation time depending on requirements.
</details>

<details>
<summary>80. How would you design a Undo-able "Whiteboard/Drawing" application's core shape/canvas model?</summary>

A `Shape` interface (implemented by `Circle`, `Rectangle`, `Line`, etc., each knowing how to `draw()` and report its own bounding box), a `Canvas` holding an ordered collection of drawn `Shape`s, and — for undo support — the **Command pattern** wrapping each drawing action (add shape, move shape, delete shape, resize shape) as an undoable command pushed onto a history stack, mirroring the same undo/redo approach discussed for the text editor and chess examples, reinforcing that this is a broadly reusable LLD pattern across many different "undoable action" domains.
</details>

<details>
<summary>81. How would you design a class hierarchy for different Discount/Promotion types in an e-commerce system where promotions can have complex eligibility rules (minimum cart value, specific product categories, first-time customers only)?</summary>

Separate the **eligibility check** from the **discount calculation** into two distinct responsibilities/interfaces (`EligibilityRule` and `DiscountCalculator`), composed together within a `Promotion` object — this avoids conflating "does this promotion apply here" logic with "how much discount does it give," letting each concern be independently extended (new eligibility rule types, new discount calculation types) and combined flexibly (the same discount calculation logic might be reused across multiple promotions with different eligibility rules).
</details>

<details>
<summary>82. How would you design a simplified "Observer-based" Event Bus (in-process, not distributed) supporting multiple event types and typed listeners?</summary>

An `EventBus` maintaining a `Map<Class<?>, List<EventListener<?>>>` mapping event types to their registered listeners, with `subscribe(Class<T> eventType, EventListener<T> listener)` and `publish(T event)` methods (using the event's runtime class to look up and notify the appropriate listeners) — a design consideration is whether listener notification should be synchronous (in the publishing thread, simple but can block the publisher on a slow listener) or asynchronous (dispatched to a thread pool, decoupling publisher performance from listener processing time, at the cost of added complexity in error handling/ordering guarantees).
</details>

<details>
<summary>83. What is the difference between designing a system with a rich domain model (business logic encapsulated within entity classes) versus an anemic domain model (entities are just data holders, logic lives entirely in separate "service" classes), and which does good LLD generally favor?</summary>

A **rich domain model** encapsulates behavior alongside data within entity classes themselves (e.g., an `Order` class has a `cancel()` method that itself enforces the business rule "can't cancel an already-shipped order," rather than that check living externally in an `OrderService`). An **anemic domain model** reduces entities to plain data containers, with all business logic and validation living in separate service classes operating *on* that data. Good object-oriented LLD generally favors a richer domain model (encapsulation, keeping data and the rules governing it together) — an anemic model is often considered an anti-pattern in OOP design, even though it's extremely common in practice, especially in typical layered enterprise applications.
</details>

<details>
<summary>84. How would you design a class structure for representing and validating a Sudoku board, focusing on clean separation of the validation rules?</summary>

A `SudokuBoard` class holding the grid state, with validation logic decomposed into independently checkable rules (`RowValidator`, `ColumnValidator`, `BoxValidator`), each implementing a common `SudokuValidator` interface with an `isValid(Board)` method — the overall `isValidBoard()` check simply runs all registered validators and requires all to pass, making it straightforward to test each rule in isolation and to extend with additional custom rule variants (e.g., a diagonal-Sudoku variant) without modifying existing validators.
</details>

<details>
<summary>85. How would you design a simplified "Rules Engine" that can evaluate a configurable set of business rules against an input object and return applicable actions?</summary>

Define a `Rule` interface with a `condition(Input)` predicate and an associated `action`/`consequence`, and a `RuleEngine` that holds a prioritized/ordered collection of `Rule`s, evaluates each rule's condition against a given input, and executes (or collects) the actions of all matching rules — considerations include whether rule evaluation should stop at the first match (like a simple decision list) or evaluate/apply all matching rules, and how rule priority/ordering conflicts are resolved when multiple rules could apply to the same input.
</details>

<details>
<summary>86. How would you design a Generic Object Pool (e.g., for expensive-to-create objects like database connections) as an LLD exercise?</summary>

An `ObjectPool<T>` class maintaining a collection of available (idle) and in-use `T` instances, with `acquire()` (returns an available instance, or blocks/creates a new one up to a configured max size if none are available) and `release(T)` (returns an instance to the available pool) methods — needs thread-safe access to the internal pool state (since multiple threads acquire/release concurrently), and a factory/supplier function for actually creating new instances when the pool needs to grow, decoupling the pool's generic management logic from the specifics of what kind of object it's pooling.
</details>

<details>
<summary>87. How would you design a class structure for a Multiplayer Game's turn management system supporting different numbers of players and turn-order rules?</summary>

A `TurnManager`/`GameEngine` holding an ordered collection of `Player`s and a `TurnOrderStrategy` interface (`SequentialTurnOrder`, `RandomTurnOrder`, or more complex rules like "skip a turn if a certain condition is met") — decoupling the specific turn-order algorithm from the core game loop, which simply asks the strategy "who goes next" rather than hardcoding a fixed round-robin assumption that might not generalize to all games/variants.
</details>

<details>
<summary>88. What is the Mediator design pattern, and how might it apply to designing a Chat Room feature where multiple users need to communicate without being directly coupled to each other?</summary>

The Mediator pattern centralizes communication between a set of objects through a single mediator object, so the objects don't need direct references to each other — for a chat room, a `ChatRoom`/`ChatMediator` object receives messages from any participating `User` and is responsible for broadcasting them to all other participants, meaning individual `User` objects only need a reference to the mediator, not to every other user directly, reducing the tightly-coupled many-to-many relationships that would otherwise exist between all participants.
</details>

<details>
<summary>89. How would you design a class hierarchy for a Notification Preference system where a user can configure different notification channels for different event types (e.g., email for security alerts, push for social activity, nothing for marketing)?</summary>

A `NotificationPreference` mapping (per user) from `EventType` to a set of enabled `NotificationChannel`s, consulted by the notification-sending logic before actually dispatching — designed so default preferences can be applied when a user hasn't explicitly configured a preference for a given event type, and so new event types/channels can be added without requiring changes to existing users' already-stored preference data (e.g., via sensible defaulting rather than requiring every user record to explicitly enumerate every possible event type).
</details>

<details>
<summary>90. How would you design a class structure to model a Recipe/Cooking application supporting ingredient scaling (e.g., "scale this recipe from 4 servings to 6 servings")?</summary>

A `Recipe` class holding a list of `Ingredient` objects (each with a quantity and unit) defined for a base serving size — a `scale(targetServings)` method returns a new, scaled representation (proportionally adjusting each ingredient's quantity) rather than mutating the original recipe, treating the recipe itself as effectively immutable reference data and scaling as a derived, on-demand transformation — also needing to handle unit-specific scaling nuances sensibly (e.g., "a pinch of salt" might not scale linearly the same way "2 cups of flour" does, a domain-specific edge case worth at least acknowledging).
</details>

<details>
<summary>91. What is the Interpreter design pattern, and where might it apply in an LLD problem involving evaluating simple user-defined expressions or rules (e.g., a basic calculator or a simple query filter language)?</summary>

The Interpreter pattern defines a grammar for a simple language and an interpreter that evaluates sentences in that grammar, typically by representing the grammar as a tree of expression objects (each implementing a common `interpret(context)` method) — for a basic calculator, `Number`, `AddExpression`, `SubtractExpression` classes could each implement `interpret()`, composed into a tree matching the parsed structure of an input expression like `"3 + 4 - 2"`, letting evaluation proceed via straightforward tree traversal/recursion.
</details>

<details>
<summary>92. How would you design a class structure for an Access Control / Permission system supporting roles, and both allow and deny rules?</summary>

Core entities: `User`, `Role` (a named collection of `Permission`s), and `Permission` (representing a specific allowed/denied action on a resource) — a common design decision is whether deny rules should always take precedence over allow rules when both could apply to the same user/resource/action combination (a common, conservative security-oriented default), and whether permissions are evaluated via direct role assignment only, or also support role hierarchies/inheritance (a "Manager" role automatically including all "Employee" role permissions).
</details>

<details>
<summary>93. How would you design a Generic Result/Either type (`Result<T, E>`) for representing an operation's success or failure explicitly in the type system, rather than relying purely on exceptions?</summary>

A generic wrapper type holding either a success value of type `T` or an error value of type `E` (but never both), with methods like `isSuccess()`, `getValue()` (only valid if success), `getError()` (only valid if failure), and often functional-style `map()`/`flatMap()` methods for chaining operations that might fail — this makes the possibility of failure an explicit, visible part of a method's return type/signature (forcing callers to handle it) rather than an implicit, easy-to-forget possibility only discoverable by reading a method's `throws` clause or documentation.
</details>

<details>
<summary>94. How would you design a class structure for representing Polymorphic Shapes (Circle, Rectangle, Triangle) with a common `area()` and `perimeter()` calculation, and what's a subtle LLD interview trap related to a `Square`/`Rectangle` relationship?</summary>

A common `Shape` interface/abstract class with `area()` and `perimeter()` methods implemented per concrete shape. The subtle trap: modeling `Square extends Rectangle` seems intuitive (a square "is a" rectangle geometrically), but if `Rectangle` exposes independent `setWidth()`/`setHeight()` mutator methods, a `Square` subclass would need to override them to keep both dimensions equal — violating the Liskov Substitution Principle, since code expecting a `Rectangle` (and calling `setWidth()` then `setHeight()` independently, expecting only the intended dimension to change) would get surprising, incorrect behavior if handed a `Square` instance instead. This is a classic illustration of why "is-a" in natural language doesn't always safely translate into "should inherit from" in OOP design when mutable state/behavior is involved.
</details>

<details>
<summary>95. How would you design a class structure for a Survey/Form Builder application supporting different question types and conditional logic (e.g., "show question 5 only if question 3 was answered 'Yes'")?</summary>

A `Question` base type (similar to the earlier Quiz example) plus a separate `VisibilityCondition`/`ConditionalLogic` object associated with a question, evaluated against previously submitted answers to determine if that question should currently be shown — keeping the conditional-display logic decoupled from the question's own content/validation logic, since these are genuinely independent concerns (a question's own validation rules don't change based on whether it's currently visible or not).
</details>

<details>
<summary>96. How would you design a class structure for a generic Pagination utility that can wrap any list-based query result?</summary>

A generic `Page<T>` class holding the current page's content (`List<T>`), plus metadata (current page number, page size, total elements, total pages) — combined with a `Pageable` request object (page number + size, optionally sort criteria) passed into query methods, and a clean separation between the pagination *metadata* structure (generically reusable across any entity type) and the actual data-fetching logic (which varies per specific query/repository).
</details>

<details>
<summary>97. How would you design a class structure for a simplified in-memory Key-Value Store supporting TTL (time-to-live) expiration for keys?</summary>

An internal `Map<K, ValueWrapper<V>>` where `ValueWrapper` holds both the actual value and an expiration timestamp — reads check the expiration timestamp and treat an expired entry as absent (either lazily removing it on access, or via a separate background "reaper" thread periodically sweeping for and removing expired entries) — the choice between lazy (check-on-read only) and active (background sweep) expiration is a genuine design trade-off between simplicity/read-path overhead versus memory being held longer than strictly necessary by never-again-accessed expired keys.
</details>

<details>
<summary>98. How would you design a class structure for a Multi-level Cache (e.g., an in-memory L1 cache backed by a larger, slower L2 cache)?</summary>

A `Cache` interface implemented consistently by both `InMemoryCache` (L1) and, e.g., a `RedisCache` (L2), with a `TieredCache`/`CacheChain` class composing both — a lookup first checks L1; on an L1 miss, it checks L2, and if found there, populates L1 before returning the result (so subsequent lookups hit the faster L1) — this layered composition (via the Decorator or Chain of Responsibility pattern, depending on exact framing) lets additional cache tiers be added/reconfigured without changing the calling code's simple `Cache` interface usage.
</details>

<details>
<summary>99. How would you design a class structure for Auditing/Change tracking on entities (recording who changed what and when)?</summary>

A generic `AuditLog`/`AuditEntry` entity capturing the entity type, entity ID, field changed, old value, new value, timestamp, and the user who made the change — populated either via explicit application code calling an `AuditService` at the point of each mutating operation, or more transparently via an interception mechanism (e.g., JPA entity listeners, or AOP around service methods) that automatically captures changes without requiring every business method to remember to call the audit logic manually — the latter is generally preferred for consistency, since manually-scattered audit calls are easy to accidentally miss in some code path.
</details>

<details>
<summary>100. How would you approach explaining your LLD design's extensibility when an interviewer asks "now add feature X" partway through the interview?</summary>

Rather than starting from scratch, explicitly walk through which existing classes/interfaces the new feature would extend or plug into (demonstrating the value of the abstractions already put in place — e.g., "since we already have a `PaymentStrategy` interface, adding Apple Pay support is just a new implementation of it, no changes needed to the checkout flow itself") — an interviewer probing extensibility this way is directly testing whether the initial design's abstractions were genuinely well-chosen (Open/Closed Principle in practice) or whether they'll need significant, disruptive rework to accommodate a fairly natural, foreseeable extension.
</details>

<details>
<summary>101. What is the difference between designing an LLD solution with a focus on "correctness for the given requirements" versus over-engineering with excessive abstraction/patterns not justified by the actual stated requirements?</summary>

Applying a design pattern or abstraction layer purely to demonstrate pattern knowledge, without a genuine corresponding requirement driving the need for that flexibility, is a common LLD interview misstep — e.g., introducing a full Strategy-pattern-based plugin architecture for a rule that the interviewer explicitly stated would never change, adds unnecessary complexity without real benefit; strong LLD candidates apply patterns where genuinely justified by stated or reasonably-inferred future requirements, and can articulate *why* a given pattern is warranted here specifically, not just apply it reflexively.
</details>

<details>
<summary>102. How would you design a class structure for a Content Moderation system that runs multiple independent checks (profanity filter, spam detection, image content check) against submitted content?</summary>

A `ModerationRule`/`ModerationCheck` interface with a `check(Content)` method returning a result (pass/flag/reject + reason), and a `ModerationPipeline` running a configured, ordered collection of checks against submitted content — the **Chain of Responsibility** pattern fits naturally here too, potentially allowing early-exit on the first hard rejection while still collecting all flags from checks that don't cause immediate rejection, and letting new moderation checks be added/removed/reordered via configuration rather than code changes to the core pipeline.
</details>

<details>
<summary>103. How would you design a class structure for a Grading/Assessment system supporting different, configurable grading scales (percentage-based, letter grade, pass/fail)?</summary>

A `GradingScale`/`GradingStrategy` interface with a method converting a raw score into the final grade representation, implemented by `PercentageGradingScale`, `LetterGradingScale`, `PassFailGradingScale` — an `Assessment`/`Course` is configured with a specific grading scale at creation time, and grade calculation logic delegates entirely to the configured strategy, keeping the core score-recording logic (which is the same regardless of how scores are ultimately presented/graded) cleanly separate from the presentation/categorization logic.
</details>

<details>
<summary>104. How would you design a class structure for an Order Fulfillment system that needs to check inventory, process payment, and arrange shipping — potentially with different behavior/ordering depending on business rules?</summary>

A natural fit for either the **Template Method** pattern (a base `OrderFulfillmentProcess` defining the overall skeleton — checkInventory → processPayment → arrangeShipping — with specific steps overridable by subclasses for different fulfillment variants like international vs domestic orders) or the **Chain of Responsibility** pattern (each step as an independent handler that can potentially halt/reject the order and pass control to the next step only if its own check passes) — the choice between these two patterns often comes down to whether the *sequence itself* needs to vary between variants (favoring Chain of Responsibility's more flexible ordering) or just the *implementation of individual, fixed-order steps* varies (favoring Template Method's more rigid but simpler structure).
</details>

<details>
<summary>105. What is a good closing question/consideration to raise at the end of an LLD interview, similar to the HLD interview's "how would this scale to 100x" closer?</summary>

"How would this design need to change if [a plausible, natural future requirement] were added?" — proactively raising and briefly addressing this yourself (rather than waiting to be asked) demonstrates the same kind of forward-thinking extensibility awareness that HLD's scaling discussion demonstrates, just applied to the LLD context of anticipating reasonable future feature/requirement changes rather than anticipating scale growth.
</details>

<details>
<summary>106. What is the Prototype design pattern, and where might it apply in an LLD problem involving creating many similar, expensive-to-construct objects (e.g., game entities with complex initial configuration)?</summary>

The Prototype pattern creates new objects by copying (cloning) an existing, pre-configured "prototype" instance rather than constructing each new instance from scratch — useful when object creation is expensive (e.g., involves loading configuration/resources) but you need many similar instances that only differ slightly; a `clone()` method on the prototype produces a new, independent copy that can then be individually customized without repeating the expensive base construction/configuration logic.
</details>

<details>
<summary>107. What is the difference between a shallow clone and a deep clone, and why does this distinction matter when implementing the Prototype pattern?</summary>

A shallow clone copies an object's top-level fields directly — if a field is itself a reference to a mutable object, both the original and clone end up sharing (pointing to) the *same* underlying nested object, so mutating it through one affects the other unexpectedly. A deep clone recursively copies all referenced objects too, ensuring true independence between the original and the clone — the Prototype pattern needs to deliberately choose (and clearly document) which behavior it provides, since accidentally shallow-cloning a genuinely mutable nested object is a common, subtle bug source.
</details>

<details>
<summary>108. How would you design a class structure for a Digital Wallet system supporting multiple currencies and currency conversion?</summary>

A `Wallet` holding a collection of `Balance` entries (one per currency the user holds), a `Money` value object (as discussed earlier) enforcing currency-awareness in arithmetic, and a separate `CurrencyConverter`/`ExchangeRateProvider` interface (decoupled from the wallet itself) responsible for looking up current conversion rates when a cross-currency operation (like a transfer requiring conversion) is needed — keeping the wallet's core balance-tracking logic independent of the (likely external, rate-fluctuating) exchange rate lookup mechanism.
</details>

<details>
<summary>109. How would you design a class structure for a Report Generation system supporting multiple output formats (PDF, CSV, HTML) from the same underlying data?</summary>

Separate the data-gathering/aggregation logic (producing a format-agnostic intermediate representation, e.g., a generic `ReportData` structure of rows/columns/summary values) from the format-specific rendering logic (`PdfReportRenderer`, `CsvReportRenderer`, `HtmlReportRenderer`, each implementing a common `ReportRenderer` interface) — this separation (similar in spirit to the earlier ETL-phase-separation discussion) means adding a new output format only requires a new renderer implementation, without touching the data-gathering logic at all, and vice versa.
</details>

<details>
<summary>110. How would you design a class structure for handling Retry logic generically/reusably across different kinds of operations in a codebase, without duplicating retry loops everywhere?</summary>

A generic `Retryer`/`RetryExecutor` utility class taking a `Supplier<T>` (the operation to retry), a `RetryPolicy` (max attempts, which exceptions are retryable, backoff strategy), and executing the operation with the configured retry behavior — callers wrap their specific operation in a lambda passed to this shared utility, rather than each individual call site hand-rolling its own (likely subtly-different, potentially buggy) retry loop.
</details>

<details>
<summary>111. How would you design a class structure for a Voting/Polling system that needs to prevent duplicate votes and support different voting methods (single choice, ranked choice, multiple choice)?</summary>

A `Poll` entity with a `VotingMethod`/`VoteTallyingStrategy` interface (encapsulating the different tallying algorithms — simple plurality count for single-choice, more complex elimination-round logic for ranked-choice), and a separate mechanism (e.g., a unique constraint on `(pollId, userId)` in a `Vote` record, or a `HashSet<UserId>` check for smaller-scale in-memory versions) enforcing the one-vote-per-user-per-poll invariant independently of the tallying logic itself.
</details>

<details>
<summary>112. How would you design a class structure for a Circuit Breaker (LLD-level implementation, not just the conceptual HLD pattern) including its state transitions?</summary>

A `CircuitBreaker` class with an internal `State` (Closed/Open/HalfOpen — a natural fit for the State pattern again), tracking a rolling failure count/rate; calls are wrapped through the circuit breaker's `execute(Supplier<T> operation)` method — in the Closed state, calls proceed normally but failures are counted, transitioning to Open once a threshold is exceeded; in the Open state, calls fail immediately without attempting the wrapped operation until a configured timeout elapses, transitioning to HalfOpen; in HalfOpen, a limited number of trial calls are allowed through to test recovery, transitioning back to Closed on success or back to Open on continued failure.
</details>

<details>
<summary>113. How would you design a class structure for representing a company's Organizational Hierarchy and supporting queries like "get all direct and indirect reports of a given manager"?</summary>

An `Employee` class with a `manager` reference (and optionally, for efficiency, a `directReports` collection) — naturally forming a tree structure; "get all reports" (direct and indirect) is a straightforward recursive/BFS-or-DFS tree traversal starting from the given manager node, collecting all descendant nodes — a design consideration for very large organizations is whether to compute this traversal on-demand each time (simple, always accurate) versus maintaining a precomputed/cached hierarchy-closure structure for very frequent queries (faster reads, added complexity in keeping it in sync as the org structure changes).
</details>

<details>
<summary>114. What is the Flyweight design pattern, and where might it apply in an LLD problem involving a very large number of similar objects (e.g., rendering millions of trees in a game, or characters in a text editor)?</summary>

The Flyweight pattern minimizes memory usage by sharing common, immutable ("intrinsic") state across many objects, while keeping only the genuinely unique ("extrinsic") state per individual object instance — e.g., for millions of trees of only a few distinct species in a game, the (large) 3D model/texture data (intrinsic, shared) is stored once per species and referenced by many lightweight tree instances that only individually store their unique position/rotation (extrinsic) — dramatically reducing total memory usage compared to naively duplicating the full model data per tree instance.
</details>

<details>
<summary>115. How would you design a class structure for a Library's Book Recommendation feature based on a user's borrowing history (a simple, non-ML rule-based version, LLD-level)?</summary>

A `RecommendationStrategy` interface (e.g., `SameAuthorStrategy`, `SameCategoryStrategy`, `PopularInCategoryStrategy`), with a `RecommendationEngine` combining/weighting results from one or more configured strategies — again reinforcing the Strategy pattern's broad applicability, and illustrating how even a "simple, non-ML" recommendation feature benefits from the same decoupling-of-algorithm-from-orchestration principle used in more sophisticated recommendation system designs.
</details>

<details>
<summary>116. How would you design a class structure for a Multi-step Form Wizard (e.g., a signup flow with several sequential steps, each with its own validation) ensuring users can't skip ahead to an invalid step?</summary>

A `WizardStep` interface/abstract class (each concrete step knowing its own validation rules and what data it collects) held in an ordered sequence by a `Wizard`/`FormWizardController`, which tracks the current step and only allows advancing to the next step once the current step's validation passes — and which, when resuming a partially-completed wizard, only allows navigating to steps up to (not beyond) the furthest step already successfully validated, preventing users from directly jumping to a later step via a manipulated URL/request without having satisfied the prerequisite steps' validation first.
</details>

<details>
<summary>117. How would you design a class structure for a Subscription/Billing system supporting different billing cycles (monthly, annual) and proration when a user upgrades/downgrades mid-cycle?</summary>

A `Subscription` entity referencing a `Plan` (with pricing and a `BillingCycle`), and a `ProrationStrategy` interface encapsulating the calculation logic for how to handle a mid-cycle plan change (e.g., calculating a prorated credit for unused time on the old plan and a prorated charge for the remaining time on the new plan) — keeping this genuinely tricky, business-rule-heavy calculation logic isolated and independently testable, separate from the broader subscription state-management logic.
</details>

<details>
<summary>118. How would you design a class structure for a Real Estate Listing search/filter system supporting many optional, combinable filter criteria (price range, bedrooms, location, amenities)?</summary>

A `Specification`/`Filter` interface (implementing the **Specification pattern**) with an `isSatisfiedBy(Listing)` method, where individual filters (`PriceRangeSpecification`, `BedroomCountSpecification`) can be combined using logical composite specifications (`AndSpecification`, `OrSpecification`) — this avoids a rigid search method with a large number of optional parameters and nested conditional logic, instead letting arbitrary filter combinations be composed dynamically and evaluated uniformly against each listing.
</details>

<details>
<summary>119. What is the Specification design pattern, and how does it relate to (and differ from) the Strategy pattern?</summary>

The Specification pattern encapsulates a business rule/predicate as a reusable, combinable object (with an `isSatisfiedBy()`-style method), specifically designed to be composed via logical operators (AND/OR/NOT) into more complex specifications from simpler ones. It's conceptually similar to Strategy (both encapsulate a piece of behavior/logic behind a common interface) but Specification is specifically oriented around boolean predicate/matching logic meant to be logically composable, whereas Strategy is a more general-purpose pattern for swapping out any algorithm, not necessarily boolean-predicate-shaped or intended for logical composition.
</details>

<details>
<summary>120. How would you design a class structure for an Auction system supporting different auction types (English/ascending, Dutch/descending, sealed-bid) with a common `Bid` submission flow?</summary>

An `Auction` abstract base class (or interface) defining the common flow (accept bid, determine winner) with an `AuctionStrategy`/subclass-specific implementation of the bid-validation and winner-determination rules per auction type — an English auction validates that a new bid exceeds the current highest bid, a Dutch auction's "bid" is really just accepting the currently-called price, and a sealed-bid auction defers all comparison until a defined close time, only revealing/comparing bids then — illustrating again how superficially different domain rules can often share a common structural skeleton (Template Method or Strategy) despite quite different underlying business logic.
</details>

<details>
<summary>121. How would you design a class structure for a Warehouse Robot/Automated fulfillment system's path-planning and task-assignment logic (LLD-level)?</summary>

Core entities: `Robot` (current position, status — idle/moving/carrying), `Task` (pick up item at location X, deliver to location Y), a `TaskAssignmentStrategy` (deciding which idle robot gets assigned a new task — nearest robot, least-recently-used robot for fair load distribution), and a `PathPlanningStrategy` (an interface around the actual pathfinding algorithm — e.g., A* — used to compute a robot's route, kept decoupled from the task-assignment logic so the pathfinding algorithm could be swapped/upgraded independently).
</details>

<details>
<summary>122. How would you design a class structure for a Video Conferencing application's core participant/room management (LLD-level, not the underlying media-streaming infrastructure)?</summary>

Core entities: `Meeting`/`Room` (holding a collection of `Participant`s, with a `MeetingState` — scheduled/in-progress/ended), `Participant` (with a `Role` — host/co-host/attendee, affecting permitted actions like muting others or ending the meeting for everyone), and permission-checking logic ideally centralized (e.g., via a `PermissionChecker` consulted before allowing any privileged action) rather than scattered as ad-hoc role checks throughout many different action-handling methods.
</details>

<details>
<summary>123. How would you design a class structure for a Loyalty/Rewards Points system supporting different point-earning rules per purchase category and point-redemption logic?</summary>

A `PointsEarningStrategy` interface (`StandardEarningRate`, `BonusCategoryEarningRate` for categories with promotional multipliers) determining points earned per transaction, and a separate `RedemptionStrategy`/`RedemptionCatalog` governing what points can be redeemed for and at what conversion rate — again the recurring theme of separating "how points are earned" from "how points are spent" as genuinely independent concerns, even though both operate on the same underlying points-balance data.
</details>

<details>
<summary>124. How would you design a class structure for a Slot Machine / simple Gambling game, focusing on the reel/symbol-matching and payout-calculation logic?</summary>

`Reel` (a collection of possible `Symbol`s with associated probabilities/weights), a `SlotMachine` holding multiple `Reel`s and a `spin()` method randomly selecting a symbol per reel (respecting each symbol's configured probability weight, not just uniform random selection), and a separate `PayoutStrategy`/`PayoutTable` evaluating the resulting symbol combination against a configured table of winning combinations and their payout multipliers — keeping the (likely frequently-tuned, business/compliance-driven) payout rules cleanly separate from the core spin-mechanics logic.
</details>

<details>
<summary>125. How would you design a class structure for a Grading Curve/Statistics utility that computes class rank, percentile, and grade distribution from a set of raw scores?</summary>

A `GradeStatisticsCalculator` operating on a `List<Score>`, computing derived values (mean, standard deviation, percentile rank per student) — a good design keeps this as a stateless utility/service operating on immutable input data and returning a new results structure, rather than mutating the original student/score records directly, since statistical calculations are naturally a "derive new information from existing data" operation rather than a "modify existing entities" operation.
</details>

<details>
<summary>126. How would you design a class structure for a Multi-tenant SaaS application's data-access layer to consistently enforce tenant isolation at the code level (complementing the HLD-level database strategy discussion)?</summary>

A common approach: a `TenantContext` (often backed by a `ThreadLocal`, populated early in the request-handling pipeline, e.g., via a filter/interceptor extracting tenant ID from an auth token) that repository/data-access base classes automatically consult and apply as an implicit filter on every query — centralizing the tenant-isolation enforcement in one well-tested place (a base repository class or a query-interceptor mechanism) rather than relying on every individual query author remembering to manually add a tenant filter condition, which is exactly the kind of "easy to forget, severe consequence" pattern well-suited to enforcing structurally rather than via developer discipline alone.
</details>

<details>
<summary>127. How would you design a class structure for a Configurable Workflow/Approval system (e.g., an expense report requiring different approval chains based on amount)?</summary>

A `WorkflowDefinition` composed of an ordered sequence of `ApprovalStep`s, each with a `condition` (does this step apply, e.g., "only if amount > $1000") and an `approverResolutionStrategy` (who is the actual approver for this step — a specific role, the requester's direct manager, a specific named individual) — a `WorkflowInstance` tracks the current state/position of a specific request moving through its applicable steps, again illustrating the recurring "configurable sequence of conditional steps" shape seen across several other LLD problems in this list (order fulfillment, content moderation).
</details>

<details>
<summary>128. How would you design a class structure for handling Internationalization (i18n) of user-facing text/messages in an application?</summary>

A `MessageResolver`/`Translator` interface abstracting the lookup of a message key + locale to the appropriately localized text (backed by resource bundles, a database, or an external translation service — the specific backing mechanism is an implementation detail hidden behind the interface), with application code always referencing messages by a stable key rather than embedding hardcoded, locale-specific strings directly throughout the codebase — additionally needing to handle locale-aware formatting concerns (dates, numbers, currency, pluralization rules which vary significantly across languages) as a related but distinct concern from pure text translation.
</details>

<details>
<summary>129. How would you design a class structure for an Idempotency-key-based deduplication utility, reusable across multiple different API endpoints in an application?</summary>

An `IdempotencyService` with a `checkAndRecord(idempotencyKey)` method (checking if the key has been seen before, and if not, atomically recording it) — implemented against a shared store with an appropriate TTL (Redis, or a database table with a unique constraint on the key) — wrapped as reusable middleware/interceptor logic (e.g., a Spring `HandlerInterceptor` or a decorator around specific service methods) rather than requiring each individual endpoint's business logic to manually implement the idempotency check itself.
</details>

<details>
<summary>130. How would you design a class structure for a "Saga Orchestrator" (LLD-level implementation of the orchestration-based Saga pattern discussed at the HLD level)?</summary>

A `SagaStep` interface with `execute()` and `compensate()` methods, a `SagaDefinition` holding an ordered sequence of `SagaStep`s, and a `SagaOrchestrator` that executes steps in sequence, and — upon any step's failure — invokes `compensate()` on all *previously successfully completed* steps in reverse order, to unwind the partial transaction — this LLD-level structure gives concrete shape to the more abstract HLD-level Saga pattern discussion, showing how the orchestration logic itself would actually be implemented as composable, testable objects.
</details>

<details>
<summary>131. How would you design a class structure for a generic "Retry with Circuit Breaker" combined resilience wrapper, composing the two patterns together?</summary>

Compose them via the **Decorator pattern** — a base operation is wrapped first by a `CircuitBreakerDecorator` (short-circuiting calls when the downstream is known to be failing) which is itself wrapped by a `RetryDecorator` (retrying transient failures) — or vice versa, depending on desired precedence — since both resilience mechanisms share a common shape (wrapping and intercepting calls to an underlying operation), composing them as stackable decorators around a common functional interface is a natural, reusable structure rather than building one large, monolithic class handling both concerns intertwined.
</details>

<details>
<summary>132. How would you design a class structure for a "Feature Toggle-aware" service that needs to seamlessly switch between an old and new implementation of some business logic during a gradual migration?</summary>

Both the old and new implementations conform to the same interface; a `FeatureToggleRouter`/proxy class implementing that same interface checks the relevant feature flag on each call and delegates to whichever concrete implementation is currently active — this lets the calling code remain entirely unaware of the migration in progress (it just depends on the interface), and the actual cutover between implementations becomes a simple, low-risk configuration change rather than a code deployment.
</details>

<details>
<summary>133. How would you design a class structure for a Distributed Cache Invalidation notification system's client-side handling (LLD-level component within a single service instance)?</summary>

A `CacheInvalidationListener` subscribing to invalidation events (e.g., from a pub/sub channel), maintaining a local reference to the actual cache instance(s) it's responsible for keeping in sync, and reacting to incoming "key X was invalidated" events by evicting that key from the local cache — designed with clear separation between the transport mechanism (how invalidation events actually arrive, an implementation detail behind an interface) and the actual local-cache-eviction logic, so the transport mechanism could be swapped (e.g., from Redis pub/sub to Kafka) without touching the eviction logic itself.
</details>

<details>
<summary>134. How would you design a class structure for a "Plugin Architecture" allowing third parties to extend an application's behavior at well-defined extension points, without modifying the core application code?</summary>

Define clear `Plugin`/`Extension` interfaces representing each well-defined extension point (e.g., a `PaymentPlugin` interface for adding new payment methods, a `ReportExporterPlugin` for new export formats), a plugin discovery/registration mechanism (Java's `ServiceLoader` SPI mechanism, or a manual registry populated at startup), and careful attention to versioning/backward compatibility of these plugin interfaces over time, since breaking changes to a plugin interface would break every third-party plugin built against the previous version.
</details>

<details>
<summary>135. What is the significance of designing plugin/extension interfaces to be as narrow and stable as possible, rather than broad and frequently-changing?</summary>

Every method added to a public extension interface is a commitment that all existing and future plugin implementations must satisfy — a narrow, minimal, carefully-considered interface is much easier to keep stable over time (avoiding breaking third-party integrations) than a broad interface with many methods that seemed convenient to add but weren't truly essential, which becomes much harder to evolve without breaking backward compatibility once external plugins depend on it.
</details>

<details>
<summary>136. How would you design a class structure for a generic "Pipeline" abstraction that processes an input through a configurable sequence of transformation steps (e.g., an image-processing pipeline: resize → watermark → compress)?</summary>

A `PipelineStep<T>` interface with a `process(T input)` method returning transformed output of the same type, and a `Pipeline<T>` holding an ordered list of steps, applying each in sequence (the output of one step becoming the input to the next) — essentially a simplified, generic realization of the Chain-of-Responsibility-adjacent "composable sequence of transformations" shape, reusable across many different processing-pipeline-style problems beyond just image processing.
</details>

<details>
<summary>137. How would you design a class structure for a Weather Station / IoT Sensor data aggregation system's core data model (LLD-level)?</summary>

Core entities: `Sensor` (type, location), `Reading` (a single timestamped measurement from a sensor), and an `AggregationStrategy` interface (`AverageAggregation`, `MaxAggregation`, `MinAggregation`) for computing summary statistics over a set of readings within a time window — designed to accommodate different sensor types with different unit systems/valid ranges (a `Reading`'s validity/plausibility check might reasonably differ meaningfully between a temperature sensor and a humidity sensor, suggesting sensor-type-specific validation logic rather than one-size-fits-all validation).
</details>

<details>
<summary>138. How would you design a class structure for a Kanban Board application supporting configurable columns/workflow states and cards moving between them, with validation of allowed transitions?</summary>

A `Board` holding an ordered collection of `Column`s (each representing a workflow state, e.g., "To Do," "In Progress," "Done"), `Card`s belonging to exactly one column at a time, and a `WorkflowTransitionRule`/`TransitionValidator` governing which column-to-column moves are actually permitted (e.g., disallowing a direct "To Do" → "Done" move if the board's configured workflow requires passing through "In Progress" first) — configurable per board rather than hardcoded, since different teams typically want different workflow rules.
</details>

<details>
<summary>139. How would you design a class structure for handling Soft-delete consistently across many different entity types in an application, avoiding repetitive boilerplate per entity?</summary>

A common `SoftDeletable` interface/base class (with `deletedAt`/`isDeleted` fields and a `softDelete()` method) implemented/extended by any entity needing this behavior, combined with a shared base repository class (or ORM-level filtering hook, like Hibernate's `@Where` or a JPA entity listener) that automatically excludes soft-deleted records from standard queries by default — centralizing this cross-cutting concern rather than requiring every single entity's repository to remember to manually add a `WHERE deleted_at IS NULL` condition to every query.
</details>

<details>
<summary>140. How would you design a class structure for a "Command Line Interface (CLI) Argument Parser" supporting different argument types (flags, options with values, positional arguments) as an LLD exercise?</summary>

An `Argument` abstract base (or interface) with `Flag` (boolean, present or not), `Option` (a named argument requiring a value), and `PositionalArgument` (order-dependent, unnamed) subtypes, registered with a `CommandLineParser` that, given raw input tokens, matches and populates each registered argument definition, and validates required arguments were actually provided — a design consideration is how to cleanly report multiple validation errors at once (e.g., several missing required arguments) rather than failing and reporting only the first error encountered, which is generally a better user experience for a CLI tool.
</details>

<details>
<summary>141. How would you design a class structure for a Rate Card / Pricing Tier system where pricing depends on usage volume with different rates at different tiers (e.g., first 1000 units at $0.10, next 9000 at $0.08, beyond that at $0.05)?</summary>

A `PricingTier` value object (a range boundary + a rate), a `TieredPricingStrategy` holding an ordered list of `PricingTier`s and a `calculateCost(usage)` method that iterates through the tiers, applying each tier's rate to the portion of usage falling within that tier's range, and summing the results — this "graduated/tiered" calculation logic is a genuinely common, slightly tricky recurring business calculation worth having a clean, well-tested, reusable implementation pattern for, since off-by-one boundary errors are an easy mistake in a naive implementation.
</details>

<details>
<summary>142. How would you design a class structure for a "Health Check Aggregator" component (LLD-level) that combines the status of multiple independent dependency health checks into one overall system health status?</summary>

A `HealthIndicator` interface (implemented per dependency — database, cache, external API) with a `check()` method returning a status (Up/Down/Degraded) plus optional details, and a `HealthAggregator` running all registered indicators and combining their individual results into one overall status (typically: overall status is the "worst" individual status among all checks, e.g., any single "Down" indicator makes the overall system status "Down") — designed so new dependency health checks can be added simply by implementing and registering a new `HealthIndicator`, without modifying the aggregation logic itself.
</details>

<details>
<summary>143. How would you design a class structure for a "Search Query Builder" that lets calling code fluently construct complex search queries programmatically (e.g., `.where("status").equals("active").and("price").greaterThan(100)`)?</summary>

A fluent `QueryBuilder` class where each method call returns `this` (or an intermediate builder state object) to enable chaining, internally accumulating a structured representation of the query conditions (e.g., a tree/list of `Condition` objects with field, operator, value, and logical connectors), culminating in a `build()` method that translates the accumulated structure into the actual underlying query format (SQL, an Elasticsearch query DSL, etc.) — cleanly separating the ergonomic, fluent API surface from the underlying query-language-specific translation logic.
</details>

<details>
<summary>144. How would you design a class structure for a "Dependency Graph" resolver that determines a safe build/deployment order for a set of interdependent modules/services?</summary>

Model modules and their dependencies as a directed graph (each module has a list of modules it depends on), and compute a valid build order via **topological sort** — the same underlying graph algorithm discussed earlier for the Task Management dependency problem, reinforcing that recognizing "this is fundamentally a dependency-graph/topological-sort problem" is a broadly reusable insight across many different-sounding LLD problems (task scheduling, build systems, this module-ordering problem) that share the same underlying graph-theoretic structure.
</details>

<details>
<summary>145. What is a good way to identify, during an LLD interview, whether a given sub-problem is fundamentally "just" a well-known data structure/algorithm problem in disguise (like topological sort, or an LRU cache's underlying structure)?</summary>

Practice recognizing the *underlying shape* of a requirement independent of its specific domain framing — "must process items respecting dependency/prerequisite ordering" is topological sort regardless of whether it's framed as tasks, course prerequisites, or build modules; "must efficiently track and evict least-recently-used items" is the LRU cache pattern regardless of whether it's framed as a web cache or a database buffer pool — this pattern-recognition skill, built through exposure to many varied problems, is often more valuable in an LLD interview than memorizing solutions to specific named problems, since real interview questions are frequently just familiar underlying patterns wrapped in an unfamiliar domain description.
</details>

<details>
<summary>146. How would you design a class structure for a "Notification Batching" system that groups multiple individual notification events for the same user into a single digest notification if they occur within a short time window?</summary>

A `NotificationBatcher` that, instead of immediately dispatching each incoming notification event, buffers events per user (e.g., in a time-windowed collection keyed by user ID) and, after a configured debounce/batching window elapses since the first buffered event for that user (or once a maximum batch size is reached), flushes the accumulated events as a single combined digest notification — requiring careful handling of the timing mechanism (a scheduled task checking for windows that have elapsed, or a per-user delayed/debounced trigger) to balance batching effectiveness against not delaying time-sensitive notifications indefinitely.
</details>

<details>
<summary>147. How would you design a class structure for a "Permission Delegation" system where a user can temporarily grant a subset of their own permissions to another user for a limited time (e.g., "out of office" delegation)?</summary>

A `Delegation` entity (delegator, delegate, specific permissions or full permission set being delegated, start/end validity window), consulted by the permission-checking logic alongside a user's own directly-assigned permissions — checking "does user X have permission Y" needs to check both X's own direct permissions AND any currently-active delegation granting X permission Y from someone else, with careful attention to correctly expiring/ignoring delegations outside their validity window and avoiding unbounded, transitive delegation chains (can a delegate further re-delegate?) unless that's an explicitly intended, carefully-scoped feature.
</details>

<details>
<summary>148. How would you design a class structure for an "Inventory Reservation" system that temporarily holds stock during a checkout process (distinct from the final, committed stock decrement), avoiding both overselling and permanently locking stock for abandoned carts?</summary>

A `StockReservation` entity (product, quantity, expiry timestamp, associated cart/order), separate from the actual committed `Inventory` count — reserving stock decrements an "available" count (but not yet the "committed sold" count) and creates a time-bounded reservation record; a background process (or lazy check-on-access) releases expired, uncompleted reservations back to available stock — this three-state model (total, reserved, available) is a recurring, genuinely important pattern for any e-commerce-style inventory system needing to balance preventing overselling against not permanently locking up stock for carts that are simply abandoned.
</details>

<details>
<summary>149. How would you design a class structure for a "Configurable Validation Pipeline" for incoming API requests, where different endpoints need different combinations of validation rules applied in a specific order?</summary>

Building on the earlier generic Validation Framework design, add explicit **ordering** and **short-circuit** semantics — a `ValidationPipeline` holding an ordered list of `ValidationRule`s specific to a given endpoint/request type, executing them in sequence and (typically) stopping at the first failure for validations that are prerequisites for subsequent ones (e.g., "field is present" must pass before "field matches expected format" is even meaningful to check), while potentially still collecting multiple independent failures for rules that aren't sequentially dependent on each other — a nuanced middle ground between always-stop-at-first-failure and always-collect-everything.
</details>

<details>
<summary>150. If asked to reflect on "what makes a really strong LLD interview answer stand out from an average one," what would you highlight?</summary>

A strong answer clarifies requirements and scope before diving into a class diagram (rather than assuming); chooses abstractions/interfaces genuinely justified by the stated (or reasonably foreseeable) requirements rather than pattern-name-dropping for its own sake; correctly applies core OOP principles (SOLID) without needing to be explicitly prompted for each one; recognizes when a sub-problem maps to a well-known underlying data structure/algorithm pattern; proactively identifies and discusses genuine trade-offs and edge cases (concurrency, validation boundaries) rather than only presenting a happy-path design; and can fluently extend the design when the interviewer introduces a new requirement mid-interview, demonstrating the abstractions chosen were actually sound rather than accidentally convenient only for the originally-stated scope.
</details>
