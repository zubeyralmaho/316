# UTMS - Undergraduate Transfer Management System
## 07 - Use Cases

**Version:** 1.0  
**Last Updated:** March 2026

---

## 1. USE CASE OVERVIEW

### 1.1 Actor Summary

| Actor | Description | Primary Use Cases |
|-------|-------------|-------------------|
| **Applicant Student** | University student applying for transfer | UC-1.x (Authentication), UC-2.x (Application) |
| **OIDB Staff** | Registrar's Office personnel | UC-3.x (Document Review), UC-4.x (Workflow) |
| **YDYO Staff** | Foreign Languages School personnel | UC-5.x (Language Evaluation) |
| **Dean's Office Staff** | Faculty administrative staff | UC-6.x (Academic Review) |
| **YGK Member** | Transfer Committee member | UC-7.x (Intibak) |
| **System Admin** | System administrator | UC-8.x (Administration) |

### 1.2 Use Case Categories

```
UC-1: Authentication & Account
├── UC-1.1: Student Login
├── UC-1.2: Student Registration
├── UC-1.3: Reset Password
├── UC-1.4: Change Password
└── UC-1.5: Update Profile

UC-2: Application Management (Student)
├── UC-2.1: Create Application
├── UC-2.2: Upload Documents
├── UC-2.3: Submit Application
├── UC-2.4: Check Application Status
├── UC-2.5: View Application Details
└── UC-2.6: Withdraw Application

UC-3: Document Review (OIDB)
├── UC-3.1: View Pending Documents
├── UC-3.2: Review Document
├── UC-3.3: Request Document Resubmission
└── UC-3.4: Approve All Documents

UC-4: Workflow Management
├── UC-4.1: Transfer to Next Stage
├── UC-4.2: Return to Previous Stage
├── UC-4.3: Reject Application
└── UC-4.4: Assign Application

UC-5: Language Evaluation (YDYO)
├── UC-5.1: View Language Pending Applications
├── UC-5.2: Evaluate Language Certificate
└── UC-5.3: Submit Language Decision

UC-6: Academic Review (Dean's Office)
├── UC-6.1: Review Application
├── UC-6.2: Forward to YGK
└── UC-6.3: Make Final Decision

UC-7: Intibak (YGK)
├── UC-7.1: View Intibak Pending Applications
├── UC-7.2: Add Course Equivalence
├── UC-7.3: Update Course Decision
├── UC-7.4: Complete Intibak
└── UC-7.5: Calculate Rankings

UC-8: Administration
├── UC-8.1: Manage Users
├── UC-8.2: Manage Departments
├── UC-8.3: View Audit Logs
├── UC-8.4: Generate Reports
└── UC-8.5: System Configuration
```

---

## 2. AUTHENTICATION USE CASES

### UC-1.1: Student Login

**Actor:** Applicant Student  
**Priority:** High  
**Preconditions:**
- User has a registered account
- User is not locked out

**Main Flow:**
1. Student navigates to login page
2. System displays login form (email, password)
3. Student enters credentials
4. Student clicks "Login" button
5. System validates credentials
6. System generates JWT tokens (access + refresh)
7. System records login timestamp
8. System redirects to student dashboard
9. System displays success message

**Alternative Flows:**

*AF-1: Invalid Credentials*
1. System detects invalid email/password
2. System increments failed attempt counter
3. System displays "Invalid email or password"
4. Student remains on login page

*AF-2: Account Locked*
1. System detects locked account (5+ failed attempts)
2. System displays "Account locked. Please reset password."
3. System provides link to password reset

*AF-3: Account Inactive*
1. System detects inactive account
2. System displays "Account is deactivated. Contact support."

**Postconditions:**
- Student is authenticated
- Session is established
- Failed attempts counter is reset

**Business Rules:**
- BR-001: Max 5 failed attempts before lockout
- BR-002: Access token expires in 30 minutes
- BR-003: Refresh token expires in 7 days

---

### UC-1.2: Student Registration

**Actor:** Applicant Student  
**Priority:** High  
**Preconditions:**
- User does not have existing account
- Application period is open (optional)

**Main Flow:**
1. Student navigates to registration page
2. System displays registration form
3. Student enters required information:
   - Email address
   - Password (with confirmation)
   - First name
   - Last name
   - Phone number (optional)
4. Student reads and accepts KVKK consent
5. Student clicks "Register" button
6. System validates all inputs
7. System creates user account (role: STUDENT)
8. System records KVKK consent timestamp
9. System sends welcome email
10. System redirects to login page
11. System displays "Registration successful"

