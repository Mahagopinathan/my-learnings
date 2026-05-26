# Interview Preparation – Java, Spring, Hibernate, Microservices

A structured, hands-on study set for backend interviews. Each file contains questions grouped by topic with concise, interview-ready answers (and code where useful).

## Contents

| File | Topics | # Questions |
|------|--------|-------------|
| [java-interview-questions.md](./java-interview-questions.md) | Core Java, OOP, Strings, Collections, Exceptions, Concurrency, JVM/GC, Java 8+, Coding | 55 |
| [spring-interview-questions.md](./spring-interview-questions.md) | IoC/DI, Bean lifecycle, Spring MVC, Spring Boot, Spring Data, AOP, Security | 55 |
| [hibernate-interview-questions.md](./hibernate-interview-questions.md) | ORM, Entities & Mappings, Persistence Context, Fetching, Caching, JPQL/Criteria, Transactions, Performance | 50 |
| [microservices-interview-questions.md](./microservices-interview-questions.md) | Fundamentals, Design, Communication, Discovery, Resilience, Data, Security, Observability, Spring Cloud | 56 |

**Total: ~216 questions**

---

## How to use this guide

1. **Start broad, then go deep.** First read each file end-to-end to spot any unfamiliar concepts. Then revisit the weak areas with hands-on practice.
2. **Talk out loud.** For each question, try to answer it as if in an interview — concise but complete (~60–90 seconds per question).
3. **Code the snippets yourself.** Don't just read. Type out the singleton, the `@Transactional`, the Saga flow, the Resilience4j config. Muscle memory helps in coding rounds.
4. **Map answers to your projects.** Pick 2–3 examples from your real work for each major topic so you can answer "Tell me a time…" naturally.
5. **Final 48 hours.** Re-read only the bolded keywords and code blocks for a quick refresh.

---

## Recommended study order

Day 1–2 — **Java core** (collections, concurrency, JVM, Java 8+).
Day 3–4 — **Spring** (IoC/DI, MVC, Boot, transactions).
Day 5 — **Hibernate / JPA** (lifecycle, fetching, N+1, caching).
Day 6–7 — **Microservices** (patterns, resilience, observability, Spring Cloud).
Day 8 — **Mock interviews** + revisit weak spots.

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

---

## Suggested follow-up topics (not in these files)

- **System design** – design Twitter/URL shortener/Uber dispatch.
- **Design patterns** – Singleton, Factory, Strategy, Observer, Builder, Decorator.
- **Databases** – indexes, B-trees, transactions, replication, sharding.
- **Kafka** – consumer groups, partitions, exactly-once, retention.
- **DevOps** – Docker, Kubernetes basics, GitHub Actions/Jenkins.
- **DSA** – arrays/strings, hash maps, two pointers, sliding window, BFS/DFS, dynamic programming basics.

Good luck with your prep!
