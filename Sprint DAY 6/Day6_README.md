# Salesforce Developer Bridge Program — Day 6

**Student:** Sudharshan Kuppili
**PIN No.:** 24PA5A0517
**Project:** Placement Management System
**Topic:** Apex Triggers and Trigger Handler / Service

---

## 📌 Project Overview

The **Placement Management System** is a Salesforce application built to manage student placements, job applications, and recruitment status across the program.

Day 6 focused on **Apex Triggers** — teaching the system to respond automatically to business events instead of relying on manual updates. Two automations were implemented:

- Preventing a student from submitting a **duplicate job application** (same Student + same Job).
- Automatically updating a **Student's placement status** when their application moves from *Shortlisted* to *Selected*.

Apex Trigger automation was required because these rules needed to be enforced consistently at the database level, every time a record was inserted or updated — not left to be remembered by a user.

---

## 🎯 Day 6 Objectives

- Understand **Before Insert** and **After Update** Trigger events.
- Use **Trigger context variables** (`Trigger.new`, `Trigger.oldMap`, `Trigger.isBefore`, `Trigger.isAfter`, `Trigger.isInsert`, `Trigger.isUpdate`).
- Prevent **duplicate Placement Applications**.
- Use **`addError()`** to stop invalid records from saving.
- Build **bulk-safe** Apex logic (no SOQL/DML inside loops).
- Separate Trigger logic from business logic using an **`ApplicationService`** class.
- Update a **related Student record** when an application becomes Selected.

---

## 🏗️ Architecture

```text
PlacementApplicationTrigger
          ↓
ApplicationService
          ↓
Business Logic
          ↓
Salesforce Database
```

The Trigger only recognises the database event (insert/update) and calls the matching `ApplicationService` method. All decision-making — querying, validating, and updating related records — lives inside `ApplicationService`. This keeps the Trigger short, readable, and easy to maintain.

---

## ⚡ Apex Trigger Implementation

**Object:** `Placement_Application__c`
**Events:** `before insert`, `after update`

```apex
trigger PlacementApplicationTrigger
on Placement_Application__c (before insert, after update) {

    if (Trigger.isBefore && Trigger.isInsert) {
        ApplicationService.validateNewApplications(Trigger.new);
    }

    if (Trigger.isAfter && Trigger.isUpdate) {
        ApplicationService.handleApplicationUpdate(
            Trigger.new,
            Trigger.oldMap
        );
    }
}
```

### Before Insert — Duplicate Prevention

- The duplicate combination checked is **Student + Job**.
- Student and Job Ids are collected from `Trigger.new`, existing applications are queried in **one SOQL call**, and duplicate combinations are built from the results.
- If a new application matches an existing Student + Job combination, `addError()` blocks the insert.
- This is a **business validation** enforced during the transaction itself, so no duplicate application can ever reach the database.

```apex
public static void validateNewApplications(
    List<Placement_Application__c> newApplications
) {
    Set<Id> studentIds = new Set<Id>();
    Set<Id> jobIds = new Set<Id>();

    for (Placement_Application__c application : newApplications) {

        if (application.Student__c != null) {
            studentIds.add(application.Student__c);
        }

        if (application.Job__c != null) {
            jobIds.add(application.Job__c);
        }
    }
    // Existing applications for these Student + Job combinations are then
    // queried in a single SOQL call, and addError() is used to block
    // any new record that matches an existing combination.
}
```

### After Update — Related Record Update

- Detects when `Status__c` changes from **Shortlisted → Selected**, using `Trigger.oldMap` to compare each record's previous value against `Trigger.new`.
- Collects the related Student Ids into a `Set<Id>` for records that just became Selected.
- Queries the related `Student__c` records once and updates their `Placement_Status__c` field to **`Selected`**.

```apex
public static void handleApplicationUpdate(
    List<Placement_Application__c> newApplications,
    Map<Id, Placement_Application__c> oldApplications
) {
    Set<Id> studentIds = new Set<Id>();

    for (Placement_Application__c newApplication : newApplications) {

        Placement_Application__c oldApplication =
            oldApplications.get(newApplication.Id);

        if (
            oldApplication.Status__c != newApplication.Status__c &&
            newApplication.Status__c == 'Selected' &&
            newApplication.Student__c != null
        ) {
            studentIds.add(newApplication.Student__c);
        }
    }

    if (studentIds.isEmpty()) {
        return;
    }

    List<Student__c> students = [
        SELECT Id, Placement_Status__c
        FROM Student__c
        WHERE Id IN :studentIds
    ];

    for (Student__c student : students) {
        student.Placement_Status__c = 'Selected';
    }
    // Students are then updated in a single bulk DML statement.
}
```

A `Placement_Status__c` picklist field was created on `Student__c` to support this update.

---

## 🧩 ApplicationService

Business logic was moved out of the Trigger and into `ApplicationService` to improve **maintainability, reusability, testability, readability**, and **bulk processing** behavior. A Trigger should coordinate, not calculate.

Methods documented for Day 6:

| Method | Purpose |
|---|---|
| `validateNewApplications()` | Runs the Before Insert duplicate-prevention logic. |
| `handleApplicationUpdate()` | Runs the After Update status-change logic. |
| `submitApplication()` | Creates new Placement Application records. |
| `updateApplicationStatus()` | Updates the `Status__c` field on an existing application (used throughout testing). |

```apex
public void updateApplicationStatus(Id applicationId, String newStatus) {

    Placement_Application__c application = [
        SELECT Id, Status__c, Interview_Date__c
        FROM Placement_Application__c
        WHERE Id = :applicationId
        LIMIT 1
    ];

    application.Status__c = newStatus;

    if (newStatus == 'Interview Scheduled') {
        application.Interview_Date__c = Date.today();
    }

    update application;

    System.debug(
        'Application status updated successfully to: ' + newStatus
    );
}
```

