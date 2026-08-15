# Salesforce Developer Bridge Program — Day 2

**Student:** Sudharshan Kuppili
**Roll No.:** 24PA5A0517
**Branch:** Computer Science and Engineering (CSE)
**Program:** Salesforce Developer Bridge Program
**Project:** Hospital OPD Management System

---

## 📌 Overview

Day 2 of the Salesforce Developer Bridge Program focused on core Apex fundamentals applied to the **Hospital OPD Management System** — Apex Collections, Governor Limits & Bulkification, and the Trigger Handler Pattern, culminating in a real duplicate-validation business rule for patient appointments.

---

## 🎯 Day 2 Objectives

- Practice Apex Collections — `List`, `Set`, and `Map`.
- Understand Salesforce Governor Limits and why Apex code must be bulkified.
- Implement the Trigger Handler Pattern to separate business logic from Triggers.
- Build duplicate appointment validation using SOQL, `Trigger.new`, `Set`, and `addError()`.

---

## 🏥 Project

The **Hospital OPD Management System** manages Patients and their Appointments. Day 2's work centered on two custom objects — `Patient__c` and `Appointment__c` — and on ensuring the same patient cannot be booked for multiple appointments on the same date.

---

## 📚 Tasks Completed

### Task 1: Apex Collections

Practiced Apex Collections by working with `List`, `Set`, and `Map`:

- Created a **List** to store sample values (city names) and iterated through them using a loop.
- Used a **Set** to store unique Patient record IDs, and learned that duplicate values are automatically removed.
- Created a **Map** to store Patient records using their Id as the key, and retrieved records efficiently.
- Executed the code using **Execute Anonymous** and verified the output in the **Debug Log**.

**Learning Outcome:** How to use List, Set, and Map collections in Apex to efficiently store, retrieve, and manipulate data.

---

### Task 2: Governor Limits & Bulkification

Learned about Salesforce **Governor Limits** and why Apex code must be bulkified:

- Understood why SOQL queries should never be placed inside loops.
- Practiced bulkification by collecting record IDs using a **Set** and executing a single SOQL query outside the loop.
- Inserted multiple Patient records in a single bulk DML statement.
- Learned how bulkified code improves performance and follows Salesforce best practices.

**Learning Outcome:** How to write scalable Apex code by following Governor Limits and Bulkification best practices.

---

### Task 3: Trigger Handler Pattern

Implemented the **Trigger Handler Pattern**:

- Created an Apex Trigger named `AppointmentTrigger` that executes **before insert** on `Appointment__c` records.
- Instead of writing business logic inside the trigger, delegated to a separate handler class named `AppointmentTriggerHandler`.
- Wrote an `AppointmentTriggerHandlerTest` class and ran it in the Developer Console, which confirmed **100% code coverage** on both `AppointmentTrigger` and `AppointmentTriggerHandler`.

**Learning Outcome:** How to separate business logic from Apex Triggers using the Trigger Handler Pattern, improving code readability and maintainability.

---

### Task 4: Duplicate Appointment Validation

Implemented duplicate appointment validation using the `AppointmentTriggerHandler` class:

- The handler receives `Trigger.new` records.
- Collects Patient IDs using a **Set**.
- Queries existing Appointment records using **SOQL**.
- Compares **Patient** and **Appointment Date**.
- If a duplicate appointment exists, prevents the record from being saved using **`addError()`**.
- Ensures that the same patient cannot have multiple appointments on the same date.

**Learning Outcome:** How to use SOQL, `Trigger.new`, Sets, and `addError()` to implement business validation in Salesforce Apex.

---

## 🏗️ Architecture

```text
AppointmentTrigger
        ↓
AppointmentTriggerHandler
        ↓
Duplicate Appointment Validation
        ↓
Appointment__c
```

`AppointmentTrigger` fires on `before insert` and immediately delegates to `AppointmentTriggerHandler.preventDuplicateAppointments()`, which performs the duplicate check against `Appointment__c` and blocks invalid records with `addError()`.

---

## 💻 Code / Implementation

### AppointmentTrigger

```apex
trigger AppointmentTrigger on Appointment__c (before insert) {
    AppointmentTriggerHandler.preventDuplicateAppointments(Trigger.new);
}
```

### AppointmentTriggerHandler — Duplicate Validation

```apex
public class AppointmentTriggerHandler {

    public static void preventDuplicateAppointments(List<Appointment__c> newAppointments){

        Set<Id> patientIds = new Set<Id>();

        for(Appointment__c app : newAppointments){
            if(app.Patient__c != null){
                patientIds.add(app.Patient__c);
            }
        }

        List<Appointment__c> existingAppointments = [
            SELECT Id, Patient__c, Appointment_Date__c
            FROM Appointment__c
            WHERE Patient__c IN :patientIds
        ];

        for(Appointment__c newApp : newAppointments){
            for(Appointment__c oldApp : existingAppointments){
                if(newApp.Patient__c == oldApp.Patient__c &&
                   newApp.Appointment_Date__c == oldApp.Appointment_Date__c){

                    newApp.addError('This patient already has an appointment on this date.');
                }
            }
        }
    }
}
```

---

## 🧪 Testing

- `AppointmentTriggerHandlerTest` class was written and run in the Developer Console.
- Result: **100% code coverage** confirmed on both `AppointmentTrigger` and `AppointmentTriggerHandler`.

---

## 📸 Evidence / Screenshots

The Day 2 documentation includes the following Developer Console screenshots as evidence:

| Figure | Description |
|---|---|
| Figure 1 | List of City Names — Execute Anonymous & Debug Log |
| Figure 2 | Set of Unique Patient IDs — Execute Anonymous & Debug Log |
| Figure 3 | Map of Patient Records by Id — Execute Anonymous & Debug Log |
| Figure 4 | Bulk Insert of Multiple Patient Records in a Single DML Statement |
| Figure 5 | AppointmentTrigger Code — 100% Code Coverage on Trigger and Handler |
| Figure 6 | AppointmentTriggerHandler — Duplicate Check Logic |

*(Screenshots are embedded in the original Day 2 documentation PDF: `Day2_Documentation_Sudharshan.pdf`.)*

---

## 🧠 Key Learnings

- How to use `List`, `Set`, and `Map` collections in Apex to store, retrieve, and manipulate data.
- Why SOQL queries should never be placed inside loops.
- How to bulkify Apex code by collecting IDs in a Set and querying once outside the loop.
- How to separate business logic from a Trigger using the Trigger Handler Pattern.
- How to write a test class that achieves 100% code coverage on a Trigger and its handler.
- How to use `Trigger.new`, SOQL, Sets, and `addError()` together to implement business validation (preventing duplicate appointments for the same patient on the same date).

---

*This README summarizes the Day 2 documentation for the Salesforce Developer Bridge Program — Hospital OPD Management System.*
