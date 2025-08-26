AI Assistant

Below are curated interview questions and model answers across Core Java, Design Patterns, Spring Boot, and Databases. They range from intermediate to advanced and focus on topics a Technical Lead should know (architecture, trade-offs, performance, concurrency, design decisions). Use these for study, whiteboard practice, or to craft follow-ups in interviews.

Core Java (Intermediate → Advanced)
1) Q: Explain Java memory model (JMM) basics and the happens-before relationship.
   A: JMM defines how threads interact through memory and guarantees visibility and ordering. The happens-before relationship is a partial order: if action A happens-before B, then effects of A are visible to B. Examples: program order (within a thread), monitor unlock → lock, volatile write → volatile read, Thread.start() happens-before first action in started thread, Thread.join() happens-after terminated thread. Without proper happens-before, you can get stale reads and race conditions.

2) Q: When would you use volatile vs synchronized vs Atomic* classes?
   A: volatile ensures visibility and prevents reordering for a single variable but does not provide atomic compound actions. Use volatile for simple flags or state where read/write is atomic and no compound update required. synchronized provides atomicity and mutual exclusion and also visibility; use it for critical sections spanning multiple variables or compound operations. java.util.concurrent.atomic (AtomicInteger, AtomicReference, etc.) provide lock-free atomic operations and CAS; use for high-performance concurrent counters, non-blocking algorithms.

3) Q: Describe how ConcurrentHashMap works (Java 8+) and its differences from HashMap and Hashtable.
   A: ConcurrentHashMap in Java 8 uses a lock-free strategy for reads and a combination of CAS and fine-grained locking for updates. It stores entries in nodes and uses ConcurrentHashMap.TreeBin when bins get large (treeified). Unlike HashMap (not thread-safe) and Hashtable (single lock for entire map), ConcurrentHashMap allows concurrent reads and updates with much higher throughput. It also doesn’t allow null keys/values and uses internal control to avoid full-map locking for most operations.

4) Q: Explain Java's class loading process and how you'd implement a plugin architecture safely.
   A: Class loading involves bootstrap, platform (or extension), and application class loaders (and custom loaders). Each loader delegates to parent by default. For a plugin system, use a separate child ClassLoader for each plugin to isolate classes and permit unloading (by dropping references), and carefully manage service interfaces loaded by a common parent to avoid duplicate types. Be mindful of security (sandboxing via SecurityManager/Policy where applicable) and resource leaks (threads, static holders).

5) Q: How does garbage collection pause time relate to throughput and memory footprint? How would you tune GC for a low-latency service?
   A: GC pause time trades off with throughput and memory usage. Generational collectors optimize for short-lived objects. For low-latency services, choose a low-pause collector (ZGC, Shenandoah, or G1 with low-pause settings), set heap size, tuning pause targets (G1MaxPauseMillis), reduce allocation rates, and avoid full GCs by ensuring sufficient headroom and minimizing large object allocations. Profile to identify survivors and adjust new generation size.

6) Q: What are strong, soft, weak, and phantom references? Give practical use-cases.
   A: Strong: normal references preventing GC. Soft: collected only when memory low — useful for caches that can be recycled. Weak: collected when no strong refs — good for canonicalizing mappings (WeakHashMap). Phantom: enqueued after finalization but before memory reclaim — used with ReferenceQueue to manage post-mortem cleanup (native resources).

7) Q: Describe how CompletableFuture works and how to avoid thread-pool starvation with async pipelines.
   A: CompletableFuture represents an async computation with methods to compose (thenApply, thenCompose), combine, and handle exceptions. By default, supplyAsync and runAsync use ForkJoinPool.commonPool; chained async methods without an explicit Executor may run on calling thread or common pool depending on method variants. To avoid starvation, provide a bounded, properly sized Executor for blocking tasks, avoid long blocking in pool threads, and separate CPU-bound and IO-bound thread pools.

