# Project Overview

## 1. Purpose

The **Hexagonal Architecture API** is a RESTful application responsible for managing customer orders and their lifecycle.

The system is designed using **Hexagonal Architecture (Ports & Adapters)** to keep business rules independent from frameworks, persistence mechanisms, delivery mechanisms, and other infrastructure concerns.

The architecture establishes explicit boundaries between the application core and external components, allowing infrastructure technologies to evolve without directly affecting business logic.

---

## 2. Objectives

The main architectural objectives of the project are:

* Keep business rules isolated from infrastructure concerns.
* Apply dependency inversion between the application core and external components.
* Define explicit contracts through input and output ports.
* Separate use cases from delivery and persistence mechanisms.
* Reduce coupling with frameworks such as Spring and Hibernate.
* Improve unit testability of business rules and application use cases.
* Allow adapters to be replaced with minimal impact on the application core.

---

## 3. Business Context

The application manages the lifecycle of customer orders.

An order contains one or more items and progresses through defined states during its lifecycle.

The system is responsible for operations such as:

* Creating an order.
* Adding items to an order.
* Calculating the total order value.
* Retrieving order information.
* Managing order status.
* Cancelling an order.

Business rules related to these operations belong to the application core and must not depend directly on HTTP, databases, persistence frameworks, or other external technologies.

---

## 4. Scope

The initial scope includes:

### Order Management

* Create orders.
* Retrieve orders by identifier.
* List orders.
* Add items to an existing order.
* Calculate order totals.
* Update order status.
* Cancel orders.

### Persistence

Order information will be persisted in a relational database through an **output port**.

The application core will define the persistence contract without knowing which database or persistence technology implements it.

The initial persistence adapter will use:

* PostgreSQL
* Spring Data JPA
* Hibernate
* Flyway

### API

The application will expose its use cases through a REST adapter using:

* Spring Web
* Bean Validation
* OpenAPI / Swagger

HTTP-specific concerns will remain inside the input adapter layer.

---

## 5. Architectural Boundaries

The system is divided into three main areas:

### Domain

Contains business concepts and rules.

The domain must remain independent from:

* Spring
* JPA
* HTTP
* PostgreSQL
* Controllers
* Persistence implementations

### Application

Contains the application's use cases and ports.

Input ports define the operations offered by the application.

Output ports define the capabilities the application requires from external systems.

### Adapters

Adapters connect external technologies to the ports defined by the application.

Examples include:

* REST controllers as input adapters.
* PostgreSQL/JPA persistence as an output adapter.

---

## 6. Dependency Direction

Dependencies must point toward the application core.

```text
External World
      │
      ▼
Input Adapter
      │
      ▼
Input Port
      │
      ▼
Use Case
      │
      ▼
Domain
      │
      ▼
Output Port
      ▲
      │
Output Adapter
      ▲
      │
Database / External System
```

The application core defines the contracts.

Infrastructure components depend on those contracts rather than the core depending on infrastructure implementations.

---

## 7. Out of Scope

The initial version does not include:

* Payment processing.
* Inventory management.
* Authentication and authorization.
* Messaging or event brokers.
* Distributed microservices.
* External customer management systems.

These capabilities may be introduced in future versions when they provide meaningful architectural value.

---

## 8. Expected Result

The final application should demonstrate an implementation where business rules and use cases can be tested independently from infrastructure.

Replacing components such as the database, REST interface, or persistence framework should not require changes to the core business logic.
