# Design Patterns Knowledge Base — SuperArchitect OS

**Version:** 1.0
**Owner:** SuperArchitect OS Core Team
**Last Updated:** 2026-03-28
**Audience:** Architect and Backend agents; any agent making application-level design decisions

---

## Overview

Design patterns are proven solutions to recurring problems in software design. They are not code to copy — they are templates to adapt. Knowing when to apply a pattern is more important than knowing how to implement it.

**Three rules for pattern use in the SuperArchitect OS:**
1. Apply patterns to solve real, present problems — not anticipated future ones
2. Name the pattern in code and documentation so readers recognize it
3. Know the anti-pattern version of every pattern (see `os/knowledge/anti-patterns.md`)

---

## Creational Patterns

Creational patterns abstract the process of object creation, making systems independent of how their objects are created, composed, and represented.

---

### Factory Method

**Intent:** Define an interface for creating an object, but let subclasses decide which class to instantiate. Defers instantiation to subclasses.

**Structure:**
- `Creator` declares the factory method returning a `Product` interface
- `ConcreteCreator` overrides the factory method to return a `ConcreteProduct`
- Client code works with `Product` interface — never knows which concrete type it has

**When to Use:**
- When a class cannot anticipate the type of objects it must create
- When subclasses should control the type of objects the superclass creates
- When you want to provide a hook for subclasses to extend an object's creation

**Example Application:**
A payment processing system where `PaymentProcessorFactory` returns `StripeProcessor`, `PayPalProcessor`, or `BankTransferProcessor` based on configuration. The checkout flow calls `factory.create_processor()` and never knows which concrete implementation it's using.

**Common Pitfalls:**
- Overuse: if you only ever have one product type, the pattern adds unnecessary indirection
- Creeping Factory: the factory grows to create everything in the system (becomes a god object)
- Hidden instantiation: factories should be explicit, not called in surprising places

---

### Abstract Factory

**Intent:** Provide an interface for creating families of related or dependent objects without specifying their concrete classes.

**Structure:**
- `AbstractFactory` interface declares creation methods for each product type in the family
- `ConcreteFactory` implements creation of a specific family (e.g., Windows UI factory, Mac UI factory)
- Products from the same factory are designed to work together

**When to Use:**
- When a system must be independent of how its products are created, composed, and represented
- When a system must work with multiple families of products
- When you want to enforce that related products are used together (consistency guarantee)

**Example Application:**
A notification system with `EmailNotificationFactory` and `SMSNotificationFactory`, each creating matching `MessageFormatter`, `MessageSender`, and `DeliveryTracker` that work consistently together.

**Common Pitfalls:**
- Supporting new product types requires changing all factory interfaces
- Can become overcomplicated if the product families have subtle variations

---

### Builder

**Intent:** Separate the construction of a complex object from its representation, allowing the same construction process to create different representations.

**Structure:**
- `Builder` interface declares steps for building parts of a complex object
- `ConcreteBuilder` implements the steps, maintains the product under construction
- `Director` controls the construction sequence
- `Product` is the complex object being built

**When to Use:**
- When construction involves many optional parameters (replaces telescoping constructors)
- When the same build process must produce different representations
- When you want fine control over the construction process

**Example Application:**
Constructing an `HttpRequest` with optional headers, query parameters, request body, timeout, and retry policy. The builder pattern provides a fluent API (`request.withHeader(...).withTimeout(...).withRetry(...).build()`) that is far more readable than a 12-parameter constructor.

**Common Pitfalls:**
- Mutable builder state — if builders are shared, partial state can leak
- Over-engineering simple objects that only need 2–3 parameters (use plain constructors instead)
- Forgetting to validate the completed product in the `build()` method

---

### Prototype

**Intent:** Specify the kinds of objects to create using a prototypical instance, and create new objects by copying the prototype.

**Structure:**
- `Prototype` interface declares a `clone()` method
- `ConcretePrototype` implements the `clone()` method
- Client creates new objects by asking a prototype to clone itself

**When to Use:**
- When object creation is expensive (large initialization, DB lookup) and copies are cheaper
- When classes to instantiate are specified at runtime
- When you need to build a registry of objects that can be cloned on demand