8) Q: Explain method handles and invokedynamic (JVM) — why were they introduced?
   A: MethodHandles and invokedynamic were added to support dynamic languages on the JVM and to enable more flexible, efficient linkage at runtime. MethodHandles provide typed, directly callable references to methods/fields with lower overhead than reflection. invokedynamic defers linkage until runtime, allowing dynamic language runtimes and frameworks to implement call-site optimizations (bootstrap methods) and improve performance.

Design Patterns (Intermediate → Advanced)
1) Q: Describe the Gang of Four Factory Method vs Abstract Factory and when to use each.
   A: Factory Method provides an interface for creating objects but lets subclasses decide which class to instantiate — useful when a class defers instantiation to subclasses for single product families. Abstract Factory provides an interface to create related families of objects (multiple product types) without specifying concrete classes — use when you need consistent sets of products (e.g., UI widgets for different platforms). Factory Method is simpler for single-product creation; Abstract Factory groups factories.

2) Q: Explain Dependency Injection (DI) and how it relates to Inversion of Control (IoC). What are pitfalls of excessive DI?
   A: DI is a technique where dependencies are provided from outside (constructor/setter/field) rather than created internally. IoC is a broader principle where control flow is inverted — the framework controls object lifecycle and wiring. Pitfalls of excessive DI: overly granular components, confusing wiring, ambiguous ownership, harder to trace runtime behavior, and test fragility if too many mocks are used. Balance by grouping responsibilities and preferring constructor injection for required dependencies.

3) Q: When would you use Strategy vs Template Method? Provide examples.
   A: Strategy encapsulates interchangeable algorithms as separate classes and selects them at runtime — use when you want swapable behavior (e.g., different pricing algorithms). Template Method defines skeleton of an algorithm in a base class while allowing subclasses to override steps — use when you have a fixed process with some steps varying among implementations (e.g., document generation lifecycle). Strategy favors composition; Template Method uses inheritance.

4) Q: How does the Proxy pattern differ from Decorator? When to use each?
   A: Both wrap another object. Proxy controls access (remote proxy, virtual proxy/lazy initialization, protection), often presenting same interface and adding access control or remote invocation. Decorator adds responsibilities dynamically to an object (e.g., adding behavior). Use Proxy for control and transparent remote/local substitution; Decorator to add features without modifying original class.

5) Q: Explain CQRS and Event Sourcing — architecture trade-offs.
   A: CQRS separates read and write models: commands mutate state, queries read optimized views. Event Sourcing persists state as sequence of events rather than mutable state. Benefits: auditability, temporal queries, scalable read models. Trade-offs: complexity (projections, eventual consistency), event schema evolution challenges, harder debugging and testing, increased operational overhead. Use when auditability, complex domains, or high read/write scalability justify cost.

6) Q: What is the Circuit Breaker pattern and key design considerations?
   A: Circuit Breaker prevents cascading failures by wrapping remote calls; it monitors failures and opens the circuit when failure threshold is exceeded, short-circuiting calls and optionally returning fallback. Considerations: failure thresholds, timeout windows, half-open trial behavior, integration with metrics/alerts, choosing fallback strategies, avoiding stateful single points, and ensuring thread-safety and latency bounds.

7) Q: Describe Hexagonal (Ports & Adapters) architecture and why it's useful for a Technical Lead to enforce.
   A: Hexagonal isolates core/domain logic from external concerns via ports (interfaces) and adapters (implementation details like DB, UI, messaging). Benefits: testability, independent evolution of infrastructure, clear boundaries, easier to swap technologies, and promotes clean domain model. For a Lead, it helps maintain long-term maintainability and team parallelism.

8) Q: How would you design a plugin system allowing third-party code but protecting main app from misuse?
   A: Use plugin isolation via separate ClassLoaders, restrict privileged APIs, run plugins in sandboxed processes or containers if stronger isolation is needed, define explicit plugin API contracts (interfaces), validate and sign plugins, limit resource use (timeouts, quotas), provide lifecycle hooks, and monitor plugin behavior. Favor least-privilege and runtime governance.

