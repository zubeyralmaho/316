# UTMS - Undergraduate Transfer Management System
## 02 - Database Schema

**Version:** 1.0  
**Last Updated:** March 2026  
**Database:** PostgreSQL 15+

---

## 1. ENTITY RELATIONSHIP DIAGRAM

```
┌─────────────────────────────────────────────────────────────────────────────────────────┐
│                                    UTMS DATABASE SCHEMA                                  │
└─────────────────────────────────────────────────────────────────────────────────────────┘

┌──────────────────┐          ┌──────────────────┐          ┌──────────────────┐
│     faculties    │          │   departments    │          │      users       │
├──────────────────┤          ├──────────────────┤          ├──────────────────┤
│ id (PK)          │─────────▶│ id (PK)          │          │ id (PK)          │
│ name             │     1:N  │ faculty_id (FK)  │          │ email            │
│ code             │          │ name             │          │ password_hash    │
│ created_at       │          │ code             │          │ first_name       │
│ updated_at       │          │ quota            │          │ last_name        │
└──────────────────┘          │ min_gpa          │          │ role             │
                              │ max_ranking      │          │ phone            │
                              │ language_required│          │ is_active        │
                              │ created_at       │          │ account_locked   │
                              │ updated_at       │          │ failed_attempts  │
                              └────────┬─────────┘          │ kvkk_consent     │
                                       │                    │ kvkk_consent_date│
                                       │ 1:N                │ last_login_at    │
                                       ▼                    │ created_at       │
                              ┌──────────────────┐          │ updated_at       │
                              │  applications    │          └────────┬─────────┘
                              ├──────────────────┤                   │
┌──────────────────┐          │ id (PK)          │◀──────────────────┘
│ workflow_history │◀─────────│ user_id (FK)     │               1:N
├──────────────────┤     1:N  │ department_id(FK)│
│ id (PK)          │          │ status           │          ┌──────────────────┐
│ application_id   │          │ current_stage    │          │   documents      │
│ from_stage       │          │ assigned_to (FK) │          ├──────────────────┤
│ to_stage         │          │ yks_score        │          │ id (PK)          │
│ action           │          │ yks_ranking      │◀─────────│ application_id   │
│ performed_by(FK) │          │ gpa              │     1:N  │ document_type    │
│ notes            │          │ transfer_grade   │          │ file_name        │
│ created_at       │          │ is_asil          │          │ file_path        │
└──────────────────┘          │ ranking_position │          │ file_size        │
                              │ created_at       │          │ mime_type        │
                              │ updated_at       │          │ status           │
┌──────────────────┐          └────────┬─────────┘          │ review_comments  │
│ intibak_records  │                   │                    │ uploaded_at      │
├──────────────────┤                   │ 1:N                │ reviewed_at      │
│ id (PK)          │◀──────────────────┤                    │ reviewed_by (FK) │
│ application_id   │                   │                    │ created_at       │
│ source_course_co │                   │                    └──────────────────┘
│ source_course_na │                   │
│ source_credits   │                   │ 1:N
│ source_grade     │                   ▼
│ target_course_co │          ┌──────────────────┐
│ target_course_na │          │  notifications   │
│ target_credits   │          ├──────────────────┤
│ decision         │          │ id (PK)          │
│ prepared_by (FK) │          │ user_id (FK)     │
│ created_at       │          │ application_id   │
│ updated_at       │          │ type             │
└──────────────────┘          │ title            │
                              │ message          │
┌──────────────────┐          │ is_read          │
│   audit_logs     │          │ read_at          │
├──────────────────┤          │ created_at       │
│ id (PK)          │          └──────────────────┘
│ user_id (FK)     │
│ action           │          ┌──────────────────┐
│ entity_type      │          │password_reset_   │
│ entity_id        │          │    tokens        │
│ old_value (JSONB)│          ├──────────────────┤
│ new_value (JSONB)│          │ id (PK)          │
│ ip_address       │          │ user_id (FK)     │
│ user_agent       │          │ token            │
│ created_at       │          │ expires_at       │
└──────────────────┘          │ used             │
                              │ created_at       │
                              └──────────────────┘
```

---

## 2. TABLE DEFINITIONS

### 2.1 users

User account information for all system users.

