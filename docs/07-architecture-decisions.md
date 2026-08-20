# Architecture Decisions

## 1. Overview

This document records the main architectural decisions adopted by the project and the reasoning behind them.

The purpose is to make architectural choices explicit, preserve technical context, and document the trade-offs considered during development.

---

## 2. Hexagonal Architecture

### Decision

The application will follow **Hexagonal Architecture (Ports & Adapters)**.

### Rationale

The business domain should remain independent from infrastructure technologies.

The architecture provides explicit boundaries between:

* Domain rules.
* Application use cases.
* Input mechanisms.
* Persistence mechanisms.
* External integrations.

This allows infrastructure components to depend on contracts defined by the application instead of the application depending directly on infrastructure implementations.

### Consequence

The project requires additional abstractions and mappings compared with a traditional layered architecture.

This additional structure is accepted in exchange for stronger separation of concerns, testability, and infrastructure independence.

---

## 3. Domain Independence

### Decision

Domain classes will be implemented as plain Java objects without Spring, JPA, or HTTP-specific annotations.

### Rationale

The domain represents business concepts and should not require an infrastructure framework to exist or execute its rules.

The following dependencies are therefore prohibited inside the domain:

```text id="a9t06p"
Spring Framework
Spring Data JPA
Hibernate
HTTP
PostgreSQL
REST-specific components
```

### Consequence

Infrastructure-specific representations must be maintained separately and mapped to domain objects.

---

## 4. Separate Domain and Persistence Models

### Decision

Domain models and JPA persistence entities will be represented by separate classes.

Example:

```text id="qsk1bu"
Domain              Persistence

Order               OrderEntity
OrderItem           OrderItemEntity
```

### Rationale

JPA entities have persistence responsibilities that should not define the business model.

Keeping them separated prevents annotations and persistence concerns from leaking into the domain.

### Consequence

Mapping is required between domain and persistence representations.

```text id="v1qzc4"
Order
  │
  ▼
OrderPersistenceMapper
  │
  ▼
OrderEntity
```

This introduces additional code but preserves architectural boundaries.

---

## 5. UUID as Domain Identifier

### Decision

Orders and product references will use `UUID` identifiers.

### Rationale

UUIDs allow identifiers to be created without requiring the domain to depend on database-generated numeric sequences.

This supports domain independence and allows identifiers to exist before persistence occurs.

### Consequence

Database columns must support UUID values and API contracts must represent identifiers consistently.

---

## 6. BigDecimal for Monetary Values

### Decision

Monetary values will use `BigDecimal`.

### Rationale

Floating-point types such as `double` and `float` may introduce precision errors and are not appropriate for business calculations involving monetary values.

### Consequence

Price calculations must use explicit `BigDecimal` operations and appropriate rounding rules whenever division or rounding becomes necessary.

---

## 7. Derived Order Total

### Decision

The order total will be calculated from its items instead of being accepted as externally controlled input.

```text id="5r89vl"
Order Total = Σ (quantity × unitPrice)
```

### Rationale

The total represents derived business information.

Allowing external actors to provide the value could create inconsistent order states.

### Consequence

The domain remains responsible for calculating the authoritative order total.

---

## 8. Domain-Controlled Status Transitions

### Decision

Order lifecycle transitions will be controlled by domain behavior.

Instead of unrestricted status modification:

```text id="7a4imf"
setStatus(COMPLETED)
```

the domain should expose meaningful operations such as:

```text id="j0a8od"
confirm()
startProcessing()
complete()
cancel()
```

### Rationale

Business rules should be enforced by the domain rather than relying on controllers or services to remember which transitions are valid.

### Consequence

Invalid domain states become harder to create accidentally.

---

## 9. Explicit Input Ports

### Decision

Application capabilities will be exposed through explicit input ports.

Examples:

```text id="k9lhza"
CreateOrderUseCase
GetOrderUseCase
ListOrdersUseCase
AddOrderItemUseCase
UpdateOrderStatusUseCase
CancelOrderUseCase
```

### Rationale

Input ports establish a clear boundary between external actors and application behavior.

REST controllers depend on application contracts rather than concrete use-case implementations.

### Consequence

