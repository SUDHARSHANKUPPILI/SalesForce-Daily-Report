# Day 8 — Salesforce Asynchronous Apex

## Overview

Day 8 covered Asynchronous Apex on the Salesforce platform — moving non-urgent work out of the
user's immediate transaction and into the background. The day combined conceptual learning with
hands-on implementation in a Developer Edition org, applied to the **Placement Management System**
project.

## Learning Objectives

- Understand synchronous vs. asynchronous Apex execution.
- Recognize when background processing is appropriate.
- Understand Future Methods, Queueable Apex, Batch Apex, and Scheduled Apex.
- Understand that asynchronous Apex still enforces Governor Limits and must remain bulk-safe.
- Apply each asynchronous type to a small, real implementation.

## Topics Covered

- Synchronous vs. Asynchronous Apex
- Future Apex
- Queueable Apex
- Batch Apex
- Scheduled Apex
- Governor Limits in asynchronous contexts
- Choosing the appropriate asynchronous type for a given scenario

## Asynchronous Apex Types

| Type | Summary |
|---|---|
| **Future** | Simple background work. Declared with `@future`, must be `static void`, with parameter types restricted to primitives/collections of primitives. |
| **Queueable** | Flexible, job-oriented background work. Implements `Queueable`, defines `execute(QueueableContext)`, supports constructor parameters and job chaining. |
| **Batch** | Large-volume processing in manageable chunks. Implements `Database.Batchable<SObject>` with `start()`, `execute()`, and `finish()`. |
| **Scheduled** | Time-based / recurring execution. Implements `Schedulable`, defines `execute(SchedulableContext)`, scheduled via `System.schedule()` with a CRON expression. |

## Hands-on Implementation

| Class | Type | Purpose |
|---|---|---|
| `PlacementFutureService` | Future | Processes a Placement Application asynchronously (`processApplication`) |
| `PlacementQueueableService` | Queueable | Processes a Placement Application via `execute(QueueableContext)` |
| `PlacementStudentBatch` | Batch | Processes `Student__c` records in batches of 200 |
| `PlacementScheduledService` | Scheduled | Runs maintenance checks on `Student__c` records |

## Salesforce APIs / Methods Used

- `@future`
- `System.enqueueJob()`
- `Database.executeBatch()`
- `Database.getQueryLocator()`
- `System.schedule()`
- `start()`, `execute()`, `finish()` (Batch Apex)
- `execute(QueueableContext)`
- `execute(SchedulableContext)`

## Verification / Evidence

- **Future Apex** — `PlacementFutureService.processApplication()` was run via Execute Anonymous
  against test record `a0ANS00000CUXcL2AX`; the debug log confirmed successful processing.
- **Queueable Apex** — `PlacementQueueableService` was enqueued via `System.enqueueJob()`; the
  Setup **Apex Jobs** list confirmed both the Queueable and Future jobs as **Completed** with
  **0 Failures**.
- **Batch Apex** — `PlacementStudentBatch` was started via `Database.executeBatch(new
  PlacementStudentBatch(), 200)`; debug logs confirmed the batch completed successfully.
- **Scheduled Apex** — `PlacementScheduledService` was scheduled via `System.schedule()` as
  **"Daily Placement Maintenance"** using the CRON expression `0 0 22 * * ?` (daily, 10:00 PM). The
  scheduling call succeeded and returned a Scheduled Job ID. The actual 10:00 PM execution was
  **not observed** during this session — only the successful scheduling is confirmed.

## Practical Scenarios

- Simple, non-urgent background operation → **Future Apex**
- Flexible background operation / chaining → **Queueable Apex**
- Very large data volume → **Batch Apex**
- Specific/recurring time → **Scheduled Apex**
- Scheduled, large-volume processing → **Scheduled Apex** triggering **Batch Apex**

## Key Learnings

- Asynchronous Apex frees the user transaction from non-urgent work, but doesn't remove Governor
  Limits — bulk-safe design still matters.
- Each asynchronous type fits a distinct kind of problem: simplicity (Future), flexibility/chaining
  (Queueable), scale (Batch), and timing (Scheduled).
- Verifying results (debug logs, Apex Jobs list) is a core part of implementing asynchronous work,
  not an afterthought.

## Conclusion

Day 8 established a working foundation in Salesforce Asynchronous Apex for the Placement
Management System. Future, Queueable, and Batch Apex were implemented, executed, and verified;
Scheduled Apex was successfully configured and confirmed as scheduled. This groundwork supports
further background-processing features as the project grows.