```sql
CREATE TABLE users (
    id                  BIGSERIAL PRIMARY KEY,
    email               VARCHAR(255) NOT NULL UNIQUE,
    password_hash       VARCHAR(255) NOT NULL,
    first_name          VARCHAR(100) NOT NULL,
    last_name           VARCHAR(100) NOT NULL,
    role                VARCHAR(50) NOT NULL,
    phone               VARCHAR(20),
    is_active           BOOLEAN DEFAULT TRUE,
    account_locked      BOOLEAN DEFAULT FALSE,
    failed_attempts     INTEGER DEFAULT 0,
    kvkk_consent        BOOLEAN DEFAULT FALSE,
    kvkk_consent_date   TIMESTAMP,
    last_login_at       TIMESTAMP,
    created_at          TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at          TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    
    CONSTRAINT chk_role CHECK (role IN (
        'STUDENT', 'OIDB_STAFF', 'DEAN_STAFF', 
        'YGK_MEMBER', 'YDYO_STAFF', 'SYSTEM_ADMIN'
    )),
    CONSTRAINT chk_email_format CHECK (email ~* '^[A-Za-z0-9._%+-]+@[A-Za-z0-9.-]+\.[A-Za-z]{2,}$')
);

-- Indexes
CREATE INDEX idx_users_email ON users(email);
CREATE INDEX idx_users_role ON users(role);
CREATE INDEX idx_users_active ON users(is_active) WHERE is_active = TRUE;
```

| Column | Type | Constraints | Description |
|--------|------|-------------|-------------|
| id | BIGSERIAL | PK | Unique identifier |
| email | VARCHAR(255) | NOT NULL, UNIQUE | User email (login) |
| password_hash | VARCHAR(255) | NOT NULL | BCrypt hashed password |
| first_name | VARCHAR(100) | NOT NULL | User's first name |
| last_name | VARCHAR(100) | NOT NULL | User's last name |
| role | VARCHAR(50) | NOT NULL | User role enum |
| phone | VARCHAR(20) | | Contact phone number |
| is_active | BOOLEAN | DEFAULT TRUE | Account active status |
| account_locked | BOOLEAN | DEFAULT FALSE | Locked due to failed attempts |
| failed_attempts | INTEGER | DEFAULT 0 | Failed login counter |
| kvkk_consent | BOOLEAN | DEFAULT FALSE | KVKK consent given |
| kvkk_consent_date | TIMESTAMP | | When consent was given |
| last_login_at | TIMESTAMP | | Last successful login |
| created_at | TIMESTAMP | DEFAULT NOW | Record creation time |
| updated_at | TIMESTAMP | DEFAULT NOW | Last update time |

---

### 2.2 faculties

Faculty/college information.

```sql
CREATE TABLE faculties (
    id              BIGSERIAL PRIMARY KEY,
    name            VARCHAR(255) NOT NULL,
    code            VARCHAR(20) NOT NULL UNIQUE,
    dean_name       VARCHAR(200),
    contact_email   VARCHAR(255),
    is_active       BOOLEAN DEFAULT TRUE,
    created_at      TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at      TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Indexes
CREATE INDEX idx_faculties_code ON faculties(code);
CREATE INDEX idx_faculties_active ON faculties(is_active) WHERE is_active = TRUE;
```

| Column | Type | Constraints | Description |
|--------|------|-------------|-------------|
| id | BIGSERIAL | PK | Unique identifier |
| name | VARCHAR(255) | NOT NULL | Faculty name |
| code | VARCHAR(20) | NOT NULL, UNIQUE | Faculty code (e.g., 'ENG') |
| dean_name | VARCHAR(200) | | Current dean name |
| contact_email | VARCHAR(255) | | Faculty contact email |
| is_active | BOOLEAN | DEFAULT TRUE | Active status |
| created_at | TIMESTAMP | DEFAULT NOW | Record creation time |
| updated_at | TIMESTAMP | DEFAULT NOW | Last update time |

---

### 2.3 departments

Academic department information with transfer criteria.

```sql
CREATE TABLE departments (
    id                  BIGSERIAL PRIMARY KEY,
    faculty_id          BIGINT NOT NULL REFERENCES faculties(id),
    name                VARCHAR(255) NOT NULL,
    code                VARCHAR(20) NOT NULL UNIQUE,
    quota               INTEGER DEFAULT 0,
    min_gpa             DECIMAL(3,2) DEFAULT 2.00,
    max_ranking         INTEGER,
    min_yks_score       DECIMAL(10,2),
    language_required   BOOLEAN DEFAULT TRUE,
    min_language_score  INTEGER DEFAULT 60,
    special_requirements TEXT,
    is_active           BOOLEAN DEFAULT TRUE,
    created_at          TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at          TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    
    CONSTRAINT chk_gpa_range CHECK (min_gpa >= 0 AND min_gpa <= 4.00),
    CONSTRAINT chk_quota_positive CHECK (quota >= 0)
);

-- Indexes
CREATE INDEX idx_departments_faculty ON departments(faculty_id);
CREATE INDEX idx_departments_code ON departments(code);
CREATE INDEX idx_departments_active ON departments(is_active) WHERE is_active = TRUE;
```

