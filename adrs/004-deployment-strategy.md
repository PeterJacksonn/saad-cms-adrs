# Adoption of Containerized Single-Tenant Deployment using Docker Compose

## Context and Problem Statement

The CMS architecture requires a **Modular Monolith** backend and a **dedicated PostgreSQL database** for each client (tenant) to ensure absolute data isolation. The system must be readily extensible to new regions and quickly provisioned for venture capitalists (VCs) and new client onboarding. The development process needs a simple, portable method to run the entire stack consistently across developer machines, testing environments, and eventual production servers.

The question is: **What deployment strategy allows for repeatable, isolated provisioning of the full CMS stack (backend, database, frontend) for every client instance while maximizing portability and minimizing setup overhead?**

## Decision Drivers

* **Absolute Tenant Isolation:** Each client must have their own secured, dedicated database and application process.
* **Repeatable Provisioning:** The entire application stack must be deployable with a single, simple command for hundreds of client instances.
* **Portability:** The deployment process must work consistently across various environments (local development, staging, production Linux servers).
* **Speed to Market:** The process must support rapid creation and deployment of the Proof-of-Concept (PoC) for VCs.
* **Alignment with Technology Stack:** Must seamlessly support FastAPI (Python) and PostgreSQL.

## Considered Options

* Containerization (Docker Compose)
* Virtual Machines (VMs)
* Bare-Metal Installation
* Managed Kubernetes (e.g., EKS/GKE)

## Decision Outcome

Chosen option: **Containerization (Docker Compose)**, because it directly addresses the core need for **repeatable, isolated, single-command provisioning** of the entire stack per client. It maximizes portability from the local development environment to the production host, making it the fastest and most reliable way to execute the single-tenant deployment model.

### Consequences

* **Good, because:**
  * **Simplified Onboarding:** A new client instance is created by simply running a script that executes `docker compose up -d` with unique variables.
  * **Environment Consistency:** Eliminates "works on my machine" issues by packaging all dependencies (Python, PostgreSQL) into containers.
  * **Resource Efficiency:** Containers are significantly lighter than full VMs, improving host machine resource usage.
  * **Scalability Path:** Docker containers are the fundamental building blocks for future migration to Kubernetes (Managed Kubernetes option) if orchestrated scaling becomes necessary.

* **Bad, because:**
  * **Operational Complexity:** Managing and monitoring hundreds of individual Docker Compose files or instances will become challenging as the client base grows past the initial PoC phase.
  * **Limited Horizontal Scaling:** Docker Compose is ideal for single-machine deployment; true elastic, auto-scaling requires migration to a full orchestrator (like Kubernetes).

### Confirmation

Compliance with this ADR will be confirmed through:
- Successful **multi-instance deployment** of the full stack (FastAPI, PostgreSQL) using the provided `docker-compose.yml` file, ensuring no resource conflicts.
- Verification that all application secrets (like database credentials) are properly externalized via `.env` files or similar host mechanisms for each instance.
- Establishing a simple scripting mechanism to **onboard** and **tear down** a client instance using the Docker Compose file structure.

## Pros and Cons of the Options

### Containerization (Docker Compose)

The use of Docker and Docker Compose to define the application, database, and all services as portable, isolated images and run them as a cohesive unit.

* Good, because it provides **extreme deployment consistency** from development to production.
* Good, because it is the **fastest path to provisioning** an isolated, full-stack environment for a new client.
* Neutral, because it requires **intermediate DevOps skill** to set up the initial Dockerfiles and Compose configuration.
* Bad, because it is **not designed for self-healing or zero-downtime rolling updates** required in mature enterprise environments (a limitation handled by Kubernetes).

---

### Virtual Machines (VMs)

Deploying the entire CMS stack (OS, PostgreSQL, FastAPI, Web Server) onto a single, heavy Virtual Machine instance per client.

* Good, because it provides **complete OS-level isolation** between clients.
* Neutral, because **existing operational knowledge** (like Chef or Ansible) can be leveraged for provisioning.
* Bad, because VMs are **resource-intensive** (high CPU/RAM overhead) compared to containers.
* Bad, because **provisioning time is slow** (minutes vs. seconds) and maintaining OS patches across hundreds of VMs is difficult.

---

### Bare-Metal Installation

Installing the operating system, language runtime (Python), database (PostgreSQL), and application files directly onto the host server for each client.

* Good, because it achieves **maximum raw performance** with no virtualization overhead.
* Neutral, because it allows **full low-level control** over system resources.
* Bad, because it is **impossible to maintain consistency** across environments (local vs. staging vs. production).
* Bad, because **dependency conflicts** (e.g., Python library versions) are inevitable, leading to high maintenance costs.

---

### Managed Kubernetes (e.g., EKS/GKE)

Using an orchestration system to manage the deployment, scaling, and networking of containers across a cluster of machines.

* Good, because it provides **elastic, self-healing, and automatic scaling** (horizontal scaling).
* Good, because it is the **ideal solution for high-scale, zero-downtime** services.
* Neutral, because it supports the container images built in the chosen Docker Compose option.
* Bad, because it introduces **significant complexity and cost** that is unwarranted for a Proof-of-Concept or small-scale initial client base.
* Bad, because managing the **multi-tenant separation** within a single cluster (namespace segregation, resource quotas) is more complex than simply running isolated Docker Compose units.
