# Hibernate / JPA Interview Questions & Answers

Covers JPA & Hibernate concepts: ORM mapping, session management, caching, fetching strategies, transactions, and performance tuning.

---

## Table of Contents
1. [Basics: ORM, JPA, Hibernate](#1-basics-orm-jpa-hibernate)
2. [Entities & Mappings](#2-entities--mappings)
3. [Session, Persistence Context & Lifecycle](#3-session-persistence-context--lifecycle)
4. [Fetching, Lazy Loading & N+1](#4-fetching-lazy-loading--n1)
5. [Caching](#5-caching)
6. [Querying: HQL, Criteria, JPQL, Native](#6-querying-hql-criteria-jpql-native)
7. [Transactions & Concurrency](#7-transactions--concurrency)
8. [Performance & Best Practices](#8-performance--best-practices)

---

## 1. Basics: ORM, JPA, Hibernate

### Q1. What is ORM?
Object-Relational Mapping maps Java objects to relational database tables, eliminating boilerplate JDBC and SQL for CRUD operations.

### Q2. What is JPA?
**Jakarta Persistence API** (formerly Java Persistence API) – a *specification* for ORM in Java. Defines entities, persistence context, EntityManager, JPQL, etc.

### Q3. What is Hibernate?
The most popular **JPA implementation**. Adds extra features (HQL, second-level cache, filters, multi-tenancy, etc.) on top of the spec.

### Q4. Advantages of Hibernate over plain JDBC?
- No SQL boilerplate; CRUD generated automatically.
- Database independence (dialects).
- Caching (1st & 2nd level).
- Lazy loading and dirty checking.
- HQL/JPQL – object-oriented query language.
- Automatic schema generation (DDL).

### Q5. Difference between `Hibernate` and `JPA`?
| Aspect | JPA | Hibernate |
|--------|-----|-----------|
| Type | Specification | Implementation |
| API | `EntityManager`, `EntityManagerFactory` | `Session`, `SessionFactory` |
| Query language | JPQL | HQL (superset of JPQL) |
| Vendor lock-in | Portable across providers | Hibernate-specific |

### Q6. Core configuration files / setup?
- `persistence.xml` (pure JPA).
- `hibernate.cfg.xml` or programmatic configuration (Hibernate-native).
- In Spring Boot, configured via `application.properties` (`spring.datasource.*`, `spring.jpa.*`).

---

## 2. Entities & Mappings

### Q7. What is an Entity?
A POJO mapped to a DB table, annotated with `@Entity` and having a primary key (`@Id`).
Requirements:
- `@Entity` annotation.
- Public/protected no-arg constructor.
- Non-final class.
- `@Id` field.

### Q8. Common JPA annotations?
- `@Entity`, `@Table`, `@Id`, `@GeneratedValue`, `@Column`.
- Relationships: `@OneToOne`, `@OneToMany`, `@ManyToOne`, `@ManyToMany`.
- `@JoinColumn`, `@JoinTable`, `@MappedBy`.
- `@Embedded`, `@Embeddable`, `@ElementCollection`.
- `@Inheritance`, `@DiscriminatorColumn`, `@DiscriminatorValue`.

### Q9. Primary key generation strategies (`@GeneratedValue`)?
- **AUTO** – provider chooses (default).
- **IDENTITY** – DB auto-increment column. *Cannot use JDBC batch inserts.*
- **SEQUENCE** – DB sequence. Recommended for Oracle/Postgres.
- **TABLE** – emulates sequence with a separate table (slow, rarely used).

### Q10. Cardinality / relationship mappings?
- `@OneToOne` – one entity ↔ one entity.
- `@OneToMany` – one parent → many children. Always **bidirectional with `@ManyToOne`** for performance.
- `@ManyToOne` – many children → one parent (the owning side).
- `@ManyToMany` – uses a join table; consider modeling as two `@OneToMany` with an explicit join entity if it has extra fields.

### Q11. Owning side vs inverse side?
The **owning side** is the one without `mappedBy` – it controls the foreign key. The **inverse side** uses `mappedBy = "fieldOnOwningSide"` and is read-only for FK purposes.
```java
@Entity
class Order {
    @ManyToOne @JoinColumn(name = "customer_id")
    Customer customer; // owning side
}
@Entity
class Customer {
    @OneToMany(mappedBy = "customer")
    List<Order> orders; // inverse side
}
```

### Q12. Cascade types?
Propagate operations from parent to child:
- `PERSIST`, `MERGE`, `REMOVE`, `REFRESH`, `DETACH`, `ALL`.
- Use carefully on `@ManyToMany` to avoid deleting shared entities.

### Q13. `orphanRemoval` vs `CascadeType.REMOVE`?
- `CascadeType.REMOVE` – removes children when parent is removed.
- `orphanRemoval = true` – removes a child when it is **disassociated** from the parent (e.g., removed from the collection).

### Q14. Inheritance strategies?
- **SINGLE_TABLE** (default) – one table for the whole hierarchy + discriminator. Fast, but lots of nullable columns.
- **JOINED** – separate tables joined on PK. Normalized, slower joins.
- **TABLE_PER_CLASS** – one table per concrete class, no joins. Wasted columns; polymorphic queries use `UNION`.

### Q15. `@Embeddable` and `@Embedded`?
Used to flatten value-object types into the owning entity's table.
```java
@Embeddable class Address { String city; String zip; }
@Entity class User { @Embedded Address address; }
```

### Q16. Difference between `@Entity` and `@Table`?
- `@Entity` marks the class as a managed entity.
- `@Table` is optional, used to override table name, schema, indexes, unique constraints.

---

## 3. Session, Persistence Context & Lifecycle

### Q17. What is the Persistence Context?
A first-level cache and a "set of managed entities" associated with an `EntityManager` (or Hibernate `Session`). Every entity in it is tracked for changes (dirty checking) and flushed at commit/flush.

### Q18. `EntityManagerFactory` vs `EntityManager`?
- `EntityManagerFactory` – heavyweight, thread-safe, **created once per app** (per persistence unit).
- `EntityManager` – lightweight, **not thread-safe**, one per unit of work / request.

### Q19. Entity lifecycle states?
- **Transient (New)** – not associated with persistence context, no DB row.
- **Managed (Persistent)** – attached to context, changes auto-synced to DB.
- **Detached** – previously managed, context closed; changes not tracked.
- **Removed** – marked for deletion at next flush.

Transitions:
```
new → persist() → managed
managed → detach()/clear()/close() → detached
detached → merge() → managed (returns a new managed copy)
managed → remove() → removed
```

### Q20. `save()` vs `persist()` vs `merge()` vs `saveOrUpdate()` vs `update()`?
- **`persist`** (JPA) – inserts; throws if entity already managed/detached. No return.
- **`save`** (Hibernate) – assigns ID and returns it; works for transient entities.
- **`merge`** (JPA) – copies state of detached entity into a managed instance and returns it.
- **`update`** (Hibernate) – reattaches a detached entity. Throws if another managed instance with same ID exists.
- **`saveOrUpdate`** (Hibernate) – save or update based on ID/version.

### Q21. What is dirty checking?
At flush time, Hibernate compares each managed entity to its **snapshot** taken when it was loaded; if any field changed, an UPDATE is issued automatically – no explicit `update()` call needed.

### Q22. What is `flush()` and when does it happen?
`flush()` synchronizes the persistence context with the DB (issues SQL but doesn't commit).
Default `FlushMode.AUTO` triggers flush:
- Before query execution that may be affected.
- Before transaction commit.
- On explicit `flush()`.

### Q23. `get()` vs `load()` (Hibernate-specific)?
- **`get(id)`** – hits DB immediately, returns `null` if not found.
- **`load(id)`** – returns a proxy without DB hit; throws `ObjectNotFoundException` when accessed if not found. Useful for setting FK without a real fetch.

### Q24. `find()` vs `getReference()` (JPA equivalents)?
- **`find()`** ≈ `get()` – returns the entity or `null`.
- **`getReference()`** ≈ `load()` – returns a lazy proxy.

---

## 4. Fetching, Lazy Loading & N+1

### Q25. `EAGER` vs `LAZY` fetching?
- **EAGER** – associated entity is loaded with the parent. Default for `@ManyToOne` and `@OneToOne`.
- **LAZY** – loaded on first access via proxy. Default for `@OneToMany` and `@ManyToMany`.
**Best practice:** prefer **LAZY** for everything; fetch eagerly only when needed via JOIN FETCH / EntityGraph.

### Q26. What is `LazyInitializationException`?
Thrown when accessing a lazy-loaded association **outside** an open session/transaction.
**Fixes:**
- Access inside the transactional method.
- Use `JOIN FETCH` / `@EntityGraph`.
- Use DTO projections.
- ❌ Don't use Open-Session-In-View as a default fix – it's an anti-pattern.

### Q27. What is the N+1 problem?
1 query to load N parents + N queries to load each parent's children = N+1 queries.
**Solutions:**
- `JOIN FETCH` in JPQL.
- `@EntityGraph(attributePaths = "children")` on the repository method.
- `@BatchSize(size = 20)` on the collection.
- `hibernate.default_batch_fetch_size`.
- DTO projections via constructor expression.

### Q28. `JOIN` vs `JOIN FETCH`?
- `JOIN` – joins for filtering only; associated entities are still lazy.
- `JOIN FETCH` – joins **and** initializes the association in the same query.

### Q29. What is `@EntityGraph`?
Declarative way to specify which attributes to fetch eagerly for a query.
```java
@EntityGraph(attributePaths = {"orders", "orders.items"})
List<Customer> findAll();
```

---

## 5. Caching

### Q30. Levels of caching in Hibernate?
- **First-level cache** – per `Session`/`EntityManager`. Always on, can't disable.
- **Second-level cache** – per `SessionFactory`, shared across sessions. Optional.
- **Query cache** – caches result IDs for queries. Requires 2nd-level cache.

### Q31. How to enable second-level cache?
1. Add a provider (Ehcache, Caffeine, Infinispan, Hazelcast).
2. Enable: `hibernate.cache.use_second_level_cache=true`.
3. Annotate entity: `@Cache(usage = CacheConcurrencyStrategy.READ_WRITE)`.

### Q32. Cache concurrency strategies?
- **READ_ONLY** – immutable data.
- **NONSTRICT_READ_WRITE** – occasional stale reads OK.
- **READ_WRITE** – uses soft locks; consistent.
- **TRANSACTIONAL** – fully transactional; needs a JTA cache.

### Q33. When to use second-level cache?
Read-mostly, rarely-changing reference data (countries, product catalog). Avoid for hot, write-heavy entities.

---

## 6. Querying: HQL, Criteria, JPQL, Native

### Q34. JPQL vs HQL?
- **JPQL** – standard, part of JPA, operates on entities/fields.
- **HQL** – Hibernate's superset of JPQL with extra features.

### Q35. JPQL example.
```java
List<User> users = em.createQuery(
    "select u from User u where u.email = :email", User.class)
  .setParameter("email", email)
  .getResultList();
```

### Q36. What is the Criteria API?
Type-safe, programmatic way to build queries. Useful for dynamic queries.
```java
CriteriaBuilder cb = em.getCriteriaBuilder();
CriteriaQuery<User> cq = cb.createQuery(User.class);
Root<User> u = cq.from(User.class);
cq.select(u).where(cb.equal(u.get("email"), email));
List<User> users = em.createQuery(cq).getResultList();
```

### Q37. Native SQL queries?
```java
List<Object[]> rows = em.createNativeQuery(
    "select id, name from users where active = ?")
  .setParameter(1, true)
  .getResultList();
```
Or map to entity: `em.createNativeQuery(sql, User.class)`.

### Q38. What are named queries?
Pre-defined, reusable queries.
```java
@NamedQuery(name = "User.findByEmail",
            query = "select u from User u where u.email = :email")
@Entity
public class User { ... }
```

### Q39. DTO projections – why?
Fetch only what's needed, avoid loading full entities and lazy-loading issues.
```java
@Query("select new com.x.UserView(u.id, u.name) from User u")
List<UserView> findAllViews();
```
Or use Spring Data interface-based projections.

### Q40. Pagination?
```java
em.createQuery("from User", User.class)
  .setFirstResult(0)
  .setMaxResults(20)
  .getResultList();
```
Spring Data: `Page<User> findAll(Pageable pageable);`

---

## 7. Transactions & Concurrency

### Q41. Optimistic vs pessimistic locking?
- **Optimistic** – uses `@Version` column; on update, checks version matches; if not, throws `OptimisticLockException`. Good for low-contention, read-heavy workloads.
- **Pessimistic** – locks row in DB (`SELECT … FOR UPDATE`). Use `LockModeType.PESSIMISTIC_READ/WRITE`. Good for hot rows.

### Q42. `@Version` annotation?
Adds optimistic concurrency control.
```java
@Entity
public class Account {
    @Id Long id;
    @Version Long version;
    BigDecimal balance;
}
```

### Q43. Transaction isolation levels?
- **READ_UNCOMMITTED** – dirty reads possible.
- **READ_COMMITTED** – default for many DBs; prevents dirty reads.
- **REPEATABLE_READ** – default for MySQL; prevents non-repeatable reads.
- **SERIALIZABLE** – strictest, prevents phantom reads.

### Q44. Default propagation in Spring `@Transactional`?
`REQUIRED` – join existing transaction or create a new one.

---

## 8. Performance & Best Practices

### Q45. How to handle bulk inserts/updates efficiently?
- Enable JDBC batching: `hibernate.jdbc.batch_size=50`.
- `hibernate.order_inserts=true`, `hibernate.order_updates=true`.
- Periodically `flush()` and `clear()` the persistence context to avoid OOM.
- Avoid `IDENTITY` PK strategy (disables batching for inserts).

### Q46. How to avoid `LazyInitializationException` properly?
- Keep DB access inside the transaction boundary.
- Fetch eagerly via `JOIN FETCH` / `@EntityGraph` only for that query.
- Use DTO projections for read APIs.

### Q47. Common performance pitfalls?
- N+1 queries.
- Eager fetching everywhere.
- Using `@OneToMany` with large collections.
- Loading whole entities for read-only data.
- Unbounded result sets (no pagination).
- Cartesian products from multiple `JOIN FETCH` on collections.

### Q48. How do you log/debug Hibernate SQL?
```properties
spring.jpa.show-sql=true
spring.jpa.properties.hibernate.format_sql=true
logging.level.org.hibernate.SQL=DEBUG
logging.level.org.hibernate.orm.jdbc.bind=TRACE
```

### Q49. How to monitor query performance?
- Hibernate statistics: `hibernate.generate_statistics=true`.
- Datasource proxy / **p6spy**.
- DB-level slow query log + `EXPLAIN ANALYZE`.
- APM tools (New Relic, Datadog, Dynatrace).

### Q50. When **not** to use Hibernate?
- Highly bulk, set-based DB workloads (use plain SQL / jOOQ / stored procedures).
- Reporting / analytics with very complex SQL.
- Performance-critical paths where you need full control over SQL.
- Schemas you don't own and that change frequently.
