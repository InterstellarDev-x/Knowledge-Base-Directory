# Low-Level Design Patterns — A First-Principles Curriculum

> **How to use this guide:** Each stage builds on the last. Don't skip ahead — later patterns only make sense once earlier ones are internalized. Every stage has a core question, a reason it matters, what you must understand before moving on, and questions worth sitting with.

---

## Learning Path

```
Stage 1: Design Principles (SOLID, GRASP, KISS/DRY/YAGNI)
    ↓
Stage 2: Gang of Four — Creational Patterns
    ↓
Stage 3: Gang of Four — Structural Patterns
    ↓
Stage 4: Gang of Four — Behavioral Patterns
    ↓
Stage 5: Concurrency Patterns
    ↓
Stage 6: Enterprise & Domain Patterns (Fowler's PoEAA)
    ↓
Stage 7: Modern Resilience & Cloud Patterns
    ↓
Stage 8: Functional & Reactive Patterns
    ↓
Stage 9: API Design Patterns
```

---

## Concept Dependency Map

```
SOLID Principles
├── SRP → Single Responsibility → guides all class design
├── OCP → Open/Closed → basis for Strategy, Decorator, Template Method
├── LSP → Liskov Substitution → basis for correct inheritance
├── ISP → Interface Segregation → basis for correct abstraction
└── DIP → Dependency Inversion → basis for IoC, DI containers

GoF Creational
├── Singleton → resource management, service locators
├── Factory Method → object creation abstraction
├── Abstract Factory → families of objects
├── Builder → complex object construction
└── Prototype → cloning, deep copy

GoF Structural
├── Adapter → interface mismatch bridging
├── Bridge → abstraction/implementation separation
├── Composite → tree structures, hierarchies
├── Decorator → behavior addition without subclassing
├── Facade → subsystem simplification
├── Flyweight → memory optimization
└── Proxy → access control, lazy loading, caching

GoF Behavioral
├── Chain of Responsibility → request pipelines
├── Command → request encapsulation, undo/redo
├── Iterator → collection traversal
├── Mediator → decoupled communication
├── Memento → state snapshots
├── Observer → event notification (→ Reactive Streams)
├── State → FSM-based behavior
├── Strategy → algorithm selection
├── Template Method → algorithm skeleton
└── Visitor → operations on object structures

Concurrency
├── Monitor Object → synchronized state access
├── Active Object → async method invocation
├── Half-Sync/Half-Async → thread pool + async I/O
├── Reactor / Proactor → event demultiplexing
└── Read-Write Lock, Producer-Consumer, Thread Pool

PoEAA (Domain / Data Patterns)
├── Transaction Script → simple procedural logic
├── Domain Model → rich OO domain
├── Table Module → schema-centric
├── Data Mapper → ORM, decoupled persistence
├── Active Record → simple persistence
├── Repository → collection-like persistence
├── Unit of Work → change tracking
├── Identity Map → object identity guarantee
└── Service Layer → application logic boundary

Modern Resilience
├── Circuit Breaker → failure isolation
├── Bulkhead → failure containment
├── Retry + Backoff → transient failure handling
├── Saga → distributed transactions
├── CQRS → read/write separation
├── Event Sourcing → append-only audit log
├── Outbox Pattern → atomic dual-write
├── Strangler Fig → incremental migration
└── Anti-Corruption Layer → context translation

Functional
├── Functor → mappable containers
├── Monad → sequential computations, effect handling
├── Option/Maybe, Either → null/error as values
├── Pipeline → function composition
└── Immutability → value semantics

Reactive
├── Observer → push notifications (GoF)
├── Reactive Streams → backpressure, async sequences
├── Event-Driven Architecture → decoupled producers/consumers
└── Backpressure → consumer-driven flow control
```

---

## Stage 1 — Design Principles: Why Code Rots

**Core question:** What makes code hard to change, and how do we design against that?

**Why this stage first:** Every pattern in the GoF catalog is an application of one or more of these principles. Learn the principle, and you can derive the pattern. Learn only the pattern, and you'll misapply it.

### SOLID

| Principle | Source | What it says |
|---|---|---|
| **S** — Single Responsibility | Robert C. Martin, *Agile Software Development* (2002) | A class should have exactly one reason to change |
| **O** — Open/Closed | Bertrand Meyer, *Object-Oriented Software Construction* (1988); restated by Martin | Open for extension, closed for modification |
| **L** — Liskov Substitution | Barbara Liskov, "Data Abstraction and Hierarchy" (OOPSLA 1987) | Subtypes must be substitutable for their base types without altering program correctness |
| **I** — Interface Segregation | Robert C. Martin, *Design Principles and Design Patterns* (2000) | No client should be forced to depend on methods it does not use |
| **D** — Dependency Inversion | Robert C. Martin, same paper | High-level modules should not depend on low-level modules; both should depend on abstractions |