| Column | Type | Constraints | Description |
|--------|------|-------------|-------------|
| id | BIGSERIAL | PK | Unique identifier |
| faculty_id | BIGINT | FK, NOT NULL | Reference to faculty |
| name | VARCHAR(255) | NOT NULL | Department name |
| code | VARCHAR(20) | NOT NULL, UNIQUE | Department code |
| quota | INTEGER | DEFAULT 0 | Transfer quota |
| min_gpa | DECIMAL(3,2) | DEFAULT 2.00 | Minimum GPA requirement |
| max_ranking | INTEGER | | Maximum YKS ranking allowed |
| min_yks_score | DECIMAL(10,2) | | Minimum YKS score |
| language_required | BOOLEAN | DEFAULT TRUE | Language cert required |
| min_language_score | INTEGER | DEFAULT 60 | Min language score |
| special_requirements | TEXT | | Special requirements text |
| is_active | BOOLEAN | DEFAULT TRUE | Active status |
| created_at | TIMESTAMP | DEFAULT NOW | Record creation time |
| updated_at | TIMESTAMP | DEFAULT NOW | Last update time |

---

### 2.4 applications

Transfer application records.

```sql
CREATE TABLE applications (
    id                  BIGSERIAL PRIMARY KEY,
    user_id             BIGINT NOT NULL REFERENCES users(id),
    department_id       BIGINT NOT NULL REFERENCES departments(id),
    application_year    INTEGER NOT NULL,
    application_term    VARCHAR(20) NOT NULL,
    status              VARCHAR(50) NOT NULL DEFAULT 'DRAFT',
    current_stage       VARCHAR(50) NOT NULL DEFAULT 'OIDB_INITIAL',
    assigned_to         BIGINT REFERENCES users(id),
    
    -- Academic Information
    source_university   VARCHAR(255) NOT NULL,
    source_department   VARCHAR(255) NOT NULL,
    source_faculty      VARCHAR(255),
    current_semester    INTEGER NOT NULL,
    yks_score           DECIMAL(10,2) NOT NULL,
    yks_ranking         INTEGER NOT NULL,
    gpa                 DECIMAL(4,2) NOT NULL,
    
    -- Calculated Fields
    transfer_grade      DECIMAL(10,4),
    is_asil             BOOLEAN,
    ranking_position    INTEGER,
    
    -- Language Info
    language_cert_type  VARCHAR(50),
    language_score      INTEGER,
    language_approved   BOOLEAN,
    
    -- Timestamps
    submitted_at        TIMESTAMP,
    evaluated_at        TIMESTAMP,
    decided_at          TIMESTAMP,
    created_at          TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at          TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    
    CONSTRAINT chk_status CHECK (status IN (
        'DRAFT', 'SUBMITTED', 'DOCUMENTS_UNDER_REVIEW', 'DOCUMENTS_APPROVED',
        'DOCUMENTS_REJECTED', 'PENDING_LANGUAGE_EVALUATION', 'LANGUAGE_APPROVED',
        'LANGUAGE_REJECTED', 'PENDING_ACADEMIC_EVALUATION', 'PENDING_INTIBAK',
        'INTIBAK_COMPLETED', 'PENDING_FACULTY_DECISION', 'ASIL', 'YEDEK',
        'REJECTED', 'WITHDRAWN'
    )),
    CONSTRAINT chk_stage CHECK (current_stage IN (
        'OIDB_INITIAL', 'YDYO_EVALUATION', 'OIDB_POST_LANGUAGE',
        'DEANS_OFFICE_REVIEW', 'YGK_EVALUATION', 'DEANS_OFFICE_FINAL',
        'OIDB_FINAL', 'COMPLETED'
    )),
    CONSTRAINT chk_term CHECK (application_term IN ('FALL', 'SPRING')),
    CONSTRAINT chk_gpa_valid CHECK (gpa >= 0 AND gpa <= 4.00),
    CONSTRAINT chk_semester CHECK (current_semester BETWEEN 1 AND 14)
);

-- Indexes
CREATE INDEX idx_applications_user ON applications(user_id);
CREATE INDEX idx_applications_department ON applications(department_id);
CREATE INDEX idx_applications_status ON applications(status);
CREATE INDEX idx_applications_stage ON applications(current_stage);
CREATE INDEX idx_applications_assigned ON applications(assigned_to);
CREATE INDEX idx_applications_year_term ON applications(application_year, application_term);
CREATE INDEX idx_applications_ranking ON applications(department_id, transfer_grade DESC NULLS LAST);

-- Unique constraint: One active application per user per term
CREATE UNIQUE INDEX idx_applications_unique_active 
    ON applications(user_id, application_year, application_term) 
    WHERE status NOT IN ('REJECTED', 'WITHDRAWN');
```

