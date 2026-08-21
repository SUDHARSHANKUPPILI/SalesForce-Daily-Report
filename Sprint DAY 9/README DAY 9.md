# Day 9 — Salesforce Placement Management

## Overview

Day 9 connected the Placement Management System's Salesforce backend to a working
student and administrator experience. Students can view eligible jobs, apply for
them, and track application status and interview details. Administrators can review
student applications and update status, interview date, and remarks. All planned
Day 9 tasks were completed and verified through the Salesforce UI and SOQL queries.

## Learning Objectives

- Build and deploy Lightning Web Components (LWC) backed by Apex controllers.
- Retrieve Salesforce data with `@AuraEnabled` methods and display it in an LWC.
- Compose multiple components into a single FlexiPage.
- Implement a user-initiated create action (Apply) from the UI through Apex.
- Handle duplicate-submission prevention and eligibility validation with clear user feedback.
- Build an administrator screen to update record status, dates, and text fields.
- Verify data changes using the Salesforce CLI (`sf data query`) and SOQL.

## Day 9 Tasks Completed

1. Placement Home / FlexiPage retrieval, verification, and deployment
2. Eligible Jobs Lightning Web Component (display and deployment)
3. Student job application flow (Apply → Placement_Application__c record)
4. Duplicate application validation
5. My Applications functionality
6. My Applications UI improvement (status badges, progression indicator)
7. Application Management (admin view of student applications)
8. Application status update
9. Interview Date
10. Remarks
11. Selected status test
12. Shortlisted status test
13. Rejected status test
14. End-to-end verification of the full workflow

## Placement Home

The `Placement_Home` FlexiPage is the student portal landing page. It was retrieved,
verified to contain the `placementHome`, `eligibleJobs`, `myApplications`, and
`adminApplications` components, and successfully redeployed via the Salesforce CLI.

## Eligible Jobs

The `eligibleJobs` LWC displays jobs a student can apply for — Job Name, Company,
Minimum CGPA, Eligible Branch, and Application Deadline — with an Apply button. Data
is retrieved from `Job__c` via the `EligibleJobsController` Apex class. The
component and controller were deployed and verified in the org.

## Job Application Flow

A student selects an eligible job and clicks **Apply**. The action calls Apex to
create a `Placement_Application__c` record, and the student sees an "Application
submitted successfully" confirmation. Applying for a job that does not meet the
student's CGPA requirement returns a validation error instead of creating a record.

## Duplicate Application Validation

Re-applying for a job the student has already applied to is blocked, with the toast
message **"You have already applied"** shown to the student, preventing a duplicate
`Placement_Application__c` record.

## My Applications

The `myApplications` LWC, backed by `MyApplicationsController`, loads and displays
the current student's applications: Job Name, Company, Applied On date, Status, and
Interview Date (when set).

## My Applications UI

The component was enhanced with a page subtitle ("Track your placement applications
and interview status"), clearer application cards, colour-coded status badges, and
a visual Applied → Shortlisted → Selected/Rejected progression indicator.

## Admin Application Management

The `adminApplications` LWC, backed by `AdminApplicationsController`, lets an
administrator view student applications (Student, Job, Company, Applied Date,
Current Status) and update Status, Interview Date, and Remarks via an Update
Application action, confirmed with an "Application updated successfully." toast.

## Application Status Management

| Status | Verified |
|---|---|
| Applied | Default status shown immediately after a successful application |
| Shortlisted | Status badge, progression indicator, and Interview Date verified |
| Selected | Status badge, progression indicator, and Interview Date verified |
| Rejected | Status display verified (rejected state handled correctly) |

## Interview Date and Remarks

Interview Date can be set from Application Management, is saved to the record, and
displays correctly in My Applications. Remarks entered by the administrator (e.g.
*"Technical Interview Scheduled"*, *"Selected after technical interview"*, *"low
cgpa"*) were saved successfully and verified using SOQL.

## Salesforce Components

| Component | Type / Role |
|---|---|
| `Placement_Home` | FlexiPage — student portal landing page |
| `eligibleJobs` | LWC — displays eligible jobs, handles Apply |
| `myApplications` | LWC — displays the student's own applications |
| `adminApplications` | LWC — administrator application management screen |
| `EligibleJobsController` | Apex controller — retrieves eligible `Job__c` records |
| `MyApplicationsController` | Apex controller — retrieves the student's applications |
| `AdminApplicationsController` | Apex controller — updates application status/details |
| `Placement_Application__c` | Custom object — stores each student's job application |

## Salesforce Concepts / APIs Used

- Lightning Web Components (LWC) — HTML, JS, and metadata configuration
- `@AuraEnabled` Apex methods for UI-to-Apex data access
- FlexiPages composed in Lightning App Builder
- Toast notifications for success and error feedback
- Salesforce CLI (`sf project deploy start`, `sf data query`, `sf apex run`)
- SOQL for data verification

## Testing & Validation

Application submission, eligibility validation, duplicate-application prevention,
status updates, interview date changes, and remarks were all tested through the
Salesforce UI. Resulting `Placement_Application__c` records were independently
verified with SOQL queries (`Student__c`, `Job__c`, `Application_Date__c`,
`Status__c`) run via the Salesforce CLI.

## End-to-End Flow

```
Eligible Job
     ↓
Apply
     ↓
Placement Application
     ↓
My Applications
     ↓
Admin Application Management
     ↓
Status / Interview Date / Remarks
     ↓
Updated My Applications
```

## Key Learnings

- Wiring an LWC to an Apex controller to display and act on Salesforce data.
- Implementing a complete student workflow: view → apply → track status.
- Designing and improving a component UI with status badges and a progression indicator.
- Building an administrator screen for status/date/text updates.
- Verifying functionality directly against Salesforce data with SOQL.
- Deploying FlexiPages, LWC bundles, and Apex classes with the Salesforce CLI.

## Final Status

**Day 9: 100% COMPLETE**

## Conclusion

Day 9 turned the Placement Management System's backend into a working, end-to-end
student and administrator experience — from viewing eligible jobs and applying, to
tracking and managing application status, interview dates, and remarks. All
functionality was tested through the UI and confirmed against Salesforce data via
SOQL.