**Example Application:**
A document template system where baseline document configurations are stored as prototypes and new documents are created by cloning the appropriate prototype, then modifying specific fields.

**Common Pitfalls:**
- Shallow vs. deep copy confusion — cloning must handle nested objects correctly
- Circular references in the object graph can make cloning recursive
- Complex objects with resource handles (file handles, DB connections) require careful clone semantics

---

### Singleton (and Why to Avoid It)

**Intent:** Ensure a class has only one instance, and provide a global point of access to it.

**Structure:**
- Private constructor prevents external instantiation
- Static `getInstance()` method returns the single instance, creating it on first call
- Instance stored as a private static field

**The Problem with Singleton:**
Singleton is the most misused and overused design pattern. It violates the Single Responsibility Principle (managing its own lifecycle in addition to its main job), makes testing harder (global state is hard to reset between tests), introduces hidden dependencies (callers import a global, not an explicit dependency), and creates coupling to a specific implementation.

**When Singleton is Acceptable:**
- System-level resources where exactly one instance is physically constrained (e.g., a handle to a hardware device)
- Logging infrastructure (though even here, dependency injection is preferable)
- Configuration objects read at startup and never modified

**The Better Alternative:**
In almost all cases, a singleton should be replaced with dependency injection of a shared instance. The container or composition root creates one instance and injects it everywhere it's needed. This gives the same "one instance" behavior with testability, explicitness, and flexibility.

---

## Structural Patterns

Structural patterns explain how to assemble objects and classes into larger structures while keeping these structures flexible and efficient.

---

### Adapter

**Intent:** Convert the interface of a class into another interface clients expect. Lets classes work together that couldn't otherwise because of incompatible interfaces.

**Structure:**
- `Target` is the interface the client expects
- `Adaptee` is the existing class with an incompatible interface
- `Adapter` wraps the `Adaptee` and implements the `Target` interface

**When to Use:**
- When integrating with a third-party library that has an incompatible interface
- When reusing existing classes without modifying them
- At system boundaries (ports-and-adapters architecture)

**Example Application:**
Your application expects a `NotificationService` interface with `send(recipient, message)`. Your team just switched email providers from SendGrid to Postmark. An `PostmarkAdapter` wraps the Postmark SDK and implements `NotificationService`. The application code never changes.

