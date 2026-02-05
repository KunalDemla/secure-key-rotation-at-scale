# Rotation API Contract (Conceptual)

This document defines the conceptual API contract for initiating and managing
key rotation within a centralized Key Management Service (KMS).

The focus is on security boundaries, idempotency, and operational safety rather
than concrete implementation details.

---

## Design Goals

- Enforce strict access control for rotation operations
- Prevent overlapping or duplicate rotations
- Ensure safe, deterministic behavior across retries
- Provide strong audit and observability signals

---

## API Boundary

The rotation API represents a **privileged control-plane operation**.

- Not exposed to end users
- Accessible only to authorized orchestration systems
- Backed by HSM-protected key material

---

## Authentication and Authorization

### Authentication
- Mutual TLS or workload identity
- Requests originate only from trusted schedulers (e.g., Control-M)

### Authorization
- Fine-grained role-based access
- Rotation permissions scoped to key domains
- Separation from data-plane cryptographic permissions

---

## Core Endpoints (Conceptual)

### Initiate Rotation

```text
POST /internal/keys/{keyId}/rotate
```

#### Intent
- Starts a new rotation workflow for the specified key

#### Preconditions

- Caller is authorized
- No active rotation is in progress
- Key is in a valid state

#### Behavior

- Acquires rotation lock
- Creates a new key version in the HSM
- Returns a rotation execution identifier

### Rotation Status
```text
GET /internal/keys/{keyId}/rotation-status
```

#### Intent

- Fetches current rotation state for a key

#### Returns

- Rotation phase
- Execution identifier
- Timestamp of last state transition

### Abort Rotation
```text
POST /internal/keys/{keyId}/abort-rotation
```

#### Intent

- Safely aborts an in-progress rotation

#### Behavior

- Preserves existing active key version
- Marks rotation as aborted
- Emits audit events

## Idempotency Guarantees

- Repeated calls with the same execution context return consistent results
- Duplicate initiation attempts do not create new key versions
- Status queries are read-only and side-effect free

## Error Handling Model

```text
4xx Errors:
- Authorization failure
- Invalid key state
- Rotation already in progress

5xx Errors:
- Infrastructure failure
- HSM unavailability
```

Errors never result in:
- Immediate key revocation
- Partial activation
- Silent failure

## Audit and Observability

Every rotation-related API call emits:
- Caller identity
- Key identifier
- Execution identifier
- Outcome (success, failure, aborted)

These events support:
- Compliance reviews
- Incident response
- Forensic analysis

## Security Properties

- Clear separation of control plane and data plane
- Explicit rotation lifecycle
- Least-privilege access enforcement
- Defense against replay and duplicate execution

## Why This Contract Matters

Key rotation is not just a background task—it is a high-risk security
operation.

A well-defined API contract:
- Prevents accidental misuse
- Makes failures observable
- Enables safe automation at scale

## Key Takeaways

- Rotation APIs must be tightly scoped and protected
- Idempotency is non-negotiable
- Observability is a security feature, not an afterthought
