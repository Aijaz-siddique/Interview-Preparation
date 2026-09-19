# Design Patterns & Design Principles — Java (Elaborative, with Code)

A deep-dive reference covering the core object-oriented design principles (SOLID and beyond) and all 23 Gang-of-Four design patterns, each with a full explanation, when to use it, and a complete, runnable Java example.

## Table of Contents

**Design Principles**
1. [Single Responsibility Principle](#1-single-responsibility-principle-srp)
2. [Open/Closed Principle](#2-openclosed-principle-ocp)
3. [Liskov Substitution Principle](#3-liskov-substitution-principle-lsp)
4. [Interface Segregation Principle](#4-interface-segregation-principle-isp)
5. [Dependency Inversion Principle](#5-dependency-inversion-principle-dip)
6. [DRY](#6-dry-dont-repeat-yourself)
7. [KISS](#7-kiss-keep-it-simple-stupid)
8. [YAGNI](#8-yagni-you-arent-gonna-need-it)
9. [Law of Demeter](#9-law-of-demeter-principle-of-least-knowledge)
10. [Composition Over Inheritance](#10-composition-over-inheritance)
11. [Program to an Interface, Not an Implementation](#11-program-to-an-interface-not-an-implementation)

**Creational Patterns:** [Singleton](#12-singleton) · [Factory Method](#13-factory-method) · [Abstract Factory](#14-abstract-factory) · [Builder](#15-builder) · [Prototype](#16-prototype)

**Structural Patterns:** [Adapter](#17-adapter) · [Bridge](#18-bridge) · [Composite](#19-composite) · [Decorator](#20-decorator) · [Facade](#21-facade) · [Flyweight](#22-flyweight) · [Proxy](#23-proxy)

**Behavioral Patterns:** [Chain of Responsibility](#24-chain-of-responsibility) · [Command](#25-command) · [Interpreter](#26-interpreter) · [Iterator](#27-iterator) · [Mediator](#28-mediator) · [Memento](#29-memento) · [Observer](#30-observer) · [State](#31-state) · [Strategy](#32-strategy) · [Template Method](#33-template-method) · [Visitor](#34-visitor)

---

# Design Principles

## 1. Single Responsibility Principle (SRP)

**Definition:** A class should have only one reason to change — one job, answering to one stakeholder/axis of change.

**Violation:**

```java
// BAD: three reasons to change — calculation rules, persistence, and print formatting.
class Invoice {
    private List<Item> items;

    public double calculateTotal() {
        return items.stream().mapToDouble(Item::getPrice).sum();
    }

    public void saveToDatabase() {
        System.out.println("Saving invoice to database...");
    }

    public void printInvoice() {
        System.out.println("Printing invoice with total: " + calculateTotal());
    }
}
```

**Refactored:**

```java
class Invoice {
    private final List<Item> items;
    public Invoice(List<Item> items) { this.items = items; }

    public double calculateTotal() {
        return items.stream().mapToDouble(Item::getPrice).sum();
    }
    public List<Item> getItems() { return items; }
}

class InvoiceRepository { // responsible only for persistence
    public void save(Invoice invoice) {
        System.out.println("Saving invoice... total=" + invoice.calculateTotal());
    }
}

class InvoicePrinter { // responsible only for presentation
    public void print(Invoice invoice) {
        invoice.getItems().forEach(i -> System.out.println(i.getName() + " - $" + i.getPrice()));
        System.out.println("Total: $" + invoice.calculateTotal());
    }
}

class Item {
    private final String name; private final double price;
    public Item(String name, double price) { this.name = name; this.price = price; }
    public String getName() { return name; }
    public double getPrice() { return price; }
}
```

A change to *how* invoices are persisted only touches `InvoiceRepository`; a change to print formatting only touches `InvoicePrinter`.

---

## 2. Open/Closed Principle (OCP)

**Definition:** Classes should be **open for extension, closed for modification** — add new behavior via new code, not by editing existing, tested code.

**Violation:**

```java
class AreaCalculator {
    public double calculateArea(Object shape) {
        if (shape instanceof Circle c) {
            return Math.PI * c.getRadius() * c.getRadius();
        } else if (shape instanceof Rectangle r) {
            return r.getWidth() * r.getHeight();
        }
        throw new IllegalArgumentException("Unknown shape"); // grows with every new shape
    }
}
```

**Refactored:**

```java
interface Shape { double calculateArea(); }

class Circle implements Shape {
    private final double radius;
    public Circle(double radius) { this.radius = radius; }
    public double calculateArea() { return Math.PI * radius * radius; }
}

class Rectangle implements Shape {
    private final double width, height;
    public Rectangle(double w, double h) { width = w; height = h; }
    public double calculateArea() { return width * height; }
}

// New shape added with ZERO changes to existing code.
class Triangle implements Shape {
    private final double base, height;
    public Triangle(double b, double h) { base = b; height = h; }
    public double calculateArea() { return 0.5 * base * height; }
}

class AreaCalculator {
    public double totalArea(List<Shape> shapes) {
        return shapes.stream().mapToDouble(Shape::calculateArea).sum();
    }
}
```

---

## 3. Liskov Substitution Principle (LSP)

**Definition:** Subtypes must be substitutable for their base type without breaking correctness — honor the behavioral contract, not just the method signature.

**Violation (classic Square/Rectangle):**

```java
class Rectangle {
    protected int width, height;
    public void setWidth(int w) { width = w; }
    public void setHeight(int h) { height = h; }
    public int getArea() { return width * height; }
}

// BAD: forcing width==height breaks callers' assumptions about Rectangle.
class Square extends Rectangle {
    @Override public void setWidth(int w) { width = w; height = w; }
    @Override public void setHeight(int h) { width = h; height = h; }
}

class Demo {
    static void resize(Rectangle r) {
        r.setWidth(5); r.setHeight(10);
        assert r.getArea() == 50; // FAILS for Square — not substitutable!
    }
}
```

**Fixed (no forced, behaviorally-incompatible inheritance):**

```java
interface Shape { int getArea(); }

class Rectangle implements Shape {
    private final int width, height;
    public Rectangle(int w, int h) { width = w; height = h; }
    public int getArea() { return width * height; }
}

class Square implements Shape {
    private final int side;
    public Square(int side) { this.side = side; }
    public int getArea() { return side * side; }
}
```

---

## 4. Interface Segregation Principle (ISP)

**Definition:** Don't force clients to depend on methods they don't use — prefer several small interfaces over one fat one.

**Violation:**

```java
interface Worker { void work(); void eat(); }

class RobotWorker implements Worker {
    public void work() { System.out.println("Robot working"); }
    public void eat() { throw new UnsupportedOperationException("Robots don't eat!"); }
}
```

**Refactored:**

```java
interface Workable { void work(); }
interface Eatable { void eat(); }

class HumanWorker implements Workable, Eatable {
    public void work() { System.out.println("Human working"); }
    public void eat() { System.out.println("Human eating lunch"); }
}

class RobotWorker implements Workable { // only implements what applies to it
    public void work() { System.out.println("Robot working"); }
}
```

---

## 5. Dependency Inversion Principle (DIP)

**Definition:** High-level modules and low-level modules should both depend on **abstractions**, not on each other directly.

**Violation:**

```java
class EmailSender {
    public void sendEmail(String msg) { System.out.println("Emailing: " + msg); }
}

class NotificationService {
    private final EmailSender emailSender = new EmailSender(); // tightly coupled!
    public void notifyUser(String msg) { emailSender.sendEmail(msg); }
}
```

**Refactored:**

```java
interface MessageSender { void send(String message); }

class EmailSender implements MessageSender {
    public void send(String message) { System.out.println("Emailing: " + message); }
}

class SmsSender implements MessageSender {
    public void send(String message) { System.out.println("Texting: " + message); }
}

class NotificationService {
    private final MessageSender messageSender;
    public NotificationService(MessageSender messageSender) { // constructor injection
        this.messageSender = messageSender;
    }
    public void notifyUser(String message) { messageSender.send(message); }
}

class DipDemo {
    public static void main(String[] args) {
        new NotificationService(new EmailSender()).notifyUser("Order shipped!");
        new NotificationService(new SmsSender()).notifyUser("Your OTP is 4821");
    }
}
```

This is the exact mechanism that makes Spring's dependency injection possible.

---

## 6. DRY (Don't Repeat Yourself)

**Definition:** Every piece of logic should have a single, authoritative representation — no duplicated business rules.

**Violation:**

```java
class OrderService {
    public double forRegular(double price) {
        double discount = price * 0.05;
        double tax = (price - discount) * 0.08;
        return price - discount + tax;
    }
    public double forPremium(double price) {
        double discount = price * 0.15; // same formula, duplicated
        double tax = (price - discount) * 0.08;
        return price - discount + tax;
    }
}
```

**Refactored:**

```java
class OrderService {
    private static final double TAX_RATE = 0.08;

    private double calculateFinalPrice(double price, double discountRate) {
        double discount = price * discountRate;
        double afterDiscount = price - discount;
        return afterDiscount + (afterDiscount * TAX_RATE);
    }

    public double forRegular(double price) { return calculateFinalPrice(price, 0.05); }
    public double forPremium(double price) { return calculateFinalPrice(price, 0.15); }
}
```

---

## 7. KISS (Keep It Simple, Stupid)

**Definition:** Avoid unnecessary complexity — solve the problem as simply as it actually requires.

**Violation (absurdly over-engineered):**

```java
interface EvenCheckStrategy { boolean isEven(int n); }
class ModuloEvenCheckStrategy implements EvenCheckStrategy {
    public boolean isEven(int n) { return n % 2 == 0; }
}
class EvenCheckStrategyFactory {
    public static EvenCheckStrategy create() { return new ModuloEvenCheckStrategy(); }
}
class EvenChecker {
    private final EvenCheckStrategy strategy = EvenCheckStrategyFactory.create();
    public boolean check(int n) { return strategy.isEven(n); }
}
```

**KISS-compliant:**

```java
class EvenChecker {
    public boolean isEven(int number) { return number % 2 == 0; }
}
```

---

## 8. YAGNI (You Aren't Gonna Need It)

**Definition:** Don't build speculative functionality "just in case" — build only what's actually required now.

**Violation:**

```java
class ReportGenerator {
    // Handles formats/languages/currencies/plugins nobody has asked for yet.
    public String generate(Report r, String format, String language, String currency,
                            boolean legacyEngine, Map<String, Object> pluginOptions) {
        return "...";
    }
}
```

**YAGNI-compliant:**

```java
class ReportGenerator {
    public String generate(Report report) {
        return "Report: " + report.getTitle() + "\n" + report.getContent();
    }
}
// Add currency/language support later, IF a real requirement demands it.
```

---

## 9. Law of Demeter (Principle of Least Knowledge)

**Definition:** Only talk to immediate "friends" — don't reach through one object to access another nested object.

**Violation ("train wreck"):**

```java
class Engine { public void start() { System.out.println("Engine started"); } }
class Car {
    private final Engine engine = new Engine();
    public Engine getEngine() { return engine; }
}
class Driver {
    private final Car car;
    public Driver(Car car) { this.car = car; }
    public Car getCar() { return car; }
}

class Demo {
    void startDriving(Driver driver) {
        driver.getCar().getEngine().start(); // BAD: reaches through 2 layers
    }
}
```

**Refactored:**

```java
class Engine { public void start() { System.out.println("Engine started"); } }
class Car {
    private final Engine engine = new Engine();
    public void start() { engine.start(); } // exposes behavior, not internals
}
class Driver {
    private final Car car;
    public Driver(Car car) { this.car = car; }
    public void drive() { car.start(); }
}

class Demo {
    void startDriving(Driver driver) { driver.drive(); } // talks only to its direct friend
}
```

---

## 10. Composition Over Inheritance

**Definition:** Prefer building behavior via composed objects (has-a) over rigid class inheritance (is-a).

**Violation:**

```java
abstract class Bird { abstract void fly(); }
class Duck extends Bird { void fly() { System.out.println("Duck flying"); } }
class Penguin extends Bird {
    void fly() { throw new UnsupportedOperationException("Penguins can't fly"); } // broken override
}
```

**Refactored:**

```java
interface FlyBehavior { void fly(); }
interface SwimBehavior { void swim(); }

class FlyWithWings implements FlyBehavior { public void fly() { System.out.println("Flying"); } }
class CannotFly implements FlyBehavior { public void fly() { System.out.println("Can't fly"); } }
class SwimNormally implements SwimBehavior { public void swim() { System.out.println("Swimming"); } }

class Bird {
    private final FlyBehavior flyBehavior;
    private final SwimBehavior swimBehavior;
    public Bird(FlyBehavior f, SwimBehavior s) { flyBehavior = f; swimBehavior = s; }
    public void performFly() { flyBehavior.fly(); }
    public void performSwim() { swimBehavior.swim(); }
}

class Demo {
    public static void main(String[] args) {
        Bird duck = new Bird(new FlyWithWings(), new SwimNormally());
        Bird penguin = new Bird(new CannotFly(), new SwimNormally());
        duck.performFly();     // Flying
        penguin.performFly();  // Can't fly — no exception, no broken override
    }
}
```

---

## 11. Program to an Interface, Not an Implementation

**Definition:** Depend on abstract types wherever a variation point might exist — the foundation underlying DIP and nearly every GoF pattern.

**Violation:**

```java
class ShoppingCart {
    private ArrayList<String> items = new ArrayList<>(); // tied to a concrete type
    public void addItem(String item) { items.add(item); }
}
```

**Refactored:**

```java
class ShoppingCart {
    private final List<String> items; // depends on the abstraction
    public ShoppingCart(List<String> items) { this.items = items; }
    public void addItem(String item) { items.add(item); }
}

class Demo {
    public static void main(String[] args) {
        ShoppingCart cart1 = new ShoppingCart(new ArrayList<>());  // fast random access
        ShoppingCart cart2 = new ShoppingCart(new LinkedList<>()); // fast insert/remove
        // ShoppingCart's logic never changes regardless of which is used.
    }
}
```

---

# Creational Patterns

Creational patterns deal with object creation — abstracting away *how* objects are instantiated so the system isn't tightly coupled to concrete classes.

## 12. Singleton

**Intent:** Ensure a class has only one instance, and provide a single, global point of access to it.

**Use when:** You need exactly one instance coordinating actions across a system — a configuration manager, a connection pool, a logger.

**Thread-safe implementation using an enum (the recommended, simplest, and safest approach):**

```java
enum ConfigurationManager {
    INSTANCE;

    private final Map<String, String> settings = new HashMap<>();

    public void set(String key, String value) { settings.put(key, value); }
    public String get(String key) { return settings.get(key); }
}

class SingletonDemo {
    public static void main(String[] args) {
        ConfigurationManager.INSTANCE.set("env", "production");
        System.out.println(ConfigurationManager.INSTANCE.get("env")); // production

        // The JVM guarantees only one instance ever exists, even under
        // concurrent access, and it's automatically serialization-safe.
    }
}
```

**Double-checked locking implementation (the classic approach, more verbose):**

```java
class DatabaseConnectionPool {
    // volatile is ESSENTIAL here — without it, another thread could see a
    // partially-constructed instance due to instruction reordering.
    private static volatile DatabaseConnectionPool instance;

    private DatabaseConnectionPool() {
        System.out.println("Expensive connection pool initialized");
    }

    public static DatabaseConnectionPool getInstance() {
        if (instance == null) {                          // first check (no locking, fast path)
            synchronized (DatabaseConnectionPool.class) {
                if (instance == null) {                   // second check (inside the lock)
                    instance = new DatabaseConnectionPool();
                }
            }
        }
        return instance;
    }
}
```

The enum approach is generally preferred in modern Java — it's inherently thread-safe, immune to reflection-based instantiation attacks, and correctly handles serialization automatically, all without needing to reason about `volatile`/locking manually.

---

## 13. Factory Method

**Intent:** Define an interface for creating an object, but let subclasses decide which class to instantiate.

**Use when:** A class can't anticipate the exact type of object it needs to create ahead of time, or you want to delegate instantiation logic to subclasses.

```java
// Product hierarchy
interface Notification {
    void notifyUser(String message);
}

class EmailNotification implements Notification {
    public void notifyUser(String message) { System.out.println("Email: " + message); }
}

class SmsNotification implements Notification {
    public void notifyUser(String message) { System.out.println("SMS: " + message); }
}

class PushNotification implements Notification {
    public void notifyUser(String message) { System.out.println("Push: " + message); }
}

// Creator declares the factory method
abstract class NotificationCreator {
    // The factory method — subclasses decide the concrete type.
    protected abstract Notification createNotification();

    // Shared logic that uses the product, unaware of its concrete type.
    public void send(String message) {
        Notification notification = createNotification();
        notification.notifyUser(message);
    }
}

class EmailNotificationCreator extends NotificationCreator {
    protected Notification createNotification() { return new EmailNotification(); }
}

class SmsNotificationCreator extends NotificationCreator {
    protected Notification createNotification() { return new SmsNotification(); }
}

class FactoryMethodDemo {
    public static void main(String[] args) {
        NotificationCreator creator = new EmailNotificationCreator();
        creator.send("Your order has shipped!"); // Email: Your order has shipped!

        creator = new SmsNotificationCreator();
        creator.send("Your OTP is 4821");        // SMS: Your OTP is 4821
    }
}
```

A simpler, static-factory-method variant (very common in real Java code):

```java
class NotificationFactory {
    public static Notification create(String type) {
        return switch (type) {
            case "EMAIL" -> new EmailNotification();
            case "SMS" -> new SmsNotification();
            case "PUSH" -> new PushNotification();
            default -> throw new IllegalArgumentException("Unknown type: " + type);
        };
    }
}
```

---

## 14. Abstract Factory

**Intent:** Provide an interface for creating **families** of related objects without specifying their concrete classes.

**Use when:** A system needs to be configured with one of several families of related products, and you need to ensure products from the same family are used together consistently.

```java
// Product families: Button and Checkbox, each with two "themes" (Light/Dark)
interface Button { void render(); }
interface Checkbox { void render(); }

class LightButton implements Button {
    public void render() { System.out.println("Rendering light button"); }
}
class DarkButton implements Button {
    public void render() { System.out.println("Rendering dark button"); }
}
class LightCheckbox implements Checkbox {
    public void render() { System.out.println("Rendering light checkbox"); }
}
class DarkCheckbox implements Checkbox {
    public void render() { System.out.println("Rendering dark checkbox"); }
}

// Abstract Factory: creates a whole FAMILY of related products
interface UIComponentFactory {
    Button createButton();
    Checkbox createCheckbox();
}

class LightThemeFactory implements UIComponentFactory {
    public Button createButton() { return new LightButton(); }
    public Checkbox createCheckbox() { return new LightCheckbox(); }
}

class DarkThemeFactory implements UIComponentFactory {
    public Button createButton() { return new DarkButton(); }
    public Checkbox createCheckbox() { return new DarkCheckbox(); }
}

class Application {
    private final Button button;
    private final Checkbox checkbox;

    // The application depends only on the abstract factory — it can never
    // accidentally mix a LightButton with a DarkCheckbox.
    public Application(UIComponentFactory factory) {
        this.button = factory.createButton();
        this.checkbox = factory.createCheckbox();
    }

    public void render() {
        button.render();
        checkbox.render();
    }
}

class AbstractFactoryDemo {
    public static void main(String[] args) {
        Application darkApp = new Application(new DarkThemeFactory());
        darkApp.render(); // Rendering dark button / Rendering dark checkbox — always consistent
    }
}
```

**Key distinction from Factory Method:** Factory Method creates **one** product via subclassing/overriding a single method. Abstract Factory creates **families of related products** (multiple factory methods bundled into one interface), guaranteeing the products used together are always from the same, compatible family.

---

## 15. Builder

**Intent:** Separate the construction of a complex object from its representation, allowing step-by-step construction and avoiding constructors with a huge number of (often optional) parameters.

**Use when:** An object has many optional fields/configuration, and you want a readable, fluent, and safe way to construct it.

```java
class HttpRequest {
    // All fields are final — the object is immutable once built.
    private final String url;
    private final String method;
    private final Map<String, String> headers;
    private final String body;
    private final int timeoutMs;

    // Private constructor — only the Builder can create instances.
    private HttpRequest(Builder builder) {
        this.url = builder.url;
        this.method = builder.method;
        this.headers = builder.headers;
        this.body = builder.body;
        this.timeoutMs = builder.timeoutMs;
    }

    @Override
    public String toString() {
        return method + " " + url + " headers=" + headers + " timeout=" + timeoutMs + "ms";
    }

    // Static nested Builder class
    static class Builder {
        private final String url;         // required
        private String method = "GET";    // sensible default
        private final Map<String, String> headers = new HashMap<>();
        private String body;
        private int timeoutMs = 5000;

        public Builder(String url) {      // required parameter enforced in the constructor
            this.url = url;
        }

        public Builder method(String method) { this.method = method; return this; }
        public Builder header(String key, String value) { this.headers.put(key, value); return this; }
        public Builder body(String body) { this.body = body; return this; }
        public Builder timeout(int timeoutMs) { this.timeoutMs = timeoutMs; return this; }

        public HttpRequest build() {
            if (method.equals("POST") && body == null) {
                throw new IllegalStateException("POST requests require a body"); // validate before building
            }
            return new HttpRequest(this);
        }
    }
}

class BuilderDemo {
    public static void main(String[] args) {
        HttpRequest request = new HttpRequest.Builder("https://api.example.com/orders")
            .method("POST")
            .header("Content-Type", "application/json")
            .header("Authorization", "Bearer token123")
            .body("{\"item\":\"widget\"}")
            .timeout(10000)
            .build();

        System.out.println(request);
    }
}
```

Notice how this avoids a constructor like `new HttpRequest(url, "POST", headers, body, 10000, null, false, ...)` where the meaning of each positional argument is unclear at the call site.

---

## 16. Prototype

**Intent:** Create new objects by copying an existing, pre-configured instance (a "prototype") rather than constructing from scratch — especially useful when object creation is expensive.

**Use when:** Creating an object is costly (loading resources, complex initialization) and you need many similar-but-slightly-different instances.

```java
interface Prototype<T> {
    T clone();
}

class GameCharacter implements Prototype<GameCharacter> {
    private String name;
    private int health;
    private int mana;
    private List<String> inventory; // mutable, reference-type field — needs a DEEP copy

    public GameCharacter(String name, int health, int mana, List<String> inventory) {
        this.name = name;
        this.health = health;
        this.mana = mana;
        this.inventory = inventory;
        System.out.println("Expensive character initialization for: " + name);
    }

    // Deep clone — copies the inventory list too, not just the reference.
    @Override
    public GameCharacter clone() {
        return new GameCharacter(this.name, this.health, this.mana, new ArrayList<>(this.inventory));
    }

    public void setName(String name) { this.name = name; }
    public void addItem(String item) { inventory.add(item); }

    @Override
    public String toString() {
        return name + " [HP=" + health + ", MP=" + mana + ", inventory=" + inventory + "]";
    }
}

class PrototypeDemo {
    public static void main(String[] args) {
        // Expensive base template, created once.
        GameCharacter warriorTemplate = new GameCharacter(
            "Warrior Template", 100, 20, new ArrayList<>(List.of("Sword", "Shield")));

        // Cheap clones — no re-running the expensive initialization logic.
        GameCharacter player1 = warriorTemplate.clone();
        player1.setName("Aragorn");
        player1.addItem("Ring of Power");

        GameCharacter player2 = warriorTemplate.clone();
        player2.setName("Boromir");

        System.out.println(player1); // Aragorn [..., inventory=[Sword, Shield, Ring of Power]]
        System.out.println(player2); // Boromir [..., inventory=[Sword, Shield]] — unaffected by player1's changes
    }
}
```

The critical detail: `clone()` performs a **deep** copy of the mutable `inventory` list — a shallow copy (just copying the list reference) would mean `player1.addItem(...)` would incorrectly also affect `player2`'s inventory, since they'd share the same underlying list.

---

# Structural Patterns

Structural patterns deal with how classes/objects are composed to form larger, more flexible structures while keeping those structures efficient and maintainable.

## 17. Adapter

**Intent:** Convert the interface of a class into another interface clients expect, letting incompatible interfaces work together without modifying either side.

**Use when:** You need to integrate an existing class (often third-party, unmodifiable) whose interface doesn't match what your code expects.

```java
// Your application's expected interface
interface PaymentProcessor {
    void processPayment(double amountInDollars);
}

// A third-party library you can't modify, with an incompatible interface
class LegacyPaymentGateway {
    public void makeTransaction(int amountInCents, String currencyCode) {
        System.out.println("Legacy gateway processing " + amountInCents + " " + currencyCode);
    }
}

// Adapter: translates calls from your interface to the legacy library's interface
class LegacyPaymentAdapter implements PaymentProcessor {
    private final LegacyPaymentGateway legacyGateway;

    public LegacyPaymentAdapter(LegacyPaymentGateway legacyGateway) {
        this.legacyGateway = legacyGateway;
    }

    @Override
    public void processPayment(double amountInDollars) {
        int amountInCents = (int) Math.round(amountInDollars * 100);
        legacyGateway.makeTransaction(amountInCents, "USD"); // translates the call
    }
}

class AdapterDemo {
    public static void main(String[] args) {
        PaymentProcessor processor = new LegacyPaymentAdapter(new LegacyPaymentGateway());
        processor.processPayment(49.99); // Legacy gateway processing 4999 USD
        // Your checkout code only ever talks to PaymentProcessor — never to the legacy class directly.
    }
}
```

---

## 18. Bridge

**Intent:** Decouple an abstraction from its implementation so the two can vary independently — avoiding a combinatorial explosion of subclasses when a class has two independent dimensions of variation.

**Use when:** You have a class hierarchy that would otherwise need to grow along two independent axes (e.g., shape type × rendering engine).

```java
// Implementation hierarchy (the "how")
interface Renderer {
    void renderCircle(float radius);
}

class VectorRenderer implements Renderer {
    public void renderCircle(float radius) {
        System.out.println("Drawing a vector circle of radius " + radius);
    }
}

class RasterRenderer implements Renderer {
    public void renderCircle(float radius) {
        System.out.println("Drawing pixels for a circle of radius " + radius);
    }
}

// Abstraction hierarchy (the "what") — holds a reference to the implementation (the BRIDGE)
abstract class Shape {
    protected Renderer renderer; // the bridge to the implementation
    protected Shape(Renderer renderer) { this.renderer = renderer; }
    public abstract void draw();
}

class Circle extends Shape {
    private final float radius;
    public Circle(Renderer renderer, float radius) {
        super(renderer);
        this.radius = radius;
    }
    public void draw() { renderer.renderCircle(radius); }
}

class BridgeDemo {
    public static void main(String[] args) {
        Shape vectorCircle = new Circle(new VectorRenderer(), 5);
        Shape rasterCircle = new Circle(new RasterRenderer(), 5);

        vectorCircle.draw(); // Drawing a vector circle of radius 5.0
        rasterCircle.draw(); // Drawing pixels for a circle of radius 5.0
        // Adding a Square shape, or a new HighDefRenderer, requires NO combinatorial
        // explosion of classes like VectorCircle/RasterCircle/VectorSquare/RasterSquare...
    }
}
```

---

## 19. Composite

**Intent:** Compose objects into tree structures to represent part-whole hierarchies, letting clients treat individual objects and compositions of objects uniformly.

**Use when:** You have a tree-like structure (a file system, a UI component tree, an org chart) and want operations to work identically on a single leaf or an entire subtree.

```java
// Common interface for both individual files and directories (composites)
interface FileSystemEntry {
    long getSize();
    void print(String indent);
}

class File implements FileSystemEntry { // leaf
    private final String name;
    private final long size;

    public File(String name, long size) { this.name = name; this.size = size; }

    public long getSize() { return size; }
    public void print(String indent) { System.out.println(indent + "- " + name + " (" + size + " bytes)"); }
}

class Directory implements FileSystemEntry { // composite
    private final String name;
    private final List<FileSystemEntry> children = new ArrayList<>();

    public Directory(String name) { this.name = name; }
    public void add(FileSystemEntry entry) { children.add(entry); }

    @Override
    public long getSize() {
        // Recursively sums children — works identically whether a child is a File or a Directory.
        return children.stream().mapToLong(FileSystemEntry::getSize).sum();
    }

    @Override
    public void print(String indent) {
        System.out.println(indent + "+ " + name + "/");
        for (FileSystemEntry child : children) {
            child.print(indent + "  ");
        }
    }
}

class CompositeDemo {
    public static void main(String[] args) {
        Directory root = new Directory("project");
        Directory src = new Directory("src");
        src.add(new File("Main.java", 1200));
        src.add(new File("Utils.java", 800));
        root.add(src);
        root.add(new File("README.md", 300));

        root.print("");                              // prints the whole tree, recursively
        System.out.println("Total size: " + root.getSize()); // 2300 — sums the whole subtree
    }
}
```

---

## 20. Decorator

**Intent:** Attach additional responsibilities to an object dynamically, providing a flexible alternative to subclassing for extending behavior.

**Use when:** You want to add optional, combinable behaviors to an object at runtime, without a combinatorial explosion of subclasses.

```java
interface Coffee {
    double getCost();
    String getDescription();
}

class PlainCoffee implements Coffee {
    public double getCost() { return 2.00; }
    public String getDescription() { return "Coffee"; }
}

// Base decorator — implements the same interface, wraps another Coffee
abstract class CoffeeDecorator implements Coffee {
    protected final Coffee decoratedCoffee;
    protected CoffeeDecorator(Coffee coffee) { this.decoratedCoffee = coffee; }
}

class MilkDecorator extends CoffeeDecorator {
    public MilkDecorator(Coffee coffee) { super(coffee); }
    public double getCost() { return decoratedCoffee.getCost() + 0.50; }
    public String getDescription() { return decoratedCoffee.getDescription() + " + Milk"; }
}

class CaramelDecorator extends CoffeeDecorator {
    public CaramelDecorator(Coffee coffee) { super(coffee); }
    public double getCost() { return decoratedCoffee.getCost() + 0.75; }
    public String getDescription() { return decoratedCoffee.getDescription() + " + Caramel"; }
}

class DecoratorDemo {
    public static void main(String[] args) {
        // Dynamically compose any combination of add-ons — no MilkCaramelCoffee subclass needed.
        Coffee order = new CaramelDecorator(new MilkDecorator(new PlainCoffee()));

        System.out.println(order.getDescription() + " = $" + order.getCost());
        // Coffee + Milk + Caramel = $3.25
    }
}
```

---

## 21. Facade

**Intent:** Provide a simplified, unified interface to a complex subsystem of classes.

**Use when:** A subsystem is complex to use directly, and most clients only need a simple, common set of operations from it.

```java
// A complex subsystem with several interacting parts
class CPU { void freeze() { System.out.println("CPU: freeze"); } void jump(long pos) { System.out.println("CPU: jump"); } void execute() { System.out.println("CPU: execute"); } }
class Memory { void load(long pos, byte[] data) { System.out.println("Memory: load"); } }
class HardDrive { byte[] read(long lba, int size) { System.out.println("HardDrive: read"); return new byte[size]; } }

// Facade — provides one simple method hiding the subsystem's complex interaction sequence
class ComputerFacade {
    private final CPU cpu = new CPU();
    private final Memory memory = new Memory();
    private final HardDrive hardDrive = new HardDrive();

    public void start() {
        cpu.freeze();
        memory.load(0, hardDrive.read(0, 1024));
        cpu.jump(0);
        cpu.execute();
    }
}

class FacadeDemo {
    public static void main(String[] args) {
        ComputerFacade computer = new ComputerFacade();
        computer.start(); // client doesn't need to know about CPU/Memory/HardDrive interactions at all
    }
}
```

---

## 22. Flyweight

**Intent:** Minimize memory usage by sharing common, immutable ("intrinsic") state across many objects, storing only genuinely unique ("extrinsic") state per instance.

**Use when:** You need to create a very large number of similar objects, and most of their state can be shared.

```java
// Intrinsic (shared, immutable) state
class TreeType {
    private final String name;
    private final String color;
    private final String texture; // imagine this is large, expensive-to-duplicate data

    public TreeType(String name, String color, String texture) {
        this.name = name; this.color = color; this.texture = texture;
    }

    public void draw(int x, int y) { // extrinsic state (position) passed in at call time
        System.out.println("Drawing " + name + " (" + color + ") at (" + x + "," + y + ")");
    }
}

// Flyweight factory — ensures each distinct TreeType is created only ONCE and reused
class TreeTypeFactory {
    private static final Map<String, TreeType> cache = new HashMap<>();

    public static TreeType get(String name, String color, String texture) {
        String key = name + color + texture;
        return cache.computeIfAbsent(key, k -> {
            System.out.println("Creating new TreeType: " + name); // only happens once per unique type
            return new TreeType(name, color, texture);
        });
    }
}

class Tree { // lightweight — holds only extrinsic (per-instance) state + a reference to the shared type
    private final int x, y;
    private final TreeType type;

    public Tree(int x, int y, TreeType type) { this.x = x; this.y = y; this.type = type; }
    public void draw() { type.draw(x, y); }
}

class FlyweightDemo {
    public static void main(String[] args) {
        List<Tree> forest = new ArrayList<>();
        // Simulating a million trees of only 2 species — TreeType is created just twice, not a million times.
        for (int i = 0; i < 1_000_000; i++) {
            TreeType type = TreeTypeFactory.get("Oak", "Green", "OakTexture");
            forest.add(new Tree(i, i * 2, type));
        }
        forest.get(0).draw(); // Drawing Oak (Green) at (0,0)
    }
}
```

---

## 23. Proxy

**Intent:** Provide a surrogate/placeholder for another object to control access to it — adding behavior (lazy loading, access control, logging) transparently.

**Use when:** You need to control access to an object — deferring expensive creation until needed, adding security checks, or adding logging/caching around calls.

```java
interface Image {
    void display();
}

class RealImage implements Image { // the real, expensive object
    private final String filename;

    public RealImage(String filename) {
        this.filename = filename;
        loadFromDisk(); // expensive operation
    }

    private void loadFromDisk() { System.out.println("Loading " + filename + " from disk..."); }

    public void display() { System.out.println("Displaying " + filename); }
}

// Proxy: defers creating the expensive RealImage until display() is actually called
class ProxyImage implements Image {
    private final String filename;
    private RealImage realImage; // null until actually needed

    public ProxyImage(String filename) { this.filename = filename; }

    @Override
    public void display() {
        if (realImage == null) {                 // lazy initialization
            realImage = new RealImage(filename);  // expensive load only happens on first actual use
        }
        realImage.display();
    }
}

class ProxyDemo {
    public static void main(String[] args) {
        Image image = new ProxyImage("large_photo.jpg");
        System.out.println("Image object created — but nothing loaded yet.");

        image.display(); // Loading large_photo.jpg from disk... / Displaying large_photo.jpg
        image.display(); // Displaying large_photo.jpg — no reload, RealImage is now cached
    }
}
```

This is precisely the mechanism Spring uses internally for `@Transactional`/AOP — the bean you inject is often actually a dynamic proxy wrapping the real object, transparently adding behavior around each method call.

---

# Behavioral Patterns

Behavioral patterns deal with communication and responsibility distribution between objects.

## 24. Chain of Responsibility

**Intent:** Pass a request along a chain of handlers, each deciding either to process it or pass it to the next handler.

**Use when:** More than one object may handle a request, and the handler isn't known in advance — e.g., a middleware/filter pipeline.

```java
abstract class SupportHandler {
    protected SupportHandler next;

    public SupportHandler setNext(SupportHandler next) {
        this.next = next;
        return next; // enables fluent chaining when building the chain
    }

    public void handle(int severity, String issue) {
        if (canHandle(severity)) {
            process(issue);
        } else if (next != null) {
            next.handle(severity, issue); // pass along the chain
        } else {
            System.out.println("No handler available for: " + issue);
        }
    }

    protected abstract boolean canHandle(int severity);
    protected abstract void process(String issue);
}

class Level1Support extends SupportHandler {
    protected boolean canHandle(int severity) { return severity <= 1; }
    protected void process(String issue) { System.out.println("Level1 resolved: " + issue); }
}

class Level2Support extends SupportHandler {
    protected boolean canHandle(int severity) { return severity <= 2; }
    protected void process(String issue) { System.out.println("Level2 resolved: " + issue); }
}

class ManagerSupport extends SupportHandler {
    protected boolean canHandle(int severity) { return severity <= 3; }
    protected void process(String issue) { System.out.println("Manager resolved: " + issue); }
}

class ChainOfResponsibilityDemo {
    public static void main(String[] args) {
        SupportHandler chain = new Level1Support();
        chain.setNext(new Level2Support()).setNext(new ManagerSupport());

        chain.handle(1, "Forgot password");     // Level1 resolved
        chain.handle(3, "Data corruption bug"); // passes through Level1 and Level2 to Manager
    }
}
```

---

## 25. Command

**Intent:** Encapsulate a request as an object, letting you parameterize clients with different requests, queue/log requests, and support undoable operations.

**Use when:** You need to decouple the object invoking an operation from the one that knows how to perform it, or need undo/redo/queuing.

```java
interface Command {
    void execute();
    void undo();
}

class Light {
    private boolean on = false;
    public void turnOn() { on = true; System.out.println("Light is ON"); }
    public void turnOff() { on = false; System.out.println("Light is OFF"); }
}

class TurnOnCommand implements Command {
    private final Light light;
    public TurnOnCommand(Light light) { this.light = light; }
    public void execute() { light.turnOn(); }
    public void undo() { light.turnOff(); }
}

class TurnOffCommand implements Command {
    private final Light light;
    public TurnOffCommand(Light light) { this.light = light; }
    public void execute() { light.turnOff(); }
    public void undo() { light.turnOn(); }
}

class RemoteControl {
    private final Deque<Command> history = new ArrayDeque<>();

    public void press(Command command) {
        command.execute();
        history.push(command); // track for undo
    }

    public void pressUndo() {
        if (!history.isEmpty()) {
            history.pop().undo();
        }
    }
}

class CommandDemo {
    public static void main(String[] args) {
        Light light = new Light();
        RemoteControl remote = new RemoteControl();

        remote.press(new TurnOnCommand(light));  // Light is ON
        remote.press(new TurnOffCommand(light)); // Light is OFF
        remote.pressUndo();                       // Light is ON (undoes the last command)
    }
}
```

---

## 26. Interpreter

**Intent:** Define a representation for a language's grammar, along with an interpreter that uses the representation to interpret sentences in that language.

**Use when:** You have a simple language/grammar to evaluate repeatedly (e.g., a basic rules engine or expression evaluator).

```java
interface Expression {
    int interpret();
}

class NumberExpression implements Expression {
    private final int number;
    public NumberExpression(int number) { this.number = number; }
    public int interpret() { return number; }
}

class AddExpression implements Expression {
    private final Expression left, right;
    public AddExpression(Expression left, Expression right) { this.left = left; this.right = right; }
    public int interpret() { return left.interpret() + right.interpret(); }
}

class SubtractExpression implements Expression {
    private final Expression left, right;
    public SubtractExpression(Expression left, Expression right) { this.left = left; this.right = right; }
    public int interpret() { return left.interpret() - right.interpret(); }
}

class InterpreterDemo {
    public static void main(String[] args) {
        // Represents the expression: (5 + 3) - 2
        Expression expression = new SubtractExpression(
            new AddExpression(new NumberExpression(5), new NumberExpression(3)),
            new NumberExpression(2)
        );

        System.out.println("Result: " + expression.interpret()); // Result: 6
    }
}
```

---

## 27. Iterator

**Intent:** Provide a way to access elements of a collection sequentially without exposing its underlying representation.

**Use when:** You want a uniform way to traverse different collection types, or want to support multiple simultaneous traversals of the same collection.

```java
interface Container<T> {
    Iterator<T> createIterator();
}

// A custom collection with an unusual internal representation (a fixed-size array with a count)
class NameRepository implements Container<String> {
    private final String[] names;
    private final int count;

    public NameRepository(String[] names, int count) { this.names = names; this.count = count; }

    @Override
    public Iterator<String> createIterator() {
        return new Iterator<>() {
            private int index = 0;

            @Override
            public boolean hasNext() { return index < count; }

            @Override
            public String next() { return names[index++]; }
        };
    }
}

class IteratorDemo {
    public static void main(String[] args) {
        Container<String> repo = new NameRepository(new String[]{"Alice", "Bob", "Carol", "Dave"}, 3);
        Iterator<String> it = repo.createIterator();

        while (it.hasNext()) {
            System.out.println(it.next()); // Alice, Bob, Carol (never touches Dave — count is 3)
        }
        // The client never needed to know NameRepository stores data in a fixed array with a separate count.
    }
}
```

*(Note: Java's own `Iterable`/`Iterator` interfaces, used pervasively by every collection class and the enhanced for-loop, are themselves a direct built-in implementation of this exact pattern.)*

---

## 28. Mediator

**Intent:** Define an object that encapsulates how a set of objects interact, promoting loose coupling by keeping objects from referring to each other directly.

**Use when:** A set of objects communicate in complex, many-to-many ways — centralize that communication through one mediator instead.

```java
interface ChatMediator {
    void sendMessage(String message, User sender);
    void addUser(User user);
}

class ChatRoom implements ChatMediator {
    private final List<User> users = new ArrayList<>();

    public void addUser(User user) { users.add(user); }

    @Override
    public void sendMessage(String message, User sender) {
        for (User user : users) {
            if (user != sender) { // don't echo back to the sender
                user.receive(message, sender.getName());
            }
        }
    }
}

class User {
    private final String name;
    private final ChatMediator mediator;

    public User(String name, ChatMediator mediator) {
        this.name = name;
        this.mediator = mediator;
        mediator.addUser(this);
    }

    public String getName() { return name; }

    public void send(String message) {
        System.out.println(name + " sends: " + message);
        mediator.sendMessage(message, this); // talks only to the mediator, not to other users directly
    }

    public void receive(String message, String from) {
        System.out.println(name + " received from " + from + ": " + message);
    }
}

class MediatorDemo {
    public static void main(String[] args) {
        ChatMediator chatRoom = new ChatRoom();
        User alice = new User("Alice", chatRoom);
        User bob = new User("Bob", chatRoom);
        User carol = new User("Carol", chatRoom);

        alice.send("Hey everyone!");
        // Bob and Carol each receive it; Alice never held direct references to Bob/Carol.
    }
}
```

---

## 29. Memento

**Intent:** Without violating encapsulation, capture and externalize an object's internal state so it can be restored to that state later (the basis for undo functionality).

**Use when:** You need to save/restore an object's state (e.g., undo in an editor) without exposing its internal implementation details to the code doing the saving.

```java
// Memento — an immutable snapshot of the editor's state
final class EditorMemento {
    private final String content; // package-private access — only Editor can read this
    EditorMemento(String content) { this.content = content; }
    String getContent() { return content; }
}

class Editor { // the "originator"
    private String content = "";

    public void type(String text) { content += text; }
    public String getContent() { return content; }

    public EditorMemento save() { return new EditorMemento(content); }
    public void restore(EditorMemento memento) { this.content = memento.getContent(); }
}

class History { // the "caretaker" — stores mementos without inspecting their contents
    private final Deque<EditorMemento> mementos = new ArrayDeque<>();

    public void push(EditorMemento memento) { mementos.push(memento); }
    public EditorMemento pop() { return mementos.pop(); }
}

class MementoDemo {
    public static void main(String[] args) {
        Editor editor = new Editor();
        History history = new History();

        editor.type("Hello");
        history.push(editor.save()); // checkpoint

        editor.type(", World!");
        history.push(editor.save()); // checkpoint

        editor.type(" This is a mistake.");
        System.out.println("Before undo: " + editor.getContent());

        editor.restore(history.pop()); // undo last checkpoint... but we need to pop twice for full undo
        history.pop();                  // (simplified demo logic)
        editor.restore(history.pop());
        System.out.println("After undo: " + editor.getContent()); // Hello
    }
}
```

---

## 30. Observer

**Intent:** Define a one-to-many dependency between objects so that when one object changes state, all its dependents are notified and updated automatically.

**Use when:** A change to one object requires changing/notifying others, and you don't want them tightly coupled — e.g., event listeners, pub/sub.

```java
interface Observer {
    void update(double price);
}

interface Subject {
    void subscribe(Observer observer);
    void unsubscribe(Observer observer);
    void notifyObservers();
}

class StockPrice implements Subject {
    private final List<Observer> observers = new ArrayList<>();
    private double price;

    public void subscribe(Observer observer) { observers.add(observer); }
    public void unsubscribe(Observer observer) { observers.remove(observer); }

    public void notifyObservers() {
        for (Observer observer : observers) {
            observer.update(price);
        }
    }

    public void setPrice(double price) {
        this.price = price;
        notifyObservers(); // automatically pushes the update to every subscriber
    }
}

class PriceDisplay implements Observer {
    private final String name;
    public PriceDisplay(String name) { this.name = name; }
    public void update(double price) { System.out.println(name + " display shows: $" + price); }
}

class PriceAlertBot implements Observer {
    public void update(double price) {
        if (price < 100) System.out.println("ALERT: Price dropped below $100!");
    }
}

class ObserverDemo {
    public static void main(String[] args) {
        StockPrice stock = new StockPrice();
        stock.subscribe(new PriceDisplay("Mobile App"));
        stock.subscribe(new PriceDisplay("Web Dashboard"));
        stock.subscribe(new PriceAlertBot());

        stock.setPrice(105.50); // both displays update, no alert
        stock.setPrice(95.20);  // both displays update, PLUS the alert fires
    }
}
```

---

## 31. State

**Intent:** Allow an object to alter its behavior when its internal state changes — the object will appear to change its class.

**Use when:** An object's behavior depends heavily on its current state, and you want to avoid a large, error-prone conditional block checking "what state am I in" throughout the class.

```java
interface VendingMachineState {
    void insertCoin(VendingMachine machine);
    void selectProduct(VendingMachine machine);
    void dispense(VendingMachine machine);
}

class IdleState implements VendingMachineState {
    public void insertCoin(VendingMachine machine) {
        System.out.println("Coin accepted.");
        machine.setState(machine.getHasCoinState());
    }
    public void selectProduct(VendingMachine machine) { System.out.println("Insert a coin first."); }
    public void dispense(VendingMachine machine) { System.out.println("Insert a coin first."); }
}

class HasCoinState implements VendingMachineState {
    public void insertCoin(VendingMachine machine) { System.out.println("Coin already inserted."); }
    public void selectProduct(VendingMachine machine) {
        System.out.println("Product selected.");
        machine.setState(machine.getDispensingState());
    }
    public void dispense(VendingMachine machine) { System.out.println("Select a product first."); }
}

class DispensingState implements VendingMachineState {
    public void insertCoin(VendingMachine machine) { System.out.println("Please wait, dispensing in progress."); }
    public void selectProduct(VendingMachine machine) { System.out.println("Already dispensing."); }
    public void dispense(VendingMachine machine) {
        System.out.println("Dispensing product... Enjoy!");
        machine.setState(machine.getIdleState());
    }
}

class VendingMachine {
    private final VendingMachineState idleState = new IdleState();
    private final VendingMachineState hasCoinState = new HasCoinState();
    private final VendingMachineState dispensingState = new DispensingState();
    private VendingMachineState currentState = idleState; // starts idle

    public void setState(VendingMachineState state) { this.currentState = state; }
    public VendingMachineState getIdleState() { return idleState; }
    public VendingMachineState getHasCoinState() { return hasCoinState; }
    public VendingMachineState getDispensingState() { return dispensingState; }

    // Delegates entirely to the current state — no giant if/else chain here.
    public void insertCoin() { currentState.insertCoin(this); }
    public void selectProduct() { currentState.selectProduct(this); }
    public void dispense() { currentState.dispense(this); }
}

class StateDemo {
    public static void main(String[] args) {
        VendingMachine machine = new VendingMachine();
        machine.selectProduct(); // Insert a coin first.
        machine.insertCoin();    // Coin accepted.
        machine.selectProduct(); // Product selected.
        machine.dispense();      // Dispensing product... Enjoy!
    }
}
```

---

## 32. Strategy

**Intent:** Define a family of interchangeable algorithms, encapsulate each one, and make them interchangeable — letting the algorithm vary independently from the clients that use it.

**Use when:** You have multiple ways of doing something (sorting, pricing, payment processing) and want to select/swap the algorithm at runtime without conditional logic scattered everywhere.

```java
interface PaymentStrategy {
    void pay(double amount);
}

class CreditCardPayment implements PaymentStrategy {
    private final String cardNumber;
    public CreditCardPayment(String cardNumber) { this.cardNumber = cardNumber; }
    public void pay(double amount) {
        System.out.println("Paid $" + amount + " using credit card ending in " + cardNumber.substring(cardNumber.length() - 4));
    }
}

class PayPalPayment implements PaymentStrategy {
    private final String email;
    public PayPalPayment(String email) { this.email = email; }
    public void pay(double amount) { System.out.println("Paid $" + amount + " using PayPal account " + email); }
}

class CryptoPayment implements PaymentStrategy {
    public void pay(double amount) { System.out.println("Paid $" + amount + " using cryptocurrency"); }
}

class ShoppingCart {
    private final List<Double> items = new ArrayList<>();
    private PaymentStrategy paymentStrategy; // the swappable algorithm

    public void addItem(double price) { items.add(price); }
    public void setPaymentStrategy(PaymentStrategy strategy) { this.paymentStrategy = strategy; }

    public void checkout() {
        double total = items.stream().mapToDouble(Double::doubleValue).sum();
        paymentStrategy.pay(total); // delegates to whichever strategy is currently set
    }
}

class StrategyDemo {
    public static void main(String[] args) {
        ShoppingCart cart = new ShoppingCart();
        cart.addItem(29.99);
        cart.addItem(15.50);

        cart.setPaymentStrategy(new CreditCardPayment("4111111111111234"));
        cart.checkout(); // Paid $45.49 using credit card ending in 1234

        cart.setPaymentStrategy(new PayPalPayment("user@example.com"));
        cart.checkout(); // Paid $45.49 using PayPal account user@example.com
    }
}
```

**Real-world Java example:** `Comparator` passed to `Collections.sort()` or `list.sort()` is a direct, everyday application of the Strategy pattern — the sorting algorithm stays the same, but the comparison logic is swappable.

---

## 33. Template Method

**Intent:** Define the skeleton of an algorithm in a base class method, deferring some specific steps to subclasses, without letting subclasses change the algorithm's overall structure.

**Use when:** Multiple classes share the same overall algorithm/process but differ in some specific steps.

```java
abstract class DataProcessor {
    // The template method — final, so subclasses can't change the overall sequence.
    public final void process() {
        readData();
        parseData();
        validateData(); // shared, common step — implemented once here
        saveData();
    }

    protected abstract void readData();
    protected abstract void parseData();

    // A shared, default implementation — subclasses CAN override if genuinely needed ("hook").
    protected void validateData() {
        System.out.println("Running standard validation...");
    }

    protected abstract void saveData();
}

class CsvDataProcessor extends DataProcessor {
    protected void readData() { System.out.println("Reading CSV file..."); }
    protected void parseData() { System.out.println("Parsing CSV rows..."); }
    protected void saveData() { System.out.println("Saving parsed CSV data to database."); }
}

class JsonDataProcessor extends DataProcessor {
    protected void readData() { System.out.println("Reading JSON file..."); }
    protected void parseData() { System.out.println("Parsing JSON structure..."); }
    // Overrides the shared validation with something JSON-specific
    @Override
    protected void validateData() { System.out.println("Running JSON schema validation..."); }
    protected void saveData() { System.out.println("Saving parsed JSON data to database."); }
}

class TemplateMethodDemo {
    public static void main(String[] args) {
        DataProcessor csvProcessor = new CsvDataProcessor();
        csvProcessor.process();
        // Reading CSV file... / Parsing CSV rows... / Running standard validation... / Saving...

        System.out.println("---");

        DataProcessor jsonProcessor = new JsonDataProcessor();
        jsonProcessor.process();
        // Reading JSON file... / Parsing JSON structure... / Running JSON schema validation... / Saving...
    }
}
```

**Real-world Java example:** Spring's `JdbcTemplate` follows this exact pattern — it handles the fixed skeleton (get connection, create statement, handle exceptions, close resources), while you supply just the specific query and row-mapping logic.

---

## 34. Visitor

**Intent:** Represent an operation to be performed on the elements of an object structure, letting you define a new operation without changing the classes of the elements it operates on.

**Use when:** You have a stable set of element types but need to add new, unrelated operations over them frequently, without modifying each element class every time.

```java
// Element hierarchy
interface Shape {
    void accept(ShapeVisitor visitor); // "accepts" a visitor — enables double dispatch
}

class Circle implements Shape {
    final double radius;
    public Circle(double radius) { this.radius = radius; }
    public void accept(ShapeVisitor visitor) { visitor.visit(this); }
}

class Rectangle implements Shape {
    final double width, height;
    public Rectangle(double width, double height) { this.width = width; this.height = height; }
    public void accept(ShapeVisitor visitor) { visitor.visit(this); }
}

// Visitor interface — one visit() method per concrete element type
interface ShapeVisitor {
    void visit(Circle circle);
    void visit(Rectangle rectangle);
}

// Concrete visitor #1: calculates total area — added WITHOUT modifying Circle/Rectangle
class AreaVisitor implements ShapeVisitor {
    private double totalArea = 0;
    public void visit(Circle circle) { totalArea += Math.PI * circle.radius * circle.radius; }
    public void visit(Rectangle rectangle) { totalArea += rectangle.width * rectangle.height; }
    public double getTotalArea() { return totalArea; }
}

// Concrete visitor #2: a completely different operation, again with no changes to Shape classes
class SvgExportVisitor implements ShapeVisitor {
    public void visit(Circle circle) {
        System.out.println("<circle r=\"" + circle.radius + "\" />");
    }
    public void visit(Rectangle rectangle) {
        System.out.println("<rect width=\"" + rectangle.width + "\" height=\"" + rectangle.height + "\" />");
    }
}

class VisitorDemo {
    public static void main(String[] args) {
        List<Shape> shapes = List.of(new Circle(5), new Rectangle(4, 6));

        AreaVisitor areaVisitor = new AreaVisitor();
        shapes.forEach(shape -> shape.accept(areaVisitor));
        System.out.println("Total area: " + areaVisitor.getTotalArea());

        System.out.println("--- SVG Export ---");
        SvgExportVisitor svgVisitor = new SvgExportVisitor();
        shapes.forEach(shape -> shape.accept(svgVisitor));
    }
}
```

**Trade-off worth noting:** Visitor makes adding new **operations** trivial (just write a new `ShapeVisitor` implementation), but adding a new **element type** (e.g., `Triangle`) requires modifying the `ShapeVisitor` interface and every existing visitor implementation — the pattern inverts the usual extensibility trade-off, so it's best suited when the element hierarchy is stable but new operations are added frequently.

---

## Quick Reference: Which Pattern Solves Which Problem?

| Problem | Pattern |
|---|---|
| Need exactly one instance of a class | Singleton |
| Don't know which concrete class to instantiate | Factory Method |
| Need to create families of related objects | Abstract Factory |
| Object has many optional constructor parameters | Builder |
| Object creation is expensive; need many similar copies | Prototype |
| Incompatible interfaces need to work together | Adapter |
| Class hierarchy varies along 2 independent dimensions | Bridge |
| Need to treat a tree of objects uniformly (leaf vs. group) | Composite |
| Need to add optional, combinable behavior to an object | Decorator |
| A subsystem is too complex to use directly | Facade |
| Need to create a huge number of similar, memory-heavy objects | Flyweight |
| Need to control/defer/guard access to an object | Proxy |
| A request may be handled by one of several handlers | Chain of Responsibility |
| Need to encapsulate a request as an object (undo/queue/log) | Command |
| Need to evaluate sentences in a simple custom grammar | Interpreter |
| Need to traverse a collection without exposing its internals | Iterator |
| Many objects communicate in a complex, tangled way | Mediator |
| Need to save/restore an object's state (undo) | Memento |
| One object's change needs to notify many dependents | Observer |
| An object's behavior depends heavily on its current state | State |
| Need interchangeable algorithms selected at runtime | Strategy |
| Multiple classes share the same overall algorithm skeleton | Template Method |
| Need new operations over a stable set of element types | Visitor |
