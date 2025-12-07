# Adoption of Bcrypt/Argon2 (via pwdlib) and Strong Password Policy for CMS Credential Security

## Context and Problem Statement

The CMS requires that all user credentials (passwords) are stored securely and that users are encouraged to choose strong, unique passwords to mitigate risks from external data breaches and brute-force attacks. Best practices dictate that passwords must never be stored in plaintext. The project uses the Python library `pwdlib` to handle hashing.

The question is: **What mandatory hashing algorithm and policy should be enforced to maximize the security of user passwords against offline attacks while maintaining an acceptable level of performance during login?**

## Decision Drivers

* **Non-Repudiation:** Passwords must be stored in an irreversible, hashed form.
* **Resistance to Brute-Force:** The hashing function must be **slow (computationally expensive)** to thwart offline cracking attempts (e.g., dictionary and brute-force attacks).
* **Salt Requirement:** Must utilize unique salts to prevent the use of pre-computed hash tables (rainbow tables).
* **User Experience:** The password policy must be enforced client-side and server-side to guide users toward strong passwords without being excessively restrictive.
* **Technology Alignment:** Must use a library readily available and supported within the Python/FastAPI ecosystem, specifically confirming the use of **pwdlib**.

## Considered Options

* **Bcrypt/Argon2 (via pwdlib)** (Slow, Adaptive Key Derivation Function)
* **PBKDF2** (Password-Based Key Derivation Function 2)
* **SHA-256** (Standard Cryptographic Hash)
* **Plaintext/Base64 Encoding**

## Decision Outcome

Chosen option: **Bcrypt/Argon2 (via pwdlib)**, because the `pwdlib.PasswordHash.recommended()` utility automatically selects the strongest, most modern, and **adaptive** hashing algorithm available (currently **Argon2** or **Bcrypt**), which are proven to be the strongest defense against offline brute-force attacks. This aligns perfectly with the security needs of the CMS and the team's choice of FastAPI/Python.

### Consequences

* **Good, because:**
    * **Adaptive Security:** The chosen algorithm's **cost factor (work factor)** can be increased over time to maintain security against improved hardware.
    * **Automated Salting:** The algorithm automatically handles the creation and storage of a unique salt for every password, eliminating rainbow table attacks.
    * **Strong Policy:** Enforcing a strong, complex password policy (length, complexity) significantly reduces the risk of successful guessing.
    * **Ease of Implementation:** **pwdlib** provides a robust, simple, and secure wrapper for the best-in-class algorithms.

* **Bad, because:**
    * **Initial Login Delay:** The intentional slowness of the algorithm means password verification adds a small but measurable delay (e.g., 200ms-500ms) to the login process.
    * **Computational Cost:** Verifying passwords uses more CPU resources on the server than fast hash functions (like SHA-256), though this is a necessary trade-off for security.

### Confirmation

Compliance with this ADR will be confirmed through:
* **Code Review:** Verification that the **pwdlib** library is implemented using `PasswordHash.recommended()` or a specifically configured secure hash (e.g., `PasswordHash.bcrypt()`).
* **Policy Enforcement:** Implementation of mandatory checks in the FastAPI application logic to enforce the chosen password complexity policy (e.g., minimum length, use of mixed case, numbers, and symbols).
* **Database Inspection:** Confirmation that the password column in PostgreSQL only contains long, randomized hash strings prefixed with the algorithm identifier (e.g., `$argon2id$`, `$bcrypt$`).

## Pros and Cons of the Options

### Bcrypt/Argon2 (via pwdlib) (Chosen Option)

Adaptive, iterative key derivation functions designed specifically for password hashing.

* Good, because it is **computationally slow and adaptive**, providing excellent defense against brute-force attacks.
* Good, because it **automatically salts** passwords, preventing rainbow table attacks.
* Neutral, because `pwdlib` hides the complexity while enforcing the highest current standards (Argon2/Bcrypt).
* Bad, because it is **intentionally slow**, adding a small, necessary delay to the login process.

---

### PBKDF2

Another widely-used, standard password hashing function that is iterative and slow.

* Good, because it is **NIST-approved** and considered secure when configured correctly.
* Good, because it is an **iterative and salted** function, resisting brute-force attempts.
* Neutral, because it requires **manual configuration** of the iteration count, which must be managed and increased over time.
* Bad, because it is generally considered **less resistant to GPU-based attacks** than Argon2 or Bcrypt.

---

### SHA-256 (Standard Cryptographic Hash)

A fast, cryptographic hash function used for data integrity, often misused for password storage.

* Good, because it is **extremely fast** and highly efficient on the CPU.
* Good, because it is **ubiquitous and simple** to implement in any language.
* Bad, because its **speed is a major security flaw**; attackers can perform billions of hash checks per second offline.
* Bad, because it **must be manually salted** and stretched (iterated many times) to provide even minimal security.

---

### Plaintext/Base64 Encoding

Storing the password directly or simply encoding it to obscure it slightly.

* Good, because it is **easy to implement** and fast to verify.
* Neutral, because... (No positive security neutral points exist for this option).
* Bad, because it provides **zero security**; a database breach immediately exposes every user's password.
* Bad, because it **violates every single security standard** and best practice (a knock-out criterion).
