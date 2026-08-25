# Day 12 – Student Portal Security & Final Validation

**Student Name:** Sudharshan Kuppili
**PIN No.:** 24PA5A0517
**Project:** Placement Management System
**Platform:** Salesforce (Developer Edition)
**Target Org Alias:** PlacementOrg
**Source API Version:** 67.0

---

## Objective

Day 12 marks the final validation phase of the project. Rather than building new
features, the focus was on confirming that everything implemented across prior
days is correctly deployed, secured, and functioning as expected:

1. Student Portal security validation
2. Permission Set verification
3. Apex class access verification
4. Permission Set assignment verification
5. Deployment verification
6. Apex test verification
7. Student data verification
8. Placement application data verification
9. Student Portal UI validation
10. Final evidence collection

---

## Project Configuration

**Lightning Web Components**
- adminApplications
- adminDashboard
- adminJobs
- eligibleJobs
- myApplications
- placementHome
- studentDashboard
- studentHome
- studentProfile

**Apex Classes**
- AdminApplicationsController
- AdminDashboardController
- AdminJobsController
- ApplicationService
- EligibleJobsController
- MyApplicationsController
- PlacementFutureService
- PlacementHomeController
- PlacementQueueableService
- PlacementScheduledService
- PlacementStudentBatch
- StudentDashboardController
- StudentProfileController

**Custom Objects**
- Job__c
- Placement_Application__c
- Student__c

**Permission Set**
- `Student_Portal` — Label: *Student Portal Access*

---

## 1. Deployment Validation

**Command:**
```
sf project deploy report --use-most-recent --target-org PlacementOrg
```

**Result:**

| Metric | Value |
|---|---|
| Deployment ID | 0AfNS00000kJRZt0AO |
| Status | Succeeded |
| Components Deployed | 52 / 52 |
| Component Errors | 0 |
| Test Errors | 0 |

The most recent deployment to PlacementOrg completed successfully with zero
component or test errors, confirming all metadata built across the project is
live in the target org.

---

## 2. Apex Test Validation

**Command:**
```
sf apex run test --target-org PlacementOrg --test-level RunLocalTests --wait 10
```

**Result:**

| Test | Outcome |
|---|---|
| AppointmentTriggerHandlerTest.testDuplicateAppointment | Passed |

| Metric | Value |
|---|---|
| Tests Ran | 1 |
| Pass Rate | 100% |
| Fail Rate | 0% |
| Skip Rate | 0% |

---

## 3. Student Data Validation

The `Student__c` record associated with the current Salesforce user was queried
and verified:

| Field | Value |
|---|---|
| Student Name | Sudharshan Kuppili |
| PIN No. | 24PA5A0517 |
| Branch | CSE |
| CGPA | 8.75 |

---

## 4. Placement Application Data Validation

Placement application records for the student were queried and verified:

- **Total Applications:** 7
- **Statuses Present:** Applied, Shortlisted, Selected, Rejected
- **Interview Dates:** Present for applicable applications

---

## 5. Student Portal Security Validation

A consolidated validation script confirmed the Student_Portal Permission Set
end-to-end — from Permission Set existence, to Apex class access, to actual
user assignment.

### 5.1 Permission Set Verification
```
sf data query --query "SELECT Id, Name, Label FROM PermissionSet WHERE Name = 'Student_Portal'" --target-org PlacementOrg
```

| Id | Name | Label |
|---|---|---|
| 0PSNS00000XQ8zr4AD | Student_Portal | Student Portal Access |

### 5.2 Apex Class Access Verification
```
sf data query --query "SELECT Id, SetupEntityId, SetupEntityType FROM SetupEntityAccess WHERE ParentId = '0PSNS00000XQ8zr4AD' AND SetupEntityType = 'ApexClass'" --target-org PlacementOrg
```

The Student_Portal Permission Set grants access to exactly three Apex classes:

- EligibleJobsController
- MyApplicationsController
- StudentProfileController

### 5.3 User Assignment Verification
```
sf data query --query "SELECT Id, Assignee.Name, PermissionSet.Name FROM PermissionSetAssignment WHERE AssigneeId = '005NS000013DZKHYA4' AND PermissionSet.Name = 'Student_Portal'" --target-org PlacementOrg
```

| Assignee | Permission Set |
|---|---|
| Kuppili Sudharshan | Student_Portal |

**Result:** The Student_Portal Permission Set exists, is scoped to exactly the
three student-facing Apex controllers, and is assigned to the student user.

---

## 6. Student Portal UI Validation

The major Student Portal sections were opened and visually verified against
the security configuration above:

- **Student Dashboard & Eligible Jobs** — Placement Dashboard statistics
  (Total Applications, Applied, Shortlisted, Selected) and the Eligible Jobs
  listing rendered correctly for the logged-in student.
- **My Applications** — Application cards (e.g., *Data Analyst - Apply Test*)
  display applied date, interview date, and status tracker, backed by
  `MyApplicationsController`.
- **Student Profile** — Displays Student Name, CGPA, and Branch consistent
  with the verified `Student__c` record, backed by `StudentProfileController`.

---

## Screenshot Evidence

| # | Screenshot | Description |
|---|---|---|
| 1 | Student Portal Security Validation | Permission Set, Apex class access, and user assignment confirmed in a single script run |
| 2 | Student Dashboard | Placement Dashboard statistics for the logged-in student |
| 3 | Eligible Jobs | Eligible Jobs section (same screenshot as Student Dashboard) |
| 4 | My Applications | Data Analyst - Apply Test application, Selected status |
| 5 | Student Profile | Student Name, CGPA, Branch fields |
| 6 | Deployment Report | 52/52 components deployed, 0 errors |
| 7 | Apex Test Run | 100% pass rate on local tests |

*Note: The Student Dashboard and Eligible Jobs sections appear together in a
single screenshot, so that screenshot is referenced for both items.*

---

## Day 12 Validation Checklist

- [x] Successful 52/52 component deployment (0 errors)
- [x] 100% Apex test pass result
- [x] Student record validation (Sudharshan Kuppili, CSE, CGPA 8.75)
- [x] 7 placement applications verified across Applied/Shortlisted/Selected/Rejected statuses
- [x] Student_Portal Permission Set confirmed in org
- [x] EligibleJobsController access confirmed
- [x] MyApplicationsController access confirmed
- [x] StudentProfileController access confirmed
- [x] Student Portal user assignment confirmed (Kuppili Sudharshan)
- [x] Student Dashboard UI validated
- [x] Eligible Jobs UI validated
- [x] My Applications UI validated
- [x] Student Profile UI validated
- [x] Full screenshot evidence collected

---

## Summary

Day 12 confirms that the Student Portal of the Placement Management System is
correctly deployed, properly secured at the Permission Set and Apex class
level, and fully functional end-to-end for the student user. This closes out
the validation phase of the project with backend security, deployment
integrity, Apex test health, and UI behavior all verified against the same
target org.