**Common Pitfalls:**
- Adapter doing too much (business logic in the adapter layer)
- Not testing both sides of the adapter (test the adapter's translation, not just the adaptee)

---

### Bridge

**Intent:** Decouple an abstraction from its implementation so that the two can vary independently.

**Structure:**
- `Abstraction` maintains a reference to the `Implementor`
- `RefinedAbstraction` extends the abstraction
- `Implementor` interface is separate from the abstraction
- `ConcreteImplementor` provides a specific implementation

**When to Use:**
- When you want to avoid a permanent binding between abstraction and implementation
- When both abstractions and implementations should be extensible by subclassing
- When changes in implementation should not affect client code

**Example Application:**
A reporting system has `Report` abstractions (SalesReport, InventoryReport) and `Renderer` implementations (PDF, HTML, CSV). Without Bridge, you'd need `SalesReportPDF`, `SalesReportHTML`, `InventoryReportPDF`, etc. — N×M classes. Bridge gives you N + M classes.

**Common Pitfalls:**
- Over-engineering when only one implementation exists
- Confusion with Adapter (Bridge is designed upfront; Adapter makes existing code work together)

---

### Composite

**Intent:** Compose objects into tree structures to represent part-whole hierarchies. Lets clients treat individual objects and compositions uniformly.

**Structure:**
- `Component` declares the interface for objects in the composition
- `Leaf` has no children; defines behavior for primitive objects
- `Composite` stores child components and implements child-related operations

**When to Use:**
- When you want clients to ignore the difference between compositions and individual objects
- When you need to represent tree structures (UI component trees, file systems, org charts)

**Example Application:**
A UI component tree where `Button`, `Input`, and `Text` are leaves and `Form`, `Panel`, `Page` are composites. A layout engine calls `component.render()` on the root and it propagates uniformly through the entire tree.

**Common Pitfalls:**
- Making the component interface too general to accommodate all possible leaf and composite behaviors
- Circular references in tree structures

---

### Decorator

**Intent:** Attach additional responsibilities to an object dynamically. Provides a flexible alternative to subclassing for extending functionality.

**Structure:**
- `Component` interface declares operations
- `ConcreteComponent` defines the base behavior
- `Decorator` maintains a reference to a component and conforms to the component interface
- `ConcreteDecorator` adds behavior before/after calling the wrapped component's operation

**When to Use:**
- When you need to add responsibilities to objects without knowing them at compile time
- When you need to add features without creating a combinatorial explosion of subclasses
- When extension by subclassing is impractical (e.g., the class is `final`)

**Example Application:**
An HTTP request handler where `LoggingDecorator`, `AuthDecorator`, `CachingDecorator`, and `RateLimitDecorator` each wrap the base handler. Stacking decorators builds the full middleware pipeline.

**Common Pitfalls:**
- Decorator order dependence (auth must come before caching, etc.) — make order explicit
- Too many small decorators become hard to debug
- Identity issues: a decorated object is not `instanceof` the concrete class

---

### Facade

**Intent:** Provide a simplified interface to a complex subsystem.

**Structure:**
- `Facade` provides a simple interface that delegates to subsystem classes
- `Subsystem` classes do the actual work; unaware of the facade

**When to Use:**
- When you want to provide a simple interface to a complex body of code
- When you want to layer your subsystems — each subsystem exposes a facade to higher layers
- To minimize the coupling between clients and subsystem internals

**Example Application:**
An `OrderFacade` that exposes `placeOrder(cart, customer)` and internally orchestrates inventory check, payment processing, order creation, confirmation email, and analytics event. Callers don't need to know about any of the subsystems.

**Common Pitfalls:**
- Facade becoming a god class that knows too much about too many subsystems
- Hiding necessary complexity (sometimes callers legitimately need fine-grained access)

---

### Flyweight

**Intent:** Use sharing to support large numbers of fine-grained objects efficiently. Reduce memory use by sharing state among similar objects.

**Structure:**
- `Flyweight` declares an interface through which flyweights can act on extrinsic state
- `ConcreteFlyweight` stores intrinsic (shared, context-free) state
- `FlyweightFactory` creates and manages flyweight objects; ensures sharing

**When to Use:**
- When a large number of similar objects consume too much memory
- When most object state can be made extrinsic (passed in at runtime)
- When object identity does not matter

**Example Application:**
Rendering 100,000 characters in a document editor. Each character cell stores only the character code (intrinsic state). Font, size, color (extrinsic state) are passed at render time. Without Flyweight, each character holds a full style object.

**Common Pitfalls:**
- Complexity of separating intrinsic from extrinsic state
- Thread safety when extrinsic state is mutated

---

### Proxy

**Intent:** Provide a surrogate or placeholder for another object to control access to it.

**Structure:**
- `Subject` declares common interface for `RealSubject` and `Proxy`
- `RealSubject` defines the real object the proxy represents
- `Proxy` maintains a reference to the RealSubject; controls access and may add additional behavior

**When to Use:**
- **Virtual proxy**: Lazy initialization of expensive objects
- **Remote proxy**: Local representative for an object in a different address space
- **Protection proxy**: Control access based on permissions
- **Caching proxy**: Cache results of expensive operations

**Example Application:**
A `CachingImageProxy` that holds an image path and only loads the full image from disk when `render()` is first called. Subsequent calls return the cached version. The proxy is indistinguishable from the real image object to callers.

**Common Pitfalls:**
- Proxy response time can be unpredictable if the RealSubject has variable latency
- Over-proxying — adding proxy layers that each add a small cost

---

## Behavioral Patterns

Behavioral patterns are concerned with algorithms and the assignment of responsibilities between objects.

---

### Chain of Responsibility

**Intent:** Avoid coupling the sender of a request to its receiver by giving more than one object a chance to handle the request. Chain the receiving objects and pass the request along the chain until an object handles it.

**When to Use:** Middleware pipelines, approval workflows, event processing.

**Example:** HTTP request middleware — authentication → rate limiting → validation → handler. Each link either handles or passes the request.

**Common Pitfalls:** Requests that fall through the entire chain unhandled; chains that are difficult to debug.

---

### Command

**Intent:** Encapsulate a request as an object, allowing you to parameterize clients with requests, queue or log requests, and support undoable operations.

**When to Use:** Undo/redo functionality, job queues, transactional operations, macro recording.

**Example:** A text editor where every edit (`InsertCommand`, `DeleteCommand`, `FormatCommand`) is an object stored on an undo stack.

**Common Pitfalls:** Command objects accumulating state that belongs in the domain model.

---

### Iterator

**Intent:** Provide a way to sequentially access elements of a collection without exposing its underlying representation.

**When to Use:** Traversal of complex data structures where the traversal algorithm should be independent of the data structure.

**Example:** A paginated API result set that presents as a simple iterable, handling page fetching transparently.

**Common Pitfalls:** Invalidating iterators by modifying the collection during iteration.

---

### Mediator

**Intent:** Define an object that encapsulates how a set of objects interact. Promotes loose coupling by keeping objects from referring to each other explicitly.

**When to Use:** When a set of objects communicate in complex but well-defined ways; when reuse of these objects is difficult because they have too many connections.

**Example:** An air traffic control system where planes don't communicate directly — they communicate through the controller. Equivalent to a chat room mediator.

**Common Pitfalls:** Mediator becoming a god object that knows everything about every participant.

---

### Observer

**Intent:** Define a one-to-many dependency between objects so that when one object changes state, all its dependents are notified and updated automatically.

**When to Use:** When a change in one object requires changing others and you don't know how many objects need to change; event-driven architectures.

**Example:** UI components updating when underlying data model changes; domain event publishers and subscribers.

**Common Pitfalls:** Memory leaks when observers are not unregistered; cascade update storms; unpredictable observer ordering.

---

### State

**Intent:** Allow an object to alter its behavior when its internal state changes. The object will appear to change its class.

**When to Use:** When an object's behavior depends on its state and must change at runtime; when complex conditionals around state are proliferating.

**Example:** An `Order` object with states `Pending`, `Confirmed`, `Shipped`, `Delivered`, `Cancelled` where each state has different allowed transitions and behaviors.

**Common Pitfalls:** Proliferation of state classes for simple state machines (switch/case may be cleaner); forgetting to handle invalid state transitions.

---

### Strategy

**Intent:** Define a family of algorithms, encapsulate each one, and make them interchangeable. Lets the algorithm vary independently from clients that use it.

**When to Use:** When you need different variants of an algorithm; when algorithm details should be hidden from clients; to replace conditional logic that selects behavior.

**Example:** A sorting service with `QuickSortStrategy`, `MergeSortStrategy`, `TimSortStrategy` that can be swapped at runtime based on input size and data characteristics.

**Common Pitfalls:** Over-engineering when only one strategy will ever exist; strategy objects with too much context dependency.

---

### Template Method

**Intent:** Define the skeleton of an algorithm in an operation, deferring some steps to subclasses. Lets subclasses redefine certain steps without changing the algorithm's structure.

**When to Use:** When you have invariant parts of an algorithm and variable parts that subclasses must fill in.

**Example:** A `DataImporter` base class with a fixed pipeline: validate → parse → transform → persist. Subclasses override specific steps for CSV vs. JSON vs. XML.

**Common Pitfalls:** The "Hollywood Problem" (don't call us, we'll call you) can confuse developers; deep inheritance hierarchies from overuse.

---

## Application-Level Patterns

These patterns operate at the application architecture level, organizing how domain logic, data, and use cases interact.

---

### Repository Pattern

**Intent:** Encapsulate the logic required to access data sources. Centralizes common data access functionality, promoting a cleaner separation between domain and data mapping layers.

**Structure:** Repository interface defined in the domain layer. Concrete repositories (Postgres, Mongo, In-Memory) implemented in the infrastructure layer. Domain objects call repositories through the interface; no SQL or ORM code in domain services.

**When to Use:** Always, for any non-trivial persistence of domain objects.

**Example Application:**
```
Domain layer:    OrderRepository (interface: find_by_id, save, delete)
Infrastructure:  PostgresOrderRepository implements OrderRepository
Infrastructure:  InMemoryOrderRepository implements OrderRepository (for tests)
```

**Common Pitfalls:** Repository methods that return database-specific types (leaking infrastructure); "generic repository" anti-pattern where the repository is so generic it provides no domain-meaningful operations.

---

### Unit of Work

**Intent:** Maintain a list of objects affected by a business transaction and coordinate the writing out of changes and the resolution of concurrency problems.

**Structure:** `UnitOfWork` tracks new, modified, and deleted objects during a business operation and commits or rolls them back atomically.

**When to Use:** When multiple repository operations must succeed or fail as a unit (e.g., creating an order and decrementing inventory must be atomic).

**Example Application:** A database transaction wrapper that all repository operations in a use case share. Commit is called once at the end of the use case handler.

**Common Pitfalls:** Long-running Units of Work with many tracked objects (memory pressure, lock contention); not flushing the Unit of Work before reads within the same transaction.

---

### Specification Pattern

**Intent:** Encapsulate business rules that determine whether an object satisfies some criteria. Specifications can be combined using logical operators.

**Structure:** `Specification<T>` has `is_satisfied_by(candidate: T) -> bool`. Specifications are composable: `a.and(b)`, `a.or(b)`, `a.not()`.

**When to Use:** When complex filtering or validation logic would otherwise pollute domain objects or services; when business rules need to be composed and reused across the system.

**Example Application:** `EligibleForDiscountSpec = ActiveCustomerSpec.and(PurchasedLastMonthSpec.or(PremiumMemberSpec))`. Each specification is a testable, named business rule.

**Common Pitfalls:** Performance issues when specifications are evaluated in memory rather than translated to DB queries; over-engineering simple conditions that don't need composability.

---

### CQRS at Application Layer

**Intent:** Separate the command (write) model from the query (read) model at the application layer. Commands change state; queries return data.

**Structure:**
- **Command handlers**: Receive commands, validate, execute domain logic, persist changes
- **Query handlers**: Receive queries, fetch optimized read models, return DTOs
- Read and write models can use different data stores or projections

**When to Use:** When read and write workloads have very different characteristics; when query-optimized views (denormalized, aggregated) would improve read performance.

**Example Application:** Order management where writes go through full domain model validation, and reads use a pre-projected `OrderSummaryView` table optimized for listing and filtering.

**Common Pitfalls:** Eventual consistency complexity when read and write stores diverge; over-applying CQRS to simple CRUD systems where the complexity is not warranted.

---

### Domain Events

**Intent:** Model significant occurrences within the domain as first-class objects. Other parts of the system can subscribe to these events and react without the originating component knowing about them.

**Structure:**
- Domain entity raises an event (e.g., `OrderPlacedEvent`)
- Events are value objects: immutable, describe what happened
- Event handlers respond to events asynchronously or synchronously
- Events are persisted alongside the aggregate state (or via outbox pattern)

**When to Use:** When an action in one domain should trigger reactions in other domains; when you need an audit trail of domain activity; when implementing eventual consistency between bounded contexts.

**Example Application:** `OrderPlacedEvent` triggers: inventory reservation in inventory domain, payment authorization in payment domain, welcome email in notifications domain. Each subscriber is decoupled from the order domain.

**Common Pitfalls:** Event versioning — events must be backward-compatible as schema evolves; event ordering guarantees (when order matters, use sequence numbers or use synchronous handlers); event storm when one event triggers many others in a cascade.

---

### Aggregate Design

**Intent:** Cluster domain objects into a single unit for data consistency. Define a boundary around a cluster of objects with a single root (Aggregate Root) that is the only entry point for modifications.

**Rules:**
1. Each aggregate has exactly one Aggregate Root
2. External objects reference only the Aggregate Root (by ID, not by reference)
3. Transactions must not span aggregate boundaries
4. Aggregates should be small — usually 1–3 entities

**When to Use:** Everywhere in a domain model. Every entity that must maintain invariants belongs to an aggregate.

**Example Application:** An `Order` aggregate containing `OrderLines`. The `Order` is the Aggregate Root — you never add a line item directly. You call `order.add_item(product, quantity)`, and the aggregate enforces invariants like "an order cannot have more than 50 line items."

**Common Pitfalls:** Large aggregates that hold too much state — they become contention bottlenecks and are hard to test; loading entire aggregates when only a partial view is needed (use read models/queries instead).
