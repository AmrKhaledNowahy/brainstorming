AI Assistant

Below is a compact, printable cheat-sheet covering Core Java, Design Patterns, Spring Boot, Databases, and added sections for System Architecture & Microservices. Each topic contains intermediate → advanced questions with concise model answers and quick tips for follow-ups. Use this for last-minute review before interviews.

Core Java — Cheat-sheet (additional questions included)
1) Q: Explain final, finally, and finalize() and their use-cases.
   A: final — keyword for constants, non-overridable methods, and non-subclassable classes. finally — block executed after try/catch for cleanup; executes even with return (except System.exit). finalize() — deprecated cleanup hook called by GC before reclaiming an object — unreliable and should be avoided; use try-with-resources or explicit close and PhantomReference for native resource cleanup.

2) Q: Describe try-with-resources and AutoCloseable vs Closeable.
   A: try-with-resources auto-closes resources implementing AutoCloseable at end of block, handling suppressed exceptions. Closeable extends AutoCloseable and declares close() throws IOException (for IO). Use try-with-resources to avoid leaks; be mindful of ordering and suppressed exceptions.

3) Q: How does String interning work and when to use it?
   A: String literals are interned in the String pool. Calling intern() on a String adds it to the pool (if absent). Interning reduces memory for many identical strings but can increase permgen/metaspace pressure and CPU for intern() calls — use only when many duplicate short strings exist (e.g., tokens).

4) Q: Explain functional interfaces, method references, and default/private methods in interfaces.
   A: Functional interfaces have a single abstract method and enable lambdas. Method references (::) are shorthand for lambdas referencing existing methods. Java 8+ allows default and static methods in interfaces; Java 9 added private methods for code reuse inside interfaces.

5) Q: What are records (Java 16+) and when would you use them?
   A: Records are concise immutable data carriers with auto-generated equals/hashCode/toString and accessors. Use for DTOs/value objects where immutability and simple data semantics are desired.

6) Q: What are sealed classes (Java 17) and benefits?
   A: Sealed classes restrict which other classes may extend them (permits clause). Benefits: better control of inheritance hierarchies, exhaustive pattern-matching, improved reasoning about types.

7) Q: How to implement and debug classpath conflicts (jar hell)?
   A: Use mvn dependency:tree / gradle dependencies to detect duplicates, use application startup logs for classloader warnings, isolate modules, use shading/relocation, enforce BOM and version alignment. For runtime, enable verbose:class to see loaded classes.

8) Q: Deep concurrency: explain lock striping, read-write locks, and StampedLock.
   A: Lock striping partitions data into segments with separate locks to reduce contention. ReadWriteLock allows multiple concurrent readers but exclusive writers. StampedLock provides optimistic reads (tryOptimisticRead) with stamps for versions and can be faster for mostly-read workloads but lacks reentrancy.

9) Q: Serialization: Java Serializable pitfalls and alternatives.
   A: Serializable is easy but fragile (versioning, security, performance). Alternatives: Protobuf/Avro/Thrift/JSON/Binary formats. For Java serialization, define serialVersionUID, prefer explicit readObject/writeObject or Externalizable when needed, and avoid exposing internal invariants.

10) Q: JVM tuning quick checklist for high throughput vs low latency.
    A: Throughput: larger heap, parallel GC (G1 tuned for throughput), increase young gen for throughput. Low-latency: use G1 with pause targets, ZGC/Shenandoah for minimal pauses, reduce stop-the-world frequency, tune region sizes, set CPU and memory affinity, and provide separate thread pools for blocking operations.

Design Patterns — Quick additions
- Builder vs Fluent API: builder isolates construction complexity; fluent is chaining for readability.
- Observer vs Event Bus: Observer is direct subscription, Event Bus decouples publisher/consumer and supports async/event-driven architecture.
- Registry vs Service Locator: Registry stores services; Service Locator hides dependency discovery (avoid for testability; prefer DI).
- Anti-Patterns: God Object, Big Ball of Mud, Golden Hammer — know examples and mitigations.

