# Salesforce Developer Bridge Program — Day 3
## Validation Rules, Flows & Triggers
**Project:** Placement Management System
**Student:** Sudharshan Kuppili
**PIN:** 24PA5A0517

---

## Overview

Day 3 focused on Salesforce declarative automation for the Placement Management System: choosing between Validation Rules, Flows, and Apex Triggers for five business requirements, then building and testing the automations.

## Files in this submission

| File | Description |
|---|---|
| `Day3_Automation_Documentation.docx` | Full Word documentation (title page, student details, all sections, figures) |
| `Day3_Automation_Documentation.pdf` | PDF version of the same documentation |
| `README.md` | This file |

## 1. Which requirements did you solve using Flow?

- **Auto-fill Application Date** — a before-save Record-Triggered Flow ("Placement Application Date Flow") sets `Application_Date__c` to `$Flow.CurrentDate` when a Placement Application is created.
- **Send Email** — a separate after-save Record-Triggered Flow ("Placement Application Email Notification") sends a confirmation email when a Placement Application is created. It had to be a second flow because the Application Date flow runs before-save and cannot perform actions like sending email.
- **Create Offer Letter** — a Record-Triggered Flow ("Create Offer Letter When Selected") fires when a Placement Application is updated with Status = Selected (and Status Is Changed = True), and creates a related Offer Letter record with the Student, Placement Application, Offer Date, and Offer Status fields mapped automatically.

## 2. Which requirements required Validation Rules?

- **Reject low CGPA** — `Student_CGPA_Eligibility`: `Student__r.CGPA__c < Job__r.Minimum_CGPA__c`
- Plus four supporting data-quality rules built alongside it:
  - `Student_Must_Be_Selected` — `ISBLANK(Student__c)`
  - `Job_Must_Be_Selected` — `ISBLANK(Job__c)`
  - `Application_Date_Must_Be_Present` — `ISBLANK(Application_Date__c)`
  - `Application_Date_After_Deadline` — `Application_Date__c > Job__r.Application_Deadline__c`

## 3. Which requirements still needed Apex?

None of the five Day 3 requirements needed Apex — all were solved declaratively with Flow and Validation Rules. Apex Triggers remain the better choice for logic Flow can't handle well: complex multi-object transactions, bulk processing at scale (e.g. Data Loader imports of thousands of records), or fine-grained control over callouts to external systems.

## 4. Why did you choose those solutions?

Each requirement was matched to the simplest tool capable of solving it declaratively:
- Field population and simple "create a related record on condition" logic → **Flow** (Record-Triggered), since it's faster to build, easier to maintain, and doesn't require Apex governance overhead.
- Data-quality / hard-stop rules that must block a save outright → **Validation Rule**, since it runs before the record is committed and needs no separate automation to maintain.
- Apex was intentionally avoided everywhere a declarative tool could meet the requirement, per the "write less Apex when declarative tools can solve the problem" principle covered in the interview warm-up.

## Known items

A screenshot from the original Application Date Flow work-in-progress (showing the flow scoped to a "Job Application" object rather than "Placement Application," dated earlier than the rest of that flow's screenshots) was excluded from the final documentation, as it did not match the final implementation.

Email delivery to the inbox was not captured as a screenshot. The Email Automation section documents the Send Email action configuration, activation (V2), and the test record (PA-0003) created to trigger it — all backed by real screenshots — without claiming a delivered-email screenshot that isn't available.
