# UTMS - Undergraduate Transfer Management System
## 06 - Business Rules & Workflow Logic

**Version:** 1.0  
**Last Updated:** March 2026

---

## 1. TRANSFER GRADE CALCULATION

### 1.1 Formula

```
Transfer Grade (TG) = (x/y × 100 × 0.9) + (0.1 × z)

Where:
  x = Applicant's YKS score
  y = Department's minimum YKS score (taban puan)
  z = Applicant's GPA (4.00 scale)
```

### 1.2 Calculation Rules

| Rule ID | Rule | Example |
|---------|------|---------|
| TG-001 | YKS score must be greater than 0 | Invalid: 0, Valid: 425.5 |
| TG-002 | Department min YKS score must be configured | Must be > 0 |
| TG-003 | GPA must be between 0.00 and 4.00 | Invalid: 4.50, Valid: 3.45 |
| TG-004 | Result is rounded to 4 decimal places | 87.5432 |
| TG-005 | Higher transfer grade = Better ranking | 105.23 > 98.45 |

### 1.3 Example Calculations

**Example 1: Strong Candidate**
```
YKS Score: 450.00
Min YKS Score: 380.00
GPA: 3.80

TG = (450/380 × 100 × 0.9) + (0.1 × 3.80)
TG = (1.1842 × 100 × 0.9) + 0.38
TG = 106.58 + 0.38
TG = 106.96
```

**Example 2: Average Candidate**
```
YKS Score: 400.00
Min YKS Score: 380.00
GPA: 2.50

TG = (400/380 × 100 × 0.9) + (0.1 × 2.50)
TG = (1.0526 × 100 × 0.9) + 0.25
TG = 94.74 + 0.25
TG = 94.99
```

---

## 2. ELIGIBILITY RULES

### 2.1 Mandatory Requirements

| Rule ID | Requirement | Validation |
|---------|-------------|------------|
| EL-001 | GPA ≥ Department minimum GPA | Applicant GPA must meet or exceed department's min_gpa |
| EL-002 | YKS Ranking ≤ Department max ranking | If department has max_ranking, applicant must be within |
| EL-003 | YKS Score ≥ Department min score | If department has min_yks_score, applicant must meet |
| EL-004 | Semester requirement | Must be enrolled in at least 2nd semester |
| EL-005 | No disciplinary action | Must have clean disciplinary record |

### 2.2 Language Requirements

| Rule ID | Requirement | Details |
|---------|-------------|---------|
| LR-001 | Language certificate required | Most departments require language proficiency |
| LR-002 | Minimum score: 60 | Default minimum language score |
| LR-003 | Accepted certificates | YDS, YÖKDİL, TOEFL, IELTS, PTE |
| LR-004 | YDYO evaluation | Final decision by YDYO staff |

### 2.3 Score Equivalencies

| Certificate | Minimum Score | Equivalent |
|-------------|---------------|------------|
| YDS | 60 | Passes |
| YÖKDİL | 60 | Passes |
| TOEFL IBT | 72 | Passes |
| IELTS | 6.0 | Passes |
| PTE Academic | 55 | Passes |

---

## 3. APPLICATION STATUS TRANSITIONS

### 3.1 Status State Machine