**Alternative Flows:**

*AF-1: Email Already Exists*
1. System detects duplicate email
2. System displays "Email already registered"
3. System suggests password reset

*AF-2: Password Mismatch*
1. System detects password ≠ confirmPassword
2. System displays "Passwords do not match"

*AF-3: Weak Password*
1. System detects password doesn't meet requirements
2. System displays specific requirement failures

*AF-4: KVKK Not Accepted*
1. Student doesn't check KVKK consent
2. System prevents registration
3. System displays "KVKK consent is required"

**Postconditions:**
- New user account created
- KVKK consent recorded
- Welcome email sent

**Validation Rules:**
- Email: Valid format, unique
- Password: Min 8 chars, 1 upper, 1 lower, 1 digit, 1 special
- First/Last name: 2-100 characters
- Phone: Turkish format (+90XXXXXXXXXX)
- KVKK: Must be true

---

### UC-1.3: Reset Password

**Actor:** Applicant Student, Staff  
**Priority:** High  
**Preconditions:**
- User has registered account
- User has access to registered email

**Main Flow:**
1. User clicks "Forgot Password" on login page
2. System displays email input form
3. User enters registered email
4. User clicks "Send Reset Link"
5. System generates unique reset token
6. System sets token expiry (60 minutes)
7. System sends email with reset link
8. System displays "If account exists, email sent"
9. User clicks link in email
10. System validates token
11. System displays new password form
12. User enters new password (with confirmation)
13. User clicks "Reset Password"
14. System validates password strength
15. System updates password hash
16. System marks token as used
17. System unlocks account if locked
18. System sends confirmation email
19. System redirects to login page

**Alternative Flows:**

*AF-1: Email Not Found*
1. System doesn't find email
2. System still displays success message (security)
3. No email is sent

*AF-2: Token Expired*
1. System detects expired token (>60 min)
2. System displays "Link expired. Request new one."
3. System redirects to forgot password page

*AF-3: Token Already Used*
1. System detects token marked as used
2. System displays "Link already used."
3. System redirects to forgot password page

**Postconditions:**
- Password is updated
- Token is invalidated
- Account is unlocked
- Confirmation email sent

---

## 3. APPLICATION MANAGEMENT USE CASES

### UC-2.1: Create Application

**Actor:** Applicant Student  
**Priority:** High  
**Preconditions:**
- Student is logged in
- Student doesn't have active application for current term
- Application period is open

**Main Flow:**
1. Student navigates to "New Application"
2. System displays application form
3. System loads available departments (with quotas)
4. Student selects target department
5. Student enters source institution info:
   - University name
   - Faculty name
   - Department name
   - Current semester
6. Student enters academic scores:
   - YKS score
   - YKS ranking
   - Current GPA
7. Student enters language info (if required):
   - Certificate type
   - Score
8. Student clicks "Save as Draft"
9. System validates input data
10. System creates application with DRAFT status
11. System displays success message
12. System redirects to application detail page

**Alternative Flows:**

*AF-1: Active Application Exists*
1. System detects existing active application
2. System displays "You already have an application"
3. System provides link to existing application

*AF-2: Application Period Closed*
1. System detects period is closed
2. System displays "Application period is closed"
3. System shows next period dates

*AF-3: Department Inactive*
1. System detects department not accepting applications
2. System removes from dropdown
3. If already selected, shows error

*AF-4: Eligibility Check Failed*
1. System checks GPA/ranking requirements
2. System displays warning (not blocking)
3. Student can still save draft

**Postconditions:**
- Application created with DRAFT status
- Application number generated
- Student can upload documents

**Data Captured:**
- Target department
- Source university, faculty, department
- Current semester (1-14)
- YKS score (decimal)
- YKS ranking (integer)
- GPA (0.00-4.00)
- Language certificate type
- Language score

---

### UC-2.2: Upload Documents

**Actor:** Applicant Student  
**Priority:** High  
**Preconditions:**
- Application exists in DRAFT status
- Student owns the application

**Main Flow:**
1. Student navigates to application documents section
2. System displays document upload interface
3. System shows required documents list with status:
   - ✓ Uploaded
   - ⊘ Missing
   - ⟳ Requires resubmission
4. Student selects document type from dropdown
5. Student selects file from device
6. System validates file:
   - Type: PDF, JPG, PNG
   - Size: ≤ 15 MB