Spring Boot (Intermediate → Advanced)
1) Q: How does Spring Boot auto-configuration work? How can you override or disable an auto-config?
   A: Auto-configuration uses @EnableAutoConfiguration / @SpringBootApplication and Conditional annotations (@ConditionalOnClass, @ConditionalOnProperty, etc.) to create beans when certain conditions apply. To override, provide your own @Bean with same type/name that takes precedence, set properties to disable specific autoconfigurations, or use @SpringBootApplication(exclude = {...}) or spring.autoconfigure.exclude in application.properties to disable a particular auto-configuration class.

2) Q: Explain how Spring handles transactions. What are propagation behaviors and isolation levels?
   A: Spring transaction management uses PlatformTransactionManager implementations (e.g., DataSourceTransactionManager, JpaTransactionManager). @Transactional creates proxies (or uses AspectJ) to start/commit/rollback transactions. Propagation controls how transactional methods join/launch transactions (REQUIRED, REQUIRES_NEW, NESTED, SUPPORTS, etc.). Isolation levels control DB concurrency (DEFAULT, READ_COMMITTED, REPEATABLE_READ, SERIALIZABLE). Proper propagation and exception handling (unchecked vs checked) determine rollbacks.

3) Q: How do you implement a large-batch data import with Spring Boot and JPA without OOM?
   A: Use streaming and batching: disable first-level cache by clearing EntityManager periodically (em.flush(); em.clear()), use batch inserts via persistence properties (hibernate.jdbc.batch_size), use StatelessSession for pure inserts, stream rows from source, use pagination with read-only queries, set fetch size on JDBC, and increase JDBC batch size. Also tune transaction boundaries to commit periodically, and avoid holding large in-memory collections.

4) Q: Describe Reactor/Reactive support in Spring (WebFlux) vs synchronous Spring MVC. When to use which?
   A: WebFlux is non-blocking, uses Project Reactor (Mono/Flux), and is suited for high concurrency with non-blocking IO (e.g., many long-polling or streaming clients). Spring MVC is thread-per-request blocking model—simple and fits many transactional, blocking IO scenarios (DB with JDBC). Use WebFlux when end-to-end stack supports non-blocking (reactive DB drivers), otherwise stick with MVC for simpler development or when using blocking resources.

5) Q: How can you secure microservices with Spring Boot in a distributed environment?
   A: Use Spring Security with OAuth2/OIDC for authentication and JWT or opaque tokens for stateless auth. Delegate auth to centralized identity provider (Keycloak, Auth0). Use token propagation, service-to-service auth (mutual TLS or mTLS, or signed JWTs), implement API Gateway for central auth and rate limiting, and enforce role/permission checks at resource/service boundary. Combine with request tracing and centralized logging/monitoring.

6) Q: Explain conditional bean creation and how custom conditions work.
   A: Conditional creation uses @Conditional and built-in conditions like @ConditionalOnMissingBean, @ConditionalOnClass, @ConditionalOnProperty. For custom conditions, implement Condition and use @Conditional(MyCondition.class). The matches() method receives context and metadata to decide whether to create bean or configuration.

7) Q: How do you implement graceful shutdown in Spring Boot?
   A: From Spring Boot 2.3+, enable server.shutdown=graceful and configure spring.lifecycle.timeout-per-shutdown-phase. For blocking tasks, implement DisposableBean or @PreDestroy to complete in-flight work and prevent new requests. For messaging, stop consuming new messages, wait for ack/processing completion, and coordinate via readiness/liveness probes in k8s.

8) Q: How do you profile and troubleshoot a slow Spring Boot startup?
   A: Use -Dspring.devtools.restart.enabled=false to avoid restart issues, enable startup actuator (spring-boot-actuator / startup tracing), use Java Flight Recorder or async-profiler to capture hotspots, inspect auto-configurations (spring.autoconfigure.exclude), enable debug logging for org.springframework to see context loading, and check classpath scanning, large @ComponentScan scopes, and heavy @PostConstruct initializations.