The application can support additional input adapters without moving business logic into those adapters.

---

## 10. Output Ports Owned by the Application

### Decision

Contracts required to communicate with external systems will be defined by the application.

Example:

```text id="9iyjkw"
OrderRepositoryPort
```

### Rationale

The application should define what capability it requires.

Infrastructure determines how that capability is implemented.

This applies the Dependency Inversion Principle.

```text id="yd0jmw"
Application
     │
     ▼
Output Port
     ▲
     │
Infrastructure Adapter
```

### Consequence

Use cases do not depend directly on Spring Data repositories or database implementations.

---

## 11. PostgreSQL as Initial Persistence Technology

### Decision

PostgreSQL will be used as the initial relational database.

### Rationale

The order domain has structured relational data and PostgreSQL provides a mature relational persistence solution.

### Consequence

PostgreSQL-specific configuration remains inside infrastructure concerns.

The core must remain independent from this choice.

---

## 12. Flyway for Database Versioning

### Decision

Database schema evolution will be managed using Flyway migrations.

### Rationale

Database changes should be reproducible, versioned, and stored with the application source code.

### Consequence

Schema changes must be introduced through migration scripts rather than manual database modifications.

---

## 13. Spring Data JPA Inside the Persistence Adapter

### Decision

Spring Data JPA will be used only inside the persistence adapter.

### Rationale

Spring Data provides convenient persistence abstractions but represents an infrastructure concern.

The application core must not depend directly on interfaces such as:

```text id="qmt6yb"
JpaRepository
CrudRepository
```

### Consequence

The persistence adapter bridges the application output port and Spring Data.

---

## 14. REST as the Initial Input Adapter

### Decision

The initial application entry point will be a REST API implemented with Spring Web.

### Rationale

REST provides a simple and widely supported interface for exposing the application's use cases.

### Consequence

HTTP concerns remain inside the input adapter.

The application core must not depend on HTTP status codes, headers, or REST-specific DTOs.

---

## 15. Separate Transport DTOs

### Decision

REST request and response objects will remain separate from domain models.

Conceptually:

```text id="i81lzp"
CreateOrderRequest
        │
        ▼
CreateOrderCommand
        │
        ▼
Domain
```

and:

```text id="yzqsyf"
Domain
   │
   ▼
OrderResponse
```

### Rationale

HTTP contracts and domain models evolve for different reasons.

Separating them prevents transport concerns from controlling the business model.

---

## 16. Business Validation vs Transport Validation

### Decision

Validation will be performed at the layer responsible for the rule.

### Transport Validation

Examples:

* Required JSON fields.
* Invalid request format.
* Invalid UUID representation.

Handled by the REST adapter.

### Business Validation

Examples:

* Quantity must be greater than zero.
* Price must be greater than zero.
* Completed orders cannot be cancelled.
* Invalid status transitions.

Handled by the domain or application core.

### Rationale

Business rules must remain valid regardless of how the application is invoked.

---

## 17. No Premature External Integrations

### Decision

Kafka, payment gateways, inventory systems, and other external integrations will not be introduced in the initial implementation.

### Rationale

Ports should represent actual application requirements rather than hypothetical abstractions.

### Consequence

New ports and adapters will only be introduced when a real use case requires them.

---

## 18. Architectural Trade-Off

Hexagonal Architecture introduces additional concepts compared with a traditional layered application.

For example:

```text id="afm7sn"
Traditional

Controller
   ↓
Service
   ↓
Repository
```

The project instead introduces explicit boundaries:

```text id="22y5bx"
Hexagonal

Input Adapter
     ↓
Input Port
     ↓
Use Case
     ↓
Output Port
     ↑
Output Adapter
```

The additional complexity is considered acceptable because this project prioritizes:

* Explicit architectural boundaries.
* Domain isolation.
* Dependency inversion.
* Testability.
* Replaceable infrastructure.

---

## 19. Decision Principle

Architectural abstractions must solve an actual boundary or dependency problem.

> **The project will favor explicit boundaries without creating abstractions that do not provide architectural value.**

Hexagonal Architecture should guide dependency direction rather than become a reason to create unnecessary interfaces or layers.
