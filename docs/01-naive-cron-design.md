# Naive Cron-Based Key Rotation Design

## Typical Key Rotation Workflow

At a high level, cryptographic key rotation usually follows a predictable sequence:

1. Generate a new cryptographic key  
2. Distribute the new key to dependent services  
3. Update references to mark the new key as active  
4. Gradually drain usage of the old key  
5. Revoke or delete the old key  

Each step is dependant on the successful completion of the previous one. Any errors or miscofigrations can cause the key to have secuirty or customer usage impacts.

In most real systems, this workflow spans multiple components, databases, and services.

---

## How Cron Is Commonly Used

To automate this workflow, many teams rely on cron-based scheduling.

A typical design looks like this:

- A cron job runs at a fixed interval (e.g., daily or weekly)
- The job executes a rotation script or service
- Near-expiry/Expired keys are indentified
- The script performs the full rotation workflow end-to-end
- Success or failure is logged locally

This approach is attractive because it is:
- simple to reason about
- easy to deploy
- familiar to most engineers

For a single instance or small system, this model often works well.

---

## Example: Single-Pool Cron Rotation

In a non-distributed environment, the rotation setup might resemble:

- One application pool or cluster
- One cron scheduler
- One rotation job with exclusive access to key state

In this case:
- rotations do not overlap
- execution order is predictable
- failures are easier to detect and debug

This environment reinforces the belief that cron-based automation is sufficient.

---

## Implicit Assumptions in This Design

The naive cron-based approach relies on several implicit assumptions:

- Only one instance executes the rotation job  
- Job execution time is shorter than the scheduling interval  
- No other process mutates key state concurrently  
- Failures are binary and immediately visible  

These assumptions are rarely documented, yet they are critical for correctness.

---

## Why This Design Appears Correct

From a local perspective, the design is reasonable:

- The rotation logic is centralized in one script or service
- Scheduling is handled externally by cron
- The system behaves predictably under normal conditions

The problem is not the design itself —  
it is that **the design does not survive scaling**.

---

## Diagram: Naive Cron-Based Rotation

At this stage, the system can be represented as:

- A single scheduler triggering a rotation workflow
- Sequential execution against shared key state

> ![Figure 1](/diagrams/naive-cron-single-pool.png)
> Figure 1: A naive cron-based key rotation design in a single-pool environment.
The rotation job executes sequentially with exclusive access to key state, making cron-based automation appear sufficient.

This diagram will serve as the baseline for understanding how things break in later documents.

<details>
<summary>Click to explain more on Figure 1 : Step 3</summary>
  
> In an HSM-backed KMS, distributing a new key does not involve exporting key material.  
> Instead, the rotation job creates a new key version inside the HSM and updates key metadata so that dependent services transparently begin using the new version via a stable logical identifier (such as a key alias).  
> Applications continue to invoke cryptographic operations against the same key reference, while the KMS ensures new operations use the active key version and existing data remains decryptable using older versions during a controlled drain period.

</details>

---

## Transition to the Next Problem

As systems grow, redundancy and horizontal scaling are introduced for availability and performance.

Cron-based scheduling is often duplicated across pools or environments —  
without revisiting the original assumptions.

The next document examines what happens when this naive design is deployed in **multi-pool architectures**.
