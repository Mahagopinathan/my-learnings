# Microservices Interview Questions & Answers

Covers microservice architecture, communication patterns, resilience, observability, deployment, and Spring Cloud.

---

## Table of Contents
1. [Fundamentals](#1-fundamentals)
2. [Design & Decomposition](#2-design--decomposition)
3. [Communication: Sync & Async](#3-communication-sync--async)
4. [Service Discovery & API Gateway](#4-service-discovery--api-gateway)
5. [Resilience & Fault Tolerance](#5-resilience--fault-tolerance)
6. [Data Management](#6-data-management)
7. [Security](#7-security)
8. [Observability](#8-observability)
9. [Deployment & DevOps](#9-deployment--devops)
10. [Spring Cloud Specific](#10-spring-cloud-specific)
11. [Scenario / Trade-offs](#11-scenario--trade-offs)

---

## 1. Fundamentals

### Q1. What are microservices?
An architectural style where an application is composed of small, **independently deployable**, loosely coupled services, each owning a single business capability and its own data.

### Q2. Microservices vs Monolith?
| Aspect | Monolith | Microservices |
|--------|----------|---------------|
| Codebase | Single | Many |
| Deployment | One unit | Independent |
| Scaling | All-or-nothing | Per-service |
| Tech stack | Uniform | Polyglot |
| Failure isolation | Low | High |
| Operational overhead | Low | High |
| Best fit | Small/medium teams | Large teams, complex domains |

### Q3. When should you NOT use microservices?
- Small team / small product.
- Strong transactional consistency requirements throughout.
- Lack of DevOps maturity (CI/CD, monitoring, automation).
- Domain isn't well understood yet — start with a **modular monolith** and split later.

### Q4. Characteristics / principles of microservices?
- Single responsibility (bounded context).
- Independent deployability.
- Decentralized data ownership (DB per service).
- Smart endpoints, dumb pipes.
- Designed for failure.
- Automation: CI/CD, IaC.
- Observable.

### Q5. SOA vs Microservices?
- SOA – heavier (often ESB-based), shared schemas/databases.
- Microservices – lighter, smart endpoints / dumb pipes, DB per service, no central ESB.

---

## 2. Design & Decomposition

### Q6. How do you decompose a monolith into microservices?
Use **Domain-Driven Design (DDD)**:
1. Identify **bounded contexts**.
2. Map entities/aggregates to a single service.
3. Use the **Strangler Fig** pattern – gradually replace pieces of the monolith via routing at an edge proxy.
4. Split data per bounded context, migrate cautiously.

### Q7. What is a bounded context?
A DDD concept: a logical boundary within which a particular model is consistent. Each microservice typically maps to one bounded context.

### Q8. What is the database-per-service pattern?
Every microservice owns its data store privately. Other services access the data only through the owning service's API. Promotes loose coupling and independent scaling/evolution of schemas.

### Q9. How do services communicate without a shared DB?
- Synchronous: REST, gRPC.
- Asynchronous: events / messages via Kafka, RabbitMQ.
- API composition / data replication / CQRS for cross-service queries.

---

## 3. Communication: Sync & Async

### Q10. Synchronous vs asynchronous communication?
- **Sync** (REST, gRPC) – simple, but tighter coupling and lower availability (failures cascade).
- **Async** (events, messages) – loose coupling, better resilience, eventual consistency, more operational complexity.

### Q11. REST vs gRPC?
| Aspect | REST | gRPC |
|--------|------|------|
| Protocol | HTTP/1.1, JSON | HTTP/2, Protobuf |
| Performance | Lower | Higher |
| Streaming | Limited | Bidirectional |
| Browser support | Native | Needs gRPC-Web |
| Schema | OpenAPI | `.proto` (strict) |

### Q12. What is event-driven architecture?
Services communicate by producing and consuming **events** asynchronously through a broker. Promotes loose coupling; services don't need to know about each other.

### Q13. Common message brokers?
- **Kafka** – distributed log, high throughput, replay-able, good for event streaming.
- **RabbitMQ** – AMQP, flexible routing (exchanges/queues), good for tasks & RPC patterns.
- **ActiveMQ**, **AWS SQS/SNS**, **Google Pub/Sub**, **Azure Service Bus**.

### Q14. At-most-once, at-least-once, exactly-once delivery?
- **At-most-once** – may lose messages.
- **At-least-once** – may duplicate. Most common; consumers must be idempotent.
- **Exactly-once** – complex (transactional outbox + idempotent consumers + Kafka EOS).

### Q15. How to make consumers idempotent?
- Track processed message IDs (e.g., in DB / Redis).
- Use unique business keys for DB upserts.
- Make handlers naturally idempotent (e.g., "set status = X").

---

## 4. Service Discovery & API Gateway

### Q16. What is service discovery?
A mechanism for services to find each other dynamically without hardcoded URLs.
- **Client-side discovery** – client queries a registry (e.g., Netflix Eureka) and load-balances itself.
- **Server-side discovery** – clients hit a load balancer / gateway that resolves services (e.g., Kubernetes Service, AWS ALB).

### Q17. What is an API Gateway?
A single entry point that fronts the microservices. Responsibilities:
- Routing & composition.
- Authentication / authorization.
- Rate limiting & throttling.
- Caching, request/response transformation.
- Logging, metrics, tracing.

Examples: **Spring Cloud Gateway**, Kong, Apigee, AWS API Gateway, Nginx, Zuul (legacy).

### Q18. What problems does an API Gateway solve?
- Avoids exposing internal service topology to clients.
- Centralizes cross-cutting concerns.
- Reduces chatty calls from clients (gateway aggregates).
- Provides **BFF** (Backend-for-Frontend) per client type (web/mobile).

### Q19. Disadvantages of API Gateway?
- Single point of failure if not HA.
- Can become a bottleneck.
- Adds latency.
- Risk of becoming a "smart pipe" – keep business logic out of it.

---

## 5. Resilience & Fault Tolerance

### Q20. What is the Circuit Breaker pattern?
Prevents cascading failures by **short-circuiting** calls to a failing service.
States:
- **Closed** – calls pass through; failures counted.
- **Open** – fails fast without calling the service for a cool-down period.
- **Half-Open** – limited calls allowed to test recovery.

Implementations: **Resilience4j** (recommended), Hystrix (deprecated).

### Q21. What is the Bulkhead pattern?
Isolate resources (thread pools, connections) per dependency so that one slow dependency doesn't exhaust resources for others, preventing total failure.

### Q22. What is retry with exponential backoff?
Re-attempt failed calls, increasing delay (e.g., 1s, 2s, 4s, 8s) and jitter to avoid thundering herd. Combine with idempotency. Always cap retries.

### Q23. Timeouts – why are they critical?
Without timeouts, a slow downstream call blocks threads, leading to thread pool exhaustion and cascading outage. Always set explicit, reasonable timeouts.

### Q24. Resilience4j key modules?
- **CircuitBreaker**, **Retry**, **RateLimiter**, **Bulkhead**, **TimeLimiter**, **Cache**.
- Lightweight, functional-style, integrates with Spring Boot via starter.

### Q25. Rate limiting strategies?
- Token bucket / leaky bucket.
- Fixed/sliding window counters.
- Distributed: Redis-backed counters, API Gateway plugins.

### Q26. Fallback pattern?
When a dependency fails, return a sensible default (cached data, empty result, degraded response) instead of an error. Implemented with `@CircuitBreaker(fallbackMethod = "...")` in Resilience4j.

---

## 6. Data Management

### Q27. How do you maintain data consistency across services?
You usually trade ACID for **eventual consistency**:
- **Saga pattern** – sequence of local transactions with compensating actions.
- **Event sourcing** – store events as the source of truth.
- **CQRS** – separate read/write models.
- **Transactional Outbox** – atomically write business data + outbox event in one DB tx; a relay publishes to broker.

### Q28. What is the Saga pattern?
A long-running business transaction split into **local transactions per service**, each emitting an event that triggers the next.
Two flavors:
- **Choreography** – services react to each other's events. Simple but can become a tangled web.
- **Orchestration** – a central saga orchestrator dictates next steps. Easier to reason about.

### Q29. What is CQRS?
**Command Query Responsibility Segregation** – separate models for writes (commands) and reads (queries). Often paired with event sourcing. Good for read-heavy systems with complex reporting needs.

### Q30. What is event sourcing?
Persist every state change as an immutable event. Current state is derived by replaying events. Provides full auditability and time-travel debugging, at the cost of complexity (versioning events, rebuilding read models).

### Q31. What is the transactional outbox pattern?
Solves the dual-write problem (write to DB + publish to broker atomically):
1. In the same DB transaction, write the business change AND insert a row in an `outbox` table.
2. A separate relay (e.g., Debezium / a Kafka Connector / a poller) reads the outbox and publishes to the broker reliably.

### Q32. How do you query data spread across services?
- **API composition** – aggregator service calls each.
- **CQRS read model** – materialized view kept up to date via events.
- **Backend-for-Frontend (BFF)** to assemble client-specific responses.
- Avoid cross-service joins on the DB layer.

---

## 7. Security

### Q33. How do you secure microservices?
- **AuthN at the edge** (API Gateway).
- **JWT / OAuth2** for token-based auth between services.
- **mTLS** for service-to-service identity (often via service mesh).
- **Centralized identity provider** (Keycloak, Okta, Cognito).
- Principle of least privilege; secrets in vaults.

### Q34. OAuth2 grant types (commonly used)?
- **Authorization Code (with PKCE)** – web/mobile/SPA users.
- **Client Credentials** – service-to-service.
- **Refresh Token** – renew access without re-auth.
- (Deprecated): Implicit, Resource Owner Password.

### Q35. JWT structure?
`header.payload.signature` – Base64URL-encoded.
- Stateless, signed (HS256/RS256).
- Don't store secrets in payload (claims are readable).
- Keep tokens short-lived; use refresh tokens.

### Q36. What is a service mesh?
Infrastructure layer (sidecar proxies like Envoy) handling service-to-service communication: mTLS, retries, traffic shifting, observability — without changing application code. Examples: **Istio**, **Linkerd**, **Consul Connect**.

---

## 8. Observability

### Q37. The three pillars of observability?
- **Logs** – discrete events.
- **Metrics** – numeric time-series (latency, error rate, throughput).
- **Traces** – end-to-end request flows across services.

### Q38. How do you implement distributed tracing?
- Propagate a **trace id** + **span ids** through headers (W3C Trace Context).
- Tools: **OpenTelemetry** (standard), **Zipkin**, **Jaeger**, **AWS X-Ray**, vendor APMs (Datadog, New Relic, Dynatrace).
- In Spring Boot 3, **Micrometer Tracing** replaces Spring Cloud Sleuth.

### Q39. How to centralize logs?
- Structured JSON logs.
- Ship via Fluentd/Fluent Bit/Filebeat to **ELK** (Elasticsearch/Logstash/Kibana), **Loki/Grafana**, **Splunk**, or cloud equivalents.
- Always include `traceId`, `spanId`, `serviceName`.

### Q40. What metrics should you monitor?
- **The four golden signals**: latency, traffic, errors, saturation.
- Per service: request rate, error rate, p50/p95/p99 latency, JVM memory/GC, thread pool, DB connection pool, circuit breaker state.

### Q41. Health checks?
- **Liveness** – is the service running? Restart if not.
- **Readiness** – is it ready to serve traffic? Pull out of LB if not.
- **Startup** – allow slow-starting apps extra time before liveness kicks in.
- Spring Boot Actuator: `/actuator/health/liveness`, `/actuator/health/readiness`.

---

## 9. Deployment & DevOps

### Q42. Why containers (Docker) for microservices?
- Consistent runtime across environments.
- Lightweight, fast startup.
- Standardized packaging, easy CI/CD.
- Foundation for orchestration (Kubernetes).

### Q43. Why Kubernetes?
Container orchestration: scheduling, scaling, self-healing, service discovery, rolling updates, config & secret management, declarative infrastructure.

### Q44. Deployment strategies?
- **Rolling update** – default, gradually replaces pods.
- **Blue-Green** – switch traffic between two identical environments.
- **Canary** – send a small % of traffic to new version, ramp up.
- **Shadow / dark launch** – mirror production traffic to new version for testing.

### Q45. 12-Factor App principles (key ones)?
- Codebase tracked in version control.
- Explicit dependencies.
- Config from environment.
- Backing services as attached resources.
- Build, release, run separated.
- Stateless processes.
- Port binding.
- Concurrency via process model.
- Disposability (fast startup, graceful shutdown).
- Dev/prod parity.
- Logs as event streams.
- Admin processes as one-off tasks.

### Q46. How do you handle configuration in microservices?
- Externalized config: env variables, config server.
- **Spring Cloud Config**, **Consul KV**, **Vault**, **AWS Parameter Store/Secrets Manager**, Kubernetes ConfigMaps/Secrets.
- Refresh dynamically via `@RefreshScope` / actuator `/refresh`.

---

## 10. Spring Cloud Specific

### Q47. What is Spring Cloud?
A set of tools building on Spring Boot for common distributed-system patterns: config, discovery, gateway, circuit breaker, tracing, etc.

### Q48. Key Spring Cloud projects (current)?
- **Spring Cloud Config** – centralized config server (Git-backed).
- **Spring Cloud Gateway** – reactive API gateway (replaces Zuul).
- **Spring Cloud OpenFeign** – declarative REST client.
- **Spring Cloud LoadBalancer** – client-side LB (replaces Ribbon).
- **Spring Cloud Stream** – message-driven microservices (Kafka/RabbitMQ).
- **Spring Cloud Sleuth → Micrometer Tracing** (Boot 3) – distributed tracing.
- **Spring Cloud Circuit Breaker** – abstraction over Resilience4j etc.
- (Older / community) **Eureka**, **Consul**, **Zookeeper** for discovery.

### Q49. Sample Feign client.
```java
@FeignClient(name = "order-service", url = "${order.service.url}")
public interface OrderClient {
    @GetMapping("/orders/{id}")
    OrderDto getOrder(@PathVariable Long id);
}
```
Then `@EnableFeignClients` in your config.

### Q50. Sample Resilience4j circuit breaker.
```java
@CircuitBreaker(name = "orderService", fallbackMethod = "fallback")
@Retry(name = "orderService")
public OrderDto getOrder(Long id) {
    return orderClient.getOrder(id);
}

private OrderDto fallback(Long id, Throwable ex) {
    return OrderDto.empty();
}
```
```yaml
resilience4j:
  circuitbreaker:
    instances:
      orderService:
        slidingWindowSize: 20
        failureRateThreshold: 50
        waitDurationInOpenState: 10s
        permittedNumberOfCallsInHalfOpenState: 3
```

---

## 11. Scenario / Trade-offs

### Q51. How would you design a system handling 1M requests/sec?
- Stateless services horizontally scaled behind LB.
- Aggressive caching (CDN + Redis/Caffeine).
- Async processing for non-critical work via Kafka.
- Read replicas / sharding for databases; CQRS where helpful.
- Auto-scaling on metrics; load shedding under overload.
- Bulkheads, circuit breakers, timeouts, rate limits.
- Capacity planning: load tests, SLOs, error budgets.

### Q52. How would you migrate a monolith to microservices?
1. Stabilize the monolith and add tests.
2. Identify bounded contexts; pick a low-risk first slice.
3. Apply **Strangler Fig** behind an API gateway / proxy.
4. Extract one service + its data; route traffic gradually.
5. Iterate; build platform capabilities (CI/CD, observability) along the way.
6. Decommission monolith pieces as they're replaced.

### Q53. How do you handle versioning?
- URI: `/api/v1/users` (most common).
- Header: `Accept: application/vnd.company.v2+json`.
- Maintain backward compatibility (additive changes); deprecate, don't break.

### Q54. How do you test microservices?
- **Unit tests** – business logic.
- **Component tests** – service in isolation with mocks.
- **Contract tests** (e.g., **Spring Cloud Contract**, **Pact**) – verify producer/consumer compatibility without full e2e.
- **Integration tests** – with **Testcontainers** for DBs/brokers.
- **End-to-end tests** – sparingly; fragile and slow.

### Q55. How do you handle schema changes in events / APIs?
- Use schema registries (Confluent Schema Registry / AWS Glue) for Avro/Protobuf.
- Apply backward-compatible changes (add optional fields).
- Version events; consumers handle multiple versions during transitions.
- Avoid removing fields; deprecate first.

### Q56. Common anti-patterns to avoid?
- **Distributed monolith** – services tightly coupled, deployed together.
- **Shared database** across services.
- **Chatty services** – too many sync hops; consider redesigning the boundaries.
- Hardcoded service URLs.
- Missing timeouts / retries / circuit breakers.
- Treating microservices as the goal — they're a tool, not the goal.
