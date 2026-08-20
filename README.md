# Hexagonal Architecture API

REST API for order management built with **Java and Spring Boot**, structured according to **Hexagonal Architecture (Ports & Adapters)**.

The project focuses on keeping business rules independent from frameworks, databases, and external technologies by applying dependency inversion and clearly separating the application core from infrastructure concerns.

## Architecture

The application follows **Hexagonal Architecture**, organizing the system around the domain and its use cases.

The core application defines the contracts required to interact with external components, while adapters provide their concrete implementations.

```text
                    ┌─────────────────────┐
                    │     REST API        │
                    │   Input Adapter     │
                    └──────────┬──────────┘
                               │
                         Input Ports
                               │
                    ┌──────────▼──────────┐
                    │     Use Cases       │
                    │    Application      │
                    └──────────┬──────────┘
                               │
                         Domain Model
                               │
                         Output Ports
                               │
                    ┌──────────▼──────────┐
                    │ Persistence Adapter │
                    │    PostgreSQL       │
                    └─────────────────────┘
```

This approach keeps the business logic isolated from infrastructure details such as HTTP, persistence frameworks, and databases.

## Main Concepts

* Hexagonal Architecture
* Ports & Adapters
* Dependency Inversion
* Domain isolation
* Use Cases
* SOLID principles
* Separation of concerns
* Testability

## Technologies

* Java 21
* Spring Boot 3
* Spring Web
* Spring Data JPA
* PostgreSQL
* Flyway
* Bean Validation
* OpenAPI / Swagger
* Maven
* Docker
* Docker Compose
* JUnit 5
* Mockito
* Testcontainers

## Domain

The application manages the lifecycle of customer orders.

The main business capabilities include:

* Creating orders
* Adding items to an order
* Calculating order totals
* Managing order status
* Cancelling orders
* Retrieving order information

Business rules are implemented inside the application core without direct dependencies on Spring, JPA, PostgreSQL, or HTTP.

## Project Structure

The project is organized around the application core and its adapters.

```text
src/main/java
│
├── domain
│   ├── model
│   ├── exception
│   └── service
│
├── application
│   ├── port
│   │   ├── in
│   │   └── out
│   └── usecase
│
├── adapter
│   ├── in
│   │   └── web
│   └── out
│       └── persistence
│
└── config
```

The dependency direction always points toward the application core.

External technologies can be replaced without changing the core business rules.

## Documentation

Detailed technical documentation is available in the [`docs`](./docs) directory.

The documentation covers:

* Project requirements
* Hexagonal Architecture
* Domain modeling
* Ports and adapters
* Application flows
* Architecture decisions
* Testing strategy

## Status

🚧 Project under active development.
