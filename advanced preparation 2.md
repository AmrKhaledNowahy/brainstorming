AI Assistant

```markdown
# Technical Lead Cheat-sheet (Core Java • Design Patterns • Spring Boot • Databases • Architecture)

> Compact, printable one-page review. Adjust font-size in your editor/export settings to fit A4/Letter.

---

## Core Java — Key Topics
- **JMM & happens-before**: Defines visibility & ordering. Use `volatile` for visibility, `synchronized` for mutual exclusion + visibility, `Atomic*` for lock-free CAS updates.
- **Concurrency primitives**: `ReentrantLock`, `ReadWriteLock`, `StampedLock` (optimistic reads), `ForkJoinPool`, `CompletableFuture` (provide Executor for blocking tasks).
- **GC & memory**: Generational collectors; choose ZGC/Shenandoah/G1 for low-pause; tune heap and survivor spaces; avoid long-lived allocations causing promotion.
- **References**: Strong (normal), Soft (memory-sensitive caches), Weak (WeakHashMap, canonicalization), Phantom (ReferenceQueue for final cleanup).
- **Class loading / plugins**: Use per-plugin ClassLoader for isolation; unload by dropping references; avoid thread/static leaks.
- **Modern Java features**: Records (immutable data), Sealed classes (restricted hierarchies), pattern matching; prefer immutability and small value objects.

---

## Design Patterns — Quick Items
- **Creational**: Factory Method (subclass decides), Abstract Factory (families), Builder (complex construction).
- **Structural**: Proxy (access control/remote), Decorator (add behavior), Adapter, Facade.
- **Behavioral**: Strategy (pluggable algorithms), Template Method (fixed skeleton), Observer, Command, Circuit Breaker (resilience).
- **Architectural**: Hexagonal (ports & adapters), CQRS + Event Sourcing (auditability vs complexity), Saga (distributed transactions).
- **Trade-offs**: Prefer composition over inheritance; emphasize clear interfaces and testability.

---

## Spring Boot — Essentials & Advanced Notes
- **Auto-configuration**: Driven by conditions (`@ConditionalOnClass`, `@ConditionalOnProperty`). Override with custom `@Bean` or `spring.autoconfigure.exclude` / `@SpringBootApplication(exclude=...)`.
- **Transactions**: `@Transactional` via proxies (or AspectJ). Propagation: `REQUIRED`, `REQUIRES_NEW`, `NESTED`; Isolation: `READ_COMMITTED`, `REPEATABLE_READ`, `SERIALIZABLE`.
- **Bean lifecycles & scopes**: `@Configuration` is proxied (ensures singletons); scopes: `singleton`, `prototype` (manual destroy), `request`, `session`.
- **Batch & JPA at scale**: Use `em.flush(); em.clear()` periodically, `hibernate.jdbc.batch_size`, `StatelessSession` for bulk inserts, JDBC fetchSize for streaming reads.
- **Reactive vs MVC**: Use WebFlux (Reactor: `Mono`/`Flux`) only when the entire stack (DB drivers, clients) is non-blocking; otherwise stick with Spring MVC.
- **Async & Threading**: Configure bounded `TaskExecutor` for `@Async`; separate pools for IO vs CPU-bound tasks; set RejectedExecutionHandler.
- **Security & Ops**: OAuth2/OIDC for auth, JWT or token introspection; Actuator for health/metrics; enable graceful shutdown (`server.shutdown=graceful`).
- **Startup & profiling**: Use Actuator startup, Java Flight Recorder, async-profiler; narrow component scanning (`basePackages`) to speed bootstrap.

---

## Databases — Key Points
- **ACID & Isolation**: `READ_UNCOMMITTED` → dirty reads; `READ_COMMITTED` prevents dirty reads; `REPEATABLE_READ` prevents non-repeatable reads; `SERIALIZABLE` prevents phantoms (highest cost).
- **MVCC**: Snapshot reads without writer blocking; long-running txns cause bloat (delayed vacuum/cleanup).
- **Indexing**: Index columns used in `WHERE`/`JOIN`/`ORDER BY`. Composite indexes follow left-most rule. Use covering indexes to avoid lookups. Avoid too many indexes (write penalty).
- **Scaling**: Partitioning (range/list/hash) for large tables; sharding for horizontal scale—choose shard key aligned with access patterns to avoid cross-shard joins.
- **OLTP vs OLAP**: Normalize for writes/consistency; denormalize/materialize (materialized views) for analytic performance. Use Redis for sub-ms caches with proper invalidation/TLL strategies.
- **Diagnostics**: `EXPLAIN ANALYZE`, check locks/waits, stats, table bloat, parameter sniffing; capture representative workloads.

---

## System Architecture & Microservices
- **Monolith vs Microservices**: Use a modular monolith first. Split when teams need independent deploy/scale, or bounded contexts justify overhead.
- **Non-functional concerns**: Observability (metrics/logs/traces), resilience (timeouts, retries, circuit breakers, bulkheads), security (mTLS/OAuth2), autoscaling, CI/CD and runbooks.
- **Data consistency**: Prefer eventual consistency with Sagas (choreography or orchestration) and compensations; use idempotency keys and design for retries.
- **Comm patterns**: Synchronous (HTTP/gRPC) simpler but tight-coupling; Asynchronous (Kafka/Pulsar) decouples and scales but adds complexity (eventual consistency).
- **Event-driven design**: Durable broker, partitioning for parallelism, schema registry (Avro/Proto), idempotent consumer design, retention/compaction strategies.
- **Deployment strategies**: Canary (subset traffic, monitor SLOs), Blue-Green (atomic switch), feature flags for gradual enablement.
- **Service discovery**: Client-side (Eureka/Consul) vs server-side (API Gateway/LB) trade-offs; prefer highly available registries and gateways.
- **Failure testing**: Chaos engineering with defined blast radius, automated rollback/runbooks, and SLO-driven guardrails.

---

## Practical Design Snapshot
- Example: URL shortener
  - Components: API service (create/redirect), ID generation (base62, hash), DB (key→URL), cache (Redis) for hot keys, analytics (events → Kafka → batch), partitioning/sharding, rate-limiting, idempotency and collision handling, TTL/archival policy.

---

## Interview Strategy & Checklist
1. Ask clarifying questions; state assumptions & constraints.
2. Present high-level architecture first; then detail components, data flow, scaling and failure modes.
3. Discuss trade-offs explicitly (consistency vs availability, complexity vs performance).
4. Mention observability (metrics/logs/traces), security, testing, deployment & rollback.
5. Summarize decisions and next steps (POC, benchmarks, migration plan).

---

*Tips to fit to one page when exporting:* reduce editor font-size to 9–10pt; use narrow page margins; export to PDF with A4/Letter and "fit to page". Good luck!
```
