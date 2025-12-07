# Adoption of JWT (JSON Web Tokens) with RBAC for Authentication and Authorisation

## Context and Problem Statement

The Complaint Management System (CMS) handles sensitive user data (client complaints, contact information) and must adhere to strict data privacy and security regulations. Crucially, the system must support multiple defined roles (**Consumer**, **Help Desk Agent**, **Support Person**, **Manager**, **System Administrator**) with tightly controlled access to specific functions and data (Role-Based Access Control - RBAC).

The question is: **What combination of authentication protocols and authorisation models should be used to securely identify users, enforce role-based permissions, and manage session state across the FastAPI backend and React frontend?**

---

## Decision Drivers

* **Data Integrity & Security:** Must protect user data and ensure compliance with industry best practices (e.g., OWASP).
* **Role Enforcement (RBAC):** Must strictly enforce permissions based on the five defined user roles.
* **Statelessness:** Must support a scalable, **stateless backend (FastAPI)** to handle multiple clients efficiently.
* **Ease of Implementation:** Should use widely supported standards that integrate easily with Python/FastAPI and React.
* **Non-Repudiation:** Passwords must be stored securely and verifiably.

---

## Considered Options

* JWT (JSON Web Tokens) with RBAC
* Traditional Cookie-based Session Management
* API Keys for all Users
* Client Certificates (PKI)

---

## Decision Outcome

Chosen option: **JWT (JSON Web Tokens) with RBAC**, because it provides a strong, stateless, and portable mechanism for handling session state and authorisation, which is ideal for a Modular Monolith API service. The use of **JWTs stored in HttpOnly/Secure cookies** addresses session management and CSRF concerns, while **password hashing (e.g., bcrypt)** ensures credential security.

### Consequences

* **Good, because:**
    * **Statelessness:** The backend does not need to store session information, making it **highly scalable** and removing database lookups for every request.
    * **Role Enforcement:** The user's role and permissions can be encoded directly into the JWT payload, allowing for efficient RBAC checks in the FastAPI router layer.
    * **Cross-Origin Compatibility:** JWTs are simple to pass between the FastAPI API and the React frontend, even if they are hosted on different domains.
    * **Credential Security:** Hashing passwords with a slow, modern algorithm like **bcrypt** protects against common credential theft attacks.

* **Bad, because:**
    * **Revocation Complexity:** JWTs are valid until they expire; immediate revocation (e.g., when a user is logged out forcibly) requires an extra mechanism (like a Redis blocklist).
    * **Token Size:** Encoding many permissions or user details can increase the size of the token, slowing down requests slightly.
    * **Storage Vulnerability:** While HttpOnly cookies mitigate XSS risks, securing the token storage in the browser remains a critical point of failure.

### Confirmation

Compliance with this ADR will be confirmed through:
* **Code Review:** Verification that a strong, slow hashing algorithm (e.g., bcrypt) is used for all password storage.
* **Authentication Flow:** Confirmation that the application uses a **Secure, HttpOnly cookie** to store the JWT access token and a separate process for handling token expiry/refresh.
* **Authorisation Testing:** End-to-End tests proving that users who lack specific roles (e.g., a "Consumer") are blocked from accessing privileged API endpoints (e.g., `/admin/onboard`).

---

## Pros and Cons of the Options

### JWT (JSON Web Tokens) with RBAC

Uses signed tokens containing user claims (including role/permissions) for stateless authentication.

* Good, because it is **stateless and highly scalable**, perfect for microservices or modular APIs.
* Good, because it **natively supports RBAC** by encoding the role in the token payload.
* Neutral, because JWTs must be protected from XSS/CSRF via proper storage mechanisms (e.g., HttpOnly, Secure cookies).
* Bad, because **token revocation** requires additional overhead (e.g., a blacklist check).

---

### Traditional Cookie-based Session Management

Relies on a session ID stored in a cookie, where the ID maps to a session object stored on the server/database.

* Good, because **revocation is instant** (simply delete the session object from the server).
* Good, because it is a **simple, mature, and well-understood protocol** for web applications.
* Neutral, because it requires server resources (database/cache) to store and look up the session state for every request.
* Bad, because it **complicates scalability** by requiring session affinity or a distributed session store.
* Bad, because it is **not natively suited for API-only communication** (e.g., mobile apps) without additional headers.

---

### API Keys for all Users

A system where every user authenticates with a long-lived, pre-generated secret key (similar to a password).

* Good, because it is **simple to implement** and manage for machine-to-machine communication.
* Neutral, because keys can be generated with associated permissions (RBAC).
* Bad, because it is **highly insecure** for human-driven sessions due to the difficulty of key rotation and the ease of key theft.
* Bad, because it **lacks session expiry and automatic rotation**, making it unsuitable for interactive user authentication.

---

### Client Certificates (PKI)

Uses cryptographic certificates installed on the client's machine (browser or device) for mutual TLS authentication.

* Good, because it provides the **highest level of cryptographic assurance and security**.
* Good, because it is **natively supported by HTTPS/TLS** protocols.
* Bad, because it has **high operational overhead** (issuing, managing, and revoking certificates) for millions of general users.
* Bad, because it is **not user-friendly** (requires users to install and manage certificate files).
