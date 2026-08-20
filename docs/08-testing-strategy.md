# Testing Strategy

## 1. Overview

The testing strategy follows the boundaries established by Hexagonal Architecture.

The primary objective is to verify business behavior independently from infrastructure whenever possible.

The project will use different test levels according to the responsibility being validated.

```text id="08wk1q"
              ┌────────────────────┐
              │   End-to-End /     │
              │ Integration Tests  │
              └─────────▲──────────┘
                        │
              ┌─────────┴──────────┐
              │   Adapter Tests    │
              └─────────▲──────────┘
                        │
              ┌─────────┴──────────┐
              │  Use Case Tests    │
              └─────────▲──────────┘
                        │
              ┌─────────┴──────────┐
              │   Domain Tests     │
              └────────────────────┘
```

The majority of business behavior should be validated through fast unit tests.

---

## 2. Testing Principles

The project follows these principles:

* Business rules must be testable without Spring.
* Use cases must be testable without PostgreSQL.
* Domain tests must not require Docker.
* Infrastructure behavior should be tested separately from business behavior.
* External dependencies should be replaced by mocks or test implementations when testing the application core.
* Integration tests should verify infrastructure boundaries.

---

## 3. Domain Unit Tests

Domain tests validate business rules directly.

These tests must not require:

```text id="05e3bc"
Spring Context
Database
HTTP Server
Docker
JPA
```

Example:

```text id="98cchv"
OrderTest

✓ shouldCalculateOrderTotal
✓ shouldRejectItemWithInvalidQuantity
✓ shouldRejectItemWithInvalidPrice
✓ shouldConfirmCreatedOrder
✓ shouldRejectInvalidStatusTransition
✓ shouldCancelCreatedOrder
✓ shouldRejectCancellationOfCompletedOrder
✓ shouldRejectModificationOfCancelledOrder
```

Conceptually:

```java id="vd4zge"
@Test
void shouldRejectCancellationOfCompletedOrder() {

    Order order = createCompletedOrder();

    assertThrows(
        InvalidOrderStatusException.class,
        order::cancel
    );
}
```

This test executes only business behavior.

---

## 4. OrderItem Unit Tests

`OrderItem` business behavior will also be tested independently.

Examples:

```text id="pc0q2y"
OrderItemTest

✓ shouldCalculateSubtotal
✓ shouldRejectZeroQuantity
✓ shouldRejectNegativeQuantity
✓ shouldRejectZeroPrice
✓ shouldRejectNegativePrice
✓ shouldRejectBlankProductName
```

These tests ensure invalid items cannot enter the domain.

---

## 5. Use Case Unit Tests

Use-case tests validate application orchestration.

Output ports will be mocked using Mockito.

Example:

```text id="5id4la"
CreateOrderServiceTest

CreateOrderService
       │
       ▼
OrderRepositoryPort
       ▲
       │
     MOCK
```

The test does not require PostgreSQL.

Conceptually:

```java id="rry0lu"
OrderRepositoryPort repository =
        mock(OrderRepositoryPort.class);

CreateOrderService service =
        new CreateOrderService(repository);
```

The test can verify:

* Domain creation.
* Repository interaction.
* Returned result.
* Error propagation.
* Correct orchestration.

---

## 6. Use Case Test Scenarios

### CreateOrderService

```text id="gzrd8s"
✓ shouldCreateOrder
✓ shouldPersistCreatedOrder
✓ shouldRejectInvalidOrder
```

### GetOrderService

```text id="5n8mdu"
✓ shouldReturnExistingOrder
✓ shouldThrowWhenOrderDoesNotExist
```

### AddOrderItemService

```text id="hhzyxj"
✓ shouldAddItemToEditableOrder
✓ shouldPersistUpdatedOrder
✓ shouldRejectModificationOfNonEditableOrder
```

### UpdateOrderStatusService

```text id="8ek1as"
✓ shouldUpdateValidOrderStatus
✓ shouldRejectInvalidTransition
```

### CancelOrderService

```text id="9gq9t6"
✓ shouldCancelOrder
✓ shouldRejectCancellationOfCompletedOrder
✓ shouldThrowWhenOrderDoesNotExist
```

---

## 7. Persistence Adapter Tests

Persistence adapter tests validate the integration between:

```text id="klz8dd"
OrderPersistenceAdapter
        │
        ▼
Spring Data JPA
        │
        ▼
PostgreSQL
```

