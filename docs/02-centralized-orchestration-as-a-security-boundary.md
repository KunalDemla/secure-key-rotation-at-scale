# Centralized Orchestration as a Security Boundary

In distributed systems, security boundaries are often discussed in terms of
networks, identities, and cryptography.

Less obvious — but equally critical — is **where control originates**.

This document explains why centralized orchestration should be treated as a
first-class security boundary for sensitive workflows such as key rotation.

---

## Control Is a Security Concern

Any system that can initiate a security-critical action effectively holds
security authority.

For key rotation, this includes the ability to:
- Generate new key material
- Activate or deactivate key versions
- Influence cryptographic trust across services

If control is decentralized, security authority becomes fragmented.

---

## The Problem with Distributed Control

In multi-pool architectures, allowing each pool to independently initiate
rotations introduces ambiguity:

- Multiple actors can trigger the same operation
- No single source of truth exists for execution state
- Failures are difficult to attribute or reason about

From a security perspective, this is equivalent to having multiple control
planes — an inherently unsafe model.

---

## Orchestration Defines Trust Boundaries

Centralized orchestration establishes a clear trust boundary:

- Only authorized orchestration systems can initiate rotation
- Execution is explicitly scoped and serialized
- All actions are traceable to a single control source

This transforms key rotation from a best-effort task into a controlled
security operation.

---

## Separation of Control Plane and Data Plane

Central orchestration reinforces a clean separation:

- **Control plane**: Initiates, sequences, and observes rotation
- **Data plane**: Performs cryptographic operations under strict constraints

This separation limits blast radius and prevents unintended side effects from
data-plane failures.

---

## Preventing Accidental Privilege Escalation

Without centralized control, any pool capable of scheduling a job implicitly
gains rotation authority.

Central orchestration:
- Reduces the number of entities with rotation privileges
- Enforces least-privilege access
- Makes privilege boundaries explicit and reviewable

Security improves by reducing who can act, not by trusting everyone to act well.

---

## Observability Is Part of the Boundary

A security boundary without visibility is incomplete.

Central orchestration provides:
- Execution context
- Phase awareness
- Correlated audit trails

This enables operators to understand **what happened**, not just that
something failed.

---

## Human Factors and Incident Response

During incidents, engineers act under pressure.

A centralized control point ensures that:
- Unsafe actions require deliberate intent
- The safest recovery path is clearly defined
- Manual interventions are visible and auditable

Security design must assume stressed humans, not ideal conditions.

---

## Orchestration Is Not Just Scheduling

Scheduling answers *when* something runs.
Orchestration answers:
- Who is allowed to run it
- Where it runs
- Whether it should run at all

For security-sensitive workflows, these questions matter more than timing.

---

## Final Takeaway

Centralized orchestration is not merely an operational convenience.
It is a **security boundary**.

By defining who can initiate, control, and observe sensitive workflows,
orchestration becomes an integral part of the system’s security posture.

Ignoring this boundary turns distributed systems into distributed risk.