> The SOLID acronym was assembled by Michael Feathers around 2004, drawing from Martin's earlier writing.

### Other Foundational Principles

| Principle | Meaning |
|---|---|
| DRY — Don't Repeat Yourself | Every piece of knowledge should have one authoritative representation in the system. Hunt & Thomas, *The Pragmatic Programmer* (1999) |
| YAGNI — You Aren't Gonna Need It | Don't build features until they're needed. Kent Beck, Extreme Programming. |
| KISS — Keep It Simple, Stupid | Prefer the simplest solution that works. |
| Law of Demeter (LoD) | A method should only call methods of: itself, its parameters, objects it creates, its direct components. Lieberherr et al., 1987. |
| Composition over Inheritance | GoF's first foundational principle. Prefer has-a over is-a for flexible designs. |
| Program to Interfaces | GoF's second foundational principle. Depend on abstractions, not concretions (DIP expressed differently). |

### GRASP Patterns (Larman)

| Pattern | Assigns responsibility to... |
|---|---|
| Information Expert | The class that has the information needed to fulfill the responsibility |
| Creator | The class that aggregates, contains, or records instances of the created class |
| Controller | A non-UI class that handles system events (Facade or Use Case Controller) |
| Low Coupling | Minimize dependencies between classes |
| High Cohesion | A class should do one focused thing |
| Polymorphism | Assign responsibility to the type that has the variation |
| Pure Fabrication | A class created purely to achieve low coupling / high cohesion, with no domain counterpart |
| Indirection | Route through a mediating object to avoid direct coupling |
| Protected Variations | Wrap instability behind a stable interface |

**Source:** Craig Larman, *Applying UML and Patterns* (3rd ed., 2004)

### What to understand before moving on
- Why does violating SRP cause cascade changes across the codebase?
- What is the difference between DIP (dependency direction) and DI containers (mechanism)?
- Why does Liskov matter — what breaks when a subtype changes behavior in unexpected ways?
- When is inheritance appropriate vs. composition?

### Questions to sit with
- Can you have SRP without well-named abstractions? How do you find the right granularity?
- YAGNI and OCP seem to be in tension — OCP says design for extension, YAGNI says don't anticipate. How do you reconcile this?

---

## Stage 2 — GoF Creational Patterns: Object Construction

**Core question:** Who is responsible for creating objects, and how do we decouple clients from the types they instantiate?

**Canonical source:** Erich Gamma, Richard Helm, Ralph Johnson, John Vlissides — *Design Patterns: Elements of Reusable Object-Oriented Software* (Addison-Wesley, 1994). 23 patterns total: 5 creational, 7 structural, 11 behavioral.

| Pattern | Intent | When to use |
|---|---|---|
| **Singleton** | Ensure a class has only one instance and provide a global access point | Shared resources (thread pool, config, registry); be cautious — global state is a design smell |
| **Factory Method** | Define an interface for creating an object, but let subclasses decide which class to instantiate | Frameworks that need to create objects of types they don't know at compile time |
| **Abstract Factory** | Provide an interface for creating families of related objects without specifying concrete classes | UI toolkits (Windows vs. Mac widgets), database drivers |
| **Builder** | Separate construction of a complex object from its representation | Objects with many optional parameters; step-by-step assembly; e.g. `StringBuilder`, query builders |
| **Prototype** | Specify kinds of objects to create using a prototypical instance; clone it | Expensive construction; object pools; avoiding subclassing just for initialization |

### What to understand before moving on
- Factory Method vs. Abstract Factory: FM is about one product, AF is about product families.
- When does Singleton become a liability? (Testing, concurrency, hidden coupling.)
- Why does Builder use a fluent interface? What invariant does it enforce?

### Questions to sit with
- Is a static factory method the same as the Factory Method pattern? (No — understand why.)
- What is the double-checked locking problem in Singleton in Java pre-Java 5?

---

## Stage 3 — GoF Structural Patterns: Composing Objects

**Core question:** How do we compose existing classes and objects into larger structures without tight coupling?