| Column | Type | Constraints | Description |
|--------|------|-------------|-------------|
| id | BIGSERIAL | PK | Unique identifier |
| user_id | BIGINT | FK, NOT NULL | Applicant user ID |
| department_id | BIGINT | FK, NOT NULL | Target department |
| application_year | INTEGER | NOT NULL | Application year |
| application_term | VARCHAR(20) | NOT NULL | FALL or SPRING |
| status | VARCHAR(50) | NOT NULL | Current status |
| current_stage | VARCHAR(50) | NOT NULL | Workflow stage |
| assigned_to | BIGINT | FK | Currently assigned staff |
| source_university | VARCHAR(255) | NOT NULL | Source university name |
| source_department | VARCHAR(255) | NOT NULL | Source department name |
| source_faculty | VARCHAR(255) | | Source faculty name |
| current_semester | INTEGER | NOT NULL | Current semester (1-14) |
| yks_score | DECIMAL(10,2) | NOT NULL | YKS exam score |
| yks_ranking | INTEGER | NOT NULL | YKS ranking |
| gpa | DECIMAL(4,2) | NOT NULL | Current GPA (0-4.00) |
| transfer_grade | DECIMAL(10,4) | | Calculated transfer grade |
| is_asil | BOOLEAN | | Primary candidate flag |
| ranking_position | INTEGER | | Position in ranking |
| language_cert_type | VARCHAR(50) | | Type of language cert |
| language_score | INTEGER | | Language test score |
| language_approved | BOOLEAN | | Language evaluation result |
| submitted_at | TIMESTAMP | | Submission timestamp |
| evaluated_at | TIMESTAMP | | Evaluation timestamp |
| decided_at | TIMESTAMP | | Final decision timestamp |
| created_at | TIMESTAMP | DEFAULT NOW | Record creation time |
| updated_at | TIMESTAMP | DEFAULT NOW | Last update time |

---

### 2.5 documents

Uploaded document records.

```sql
CREATE TABLE documents (
    id                  BIGSERIAL PRIMARY KEY,
    application_id      BIGINT NOT NULL REFERENCES applications(id) ON DELETE CASCADE,
    document_type       VARCHAR(50) NOT NULL,
    file_name           VARCHAR(255) NOT NULL,
    original_file_name  VARCHAR(255) NOT NULL,
    file_path           VARCHAR(500) NOT NULL,
    file_size           BIGINT NOT NULL,
    mime_type           VARCHAR(100) NOT NULL,
    checksum            VARCHAR(64),
    status              VARCHAR(50) DEFAULT 'PENDING',
    review_comments     TEXT,
    uploaded_at         TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    reviewed_at         TIMESTAMP,
    reviewed_by         BIGINT REFERENCES users(id),
    created_at          TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at          TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    
    CONSTRAINT chk_document_type CHECK (document_type IN (
        'TRANSCRIPT', 'YKS_RESULT', 'LANGUAGE_CERTIFICATE',
        'COURSE_CONTENT', 'ID_CARD', 'PORTFOLIO',
        'DISCIPLINARY_CERTIFICATE', 'STUDENT_CERTIFICATE', 'OTHER'
    )),
    CONSTRAINT chk_document_status CHECK (status IN (
        'PENDING', 'APPROVED', 'REJECTED', 'REQUIRES_RESUBMISSION'
    )),
    CONSTRAINT chk_file_size CHECK (file_size > 0 AND file_size <= 15728640) -- 15MB
);

-- Indexes
CREATE INDEX idx_documents_application ON documents(application_id);
CREATE INDEX idx_documents_type ON documents(document_type);
CREATE INDEX idx_documents_status ON documents(status);
CREATE INDEX idx_documents_reviewer ON documents(reviewed_by);
```

| Column | Type | Constraints | Description |
|--------|------|-------------|-------------|
| id | BIGSERIAL | PK | Unique identifier |
| application_id | BIGINT | FK, NOT NULL | Parent application |
| document_type | VARCHAR(50) | NOT NULL | Type of document |
| file_name | VARCHAR(255) | NOT NULL | Stored file name (UUID) |
| original_file_name | VARCHAR(255) | NOT NULL | Original upload name |
| file_path | VARCHAR(500) | NOT NULL | Storage path |
| file_size | BIGINT | NOT NULL | Size in bytes |
| mime_type | VARCHAR(100) | NOT NULL | MIME type |
| checksum | VARCHAR(64) | | SHA-256 checksum |
| status | VARCHAR(50) | DEFAULT PENDING | Review status |
| review_comments | TEXT | | Reviewer comments |
| uploaded_at | TIMESTAMP | DEFAULT NOW | Upload timestamp |
| reviewed_at | TIMESTAMP | | Review timestamp |
| reviewed_by | BIGINT | FK | Reviewer user ID |
| created_at | TIMESTAMP | DEFAULT NOW | Record creation time |
| updated_at | TIMESTAMP | DEFAULT NOW | Last update time |

