# Ports and Adapters

## 1. Overview

In Hexagonal Architecture, the application core communicates with the outside world through **ports**.

Ports define contracts.

Adapters connect those contracts to concrete technologies such as:

* REST
* PostgreSQL
* JPA
* External APIs
* Messaging systems

The application core must not depend directly on these technologies.

The general flow is:

```text
External Actor
     │
     ▼
Input Adapter
     │
     ▼
Input Port
     │
     ▼
Application Core
     │
     ▼
Output Port
     │
     ▼
Output Adapter
     │
     ▼
External System
```

---

## 2. Ports

A port is an abstraction that defines how the application interacts with external components.

Ports are divided into:

* Input Ports
* Output Ports

They belong to the application boundary.

---

## 3. Input Ports

Input ports define operations exposed by the application.

They represent what external actors are allowed to ask the application to do.

Examples:

```text
CreateOrderUseCase
GetOrderUseCase
ListOrdersUseCase
AddOrderItemUseCase
UpdateOrderStatusUseCase
CancelOrderUseCase
```

Conceptually:

```java
public interface CreateOrderUseCase {

    Order create(CreateOrderCommand command);

}
```

The interface defines the operation.

It does not define how HTTP works or how the request reaches the application.

---

## 4. Input Adapter

The initial input adapter is the REST API.

Its responsibility is to translate HTTP communication into application calls.

Example:

```text
HTTP Request
     │
     ▼
OrderController
     │
     ▼
CreateOrderUseCase
```

The controller may receive:

```http
POST /orders
```

with a JSON body.

The controller converts that HTTP request into an application command.

Conceptually:

```text
JSON Request
     │
     ▼
CreateOrderRequest
     │
     ▼
CreateOrderCommand
     │
     ▼
CreateOrderUseCase
```

The controller must not contain core business rules.

---

## 5. Output Ports

Output ports define capabilities that the application requires from the outside world.

The application declares what it needs without knowing which technology will provide the implementation.

For the initial version, the primary output port will be related to order persistence.

Conceptually:

```java
public interface OrderRepositoryPort {

    Order save(Order order);

    Optional<Order> findById(UUID id);

    List<Order> findAll();

}
```

The application only knows this contract.

It does not know if the implementation uses:

* PostgreSQL
* MongoDB
* An external API
* An in-memory structure
* Another persistence mechanism

---

## 6. Output Adapter

The output adapter implements an output port using a concrete external technology.

The initial implementation will use PostgreSQL with Spring Data JPA.

Conceptually:

```text
OrderRepositoryPort
        ▲
        │
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

The persistence adapter is responsible for infrastructure-specific concerns.

---

## 7. Persistence Mapping

The application domain and persistence models remain separated.

The persistence adapter converts between them.

```text
Domain                         Persistence

Order                         OrderEntity
  │                               │
  │                               │
  └──────► OrderMapper ◄──────────┘
```

When saving:

```text
Order
  │
  ▼
OrderMapper
  │
  ▼
OrderEntity
  │
  ▼
JPA
  │
  ▼
PostgreSQL
```

When reading:

```text
PostgreSQL
  │
  ▼
JPA
  │
  ▼
OrderEntity
  │
  ▼
OrderMapper
  │
  ▼
Order
```

This prevents JPA-specific details from leaking into the domain.

---

## 8. Initial Input Ports

The project will initially define the following input ports.

### CreateOrderUseCase

Responsible for creating a new order.

```text
CreateOrderCommand
        │
        ▼
CreateOrderUseCase
        │
        ▼
Order
```

---

### GetOrderUseCase

Responsible for retrieving an order by its identifier.

```text
Order ID
   │
   ▼
GetOrderUseCase
   │
   ▼
Order
```

---

### ListOrdersUseCase

Responsible for retrieving registered orders.

```text
ListOrdersUseCase
       │
       ▼
List<Order>
```

---

### AddOrderItemUseCase

Responsible for adding an item to an editable order.

```text
Order ID + Item Data
          │
          ▼
AddOrderItemUseCase
          │
          ▼
