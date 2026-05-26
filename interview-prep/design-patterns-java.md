# Design Patterns with Java Examples

A practical guide to the most commonly asked design patterns in Java interviews — Creational, Structural, and Behavioral — each with intent, when-to-use, a runnable example, and a real-world reference from the JDK or Spring.

---

## Table of Contents
1. [Why Design Patterns?](#1-why-design-patterns)
2. [Creational Patterns](#2-creational-patterns)
   - [Singleton](#21-singleton)
   - [Factory Method](#22-factory-method)
   - [Abstract Factory](#23-abstract-factory)
   - [Builder](#24-builder)
   - [Prototype](#25-prototype)
3. [Structural Patterns](#3-structural-patterns)
   - [Adapter](#31-adapter)
   - [Decorator](#32-decorator)
   - [Proxy](#33-proxy)
   - [Facade](#34-facade)
   - [Composite](#35-composite)
   - [Bridge](#36-bridge)
   - [Flyweight](#37-flyweight)
4. [Behavioral Patterns](#4-behavioral-patterns)
   - [Strategy](#41-strategy)
   - [Observer](#42-observer)
   - [Template Method](#43-template-method)
   - [Command](#44-command)
   - [Iterator](#45-iterator)
   - [State](#46-state)
   - [Chain of Responsibility](#47-chain-of-responsibility)
   - [Mediator](#48-mediator)
   - [Visitor](#49-visitor)
   - [Memento](#410-memento)
5. [SOLID Principles](#5-solid-principles)
6. [Common Interview Questions](#6-common-interview-questions)

---

## 1. Why Design Patterns?

Design patterns are reusable solutions to common software design problems. They:
- Provide a **shared vocabulary** for engineers.
- Encode **best practices** for flexibility and maintainability.
- Help avoid common pitfalls (rigid, fragile, or coupled designs).

The **Gang of Four (GoF)** book classifies patterns into:
- **Creational** – object creation (Singleton, Factory, Builder, Prototype, Abstract Factory).
- **Structural** – composition of classes/objects (Adapter, Decorator, Proxy, Facade, Composite, Bridge, Flyweight).
- **Behavioral** – communication between objects (Strategy, Observer, Template Method, Command, Iterator, State, Chain of Responsibility, Mediator, Visitor, Memento).

> Don't over-engineer: apply patterns when they solve a real problem, not for their own sake.

---

## 2. Creational Patterns

### 2.1 Singleton
**Intent:** ensure a class has only one instance and provide global access.
**When to use:** caches, configuration, connection pools, loggers.

**Pitfalls:** thread safety, serialization, reflection, classloader issues.

**Best implementation in Java – enum singleton (Joshua Bloch's recommendation):**
```java
public enum ConfigManager {
    INSTANCE;

    private final Properties props = loadProps();

    public String get(String key) { return props.getProperty(key); }

    private Properties loadProps() {
        Properties p = new Properties();
        // load from file, env, etc.
        return p;
    }
}

// Usage
ConfigManager.INSTANCE.get("db.url");
```

**Thread-safe lazy singleton (double-checked locking):**
```java
public class Singleton {
    private static volatile Singleton instance;
    private Singleton() { }

    public static Singleton getInstance() {
        if (instance == null) {
            synchronized (Singleton.class) {
                if (instance == null) {
                    instance = new Singleton();
                }
            }
        }
        return instance;
    }
}
```

**JDK examples:** `Runtime.getRuntime()`, `Desktop.getDesktop()`.
**Spring:** beans are singleton-scoped by default (managed by container).

---

### 2.2 Factory Method
**Intent:** define an interface for creating objects, letting subclasses decide which class to instantiate.
**When to use:** when client code shouldn't depend on concrete classes.

```java
interface Notification {
    void send(String to, String msg);
}
class EmailNotification implements Notification {
    public void send(String to, String msg) { /* SMTP */ }
}
class SmsNotification implements Notification {
    public void send(String to, String msg) { /* SMS gateway */ }
}

class NotificationFactory {
    public static Notification create(String channel) {
        return switch (channel.toLowerCase()) {
            case "email" -> new EmailNotification();
            case "sms"   -> new SmsNotification();
            default -> throw new IllegalArgumentException(channel);
        };
    }
}

// Usage
NotificationFactory.create("email").send("a@b.com", "Hi");
```

**JDK examples:** `Calendar.getInstance()`, `NumberFormat.getInstance()`.

---

### 2.3 Abstract Factory
**Intent:** create families of related objects without specifying concrete classes.
**When to use:** UI toolkits per platform, DB driver families.

```java
// Abstract products
interface Button { void render(); }
interface Checkbox { void render(); }

// Concrete products – Mac
class MacButton implements Button { public void render() { System.out.println("Mac button"); } }
class MacCheckbox implements Checkbox { public void render() { System.out.println("Mac checkbox"); } }

// Concrete products – Windows
class WinButton implements Button { public void render() { System.out.println("Win button"); } }
class WinCheckbox implements Checkbox { public void render() { System.out.println("Win checkbox"); } }

// Abstract factory
interface UIFactory {
    Button createButton();
    Checkbox createCheckbox();
}
class MacUIFactory implements UIFactory {
    public Button createButton() { return new MacButton(); }
    public Checkbox createCheckbox() { return new MacCheckbox(); }
}
class WinUIFactory implements UIFactory {
    public Button createButton() { return new WinButton(); }
    public Checkbox createCheckbox() { return new WinCheckbox(); }
}

// Usage – client picks family at runtime
UIFactory ui = isMac() ? new MacUIFactory() : new WinUIFactory();
ui.createButton().render();
ui.createCheckbox().render();
```

---

### 2.4 Builder
**Intent:** construct complex objects step by step; separate construction from representation.
**When to use:** objects with many optional parameters; immutable objects.

```java
public final class HttpRequest {
    private final String url;
    private final String method;
    private final Map<String, String> headers;
    private final String body;
    private final Duration timeout;

    private HttpRequest(Builder b) {
        this.url = b.url;
        this.method = b.method;
        this.headers = Map.copyOf(b.headers);
        this.body = b.body;
        this.timeout = b.timeout;
    }

    public static Builder builder(String url) { return new Builder(url); }

    public static class Builder {
        private final String url;
        private String method = "GET";
        private final Map<String, String> headers = new HashMap<>();
        private String body;
        private Duration timeout = Duration.ofSeconds(30);

        Builder(String url) { this.url = url; }
        public Builder method(String m) { this.method = m; return this; }
        public Builder header(String k, String v) { this.headers.put(k, v); return this; }
        public Builder body(String b) { this.body = b; return this; }
        public Builder timeout(Duration t) { this.timeout = t; return this; }
        public HttpRequest build() { return new HttpRequest(this); }
    }
}

// Usage – readable and safe
HttpRequest req = HttpRequest.builder("https://api.example.com/users")
    .method("POST")
    .header("Content-Type", "application/json")
    .body("{\"name\":\"Alice\"}")
    .timeout(Duration.ofSeconds(5))
    .build();
```

**JDK / Library examples:** `StringBuilder`, `Stream.Builder`, `java.net.http.HttpRequest.Builder`, **Lombok `@Builder`**.

---

### 2.5 Prototype
**Intent:** create new objects by copying an existing instance.
**When to use:** expensive object creation; many similar objects.

```java
public class Document implements Cloneable {
    private String title;
    private List<String> sections = new ArrayList<>();

    @Override
    public Document clone() {
        try {
            Document copy = (Document) super.clone();
            copy.sections = new ArrayList<>(this.sections); // deep copy mutable fields
            return copy;
        } catch (CloneNotSupportedException e) {
            throw new AssertionError(e);
        }
    }
}
```
**Note:** Java's `Cloneable` is awkward; prefer **copy constructors** or **static factory methods** in modern code.

---

## 3. Structural Patterns

### 3.1 Adapter
**Intent:** convert the interface of a class into another interface clients expect.
**When to use:** integrating with legacy code or third-party libraries.

```java
// Target interface expected by client
interface PaymentProcessor {
    void pay(double amount);
}

// Adaptee – existing third-party API
class StripeApi {
    public void chargeInCents(int cents) { /* ... */ }
}

// Adapter
class StripeAdapter implements PaymentProcessor {
    private final StripeApi stripe;
    public StripeAdapter(StripeApi stripe) { this.stripe = stripe; }
    public void pay(double amount) { stripe.chargeInCents((int) (amount * 100)); }
}

// Client uses the target interface
PaymentProcessor processor = new StripeAdapter(new StripeApi());
processor.pay(19.99);
```

**JDK examples:** `Arrays.asList()`, `Collections.list(Enumeration)`, `InputStreamReader` (adapts byte stream to char stream).

---

### 3.2 Decorator
**Intent:** add behavior to objects dynamically without subclassing.
**When to use:** add cross-cutting responsibilities (logging, encryption, compression) flexibly.

```java
interface DataSource {
    void writeData(String data);
    String readData();
}

class FileDataSource implements DataSource {
    public void writeData(String data) { /* write to file */ }
    public String readData() { return "file contents"; }
}

abstract class DataSourceDecorator implements DataSource {
    protected final DataSource wrappee;
    DataSourceDecorator(DataSource source) { this.wrappee = source; }
    public void writeData(String data) { wrappee.writeData(data); }
    public String readData() { return wrappee.readData(); }
}

class EncryptionDecorator extends DataSourceDecorator {
    EncryptionDecorator(DataSource source) { super(source); }
    @Override public void writeData(String data) { super.writeData(encrypt(data)); }
    @Override public String readData() { return decrypt(super.readData()); }
    private String encrypt(String d) { return "ENC(" + d + ")"; }
    private String decrypt(String d) { return d.replace("ENC(", "").replace(")", ""); }
}

class CompressionDecorator extends DataSourceDecorator {
    CompressionDecorator(DataSource source) { super(source); }
    @Override public void writeData(String data) { super.writeData(compress(data)); }
    private String compress(String d) { return d; /* zip */ }
}

// Usage – compose at runtime
DataSource ds = new CompressionDecorator(new EncryptionDecorator(new FileDataSource()));
ds.writeData("secret");
```

**JDK examples:** `BufferedInputStream` wraps `InputStream`; `Collections.unmodifiableList`, `synchronizedMap`.

---

### 3.3 Proxy
**Intent:** provide a surrogate for another object to control access (lazy load, security, caching, remoting).

```java
interface Image {
    void display();
}

class RealImage implements Image {
    private final String filename;
    public RealImage(String filename) {
        this.filename = filename;
        loadFromDisk(); // expensive
    }
    public void display() { System.out.println("Display " + filename); }
    private void loadFromDisk() { System.out.println("Loading " + filename); }
}

class ImageProxy implements Image {
    private final String filename;
    private RealImage real;
    public ImageProxy(String filename) { this.filename = filename; }
    public void display() {
        if (real == null) real = new RealImage(filename);   // lazy
        real.display();
    }
}

Image img = new ImageProxy("photo.png");
// Loading happens only on first display()
img.display();
img.display();
```
**Variants:** **Virtual** (lazy), **Protection** (security), **Remote** (RMI), **Smart** (logging, ref counting).
**JDK / Spring:** `java.lang.reflect.Proxy`, **Spring AOP**, Hibernate lazy loading proxies.

---

### 3.4 Facade
**Intent:** provide a unified, simpler interface to a complex subsystem.
**When to use:** hide complexity behind a clean API.

```java
class CpuService { void freeze() {} void execute(long addr) {} void jump(long addr) {} }
class MemoryService { void load(long addr, byte[] data) {} }
class HardDriveService { byte[] read(long lba, int size) { return new byte[]{}; } }

class ComputerFacade {
    private final CpuService cpu = new CpuService();
    private final MemoryService memory = new MemoryService();
    private final HardDriveService hd = new HardDriveService();

    public void start() {
        cpu.freeze();
        memory.load(0, hd.read(0, 1024));
        cpu.jump(0);
        cpu.execute(0);
    }
}

new ComputerFacade().start();
```
Spring's `JdbcTemplate` and `RestTemplate` are facades over messy underlying APIs.

---

### 3.5 Composite
**Intent:** treat individual objects and compositions uniformly via a tree structure.
**When to use:** file systems, GUI components, organizational hierarchies.

```java
interface FileSystemNode {
    int sizeKb();
}

class FileNode implements FileSystemNode {
    private final int size;
    FileNode(int size) { this.size = size; }
    public int sizeKb() { return size; }
}

class DirectoryNode implements FileSystemNode {
    private final List<FileSystemNode> children = new ArrayList<>();
    public DirectoryNode add(FileSystemNode n) { children.add(n); return this; }
    public int sizeKb() { return children.stream().mapToInt(FileSystemNode::sizeKb).sum(); }
}

DirectoryNode root = new DirectoryNode()
    .add(new FileNode(100))
    .add(new DirectoryNode()
        .add(new FileNode(50))
        .add(new FileNode(150)));
System.out.println(root.sizeKb());  // 300
```

---

### 3.6 Bridge
**Intent:** decouple an abstraction from its implementation so they can vary independently.
**When to use:** avoiding a class explosion when you have multiple orthogonal dimensions of variation.

```java
// Implementation
interface Renderer { void renderShape(String shape); }
class VectorRenderer implements Renderer { public void renderShape(String s) { System.out.println("Vector " + s); } }
class RasterRenderer implements Renderer { public void renderShape(String s) { System.out.println("Pixels " + s); } }

// Abstraction
abstract class Shape {
    protected final Renderer renderer;
    Shape(Renderer renderer) { this.renderer = renderer; }
    abstract void draw();
}
class Circle extends Shape {
    Circle(Renderer r) { super(r); }
    void draw() { renderer.renderShape("circle"); }
}

new Circle(new VectorRenderer()).draw();
new Circle(new RasterRenderer()).draw();
```

---

### 3.7 Flyweight
**Intent:** share fine-grained objects to save memory.
**When to use:** lots of similar objects (characters in a document, particles in a game).

```java
// Intrinsic state shared across many uses
class CharacterStyle {
    private final String font;
    private final int size;
    private final String color;
    public CharacterStyle(String font, int size, String color) {
        this.font = font; this.size = size; this.color = color;
    }
}

class StyleFactory {
    private static final Map<String, CharacterStyle> cache = new HashMap<>();
    public static CharacterStyle get(String font, int size, String color) {
        return cache.computeIfAbsent(font + "-" + size + "-" + color,
            k -> new CharacterStyle(font, size, color));
    }
}
// Many characters can share the same CharacterStyle instance
```
**JDK example:** `Integer.valueOf(int)` caches values from -128..127.

---

## 4. Behavioral Patterns

### 4.1 Strategy
**Intent:** define a family of algorithms, encapsulate each, and make them interchangeable at runtime.
**When to use:** swap behaviors without `if/else` everywhere.

```java
interface DiscountStrategy {
    BigDecimal apply(BigDecimal amount);
}

class NoDiscount implements DiscountStrategy {
    public BigDecimal apply(BigDecimal a) { return a; }
}
class PercentageDiscount implements DiscountStrategy {
    private final BigDecimal pct;
    PercentageDiscount(BigDecimal pct) { this.pct = pct; }
    public BigDecimal apply(BigDecimal a) { return a.multiply(BigDecimal.ONE.subtract(pct)); }
}
class FlatDiscount implements DiscountStrategy {
    private final BigDecimal off;
    FlatDiscount(BigDecimal off) { this.off = off; }
    public BigDecimal apply(BigDecimal a) { return a.subtract(off).max(BigDecimal.ZERO); }
}

class Checkout {
    private final DiscountStrategy strategy;
    public Checkout(DiscountStrategy strategy) { this.strategy = strategy; }
    public BigDecimal total(BigDecimal subtotal) { return strategy.apply(subtotal); }
}

// Usage
new Checkout(new PercentageDiscount(new BigDecimal("0.10"))).total(new BigDecimal("100"));
```

**Modern Java tip:** strategies can be lambdas — `DiscountStrategy fifteenOff = a -> a.subtract(BigDecimal.valueOf(15));`.

**JDK examples:** `Comparator`, `Runnable`, all `java.util.function` interfaces.

---

### 4.2 Observer
**Intent:** define a one-to-many dependency so that when one object changes state, all dependents are notified.
**When to use:** event-driven systems, UI updates, pub/sub.

```java
interface Observer<T> { void onEvent(T event); }

class EventBus<T> {
    private final List<Observer<T>> observers = new CopyOnWriteArrayList<>();
    public void subscribe(Observer<T> o) { observers.add(o); }
    public void unsubscribe(Observer<T> o) { observers.remove(o); }
    public void publish(T event) { observers.forEach(o -> o.onEvent(event)); }
}

EventBus<String> bus = new EventBus<>();
bus.subscribe(System.out::println);
bus.publish("Order created");
```

**JDK examples:** `java.beans.PropertyChangeListener`, Swing listeners.
**Modern alternatives:** **`Flow.Publisher`** (reactive streams), **Spring's `ApplicationEventPublisher`** & `@EventListener`.

---

### 4.3 Template Method
**Intent:** define the skeleton of an algorithm in a base class; let subclasses fill in steps without changing the structure.

```java
abstract class DataImporter {
    public final void importData() {  // template
        open();
        var rows = read();
        validate(rows);
        save(rows);
        close();
    }
    protected abstract void open();
    protected abstract List<String> read();
    protected void validate(List<String> rows) { /* default */ }
    protected abstract void save(List<String> rows);
    protected abstract void close();
}

class CsvImporter extends DataImporter {
    protected void open() { /* open file */ }
    protected List<String> read() { return List.of("a", "b"); }
    protected void save(List<String> rows) { /* insert into DB */ }
    protected void close() { /* close file */ }
}
```
**JDK / Spring examples:** `HttpServlet.service()`, `AbstractList`, **`JdbcTemplate`**, `AbstractApplicationContext.refresh()`.

---

### 4.4 Command
**Intent:** encapsulate a request as an object so it can be queued, logged, undone.
**When to use:** undo/redo, task queues, macros.

```java
interface Command { void execute(); }

class TextEditor {
    private final StringBuilder text = new StringBuilder();
    public void append(String s) { text.append(s); }
    public void deleteLast(int n) { text.delete(text.length() - n, text.length()); }
    public String content() { return text.toString(); }
}

class AppendCommand implements Command {
    private final TextEditor editor;
    private final String text;
    public AppendCommand(TextEditor e, String t) { editor = e; text = t; }
    public void execute() { editor.append(text); }
    public void undo() { editor.deleteLast(text.length()); }
}

// Invoker
Deque<AppendCommand> history = new ArrayDeque<>();
TextEditor editor = new TextEditor();
AppendCommand c = new AppendCommand(editor, "Hello ");
c.execute();
history.push(c);
history.pop().undo();
```

---

### 4.5 Iterator
**Intent:** provide a way to access elements of a collection sequentially without exposing its underlying representation.

```java
class TreeNode<T> {
    T value;
    List<TreeNode<T>> children = new ArrayList<>();
}

class DfsIterator<T> implements Iterator<T> {
    private final Deque<TreeNode<T>> stack = new ArrayDeque<>();
    public DfsIterator(TreeNode<T> root) { stack.push(root); }
    public boolean hasNext() { return !stack.isEmpty(); }
    public T next() {
        TreeNode<T> n = stack.pop();
        for (int i = n.children.size() - 1; i >= 0; i--) stack.push(n.children.get(i));
        return n.value;
    }
}
```
**JDK example:** the entire `Iterator`/`Iterable` framework. Java's enhanced `for` loop relies on it.

---

### 4.6 State
**Intent:** allow an object to change its behavior when its internal state changes — appears as if the class changed.

```java
interface OrderState {
    void next(OrderContext ctx);
    String name();
}

class OrderContext {
    private OrderState state = new Created();
    public void setState(OrderState s) { this.state = s; }
    public void advance() { state.next(this); }
    public String currentState() { return state.name(); }
}

class Created implements OrderState {
    public void next(OrderContext ctx) { ctx.setState(new Paid()); }
    public String name() { return "CREATED"; }
}
class Paid implements OrderState {
    public void next(OrderContext ctx) { ctx.setState(new Shipped()); }
    public String name() { return "PAID"; }
}
class Shipped implements OrderState {
    public void next(OrderContext ctx) { ctx.setState(new Delivered()); }
    public String name() { return "SHIPPED"; }
}
class Delivered implements OrderState {
    public void next(OrderContext ctx) { /* terminal */ }
    public String name() { return "DELIVERED"; }
}
```

---

### 4.7 Chain of Responsibility
**Intent:** pass a request along a chain of handlers until one handles it.
**When to use:** middleware, validation pipelines, logging filters.

```java
abstract class LogHandler {
    protected LogHandler next;
    protected final Level minLevel;
    LogHandler(Level minLevel) { this.minLevel = minLevel; }
    public LogHandler chain(LogHandler n) { this.next = n; return n; }
    public void handle(Level level, String msg) {
        if (level.ordinal() >= minLevel.ordinal()) write(level, msg);
        if (next != null) next.handle(level, msg);
    }
    protected abstract void write(Level level, String msg);
    enum Level { DEBUG, INFO, WARN, ERROR }
}

class ConsoleLogger extends LogHandler {
    ConsoleLogger(Level l) { super(l); }
    protected void write(Level l, String m) { System.out.println(l + " " + m); }
}
class FileLogger extends LogHandler {
    FileLogger(Level l) { super(l); }
    protected void write(Level l, String m) { /* append to file */ }
}

LogHandler chain = new ConsoleLogger(LogHandler.Level.DEBUG);
chain.chain(new FileLogger(LogHandler.Level.WARN));
chain.handle(LogHandler.Level.WARN, "low disk");
```
**Servlet API:** the **filter chain** is a textbook example.

---

### 4.8 Mediator
**Intent:** define an object that encapsulates how a set of objects interact, reducing direct dependencies.
**Example:** a chat room where users send messages via the room rather than to each other directly.

```java
class ChatRoom {
    private final List<User> users = new ArrayList<>();
    public void register(User u) { users.add(u); u.setRoom(this); }
    public void send(String from, String msg) {
        users.stream().filter(u -> !u.name().equals(from))
                      .forEach(u -> u.receive(from, msg));
    }
}

class User {
    private final String name;
    private ChatRoom room;
    User(String name) { this.name = name; }
    void setRoom(ChatRoom r) { this.room = r; }
    public String name() { return name; }
    public void send(String msg) { room.send(name, msg); }
    public void receive(String from, String msg) { System.out.printf("%s -> %s: %s%n", from, name, msg); }
}
```

---

### 4.9 Visitor
**Intent:** separate algorithms from the objects they operate on; add new operations without modifying classes.

```java
interface Shape { <T> T accept(ShapeVisitor<T> v); }
class Circle implements Shape { double r; Circle(double r){this.r=r;} public <T> T accept(ShapeVisitor<T> v){ return v.visit(this); } }
class Square implements Shape { double s; Square(double s){this.s=s;} public <T> T accept(ShapeVisitor<T> v){ return v.visit(this); } }

interface ShapeVisitor<T> {
    T visit(Circle c);
    T visit(Square s);
}

class AreaVisitor implements ShapeVisitor<Double> {
    public Double visit(Circle c) { return Math.PI * c.r * c.r; }
    public Double visit(Square s) { return s.s * s.s; }
}

double area = new Circle(3).accept(new AreaVisitor());
```
Java's pattern matching for switch (Java 21+) often replaces this.

---

### 4.10 Memento
**Intent:** capture and externalize an object's internal state without violating encapsulation, so it can be restored later.
**When to use:** undo/redo, snapshots.

```java
class Editor {
    private String content = "";

    public void type(String s) { content += s; }
    public String content() { return content; }

    public Memento save() { return new Memento(content); }
    public void restore(Memento m) { this.content = m.state; }

    static class Memento {
        private final String state;
        private Memento(String state) { this.state = state; }
    }
}

Editor e = new Editor();
e.type("Hello ");
Editor.Memento snapshot = e.save();
e.type("World");
e.restore(snapshot);
System.out.println(e.content()); // "Hello "
```

---

## 5. SOLID Principles

These are the foundation that most patterns implement.

### S — Single Responsibility
A class should have only one reason to change. Don't mix HTTP handling, business logic, and persistence in one class.

### O — Open/Closed
Open for extension, closed for modification. Add behavior via new classes (Strategy, Decorator), not by editing existing ones.

### L — Liskov Substitution
Subtypes must be usable wherever their base types are expected. A `Square` that breaks `Rectangle.setWidth()` invariants is a violation.

### I — Interface Segregation
Clients shouldn't be forced to depend on methods they don't use. Split fat interfaces (e.g., `Reader`, `Writer` instead of `ReadWriter`).

### D — Dependency Inversion
High-level modules shouldn't depend on low-level modules; both should depend on abstractions. This is exactly what DI frameworks (Spring) enable.

---

## 6. Common Interview Questions

### Q1. Singleton vs static class — when to use which?
- **Singleton** – an actual instance; can implement interfaces, be passed around, mocked, lazy-loaded.
- **Static class** – purely procedural utilities (`Math`, `Collections`); cannot implement interfaces or be replaced for testing.
Prefer singleton (or DI-managed bean) when behavior may evolve or needs to be substituted.

### Q2. How is Builder different from Factory?
- **Factory** decides **which class** to instantiate.
- **Builder** decides **how to construct** an object step-by-step. They can be combined.

### Q3. Decorator vs Proxy?
Both wrap a target. Difference is intent:
- **Decorator** adds new behavior.
- **Proxy** controls access (lazy load, security, remote, caching).

### Q4. Strategy vs State?
Both swap behavior at runtime, but:
- **Strategy** is chosen by the **client**; behaviors are interchangeable variants.
- **State** transitions happen **internally** — the object changes its state in response to events.

### Q5. Adapter vs Facade?
- **Adapter** changes one interface to another (1:1).
- **Facade** simplifies a complex subsystem with a new, simpler interface.

### Q6. When NOT to use a pattern?
When the resulting design is more complex than the problem demands. Patterns are tools, not goals. A direct, simple implementation is often the right answer for small problems.

### Q7. Patterns you'll meet daily in Spring / JDK
- **Singleton** – Spring beans (default scope).
- **Factory** – `BeanFactory`, `ApplicationContext`.
- **Proxy** – Spring AOP, transactional proxies, Hibernate lazy loading.
- **Template Method** – `JdbcTemplate`, `RestTemplate`, `WebMvcConfigurer` callbacks.
- **Strategy** – `Comparator`, `RetryPolicy`, `Resilience4j` configurations.
- **Observer** – `ApplicationEventPublisher` / `@EventListener`.
- **Decorator** – `BufferedReader`, `Collections.unmodifiableList`.
- **Builder** – `UriComponentsBuilder`, `MockMvcBuilders`, `HttpRequest.newBuilder()`.
- **Chain of Responsibility** – Servlet filter chain, Spring Security filter chain.
- **Adapter** – `HandlerAdapter`, `MessageConverter`.

### Q8. Anti-patterns to avoid
- **God object** – one class doing everything.
- **Singleton overuse** – global mutable state, hard to test.
- **Big ball of mud** – no clear architecture.
- **Spaghetti inheritance** – deep hierarchies; prefer composition over inheritance.
- **Premature abstraction** – patterns introduced before the second similar requirement appears.

> **Rule of thumb:** Write the simple version first. Refactor to a pattern when the **second** or **third** similar requirement makes the abstraction worthwhile.