7. System uploads and stores file
8. System generates checksum
9. System creates document record (PENDING status)
10. System displays upload success
11. System updates document list

**Alternative Flows:**

*AF-1: Invalid File Type*
1. System detects unsupported file type
2. System displays "Only PDF, JPG, PNG allowed"
3. Upload is rejected

*AF-2: File Too Large*
1. System detects file > 15 MB
2. System displays "File exceeds 15 MB limit"
3. Upload is rejected

*AF-3: Duplicate Document Type*
1. System detects document type already uploaded
2. System displays "Delete existing document first"
3. Upload is rejected

*AF-4: Application Not Draft*
1. System detects application already submitted
2. System hides upload interface
3. Only view is available

**Postconditions:**
- Document stored securely
- Document record created
- Document list updated

**Required Documents:**
1. Transcript (TRANSCRIPT)
2. YKS Result (YKS_RESULT)
3. ID Card (ID_CARD)
4. Student Certificate (STUDENT_CERTIFICATE)
5. Disciplinary Certificate (DISCIPLINARY_CERTIFICATE)
6. Course Contents (COURSE_CONTENT)
7. Language Certificate (if required)

---

### UC-2.3: Submit Application

**Actor:** Applicant Student  
**Priority:** High  
**Preconditions:**
- Application is in DRAFT status
- All required documents uploaded
- Student owns the application

**Main Flow:**
1. Student reviews application details
2. Student reviews uploaded documents
3. Student clicks "Submit Application"
4. System displays confirmation dialog:
   "Application cannot be modified after submission. Continue?"
5. Student confirms submission
6. System validates all required documents present
7. System changes status: DRAFT → SUBMITTED
8. System sets submittedAt timestamp
9. System creates workflow history entry
10. System sends submission notification (email + in-app)
11. System displays success with application number
12. System redirects to status tracking page

**Alternative Flows:**

*AF-1: Missing Documents*
1. System detects missing required documents
2. System displays list of missing documents
3. Submission is blocked
4. Student redirected to upload page

*AF-2: User Cancels*
1. Student clicks "Cancel" on confirmation
2. Application remains in DRAFT
3. Student returns to application detail

**Postconditions:**
- Status changed to SUBMITTED
- Application enters workflow
- Notification sent to student
- Workflow begins at OIDB_INITIAL stage

**Notifications Generated:**
- Email: Application submission confirmation
- In-app: APPLICATION_SUBMITTED notification

---

### UC-2.4: Check Application Status

**Actor:** Applicant Student  
**Priority:** High  
**Preconditions:**
- Student is logged in
- Student has at least one application

**Main Flow:**
1. Student navigates to "My Applications"
2. System retrieves student's applications
3. System displays application list with:
   - Application number
   - Target department
   - Current status (badge)
   - Submission date
   - Last update date
4. Student clicks on specific application
5. System displays detailed status view:
   - Current stage indicator
   - Progress percentage
   - Timeline with all stages
   - Completed vs pending stages
6. System highlights current position
7. System shows estimated completion time

**Status Timeline Display:**
```
[✓] OIDB Initial Review          - Completed 03/15
[✓] Language Evaluation          - Completed 03/17
[●] Academic Evaluation          - In Progress
[ ] Intibak Preparation          - Pending
[ ] Faculty Decision             - Pending
[ ] Result Publication           - Pending
```

**Postconditions:**
- Student sees current status
- Student understands progress
- Student knows next steps

---

### UC-2.5: View Application Details

**Actor:** Applicant Student  
**Priority:** Medium  
**Preconditions:**
- Application exists
- Student owns the application

**Main Flow:**
1. Student selects application from list
2. System retrieves full application data
3. System displays sections:
   
   **A. Basic Information**
   - Application number
   - Status & stage
   - Submission date
   
   **B. Target Department**
   - Faculty name
   - Department name
   - Quota
   
   **C. Source Institution**
   - University
   - Faculty
   - Department
   - Current semester
   
   **D. Academic Scores**
   - YKS score
   - YKS ranking
   - GPA
   - Transfer grade (if calculated)
   
   **E. Language Information**
   - Certificate type
   - Score
   - Evaluation result
   
   **F. Documents**
   - List with status badges
   - Download links
   - Review comments (if any)
   
   **G. Workflow History**
   - Timeline of all actions
   - Who performed each action
   - Timestamps

**Postconditions:**
- Student has complete visibility
- Student can download own documents

---

