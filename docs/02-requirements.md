# Requirements

## 1. Functional Requirements

### FR-01 — Create Order

The system must allow the creation of a new order.

A newly created order must:

* Have a unique identifier.
* Have a creation date.
* Start with an initial status.
* Be associated with at least one order item.

---

### FR-02 — Add Item to Order

The system must allow items to be added to an existing order while the order is still editable.

Each item must contain:

* Product identifier.
* Product name.
* Quantity.
* Unit price.

The system must recalculate the order total after adding an item.

---

### FR-03 — Retrieve Order by ID

The system must allow retrieving an order by its unique identifier.

The response must include:

* Order identifier.
* Creation date.
* Current status.
* Order items.
* Total amount.

---

### FR-04 — List Orders

The system must allow listing registered orders.

The initial implementation may return all orders without pagination.

Pagination may be introduced in future versions.

---

### FR-05 — Calculate Order Total

The system must calculate the total value of an order based on its items.

For each item:

```text
item total = quantity × unit price
```

The final order total must be calculated as:

```text
order total = sum of all item totals
```

---

### FR-06 — Update Order Status

The system must allow valid transitions between order statuses.

The initial order lifecycle will contain the following statuses:

```text
CREATED
CONFIRMED
PROCESSING
COMPLETED
CANCELLED
```

Invalid status transitions must be rejected.

---

### FR-07 — Cancel Order

The system must allow cancellation of an order when cancellation is permitted by the business rules.

Once cancelled, the order must not return to an active status.

---

## 2. Business Rules

### BR-01 — Order Must Contain Items

An order must contain at least one item before it can be created.

---

### BR-02 — Item Quantity Must Be Valid

The quantity of an order item must be greater than zero.

```text
quantity > 0
```

---

### BR-03 — Unit Price Must Be Valid

The unit price of an order item must be greater than zero.

```text
unit price > 0
```

---

### BR-04 — Order Total Is Derived

The total amount of an order must not be manually informed by external clients.

The total must be calculated by the domain based on the order items.

---

### BR-05 — Order Status Must Follow Valid Transitions

The initial valid lifecycle is:

```text
CREATED
   ↓
CONFIRMED
   ↓
PROCESSING
   ↓
COMPLETED
```

Cancellation is allowed only before completion.

```text
CREATED ────────► CANCELLED
CONFIRMED ──────► CANCELLED
PROCESSING ─────► CANCELLED
```

A completed order cannot be cancelled.

A cancelled order cannot transition to another status.

---

### BR-06 — Completed Orders Cannot Be Modified

Once an order reaches the `COMPLETED` status:

* Items cannot be added.
* Items cannot be modified.
* The order cannot be cancelled.

---

### BR-07 — Cancelled Orders Cannot Be Modified

Once an order reaches the `CANCELLED` status:

* Items cannot be added.
* Items cannot be modified.
* Status changes are not allowed.

---

### BR-08 — Order Identifier Must Be Unique

Each order must have a unique identifier.

The domain must not depend on a database-specific mechanism to represent this identifier.

---

## 3. Non-Functional Requirements

### NFR-01 — Architecture

The application must follow **Hexagonal Architecture (Ports & Adapters)**.

Business rules must remain isolated from infrastructure components.

---

### NFR-02 — Dependency Direction

Dependencies must point toward the application core.

The domain and use cases must not depend directly on:

* Spring Framework.
* Spring Data JPA.
* Hibernate.
* PostgreSQL.
* HTTP.
* REST controllers.

---

### NFR-03 — Testability

Business rules and use cases must be testable without starting:

* Spring Application Context.
* HTTP server.
* PostgreSQL.
* Docker containers.

Unit tests must be able to use mocked or in-memory implementations of output ports.

---

### NFR-04 — Persistence

The initial persistence adapter must use:

* PostgreSQL.
* Spring Data JPA.
* Hibernate.
* Flyway.

Persistence entities must remain inside the output adapter and must not become domain entities.

---

### NFR-05 — API Documentation

REST endpoints must be documented using **OpenAPI / Swagger**.

---

### NFR-06 — Validation

Input data received through the REST API must be validated before reaching the application core when the validation is related to the transport contract.

Business validations must remain inside the domain or application layer.

---

### NFR-07 — Error Handling

The REST adapter must provide consistent HTTP error responses.

Infrastructure-specific exceptions must not leak into the application core.

---

### NFR-08 — Database Versioning

Database schema changes must be versioned using **Flyway migrations**.

---

## 4. Initial Constraints

The initial version will use:

* Java 21.
* Spring Boot 3.
* Maven.
* PostgreSQL.
* REST as the primary input adapter.

The architecture must allow these technologies to be replaced or extended without modifying core business rules.

---

## 5. Future Requirements

Possible future extensions include:

* Pagination and filtering.
* Payment integration.
* Inventory validation.
* Messaging with Kafka.
* Domain events.
* Additional input adapters.
* Additional persistence implementations.
* Authentication and authorization.
