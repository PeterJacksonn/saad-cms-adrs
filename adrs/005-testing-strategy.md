# Adoption of the Testing Pyramid Strategy for CMS

## Context and Problem Statement

The Complaint Management System (CMS) needs a comprehensive testing strategy to ensure reliability, performance, and compliance with strict user expectations (e.g., 24/7 availability, WCAG 2 accessibility). The strategy must cover the entire stack (**React Frontend**, **FastAPI Backend**, **PostgreSQL DB**) and be efficient enough to support rapid development cycles for the Proof-of-Concept (PoC) while building a robust foundation for future scale.

The question is: **What structured testing strategy should be adopted to balance manual testing requirements (use cases, accessibility) with efficient, automated quality assurance across the full stack?**

## Decision Drivers

* **Reliability & Data Integrity:** Ensuring core business logic (complaint lifecycle) and data consistency are guaranteed.
* **Speed of Feedback:** Tests must run quickly to support continuous integration and rapid development.
* **Coverage:** Must cover all layers: UI, API/Logic, and Database.
* **Performance:** Critical to confirm the system can handle the required scale (20M+ users).
* **Usability & Accessibility:** Manual validation is necessary for adherence to standards like WCAG 2.

## Considered Options

* Testing Pyramid Strategy (Automated Focus)
* Ice Cream Cone Strategy (Manual/UI Focus)
* Pure Manual End-to-End Testing
* Focus on Unit Testing Only
* Load/Stress Testing Only

## Decision Outcome

Chosen option: **Testing Pyramid Strategy**, because it provides a structured, automated approach that prioritizes fast, cheap tests at the foundation (**Unit Tests**) and applies more targeted, slower tests (**Integration, E2E**) where they provide the most value. This balances the need for robust automated checks with necessary manual testing for use cases and compliance, while building a foundation that supports future investment requests.

### Consequences

* **Good, because:**
  * **Fast Feedback Loop:** Developers get immediate feedback from Unit and Integration tests.
  * **Low Maintenance Cost:** Defects are caught early in smaller, easier-to-fix components.
  * **Clear Prioritization:** Effort is focused on the most critical and frequently changing components (the business logic).
  * **Foundation for CI/CD:** Provides the necessary automated gates for continuous deployment.

* **Bad, because:**
  * **Requires Discipline:** Developers must commit to writing detailed Unit and Integration tests, increasing initial coding time.
  * **Limited Real-World Coverage:** Does not *guarantee* that deployment and networking issues are caught (requires Infrastructure Testing, which is secondary).
  * **Manual Effort Remains:** Accessibility and complex use-case testing still require time-consuming manual effort.

### Confirmation

Compliance with this ADR will be confirmed through:
* **Code Coverage Metrics:** Achieving a minimum target (e.g., 80%) for unit test coverage on the FastAPI backend logic.
* **Pipeline Success:** Successful execution of the full automated test suite (Unit, Integration, E2E) in the Continuous Integration (CI) environment before deployment.
* **Manual Sign-off:** Formal sign-off by a quality assurance representative after manual testing confirms key use cases and WCAG 2 compliance.

## Pros and Cons of the Options

### Testing Pyramid Strategy (Automated Focus)

A balanced approach where Unit Tests form the base (high volume, fast), Integration Tests sit in the middle, and UI/E2E Tests are at the top (low volume, slow).

[Image of the Testing Pyramid structure with Unit tests at the bottom, Integration tests in the middle, and E2E tests at the top]


* Good, because it provides the **best balance of confidence, speed, and cost** for modern application development.
* Good, because it ensures **critical business logic (FastAPI services)** is thoroughly validated with fast Unit Tests.
* Neutral, because it requires **three different testing tools** (e.g., Python's `pytest`, React's `Testing Library`, Playwright/Cypress for E2E).
* Bad, because E2E tests can be **flaky and slow** if not managed carefully.

---

### Ice Cream Cone Strategy (Manual/UI Focus)

The inverse of the Pyramid, where the majority of effort is placed on manual and End-to-End (E2E) UI testing, with minimal Unit Tests.

* Good, because it provides **high confidence** that the user's view of the entire system works.
* Neutral, because it is **easy to explain** to non-technical stakeholders.
* Bad, because tests are **extremely slow to execute**, blocking the CI/CD pipeline.
* Bad, because it is **very costly** to maintain, as small backend changes frequently break large E2E tests.

---

### Pure Manual End-to-End Testing

Relying entirely on QA engineers or developers manually clicking through use cases.

* Good, because it is the **only way to fully confirm WCAG 2 Accessibility** and complex human-driven use cases.
* Neutral, because it requires **no initial coding effort** for the testing framework.
* Bad, because it is **slow, error-prone, and non-repeatable**, making regression impossible without a large, dedicated team.
* Bad, because it **cannot be run consistently** in a CI/CD pipeline, guaranteeing late-stage defect discovery.

---

### Focus on Unit Testing Only

Automating only the smallest components (functions, classes) of the FastAPI backend and React components, ignoring how they interact.

* Good, because it provides the **fastest possible test suite** execution.
* Good, because Unit tests are the **easiest to write and maintain**.
* Neutral, because it ensures individual modules (e.g., data validation logic) are perfect.
* Bad, because it **provides zero confidence** that the services, database, or API endpoints work together correctly.
* Bad, because failures often occur **at the seams** (API boundaries, database connection) which are completely untestable with this approach.

---

### Load/Stress Testing Only

Focusing solely on tools to simulate high volumes of concurrent users to test performance and scalability.

* Good, because it is **critical for validating the 20M+ user scalability requirement**.
* Good, because it identifies resource bottlenecks (e.g., slow PostgreSQL queries or FastAPI worker capacity).
* Neutral, because performance data is useful for optimization.
* Bad, because it **does not verify correctness**—a system can be fast but wrong.
* Bad, because Load testing tools are often **complex** (e.g., JMeter, Locust) and require specialized setup time, diverting effort from functional development.
