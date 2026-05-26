# Interview Preparation – Java, Spring, Hibernate, Microservices & More

A structured, hands-on study set for backend interviews. Each file contains questions grouped by topic with concise, interview-ready answers (and code where useful).

## Contents

| File | Topics | # Questions |
|------|--------|-------------|
| [java-interview-questions.md](./java-interview-questions.md) | Core Java, OOP, Strings, Collections, Exceptions, Concurrency, JVM/GC, Java 8+, Coding | 55 |
| [java-coding-logic-questions.md](./java-coding-logic-questions.md) | Number/string/array problems, patterns, recursion, searching/sorting, linked list & stack, Java 8 streams | 65 |
| [spring-interview-questions.md](./spring-interview-questions.md) | IoC/DI, Bean lifecycle, Spring MVC, Spring Boot, Spring Data, AOP, Security | 55 |
| [hibernate-interview-questions.md](./hibernate-interview-questions.md) | ORM, Entities & Mappings, Persistence Context, Fetching, Caching, JPQL/Criteria, Transactions, Performance | 50 |
| [microservices-interview-questions.md](./microservices-interview-questions.md) | Fundamentals, Design, Communication, Discovery, Resilience, Data, Security, Observability, Spring Cloud | 56 |
| [sql-database-questions.md](./sql-database-questions.md) | RDBMS basics, joins, CTEs, window functions, indexes, transactions, classic SQL query problems | 58 |
| [mongodb-interview-questions.md](./mongodb-interview-questions.md) | CRUD, operators, indexes, aggregation pipeline, schema design, replication, sharding, Spring Data MongoDB | 57 |
| [kafka-interview-questions.md](./kafka-interview-questions.md) | Architecture, partitions/replication, producers/consumers, delivery semantics, Streams/Connect, Schema Registry, Spring for Apache Kafka, ops & tuning | 60 |
| [system-design-primer.md](./system-design-primer.md) | Interview approach, CAP/PACELC, building blocks, caching, DB choices, **worked designs**: URL shortener, rate limiter, ID generator, news feed, chat, notifications, crawler, video streaming, distributed cache, autocomplete | 10 designs |
| [design-patterns-java.md](./design-patterns-java.md) | All GoF patterns (Creational, Structural, Behavioral) with runnable Java examples + SOLID + interview Q&A | 25+ patterns |
| [docker-kubernetes-questions.md](./docker-kubernetes-questions.md) | Containers vs VMs, Dockerfile/Compose, K8s architecture, pods/deployments/statefulsets, services/ingress, configmaps/secrets, HPA/probes, observability, troubleshooting, kubectl | 65 |

**Total: ~520+ questions and 10 worked system designs**

---

## How to use this guide

1. **Start broad, then go deep.** First read each file end-to-end to spot any unfamiliar concepts. Then revisit the weak areas with hands-on practice.
2. **Talk out loud.** For each question, try to answer it as if in an interview — concise but complete (~60–90 seconds per question).
3. **Code the snippets yourself.** Don't just read. Type out the singleton, the `@Transactional`, the Saga flow, the Resilience4j config. Muscle memory helps in coding rounds.
4. **Map answers to your projects.** Pick 2–3 examples from your real work for each major topic so you can answer "Tell me a time…" naturally.
5. **Final 48 hours.** Re-read only the bolded keywords and code blocks for a quick refresh.

---

## Recommended study order

Day 1–2  — **Java core** (collections, concurrency, JVM, Java 8+).
Day 3    — **Java coding/logic** (numbers, strings, arrays, streams).
Day 4–5  — **Spring** (IoC/DI, MVC, Boot, transactions).
Day 6    — **Hibernate / JPA** (lifecycle, fetching, N+1, caching).
Day 7–8  — **Microservices** (patterns, resilience, observability, Spring Cloud).
Day 9    — **SQL / databases** (joins, window functions, query problems).
Day 10   — **MongoDB** (CRUD, aggregation, indexes, schema design).
Day 11   — **Kafka** (architecture, semantics, producers/consumers, ops).
Day 12   — **Design patterns** (GoF + SOLID).
Day 13   — **Docker & Kubernetes** (containers, pods, services, deployments).
Day 14–15 — **System design** (frameworks + worked problems).
Day 16   — **Mock interviews** + revisit weak spots.

---

## Topics that show up in almost every interview

These are the "must-be-fluent" topics — make sure you can speak about them without hesitation:

- `HashMap` internals (Java 8 treeification).
- `equals`/`hashCode` contract.
- `volatile` vs `synchronized`; thread-safe singleton.
- Streams – `map` vs `flatMap`, `Collectors`.
- Spring DI & bean lifecycle; constructor injection over field injection.
- `@Transactional` propagation, isolation, common pitfalls (self-invocation, private methods).
- Hibernate N+1 problem and how to fix it.
- `LazyInitializationException` causes and proper fixes.
- Microservices: Saga pattern, Circuit Breaker, API Gateway, JWT/OAuth2.
- CAP theorem & eventual consistency basics.
- SQL: joins, GROUP BY vs HAVING, window functions (ROW_NUMBER/RANK/DENSE_RANK), Nth highest salary.
- MongoDB: embed vs reference, aggregation pipeline, compound indexes (ESR rule), shard key choice.
- Kafka: partitions/ordering, consumer groups, delivery semantics, idempotent producer + transactions.
- Design patterns: Singleton, Factory, Strategy, Observer, Decorator, Proxy, Builder, Template Method.
- Kubernetes: Pod vs Deployment vs StatefulSet, Service types, ConfigMap/Secret, probes, rolling updates.
- System design: URL shortener, rate limiter, distributed ID, news feed (push vs pull), chat (WebSocket).

---

## Suggested follow-up topics (deeper dives, not in these files)

- **DSA practice** – LeetCode top 150 patterns (two pointers, sliding window, BFS/DFS, DP).
- **Reactive Java** – Project Reactor, WebFlux, R2DBC.
- **Cloud platforms** – AWS (EKS, RDS, SQS/SNS, Lambda), GCP, or Azure equivalents.
- **GraphQL** – schema design, N+1, federation.
- **gRPC & Protocol Buffers** – streaming, deadlines, interceptors.
- **CI/CD** – GitHub Actions, Jenkins pipelines, ArgoCD/Flux for GitOps.
- **Security** – OWASP top 10, JWT pitfalls, secrets management, mTLS.

Good luck with your prep!
