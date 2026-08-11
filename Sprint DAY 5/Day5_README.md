# Salesforce Developer Bridge Program — Day 5
## Building Complete Business Transactions with SOQL, DML and Apex
**Project:** Placement Management System
**Student:** Sudharshan Kuppili
**PIN:** 24PA5A0517

---

## Overview

Day 5 extended the Placement Management System's `ApplicationService` Apex class into a complete, end-to-end business transaction: retrieving Student and Job information with SOQL, preventing duplicate applications, validating business rules, creating Placement Application records with DML, updating application status, and automatically recording an Interview Date when status changes to Interview Scheduled.

## Files in this submission

| File | Description |
|---|---|
| `Day5_Documentation.docx` | Full Word documentation (title page, student details, all sections, figures) |
| `Day5_Documentation.pdf` | PDF version of the same documentation |
| `README.md` | This file |

## What was implemented

1. **Student Information Retrieval** — SOQL query for Id, Name, CGPA, Branch, Active Backlogs, and Graduation Year, retrieving only what's needed for the eligibility decision.
2. **Job Information and Eligibility** — SOQL query for the Job's Minimum CGPA, Eligible Branch, Active Backlog Limit, and Application Deadline.
3. **Duplicate Application Prevention** — checks for an existing Placement Application for the same Student/Job combination before proceeding; tested and confirmed with a "Duplicate application found." debug result.
4. **Business Validation** — CGPA, Branch, Active Backlog, and Deadline checks, each returning immediately with a descriptive message on failure, all executed before any DML.
5. **Creating the Placement Application** — a validated application is inserted via DML; tested end-to-end, producing record PA-0005 with Status = Applied.
6. **Application Status Update** — `updateApplicationStatus()` retrieves and updates an existing record rather than creating a new one; tested by moving PA-0001 to Status = Shortlisted.
7. **Interview Date Enhancement** — `Interview_Date__c` field added; `updateApplicationStatus()` now sets it automatically to `Date.today()` whenever status becomes Interview Scheduled; tested on PA-0005.

## Notes on scope

- **Key Learnings section removed** at your request — the documentation now runs from Day Objective through Conclusion and the supporting-evidence appendix, without a separate learnings section.
- No placeholder screenshots were used. Every figure in the document is backed by an actual screenshot you provided.
- Two early, superseded screenshots (an initial skeleton version of `submitApplication()` containing only a placeholder debug line) were left out of the final document since the fuller, later versions of the same method make them redundant.
- The Day 3 "Placement Application Email Notification" flow screenshot you included in this batch was excluded — it's Day 3 Flow/automation work, not part of Day 5's Apex/SOQL/DML scope.