| Pattern | Intent | Classic example |
|---|---|---|
| **Adapter** | Convert the interface of a class into another interface clients expect | Legacy API wrapping; `java.io.InputStreamReader` adapts InputStream to Reader |
| **Bridge** | Decouple an abstraction from its implementation so the two can vary independently | GUI abstraction (Shape) over rendering implementations (SVG, Canvas) |
| **Composite** | Compose objects into tree structures to represent part-whole hierarchies | File system (File and Directory both implement Component); UI widget trees |
| **Decorator** | Attach additional responsibilities to an object dynamically | `java.io` streams (BufferedInputStream wrapping FileInputStream); middleware chains |
| **Facade** | Provide a simplified interface to a complex subsystem | Compiler subsystem facade; REST API over a complex domain model |
| **Flyweight** | Use sharing to support large numbers of fine-grained objects efficiently | Character rendering in text editors; game sprites; string interning |
| **Proxy** | Provide a surrogate or placeholder for another object to control access | Virtual proxy (lazy loading), protection proxy (access control), remote proxy (RPC stub), cache proxy |

### What to understand before moving on
- Adapter vs. Facade: Adapter works with an existing interface, Facade defines a new simpler one.
- Decorator vs. Inheritance: Decorator adds behavior at runtime, inheritance at compile time. What are the tradeoffs?
- Bridge vs. Adapter: Bridge is designed upfront to separate concerns; Adapter is retrofitted.
- Proxy vs. Decorator: Both wrap an object, but Proxy controls access, Decorator adds behavior.

### Questions to sit with
- Why is Composite considered structural if it's fundamentally recursive? What does that recursion buy you?
- When does Flyweight introduce correctness bugs (shared vs. extrinsic state)?

---

## Stage 4 — GoF Behavioral Patterns: Object Collaboration

**Core question:** How do we distribute responsibilities between objects and define communication protocols between them?

| Pattern | Intent | Classic example |
|---|---|---|
| **Chain of Responsibility** | Pass a request along a chain of handlers until one handles it | Middleware pipelines (Express.js), logging handlers, exception handling |
| **Command** | Encapsulate a request as an object, allowing undo/redo, queuing, logging | Text editor undo; transaction logs; job queues |
| **Interpreter** | Define a grammar and an interpreter for sentences in that language | SQL parsers, regex engines, DSLs |
| **Iterator** | Provide a way to sequentially access elements of a collection | `for-each` loops; Java `Iterator`; Python `__iter__` |
| **Mediator** | Define an object that encapsulates how a set of objects interact | Chat rooms; air traffic control; UI form validation logic |
| **Memento** | Capture and externalize an object's internal state so it can be restored | Undo in editors; game save states; database transaction snapshots |
| **Observer** | Define a one-to-many dependency so when one object changes state, dependents are notified | Event listeners; MVC (Model notifies View); pub/sub |
| **State** | Allow an object to alter its behavior when its internal state changes | Order lifecycle (Pending → Processing → Shipped → Delivered); TCP connection states |
| **Strategy** | Define a family of algorithms, encapsulate each one, make them interchangeable | Sorting algorithms; payment processors; compression codecs |
| **Template Method** | Define the skeleton of an algorithm in a base class, letting subclasses override steps | Frameworks that call user-defined hooks; data processing pipelines |
| **Visitor** | Represent an operation to be performed on elements of an object structure without changing their classes | Compilers (AST node operations); serialization; pretty-printers |

### What to understand before moving on
- Observer is the foundation of every event-driven system and reactive library. Master it.
- Strategy vs. Template Method: Strategy uses composition (inject the algorithm), Template Method uses inheritance (override the step).
- Command vs. Chain of Responsibility: Command encapsulates intent; CoR routes the command.
- State vs. Strategy: Both use polymorphism, but State transitions itself internally, Strategy is set externally.

### Questions to sit with
- Why is Visitor controversial? (It violates OCP — adding a new element type breaks all visitors.)
- What is the "Hollywood Principle" ("don't call us, we'll call you") and which patterns embody it?

---

## Stage 5 — Concurrency Patterns

**Core question:** How do we safely share state and coordinate between concurrent threads or processes?

**Canonical source:** Douglas Schmidt et al., *Pattern-Oriented Software Architecture, Vol. 2: Patterns for Concurrent and Networked Objects* (POSA2, Wiley, 2000)

