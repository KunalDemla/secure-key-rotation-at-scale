# Concurrency and Collision in Key Rotation Workflows

## When Time-Based Scheduling Meets Long-Running Workflows

Cron-based schedulers operate on a simple model:
- trigger a job at a fixed time
- assume the job completes before the next trigger
- treat each execution as independent

Key rotation workflows violate these assumptions.

They are:
- stateful
- long-running
- dependent on external systems
- sensitive to execution order

As a result, overlapping executions are not edge cases — they are inevitable.

---

## Overlapping Execution Windows

In real systems, key rotation jobs may take longer than expected due to:
- network latency
- throttling in downstream systems
- load on the key management service
- backoff and retries

When execution time exceeds the scheduling interval:
- the next cron trigger fires
- a new rotation job starts
- both executions operate concurrently

This creates **overlapping execution windows**.

---

## Collision on Shared State

The most dangerous collisions occur on shared key metadata, including:
- key aliases
- version status (ACTIVE, DEPRECATED, REVOKED)
- rotation timestamps
- drain period markers

Concurrent updates can result in:
- last-writer-wins behavior
- stale reads followed by destructive writes
- unexpected state regressions

These issues are difficult to detect and often non-deterministic.

---

## Retry Amplification

Failures during rotation often trigger retries.

In cron-based designs:
- retries are frequently time-based
- failed jobs may retry independently
- multiple retries may overlap with scheduled executions

This creates **retry amplification**, where:
- a single failure spawns multiple concurrent executions
- each retry increases system instability
- recovery becomes harder rather than easier

---

## Non-Idempotent Operations

Key rotation operations are rarely idempotent by default.

Examples include:
- generating a new key version
- transitioning key state
- revoking old keys

Re-executing these steps without coordination can:
- create excess key versions
- prematurely revoke active keys
- break compatibility with existing data

Cron provides no guarantees that operations are executed exactly once.

---

## The Illusion of Safety Through Delays

A common mitigation is to introduce:
- sleep statements
- randomized delays
- staggered cron schedules

These approaches:
- reduce collision probability
- do not eliminate it
- fail under load or time drift

They replace deterministic correctness with probabilistic safety.

---

## Diagram: Overlapping Rotation Executions

> ![Figure 3](/diagrams/overlapping-rotation-jobs.png)
> Figure 3: Overlapping key rotation jobs concurrently modify shared key metadata.  
Without coordination or execution ownership, rotation outcomes depend on timing rather than design, leading to inconsistent and unsafe key state.

---

## Why These Problems Escalate at Scale

As systems grow:
- execution time increases
- retry frequency rises
- failure probability compounds

Concurrency issues scale faster than system capacity.

Without explicit coordination, the system becomes increasingly fragile.

---

## Transition to the Next Problem

Concurrency and collision are symptoms of a missing capability:
**controlled execution ownership**.

The next document explores how centralized orchestration introduces serialization, visibility, and coordination to restore correctness.
