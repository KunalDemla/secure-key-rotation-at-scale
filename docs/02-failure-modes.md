# Failure Modes in Multi-Pool Architectures

## From Single Pool to Multi-Pool

As systems scale, redundancy and isolation are introduced to improve availability, resilience, and blast-radius containment.

A typical evolution looks like:
- multiple application pools or clusters
- independent deployment units
- separate execution environments
- horizontal autoscaling within each pool

Cron-based automation is often duplicated across these pools without re-evaluating its original assumptions.

The result is a system where **multiple schedulers independently trigger the same security-critical workflow**.

---

## Multi-Pool Cron Deployment Model

In a multi-pool architecture:

- each pool runs its own cron scheduler
- each scheduler triggers the same rotation job
- all jobs operate on the same global key state

No pool is aware of the others.

From the perspective of each scheduler, it is the sole executor.

---

## Failure Mode 1: Duplicate Rotation Execution

The most common failure occurs when:

- cron jobs in different pools trigger at roughly the same time
- each job begins a full rotation workflow
- both believe they have exclusive ownership

Outcomes include:
- multiple key versions created for a single rotation window
- conflicting updates to key metadata
- non-deterministic final key state

These failures are often silent and only detectable via audit analysis.

> ⚠️ **Callout:**  
> In Figure 2, both Pool A and Pool B trigger rotation jobs at the same scheduled time.  
> Neither scheduler is aware that another execution is already in progress.

---

## Failure Mode 2: Overlapping Drain Windows

Key rotation workflows typically rely on a drain period during which:

- new operations use the active key
- old key versions remain available for decryption or verification

When multiple rotation jobs execute concurrently:
- drain windows overlap or reset
- deprecated keys may be revoked prematurely
- consumers experience intermittent cryptographic failures

The system appears healthy while silently breaking guarantees.

---

## Failure Mode 3: Partial Execution Across Pools

Not all failures are clean.

Common scenarios include:
- one pool completing rotation successfully
- another pool failing mid-execution
- retries triggering additional rotations

This leads to:
- inconsistent key state
- broken assumptions about rotation history
- complex recovery procedures

Manual intervention often becomes the only safe resolution.

---

## Failure Mode 4: Race Conditions on Shared Metadata

Key activation relies on updating shared metadata such as:
- key aliases
- version status
- rotation timestamps

Concurrent updates introduce race conditions:
- last-writer-wins behavior
- lost updates
- stale reads

Without strong coordination, correctness depends on timing rather than design.

> ⚠️ **Callout:**  
> The key alias represents shared global state.  
> Concurrent updates from independent pools introduce race conditions that cron cannot detect or prevent.

---

## Why Cron Cannot Prevent These Failures

Cron provides:
- time-based triggering
- no global locking
- no awareness of in-progress executions
- no built-in serialization across pools

Attempts to mitigate this using:
- sleep delays
- best-effort locks
- ad-hoc leader election

tend to increase complexity without providing strong guarantees.

---

## Diagram: Cron-Based Rotation in Multi-Pool Architecture

> ![Figure 2](/diagrams/naive-cron-multi-pool.png)
> Figure 2: In a multi-pool architecture, independent cron schedulers trigger key rotation jobs concurrently.  
Each job operates on shared key metadata without global coordination, leading to race conditions and inconsistent key state.

---

## Why These Failures Are Hard to Detect

Many of these issues:
- do not cause immediate outages
- only affect specific consumers
- surface long after the rotation event

By the time failures are noticed:
- logs may be incomplete
- state may have changed further
- forensic reconstruction becomes difficult


> ⚠️ **Callout:**  
> These failures are not caused by bugs in rotation logic, but by the absence of execution ownership and coordination.

---

## Transition to the Next Problem

The failure modes described above are symptoms of a deeper issue.

At scale, the problem is not scheduling —  
it is **concurrent execution without coordination**.

The next document examines how overlapping execution windows and retries amplify these failures through concurrency and collision.