```
                                    ┌──────────────┐
                                    │    DRAFT     │
                                    └──────┬───────┘
                                           │ submit()
                                           ▼
                                    ┌──────────────┐
                                    │  SUBMITTED   │
                                    └──────┬───────┘
                                           │
                                           ▼
                              ┌────────────────────────┐
                              │ DOCUMENTS_UNDER_REVIEW │
                              └────────────┬───────────┘
                                           │
                         ┌─────────────────┼─────────────────┐
                         ▼                 │                 ▼
              ┌──────────────────┐         │      ┌──────────────────┐
              │DOCUMENTS_REJECTED│         │      │DOCUMENTS_APPROVED│
              └──────────────────┘         │      └────────┬─────────┘
                      │                    │               │
                      ▼                    │               ▼
               [TERMINAL]                  │    ┌─────────────────────────┐
                                           │    │PENDING_LANGUAGE_EVALUATION│
                                           │    └────────────┬────────────┘
                                           │                 │
                                           │    ┌────────────┴────────────┐
                                           │    ▼                         ▼
                                           │ ┌─────────────────┐  ┌─────────────────┐
                                           │ │LANGUAGE_REJECTED│  │LANGUAGE_APPROVED│
                                           │ └─────────────────┘  └────────┬────────┘
                                           │         │                     │
                                           │         ▼                     ▼
                                           │  [TERMINAL]        ┌─────────────────────────┐
                                           │                    │PENDING_ACADEMIC_EVALUATION│
                                           │                    └────────────┬────────────┘
                                           │                                 │
                                           │                                 ▼
                                           │                    ┌─────────────────┐
                                           │                    │ PENDING_INTIBAK │
                                           │                    └────────┬────────┘
                                           │                             │
                                           │                             ▼
                                           │                    ┌─────────────────┐
                                           │                    │INTIBAK_COMPLETED│
                                           │                    └────────┬────────┘
                                           │                             │
                                           │                             ▼
                                           │                    ┌────────────────────────┐
                                           │                    │PENDING_FACULTY_DECISION│
                                           │                    └────────────┬───────────┘
                                           │                                 │
                                           │                    ┌────────────┼────────────┐
                                           │                    ▼            │            ▼
                                           │              ┌──────────┐       │      ┌──────────┐
                                           │              │   ASIL   │       │      │  YEDEK   │
                                           │              └──────────┘       │      └──────────┘
                                           │                    │            │            │
                                           │                    ▼            ▼            ▼
                                           │              [TERMINAL]   ┌──────────┐ [TERMINAL]
                                           │                           │ REJECTED │
                                           │                           └──────────┘
                                           │                                 │
                                           │                                 ▼
                                           │                           [TERMINAL]
                                           │
                    At any non-terminal state
                                           │
                                           ▼
                                    ┌──────────────┐
                                    │  WITHDRAWN   │
                                    └──────────────┘
                                           │
                                           ▼
                                     [TERMINAL]
```

### 3.2 Status Transition Rules

| From Status | To Status | Trigger | Actor |
|-------------|-----------|---------|-------|
| DRAFT | SUBMITTED | Application submission | Student |
| SUBMITTED | DOCUMENTS_UNDER_REVIEW | Auto on submission | System |
| DOCUMENTS_UNDER_REVIEW | DOCUMENTS_APPROVED | All docs approved | OIDB Staff |
| DOCUMENTS_UNDER_REVIEW | DOCUMENTS_REJECTED | Any doc rejected | OIDB Staff |
| DOCUMENTS_APPROVED | PENDING_LANGUAGE_EVALUATION | Forward to YDYO | OIDB Staff |
| PENDING_LANGUAGE_EVALUATION | LANGUAGE_APPROVED | Language passed | YDYO Staff |
| PENDING_LANGUAGE_EVALUATION | LANGUAGE_REJECTED | Language failed | YDYO Staff |
| LANGUAGE_APPROVED | PENDING_ACADEMIC_EVALUATION | Forward to Dean | OIDB Staff |
| PENDING_ACADEMIC_EVALUATION | PENDING_INTIBAK | Forward to YGK | Dean's Office |
| PENDING_INTIBAK | INTIBAK_COMPLETED | Intibak done | YGK Member |
| INTIBAK_COMPLETED | PENDING_FACULTY_DECISION | Forward to board | YGK Member |
| PENDING_FACULTY_DECISION | ASIL/YEDEK/REJECTED | Final decision | Faculty Board |
| Any non-terminal | WITHDRAWN | Withdrawal request | Student |

---

## 4. WORKFLOW STAGES

### 4.1 Stage Definitions

| Stage | Responsible Role | Actions Available | Next Stage |
|-------|------------------|-------------------|------------|
| OIDB_INITIAL | OIDB Staff | Review docs, Approve/Reject, Request more docs | YDYO_EVALUATION |
| YDYO_EVALUATION | YDYO Staff | Evaluate language, Approve/Reject | OIDB_POST_LANGUAGE |
| OIDB_POST_LANGUAGE | OIDB Staff | Verify, Calculate grade, Forward | DEANS_OFFICE_REVIEW |
| DEANS_OFFICE_REVIEW | Dean's Office | Review, Forward to YGK | YGK_EVALUATION |
| YGK_EVALUATION | YGK Member | Prepare intibak, Complete | DEANS_OFFICE_FINAL |
| DEANS_OFFICE_FINAL | Faculty Board | Final decision (ASIL/YEDEK/REJECT) | OIDB_FINAL |
| OIDB_FINAL | OIDB Staff | Publish results | COMPLETED |
| COMPLETED | - | - | - |

