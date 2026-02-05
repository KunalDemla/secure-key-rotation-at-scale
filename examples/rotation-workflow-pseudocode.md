# Key Rotation Workflow (Pseudocode)

This document provides a high-level, conceptual pseudocode representation of a
secure key rotation workflow. It focuses on execution order, safety guarantees,
and failure handling without exposing production implementation details.

---

## Assumptions

- Rotation is triggered only via centralized orchestration (e.g., Control-M)
- A single execution pool is designated per rotation
- KMS operations are backed by an HSM
- All steps are idempotent

---

## High-Level Workflow

```text
rotateKey(keyId):
    acquireRotationLock(keyId)

    validateCurrentKeyState(keyId)
    validateNoActiveRotation(keyId)

    newKeyVersion = generateNewKey(keyId)

    distributeKeyToDependentServices(newKeyVersion)

    waitForDependencyAcknowledgement(newKeyVersion)

    activateNewKey(keyId, newKeyVersion)

    deprecateOldKeyVersions(keyId)

    scheduleDeferredRevocation(keyId)

    releaseRotationLock(keyId)
```

## Detailed Step Breakdown
### Acquire Rotation Lock
- Ensures exclusive execution for the given key domain
- Prevents concurrent or duplicate rotation attempts
- Failure here aborts the workflow immediately

### Validate Current Key State

- Confirms expected key status (active, not revoked)
- Ensures metadata consistency
- Aborts if unexpected state is detected

### Generate New Key
- New key material is generated within the HSM
- A new key version is created but not yet active
- No dependent service can use the key at this stage

### Distribute Key to Dependent Services
- New key version is securely propagated
- Services receive metadata and access permissions
- Key is not yet used for encryption

### Dependency Acknowledgement
- Dependent services confirm readiness
- Rotation halts if acknowledgements are incomplete
- Prevents partial activation across the fleet

### Activate New Key
- New key version becomes active for encryption
- Alias mappings are updated atomically
- Decryption with previous versions remains valid

### Deprecate Old Key Versions
- Old key versions are marked deprecated
- Encryption is disabled, decryption remains allowed
- No immediate revocation occurs

### Deferred Revocation
- Revocation is scheduled after a safety window
- Allows rollback-free recovery
- Aligns with compliance and retention policies

### Release Rotation Lock
- Lock is released only after successful completion
- Enables future rotations
- Marks workflow as completed in audit logs

## Failure Handling (Simplified)
> onFailure(stage):  
> &nbsp;&nbsp;&nbsp;&nbsp; abortWorkflow()  
> &nbsp;&nbsp;&nbsp;&nbsp; preserveCurrentKeyState()  
> &nbsp;&nbsp;&nbsp;&nbsp; emitAuditEvent(stage)  
> &nbsp;&nbsp;&nbsp;&nbsp; requireManualReviewIfNeeded()  


Failures never trigger immediate revocation or blind retries.

## Key Properties
- Deterministic execution
- Safe abort semantics
- No overlapping rotations
- Audit-friendly
- Designed for distributed environments