Spring Boot — Expanded Q&A
1) Q: How does component scanning work and what are common pitfalls?
   A: @ComponentScan scans packages for stereotypes (@Component, @Service, @Repository, @Controller). Pitfalls: wide scan scopes causing unintended beans, circular dependencies, slow startup if scanning many classes, and accidentally picking up test-scoped classes. Prefer explicit basePackages and use @SpringBootApplication on root package to limit scope.

2) Q: Explain bean scopes (singleton, prototype, request, session, application) and usage.
   A: singleton — one bean per Spring container (default). prototype — new instance each injection request (no lifecycle callbacks by container after creation). request/session — web-scoped beans per HTTP request/session. application — ServletContext-scoped. Use prototype for stateful per-use objects, but manage destruction manually.

3) Q: How do @Configuration classes differ from @Component classes for bean methods?
   A: @Configuration classes are CGLIB proxied so @Bean method calls are intercepted and return singletons; @Component with @Bean lacks that proxying — multiple instances may be created if methods call one another. Use @Configuration for inter-bean references.

4) Q: How to handle circular dependencies?
   A: Constructor injection forbids circular refs; use setter or field injection (not recommended), use @Lazy on one dependency, refactor to break cycle, or extract common interface/service.

5) Q: Explain custom Spring Boot starters and best practices.
   A: Starters are opinionated dependency aggregators plus auto-configuration modules. Create starters with proper auto-config using Conditional annotations, provide properties prefix, and keep them lightweight. Document configurations and provide properties for toggling.

6) Q: How to manage configuration across environments (profiles, config servers)?
   A: Use Spring Profiles (@Profile, spring.profiles.active), externalized configuration (application-{profile}.yml), and centralized config via Spring Cloud Config or Vault for secrets. Keep environment-specific overrides outside the artifact.

7) Q: How does @EnableAsync and TaskExecutor work? How to avoid thread pool exhaustion?
   A: @EnableAsync enables async processing for methods annotated @Async; configure Executor bean with bounded queue and appropriate pool size. Avoid unbounded queues, set RejectedExecutionHandler, separate IO and CPU bound pools.

8) Q: How to implement health checks and readiness/liveness probes?
   A: Use Spring Boot Actuator endpoints (/actuator/health). Implement HealthIndicator beans for custom checks (DB connectivity, dependent services). Expose readiness/liveness by mapping Actuator endpoints and configure Kubernetes probes accordingly.

9) Q: How to detect memory leaks in Spring apps?
   A: Use heap dumps (jmap/jcmd), analyze with Eclipse MAT, look for retained sizes, static collections, ThreadLocal leaks, non-daemon threads, large caches without eviction, and unclosed resources.

10) Q: Spring Security: method-level vs web-level security and OAuth2 resource-server basics.
    A: Method-level uses @PreAuthorize/@PostAuthorize and secures service layer. Web-level uses HttpSecurity config for URL-based rules. OAuth2 resource-server validates JWTs or introspects tokens; configure JwtDecoder and map claims to authorities.

Databases — Additions (short answers)
- Normalization vs Denormalization: normalize for write consistency; denormalize for read performance (use caching/ETL).
- Partitioning strategies: range, list, hash — choose based on query patterns and maintenance needs.
- Connection pool tuning: size based on CPU cores and average request latency; avoid too many connections causing DB contention.
- Backup & Restore strategies: point-in-time recovery (WAL/perf), logical backups for portability, physical for speed — test restores regularly.
- Multi-Region replication: manage latency, conflict resolution (last write wins vs CRDTs), and read locality.

System Architecture & Microservices — Questions & Answers
1) Q: How do you decide between a monolith and microservices?
   A: Consider team size/organizational boundaries, deployment independence needs, complexity overhead (distributed systems, latency, operational cost). Start with a modular monolith and split when teams/scale/independent deployment or scaling needs justify microservices.