---

### 2.6 workflow_history

Application workflow audit trail.

```sql
CREATE TABLE workflow_history (
    id                  BIGSERIAL PRIMARY KEY,
    application_id      BIGINT NOT NULL REFERENCES applications(id) ON DELETE CASCADE,
    from_stage          VARCHAR(50),
    to_stage            VARCHAR(50) NOT NULL,
    action              VARCHAR(100) NOT NULL,
    performed_by        BIGINT REFERENCES users(id),
    notes               TEXT,
    metadata            JSONB,
    created_at          TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Indexes
CREATE INDEX idx_workflow_application ON workflow_history(application_id);
CREATE INDEX idx_workflow_performed_by ON workflow_history(performed_by);
CREATE INDEX idx_workflow_created ON workflow_history(created_at DESC);
CREATE INDEX idx_workflow_action ON workflow_history(action);
```

| Column | Type | Constraints | Description |
|--------|------|-------------|-------------|
| id | BIGSERIAL | PK | Unique identifier |
| application_id | BIGINT | FK, NOT NULL | Parent application |
| from_stage | VARCHAR(50) | | Previous stage |
| to_stage | VARCHAR(50) | NOT NULL | New stage |
| action | VARCHAR(100) | NOT NULL | Action performed |
| performed_by | BIGINT | FK | User who performed action |
| notes | TEXT | | Optional notes |
| metadata | JSONB | | Additional data |
| created_at | TIMESTAMP | DEFAULT NOW | Event timestamp |

---

### 2.7 intibak_records

Course equivalence (intibak) records.

```sql
CREATE TABLE intibak_records (
    id                      BIGSERIAL PRIMARY KEY,
    application_id          BIGINT NOT NULL REFERENCES applications(id) ON DELETE CASCADE,
    
    -- Source Course Info
    source_course_code      VARCHAR(50) NOT NULL,
    source_course_name      VARCHAR(255) NOT NULL,
    source_credits          INTEGER NOT NULL,
    source_ects             DECIMAL(4,1),
    source_grade            VARCHAR(10) NOT NULL,
    source_grade_point      DECIMAL(3,2),
    
    -- Target Course Info
    target_course_code      VARCHAR(50),
    target_course_name      VARCHAR(255),
    target_credits          INTEGER,
    target_ects             DECIMAL(4,1),
    
    -- Decision
    decision                VARCHAR(50) NOT NULL DEFAULT 'PENDING',
    decision_notes          TEXT,
    
    -- Metadata
    prepared_by             BIGINT REFERENCES users(id),
    approved_by             BIGINT REFERENCES users(id),
    created_at              TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at              TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    
    CONSTRAINT chk_intibak_decision CHECK (decision IN (
        'PENDING', 'ACCEPTED', 'REJECTED', 'CONDITIONAL'
    ))
);

-- Indexes
CREATE INDEX idx_intibak_application ON intibak_records(application_id);
CREATE INDEX idx_intibak_decision ON intibak_records(decision);
CREATE INDEX idx_intibak_prepared_by ON intibak_records(prepared_by);
```

| Column | Type | Constraints | Description |
|--------|------|-------------|-------------|
| id | BIGSERIAL | PK | Unique identifier |
| application_id | BIGINT | FK, NOT NULL | Parent application |
| source_course_code | VARCHAR(50) | NOT NULL | Source course code |
| source_course_name | VARCHAR(255) | NOT NULL | Source course name |
| source_credits | INTEGER | NOT NULL | Source credits |
| source_ects | DECIMAL(4,1) | | Source ECTS credits |
| source_grade | VARCHAR(10) | NOT NULL | Grade received |
| source_grade_point | DECIMAL(3,2) | | Grade point value |
| target_course_code | VARCHAR(50) | | Equivalent course code |
| target_course_name | VARCHAR(255) | | Equivalent course name |
| target_credits | INTEGER | | Target credits |
| target_ects | DECIMAL(4,1) | | Target ECTS |
| decision | VARCHAR(50) | NOT NULL | Equivalence decision |
| decision_notes | TEXT | | Decision notes |
| prepared_by | BIGINT | FK | YGK member who prepared |
| approved_by | BIGINT | FK | Approver |
| created_at | TIMESTAMP | DEFAULT NOW | Record creation time |
| updated_at | TIMESTAMP | DEFAULT NOW | Last update time |

