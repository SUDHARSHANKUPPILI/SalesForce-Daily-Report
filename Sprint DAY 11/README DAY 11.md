# Placement Management System — Day 11

## Overview

Day 11 focused on completing, verifying, deploying, and testing the student-facing side of the
Placement Management System in the Salesforce org. Work centered on the Student Dashboard, My
Applications, live data verification, duplicate-application handling, the Student Portal permission
set, deployment verification, and Apex testing.

## Day 11 Objectives

- Verify the Student Dashboard (`studentDashboard` LWC) against live org data.
- Verify My Applications (`myApplications` LWC) against live org data.
- Confirm the Student__c and Placement_Application__c records via Salesforce CLI queries.
- Confirm duplicate-application validation is working correctly.
- Create, deploy, and assign the Student Portal permission set.
- Verify the latest deployment to PlacementOrg.
- Run and pass local Apex tests.

## Features Verified

### Student Dashboard

`StudentDashboardController.getDashboardData()` (called via `@wire`) powers the `studentDashboard`
LWC, which was confirmed to display:

- Welcome message using the logged-in student's name
- Total Applications count
- Applied, Shortlisted, Selected, and Rejected counts
- Upcoming Interview section
- Quick Navigation — Eligible Jobs, My Applications, Student Profile

### My Applications

`MyApplicationsController` retrieves the `Student__c` record owned by the current Salesforce user,
then that student's `Placement_Application__c` records. The `myApplications` LWC was confirmed to
display Job name, Company name, Application date, Interview date (when available), current status,
status timeline, and Remarks (when available).

### Duplicate Application Handling

Re-applying to a job already applied for correctly returns:

> "You have already applied for this job."

This confirms the validation prevents duplicate `Placement_Application__c` records.

## Data Verification

**Student__c record**

| Field | Value |
|---|---|
| Name | Sudharshan Kuppili |
| CGPA | 8.75 |
| Branch | CSE |
| Record Id | a08NS00001UpMRcYAN |

**Placement_Application__c records (7 total)**

| Job | Status | Applied On | Interview Date |
|---|---|---|---|
| Data Analyst - Apply Test | Selected | 2026-08-22 | 2026-08-29 |
| Software Engineer - Eligible Test | Shortlisted | 2026-08-21 | 2026-08-25 |
| Java Developer Test | Shortlisted | 2026-08-11 | 2026-08-24 |
| Software Engineer | Rejected | 2026-08-08 | null |
| Software Engineer | Selected | 2026-08-08 | null |
| Software Engineer | Applied | 2026-08-08 | null |
| Software Engineer | Selected | 2026-08-07 | null |

Verified via Salesforce CLI SOQL queries against `PlacementOrg`.

## Student Portal Permission Set

| Attribute | Value |
|---|---|
| Metadata file | `Student_Portal.permissionset-meta.xml` |
| Label | Student Portal Access |
| Apex class access | `EligibleJobsController`, `MyApplicationsController`, `StudentProfileController` |
| Deployment status | Deployed successfully |
| Assignment status | Assigned to the Salesforce user |

## Deployment

Verified via `sf project deploy report --use-most-recent --target-org PlacementOrg`.

- **Target org:** `PlacementOrg` (`sudharshan5@vishnu.edu.in`)
- **Status:** Succeeded
- **Components Deployed:** 52/52
- **Component Errors:** 0
- **Test Errors:** 0
- **Deploy Id:** `0AfNS00000kJRZt0AO`
- **Zip Size:** 120,673 bytes

## Apex Testing

```
sf apex run test --target-org PlacementOrg --test-level RunLocalTests --wait 10
```

| Test | Outcome | Runtime (ms) |
|---|---|---|
| `AppointmentTriggerHandlerTest.testDuplicateAppointment` | Pass | 201 |

**Test Summary:** Tests Ran 1 · Pass Rate 100% · Fail Rate 0% · Skip Rate 0%
Org Id `00DNS00000xQvez2AC` · Username `sudharshan5@vishnu.edu.in`

## Evidence Summary

| Task | Evidence Type | Source |
|---|---|---|
| Student Dashboard | UI screenshot | `Screenshot_2026-08-23_185406.png` |
| My Applications | UI screenshot | `Screenshot_2026-08-23_182930.png` |
| Student + Application data (7 records) | Terminal (CLI query) | `Screenshot_2026-08-23_184514.png` |
| Duplicate application handling | UI screenshot (error toast) | `Screenshot_2026-08-23_184718.png` |
| Student Portal permission set | Terminal (metadata diff) | `Screenshot_2026-08-23_203802.png` |
| Apex test result | Terminal (test run output) | `Screenshot_2026-08-23_203802.png` |
| Deployment status (52/52, Succeeded, 0 errors) | Terminal (deploy report output) | `1787499515154_image.png` |

## Final Result

Day 11 completed verification of the student-facing placement functionality. Application data was
confirmed directly in Salesforce, the Student Portal permission set was configured and assigned,
deployment to `PlacementOrg` completed without errors, and the available Apex test passed
successfully. The student portal is ready for the next stage of the project.