2) Q: What are the key non-functional concerns when designing microservices?
   A: Observability (metrics, logs, traces), resilience (circuit breakers, retries, timeouts), security (authn/authz, mTLS), scalability (autoscaling, stateless services), data consistency, and operability (CI/CD, deployment strategy, rollback).

3) Q: How to handle data consistency in microservices?
   A: Prefer eventual consistency with sagas, compensate on failures, use idempotency keys, careful idempotent design for retries, design bounded contexts to minimize cross-service transactions. For strict consistency, consider co-locating data or using distributed transactions selectively.

4) Q: Describe API versioning strategies and trade-offs.
   A: URI versioning (/v1/...), header-based, content negotiation (Accept header), and semantic versioning for contracts. URI versioning is simple but can proliferate endpoints; header-based is cleaner but harder for clients and caching. Prefer backwards-compatible additive changes and deprecate old versions with clear lifecycle.

5) Q: How to design a scalable event-driven system?
   A: Use durable message brokers (Kafka, Pulsar) with partitioning for parallelism, design idempotent consumers, use consumer groups for scaling, manage topic retention and compaction for state, design event schemas (Avro/Protobuf) with schema registry, and ensure observability of event flows.

6) Q: What are the pros/cons of synchronous vs asynchronous communication between services?
   A: Synchronous (HTTP/gRPC): simpler, easier for immediate responses, but tight coupling and cascading failures. Asynchronous (messaging/events): decouples, more resilient and scalable, enables event sourcing, but adds complexity (eventual consistency, debugging).

7) Q: How to perform canary releases and blue-green deployments for services?
   A: Canary: route a subset of traffic to new version using load balancer, monitor metrics and roll forward/rollback based on SLOs. Blue-green: deploy new version to idle environment, switch traffic atomically, and retain previous for quick rollback. Automate with CI/CD and health checks.

8) Q: Service discovery patterns (client-side vs server-side) and trade-offs.
   A: Client-side (Eureka/Consul): client queries registry and load-balances; reduces central LB cost but adds client complexity. Server-side (API Gateway/load balancer): centralizes routing and simplifies clients but can be a single point of failure unless highly available.

9) Q: How to handle cross-cutting concerns (logging, tracing, auth) in microservices?
   A: Use shared libraries and middleware (filters/interceptors), centralized logging (ELK), distributed tracing (OpenTelemetry, Jaeger), and centralized auth (OIDC provider) with token propagation. Keep concerns orthogonal with consistent headers (trace-id) and standards.

10) Q: How to design for failure — chaos engineering practices?
    A: Inject failures (latency, instance termination, network partitions), run experiments in controlled environments, define blast radius, monitor SLOs and runbooks, and automate recovery (self-healing, restarts).

Practical system-design question (short)
Q: Design a scalable URL shortener — key components and data model?
A: API service for create/redirect, hashing/ID generation (base62), data store (NoSQL or RDBMS) with key→URL mapping, caching (Redis) for hot redirects, analytics pipeline (events to Kafka → batch aggregation), partitioned DB for scale, consistent hashing for URL lookup, handle collisions with retries, rate limiting, and TTL management. Ensure idempotency and safe deletion.

Interview Tips — How to answer effectively
- Clarify requirements, constraints, SLOs, and expected load before proposing designs.
- Discuss trade-offs explicitly (complexity vs performance, consistency vs availability).
- Sketch high-level architecture first, then dive into components (data flow, failure modes).
- Mention observability, security, testing, deployment, and rollback strategies.
- Use examples from past experience and quantify decisions when possible.

Quick Reference: One-page checklist for live interview
- Ask clarifying questions first.
- State assumptions & constraints.
- Present high-level approach, then detail components.
- Discuss data flow, scaling, failure handling.
- Summarize trade-offs & next steps.
- Suggest follow-ups (benchmarks, POCs).

If you want, I can:
- Generate a printable one-page PDF layout of this cheat-sheet.
- Produce a 30–45 minute mock interview script with timed questions and suggested follow-ups.
- Create flashcards (Q/A pairs) exportable as CSV for study apps.

Which of these would you like next?