---

### 2.8 notifications

User notification records.

```sql
CREATE TABLE notifications (
    id                  BIGSERIAL PRIMARY KEY,
    user_id             BIGINT NOT NULL REFERENCES users(id) ON DELETE CASCADE,
    application_id      BIGINT REFERENCES applications(id) ON DELETE SET NULL,
    type                VARCHAR(50) NOT NULL,
    title               VARCHAR(255) NOT NULL,
    message             TEXT NOT NULL,
    priority            VARCHAR(20) DEFAULT 'NORMAL',
    is_read             BOOLEAN DEFAULT FALSE,
    read_at             TIMESTAMP,
    action_url          VARCHAR(500),
    created_at          TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    
    CONSTRAINT chk_notification_type CHECK (type IN (
        'APPLICATION_SUBMITTED', 'DOCUMENT_REQUESTED', 'DOCUMENT_APPROVED',
        'DOCUMENT_REJECTED', 'STATUS_CHANGED', 'STAGE_CHANGED',
        'LANGUAGE_RESULT', 'RANKING_RESULT', 'FINAL_DECISION',
        'SYSTEM_ANNOUNCEMENT', 'REMINDER'
    )),
    CONSTRAINT chk_priority CHECK (priority IN ('LOW', 'NORMAL', 'HIGH', 'URGENT'))
);

-- Indexes
CREATE INDEX idx_notifications_user ON notifications(user_id);
CREATE INDEX idx_notifications_application ON notifications(application_id);
CREATE INDEX idx_notifications_unread ON notifications(user_id, is_read) WHERE is_read = FALSE;
CREATE INDEX idx_notifications_created ON notifications(created_at DESC);
CREATE INDEX idx_notifications_type ON notifications(type);
```

| Column | Type | Constraints | Description |
|--------|------|-------------|-------------|
| id | BIGSERIAL | PK | Unique identifier |
| user_id | BIGINT | FK, NOT NULL | Target user |
| application_id | BIGINT | FK | Related application |
| type | VARCHAR(50) | NOT NULL | Notification type |
| title | VARCHAR(255) | NOT NULL | Notification title |
| message | TEXT | NOT NULL | Notification message |
| priority | VARCHAR(20) | DEFAULT NORMAL | Priority level |
| is_read | BOOLEAN | DEFAULT FALSE | Read status |
| read_at | TIMESTAMP | | When read |
| action_url | VARCHAR(500) | | Action link |
| created_at | TIMESTAMP | DEFAULT NOW | Creation time |

---

### 2.9 audit_logs

System audit trail.

```sql
CREATE TABLE audit_logs (
    id                  BIGSERIAL PRIMARY KEY,
    user_id             BIGINT REFERENCES users(id) ON DELETE SET NULL,
    action              VARCHAR(100) NOT NULL,
    entity_type         VARCHAR(100) NOT NULL,
    entity_id           BIGINT,
    old_value           JSONB,
    new_value           JSONB,
    ip_address          VARCHAR(50),
    user_agent          VARCHAR(500),
    session_id          VARCHAR(100),
    created_at          TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Indexes
CREATE INDEX idx_audit_user ON audit_logs(user_id);
CREATE INDEX idx_audit_entity ON audit_logs(entity_type, entity_id);
CREATE INDEX idx_audit_action ON audit_logs(action);
CREATE INDEX idx_audit_created ON audit_logs(created_at DESC);

-- Partition by month for better performance
-- CREATE TABLE audit_logs_y2026m03 PARTITION OF audit_logs
--     FOR VALUES FROM ('2026-03-01') TO ('2026-04-01');
```

| Column | Type | Constraints | Description |
|--------|------|-------------|-------------|
| id | BIGSERIAL | PK | Unique identifier |
| user_id | BIGINT | FK | Acting user |
| action | VARCHAR(100) | NOT NULL | Action type |
| entity_type | VARCHAR(100) | NOT NULL | Entity class name |
| entity_id | BIGINT | | Entity primary key |
| old_value | JSONB | | Previous state |
| new_value | JSONB | | New state |
| ip_address | VARCHAR(50) | | Client IP |
| user_agent | VARCHAR(500) | | Browser user agent |
| session_id | VARCHAR(100) | | Session identifier |
| created_at | TIMESTAMP | DEFAULT NOW | Event timestamp |

---

### 2.10 password_reset_tokens

Password reset token management.

