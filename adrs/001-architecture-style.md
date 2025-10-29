# Adoption of a Modular Monolithic Architecture for the Complaint Management System (CMS)

## Context and Problem Statement

The Complaint Management System (CMS) needs a software architecture to support multiple large clients (e.g., banks, telecoms) where each client will have its own deployment.
The architecture must allow:
- Simpler deployment and maintenance per company.
- Scalable performance (handling millions of users).
- Future extensibility (e.g., adding chatbot integration or decomposing services if needed).

The question is: **What architectural style should be used to balance simplicity, maintainability, scalability, and future flexibility?**

## Decision Drivers

* Each client has its **own isolated deployment** — so ease of deployment and maintenance are key.
* **Incremental scalability** is sufficient (only ~10% annual user growth).
* **Development speed and simplicity** are more critical at this stage than distributed scalability.
* The system should be **modular and cleanly separated** by feature/domain to avoid a “big ball of mud.”
* **Future flexibility** to evolve into a distributed or service-oriented architecture if required.

## Considered Options

* Modular Monolith
* Service-Oriented Architecture (SOA)
* Microservices Architecture

## Decision Outcome

**Chosen option:** *Modular Monolith*
because it best balances simplicity, maintainability, and future flexibility. It allows feature-based modularisation within a single deployable unit, making deployments per client straightforward while still supporting internal separation of concerns.

### Consequences

* **Good, because:**
  - Single deployment per company simplifies maintenance, hosting, and version control.
  - Modular code structure supports clean domain boundaries (e.g., Auth, Complaints, Reporting, Admin).
  - Easier to test and debug since all logic runs in one process.
  - Allows future **modularization into services** if needed — boundaries can be externalized later.
  - Lower infrastructure and operational overhead compared to SOA/microservices.

* **Bad, because:**
  - Scaling requires scaling the whole application, not individual modules.
  - Failures in one module could affect others (shared process space).
  - Limited flexibility in technology choices per module (all share the same stack).

### Confirmation

Compliance with this ADR will be confirmed through:
- **Code and architecture reviews** ensuring each domain module (e.g., Auth, Complaints) has defined interfaces and minimal cross-dependencies.
- Enforcing modular boundaries in the backend structure (e.g., via NestJS modules or equivalent).
- Future decomposition feasibility can be verified by checking whether modules can be run independently without major coupling.

## Pros and Cons of the Options

### Modular Monolith

A single application with clear internal modular boundaries separating features/domains.

* Good, because it provides **simpler deployment and maintenance** for each company instance.
* Good, because it **supports modularity and future separation** if scaling demands increase.
* Good, because it **reduces complexity** compared to distributed architectures.
* Neutral, because it scales vertically rather than horizontally per service.
* Bad, because a **failure in one module affects all** modules in the same process.

---

### Service-Oriented Architecture (SOA)

Multiple deployable services communicating over network APIs.

* Good, because it allows **scaling specific services independently**.
* Good, because it provides **fault isolation** and potential for parallel team ownership.
* Bad, because it **increases deployment complexity** (network management, API gateways, orchestration).
* Bad, because it’s **overkill for a single-tenant-per-deployment system**.
* Bad, because **increased latency and debugging complexity** are introduced with distributed communication.

---

### Microservices Architecture

Highly decoupled, fine-grained services each owning their own data and APIs.

* Good, because it supports **massive scalability and team autonomy**.
* Good, because it provides **strong fault isolation**.
* Bad, because it adds **significant complexity** in communication, monitoring, and orchestration.
* Bad, because **deployment overhead** is unjustified for this project’s scale and growth expectations.
* Bad, because it **increases operational cost and complexity** without clear short-term benefit.