| Pattern | Intent |
|---|---|
| **Monitor Object** | Synchronize concurrent method execution to ensure only one method runs within an object at a time; exposes condition variables for waiting |
| **Active Object** | Decouple method execution from method invocation for objects that reside in their own thread of control; each method call becomes a Message placed on a queue |
| **Half-Sync / Half-Async** | Separate synchronous and asynchronous processing into two intercommunicating layers; synchronous layer queues work for async workers |
| **Thread Pool** | Manage a pool of worker threads; submit tasks to the pool instead of creating threads per task; controls resource consumption |
| **Producer-Consumer** | Decouple producers of data from consumers via a shared bounded buffer; producers block when full, consumers block when empty |
| **Read-Write Lock** | Allow concurrent reads but exclusive writes; readers don't block each other, writers block everyone |
| **Reactor** | Handles service requests delivered concurrently by dispatching them synchronously to associated handlers; event loop |
| **Proactor** | Demultiplexes and dispatches service requests triggered by the completion of asynchronous operations; completion-based (vs. Reactor's readiness-based) |
| **Scheduler** | Control the order in which threads access shared resources based on a defined policy |
| **Thread-Specific Storage** | Allow multiple threads to use one "logically global" access point without sharing data — each thread has its own copy |

### Key distinctions
- **Reactor vs. Proactor:** Reactor is notified when a handle is ready for I/O (you do the I/O); Proactor is notified when the I/O is complete (the OS did the I/O). Node.js and `select()/epoll` are Reactor. Windows IOCP is Proactor.
- **Active Object vs. Thread Pool:** Active Object is per-object (each active object has its own thread/queue); Thread Pool is shared across all tasks.
- **Monitor vs. Mutex:** Monitor bundles the mutex with the condition variable and the object. Java's `synchronized` blocks are Monitors.

### Additional reading
- Brian Goetz et al., *Java Concurrency in Practice* (2006) — the concurrency patterns bible for JVM
- Maurice Herlihy & Nir Shavit, *The Art of Multiprocessor Programming* (2008) — lock-free and wait-free algorithms

### What to understand before moving on
- What is the ABA problem in lock-free algorithms?
- Why is double-checked locking broken without memory barriers?
- What does the Java Memory Model guarantee about visibility?

---

## Stage 6 — Enterprise & Domain Patterns (Fowler's PoEAA)

**Core question:** How do we organize business logic and persist domain state without coupling them together?

**Canonical source:** Martin Fowler, *Patterns of Enterprise Application Architecture* (PoEAA, Addison-Wesley, 2002)

### Domain Logic Patterns

| Pattern | Intent | When to use |
|---|---|---|
| **Transaction Script** | Organize business logic as a single procedure that handles one transaction | Simple CRUD; scripting; when the domain is thin |
| **Domain Model** | An object model of the domain that incorporates both behavior and data | Complex business logic; when behavior belongs with data |
| **Table Module** | A single instance that handles business logic for all rows in a database table | When the domain maps closely to database tables; middle ground between TS and DM |
| **Service Layer** | Defines application's boundary; establishes a set of available operations; coordinates responses | Separates application logic from domain logic; entry point for remote clients |

### Data Source Patterns

| Pattern | Intent | When to use |
|---|---|---|
| **Active Record** | An object that wraps a row in a database table, encapsulates database access, and adds domain logic | Simple domains; 1:1 row-to-object mapping; Rails `ActiveRecord`, Django models |
| **Data Mapper** | A layer of mappers that moves data between objects and the database while keeping them independent of each other | Complex domains where object model and schema diverge; ORMs like Hibernate, SQLAlchemy |
| **Repository** | Mediates between the domain and data mapping layers using a collection-like interface for accessing domain objects | Domain-driven designs; hides query complexity; unit-testable by swapping to in-memory |
| **Table Data Gateway** | An object that acts as a Gateway to a database table — one instance per table | Simple access to single tables |
| **Row Data Gateway** | An object that acts as a Gateway to a single record — one instance per row | Simple per-record operations |

### Object-Relational Behavioral Patterns

| Pattern | Intent |
|---|---|
| **Unit of Work** | Maintains a list of objects affected by a business transaction and coordinates writing out changes and resolving concurrency problems |
| **Identity Map** | Ensures each object gets loaded only once by keeping every loaded object in a map; lookup object before loading |
| **Lazy Load** | An object that doesn't contain all of the data you need but knows how to get it on demand |

### Key relationships
- **Active Record vs. Data Mapper:** AR conflates domain object and persistence (simple but coupled). DM separates them (more complex, but domain is database-agnostic).
- **Repository vs. DAO:** Repository exposes a collection metaphor with domain semantics (`findActiveOrders()`); DAO is more mechanical and database-centric.
- **Unit of Work + Identity Map:** Almost always used together. UoW tracks changes; Identity Map prevents loading the same row twice.

### Also essential
- Eric Evans, *Domain-Driven Design* (2003) — Bounded Contexts, Aggregates, Entities, Value Objects, Domain Events, Repositories as a first-class concept
- Vaughn Vernon, *Implementing Domain-Driven Design* (2013) — practical DDD with Aggregates, CQRS, Event Sourcing

### What to understand before moving on
- Why is the Domain Model pattern sometimes called "Anemic Domain Model anti-pattern" when misused?
- What invariant does the Aggregate Root enforce in DDD?
- When does a Repository implementation leak query logic into the domain?

---

## Stage 7 — Modern Resilience & Cloud Patterns

**Core question:** How do we build systems that degrade gracefully, recover automatically, and evolve without rewriting?

**Canonical sources:**
- Nygard, *Release It!* (2nd ed., 2018) — Circuit Breaker, Bulkhead, Timeout patterns
- Microsoft Azure Architecture Center — [learn.microsoft.com/azure/architecture/patterns](https://learn.microsoft.com/en-us/azure/architecture/patterns/)
- Fowler, [martinfowler.com/bliki/CQRS.html](https://martinfowler.com/bliki/CQRS.html) and [martinfowler.com/eaaDev/EventSourcing.html](https://martinfowler.com/eaaDev/EventSourcing.html)

### Stability Patterns

| Pattern | Intent | Key mechanism |
|---|---|---|
| **Circuit Breaker** | Stop cascading failures by wrapping remote calls in a state machine (Closed → Open → Half-Open) | Fail fast when a downstream service is down; probe recovery before resuming traffic |
| **Bulkhead** | Isolate failures by partitioning resources (thread pools, connection pools) per downstream service | Failure in one partition doesn't exhaust resources for others; named after ship compartments |
| **Retry + Exponential Backoff** | Retry transient failures with increasing delays and jitter | Prevents thundering herd on recovery; jitter randomizes retry storms |
| **Timeout** | Set a maximum wait time on all remote calls; never wait indefinitely | Prevents thread starvation from slow upstreams |
| **Rate Limiter / Throttle** | Control the rate of incoming requests to protect a service | Token bucket, leaky bucket, sliding window counter algorithms |

> **Circuit Breaker + Retry:** These are complementary but distinct. Retry assumes transient failure → eventual success. Circuit Breaker assumes prolonged failure → stop trying. Combine them: let Retry handle brief blips; let Circuit Breaker trip when retries exhaust.

### Data/Event Patterns

| Pattern | Intent | Key insight |
|---|---|---|
| **CQRS** (Command Query Responsibility Segregation) | Separate the read model (Query) from the write model (Command); each can be optimized and scaled independently | Fowler: only appropriate for complex domains with asymmetric read/write load; adds significant complexity for simple CRUD |
| **Event Sourcing** | Store state as an ordered, immutable log of events; derive current state by replaying the log | Enables full audit history, temporal queries, and event-driven projections; hazard: replaying events against external systems sends duplicate side effects |
| **Outbox Pattern** | Atomically write a domain event to a local outbox table in the same transaction as the state change; a relay process reads and publishes to the message broker | Solves the dual-write problem: "update DB and publish event" without distributed transactions |
| **Saga** | Manage distributed transactions as a sequence of local transactions; if any step fails, run compensating transactions to undo prior steps | Two implementations: **choreography** (each service publishes events that trigger the next step — decentralized, no SPOF, but harder to trace) and **orchestration** (central saga orchestrator sends commands to each participant — easier to trace, central SPOF) |

### Structural / Migration Patterns

| Pattern | Intent |
|---|---|
| **Strangler Fig** | Incrementally replace a legacy system by routing new functionality through a façade; gradually strangle the old system until it can be decommissioned. Named after the strangler fig tree that envelops a host tree. |
| **Anti-Corruption Layer (ACL)** | Translate between two bounded contexts or systems with incompatible models; isolates your domain from a foreign domain's concepts. Evans, DDD (2003). |
| **Sidecar** | Attach a helper process/container alongside a main application container to provide peripheral capabilities (logging, proxy, config) without changing the main app |
| **Ambassador** | A sidecar that proxies outbound network calls; handles retry, circuit breaking, observability at the proxy layer |

### What to understand before moving on
- What is the "dual write" problem and why does the Outbox pattern solve it atomically?
- What is the difference between Saga choreography and orchestration — when does each break?
- Why does CQRS increase operational complexity? What does "eventual consistency" mean for the read model?
- Why does Event Sourcing make replaying against external systems dangerous?

### Questions to sit with
- CQRS + Event Sourcing are often mentioned together. Are they required together? (No. Understand why each is independently useful.)
- If Event Sourcing makes "now" derivable from the log, why do systems still maintain a materialized current state?

---

## Stage 8 — Functional & Reactive Patterns

**Core question:** How do we handle side effects, optional values, and asynchronous streams in a composable, type-safe way?

### Functional Patterns

These patterns come from category theory and functional programming. They've entered mainstream software through languages like Haskell, Scala, Rust, and through libraries like Java's `Optional`, Scala's `Try/Either`, Rust's `Result/Option`.

| Pattern | What it is | Example |
|---|---|---|
| **Functor** | Any type `F<A>` with a `map(f: A → B): F<B>` operation that preserves structure | `List.map`, `Optional.map`, `Future.map` |
| **Monad** | A Functor with `flatMap` (also called `bind` or `>>=`): `F<A>` → `(A → F<B>)` → `F<B>`; chains computations that produce wrapped values | `Optional.flatMap` (null propagation), `Future.flatMap` (async sequencing), `List.flatMap` (cartesian product) |
| **Option / Maybe** | Represents a value that may or may not be present; eliminates null pointer exceptions | Haskell `Maybe`, Java `Optional`, Rust `Option<T>`, Scala `Option` |
| **Either / Result** | Represents a computation that may fail; `Left` = error, `Right` = success; error is a first-class value | Haskell `Either`, Rust `Result<T, E>`, Scala `Either`, Go `(T, error)` idiom |
| **Try** | Like Either but specialized for exception-based error handling; captures exceptions as values | Scala `Try`, Vavr's `Try` in Java |
| **Reader Monad** | Represents a computation that reads from a shared environment (dependency injection as a value) | Config passing; thread-local-free dependency injection |
| **Writer Monad** | A computation that produces a log alongside its value | Accumulating diagnostics without side effects |
| **State Monad** | A computation that threads mutable state through as a value | Stateful computations in pure functional code |
| **Pipeline / Function Composition** | Chain pure functions `f: A→B`, `g: B→C` into `g∘f: A→C` | Unix pipes; Elixir `|>` operator; functional `andThen`/`compose` |
| **Immutability** | Values never change after creation; all updates produce new values | Eliminates aliasing bugs and concurrent modification; enables structural sharing (persistent data structures) |

**Recommended reading:**
- Bartosz Milewski, *Category Theory for Programmers* (free online, 2017) — mathematical grounding for Functor/Monad
- Michael Fogus & Chris Houser, *The Joy of Clojure* (2014) — immutability and persistent data structures
- Scott Wlaschin, [fsharpforfunandprofit.com](https://fsharpforfunandprofit.com) — Railway-Oriented Programming (practical Either/Result patterns)

### Reactive Patterns

| Pattern | Intent | Source |
|---|---|---|
| **Observer (GoF)** | Push-based notification from subject to subscribers | GoF (1994) — foundational |
| **Reactive Streams** | Asynchronous stream processing with non-blocking backpressure; defines Publisher/Subscriber/Subscription/Processor | Reactive Streams Specification (2013); JDK9 `java.util.concurrent.Flow` |
| **Backpressure** | Consumer signals its capacity to the producer; producer slows down or buffers to match; prevents consumer overflow | Reactive Streams; `request(n)` protocol |
| **Event-Driven Architecture (EDA)** | Components communicate only via events on a bus; producers and consumers are fully decoupled | Kleppmann, *Designing Data-Intensive Applications* Ch.11; enterprise patterns |
| **Publish-Subscribe** | Publishers emit events to topics; subscribers receive events from topics they subscribe to; broker decouples them | Apache Kafka, Google Pub/Sub, AWS SNS/SQS |

**Recommended reading:**
- Jonas Bonér et al., *The Reactive Manifesto* (2014) — Responsive, Resilient, Elastic, Message-Driven
- Erik Meijer, "Your Mouse is a Database" (2012) — unifying Iterable and Observable under duality
- Kleppmann, *Designing Data-Intensive Applications* (2017) — Chapters 11–12 on stream processing and event-driven systems

### What to understand before moving on
- What is the monad laws? (Left identity, right identity, associativity.) Why do they matter?
- What is the difference between a cold and hot Observable/Publisher?
- How does backpressure differ from buffering — and when does buffering become a liability?

---

## Stage 9 — API Design Patterns

**Core question:** How do we design the boundary between components, services, or systems so that it remains stable, evolvable, and easy to use correctly?

### REST Patterns

| Pattern | Intent |
|---|---|
| **Resource-Oriented Design** | Model APIs as resources (nouns) not operations (verbs); `GET /orders/123` not `POST /getOrder` |
| **HATEOAS** | Hypermedia As The Engine Of Application State; responses include links to valid next actions; clients navigate without out-of-band knowledge |
| **Pagination Patterns** | Offset (`?page=2&size=20`), cursor-based (`?after=<cursor>`), seek/keyset; cursor-based is stable under concurrent inserts |
| **Idempotency Keys** | Client provides a unique key per request; server deduplicates; enables safe retry of non-idempotent operations (POST payments) |
| **Versioning** | URL versioning (`/v1/`), header versioning, content negotiation; each has stability vs. flexibility tradeoffs |
| **ETags + Conditional Requests** | Server returns `ETag` with response; client sends `If-None-Match`; server returns 304 if unchanged; optimistic concurrency via `If-Match` |

**Canonical source:** Roy Fielding, "Architectural Styles and the Design of Network-based Software Architectures" (PhD dissertation, UC Irvine, 2000) — the REST dissertation. Also: Google Cloud API Design Guide (aip.dev)

### GraphQL Patterns

| Pattern | Intent |
|---|---|
| **Query Complexity Limiting** | Assign a cost to each field; reject queries exceeding a budget; prevents O(n²) queries via nested relations |
| **DataLoader / N+1 Prevention** | Batch and cache data fetching per request; coalesces multiple individual loads into a single batch query |
| **Schema-First Design** | Define the schema as a contract before implementing resolvers; teams can work in parallel |
| **Cursor-Based Connections** | Relay specification: `edges`, `node`, `cursor`, `PageInfo`; stable pagination for collections |
| **Persisted Queries** | Pre-register queries by hash; clients send hash not query string; reduces payload, enables whitelisting |

**Canonical source:** Lee Byron & Nick Schrock, GraphQL specification; Relay Cursor Connections Specification

### gRPC / Protocol Buffer Patterns

| Pattern | Intent |
|---|---|
| **Service Definition First** | Define `.proto` IDL → generate code; contract is the source of truth |
| **Unary vs. Streaming** | Four RPC types: unary, server-streaming, client-streaming, bidirectional streaming; choose per interaction shape |
| **Deadlines / Cancellation** | Always set deadlines on RPCs; propagate cancellation context through call chains |
| **Interceptors / Middleware** | Chain unary and streaming interceptors for cross-cutting concerns (auth, logging, tracing, retry) |
| **Backward Compatibility Rules** | Never change field numbers; never remove required fields; add optional fields only; enables rolling upgrades |

**Canonical source:** Google Protocol Buffers documentation; gRPC documentation

### Internal API / Module Design Patterns

| Pattern | Intent |
|---|---|
| **Facade** | Expose a simple interface over a complex subsystem (GoF Stage 3 — applies to APIs too) |
| **Gateway** | A single entry point that routes, aggregates, and translates between internal services and clients |
| **Strangler Fig** | Incrementally replace one API with another using a proxy layer (Stage 7 — applies at API level too) |
| **Tolerant Reader** | When consuming an API, only extract what you need; ignore unknown fields; allows producers to evolve |
| **Consumer-Driven Contracts** | Consumers define the contract they need; producers verify they fulfill it; Pact framework |

### What to understand before moving on
- What is idempotency and why does it matter for retries? Which HTTP methods are idempotent by definition?
- What is the N+1 query problem in GraphQL/ORMs and how does DataLoader solve it?
- Why does cursor-based pagination beat offset pagination for large datasets under concurrent writes?

---

## Full Reading List

### Books (ordered by dependency)

| # | Title | Authors | Year | Stage |
|---|---|---|---|---|
| 1 | *The Pragmatic Programmer* | Hunt & Thomas | 1999 | 1 |
| 2 | *Design Principles and Design Patterns* | Robert C. Martin | 2000 | 1 |
| 3 | *Agile Software Development, Principles, Patterns, and Practices* | Robert C. Martin | 2002 | 1 |
| 4 | *Applying UML and Patterns* (3rd ed.) | Craig Larman | 2004 | 1 |
| 5 | *Design Patterns: Elements of Reusable Object-Oriented Software* | Gamma, Helm, Johnson, Vlissides | 1994 | 2–4 |
| 6 | *Head First Design Patterns* | Freeman & Robson | 2004 | 2–4 (accessible entry) |
| 7 | *Pattern-Oriented Software Architecture, Vol. 2* (POSA2) | Schmidt, Stal, Rohnert, Buschmann | 2000 | 5 |
| 8 | *Java Concurrency in Practice* | Goetz et al. | 2006 | 5 |
| 9 | *The Art of Multiprocessor Programming* | Herlihy & Shavit | 2008 | 5 |
| 10 | *Patterns of Enterprise Application Architecture* | Martin Fowler | 2002 | 6 |
| 11 | *Domain-Driven Design* | Eric Evans | 2003 | 6 |
| 12 | *Implementing Domain-Driven Design* | Vaughn Vernon | 2013 | 6 |
| 13 | *Release It!* (2nd ed.) | Michael T. Nygard | 2018 | 7 |
| 14 | *Building Microservices* (2nd ed.) | Sam Newman | 2021 | 7 |
| 15 | *Designing Data-Intensive Applications* | Martin Kleppmann | 2017 | 7–8 |
| 16 | *Category Theory for Programmers* | Bartosz Milewski | 2017 | 8 |
| 17 | *Functional Programming in Scala* | Chiusano & Bjarnason | 2014 | 8 |
| 18 | *The Joy of Clojure* | Fogus & Houser | 2014 | 8 |

### Seminal Papers & Articles

| Title | Authors | Year | Stage |
|---|---|---|---|
| "Data Abstraction and Hierarchy" | Barbara Liskov | 1987 | 1 |
| Fielding REST dissertation | Roy Fielding | 2000 | 9 |
| "CQRS" bliki | Martin Fowler | 2011 | 7 |
| "Event Sourcing" | Martin Fowler | 2005 | 7 |
| "Saga pattern" (original paper) | Garcia-Molina & Salem | 1987 | 7 |
| Reactive Manifesto | Bonér, Farley, Kuhn, Thompson | 2014 | 8 |
| Reactive Streams Specification | Bonér et al. | 2013 | 8 |
| "Your Mouse is a Database" | Erik Meijer | 2012 | 8 |
| "Railway-Oriented Programming" | Scott Wlaschin | 2014 | 8 |

### Engineering Blogs & Online References

| Source | What it covers |
|---|---|
| [martinfowler.com](https://martinfowler.com) | PoEAA patterns, CQRS, Event Sourcing, DDD, refactoring — Fowler's living catalog |
| [refactoring.guru/design-patterns](https://refactoring.guru/design-patterns) | GoF patterns with real-world examples, UML, code in multiple languages |
| [sourcemaking.com/design_patterns](https://sourcemaking.com/design_patterns) | GoF patterns + anti-patterns |
| [Azure Architecture Patterns](https://learn.microsoft.com/en-us/azure/architecture/patterns/) | Cloud patterns: Circuit Breaker, CQRS, Event Sourcing, Saga, Strangler Fig, Outbox, Bulkhead, etc. |
| [microservices.io](https://microservices.io/patterns/) | Chris Richardson's microservices pattern catalog: Saga, CQRS, Outbox, API Gateway, Sidecar |
| [aip.dev](https://aip.dev) | Google API Improvement Proposals — REST API design guidance |
| [fsharpforfunandprofit.com](https://fsharpforfunandprofit.com) | Railway-Oriented Programming; functional domain modeling |
| [Bartosz Milewski's Blog](https://bartoszmilewski.com) | Category theory → functional patterns |
| [High Scalability](http://highscalability.com) | Real-world systems engineering case studies |
| Nygard's blog — [thinkrelevance.com](https://thinkrelevance.com) | Stability patterns; Circuit Breaker origin |

---

## Pattern Anti-Patterns to Know

These are patterns that are commonly misapplied or overused. Know the pattern, know the failure mode.

| Anti-Pattern | What goes wrong |
|---|---|
| **Singleton overuse** | Becomes global state; makes testing impossible; hides dependencies |
| **Anemic Domain Model** | Domain objects are data bags; all logic in service classes; violates DDD's core idea |
| **God Object** | One class knows everything and does everything; violates SRP completely |
| **Premature CQRS** | Applying CQRS to a simple CRUD system adds operational complexity with no benefit; Fowler's warning |
| **Event Sourcing everywhere** | Event replay for simple systems is painful — debugging, tooling, schema evolution all become harder |
| **Service Locator** | Looks like DI but hides dependencies; makes testing hard; prefer explicit constructor injection |
| **Overloaded Aggregate** | DDD Aggregates that span too much; become bottlenecks and concurrency nightmares |
| **Chatty Interfaces** | Too many fine-grained calls across a boundary; should be one coarse-grained call |
| **Inheritance for code reuse** | Coupling is transitive in inheritance; prefer composition for reuse |
| **Magic Strings / Stringly-typed systems** | Behavioral variation encoded in strings instead of types; no compiler help |

---

## Questions to Come Back To (Cross-Stage)

1. Every pattern is a named solution to a recurring problem in a context. What are the three parts of that definition — and why does "context" matter as much as "solution"?
2. The GoF write: "Each design pattern systematically names, explains, and evaluates an important and recurring design." What does "recurring" mean for pattern validity?
3. SOLID principles, GoF patterns, and PoEAA patterns were all developed in the OOP era. Which translate cleanly to functional programming? Which don't?
4. Concurrency patterns from POSA2 assume threads and shared memory. How do they map to actor models (Erlang/Akka) or async/await models?
5. Circuit Breaker, Bulkhead, and Retry all address the same root problem: partial failures in distributed systems. When should you layer all three vs. pick one?
6. CQRS and Event Sourcing are architectural patterns, not LLD patterns in the traditional sense. What makes them relevant at the class/service design level?
7. The Outbox Pattern solves the dual-write problem. What is the dual-write problem — and why can't a distributed transaction (2PC) just solve it instead?