### 4.2 Stage Processing Order

```
1. OIDB_INITIAL        → Document verification
2. YDYO_EVALUATION     → Language evaluation
3. OIDB_POST_LANGUAGE  → Post-language processing, grade calculation
4. DEANS_OFFICE_REVIEW → Academic review
5. YGK_EVALUATION      → Intibak preparation (course equivalence)
6. DEANS_OFFICE_FINAL  → Faculty board decision
7. OIDB_FINAL          → Result publication
8. COMPLETED           → Process complete
```

### 4.3 Stage Duration Expectations

| Stage | Expected Duration | Maximum Duration |
|-------|-------------------|------------------|
| OIDB_INITIAL | 2-3 days | 5 days |
| YDYO_EVALUATION | 1-2 days | 3 days |
| OIDB_POST_LANGUAGE | 1 day | 2 days |
| DEANS_OFFICE_REVIEW | 1-2 days | 3 days |
| YGK_EVALUATION | 3-5 days | 7 days |
| DEANS_OFFICE_FINAL | 1-2 days | 3 days |
| OIDB_FINAL | 1 day | 2 days |
| **Total** | **10-16 days** | **25 days** |

---

## 5. DOCUMENT RULES

### 5.1 Required Documents

| Document Type | Required | Description | Validation |
|---------------|----------|-------------|------------|
| TRANSCRIPT | Yes | Official transcript | Must show GPA, courses |
| YKS_RESULT | Yes | YKS exam result | Must match claimed score |
| ID_CARD | Yes | National ID copy | Clear, readable |
| STUDENT_CERTIFICATE | Yes | Current enrollment | Recent date |
| DISCIPLINARY_CERTIFICATE | Yes | Clean record | No actions listed |
| COURSE_CONTENT | Yes | Course syllabi | For intibak |
| LANGUAGE_CERTIFICATE | Conditional | If required by dept | Valid certificate |
| PORTFOLIO | Conditional | Design departments | If required |

### 5.2 Document Upload Rules

| Rule ID | Rule | Value |
|---------|------|-------|
| DOC-001 | Maximum file size | 15 MB |
| DOC-002 | Allowed file types | PDF, JPG, JPEG, PNG |
| DOC-003 | One document per type | Cannot upload duplicate types |
| DOC-004 | Upload only in DRAFT status | Cannot upload after submission |
| DOC-005 | Delete only in DRAFT status | Cannot delete after submission |

### 5.3 Document Review Rules

| Rule ID | Rule | Details |
|---------|------|---------|
| REV-001 | Only OIDB can review | Other roles cannot change document status |
| REV-002 | Review statuses | APPROVED, REJECTED, REQUIRES_RESUBMISSION |
| REV-003 | Comments required for rejection | Must provide reason |
| REV-004 | All docs must be approved | Before forwarding to next stage |
| REV-005 | Re-upload allowed | If REQUIRES_RESUBMISSION status |

---

## 6. RANKING RULES

### 6.1 Ranking Calculation

| Rule ID | Rule | Details |
|---------|------|---------|
| RNK-001 | Sort by transfer grade | Descending order (highest first) |
| RNK-002 | Same grade = same rank | Ties get same position |
| RNK-003 | Quota determines ASIL count | Top N candidates are ASIL |
| RNK-004 | Remaining are YEDEK | After quota filled |
| RNK-005 | Only eligible candidates ranked | Must pass all requirements |

### 6.2 Ranking Categories

| Category | Definition | Next Steps |
|----------|------------|------------|
| ASIL (Primary) | Within quota, accepted | Registration process |
| YEDEK (Waitlist) | Outside quota, waiting | Wait for ASIL withdrawal |
| REJECTED | Did not meet requirements | Process ends |