### UC-2.6: Withdraw Application

**Actor:** Applicant Student  
**Priority:** Medium  
**Preconditions:**
- Application is in withdrawable state (not ASIL, not already withdrawn/rejected)
- Student owns the application

**Main Flow:**
1. Student views application details
2. Student clicks "Withdraw Application"
3. System displays confirmation dialog with warning
4. System optionally asks for withdrawal reason
5. Student confirms withdrawal
6. System changes status to WITHDRAWN
7. System changes stage to COMPLETED
8. System creates workflow history entry
9. System sends withdrawal confirmation notification
10. System updates application list

**Alternative Flows:**

*AF-1: ASIL Status*
1. System detects application is ASIL (accepted)
2. System blocks withdrawal
3. System displays "Contact OIDB for withdrawal after acceptance"

*AF-2: Already Terminal*
1. System detects terminal status
2. Withdraw button is disabled
3. No action possible

**Postconditions:**
- Application status is WITHDRAWN
- Workflow is terminated
- Quota is freed for others
- Student notified

---

## 4. DOCUMENT REVIEW USE CASES (OIDB)

### UC-3.1: View Pending Documents

**Actor:** OIDB Staff  
**Priority:** High  
**Preconditions:**
- User is OIDB Staff or Admin
- User is logged in

**Main Flow:**
1. Staff navigates to "Document Review" section
2. System retrieves applications at OIDB_INITIAL stage
3. System displays list with:
   - Application number
   - Student name
   - Department
   - Total documents
   - Pending documents count
   - Submitted date
4. Staff can filter by:
   - Department
   - Date range
   - Document status
5. Staff sorts by oldest first (default)
6. Staff clicks application to review

**Postconditions:**
- Staff has prioritized work queue
- Staff can access individual applications

---

### UC-3.2: Review Document

**Actor:** OIDB Staff  
**Priority:** High  
**Preconditions:**
- Application is at reviewable stage
- Document exists and is PENDING

**Main Flow:**
1. Staff opens application detail
2. Staff navigates to documents tab
3. Staff clicks document to review
4. System opens document viewer
5. Staff examines document content
6. Staff selects review decision:
   - APPROVED - Document is valid
   - REJECTED - Document is invalid
   - REQUIRES_RESUBMISSION - Need corrected version
7. Staff enters comments (required for non-approval)
8. Staff clicks "Submit Review"
9. System updates document status
10. System records reviewer and timestamp
11. System sends notification to student
12. System updates pending count

**Alternative Flows:**

*AF-1: Document Already Reviewed*
1. System shows document status as non-pending
2. Review options are disabled
3. Staff can only view

**Postconditions:**
- Document status updated
- Student notified
- Audit trail created

---

### UC-3.3: Request Document Resubmission

**Actor:** OIDB Staff  
**Priority:** Medium  
**Preconditions:**
- Document is PENDING
- Document has issues that can be corrected

**Main Flow:**
1. Staff reviews document
2. Staff identifies fixable issue
3. Staff selects "Requires Resubmission"
4. Staff enters detailed instructions:
   - What is wrong
   - What is expected
   - Example if applicable
5. Staff submits review
6. System sets status to REQUIRES_RESUBMISSION
7. System sends notification with instructions
8. System allows student to re-upload
9. New upload replaces old document

**Postconditions:**
- Document marked for resubmission
- Student notified with instructions
- Student can upload corrected version

---

### UC-3.4: Approve All Documents

**Actor:** OIDB Staff  
**Priority:** High  
**Preconditions:**
- All documents for application are APPROVED
- Application is at OIDB_INITIAL stage

**Main Flow:**
1. Staff reviews final document
2. System detects all documents now APPROVED
3. System displays "All documents approved" message
4. Staff clicks "Forward to Language Evaluation"
5. System confirms action
6. System transfers to YDYO_EVALUATION stage
7. System changes status to PENDING_LANGUAGE_EVALUATION
8. System creates workflow history
9. System notifies student of progress

**Postconditions:**
- Application moved to next stage
- YDYO staff can now process
- Student notified

---

## 5. LANGUAGE EVALUATION USE CASES (YDYO)

### UC-5.1: View Language Pending Applications

**Actor:** YDYO Staff  
**Priority:** High  
**Preconditions:**
- User is YDYO Staff
- Applications exist at YDYO_EVALUATION stage

**Main Flow:**
1. Staff logs in and navigates to dashboard
2. System shows applications pending language review
3. Display includes:
   - Student name
   - Target department
   - Certificate type
   - Claimed score
   - Days waiting