---

## 🚀 Bulkification & Governor Limits

- SOQL and DML were kept **outside of loops** in both the duplicate-prevention and After Update logic.
- Student and Job Ids are collected into `Set<Id>` variables before any query runs.
- A **single SOQL query** retrieves all related records at once; results are processed in memory.
- A **single DML operation** commits all changes, instead of one DML per record.

This pattern (collect → query once → process in memory → one DML) ensures the code behaves correctly whether one record or many records are processed in the same transaction, and stays within Salesforce **Governor Limits**.

---

## 🛡️ Validation & Error Handling

`addError()` is used on the duplicate Placement Application to:

- Stop the invalid record from being inserted.
- Display a meaningful message to the user.
- Enforce the business rule during the database transaction, before any data is committed.

**Error produced:**

```
System.DmlException: Insert failed. First exception on row 0; first error:
FIELD_CUSTOM_VALIDATION_EXCEPTION, Duplicate application found. This student
has already applied for this job.: []
```

---

## 🧪 Testing

| Test | Expected Result | Actual Result | Status |
|---|---|---|---|
| Duplicate Application | Insert blocked | Duplicate blocked | ✅ PASS |
| Duplicate Count | Only 1 application | Count = 1 | ✅ PASS |
| Shortlisted → Selected | Related Student updated | Placement Status = Selected | ✅ PASS |
| Trigger Execution | No unexpected errors | Successful | ✅ PASS |
| Service Class | Compiles successfully | No errors | ✅ PASS |

---

## 📊 Results

Day 6 successfully demonstrated two working business automation scenarios on the Placement Management System:

- Duplicate job applications (same Student + same Job) are **blocked before insert**, with a clear validation message.
- When an application status changes from **Shortlisted to Selected**, the related Student's `Placement_Status__c` is **automatically updated to Selected**.

Both automations follow Trigger-Service separation, are bulk-safe, and were verified through Execute Anonymous testing in the Developer Console.

---

## 🧠 Key Learnings

- Before Triggers suit **validation** and modifying a record before it saves.
- After Triggers suit actions that depend on the **saved record** or its relationships.
- SOQL and DML should **never** sit inside a loop.
- `Set`/`List` collections are the foundation of bulk-safe Apex.
- `addError()` blocks invalid data while giving the user a clear reason.
- `Trigger.oldMap` lets you compare old vs. new field values during an update.
- Business logic belongs in a **service class**, not the Trigger itself.
- Related-record updates must be implemented in a bulk-safe manner.

---

## 💼 Interview Concepts

- **What is an Apex Trigger?** Apex code that executes automatically before or after specific database events on Salesforce records.
- **Before vs. After Trigger:** Before Triggers validate/modify a record prior to save; After Triggers act on the saved record or its relationships.
- **`Trigger.new`:** List of the new/updated record versions in the current transaction.
- **`Trigger.oldMap`:** Map of the previous version of each record, keyed by Id — used to detect field changes.
- **`addError()`:** Prevents a record from saving and shows a meaningful validation error.
- **Governor Limits:** Salesforce-imposed limits (e.g., SOQL query count) that make SOQL/DML inside loops dangerous at scale.
- **Bulkification:** Writing Apex that works correctly for 1 record or thousands, via Sets/Lists, one SOQL query, and one DML statement.
- **Trigger-Service Pattern:** Keep the Trigger lightweight; delegate all business logic to a reusable service class (`ApplicationService`).

---

## 📸 Evidence / Screenshots

The final documentation includes Developer Console and Salesforce UI screenshots as evidence, mapped to the sections above:

| Evidence | Shows |
|---|---|
| `PlacementApplicationTrigger.apxt` (Developer Console) | Trigger source code — Before Insert / After Update blocks. |
| `ApplicationService.apxc` — `validateNewApplications()` | Student/Job Id collection for duplicate prevention. |
| Execute Anonymous — duplicate insert attempt | Apex code used to trigger the duplicate test. |
| Execute Anonymous Error dialog | `FIELD_CUSTOM_VALIDATION_EXCEPTION` confirming the duplicate was blocked. |
| `ApplicationService.apxc` — `handleApplicationUpdate()` | Shortlisted → Selected comparison and Student update logic. |
| Execute Anonymous — status set to Selected | Debug log: "Application selected" / "Application status updated successfully to: Selected". |
| Execute Anonymous — verification query | Debug log confirming `Application Status: Selected` and Student Name. |
| `ApplicationService.apxc` — `updateApplicationStatus()` | Shared method used across Day 6 testing. |
| Execute Anonymous — application count query | Debug log: "Application count: 1" (confirms no duplicate persisted). |
| Execute Anonymous — status set to Shortlisted | Setup step ahead of the Selected transition test. |

Supplementary screenshots (Interview Scheduled status test, a Placement Application detail page, an Id-verification query, and a second Developer Console view of the trigger) are also included in the final documentation's Evidence Appendix as supporting material.

---

## 📁 Project Structure

Based on the Apex classes, trigger, and objects referenced in the final documentation:

```text
force-app/
└── main/
    └── default/
        ├── classes/
        │   └── ApplicationService.cls
        ├── triggers/
        │   └── PlacementApplicationTrigger.trigger
        └── objects/
            ├── Placement_Application__c/
            │   └── fields/
            │       ├── Student__c
            │       ├── Job__c
            │       ├── Application_Date__c
            │       ├── Status__c
            │       └── Interview_Date__c
            └── Student__c/
                └── fields/
                    └── Placement_Status__c
```

---

*This README summarizes the official Day 6 documentation for the Salesforce Developer Bridge Program — Placement Management System.*