### 6.3 YEDEK Promotion Rules

| Rule ID | Rule | Details |
|---------|------|---------|
| YDK-001 | ASIL withdrawal triggers review | Check next YEDEK |
| YDK-002 | YEDEK promoted in order | Position 6 before position 7 |
| YDK-003 | Notification sent | Email and in-app notification |
| YDK-004 | Registration deadline | Must respond within X days |

---

## 7. INTIBAK (COURSE EQUIVALENCE) RULES

### 7.1 Course Matching Rules

| Rule ID | Rule | Details |
|---------|------|---------|
| INT-001 | Credit comparison | Source credits ≥ Target credits |
| INT-002 | Content similarity | 70%+ content match required |
| INT-003 | Grade requirement | Passing grade in source course |
| INT-004 | ECTS consideration | Similar ECTS preferred |
| INT-005 | Manual evaluation | YGK member decides each course |

### 7.2 Intibak Decisions

| Decision | Meaning | Credit Transfer |
|----------|---------|-----------------|
| ACCEPTED | Full equivalence | Full credits transferred |
| CONDITIONAL | Partial match | May need additional work |
| REJECTED | No equivalence | Must retake course |
| PENDING | Not yet decided | Awaiting evaluation |

### 7.3 Intibak Completion Rules

| Rule ID | Rule | Details |
|---------|------|---------|
| INT-C01 | All courses must have decision | No PENDING allowed |
| INT-C02 | Summary generated | Total accepted credits calculated |
| INT-C03 | Forward to faculty board | After completion |
| INT-C04 | Record preserved | For student registration |

---

## 8. NOTIFICATION RULES

### 8.1 Automatic Notifications

| Event | Notification Type | Recipients | Priority |
|-------|-------------------|------------|----------|
| Application submitted | APPLICATION_SUBMITTED | Student | HIGH |
| Document approved | DOCUMENT_APPROVED | Student | NORMAL |
| Document rejected | DOCUMENT_REJECTED | Student | HIGH |
| Stage changed | STAGE_CHANGED | Student | NORMAL |
| Language result | LANGUAGE_RESULT | Student | HIGH |
| Final decision | FINAL_DECISION | Student | URGENT |
| YEDEK promoted | RANKING_RESULT | Student | URGENT |

### 8.2 Email Notifications

| Event | Email Sent | Template |
|-------|------------|----------|
| Registration | Yes | welcome |
| Password reset request | Yes | password-reset |
| Password changed | Yes | password-changed |
| Application submitted | Yes | application-submitted |
| Document rejected | Yes | document-rejected |
| Language result | Yes | language-result |
| Final decision | Yes | final-decision |

### 8.3 Notification Retention

| Type | Retention Period | After Period |
|------|------------------|--------------|
| All notifications | 1 year | Soft delete |
| Read notifications | 6 months | Archive |
| Unread notifications | 1 year | Keep active |

---

## 9. USER ACCOUNT RULES

### 9.1 Password Rules

| Rule ID | Rule | Requirement |
|---------|------|-------------|
| PWD-001 | Minimum length | 8 characters |
| PWD-002 | Uppercase required | At least 1 |
| PWD-003 | Lowercase required | At least 1 |
| PWD-004 | Digit required | At least 1 |
| PWD-005 | Special char required | At least 1 (@$!%*?&) |
| PWD-006 | Password history | Cannot reuse last 5 |

### 9.2 Account Lockout Rules

| Rule ID | Rule | Value |
|---------|------|-------|
| LOCK-001 | Failed attempts threshold | 5 attempts |
| LOCK-002 | Lockout action | Account locked |
| LOCK-003 | Unlock method | Admin unlock or password reset |
| LOCK-004 | Counter reset | On successful login |

### 9.3 Session Rules

| Rule ID | Rule | Value |
|---------|------|-------|
| SES-001 | Access token expiry | 30 minutes |
| SES-002 | Refresh token expiry | 7 days |
| SES-003 | Single session | Multiple devices allowed |
| SES-004 | Token refresh | Before expiry |

---

## 10. AUTHORIZATION RULES

### 10.1 Role-Based Access