Databases (Intermediate → Advanced)
1) Q: Explain ACID properties and how isolation levels (READ UNCOMMITTED → SERIALIZABLE) map to phenomena (dirty read, non-repeatable read, phantom read).
   A: ACID: Atomicity, Consistency, Isolation, Durability. Isolation levels:
    - READ UNCOMMITTED: allows dirty reads.
    - READ COMMITTED: prevents dirty reads, but allows non-repeatable reads and phantoms.
    - REPEATABLE READ: prevents dirty and non-repeatable reads but may allow phantoms (depends on DB; InnoDB uses next-key locks to prevent phantoms).
    - SERIALIZABLE: highest level, ensures transactions appear serial, preventing all three phenomena at cost of concurrency.

2) Q: How does MVCC work and how does it affect long-running transactions?
   A: MVCC stores multiple versions of rows so readers see a consistent snapshot without blocking writers. Long-running transactions retain old snapshots and prevent cleanup (vacuum/garbage collection), increasing storage and causing write amplification or query overhead. In databases like PostgreSQL, long transactions can bloat tables and delay VACUUM.

3) Q: Explain optimistic vs pessimistic locking and appropriate use-cases.
   A: Pessimistic locking acquires DB locks (SELECT ... FOR UPDATE), preventing concurrent modifications — use for high-conflict cases or when business correctness demands strict locking. Optimistic locking uses version/timestamp checks (e.g., @Version in JPA) and fails transaction on conflict — use in low-contention and high-scale scenarios for better throughput.

4) Q: How do you design indexes for a large OLTP table with mixed reads/writes?
   A: Index selectively: index columns used in WHERE, JOIN, ORDER BY; prefer composite indexes ordered to match query predicates (leftmost prefix rule). Avoid excess indexes because they slow writes. Use covering indexes for frequent queries to avoid lookups. Monitor with EXPLAIN plans and use partial or filtered indexes for sparse predicates. Consider partitioning for very large tables and maintain statistics.

5) Q: What is database sharding and how would you choose a sharding key?
   A: Sharding splits data horizontally across nodes. Choose a sharding key that evenly distributes load and avoids cross-shard joins/transactions; it should align with common access patterns (e.g., user_id if most operations are per-user). Avoid monotonically increasing keys (hotspots). Consider re-sharding strategy, data locality, backup, and distributed transactions complexity.

6) Q: When and how to use materialized views vs caching (Redis)?
   A: Materialized views are stored query results maintained by DB (good for consistency and complex aggregations with DB-managed refresh). Use when you need ACID consistency with DB data and queries are expensive. Cache (Redis) is better for ultra-low-latency reads, cross-service caching, and when eventual consistency is acceptable. Consider invalidation strategies, TTLs, and cache warming.

7) Q: Describe how to diagnose a slow query in production.
   A: Capture query plan (EXPLAIN ANALYZE), check indexes and statistics, examine I/O and CPU usage, check locks and wait events, review parameter sniffing/plan caching issues, check table/partition bloat, and analyze execution frequency. Reproduce with representative data, test indexing, rewrite query (e.g., avoid functions on indexed columns), or add covering index/partitioning as needed.

8) Q: How would you design transactions across multiple microservices that each have their own database?
   A: Prefer sagas (choreography or orchestration) implementing compensating transactions for eventual consistency rather than distributed two-phase commit (2PC) because 2PC is complex and brittle at scale. Use idempotent operations, correlation IDs, retries with backoff, and observability. For stronger consistency in some cases, consider distributed transactions only when necessary and supported.

Practice / Tips for Technical Lead interviews
- Focus on trade-offs: always explain alternatives, costs, and why you’d choose one approach.
- Bring architecture-level concerns: monitoring, observability, deployment, rollbacks, and team practices.
- For coding or design exercises, clarify constraints and requirements before diving in.
- Emphasize mentorship: how you'd pair with junior engineers, code review approach, and design docs.

If you want, I can:
- Generate a mock interview script (30–45 minutes) with follow-ups.
- Provide whiteboard-style explanations for any question.
- Produce a one-page cheat-sheet you can print.

Which of those would you like next?