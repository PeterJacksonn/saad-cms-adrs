# Adoption of FastAPI for the Backend Framework

## Context and Problem Statement

The Complaint Management System (CMS) requires a backend framework to handle user complaints, authentication, and administrative functions via RESTful APIs.
The backend must be efficient, easy to implement, and maintainable for a small-scale proof of concept while remaining scalable for future growth.
The question is: **Which backend framework should be used to build a lightweight, performant, and easy-to-develop API service for the CMS?**

## Decision Drivers

* Ease of development and learning curve given limited time for delivery
* Developer familiarity and prior experience
* Performance and scalability
* Built-in support for modern API standards (e.g., OpenAPI, JSON)
* Good documentation and active community support

## Considered Options

* Flask (Python)
* FastAPI (Python)
* Express.js (Node.js)
* Spring Boot (Java)
* Gin (Go)

## Decision Outcome

Chosen option: **FastAPI**, because it provides a clean, modern, and efficient framework for building RESTful APIs with minimal boilerplate, strong typing via Pydantic, and automatic API documentation generation.
It offers a significant step up in features and performance compared to **Flask**, while maintaining the development speed edge over **Spring Boot** and **Gin**. It also aligns with existing developer experience in Python, allowing faster and more reliable implementation compared to alternatives that require new language or ecosystem learning.

### Consequences

* Good, because it enables **rapid API development** with built-in validation and documentation.
* Good, because **Python familiarity** reduces learning time and development risk.
* Good, because **asynchronous support** allows for efficient handling of concurrent requests.
* Bad, because **Python’s performance** may be slightly lower compared to Node.js or Java for highly concurrent workloads.
* Bad, because **fewer enterprise-scale libraries** are available compared to Spring Boot.

### Confirmation

Compliance with this ADR can be confirmed by:
- Reviewing that API endpoints are implemented using FastAPI with proper Pydantic models and route definitions.
- Verifying automatic OpenAPI documentation is generated at runtime (via `/docs` endpoint).
- Confirming backend integration and endpoint testing in Postman or similar tools.

## Pros and Cons of the Options

### Flask (Python)

A micro-framework in Python, offering a simple core with minimal dependencies, requiring extensive use of extensions.

* Good, because it is **extremely lightweight** and minimal.
* Good, because it has **broad Python community support** and is well-documented.
* Neutral, because it is **highly flexible**, but this requires more architectural discipline from the team.
* Bad, because it **lacks modern features** like built-in input validation (requires extensions).
* Bad, because it **does not natively support async/await** for high-performance concurrent I/O.

---

### FastAPI (Python)

Modern Python web framework designed for fast, type-safe, and developer-friendly API creation.

* Good, because it’s **easy to learn** and matches existing developer experience.
* Good, because it **automatically generates documentation** (Swagger/OpenAPI).
* Good, because it supports **async I/O** for concurrent requests.
* Neutral, because while performant, it’s **not as fast as compiled languages**.
* Bad, because it has **fewer built-in enterprise integrations** than Spring Boot.

### Express.js (Node.js)

Minimalist JavaScript web framework widely used for building APIs.

* Good, because it’s **lightweight and flexible**.
* Good, because it has a **large ecosystem** of middleware and libraries.
* Neutral, because it requires **learning JavaScript on the backend**, which may slow progress.
* Bad, because **less structured by default**, increasing risk of inconsistent architecture.

### Spring Boot (Java)

Enterprise-grade framework for building robust and scalable web services.

* Good, because it provides **enterprise-level tools and structure**.
* Good, because it’s **highly performant and reliable** for large systems.
* Neutral, because it offers **strong typing and maturity**, but may be overkill for a proof of concept.
* Bad, because it has a **steeper learning curve** and **higher setup complexity**.

---

### Gin (Go)

A high-performance HTTP web framework written in Go, which is a statically typed, compiled language.

* Good, because it offers **exceptional raw performance and low latency** (best performance of all options).
* Good, because it compiles to a **single binary**, making deployment simple and efficient.
* Bad, because it requires **learning Go**, which drastically increases development time for a team unfamiliar with the language.
* Bad, because **boilerplate for simple CRUD operations** is higher than in Python frameworks.
* Bad, because the **ecosystem is smaller** than Python/Java/Node.js for niche libraries.
