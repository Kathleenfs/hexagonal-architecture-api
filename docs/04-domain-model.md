# Domain Model

## 1. Overview

The domain model represents the core business concepts and rules of the Order Management application.

Following Hexagonal Architecture principles, domain objects remain independent from infrastructure technologies such as Spring, JPA, Hibernate, PostgreSQL, and HTTP.

The initial domain consists of three main concepts:

* `Order`
* `OrderItem`
* `OrderStatus`

```text
Order
 │
 ├── OrderItem
 ├── OrderItem
 ├── OrderItem
 │
 └── OrderStatus
```

---

## 2. Order

`Order` is the main aggregate of the domain.

It represents a customer order and is responsible for maintaining the consistency of its internal state.

### Attributes

| Attribute   | Type            | Description                  |
| ----------- | --------------- | ---------------------------- |
| `id`        | UUID            | Unique order identifier      |
| `items`     | List<OrderItem> | Items belonging to the order |
| `status`    | OrderStatus     | Current lifecycle status     |
| `createdAt` | LocalDateTime   | Order creation timestamp     |
| `updatedAt` | LocalDateTime   | Last update timestamp        |

The total amount is derived from the order items and must not be treated as externally controlled state.

### Responsibilities

`Order` is responsible for:

* Managing its items.
* Calculating its total amount.
* Controlling status transitions.
* Preventing invalid modifications.
* Protecting business invariants.

---

## 3. OrderItem

`OrderItem` represents a product included in an order.

### Attributes

| Attribute     | Type       | Description        |
| ------------- | ---------- | ------------------ |
| `productId`   | UUID       | Product identifier |
| `productName` | String     | Product name       |
| `quantity`    | Integer    | Quantity ordered   |
| `unitPrice`   | BigDecimal | Price of one unit  |

### Derived Value

Each item calculates its subtotal using:

```text
subtotal = quantity × unitPrice
```

The subtotal is derived from the item's state and must not be manually provided.

### Invariants

An `OrderItem` must always satisfy:

```text
quantity > 0
unitPrice > 0
productId != null
productName != null
productName != blank
```

Invalid items must not exist inside a valid `Order`.

---

## 4. Order Total

The total value of an order is calculated from its items.

```text
Order Total
    =
Σ (OrderItem.quantity × OrderItem.unitPrice)
```

Example:

```text
Item A
2 × 50.00 = 100.00

Item B
1 × 25.00 = 25.00

Order Total = 125.00
```

The total must be calculated by the domain.

External clients, controllers, and persistence mechanisms must not define the business value of the total.

---

## 5. Order Status

`OrderStatus` represents the lifecycle state of an order.

The initial statuses are:

```text
CREATED
CONFIRMED
PROCESSING
COMPLETED
CANCELLED
```

The expected lifecycle is:

```text
CREATED
   │
   ▼
CONFIRMED
   │
   ▼
PROCESSING
   │
   ▼
COMPLETED
```

Cancellation can occur before completion:

```text
CREATED ─────────► CANCELLED

CONFIRMED ───────► CANCELLED

PROCESSING ──────► CANCELLED
```

---

## 6. Status Transition Rules

The domain controls valid status transitions.

| Current Status | Allowed Next Status       |
| -------------- | ------------------------- |
| `CREATED`      | `CONFIRMED`, `CANCELLED`  |
| `CONFIRMED`    | `PROCESSING`, `CANCELLED` |
| `PROCESSING`   | `COMPLETED`, `CANCELLED`  |
| `COMPLETED`    | None                      |
| `CANCELLED`    | None                      |

Examples of invalid transitions include:

```text
CREATED → COMPLETED       ❌

CONFIRMED → CREATED       ❌

COMPLETED → CANCELLED     ❌

CANCELLED → PROCESSING    ❌
```

Invalid transitions must be rejected by the domain.

---

## 7. Order Modification Rules

Orders can only be modified while their current state permits modification.

### CREATED

Items may be added or modified.

### CONFIRMED

The order has been confirmed and its items must no longer be modified.

### PROCESSING

The order is being processed and its items must no longer be modified.

### COMPLETED

The order is final.

The following operations are prohibited:

* Adding items.
* Modifying items.
* Cancelling the order.
* Changing status.

### CANCELLED

The order is final.

The following operations are prohibited:

* Adding items.
* Modifying items.
* Reactivating the order.
* Changing status.

---

## 8. Domain Invariants

A valid `Order` must always satisfy the following invariants:

1. The order identifier must not be null after creation.
2. The order must contain at least one valid item.
3. Every item must have a quantity greater than zero.
4. Every item must have a unit price greater than zero.
5. The total must be derived from the order items.
6. Status transitions must follow the defined lifecycle.
7. Completed orders cannot be modified.
8. Cancelled orders cannot be modified.

These rules belong to the domain and must not depend on controllers, database constraints, or infrastructure validation.

---

## 9. Domain Behavior

Instead of exposing its internal state for unrestricted modification, the domain should expose operations that represent business behavior.

Conceptually:

```text
Order
 │
 ├── addItem(...)
 ├── calculateTotal()
 ├── confirm()
 ├── startProcessing()
 ├── complete()
 └── cancel()
```

This allows the `Order` object itself to protect its business rules.

For example:

```text
order.cancel()
```

is preferred over unrestricted state modification such as:

```text
order.status = CANCELLED
```

The first approach allows the domain to verify whether cancellation is permitted before changing its state.

---

## 10. Domain Independence

Domain objects must not depend directly on infrastructure annotations or components.

The domain must not require:

```java
@Entity
@Table
@Column
@Repository
@Service
@RestController
```

The conceptual dependency remains:

```text
                DOMAIN
                   ▲
                   │
        ┌──────────┴──────────┐
        │                     │
   Application            Adapters
```

Infrastructure may depend on and translate domain objects.

The domain must not depend on infrastructure.

---

## 11. Persistence Representation

The persistence adapter will use separate persistence models.

Conceptually:

```text
DOMAIN                          PERSISTENCE

Order                           OrderEntity
  │                                  │
  │                                  │
OrderItem                      OrderItemEntity

        ◄────── Mapper ──────►
```

This allows JPA-specific concerns to remain outside the domain.

For example:

```text
Order
```

represents business behavior, while:

```text
OrderEntity
```

represents how the order is stored in PostgreSQL.

---

## 12. Domain Principle

The domain is responsible for maintaining its own consistency.

> A domain object should not depend on external layers to prevent it from entering an invalid business state.

Controllers validate HTTP input.

Use cases coordinate application operations.

Adapters communicate with external technologies.

The domain protects the business rules.
