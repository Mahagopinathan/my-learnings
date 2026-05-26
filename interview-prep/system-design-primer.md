# System Design Primer for Interviews

A practical primer for system design rounds: core concepts, building blocks, and worked-out designs for common interview problems (URL shortener, rate limiter, chat, news feed, etc.).

---

## Table of Contents
1. [Interview Approach](#1-interview-approach)
2. [Core Concepts](#2-core-concepts)
3. [Building Blocks](#3-building-blocks)
4. [Caching Patterns](#4-caching-patterns)
5. [Database Choices](#5-database-choices)
6. [Worked Designs](#6-worked-designs)
   - [URL Shortener (TinyURL/bit.ly)](#61-url-shortener)
   - [Rate Limiter](#62-rate-limiter)
   - [Distributed ID Generator](#63-distributed-id-generator)
   - [News Feed (Twitter-like)](#64-news-feed)
   - [Chat / Messenger](#65-chat--messenger)
   - [Notification Service](#66-notification-service)
   - [Web Crawler](#67-web-crawler)
   - [Video Streaming (YouTube/Netflix)](#68-video-streaming)
   - [Distributed Cache (Redis-like)](#69-distributed-cache)
   - [Search Autocomplete](#610-search-autocomplete)
7. [Common Trade-offs](#7-common-trade-offs)

---

## 1. Interview Approach

### A repeatable framework (~45 min)

1. **Clarify requirements (5 min)**
   - Functional: what features, what users do.
   - Non-functional: scale (DAU, QPS), latency, availability, consistency, durability, cost.
   - Out-of-scope: be explicit.

2. **Estimate scale (5 min)**
   - Users (DAU), QPS (read & write), data per record, total storage per year.
   - Read/write ratio (often 10:1 or 100:1 for content).
   - Bandwidth.

3. **Define API (5 min)**
   - Resource-style endpoints (`POST /shorten`, `GET /:code`).
   - Request/response shapes.

4. **High-level design (10 min)**
   - Client → LB → Service → Cache/DB → async pipelines.
   - Identify components and data flow.

5. **Deep dive into key components (15 min)**
   - Data model, indexes, partitioning, replication.
   - The most interesting/risky parts (rate limiting, ID generation, caching, fanout).

6. **Bottlenecks & scale-up (5 min)**
   - Identify hot spots, single points of failure.
   - Discuss horizontal scaling, sharding, replication, async processing, CDN, etc.

### Estimation cheat-sheet
- 1 day = 86,400 sec ≈ **10⁵ s**.
- 1 million per day ≈ **12 QPS** average.
- 1 GB ≈ 10⁹ bytes; 1 TB ≈ 10¹².
- L1 cache ≈ 1 ns; RAM ≈ 100 ns; SSD random read ≈ 100 µs; HDD seek ≈ 10 ms; cross-DC RTT ≈ 100+ ms.

---

## 2. Core Concepts

### Q1. CAP theorem
A distributed system can give at most **two** of:
- **Consistency** – all nodes see the same data.
- **Availability** – every request gets a response.
- **Partition tolerance** – the system keeps working despite network partitions.

In practice, partitions happen → choose **CP** or **AP**.

### Q2. PACELC
Extends CAP: **if Partition, choose A vs C; Else (no partition), choose Latency vs Consistency.** Most real systems trade latency for consistency in normal operation.

### Q3. ACID vs BASE
- **ACID** (RDBMS) – Atomicity, Consistency, Isolation, Durability.
- **BASE** (NoSQL) – Basically Available, Soft state, Eventual consistency.

### Q4. Consistency models
- **Strong** – read returns the latest write (e.g., Spanner).
- **Linearizable** – strong + total order across nodes.
- **Sequential** – operations appear in some consistent global order.
- **Causal** – preserves cause-effect order.
- **Eventual** – converges given no new writes.

### Q5. Vertical vs horizontal scaling
- **Vertical** – bigger machine. Simple but bounded, single point of failure.
- **Horizontal** – more machines. Scales further but introduces distributed-system complexity.

### Q6. Latency vs throughput
- **Latency** – time for one request (ms).
- **Throughput** – requests per second.
A system can have high throughput with high latency (batch) or low latency with limited throughput.

### Q7. Availability targets
| Nines | Downtime/year |
|-------|---------------|
| 99% (two nines) | ~3.65 days |
| 99.9% | ~8.76 hours |
| 99.99% | ~52.6 minutes |
| 99.999% | ~5.26 minutes |

---

## 3. Building Blocks

### Q8. Load balancers
- **L4** – TCP/UDP level (NLB, IPVS).
- **L7** – HTTP-aware, can route by path/header (nginx, HAProxy, ALB, Envoy).
- Strategies: round-robin, least-connections, IP-hash, weighted, consistent hash.
- Provide health checks and SSL termination.

### Q9. CDN
Edge servers caching static (and sometimes dynamic) content close to users. Reduce latency, offload origin, cut bandwidth costs. Examples: CloudFront, Cloudflare, Akamai, Fastly.

### Q10. Reverse proxy vs API gateway
- **Reverse proxy** (nginx) – TLS, routing, caching, request limiting.
- **API gateway** – adds auth, rate limiting, transformation, aggregation, billing, schema validation. Examples: Kong, AWS API Gateway, Apigee.

### Q11. Message queues vs streams
- **Queue** (RabbitMQ, SQS) – work distribution; one message → one consumer.
- **Stream/log** (Kafka, Kinesis) – many consumers, replay, retention, ordered partitions.

### Q12. Async patterns
- **Pub/Sub** – fan-out events.
- **Work queue** – distribute tasks to workers.
- **Saga** – long-running multi-step transactions.
- **Outbox** – write business + event in one DB tx, relay later.
- **Backpressure** – flow control to avoid overload.

### Q13. Service discovery
- **Client-side** – clients query registry (Eureka, Consul) and load balance.
- **Server-side** – clients hit a fixed LB (Kubernetes Service, AWS ELB) which routes.

### Q14. Sharding
Splitting data across nodes by key:
- **Range-based** – sorted, supports range queries; risk of hotspots.
- **Hash-based** – uniform, no range queries.
- **Consistent hashing** – minimizes movement when nodes are added/removed.
- **Directory-based** – metadata service maps key→shard.

### Q15. Replication
- **Leader–follower** – writes to leader, reads from leader/followers; failover via election.
- **Multi-leader** – writes accepted on multiple leaders; conflict resolution required.
- **Leaderless** (Dynamo-style) – writes go to N nodes; quorum reads/writes (R + W > N).

---

## 4. Caching Patterns

### Q16. Where to cache
- **Browser / app** – fastest, near user.
- **CDN** – static assets, API responses.
- **Reverse proxy** – HTTP responses.
- **Application** – in-process (Caffeine, Guava).
- **Distributed cache** – Redis/Memcached.
- **Database** – query/result cache.

### Q17. Cache strategies
- **Cache-aside (lazy load)** – app reads cache first; on miss, loads from DB and populates. Most common.
- **Read-through** – cache fetches from DB itself.
- **Write-through** – write goes to cache, which writes to DB.
- **Write-back (write-behind)** – write to cache, async to DB. Risk on cache failure.
- **Refresh-ahead** – refresh hot keys before they expire.

### Q18. Cache eviction policies
- **LRU** – least recently used (most common).
- **LFU** – least frequently used.
- **FIFO**, **TTL-based**, **Random**.

### Q19. Cache problems
- **Cold cache** – mass miss after restart → DB stampede.
- **Thundering herd** – many concurrent misses on same key → use locks or "single-flight".
- **Cache stampede / dog-pile** – use jitter on TTL.
- **Stale data** – use shorter TTL or event-based invalidation.
- **Hot keys** – sharded keys, local caching, or replicate hot key to multiple nodes.

---

## 5. Database Choices

### Q20. SQL vs NoSQL
- **SQL** – complex relations, ACID, strong consistency, structured schema (Postgres, MySQL).
- **NoSQL** flavors:
  - **Key-Value** (Redis, DynamoDB) – simple, fast lookups.
  - **Document** (MongoDB, Couchbase) – flexible JSON-ish data.
  - **Column** (Cassandra, HBase) – wide rows, write-heavy, time series.
  - **Graph** (Neo4j) – relationships are the main query.

### Q21. When to denormalize?
For read-heavy workloads where joins are too slow or impossible at scale (NoSQL). Trade write complexity for read speed. Use materialized views or async pipelines to keep duplicates in sync.

### Q22. Indexing trade-offs?
Reads faster, writes slower (every index updated). Storage overhead. Indexes shouldn't be added blindly.

---

## 6. Worked Designs

### 6.1 URL Shortener
**Functional:** `POST /shorten {longUrl}` → `shortUrl`; `GET /:code` → 301/302 redirect; analytics; expiry.
**Non-functional:** 100M new URLs/day; 100:1 read/write ratio; very low latency redirect; high availability.

**Estimation**
- Writes ≈ 100M/day ≈ 1.2k QPS; reads ≈ 120k QPS.
- 7-char base62 codes → 62⁷ ≈ 3.5 trillion combinations.
- Storage: 100M * 500 bytes/record ≈ 50 GB/day → ~18 TB/year.

**API**
```
POST /api/v1/shorten   { longUrl, customAlias?, expiresAt? } -> { shortUrl }
GET  /:code            -> 301 redirect
```

**Short code generation – options**
1. **Base62 of an auto-increment ID** – simple but exposes order; needs distributed ID (Snowflake / DB sequence).
2. **Hash + truncate** (MD5 / SHA-256) – collisions possible → check & retry.
3. **Pre-generated codes** in a "available codes" table – fast, simple, but needs to keep table topped up.

**High-level architecture**
```
Client → CDN (cache redirects) → API Gateway → Shortener Service → Cache (Redis, code→longUrl) → DB (Cassandra/Postgres)
                                                                   └→ Analytics pipeline (Kafka → Spark)
```

**Data model**
- `urls(code PK, long_url, user_id, created_at, expires_at, click_count)`.

**Key decisions**
- Cache hot codes in Redis (LRU) – most reads served from cache.
- Use 301 vs 302: 302 lets you change destination later (and lets analytics fire).
- Sharding by code (consistent hash). Cassandra works well for huge KV workload.
- Async analytics – fire an event to Kafka on each redirect; aggregate downstream.

**Scaling considerations**
- Reads: scale via cache + read replicas + CDN.
- Writes: distributed ID generator (Snowflake) avoids DB contention.
- Hot links → shard cache, replicate hot keys.

---

### 6.2 Rate Limiter
**Goal:** allow N requests per user/IP/API key per time window; block/throttle excess.

**Where to deploy?**
- API gateway (preferred for cross-service rules).
- Per-service for local fairness.
- Sidecar / service mesh.

**Algorithms**

| Algorithm | How it works | Pros | Cons |
|-----------|--------------|------|------|
| **Fixed window counter** | Counter per user per fixed window (e.g., per minute) | Simple | Bursts at window boundary |
| **Sliding window log** | Store timestamps; count those in last N seconds | Accurate | Memory-heavy |
| **Sliding window counter** | Weighted blend of current & previous window | Smooth, low memory | Approximation |
| **Token bucket** | Tokens refill at rate r, max b; one token per request | Allows bursts up to bucket size | Slightly more complex |
| **Leaky bucket** | Queue of fixed size processed at constant rate | Smooths traffic | No burst handling |

**Distributed rate limiter (Redis-based, token bucket)**
```
key = "rl:{user_id}"
  fields: tokens, lastRefillTs
1) Read current tokens & lastRefillTs.
2) Add (now - lastRefillTs) * rate tokens, capped at burst.
3) If tokens >= 1, decrement and allow. Else reject (429).
4) Persist state. Use Lua script for atomicity.
```

**Considerations**
- **Distributed counters** must be atomic (`INCR`, Lua scripts, `WATCH/MULTI/EXEC`).
- **Identity** to limit on: user id (auth'd), API key, IP (anonymous).
- **Headers**: `X-RateLimit-Limit`, `Remaining`, `Reset`.
- **Soft vs hard limits**, **graceful degradation**, **bypass for internal services**.
- **Fail open vs closed** when the limiter itself fails.

---

### 6.3 Distributed ID Generator (Snowflake-style)
**Goal:** generate unique 64-bit IDs at scale, roughly time-ordered, no central bottleneck.

**Layout (Twitter Snowflake)**
```
| sign (1) | timestamp ms (41) | machine id (10) | sequence (12) |
```
- 41 bits for ms since epoch → ~69 years.
- 10 bits machine id → 1,024 machines.
- 12 bits sequence → 4,096 IDs/ms/machine.

**Properties**
- Unique across cluster.
- Roughly sortable (good for time-based queries).
- Fully local generation – no DB calls.
- Clock skew is the enemy – monitor NTP, reject backward jumps.

**Alternative approaches**
- DB auto-increment with stride per node (`step=N`).
- ULID/UUID v7 – timestamp + random.
- Centralized ticket server (e.g., Flickr's two-server scheme).

---

### 6.4 News Feed
**Functional:** post, follow, view a feed of posts from people you follow, in roughly time order.
**Non-functional:** 100M DAU; reads >> writes; freshness; low latency feed load.

**Two main approaches**

1. **Fan-out on write (push)** – when user posts, copy to feed cache of every follower.
   - Read: super fast (just read precomputed feed).
   - Write: very expensive for celebrities (millions of followers).

2. **Fan-out on read (pull)** – on feed view, fetch latest posts from each followee and merge.
   - Write: cheap.
   - Read: expensive at scale.

3. **Hybrid (Twitter-style)**
   - Fan-out on write for normal users.
   - Pull on read for celebrities; merged with pre-computed feed at read time.

**Components**
- Post service → DB + push event to fanout pipeline.
- Fanout workers → write to per-user feed in Redis (capped list of post IDs).
- Feed service → read user's feed list from Redis, hydrate posts.
- Timeline cache → most recent N posts per user.
- Search/notification pipelines off the same event stream.

**Storage**
- Posts: NoSQL/wide-column (Cassandra) for scalability.
- User graph (follow): graph DB or a wide-column keyed by user.
- Feeds: Redis (sorted sets keyed by user).

---

### 6.5 Chat / Messenger
**Functional:** 1:1, group chat, presence, read receipts, push notifications, history.
**Non-functional:** real-time delivery, scale, message ordering per conversation.

**Connection layer**
- Long-lived **WebSocket** connections from client to a connection service (or use MQTT).
- Connection service is stateless w.r.t. business logic; just maintains user→socket map.
- Use **sticky** connections via consistent hash to a node.

**Message flow**
1. Sender → connection node A → message service.
2. Persist to chat history DB (e.g., Cassandra).
3. Lookup recipient's connection node B via session/presence service.
4. Forward via internal pub/sub (Kafka/Redis).
5. Recipient delivered → ack → update read receipt.

**Considerations**
- Per-conversation **monotonic message id** (or Snowflake) for ordering.
- Offline users → push notifications + sync on reconnect.
- End-to-end encryption (Signal protocol) for E2EE.
- Group chat: store membership; fan out to each member's inbox.

---

### 6.6 Notification Service
**Goal:** unified service to send email/SMS/push/in-app notifications at scale.

**Components**
- **Notification API** – clients submit notifications (template + audience + channel).
- **Template service** – versioned templates, localization.
- **Preference service** – user preferences, opt-outs, do-not-disturb hours.
- **Scheduler** – future / recurring notifications.
- **Queue** (Kafka/SQS) – decouple from delivery.
- **Channel workers** – each channel (Email/Push/SMS) consumes from queue and calls provider (SES, FCM, Twilio).
- **Idempotency store** – dedupe (`notification_id`).
- **Analytics** – delivery, open, click events to data lake.

**Design issues**
- **Rate limiting** per provider.
- **Retries with exponential backoff**.
- **Dead letter queue** for permanent failures.
- **Webhook callbacks** for delivery status.
- **A/B testing**.

---

### 6.7 Web Crawler
**Goal:** crawl billions of web pages, respect politeness, extract content, deduplicate.

**Components**
- **Frontier** – queue of URLs to fetch (priority by importance, freshness).
- **Fetcher pool** – downloads pages, respecting `robots.txt` and per-domain delays.
- **Parser** – extracts links + content.
- **Dedup service** – URL & content fingerprint (SimHash) to skip duplicates.
- **Storage** – HTML store (S3-like), parsed text in search index.
- **Politeness layer** – per-host token bucket; DNS cache.

**Design issues**
- **Distributed frontier** – shard by host so all URLs of a domain go to one worker (politeness).
- **Trap detection** – limit per-host, depth, infinite calendar URLs.
- **Recrawl scheduling** – higher priority for frequently changing pages.
- **Compliance** with `robots.txt` and rate limits.

---

### 6.8 Video Streaming (YouTube/Netflix)
**Functional:** upload, transcode, play in multiple resolutions, recommend, search.
**Non-functional:** very high bandwidth, global low-latency playback, durability.

**Upload pipeline**
1. Client uploads to ingest endpoint → object storage (S3).
2. Trigger transcoding jobs (multiple bitrates/resolutions, codecs, segments for HLS/DASH).
3. Generate thumbnails, captions, metadata.
4. Persist to metadata DB; mark video "ready".

**Playback pipeline**
1. Client requests playback → metadata service.
2. Service returns CDN URL + manifest (HLS/DASH `.m3u8` / `.mpd`).
3. Player fetches segments adaptively (ABR – adaptive bitrate).
4. CDN edges serve segments; misses pull from origin (S3).

**Other components**
- Recommendation service (offline ML pipelines + online ranking).
- Search index (Elasticsearch) on titles/descriptions.
- View counters, watch history (eventual consistency OK).
- DRM for premium content.

---

### 6.9 Distributed Cache (Redis-like)
**Goal:** in-memory KV store, scale horizontally, HA.

**Components**
- **Hash partitioning** – consistent hashing for keys → nodes.
- **Replication** – primary/replica per shard for failover.
- **Coordination** – cluster metadata, slot map (Redis Cluster uses 16k slots).
- **Eviction** (LRU/LFU) when memory full.
- **Persistence** – snapshots (RDB) + append-only log (AOF) for durability.

**Design issues**
- Hot keys → multi-replica reads, request coalescing.
- Failure detection & failover (Sentinel, Cluster).
- Re-sharding without downtime.

---

### 6.10 Search Autocomplete
**Goal:** suggest top queries as the user types, in <100ms.

**Approach**
- Build a **trie** of popular queries with frequency counts; at each node store top-k child suggestions.
- Update offline from query logs (aggregated nightly / hourly).
- Serve from in-memory service; shard by prefix.
- For personalization, blend global trie with per-user history at request time.

**Considerations**
- Real-time updates via streaming (small delta on top of nightly batch).
- Cache responses by prefix.
- Handle typos with edit distance / phonetic matching.

---

## 7. Common Trade-offs

- **Strong vs eventual consistency** – critical paths (payments, auth) need strong; analytics, feeds, counts can be eventual.
- **SQL vs NoSQL** – relational queries vs scale & schema flexibility.
- **Read replicas vs caching** – replicas extend read capacity but still hit DB; caches absorb most reads but can be stale.
- **Sync vs async** – sync is simple but couples failure domains; async scales but needs reliability patterns.
- **Push vs pull** for fan-out – push is fast at read time but expensive for hubs; pull is cheap to write but expensive to read.
- **Monolith vs microservices** – microservices help large orgs scale ownership but add operational complexity.
- **Build vs buy** – managed services (RDS, MSK, DynamoDB) trade flexibility/cost for speed and reliability.

### Final tips for the interview
- **Ask** before assuming. Clarify scope and constraints.
- **Make trade-offs explicit.** "I'm choosing X over Y because…"
- **Drive the discussion.** Don't wait for the interviewer to ask "what about scale?".
- **Use real-world numbers** to back up choices.
- **It's okay to revise.** Better to identify and fix a flaw than to defend a bad choice.
