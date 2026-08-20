# Hexagonal Architecture

## 1. Overview

This project follows **Hexagonal Architecture**, also known as **Ports & Adapters Architecture**.

The main objective is to protect the application core from external technologies and infrastructure concerns.

Business rules and application use cases remain at the center of the system, while external components communicate with the core through explicitly defined ports.

```text
                 External World
                       │
                       ▼
              ┌─────────────────┐
              │  Input Adapter  │
              │    REST API     │
              └────────┬────────┘
                       │
                  Input Port
                       │
                       ▼
              ┌─────────────────┐
              │    Use Cases    │
              │   Application   │
              └────────┬────────┘
                       │
                       ▼
              ┌─────────────────┐
              │     Domain      │
              │ Business Rules  │
              └────────┬────────┘
                       │
                  Output Port
                       ▲
                       │
              ┌────────┴────────┐
              │ Output Adapter  │
              │   Persistence   │
              └────────┬────────┘
                       │
                       ▼
                  PostgreSQL
```

---

## 2. Application Core

The application core contains the business behavior of the system.

It is composed primarily of:

* Domain models.
* Business rules.
* Use cases.
* Input ports.
* Output ports.

The core must not depend directly on infrastructure technologies.

Examples of technologies that must remain outside the core include:

* Spring Web.
* Spring Data JPA.
* Hibernate.
* PostgreSQL.
* HTTP.
* Docker.

This separation allows the business behavior to evolve independently from infrastructure decisions.

---

## 3. Domain

The domain represents the business concepts and rules of the application.

Examples include:

* `Order`
* `OrderItem`
* `OrderStatus`

The domain is responsible for protecting its own invariants.

For example:

```text
Order
 ├── must contain valid items
 ├── calculates its own total
 ├── controls valid status transitions
 ├── prevents modification after completion
 └── prevents modification after cancellation
```

Domain objects must not contain annotations or dependencies related to infrastructure frameworks.

For example, domain models must not require:

```java
@Entity
@Table
@Column
@RestController
@Service
@Repository
```

The domain should remain plain Java whenever possible.

---

## 4. Application Layer

The application layer coordinates the execution of business operations.

Each relevant application operation is represented as a **use case**.

Examples:

```text
CreateOrder
GetOrder
ListOrders
AddOrderItem
UpdateOrderStatus
CancelOrder
```

Use cases orchestrate domain objects and communicate with external resources through output ports.

They must not know how those external resources are implemented.

---

## 5. Input Ports

Input ports define the operations that the application exposes to the outside world.

Example:

```java
public interface CreateOrderUseCase {

    Order create(CreateOrderCommand command);

}
```

The REST controller depends on this contract.

```text
REST Controller
      │
      ▼
CreateOrderUseCase
      │
      ▼
Application Service
```

This prevents the REST adapter from becoming directly coupled to the internal implementation of the use case.

---

## 6. Output Ports

Output ports define capabilities required by the application from external systems.

Persistence is one example.

```java
public interface OrderRepositoryPort {

    Order save(Order order);

    Optional<Order> findById(UUID id);

}
```

The application defines the contract.

The infrastructure provides the implementation.

```text
Application
     │
     ▼
OrderRepositoryPort
     ▲
     │
Persistence Adapter
     │
     ▼
Spring Data JPA
     │
     ▼
PostgreSQL
```

The application therefore knows that orders can be persisted, but it does not know whether the implementation uses PostgreSQL, MongoDB, an external API, or another technology.

---

## 7. Input Adapters

Input adapters allow external actors to invoke application use cases.

The initial input adapter will be a REST API implemented with Spring Web.

Responsibilities include:

* Receiving HTTP requests.
* Validating request contracts.
* Converting request DTOs into application commands.
* Calling input ports.
* Converting application results into HTTP responses.
* Mapping application errors to HTTP status codes.

Input adapters must not contain business rules.

Example:

```text
POST /orders
      │
      ▼
OrderController
      │
      ▼
CreateOrderUseCase
```

---

## 8. Output Adapters

Output adapters implement capabilities required by the application.

The initial persistence adapter will use:

* Spring Data JPA.
* Hibernate.
* PostgreSQL.

Responsibilities include:

* Implementing output ports.
* Mapping domain objects to persistence entities.
* Mapping persistence entities back to domain objects.
* Executing infrastructure-specific operations.

Example:

```text
OrderRepositoryPort
        ▲
        │ implements
        │
OrderPersistenceAdapter
        │
        ▼
SpringDataOrderRepository
        │
        ▼
PostgreSQL
```

---

## 9. Domain Model vs Persistence Entity

Domain models and persistence entities serve different purposes and will remain separated.

### Domain Model

Represents business behavior.

```text
Order
```

Contains:

* Business state.
* Business rules.
* Domain behavior.

It does not contain JPA annotations.

### Persistence Entity

Represents how data is stored.

```text
OrderEntity
```

Contains infrastructure-specific mapping such as:

```java
@Entity
@Table(name = "orders")
```

The persistence adapter is responsible for converting between both representations.

```text
Order
  │
  ▼
OrderPersistenceMapper
  │
  ▼
OrderEntity
```

This prevents database concerns from leaking into the domain.

---

## 10. Dependency Inversion

Dependency inversion is one of the central principles of this architecture.

Instead of the application depending on a concrete repository:

```text
Application
     │
     ▼
PostgreSQL Repository
```

the application depends on an abstraction that it controls:

```text
Application
     │
     ▼
OrderRepositoryPort
     ▲
     │
Persistence Adapter
```

The infrastructure therefore depends on the application's contract.

This reverses the traditional dependency direction.

---

## 11. Dependency Rules

The project follows these dependency rules:

### Domain

May depend on:

* Java language and standard library.
* Other domain components.

Must not depend on:

* Application adapters.
* Spring.
* JPA.
* HTTP.
* Database implementations.

### Application

May depend on:

* Domain.
* Application ports.

Must not depend on:

* REST controllers.
* JPA repositories.
* PostgreSQL.
* Infrastructure implementations.

### Input Adapters

May depend on:

* Input ports.
* Application contracts.
* Framework-specific web components.

### Output Adapters

May depend on:

* Output ports.
* Domain models when mapping is required.
* Infrastructure frameworks.

The core must never depend on an adapter.

---

## 12. Expected Benefits

The architecture provides:

### Low Coupling

Business rules are not directly coupled to infrastructure technologies.

### Testability

Use cases can be tested using mocked output ports without requiring a database or web server.

### Replaceable Infrastructure

Infrastructure implementations can be changed without modifying core business rules.

### Explicit Boundaries

Ports clearly define how the application communicates with external components.

### Maintainability

Responsibilities remain separated, reducing the impact of infrastructure changes on business logic.

---

## 13. Architectural Principle

The central rule of the project is:

> **The application core defines what it needs. Infrastructure decides how to provide it.**

Ports belong to the application boundary.

Adapters belong to infrastructure.

Dependencies always point toward the application core.