4. Staff sorts by priority/date
5. Staff selects application to evaluate

---

### UC-5.2: Evaluate Language Certificate

**Actor:** YDYO Staff  
**Priority:** High  
**Preconditions:**
- Application is at YDYO_EVALUATION stage
- Language certificate is uploaded

**Main Flow:**
1. Staff opens application
2. Staff views language certificate document
3. Staff verifies:
   - Certificate authenticity
   - Certificate type validity
   - Score against requirement
   - Certificate date (not expired)
4. Staff compares score with department minimum
5. Staff makes evaluation decision

**Evaluation Criteria:**
| Factor | Check |
|--------|-------|
| Certificate Type | YDS, YÖKDİL, TOEFL, IELTS, PTE |
| Score | ≥ Department minimum (default 60) |
| Date | Within validity period |
| Authenticity | Verification code check |

---

### UC-5.3: Submit Language Decision

**Actor:** YDYO Staff  
**Priority:** High  
**Preconditions:**
- Application at YDYO_EVALUATION
- Staff has evaluated certificate

**Main Flow:**
1. Staff selects decision:
   - APPROVED - Meets requirements
   - REJECTED - Does not meet requirements
2. Staff enters evaluation notes
3. Staff clicks "Submit Evaluation"
4. System updates application:
   - languageApproved = true/false
   - status = LANGUAGE_APPROVED/LANGUAGE_REJECTED
5. System creates workflow history
6. If approved:
   - Stage → OIDB_POST_LANGUAGE
7. If rejected:
   - Stage → COMPLETED
   - Status → LANGUAGE_REJECTED (terminal)
8. System notifies student

**Postconditions:**
- Language evaluation recorded
- Application proceeds or terminates
- Student notified with result

---

## 6. ACADEMIC REVIEW USE CASES (DEAN'S OFFICE)

### UC-6.1: Review Application

**Actor:** Dean's Office Staff  
**Priority:** High  
**Preconditions:**
- Application is at DEANS_OFFICE_REVIEW stage

**Main Flow:**
1. Staff views application details
2. Staff reviews:
   - Academic credentials
   - Transfer grade
   - Eligibility status
   - Document completeness
3. Staff verifies alignment with faculty requirements
4. Staff decides on action:
   - Forward to YGK
   - Return for clarification
   - Reject application

---

### UC-6.2: Forward to YGK

**Actor:** Dean's Office Staff  
**Priority:** High  
**Preconditions:**
- Application passed initial review
- Ready for intibak preparation

**Main Flow:**
1. Staff clicks "Forward to YGK"
2. Staff optionally assigns to specific YGK member
3. Staff adds notes for YGK
4. System transfers stage to YGK_EVALUATION
5. System updates status to PENDING_INTIBAK
6. System notifies student and YGK

---

### UC-6.3: Make Final Decision

**Actor:** Dean's Office Staff (Faculty Board)  
**Priority:** High  
**Preconditions:**
- Application is at DEANS_OFFICE_FINAL stage
- Intibak is completed
- Rankings are calculated

**Main Flow:**
1. Staff views completed application with intibak
2. Staff sees ranking position
3. Staff sees quota and current accepted count
4. Staff makes decision:
   - ASIL - Accept as primary
   - YEDEK - Place on waitlist
   - REJECTED - Reject application
5. Staff enters decision justification
6. System updates application status
7. System creates workflow history
8. System notifies student with result
9. Application moves to OIDB_FINAL

**Decision Factors:**
- Transfer grade ranking
- Department quota
- Intibak results
- Overall suitability

---

## 7. INTIBAK USE CASES (YGK)

### UC-7.1: View Intibak Pending Applications

**Actor:** YGK Member  
**Priority:** High  
**Preconditions:**
- Applications exist at YGK_EVALUATION stage

**Main Flow:**
1. YGK member navigates to intibak dashboard
2. System displays pending applications:
   - Student name
   - Source department
   - Target department
   - Course count (from transcript)
   - Status
3. Member selects application to process

---

### UC-7.2: Add Course Equivalence

**Actor:** YGK Member  
**Priority:** High  
**Preconditions:**
- Application is at YGK_EVALUATION stage

**Main Flow:**
1. Member views student's transcript
2. Member views course contents document
3. Member selects source course to evaluate
4. System displays course details:
   - Code, name, credits, grade
