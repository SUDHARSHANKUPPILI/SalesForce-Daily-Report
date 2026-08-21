# Salesforce Developer Bridge Program — Day 7 Documentation

**Project:** Placement Management System
**Day:** Day 7
**Topic:** Bulk Processing and Governor Limits
**Student:** Sudharshan Kuppili

---

## Overview

Day 7 focused on Salesforce Governor Limits and bulk-safe Apex design. The goal was to move from thinking about a single record at a time to thinking about collections of records, and to redesign Apex logic in the Placement Management System so it can safely process 1, 10, or 200 records without changing its structure.

## Objectives / Learning Goals

- Understand Salesforce Governor Limits and why they exist
- Understand bulkification as a design approach, not an optimization
- Avoid SOQL inside loops
- Avoid DML inside loops
- Use Lists, Sets, and Maps for bulk-safe processing
- Process multiple records safely within a single transaction
- Test bulk-safe Apex using Execute Anonymous

## Topics Covered

1. Governor Limits and why Salesforce enforces them
2. Danger of SOQL inside loops
3. Bulkification and the seven-step bulk-processing pattern
4. Lists, Sets, and Maps
5. The existing `PlacementApplicationTrigger`
6. Bulk-safe duplicate validation (`validateNewApplications()`)
7. Bulk SOQL using `IN` with collected ID sets
8. Danger of DML inside loops
9. `ApplicationService.handleApplicationUpdate()`
10. Execute Anonymous testing of the above logic

## Salesforce Concepts Learned

- **Governor Limits** are per-transaction boundaries Salesforce enforces on shared platform resources (e.g., SOQL queries, DML statements, CPU time, heap size), since Salesforce is a multi-tenant platform.
- **Bulkification** means designing Apex so the same logic safely handles any number of records.
- **`Trigger.new`** is a collection of the new/incoming record versions being processed, not a single record.
- **`Trigger.oldMap`** provides the previous versions of records, keyed by Id, used to detect what changed (e.g., a status change to "Selected").
- **`Set<Id>`** is used to collect unique Student and Job IDs before querying, automatically removing duplicates.
- **Bulk SOQL** retrieves related records for an entire collection in one query using `IN`, instead of querying inside a loop.
- **Bulk DML** modifies records in memory inside a loop and performs a single DML statement (e.g., `update students;`) after the loop, instead of one DML statement per record.
- **`addError()`** is used inside validation logic to block invalid records (such as duplicate applications) before they are saved.

Governor limits referenced in the documentation (synchronous Apex):

| Resource | Limit |
|---|---|
| SOQL queries per transaction | 100 |
| SOQL records retrieved per transaction | 50,000 |
| DML statements per transaction | 150 |
| DML records processed per transaction | 10,000 |
| CPU time per transaction | 10,000 ms |
| Heap size per transaction | 6 MB |

## Hands-On Work Completed

- Reviewed the existing `PlacementApplicationTrigger`, which routes `before insert` events to `ApplicationService.validateNewApplications()` and `after update` events to `ApplicationService.handleApplicationUpdate()`.
- Reviewed the bulk-safe implementation of `validateNewApplications()`, which collects Student and Job IDs into Sets, runs a single SOQL query for existing applications, builds a set of existing Student+Job keys, and calls `addError()` on duplicates.
- Reviewed the bulk-safe implementation of `handleApplicationUpdate()`, which detects applications whose status changed to "Selected", queries the related Students once, updates `Placement_Status__c` in memory, and performs a single bulk `update`.
- Ran a read-only Execute Anonymous test that retrieved five Placement Application records in one query and processed them as a collection.
- Identified an existing Placement Application's Student and Job IDs using a read-only query, for use in a controlled duplicate test.
- Ran an initial duplicate-validation test that failed due to a manually typed, invalid Job Id (an incorrect-type error unrelated to the validation logic).
- Corrected the test by using the Student and Job IDs retrieved from the existing application, and successfully reproduced the duplicate-validation block via `addError()`.
- Updated an existing Placement Application's `Status__c` to "Selected" and confirmed the after-update trigger fired.
- Verified, via a read-only query, that the related Student record's `Placement_Status__c` was updated to "Selected".

## Key Apex / Salesforce Concepts Demonstrated

- Seven-step bulk-processing pattern: receive records → collect IDs → query once outside loops → store in Sets/Maps/Lists → process in memory → collect records needing changes → perform DML once.
- Trigger-to-service delegation: the trigger only recognizes the event context (`before insert`, `after update`) and delegates all logic to `ApplicationService`, keeping the trigger lightweight.
- Detecting meaningful business changes (e.g., status becoming "Selected") using `Trigger.new` compared against `Trigger.oldMap`, rather than only checking the current value.

## Important Implementation Details

- `Set<Id> studentIds` and `Set<Id> jobIds` are used in `validateNewApplications()` to collect unique IDs before querying.
- The existing-applications lookup uses:
  ```
  List<Placement_Application__c> existingApplications = [
      SELECT Id, Student__c, Job__c
      FROM Placement_Application__c
      WHERE Student__c IN :studentIds
      AND Job__c IN :jobIds
  ];
  ```
- The bulk-safe Student update pattern used in `handleApplicationUpdate()`:
  ```
  for (Student__c student : students) {
      student.Placement_Status__c = 'Selected';
  }
  update students;
  ```
- Duplicate applications are blocked using `Database.insert(duplicateApp, false)` combined with `addError()` inside the validation logic, allowing the failed result to be inspected via `Database.SaveResult`.

## Screenshots / Evidence

The documentation includes screenshots captured from the Salesforce Developer Console (Execute Anonymous window and debug logs), placed alongside the relevant section:

- **Figure 1** — Bulk processing test showing five Placement Applications retrieved and processed as a single collection (`Number of applications processed: 5`).
- **Figure 2** — Existing Placement Application's Student and Job IDs identified, alongside the corrected, successful duplicate-validation test (`Insert Success: false`, "Duplicate application found. This student has already applied for this job.").
- **Figure 3** — Earlier duplicate-validation attempt that failed due to an incorrect Job Id (included for transparency; not the final result).
- **Figure 4** — Placement Application `Status__c` successfully updated to "Selected", confirming the after-update trigger fired.
- **Figure 5** — Final verification showing the related Student record with `Placement Status: Selected`.

## Test Results Summary

| Test | Result |
|---|---|
| Bulk processing of 5 applications | Passed |
| Existing application Student/Job identification | Passed |
| Duplicate application validation | Passed |
| Application status changed to Selected | Passed |
| Related Student Placement Status updated to Selected | Passed |

## What I Learned

- Governor Limits are engineering boundaries on shared Salesforce resources, not arbitrary restrictions.
- Code that works for a single record can still fail at scale if SOQL or DML sits inside a loop.
- `Trigger.new` should always be treated as a collection, and Apex should be designed around that assumption from the start.
- Sets remove duplicate IDs before querying; Maps/collections enable fast in-memory processing after a single query.
- Keeping the trigger lightweight and delegating logic to a service class improves readability and maintainability.

## Conclusion

Day 7 demonstrated the shift from individual-record thinking to collection-level thinking in Apex development. The Placement Management System's `ApplicationService` and `PlacementApplicationTrigger` were tested with multiple Placement Applications processed as a single collection, duplicate application validation was verified using a corrected, successful test, and the "Selected" status was confirmed to propagate correctly from a Placement Application to its related Student record through the after-update trigger and a single bulk DML statement. This confirmed that the same bulk-safe Apex logic can handle one record or many records without changing its structure — the core goal of bulkification.