Updated Order
```

---

### UpdateOrderStatusUseCase

Responsible for executing valid order status transitions.

```text
Order ID + Target Status
          │
          ▼
UpdateOrderStatusUseCase
          │
          ▼
Updated Order
```

---

### CancelOrderUseCase

Responsible for cancelling an order according to domain rules.

```text
Order ID
   │
   ▼
CancelOrderUseCase
   │
   ▼
Cancelled Order
```

---

## 9. Initial Output Ports

The initial application requires persistence capabilities.

### OrderRepositoryPort

The persistence port will initially provide operations such as:

```text
save(order)

findById(id)

findAll()
```

The exact implementation remains outside the application core.

---

## 10. Future Output Ports

Additional ports may be introduced when the application requires new external capabilities.

Examples include:

```text
PaymentPort
InventoryPort
NotificationPort
EventPublisherPort
CustomerPort
```

These ports should only be created when the application actually needs to communicate with an external capability.

Ports must not be created automatically for every domain entity.

---

## 11. Adapter Examples

Possible adapters include:

| Port                | Adapter                    | Technology           |
| ------------------- | -------------------------- | -------------------- |
| CreateOrderUseCase  | OrderController            | Spring Web           |
| GetOrderUseCase     | OrderController            | Spring Web           |
| OrderRepositoryPort | OrderPersistenceAdapter    | Spring Data JPA      |
| PaymentPort         | PaymentGatewayAdapter      | External payment API |
| EventPublisherPort  | KafkaEventPublisherAdapter | Kafka                |

The port represents the contract.

The adapter represents the concrete implementation or translation.

---

## 12. Dependency Direction

The application core owns the contracts.

Infrastructure implements them.

```text
             APPLICATION CORE

         ┌─────────────────────┐
         │   Input Ports       │
         │   Use Cases         │
         │   Domain            │
         │   Output Ports      │
         └──────────▲──────────┘
                    │
                    │ depends on core
                    │
         ┌──────────┴──────────┐
         │      Adapters       │
         └─────────────────────┘
```

The core must not import concrete adapter implementations.

---

## 13. Input vs Output Perspective

The terms input and output are defined from the perspective of the application core.

### Input

Something wants to invoke the application.

```text
User
 │
 ▼
REST
 │
 ▼
Input Adapter
 │
 ▼
Input Port
 │
 ▼
CORE
```

### Output

The application needs something from outside.

```text
CORE
 │
 ▼
Output Port
 │
 ▼
Output Adapter
 │
 ▼
Database / External Service
```

---

## 14. Example: Create Order

A complete create-order interaction can be represented as:

```text
Client
  │
  ▼
POST /orders
  │
  ▼
OrderController
  │
  ▼
CreateOrderUseCase
  │
  ▼
CreateOrderService
  │
  ▼
Order Domain
  │
  ▼
OrderRepositoryPort
  │
  ▼
OrderPersistenceAdapter
  │
  ▼
Spring Data JPA
  │
  ▼
PostgreSQL
```

The application core handles the business operation.

The adapters handle external communication.

---

## 15. Replaceability

One of the main goals of ports and adapters is replaceability.

For example, the application may initially use:

```text
OrderRepositoryPort
       │
       ▼
JPA Adapter
       │
       ▼
PostgreSQL
```

A future implementation could use:

```text
OrderRepositoryPort
       │
       ▼
Mongo Adapter
       │
       ▼
MongoDB
```

The use cases and domain do not need to change because they depend on the port instead of the concrete persistence technology.

---

## 16. Port Creation Principle

A port should exist because the application has an interaction boundary.

It should not exist simply because a domain entity exists.

For example:

```text
Order
OrderItem
OrderStatus
```

do not automatically require:

```text
OrderPort
OrderItemPort
OrderStatusPort
```

Ports are defined according to application interactions and external dependencies.

---

## 17. Architectural Principle

The relationship between ports and adapters can be summarized as:

> **Ports describe what the application can do or what it needs. Adapters describe how the external world communicates with those ports.**

The application core controls the contracts.

External technologies remain replaceable implementation details.