5. Member searches for equivalent target course
6. Member compares:
   - Course contents (70%+ match)
   - Credit hours
   - ECTS credits
7. Member decides:
   - ACCEPTED - Full equivalence
   - CONDITIONAL - Partial, with requirements
   - REJECTED - No equivalence
8. Member adds notes explaining decision
9. System saves intibak record

**Course Comparison Interface:**
```
Source Course          →    Target Course
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
BIL101 Programming I   →    CENG111 Intro to CS
Credits: 4             →    Credits: 4
Grade: AA (4.00)       →    
ECTS: 5                →    ECTS: 5

Decision: [ACCEPTED ▼]
Notes: [Content matches curriculum]
```

---

### UC-7.3: Update Course Decision

**Actor:** YGK Member  
**Priority:** Medium  
**Preconditions:**
- Intibak record exists
- Intibak not yet completed

**Main Flow:**
1. Member views existing intibak records
2. Member selects record to update
3. Member modifies:
   - Target course mapping
   - Decision
   - Notes
4. Member saves changes
5. System updates record

---

### UC-7.4: Complete Intibak

**Actor:** YGK Member  
**Priority:** High  
**Preconditions:**
- All courses have been evaluated
- No PENDING decisions remain

**Main Flow:**
1. Member reviews all intibak records
2. System shows summary:
   - Total source credits: 45
   - Accepted credits: 38
   - Acceptance rate: 84%
3. Member verifies completeness
4. Member clicks "Complete Intibak"
5. System validates all decisions made
6. System calculates credit summary
7. System changes status to INTIBAK_COMPLETED
8. System transfers to DEANS_OFFICE_FINAL
9. System notifies relevant parties

**Intibak Summary:**
```
Intibak Summary
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Courses Evaluated: 15
Accepted: 12 (80%)
Conditional: 2 (13%)
Rejected: 1 (7%)

Credits Accepted: 38/45 (84%)
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

---

### UC-7.5: Calculate Rankings

**Actor:** YGK Member, OIDB Staff  
**Priority:** High  
**Preconditions:**
- Multiple applications ready for ranking
- Transfer grades calculated

**Main Flow:**
1. Staff selects department and term
2. System retrieves eligible applications
3. System calculates/verifies transfer grades
4. System sorts by transfer grade (descending)
5. System assigns rankings
6. System determines ASIL vs YEDEK:
   - Positions 1-N (quota) = ASIL
   - Positions N+1 onwards = YEDEK
7. Staff reviews rankings
8. Staff confirms rankings
9. System saves ranking positions
10. System notifies students

**Ranking Display:**
```
Computer Engineering - Fall 2026
Quota: 5

Rank  Name            Transfer Grade  Status
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
1     Ali Yılmaz      105.2341        ASIL
2     Ayşe Kaya       102.8765        ASIL
3     Mehmet Demir    101.5432        ASIL
4     Zeynep Öz       99.8901         ASIL
5     Can Özkan       98.4567         ASIL
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
6     Deniz Ak        97.2345         YEDEK
7     Elif Tan        96.1234         YEDEK
...
```

---

## 8. ADMINISTRATION USE CASES

### UC-8.1: Manage Users

**Actor:** System Admin  
**Priority:** High

**Operations:**
- List all users with filtering/search
- View user details
- Create staff accounts
- Activate/deactivate users
- Unlock locked accounts
- Change user roles
- Reset user passwords

---

### UC-8.2: Manage Departments

**Actor:** System Admin  
**Priority:** Medium

**Operations:**
- View all departments
- Update department quotas
- Update minimum requirements
- Enable/disable departments
- Configure language requirements

---

### UC-8.3: View Audit Logs

**Actor:** System Admin  
**Priority:** Medium

**Operations:**
- Search audit logs by:
  - User
  - Action type
  - Entity type
  - Date range
- View action details
- Export logs

---

### UC-8.4: Generate Reports

**Actor:** System Admin, OIDB Staff  
**Priority:** Medium

**Available Reports:**
- Applications by status
- Applications by department
- Processing time analysis
- Document rejection rates
- Ranking summaries
- User activity

---

### UC-8.5: System Configuration

**Actor:** System Admin  
**Priority:** Low

**Configuration Options:**
- Application period dates
- Email templates
- Notification settings
- System parameters

---

## 9. REVISION HISTORY

| Version | Date | Author | Changes |
|---------|------|--------|---------|
| 1.0 | Mar 2026 | Group 05 | Initial version |
