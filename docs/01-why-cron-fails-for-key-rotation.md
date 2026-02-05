# Why Cron Jobs Fail for Secure Key Rotation

Key rotation is often treated as a routine maintenance task.
In distributed systems, this assumption breaks down quickly.

What appears safe in a single-instance environment becomes fragile,
dangerous, and opaque when applied to multi-pool, security-critical systems.

This document explains why traditional cron-based scheduling fails for
secure key rotation and what properties are required instead.

---

## The Assumption That Breaks

Cron is built on a simple assumption:

> “If a job runs at a fixed time, it will run once.”

In distributed environments, this assumption is no longer valid.

When multiple pools, instances, or regions exist, cron becomes:
- Pool-local, not system-wide
- Blind to concurrent execution
- Unaware of global state

For key rotation, this is unacceptable.

---

## Key Rotation Is a Stateful Security Workflow

Key rotation is not a stateless batch job.

It involves:
- Generating new key material
- Coordinating dependent services
- Preserving decryption guarantees
- Maintaining audit and compliance invariants

A second, overlapping execution is not redundant — it is **dangerous**.

Cron provides no native protection against this class of failure.

---

## Overlapping Execution Is Not a Corner Case

In multi-pool systems, overlapping execution is the default failure mode.

Common triggers include:
- Clock skew between pools
- Manual re-runs during incident response
- Deployment restarts
- Partial failures interpreted as “safe to retry”

Cron does not prevent two pools from rotating the same key at the same time.

Once this happens, the system enters an undefined state that is difficult
to detect and even harder to recover from safely.

---

## Retries Make the Situation Worse

Cron encourages retries by design.

For stateful security workflows, retries introduce:
- Duplicate key generation
- Conflicting activation windows
- Unclear ownership of the active key

A retry is not a continuation — it is a new execution.
Cron cannot distinguish between the two.

In key management systems, this distinction matters.

---

## Cron Lacks Observability and Intent

When a cron job fails, operators often see:
- A non-zero exit code
- Minimal execution context
- No correlation to system state

This leads to unsafe decisions:
- “Just run it again”
- “It probably didn’t do anything”
- “We can clean it up later”

Cron does not encode *intent* or *phase*.
Security workflows require both.

---

## Why Local Scheduling Fails at Scale

Cron optimizes for locality and simplicity.

Security workflows require:
- Global coordination
- Single-execution guarantees
- Centralized visibility
- Explicit failure semantics

Local scheduling mechanisms cannot provide these properties reliably,
regardless of how carefully they are configured.

---

## What Actually Solves the Problem

Secure key rotation requires orchestration, not scheduling.

Effective solutions provide:
- Centralized execution control
- Explicit locking and idempotency
- No blind retries
- Strong audit and observability signals

These properties turn key rotation from a best-effort task into a
deterministic, inspectable workflow.

---

## Final Takeaway

Cron is a powerful tool — for the problems it was designed to solve.

Secure key rotation is not one of them.

Treating key rotation as a simple scheduled job ignores the distributed,
stateful, and high-risk nature of the operation.

In security-critical systems, correctness must come before convenience.
