# Threat Model

This document outlines the threat model for the centralized key rotation system. It identifies relevant threat actors, assets, trust boundaries, and mitigations, focusing on risks introduced by automation, orchestration, and distributed execution.

---

## Assets

The system protects the following high-value assets:
- Cryptographic keys stored and managed by the KMS
- HSM-backed key material and operations
- Key state metadata (active, deprecated, revoked)
- Audit logs and rotation history
- Rotation control plane (orchestration APIs and workflows)

Loss or misuse of these assets can directly impact confidentiality, integrity, availability, and compliance.

---

## Threat Actors

Potential threat actors include:
- External attackers attempting unauthorized key access
- Compromised internal services misusing valid credentials
- Malicious or negligent insiders
- Automation or orchestration failures behaving as unintended actors

The model assumes that dependent services may fail or behave unexpectedly.

---

## Trust Boundaries

Key trust boundaries include:
- Between Control-M and the rotation API
- Between the rotation workflow and the KMS/HSM
- Between the KMS and dependent services
- Between execution pools in a multi-pool architecture

All transitions across these boundaries are explicitly authenticated and authorized.

---

## Identified Threats

### Unauthorized Key Rotation
An attacker or misconfigured system attempts to trigger key rotation outside the approved workflow.

**Mitigation**
- Rotation APIs restricted to Control-M identity
- Strong authentication and authorization checks
- Explicit execution ownership validation

---

### Concurrent or Duplicate Rotation
Multiple rotation workflows attempt to operate on the same key domain simultaneously.

**Mitigation**
- Single-entry orchestration through Control-M
- Deterministic pool designation via load balancer
- Rotation locks enforced at workflow start

---

### Partial Execution Abuse
A rotation partially completes, leaving the system in an inconsistent state that could be exploited.

**Mitigation**
- Idempotent workflow steps
- Pre- and post-condition validation
- Abort-on-inconsistency behavior

---

### Key State Manipulation
An attacker or faulty process attempts to alter key metadata (aliases, states) directly.

**Mitigation**
- No direct key state mutation outside the rotation workflow
- Immutable audit logging
- Strict IAM policies for key operations

---

### Audit Log Tampering
An attacker attempts to hide or modify evidence of key lifecycle events.

**Mitigation**
- Centralized, append-only logging
- Correlation IDs across all rotation stages
- Restricted write access to audit logs

---

### Denial of Service via Rotation Abuse
Repeated rotation triggers are used to exhaust resources or destabilize dependent services.

**Mitigation**
- Rate limiting on rotation APIs
- Manual approval for emergency rotations
- Alerting on abnormal rotation frequency

---

## Assumptions

The threat model assumes:
- The HSM enforces strong cryptographic guarantees
- Control-M is a trusted execution authority
- Dependent services correctly consume key state updates
- Network-level protections (TLS, mTLS) are in place

Violations of these assumptions are treated as incidents.

---

## Residual Risk

Even with mitigations, residual risks include:
- Operational errors during emergency rotations
- Delayed detection of subtle partial failures
- Dependency misconfiguration in downstream services

These risks are addressed through monitoring, training, and operational playbooks.

---

## Summary

This threat model demonstrates that centralized orchestration, strict trust boundaries, and deterministic execution significantly reduce the attack surface of automated key rotation. By treating rotation as a security-critical workflow, the system mitigates both malicious and accidental threats while maintaining auditability and compliance.