| Resource | STUDENT | OIDB | DEAN | YGK | YDYO | ADMIN |
|----------|---------|------|------|-----|------|-------|
| Own applications | CRUD | R | R | R | R | CRUD |
| All applications | - | RU | RU | RU | RU | CRUD |
| Documents (own) | CRD | R | R | R | R | CRUD |
| Documents (review) | - | U | - | - | - | U |
| Language evaluation | - | - | - | - | U | U |
| Intibak | - | - | - | CRU | - | CRUD |
| Final decision | - | - | U | - | - | U |
| Users | - | - | - | - | - | CRUD |
| System settings | - | - | - | - | - | CRUD |

### 10.2 Stage-Based Access

| Stage | Who Can Access | Who Can Act |
|-------|---------------|-------------|
| OIDB_INITIAL | OIDB, Admin | OIDB, Admin |
| YDYO_EVALUATION | YDYO, OIDB, Admin | YDYO, Admin |
| OIDB_POST_LANGUAGE | OIDB, Admin | OIDB, Admin |
| DEANS_OFFICE_REVIEW | Dean, OIDB, Admin | Dean, Admin |
| YGK_EVALUATION | YGK, Dean, Admin | YGK, Admin |
| DEANS_OFFICE_FINAL | Dean, Admin | Dean, Admin |
| OIDB_FINAL | OIDB, Admin | OIDB, Admin |

---

## 11. VALIDATION RULES

### 11.1 Application Validation

| Field | Validation | Error Message |
|-------|------------|---------------|
| departmentId | Required, must exist | Department not found |
| applicationTerm | FALL or SPRING | Invalid application term |
| sourceUniversity | Required, max 255 | Source university is required |
| sourceDepartment | Required, max 255 | Source department is required |
| currentSemester | 1-14 | Semester must be between 1 and 14 |
| yksScore | Required, > 0 | YKS score must be positive |
| yksRanking | Required, > 0 | YKS ranking must be positive |
| gpa | Required, 0.00-4.00 | GPA must be between 0.00 and 4.00 |

### 11.2 User Validation

| Field | Validation | Error Message |
|-------|------------|---------------|
| email | Required, valid format, unique | Invalid email format / Email already exists |
| password | Required, 8+ chars, complexity | Password does not meet requirements |
| firstName | Required, 2-100 chars | First name is required |
| lastName | Required, 2-100 chars | Last name is required |
| phone | Turkish format (+90...) | Invalid phone number format |
| kvkkConsent | Must be true | KVKK consent is required |

---

## 12. ERROR CODES

### 12.1 Authentication Errors

| Code | Message | HTTP Status |
|------|---------|-------------|
| AUTH_001 | Invalid email or password | 401 |
| AUTH_002 | Token expired | 401 |
| AUTH_003 | Invalid token | 401 |
| AUTH_004 | Account locked | 403 |
| AUTH_005 | Account not active | 403 |
| AUTH_006 | KVKK consent required | 403 |

### 12.2 Application Errors

| Code | Message | HTTP Status |
|------|---------|-------------|
| APP_001 | Application not found | 404 |
| APP_002 | Application already exists for this term | 409 |
| APP_003 | Application period closed | 422 |
| APP_004 | Invalid status transition | 422 |
| APP_005 | Cannot update submitted application | 422 |
| APP_006 | Required documents missing | 422 |

### 12.3 Document Errors

| Code | Message | HTTP Status |
|------|---------|-------------|
| DOC_001 | Document not found | 404 |
| DOC_002 | Invalid file type | 400 |
| DOC_003 | File size exceeded | 400 |
| DOC_004 | Document type already uploaded | 409 |
| DOC_005 | Cannot delete from submitted application | 422 |

### 12.4 Workflow Errors

| Code | Message | HTTP Status |
|------|---------|-------------|
| WF_001 | Invalid workflow action | 422 |
| WF_002 | Not authorized for this stage | 403 |
| WF_003 | Application not at expected stage | 422 |
| WF_004 | Cannot proceed without completing current step | 422 |

---

## 13. REVISION HISTORY

| Version | Date | Author | Changes |
|---------|------|--------|---------|
| 1.0 | Mar 2026 | Group 05 | Initial version |
