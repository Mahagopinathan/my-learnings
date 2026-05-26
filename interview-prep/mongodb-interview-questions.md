# MongoDB Interview Questions & Answers

Covers MongoDB fundamentals, CRUD, query operators, aggregation, indexes, schema design, replication, sharding, and Spring Data MongoDB integration.

---

## Table of Contents
1. [Fundamentals](#1-fundamentals)
2. [CRUD Operations](#2-crud-operations)
3. [Query Operators](#3-query-operators)
4. [Indexes](#4-indexes)
5. [Aggregation Framework](#5-aggregation-framework)
6. [Schema Design](#6-schema-design)
7. [Replication, Sharding & High Availability](#7-replication-sharding--high-availability)
8. [Transactions & Consistency](#8-transactions--consistency)
9. [Performance & Best Practices](#9-performance--best-practices)
10. [Spring Data MongoDB](#10-spring-data-mongodb)

---

## 1. Fundamentals

### Q1. What is MongoDB?
A document-oriented, NoSQL database that stores data in flexible, JSON-like documents (BSON). Schemaless, horizontally scalable, supports rich queries and indexes.

### Q2. MongoDB vs RDBMS terminology?
| RDBMS | MongoDB |
|-------|---------|
| Database | Database |
| Table | Collection |
| Row | Document |
| Column | Field |
| Join | `$lookup` (aggregation) |
| Primary key | `_id` (auto if not provided) |

### Q3. What is BSON?
**Binary JSON** – the format MongoDB uses internally and over the wire. Adds types JSON lacks: `ObjectId`, `Date`, `Binary`, `Decimal128`, etc.

### Q4. What is `_id`?
Every document must have a unique `_id`. By default, MongoDB generates an `ObjectId` (12 bytes: timestamp + machine + process + counter). You can supply your own.

### Q5. Advantages of MongoDB?
- Flexible schema – easy to evolve.
- Horizontal scalability via sharding.
- Rich query language and secondary indexes.
- Built-in replication for HA.
- Aggregation framework, GeoJSON, full-text search.

### Q6. When NOT to use MongoDB?
- Strict multi-record ACID transactions across many entities (RDBMS still simpler/better).
- Highly relational data with complex joins.
- Apps that benefit more from a fixed schema and SQL reporting tools.

### Q7. Document vs Embedded vs Referenced?
- **Embedded** – nested documents inside the parent.
- **Referenced** – store an `ObjectId` pointing to another document (manual join).
- Choose based on access patterns: embed for "one-to-few" and read-together data; reference for "one-to-many" or shared/large data.

---

## 2. CRUD Operations

### Q8. Insert documents
```js
// Single
db.users.insertOne({ name: "Alice", age: 30, city: "Pune" });

// Many
db.users.insertMany([
  { name: "Bob", age: 25 },
  { name: "Carol", age: 28 }
]);
```

### Q9. Find documents
```js
db.users.find({ city: "Pune" });
db.users.find({ age: { $gte: 18 } });
db.users.findOne({ _id: ObjectId("...") });
db.users.find({}, { name: 1, _id: 0 });    // projection
db.users.find().sort({ age: -1 }).limit(5).skip(10);
```

### Q10. Update documents
```js
db.users.updateOne(
    { name: "Alice" },
    { $set: { age: 31, city: "Mumbai" } }
);

db.users.updateMany(
    { active: false },
    { $set: { archived: true } }
);

// Upsert – insert if no match
db.users.updateOne(
    { email: "x@y.com" },
    { $set: { name: "X" } },
    { upsert: true }
);
```

### Q11. Delete documents
```js
db.users.deleteOne({ name: "Alice" });
db.users.deleteMany({ active: false });
db.users.drop();   // drop entire collection
```

### Q12. Bulk write
```js
db.users.bulkWrite([
    { insertOne: { document: { name: "X" } } },
    { updateOne: { filter: { name: "Y" }, update: { $set: { age: 40 } } } },
    { deleteOne: { filter: { name: "Z" } } }
]);
```

---

## 3. Query Operators

### Q13. Comparison operators
- `$eq`, `$ne`, `$gt`, `$gte`, `$lt`, `$lte`, `$in`, `$nin`.
```js
db.users.find({ age: { $in: [25, 30, 35] } });
db.users.find({ city: { $nin: ["Pune", "Mumbai"] } });
```

### Q14. Logical operators
- `$and`, `$or`, `$not`, `$nor`.
```js
db.users.find({
    $or: [
        { city: "Pune" },
        { age: { $gte: 30 } }
    ]
});
```

### Q15. Element operators
- `$exists`, `$type`.
```js
db.users.find({ phone: { $exists: true } });
db.users.find({ age: { $type: "int" } });
```

### Q16. Array operators
- `$all`, `$elemMatch`, `$size`.
```js
db.users.find({ tags: { $all: ["mongo", "java"] } });
db.scores.find({ scores: { $elemMatch: { $gte: 80, $lte: 90 } } });
db.lists.find({ items: { $size: 3 } });
```

### Q17. Update operators
- `$set`, `$unset`, `$inc`, `$mul`, `$rename`, `$min`, `$max`.
- Array: `$push`, `$pull`, `$addToSet`, `$pop`, `$each`.
```js
db.users.updateOne({ _id: 1 }, { $inc: { loginCount: 1 } });
db.users.updateOne({ _id: 1 }, { $push: { tags: "premium" } });
db.users.updateOne({ _id: 1 }, { $addToSet: { tags: { $each: ["a", "b"] } } });
```

### Q18. Regex / text search
```js
db.users.find({ name: /^A/ });               // starts with A
db.users.find({ name: { $regex: "alice", $options: "i" } });

// Text index + $text
db.articles.createIndex({ body: "text" });
db.articles.find({ $text: { $search: "mongodb tutorial" } });
```

---

## 4. Indexes

### Q19. Why use indexes?
Speed up reads by avoiding full collection scans. Without indexes, queries do a `COLLSCAN`. Indexes are B-Tree structures (or special types).

### Q20. Index types?
- **Single field** – `{ field: 1 }`.
- **Compound** – multiple fields, leftmost prefix rule.
- **Multikey** – on array fields.
- **Text** – full-text search.
- **Geospatial** – `2d`, `2dsphere` for GeoJSON.
- **Hashed** – for sharding by hash.
- **TTL** – auto-delete after expiry.
- **Wildcard**, **Partial**, **Unique**, **Sparse**.

### Q21. Create indexes
```js
db.users.createIndex({ email: 1 }, { unique: true });
db.users.createIndex({ city: 1, age: -1 });
db.sessions.createIndex({ createdAt: 1 }, { expireAfterSeconds: 3600 }); // TTL
db.places.createIndex({ location: "2dsphere" });
```

### Q22. ESR rule for compound indexes
For optimal compound indexes, order fields as **Equality, Sort, Range**:
- `Equality` fields first (e.g., `status`).
- Then `Sort` fields (e.g., `createdAt`).
- Then `Range` fields (e.g., `price`).

### Q23. Inspect query performance
```js
db.users.find({ email: "x@y.com" }).explain("executionStats");
```
Look for `IXSCAN` (good) vs `COLLSCAN` (bad), `nReturned`, `totalDocsExamined`, `executionTimeMillis`.

### Q24. Drop / list indexes
```js
db.users.getIndexes();
db.users.dropIndex("email_1");
```

---

## 5. Aggregation Framework

### Q25. What is aggregation?
A pipeline of stages that transforms documents (filter, group, project, etc.). Replaces SQL `GROUP BY`/`JOIN`/window-style operations.

### Q26. Common stages
- `$match` – filter (use early to leverage indexes).
- `$project` – reshape/select fields.
- `$group` – group by key with accumulators (`$sum`, `$avg`, `$min`, `$max`, `$push`).
- `$sort`, `$limit`, `$skip`.
- `$lookup` – left outer join with another collection.
- `$unwind` – flatten arrays.
- `$addFields` / `$set` – add computed fields.
- `$facet` – multiple sub-pipelines in parallel.
- `$bucket`, `$bucketAuto` – histogram-style grouping.
- `$out` / `$merge` – write results to a collection.

### Q27. Total revenue per product
```js
db.orders.aggregate([
    { $match: { status: "PAID" } },
    { $group: { _id: "$productId", revenue: { $sum: "$amount" }, count: { $sum: 1 } } },
    { $sort: { revenue: -1 } },
    { $limit: 10 }
]);
```

### Q28. Join orders with customers
```js
db.orders.aggregate([
    { $lookup: {
        from: "customers",
        localField: "customerId",
        foreignField: "_id",
        as: "customer"
    }},
    { $unwind: "$customer" },
    { $project: { _id: 1, amount: 1, "customer.name": 1, "customer.email": 1 } }
]);
```

### Q29. Average order value per month
```js
db.orders.aggregate([
    { $group: {
        _id: { y: { $year: "$createdAt" }, m: { $month: "$createdAt" } },
        avgValue: { $avg: "$amount" },
        count: { $sum: 1 }
    }},
    { $sort: { "_id.y": 1, "_id.m": 1 } }
]);
```

### Q30. Top 3 customers by total spend per region (window-style)
```js
db.orders.aggregate([
    { $group: { _id: { region: "$region", customer: "$customerId" }, total: { $sum: "$amount" } } },
    { $sort: { "_id.region": 1, total: -1 } },
    { $group: {
        _id: "$_id.region",
        topCustomers: { $push: { customer: "$_id.customer", total: "$total" } }
    }},
    { $project: { topCustomers: { $slice: ["$topCustomers", 3] } } }
]);
```

---

## 6. Schema Design

### Q31. Embed vs reference – rules of thumb?
**Embed when:**
- Sub-document is accessed together with the parent.
- One-to-few relationship.
- Sub-document doesn't grow unbounded.
- Atomic updates with the parent matter.

**Reference when:**
- Many-to-many or large/unbounded one-to-many.
- Sub-entity is shared across parents.
- Sub-entity changes independently.
- Document size would exceed limits.

### Q32. Document size limit?
**16 MB per document.** Use GridFS for larger files (binaries split into chunks).

### Q33. Common schema patterns
- **Bucket pattern** – group time-series data (e.g., metrics per hour).
- **Outlier pattern** – store rare large data outside main doc.
- **Computed pattern** – store precomputed values to avoid expensive aggregation.
- **Subset pattern** – embed a subset (last 5 reviews) and store rest separately.
- **Polymorphic pattern** – heterogeneous docs in one collection.

### Q34. Anti-patterns to avoid
- Massive arrays growing unbounded inside a document (lookups & updates get slow).
- Treating MongoDB like a relational DB with many tiny collections and frequent `$lookup`s.
- Not adding indexes for frequent queries.
- Ignoring document size growth (16 MB limit).

---

## 7. Replication, Sharding & High Availability

### Q35. What is a replica set?
A group of `mongod` instances maintaining the same data – one **primary** (writes) + multiple **secondaries** (read replicas + failover candidates) + optional **arbiter** (vote-only, no data).

### Q36. Failover?
If primary becomes unavailable, eligible secondaries hold an election. Whoever gets a majority becomes the new primary. Apps using the official driver auto-reconnect.

### Q37. Read preferences?
- `primary` (default) – strongly consistent reads.
- `primaryPreferred`, `secondary`, `secondaryPreferred`, `nearest` – trade consistency for latency/scaling.

### Q38. Write concern?
Controls acknowledgment level:
- `w: 1` – primary only (default).
- `w: "majority"` – majority of replica set (durable across failover).
- `w: 0` – fire-and-forget (no ack).
- `j: true` – wait until journaled.

### Q39. Read concern?
- `local`, `available` – fast, may read uncommitted-by-majority data.
- `majority` – returns data acknowledged by majority.
- `linearizable` – strongest consistency for reads.

### Q40. What is sharding?
Horizontal scaling by partitioning data across multiple servers (shards) using a **shard key**.
Components:
- **Shard** – replica set holding a subset of data.
- **mongos** – router clients connect to.
- **Config servers** – store cluster metadata (themselves a replica set).

### Q41. Choosing a shard key – best practices?
- High cardinality (many unique values).
- Even distribution (no hotspot).
- Aligned with frequent query patterns (so queries are routed, not broadcast).
- **Avoid monotonically increasing keys** (e.g., timestamps) → use **hashed** sharding instead.

### Q42. Range vs Hashed sharding?
- **Range** – good for range queries; risk of uneven chunks.
- **Hashed** – uniform distribution; range queries become scatter-gather.

---

## 8. Transactions & Consistency

### Q43. Does MongoDB support ACID transactions?
- **Single document writes** – atomic by default.
- **Multi-document transactions** – supported since 4.0 (replica sets) and 4.2 (sharded clusters). Use sparingly; they're more expensive.

### Q44. Multi-document transaction example (driver-level)
```js
const session = db.getMongo().startSession();
session.startTransaction();
try {
    session.getDatabase("bank").accounts.updateOne({ _id: "A" }, { $inc: { bal: -100 } });
    session.getDatabase("bank").accounts.updateOne({ _id: "B" }, { $inc: { bal: +100 } });
    session.commitTransaction();
} catch (e) {
    session.abortTransaction();
} finally {
    session.endSession();
}
```

### Q45. CAP/PACELC – where does MongoDB sit?
By default, MongoDB favors **Consistency + Partition tolerance** (CP). Reads from primary are strongly consistent; with `w: majority`, writes are durable. You can trade consistency for latency via read preferences/concerns.

---

## 9. Performance & Best Practices

### Q46. Common performance tips
- Use indexes; check `explain()` plans.
- Project only needed fields.
- Use compound indexes ordered by **ESR** (Equality, Sort, Range).
- Avoid `$where` and JavaScript expressions on the server.
- Avoid huge unbounded arrays; consider bucket pattern.
- Batch with `bulkWrite` for high write volume.
- Use connection pooling; don't create a new client per request.

### Q47. Common monitoring tools
- `mongostat`, `mongotop`.
- Atlas/Ops Manager dashboards.
- `db.serverStatus()`, `db.collection.stats()`.
- Slow query log (`profile` level 1/2).
- Prometheus exporters / APM (Datadog, New Relic).

### Q48. How to handle large file storage?
Use **GridFS** – splits files into chunks (default 255KB) and stores metadata separately. Useful when files exceed the 16 MB document limit.

### Q49. Capped collections?
Fixed-size collections that overwrite oldest documents when full. Useful for logs/event buffers. Insertion order is preserved; deletes/updates that change size are restricted.

### Q50. Backup strategies?
- **mongodump/mongorestore** – logical backup.
- **Filesystem snapshot** – consistent if WiredTiger journal is included.
- **Atlas / Ops Manager** – continuous backup with PIT restore.

---

## 10. Spring Data MongoDB

### Q51. Quick setup
```yaml
spring:
  data:
    mongodb:
      uri: mongodb://localhost:27017/myapp
```

### Q52. Define a document
```java
@Document(collection = "users")
public class User {
    @Id
    private String id;

    @Indexed(unique = true)
    private String email;

    private String name;
    private List<String> roles;
    private Address address;

    @CreatedDate
    private Instant createdAt;
}
```

### Q53. Repository
```java
public interface UserRepository extends MongoRepository<User, String> {
    Optional<User> findByEmail(String email);
    List<User> findByRolesContaining(String role);

    @Query("{ 'address.city': ?0, 'active': true }")
    List<User> findActiveByCity(String city);
}
```

### Q54. MongoTemplate for complex queries / aggregations
```java
@Autowired
MongoTemplate mongoTemplate;

Query q = new Query(Criteria.where("status").is("ACTIVE")
                            .and("age").gte(18))
            .with(Sort.by(Sort.Direction.DESC, "createdAt"))
            .limit(50);
List<User> users = mongoTemplate.find(q, User.class);

Aggregation agg = Aggregation.newAggregation(
    Aggregation.match(Criteria.where("status").is("PAID")),
    Aggregation.group("productId").sum("amount").as("revenue"),
    Aggregation.sort(Sort.Direction.DESC, "revenue"),
    Aggregation.limit(10)
);
List<Document> result = mongoTemplate.aggregate(agg, "orders", Document.class).getMappedResults();
```

### Q55. Common annotations
- `@Document` – marks a class as a MongoDB document.
- `@Id` – primary key.
- `@Field` – maps field name in DB.
- `@Indexed`, `@CompoundIndex`, `@TextIndexed`.
- `@DBRef` – reference to another document (use sparingly; prefer manual reference + `$lookup`).
- `@CreatedDate`, `@LastModifiedDate` (with `@EnableMongoAuditing`).

### Q56. Optimistic locking with `@Version`
```java
@Document
public class Order {
    @Id String id;
    @Version Long version;
    BigDecimal amount;
}
```
On concurrent update with stale version → `OptimisticLockingFailureException`.

### Q57. Transactions in Spring Data MongoDB
Requires a replica set (or sharded cluster).
```java
@Transactional
public void transfer(String fromId, String toId, BigDecimal amount) {
    accountRepo.debit(fromId, amount);
    accountRepo.credit(toId, amount);
}
```
Configure a `MongoTransactionManager` bean.
