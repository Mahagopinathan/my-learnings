# Apache Kafka Interview Questions & Answers

Covers Kafka fundamentals, architecture, producers, consumers, partitions, replication, delivery semantics, Kafka Streams, and Spring integration.

---

## Table of Contents
1. [Fundamentals](#1-fundamentals)
2. [Architecture & Components](#2-architecture--components)
3. [Topics, Partitions & Replication](#3-topics-partitions--replication)
4. [Producers](#4-producers)
5. [Consumers & Consumer Groups](#5-consumers--consumer-groups)
6. [Delivery Semantics](#6-delivery-semantics)
7. [Performance & Reliability](#7-performance--reliability)
8. [Kafka Streams & Connect](#8-kafka-streams--connect)
9. [Schema Registry](#9-schema-registry)
10. [Spring for Apache Kafka](#10-spring-for-apache-kafka)
11. [Operations & Troubleshooting](#11-operations--troubleshooting)

---

## 1. Fundamentals

### Q1. What is Apache Kafka?
A distributed, partitioned, replicated **commit log** designed for high-throughput, low-latency event streaming. Use cases: messaging, log aggregation, event sourcing, stream processing, change data capture (CDC).

### Q2. Kafka vs traditional message queues (RabbitMQ, ActiveMQ)?
| Aspect | Kafka | RabbitMQ |
|--------|-------|----------|
| Model | Distributed log (pull-based) | Broker with queues (push-based) |
| Throughput | Very high (millions/sec) | Moderate |
| Retention | Configurable (days/forever) | Until consumed |
| Replay | Yes (offset-based) | No |
| Routing | Simple (topic + key) | Rich (exchanges, bindings) |
| Best for | Streaming, event sourcing, log shipping | Task queues, RPC, complex routing |

### Q3. Key features of Kafka?
- Horizontally scalable through partitioning.
- Durable, replicated storage.
- High throughput (sequential disk I/O, zero-copy).
- Low latency (single-digit ms).
- Replay-able event log.
- Stream processing via Kafka Streams / ksqlDB.

### Q4. Use cases for Kafka?
- Asynchronous service-to-service communication (event-driven microservices).
- Centralized log/metrics pipeline.
- CDC (Debezium → Kafka).
- Real-time analytics & monitoring.
- Event sourcing / audit trails.
- Buffer between producers and slower consumers.

---

## 2. Architecture & Components

### Q5. Core components of a Kafka cluster?
- **Broker** – a Kafka server holding topic partitions.
- **Topic** – a named stream of messages.
- **Partition** – an ordered, append-only log within a topic.
- **Producer** – publishes records.
- **Consumer** – subscribes and reads records.
- **Consumer group** – a set of consumers sharing the load on a topic.
- **ZooKeeper / KRaft** – cluster metadata & controller election.

### Q6. Role of ZooKeeper in Kafka?
Historically used to store cluster metadata (broker list, topic configs, ACLs) and elect the **controller** broker. **KRaft** (Kafka Raft, GA in 3.3+) replaces ZooKeeper with an internal consensus protocol — recommended for new deployments.

### Q7. What is the controller broker?
One broker elected to manage cluster-wide responsibilities: partition leader assignments, broker membership, replica reassignment, topic creation/deletion.

### Q8. What is the ISR?
**In-Sync Replicas** – the set of replicas that are caught up with the leader (within `replica.lag.time.max.ms`). Only ISR members are eligible to be elected leader.

### Q9. Hierarchy of records?
`Cluster → Topic → Partition → Segment → Record (key, value, headers, timestamp, offset)`. Each partition is split into log **segments** on disk to support efficient retention and cleanup.

---

## 3. Topics, Partitions & Replication

### Q10. What is a topic?
A logical name for a stream of related records. Topics are split into partitions for parallelism and scaling.

### Q11. What is a partition?
An ordered, immutable sequence of records, each with a unique offset. **Order is guaranteed only within a partition**, not across the topic.

### Q12. How is the partition decided for a record?
- If a partition is set explicitly → that partition.
- Else if a key is set → `hash(key) % partitions` (default partitioner).
- Else → sticky/round-robin partitioner.

### Q13. Why are partitions important?
- **Parallelism** – consumers in a group can read in parallel, one partition each.
- **Scalability** – partitions can spread across brokers.
- **Ordering guarantee** – per partition.

### Q14. What is replication?
Each partition has a configurable number of **replicas**: one **leader** (handles all reads/writes) and several **followers** (replicate from leader). If the leader fails, a follower from the ISR takes over.

### Q15. `replication.factor` vs `min.insync.replicas`?
- **`replication.factor`** – total number of replicas per partition (e.g., 3).
- **`min.insync.replicas`** – minimum ISR count required to accept writes when `acks=all` (e.g., 2). Trade-off: stronger durability vs availability.

### Q16. What is log compaction?
Retention policy that keeps the **latest value per key** instead of all records. Useful for change log streams (e.g., user profile last update). Configure `cleanup.policy=compact` (or `compact,delete`).

### Q17. Retention configurations?
- `retention.ms` (default 7 days) – delete records older than X.
- `retention.bytes` – cap partition size.
- `cleanup.policy=delete` (default) or `compact`.

---

## 4. Producers

### Q18. Producer flow (high-level)?
1. Serialize record key/value.
2. Pick partition (key hash / explicit / round-robin).
3. Append to in-memory **batch** keyed by partition.
4. Sender thread sends batches to leader brokers.
5. Broker writes to log + replicates; sends acknowledgment.

### Q19. Producer key configurations?
- `acks` – 0, 1, all (-1).
- `retries` – attempts on retriable errors.
- `enable.idempotence=true` – prevents duplicates from retries (uses producer id + sequence).
- `max.in.flight.requests.per.connection` – set to 5 or less when idempotence is enabled (to keep ordering).
- `compression.type` – `gzip`, `snappy`, `lz4`, `zstd`.
- `linger.ms` – wait this long before sending a batch (better batching, slight latency).
- `batch.size` – max bytes per batch.
- `transactional.id` – enables transactions.

### Q20. What does `acks` mean?
- `acks=0` – no ack needed (fastest, can lose messages).
- `acks=1` – leader acks after writing (loss possible if leader crashes before replication).
- `acks=all` (or `-1`) – all ISR replicas must ack (strongest durability, requires `min.insync.replicas`).

### Q21. What is an idempotent producer?
With `enable.idempotence=true`, Kafka assigns each producer a **PID** and tracks a sequence number per partition, deduplicating retries. Guarantees **exactly-once delivery to a partition** for a single producer session.

### Q22. What are Kafka transactions?
Producers can write to multiple partitions/topics atomically via `initTransactions / beginTransaction / commitTransaction / abortTransaction`. Combined with idempotence, enables **exactly-once semantics (EOS)** end-to-end (consume → process → produce).

### Q23. Sample Java producer (idempotent + transactional)
```java
Properties p = new Properties();
p.put("bootstrap.servers", "broker1:9092");
p.put("key.serializer", StringSerializer.class.getName());
p.put("value.serializer", StringSerializer.class.getName());
p.put("acks", "all");
p.put("enable.idempotence", true);
p.put("transactional.id", "order-producer-1");

KafkaProducer<String, String> producer = new KafkaProducer<>(p);
producer.initTransactions();
try {
    producer.beginTransaction();
    producer.send(new ProducerRecord<>("orders", orderId, payload));
    producer.send(new ProducerRecord<>("audit", orderId, "CREATED"));
    producer.commitTransaction();
} catch (KafkaException e) {
    producer.abortTransaction();
}
```

---

## 5. Consumers & Consumer Groups

### Q24. What is a consumer group?
A set of consumers sharing a `group.id`. Kafka assigns each partition to exactly one consumer in the group. Adding/removing consumers triggers a **rebalance**.

### Q25. Rules for consumer-to-partition mapping?
- **A partition is consumed by only one consumer** in a group at a time.
- A consumer can read multiple partitions.
- **If consumers > partitions**, some consumers will be idle.

### Q26. What is offset?
A monotonically increasing position of a record within a partition. Consumers commit offsets to track "what has been processed". Stored in the internal `__consumer_offsets` topic.

### Q27. Auto vs manual offset commit?
- `enable.auto.commit=true` – periodic auto commits (`auto.commit.interval.ms`). Easy but risks duplicates/loss.
- Manual commit – `commitSync()` / `commitAsync()` after processing. Recommended for reliable consumption.

### Q28. `auto.offset.reset` options?
When no committed offset exists for a consumer/group:
- `earliest` – start from oldest.
- `latest` (default) – start from newest.
- `none` – throw exception.

### Q29. What is a rebalance?
Process of reassigning partitions to consumers when group membership or topic metadata changes. During rebalance, consumption is paused.
- **Eager** rebalance (default in older clients) – stop the world.
- **Cooperative / incremental** rebalance (`CooperativeStickyAssignor`) – moves only affected partitions, much smoother.

### Q30. Sample Java consumer
```java
Properties p = new Properties();
p.put("bootstrap.servers", "broker1:9092");
p.put("group.id", "order-service");
p.put("key.deserializer", StringDeserializer.class.getName());
p.put("value.deserializer", StringDeserializer.class.getName());
p.put("enable.auto.commit", false);
p.put("auto.offset.reset", "earliest");

try (KafkaConsumer<String, String> consumer = new KafkaConsumer<>(p)) {
    consumer.subscribe(List.of("orders"));
    while (true) {
        ConsumerRecords<String, String> records = consumer.poll(Duration.ofMillis(500));
        for (ConsumerRecord<String, String> rec : records) {
            process(rec);
        }
        consumer.commitSync();
    }
}
```

### Q31. How to make consumers idempotent?
- Track processed message keys (e.g., in DB / Redis).
- Use unique business keys for upserts.
- Make handlers naturally idempotent ("set status=X" rather than "increment count").

---

## 6. Delivery Semantics

### Q32. The three delivery semantics?
- **At-most-once** – may lose messages, never duplicates. Commit offsets *before* processing.
- **At-least-once** (most common) – may duplicate, never lose. Commit offsets *after* processing.
- **Exactly-once** – no loss, no duplicate. Requires idempotent producer + transactions + transactional consumer (`isolation.level=read_committed`).

### Q33. How to achieve exactly-once end-to-end?
1. Producer: `enable.idempotence=true` and `transactional.id`.
2. Consumer: `isolation.level=read_committed`.
3. Pattern **consume → process → produce** within a Kafka transaction (using `producer.sendOffsetsToTransaction(...)`).
4. For external sinks (DB), prefer **transactional outbox** + idempotent writes; true EOS to external systems is hard.

---

## 7. Performance & Reliability

### Q34. Why is Kafka so fast?
- **Sequential disk I/O** + page cache (no random I/O on hot path).
- **Zero-copy** (`sendfile`) from page cache to NIC.
- **Batching & compression** by producer and broker.
- **Partitioning** for parallelism.
- **No per-message broker bookkeeping** — consumers track their own offsets.

### Q35. How to tune throughput?
- Larger `batch.size`, slight `linger.ms` (e.g., 5–20 ms).
- Compression (`lz4`, `snappy`, `zstd`).
- More partitions (parallelism), but not too many (more open file handles, replication overhead).
- Larger `fetch.min.bytes` / `fetch.max.wait.ms` on consumers.
- Tune broker `num.io.threads`, `num.network.threads`.

### Q36. How to tune latency?
- Lower `linger.ms` (close to 0).
- Smaller batches.
- Avoid heavy compression.
- Co-locate consumers near brokers.

### Q37. How does Kafka guarantee ordering?
Only **within a partition**. To preserve per-entity order across messages, use that entity's id as the **message key** so all related records go to the same partition.

### Q38. How to handle "poison pill" messages?
A message that consistently fails processing.
- Use a **Dead Letter Topic (DLT)**: route failed messages there after N retries.
- Spring Kafka: `DefaultErrorHandler` with `DeadLetterPublishingRecoverer`.
- Continue consuming; investigate the DLT separately.

### Q39. How to prevent duplicates from consumers?
Idempotent processing logic + tracking processed offsets/keys + manual offset commits **after** successful processing.

### Q40. Kafka guarantees a consumer never reads its own producer's messages?
**No** — Kafka delivers all messages on subscribed topics to consumers regardless of source. Filtering self-produced messages is the application's job (e.g., via headers).

---

## 8. Kafka Streams & Connect

### Q41. What is Kafka Streams?
A Java library for building **stream processing** apps using Kafka topics as input/output. Provides stateful operations (aggregations, joins, windowing) backed by RocksDB and changelog topics. No separate cluster needed — runs inside your service.

### Q42. KStream vs KTable vs GlobalKTable?
- **KStream** – record stream (each record is an independent event).
- **KTable** – changelog stream (latest value per key); local, partitioned.
- **GlobalKTable** – fully replicated table on every instance, useful for joins on non-co-partitioned data.

### Q43. Sample Streams topology
```java
StreamsBuilder builder = new StreamsBuilder();
builder.stream("orders", Consumed.with(Serdes.String(), orderSerde))
    .filter((k, o) -> o.getAmount() > 100)
    .groupBy((k, o) -> o.getCustomerId())
    .count()
    .toStream()
    .to("high-value-customer-counts", Produced.with(Serdes.String(), Serdes.Long()));
```

### Q44. What is Kafka Connect?
A framework for **integrating Kafka with external systems** via reusable **source** and **sink** connectors (no code). Examples: JDBC, Debezium (CDC), Elasticsearch, S3, MongoDB.

### Q45. ksqlDB?
A streaming SQL engine on top of Kafka Streams. Define streams/tables and run SQL-like queries to transform, aggregate, and join.

---

## 9. Schema Registry

### Q46. What is a schema registry?
Centralized service (Confluent Schema Registry, AWS Glue) storing **schemas** for messages (Avro, Protobuf, JSON Schema). Producers register/look up schemas; consumers fetch them by ID. Enables schema evolution and validation.

### Q47. Compatibility modes?
- **BACKWARD** – new schema can read old data (default).
- **FORWARD** – old schema can read new data.
- **FULL** – both.
- **NONE** – no checks.
Pick based on whether consumers or producers upgrade first.

### Q48. Why Avro/Protobuf over plain JSON?
- Compact binary encoding (smaller, faster).
- Strict, evolvable schemas.
- Code generation in many languages.
- Cross-team contract enforcement.

---

## 10. Spring for Apache Kafka

### Q49. Configuration
```yaml
spring:
  kafka:
    bootstrap-servers: localhost:9092
    consumer:
      group-id: order-service
      auto-offset-reset: earliest
      enable-auto-commit: false
      key-deserializer: org.apache.kafka.common.serialization.StringDeserializer
      value-deserializer: org.springframework.kafka.support.serializer.JsonDeserializer
      properties:
        spring.json.trusted.packages: com.example.events
    producer:
      acks: all
      key-serializer: org.apache.kafka.common.serialization.StringSerializer
      value-serializer: org.springframework.kafka.support.serializer.JsonSerializer
    listener:
      ack-mode: manual
```

### Q50. Producer with `KafkaTemplate`
```java
@Service
public class OrderPublisher {
    private final KafkaTemplate<String, OrderEvent> template;

    public void publish(OrderEvent event) {
        template.send("orders", event.getOrderId(), event)
                .whenComplete((res, ex) -> {
                    if (ex != null) log.error("Send failed", ex);
                });
    }
}
```

### Q51. Consumer with `@KafkaListener`
```java
@Component
public class OrderConsumer {
    @KafkaListener(topics = "orders", groupId = "order-service")
    public void onMessage(OrderEvent event, Acknowledgment ack) {
        try {
            handle(event);
            ack.acknowledge();
        } catch (Exception e) {
            // Let DefaultErrorHandler retry / send to DLT
            throw e;
        }
    }
}
```

### Q52. Dead Letter Topic with Spring
```java
@Bean
public DefaultErrorHandler errorHandler(KafkaTemplate<String, Object> template) {
    DeadLetterPublishingRecoverer recoverer =
        new DeadLetterPublishingRecoverer(template,
            (rec, ex) -> new TopicPartition(rec.topic() + ".DLT", rec.partition()));
    return new DefaultErrorHandler(recoverer, new FixedBackOff(1000L, 3));
}
```

### Q53. Concurrent listeners
```java
@KafkaListener(topics = "orders", concurrency = "4")
public void onMessage(OrderEvent event) { ... }
```
Up to `min(concurrency, partitions)` threads will process partitions in parallel.

---

## 11. Operations & Troubleshooting

### Q54. Common monitoring metrics?
- **Broker:** under-replicated partitions, request latency, disk usage, ISR shrinks.
- **Producer:** `record-error-rate`, `request-latency-avg`, batch size, retries.
- **Consumer:** **consumer lag** (`records-lag-max`), commit rate, rebalance count.
- **JVM/OS:** GC pauses, page cache hit rate.

### Q55. What is consumer lag and how do you reduce it?
Lag = `latestOffset - currentOffset` per partition.
**Reduce by:**
- Adding consumer instances (up to partition count).
- Increasing partition count (requires careful planning – keys to partitions are not stable).
- Faster processing (async I/O, batching, parallelism inside the consumer).
- Tuning `max.poll.records` and `max.poll.interval.ms`.

### Q56. What is `max.poll.interval.ms`?
Max time a consumer can go between `poll()` calls before it is considered failed and removed from the group (triggers rebalance). Increase if processing is slow; otherwise process asynchronously.

### Q57. How to scale Kafka?
- Add **brokers**; reassign partitions to balance.
- Add **partitions** to topics for more parallelism (note: changes key distribution).
- Add **consumers** in a group up to partition count.
- Tune client batching, compression, and broker resources.

### Q58. How to choose number of partitions?
- Estimate target throughput per partition (typical: tens of MB/s).
- Aim for `target_throughput / per_partition_throughput` partitions.
- Consider future scaling (over-provision a bit).
- Watch out for too many – open file handles, replication overhead, longer leader-election times.

### Q59. Common production pitfalls?
- Single broker / replication factor 1 in production.
- Auto-creating topics with default settings.
- `acks=1` for critical data.
- Auto-commit enabled with non-idempotent processing.
- Massive consumer lag from `max.poll.records` set too high but slow processors.
- Hot keys causing partition skew.
- Ignoring DLT — failed messages clog consumers.

### Q60. Useful Kafka CLI commands?
```bash
# List topics
kafka-topics.sh --bootstrap-server localhost:9092 --list

# Create topic
kafka-topics.sh --bootstrap-server localhost:9092 \
  --create --topic orders --partitions 6 --replication-factor 3

# Describe topic
kafka-topics.sh --bootstrap-server localhost:9092 --describe --topic orders

# Console producer/consumer
kafka-console-producer.sh --bootstrap-server localhost:9092 --topic orders
kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic orders --from-beginning

# Consumer group lag
kafka-consumer-groups.sh --bootstrap-server localhost:9092 \
  --describe --group order-service

# Reset offsets
kafka-consumer-groups.sh --bootstrap-server localhost:9092 \
  --group order-service --topic orders --reset-offsets --to-earliest --execute
```