These tests may require Spring and a real database environment.

Testcontainers will be used to provide PostgreSQL during integration tests.

---

## 8. Testcontainers

Testcontainers will allow integration tests to execute against a real PostgreSQL instance running inside a temporary Docker container.

Conceptually:

```text id="r8kyba"
Integration Test
       │
       ▼
Persistence Adapter
       │
       ▼
PostgreSQL Container
       │
       ▼
Temporary Test Database
```

The container exists only for the test lifecycle.

This avoids relying exclusively on an in-memory database with behavior that may differ from production PostgreSQL.

---

## 9. Persistence Test Scenarios

Examples include:

```text id="0p2otv"
OrderPersistenceAdapterTest

✓ shouldPersistOrder
✓ shouldPersistOrderItems
✓ shouldFindOrderById
✓ shouldReturnEmptyWhenOrderDoesNotExist
✓ shouldListOrders
✓ shouldRestoreDomainObjectFromPersistence
```

These tests also validate domain/entity mapping.

---

## 10. REST Adapter Tests

REST adapter tests validate HTTP-specific behavior.

Responsibilities include:

* Endpoint mapping.
* Request deserialization.
* Input validation.
* HTTP status codes.
* Response serialization.
* Exception mapping.

Examples:

```text id="x32n8u"
OrderControllerTest

✓ shouldReturn201WhenOrderIsCreated
✓ shouldReturn200WhenOrderExists
✓ shouldReturn404WhenOrderDoesNotExist
✓ shouldReturn400ForInvalidRequest
✓ shouldReturnConflictForInvalidBusinessOperation
```

The controller tests should focus on HTTP behavior rather than retesting domain rules.

---

## 11. Mapper Tests

Mappings between architectural boundaries may be tested independently when they contain meaningful transformation logic.

Examples include:

```text id="ctq2dq"
OrderPersistenceMapperTest

Order → OrderEntity
OrderEntity → Order
```

and:

```text id="uzx3eu"
OrderWebMapperTest

CreateOrderRequest → CreateOrderCommand
Order → OrderResponse
```

The goal is to ensure data is correctly translated between representations.

---

## 12. Test Isolation

Each test level should validate its own responsibility.

For example:

```text id="lx5u17"
Domain Test
    │
    └── Business rules

Use Case Test
    │
    └── Application orchestration

Adapter Test
    │
    └── Technology integration

REST Test
    │
    └── HTTP contract
```

A controller test should not be the primary test for an order status business rule.

A domain test should not require PostgreSQL to verify cancellation behavior.

---

## 13. Test Pyramid

The project aims to maintain a test distribution similar to:

```text id="j6ox9z"
                 /\
                /  \
               / E2E\
              /------\
             /Adapter \
            /----------\
           / Use Cases  \
          /--------------\
         / Domain Tests   \
        /__________________\
```

The lower levels should contain more tests because they are:

* Faster.
* Easier to isolate.
* Easier to maintain.
* More precise when identifying failures.

---

## 14. Testing the Dependency Inversion

Testing also demonstrates the architectural benefit of output ports.

Production:

```text id="wajspq"
CreateOrderService
       │
       ▼
OrderRepositoryPort
       ▲
       │
OrderPersistenceAdapter
       │
       ▼
PostgreSQL
```

Unit test:

```text id="9pj6o2"
CreateOrderService
       │
       ▼
OrderRepositoryPort
       ▲
       │
     Mockito
```

The use case does not change.

Only the implementation behind the port changes.

This is one of the main practical benefits of dependency inversion.

---

## 15. Tools

The initial testing stack will include:

* **JUnit 5** — test framework.
* **Mockito** — mocking output ports and collaborators.
* **Spring Boot Test** — Spring integration testing.
* **MockMvc** — REST adapter testing.
* **Testcontainers** — PostgreSQL integration testing.

---

## 16. Continuous Integration

The test suite should be suitable for execution in a CI/CD pipeline.

The expected pipeline will eventually include:

```text id="pfr9yh"
Build
  │
  ▼
Unit Tests
  │
  ▼
Integration Tests
  │
  ▼
Package
```

A future GitHub Actions workflow may automate these steps.

---

## 17. Testing Principle

The central testing principle is:

> **Business behavior should be testable without requiring infrastructure.**

Infrastructure tests verify that adapters correctly connect external technologies to the application boundaries.

Domain and use-case tests verify that the application behaves correctly regardless of those technologies.