```sql
CREATE TABLE password_reset_tokens (
    id                  BIGSERIAL PRIMARY KEY,
    user_id             BIGINT NOT NULL REFERENCES users(id) ON DELETE CASCADE,
    token               VARCHAR(255) NOT NULL UNIQUE,
    expires_at          TIMESTAMP NOT NULL,
    used                BOOLEAN DEFAULT FALSE,
    used_at             TIMESTAMP,
    created_at          TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Indexes
CREATE INDEX idx_reset_tokens_user ON password_reset_tokens(user_id);
CREATE INDEX idx_reset_tokens_token ON password_reset_tokens(token);
CREATE INDEX idx_reset_tokens_expires ON password_reset_tokens(expires_at);

-- Auto-cleanup of expired tokens (can be scheduled job)
-- DELETE FROM password_reset_tokens WHERE expires_at < NOW() - INTERVAL '24 hours';
```

| Column | Type | Constraints | Description |
|--------|------|-------------|-------------|
| id | BIGSERIAL | PK | Unique identifier |
| user_id | BIGINT | FK, NOT NULL | Token owner |
| token | VARCHAR(255) | NOT NULL, UNIQUE | Reset token |
| expires_at | TIMESTAMP | NOT NULL | Token expiry |
| used | BOOLEAN | DEFAULT FALSE | Token used flag |
| used_at | TIMESTAMP | | When used |
| created_at | TIMESTAMP | DEFAULT NOW | Creation time |

---

## 3. ENUMERATIONS

### 3.1 UserRole

```sql
-- Values: STUDENT, OIDB_STAFF, DEAN_STAFF, YGK_MEMBER, YDYO_STAFF, SYSTEM_ADMIN
```

### 3.2 ApplicationStatus

```sql
-- Values:
-- DRAFT                      - Initial state, not submitted
-- SUBMITTED                  - Application submitted
-- DOCUMENTS_UNDER_REVIEW     - Documents being reviewed
-- DOCUMENTS_APPROVED         - All documents approved
-- DOCUMENTS_REJECTED         - Documents rejected
-- PENDING_LANGUAGE_EVALUATION - Waiting for YDYO review
-- LANGUAGE_APPROVED          - Language requirement met
-- LANGUAGE_REJECTED          - Language requirement not met
-- PENDING_ACADEMIC_EVALUATION - Waiting for academic review
-- PENDING_INTIBAK            - Waiting for intibak preparation
-- INTIBAK_COMPLETED          - Intibak prepared
-- PENDING_FACULTY_DECISION   - Waiting for faculty board
-- ASIL                       - Primary candidate
-- YEDEK                      - Waitlist candidate
-- REJECTED                   - Application rejected
-- WITHDRAWN                  - Application withdrawn by student
```

### 3.3 WorkflowStage

```sql
-- Values:
-- OIDB_INITIAL       - Initial OIDB review
-- YDYO_EVALUATION    - Language evaluation at YDYO
-- OIDB_POST_LANGUAGE - OIDB post-language review
-- DEANS_OFFICE_REVIEW - Dean's office review
-- YGK_EVALUATION     - YGK committee evaluation
-- DEANS_OFFICE_FINAL - Faculty board decision
-- OIDB_FINAL         - Final publication by OIDB
-- COMPLETED          - Process completed
```

### 3.4 DocumentType

```sql
-- Values:
-- TRANSCRIPT              - Official transcript
-- YKS_RESULT              - YKS exam result document
-- LANGUAGE_CERTIFICATE    - Language proficiency certificate
-- COURSE_CONTENT          - Course content/syllabus
-- ID_CARD                 - Identity card
-- PORTFOLIO               - Portfolio (for design departments)
-- DISCIPLINARY_CERTIFICATE - Disciplinary record certificate
-- STUDENT_CERTIFICATE     - Student status certificate
-- OTHER                   - Other documents
```

### 3.5 DocumentStatus

```sql
-- Values:
-- PENDING               - Waiting for review
-- APPROVED              - Document approved
-- REJECTED              - Document rejected
-- REQUIRES_RESUBMISSION - Needs to be resubmitted
```

---

## 4. DATABASE FUNCTIONS

### 4.1 Calculate Transfer Grade

```sql
CREATE OR REPLACE FUNCTION calculate_transfer_grade(
    p_yks_score DECIMAL,
    p_min_yks_score DECIMAL,
    p_gpa DECIMAL
) RETURNS DECIMAL AS $$
BEGIN
    -- Formula: TG = (x/y * 100 * 0.9) + (0.1 * z)
    -- x = yks_score, y = min_yks_score, z = gpa
    RETURN ROUND(
        (p_yks_score / p_min_yks_score * 100 * 0.9) + (0.1 * p_gpa),
        4
    );
END;
$$ LANGUAGE plpgsql IMMUTABLE;
```

