# Java Interview Questions & Answers

A curated set of Java interview questions covering core concepts, OOP, collections, concurrency, JVM, and Java 8+ features.

---

## Table of Contents
1. [Core Java & OOP](#1-core-java--oop)
2. [Strings, Equals & HashCode](#2-strings-equals--hashcode)
3. [Collections Framework](#3-collections-framework)
4. [Exception Handling](#4-exception-handling)
5. [Multithreading & Concurrency](#5-multithreading--concurrency)
6. [JVM, Memory & Garbage Collection](#6-jvm-memory--garbage-collection)
7. [Java 8+ Features](#7-java-8-features)
8. [Coding / Scenario Questions](#8-coding--scenario-questions)

---

## 1. Core Java & OOP

### Q1. What are the main features of Java?
- **Platform independent** (Write Once, Run Anywhere via JVM bytecode).
- **Object-Oriented**, **Robust**, **Secure**, **Multithreaded**.
- **Automatic memory management** via Garbage Collector.
- **Rich API** and a strong ecosystem (Spring, Hibernate, etc.).

### Q2. Difference between JDK, JRE, and JVM?
| Component | Purpose |
|-----------|---------|
| **JVM** | Runtime engine that executes bytecode. Platform-dependent. |
| **JRE** | JVM + standard libraries needed to *run* Java apps. |
| **JDK** | JRE + development tools (javac, javadoc, debugger) needed to *develop* Java apps. |

### Q3. Explain the four pillars of OOP.
- **Encapsulation** – binding data and methods, hiding internals (private fields + getters/setters).
- **Inheritance** – child class reuses/extends parent (`extends`, `implements`).
- **Polymorphism** – same method behaves differently (overloading = compile-time, overriding = runtime).
- **Abstraction** – exposing only essential details via abstract classes / interfaces.

### Q4. Difference between abstract class and interface?
| Aspect | Abstract Class | Interface |
|--------|---------------|-----------|
| Methods | Abstract + concrete | Abstract + default + static (Java 8+), private (Java 9+) |
| Variables | Any modifier | `public static final` only |
| Constructors | Yes | No |
| Inheritance | Single | Multiple |
| Use case | "Is-a" with shared base behavior | Pure contract / capability |

### Q5. Can we override a static method?
No. Static methods belong to the class, not the instance. Defining a static method with the same signature in a subclass is **method hiding**, not overriding.

### Q6. What is the difference between `final`, `finally`, and `finalize`?
- **final** – keyword: prevents inheritance (class), overriding (method), reassignment (variable).
- **finally** – block: always executes after try/catch (used for cleanup).
- **finalize()** – method called by GC before reclaiming an object (deprecated since Java 9).

### Q7. Difference between `==` and `.equals()`?
- `==` compares **references** for objects, **values** for primitives.
- `.equals()` compares **content** (when properly overridden, e.g., in `String`, `Integer`).

### Q8. Why is Java not 100% object-oriented?
Because it uses **primitive types** (`int`, `char`, `boolean`, etc.) which are not objects. Wrapper classes exist, but primitives themselves are not OO.

### Q9. What is the difference between method overloading and method overriding?
| Feature | Overloading | Overriding |
|---------|-------------|------------|
| Class | Same class | Parent–child class |
| Signature | Different parameters | Same signature |
| Polymorphism | Compile-time | Runtime |
| Return type | Can differ | Must be same/covariant |

### Q10. What is a constructor? Types?
Special method to initialize objects.
- **Default** – no-arg, provided by compiler if none defined.
- **Parameterized** – takes arguments.
- **Copy** – takes another object of same class (Java doesn't provide one by default).

---

## 2. Strings, Equals & HashCode

### Q11. Why are Strings immutable in Java?
- **Security** (used in class loading, file paths, network connections).
- **Thread safety** (no synchronization needed).
- **String pool optimization** (interning).
- **Caching of hashcode** for use in HashMaps.

### Q12. Difference between `String`, `StringBuilder`, and `StringBuffer`?
| Class | Mutable | Thread-safe | Performance |
|-------|---------|-------------|-------------|
| String | No | Yes (immutable) | Slow on concat |
| StringBuilder | Yes | No | Fast (single-threaded) |
| StringBuffer | Yes | Yes (synchronized) | Slower than StringBuilder |

### Q13. What is the String pool?
A special memory area in the heap where string literals are stored. If a literal already exists, the same reference is reused.
```java
String a = "hello";
String b = "hello";       // same reference as a
String c = new String("hello"); // new object on heap
a == b // true
a == c // false
```

### Q14. Contract between `equals()` and `hashCode()`?
- If `a.equals(b)` is true → `a.hashCode() == b.hashCode()` **must** be true.
- If hashcodes are equal → equals may or may not be true.
- Violating this breaks `HashMap`, `HashSet`, etc.

---

## 3. Collections Framework

### Q15. Difference between `ArrayList` and `LinkedList`?
| Aspect | ArrayList | LinkedList |
|--------|-----------|------------|
| Internal | Dynamic array | Doubly linked list |
| Random access | O(1) | O(n) |
| Insertion/deletion (middle) | O(n) | O(1) once node found |
| Memory | Less overhead | More (next/prev pointers) |

### Q16. Difference between `HashMap` and `Hashtable`?
- `HashMap` → not synchronized, allows one null key, many null values.
- `Hashtable` → synchronized (legacy), no nulls allowed.
- Modern alternative: **`ConcurrentHashMap`** (segment/bucket-level locking).

### Q17. How does HashMap work internally?
- Backed by an array of `Node<K,V>` (buckets).
- Key's `hashCode()` → hashed → bucket index.
- Collisions resolved via **linked list**, converted to **balanced tree (Red-Black)** when bucket size > 8 (Java 8+).
- Default capacity = 16, load factor = 0.75 → resize doubles capacity.

### Q18. Difference between `HashSet`, `LinkedHashSet`, and `TreeSet`?
- `HashSet` – no order, O(1).
- `LinkedHashSet` – insertion order, O(1).
- `TreeSet` – sorted (natural / comparator), O(log n).

### Q19. `Comparable` vs `Comparator`?
- `Comparable<T>` – natural ordering, `compareTo()` defined inside the class.
- `Comparator<T>` – external/custom ordering, `compare()` defined separately.

### Q20. Fail-fast vs Fail-safe iterators?
- **Fail-fast** – throw `ConcurrentModificationException` if collection is modified during iteration (`ArrayList`, `HashMap`).
- **Fail-safe** – work on a clone, no exception (`ConcurrentHashMap`, `CopyOnWriteArrayList`).

---

## 4. Exception Handling

### Q21. Checked vs Unchecked exceptions?
- **Checked** – checked at compile time, must be declared/handled (`IOException`, `SQLException`).
- **Unchecked** – runtime, extend `RuntimeException` (`NullPointerException`, `IllegalArgumentException`).
- **Errors** – serious problems, should not be caught (`OutOfMemoryError`).

### Q22. Can we have try without catch?
Yes – `try` with `finally`, or `try-with-resources`.

### Q23. What is try-with-resources?
Automatically closes resources implementing `AutoCloseable`:
```java
try (BufferedReader br = new BufferedReader(new FileReader("f.txt"))) {
    return br.readLine();
}
```

### Q24. Can `finally` block be skipped?
Yes – if `System.exit(0)` is called, JVM crashes, or thread is killed.

### Q25. Difference between `throw` and `throws`?
- `throw` – actually throws an exception instance.
- `throws` – declaration in method signature listing possible checked exceptions.

---

## 5. Multithreading & Concurrency

### Q26. Difference between Process and Thread?
- **Process** – independent execution unit with its own memory.
- **Thread** – lightweight subunit of a process, shares memory.

### Q27. Ways to create a thread?
1. Extend `Thread` and override `run()`.
2. Implement `Runnable` (preferred – allows other inheritance).
3. Implement `Callable<V>` + `FutureTask` (returns result, throws checked exceptions).
4. Use `ExecutorService` / thread pools.

### Q28. Lifecycle of a thread?
`NEW` → `RUNNABLE` → `RUNNING` → `BLOCKED/WAITING/TIMED_WAITING` → `TERMINATED`.

### Q29. `synchronized` keyword?
Provides mutual exclusion. Can be applied at:
- Method level (locks `this` or class object for static).
- Block level (`synchronized(obj) { … }`).

### Q30. `wait()`, `notify()`, `notifyAll()`?
- Defined in `Object`, must be called inside `synchronized` block.
- `wait()` releases the lock and waits.
- `notify()` wakes one waiting thread; `notifyAll()` wakes all.

### Q31. `volatile` vs `synchronized`?
- **volatile** – guarantees **visibility** of changes across threads (no caching), but not atomicity.
- **synchronized** – guarantees both **visibility** and **atomicity**, but heavier.

### Q32. What is a deadlock? How to avoid?
Two or more threads waiting on each other's locks indefinitely.
**Avoidance:** consistent lock ordering, `tryLock` with timeout, avoid nested locks, use higher-level concurrency utilities.

### Q33. Executor Framework?
- `ExecutorService` manages thread pools.
- Common factory methods: `newFixedThreadPool`, `newCachedThreadPool`, `newSingleThreadExecutor`, `newScheduledThreadPool`.
- `submit()` returns a `Future<V>`.

### Q34. `Callable` vs `Runnable`?
| Feature | Runnable | Callable |
|---------|----------|----------|
| Returns | void | V |
| Throws checked | No | Yes |
| Method | `run()` | `call()` |

### Q35. What is a `CompletableFuture`?
A `Future` with composable async operations: `thenApply`, `thenCompose`, `thenCombine`, `exceptionally`. Backed by `ForkJoinPool.commonPool()` by default.

### Q36. What is the `ForkJoinPool`?
Pool optimized for divide-and-conquer using **work-stealing**. Used by parallel streams and `CompletableFuture`.

---

## 6. JVM, Memory & Garbage Collection

### Q37. JVM memory areas?
- **Heap** – objects (Young Gen: Eden + S0/S1, Old Gen).
- **Stack** – per-thread, stores frames (locals, partial results).
- **Metaspace** (Java 8+, replaces PermGen) – class metadata.
- **PC Register** – per-thread, current instruction.
- **Native Method Stack**.

### Q38. How does Garbage Collection work?
- JVM tracks reachable objects from GC roots (stack refs, static refs).
- Unreachable objects are reclaimed.
- **Minor GC** cleans Young Gen; **Major/Full GC** cleans Old Gen.
- Common collectors: Serial, Parallel, CMS (deprecated), **G1** (default since Java 9), **ZGC**, **Shenandoah** (low-pause).

### Q39. Strong, Soft, Weak, Phantom references?
- **Strong** – default, never collected while reachable.
- **Soft** – collected when memory is low (good for caches).
- **Weak** – collected at next GC (e.g., `WeakHashMap`).
- **Phantom** – used for post-mortem cleanup via `ReferenceQueue`.

### Q40. What causes a memory leak in Java?
- Static collections holding references.
- Unclosed resources (streams, connections).
- Listeners/callbacks not deregistered.
- Inner classes holding outer references.

---

## 7. Java 8+ Features

### Q41. Key Java 8 features?
- Lambda expressions
- Functional interfaces (`@FunctionalInterface`)
- Streams API
- `Optional`
- Default & static methods in interfaces
- New Date/Time API (`java.time`)
- `CompletableFuture`

### Q42. What is a functional interface? Examples?
Interface with exactly one abstract method.
- `Function<T,R>`, `Predicate<T>`, `Consumer<T>`, `Supplier<T>`, `BiFunction<T,U,R>`, `Runnable`, `Callable`.

### Q43. Stream API – intermediate vs terminal operations?
- **Intermediate** (lazy, return Stream): `filter`, `map`, `sorted`, `distinct`, `limit`, `peek`.
- **Terminal** (trigger execution): `collect`, `forEach`, `reduce`, `count`, `findFirst`, `anyMatch`.

### Q44. `map()` vs `flatMap()`?
- `map` – one-to-one transformation.
- `flatMap` – one-to-many (flattens nested streams).

### Q45. `Optional` – why and how?
Container object that may or may not contain a value, helps avoid `NullPointerException`.
```java
Optional<User> user = repo.findById(id);
String name = user.map(User::getName).orElse("Anonymous");
```
**Don't:** use `Optional` as a field or method parameter.

### Q46. New features by version (highlights)?
- **Java 9** – modules (JPMS), JShell.
- **Java 10** – `var` (local-variable type inference).
- **Java 11 (LTS)** – HTTP Client, `String.repeat`, `var` in lambdas.
- **Java 14** – switch expressions, records (preview).
- **Java 16** – records (final), pattern matching for `instanceof`.
- **Java 17 (LTS)** – sealed classes, pattern matching.
- **Java 21 (LTS)** – virtual threads (Project Loom), pattern matching for switch, sequenced collections.

### Q47. What are records (Java 16+)?
Immutable data carriers with auto-generated constructor, accessors, `equals`, `hashCode`, `toString`.
```java
public record Point(int x, int y) {}
```

### Q48. What are sealed classes (Java 17)?
Restrict which classes can extend/implement them.
```java
public sealed interface Shape permits Circle, Square, Triangle {}
```

### Q49. What are virtual threads (Java 21)?
Lightweight threads managed by the JVM (not OS). Massively scalable for I/O-bound workloads.
```java
Thread.startVirtualThread(() -> doWork());
```

---

## 8. Coding / Scenario Questions

### Q50. Reverse a string without using built-in `reverse()`.
```java
String reverse(String s) {
    char[] c = s.toCharArray();
    int i = 0, j = c.length - 1;
    while (i < j) { char t = c[i]; c[i++] = c[j]; c[j--] = t; }
    return new String(c);
}
```

### Q51. Find duplicates in a list using streams.
```java
list.stream()
    .collect(Collectors.groupingBy(Function.identity(), Collectors.counting()))
    .entrySet().stream()
    .filter(e -> e.getValue() > 1)
    .map(Map.Entry::getKey)
    .toList();
```

### Q52. Count word frequency from a sentence.
```java
Map<String, Long> freq = Arrays.stream(sentence.split("\\s+"))
    .collect(Collectors.groupingBy(Function.identity(), Collectors.counting()));
```

### Q53. Find the second highest number in a list.
```java
int second = list.stream()
    .distinct()
    .sorted(Comparator.reverseOrder())
    .skip(1)
    .findFirst()
    .orElseThrow();
```

### Q54. Implement a thread-safe singleton.
```java
public class Singleton {
    private static volatile Singleton instance;
    private Singleton() {}
    public static Singleton getInstance() {
        if (instance == null) {
            synchronized (Singleton.class) {
                if (instance == null) instance = new Singleton();
            }
        }
        return instance;
    }
}
```
Or use **enum singleton** (simplest, serialization-safe):
```java
public enum Singleton { INSTANCE; public void doWork() {} }
```

### Q55. Producer-Consumer using `BlockingQueue`.
```java
BlockingQueue<Integer> q = new ArrayBlockingQueue<>(10);
// Producer
new Thread(() -> { try { q.put(1); } catch (InterruptedException e) {} }).start();
// Consumer
new Thread(() -> { try { Integer v = q.take(); } catch (InterruptedException e) {} }).start();
```