### 4.2 Update Timestamps Trigger

```sql
CREATE OR REPLACE FUNCTION update_updated_at_column()
RETURNS TRIGGER AS $$
BEGIN
    NEW.updated_at = CURRENT_TIMESTAMP;
    RETURN NEW;
END;
$$ LANGUAGE plpgsql;

-- Apply to all tables with updated_at
CREATE TRIGGER update_users_updated_at
    BEFORE UPDATE ON users
    FOR EACH ROW EXECUTE FUNCTION update_updated_at_column();

CREATE TRIGGER update_applications_updated_at
    BEFORE UPDATE ON applications
    FOR EACH ROW EXECUTE FUNCTION update_updated_at_column();

-- ... (apply to other tables)
```

### 4.3 Audit Log Trigger

```sql
CREATE OR REPLACE FUNCTION audit_log_trigger()
RETURNS TRIGGER AS $$
BEGIN
    IF TG_OP = 'INSERT' THEN
        INSERT INTO audit_logs (action, entity_type, entity_id, new_value, created_at)
        VALUES ('INSERT', TG_TABLE_NAME, NEW.id, row_to_json(NEW), CURRENT_TIMESTAMP);
        RETURN NEW;
    ELSIF TG_OP = 'UPDATE' THEN
        INSERT INTO audit_logs (action, entity_type, entity_id, old_value, new_value, created_at)
        VALUES ('UPDATE', TG_TABLE_NAME, NEW.id, row_to_json(OLD), row_to_json(NEW), CURRENT_TIMESTAMP);
        RETURN NEW;
    ELSIF TG_OP = 'DELETE' THEN
        INSERT INTO audit_logs (action, entity_type, entity_id, old_value, created_at)
        VALUES ('DELETE', TG_TABLE_NAME, OLD.id, row_to_json(OLD), CURRENT_TIMESTAMP);
        RETURN OLD;
    END IF;
END;
$$ LANGUAGE plpgsql;

-- Apply to sensitive tables
CREATE TRIGGER audit_applications
    AFTER INSERT OR UPDATE OR DELETE ON applications
    FOR EACH ROW EXECUTE FUNCTION audit_log_trigger();
```

---

## 5. SAMPLE DATA

### 5.1 Faculties

```sql
INSERT INTO faculties (name, code) VALUES
('Faculty of Engineering', 'ENG'),
('Faculty of Science', 'SCI'),
('Faculty of Architecture', 'ARCH');
```

### 5.2 Departments

```sql
INSERT INTO departments (faculty_id, name, code, quota, min_gpa, max_ranking, language_required) VALUES
(1, 'Computer Engineering', 'CENG', 5, 2.50, 300000, true),
(1, 'Civil Engineering', 'CE', 3, 2.30, 350000, true),
(1, 'Mechanical Engineering', 'ME', 4, 2.30, 350000, true),
(2, 'Physics', 'PHYS', 3, 2.20, 400000, true),
(2, 'Chemistry', 'CHEM', 3, 2.20, 400000, true),
(3, 'Architecture', 'ARCH', 2, 2.50, 250000, true);
```

---

## 6. INDEXES SUMMARY

| Table | Index Name | Columns | Purpose |
|-------|------------|---------|---------|
| users | idx_users_email | email | Login lookup |
| users | idx_users_role | role | Role filtering |
| applications | idx_applications_user | user_id | User's applications |
| applications | idx_applications_status | status | Status filtering |
| applications | idx_applications_stage | current_stage | Workflow queries |
| applications | idx_applications_ranking | department_id, transfer_grade | Ranking queries |
| documents | idx_documents_application | application_id | Document listing |
| notifications | idx_notifications_unread | user_id, is_read | Unread count |
| audit_logs | idx_audit_entity | entity_type, entity_id | Entity history |

---

## 7. DATA RETENTION POLICY

| Data Type | Retention Period | Action |
|-----------|------------------|--------|
| Applications | 5 years | Archive then delete |
| Documents | 5 years | Archive then delete |
| Audit Logs | 7 years | Partition by month |
| Notifications | 1 year | Soft delete |
| Password Reset Tokens | 24 hours | Hard delete |
| Session Data | 7 days | Hard delete |

---

## 8. BACKUP STRATEGY

```bash
# Daily backup
pg_dump -h localhost -U utms_user -d utms -F c -f /backups/utms_$(date +%Y%m%d).dump

# Retention: Keep 7 daily, 4 weekly, 12 monthly backups
```

---

## 9. REVISION HISTORY

| Version | Date | Author | Changes |
|---------|------|--------|---------|
| 1.0 | Mar 2026 | Group 05 | Initial version |
