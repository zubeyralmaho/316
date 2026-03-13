# UTMS - Undergraduate Transfer Management System
## 03 - API Specification

**Version:** 1.0  
**Last Updated:** March 2026  
**Base URL:** `https://api.utms.iyte.edu.tr/api/v1`

---

## 1. API OVERVIEW

### 1.1 General Information

| Property | Value |
|----------|-------|
| Protocol | HTTPS (TLS 1.2+) |
| Format | JSON |
| Authentication | JWT Bearer Token |
| API Version | v1 |
| Rate Limiting | 100 requests/minute per user |

### 1.2 Common Headers

**Request Headers:**
```http
Content-Type: application/json
Accept: application/json
Authorization: Bearer <jwt_token>
Accept-Language: tr-TR, en-US
X-Request-ID: <uuid>
```

**Response Headers:**
```http
Content-Type: application/json
X-Request-ID: <uuid>
X-RateLimit-Limit: 100
X-RateLimit-Remaining: 95
X-RateLimit-Reset: 1234567890
```

### 1.3 Standard Response Format

**Success Response:**
```json
{
  "success": true,
  "data": { ... },
  "message": "Operation completed successfully",
  "timestamp": "2026-03-13T10:30:00Z"
}
```

**Error Response:**
```json
{
  "success": false,
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Validation failed",
    "details": [
      {
        "field": "email",
        "message": "Invalid email format"
      }
    ]
  },
  "timestamp": "2026-03-13T10:30:00Z"
}
```

### 1.4 HTTP Status Codes

| Code | Meaning | Usage |
|------|---------|-------|
| 200 | OK | Successful GET, PUT, PATCH |
| 201 | Created | Successful POST |
| 204 | No Content | Successful DELETE |
| 400 | Bad Request | Validation error |
| 401 | Unauthorized | Missing/invalid token |
| 403 | Forbidden | Insufficient permissions |
| 404 | Not Found | Resource not found |
| 409 | Conflict | Duplicate resource |
| 422 | Unprocessable Entity | Business rule violation |
| 429 | Too Many Requests | Rate limit exceeded |
| 500 | Internal Server Error | Server error |

### 1.5 Error Codes

| Code | Description |
|------|-------------|
| AUTH_001 | Invalid credentials |
| AUTH_002 | Token expired |
| AUTH_003 | Token invalid |
| AUTH_004 | Account locked |
| AUTH_005 | Account not verified |
| VAL_001 | Required field missing |
| VAL_002 | Invalid format |
| VAL_003 | Value out of range |
| APP_001 | Application not found |
| APP_002 | Application already exists |
| APP_003 | Application period closed |
| APP_004 | Invalid status transition |
| DOC_001 | Document not found |
| DOC_002 | Invalid file type |
| DOC_003 | File size exceeded |
| WF_001 | Invalid workflow action |
| WF_002 | Unauthorized stage access |

---

## 2. AUTHENTICATION ENDPOINTS

### 2.1 POST /auth/login

User authentication.

**Request:**
```http
POST /api/v1/auth/login
Content-Type: application/json

{
  "email": "student@example.com",
  "password": "SecurePassword123!"
}
```

**Response (200 OK):**
```json
{
  "success": true,
  "data": {
    "accessToken": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
    "refreshToken": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
    "tokenType": "Bearer",
    "expiresIn": 1800,
    "user": {
      "id": 1,
      "email": "student@example.com",
      "firstName": "Ahmet",
      "lastName": "Yılmaz",
      "role": "STUDENT",
      "lastLoginAt": "2026-03-12T14:30:00Z"
    }
  },
  "message": "Login successful"
}
```

**Error Responses:**

| Status | Code | Message |
|--------|------|---------|
| 401 | AUTH_001 | Invalid email or password |
| 403 | AUTH_004 | Account locked due to failed attempts |
| 403 | AUTH_005 | Account not activated |

---

### 2.2 POST /auth/register

New user registration.

**Request:**
```http
POST /api/v1/auth/register
Content-Type: application/json

{
  "email": "student@example.com",
  "password": "SecurePassword123!",
  "confirmPassword": "SecurePassword123!",
  "firstName": "Ahmet",
  "lastName": "Yılmaz",
  "phone": "+905551234567",
  "kvkkConsent": true
}
```

**Validation Rules:**
- `email`: Required, valid email format, unique
- `password`: Required, min 8 chars, 1 uppercase, 1 lowercase, 1 digit, 1 special char
- `confirmPassword`: Must match password
- `firstName`: Required, 2-100 chars
- `lastName`: Required, 2-100 chars
- `phone`: Optional, valid Turkish phone format
- `kvkkConsent`: Required, must be true

**Response (201 Created):**
```json
{
  "success": true,
  "data": {
    "id": 1,
    "email": "student@example.com",
    "firstName": "Ahmet",
    "lastName": "Yılmaz",
    "role": "STUDENT",
    "createdAt": "2026-03-13T10:30:00Z"
  },
  "message": "Registration successful. Please check your email for verification."
}
```

---

### 2.3 POST /auth/refresh

Refresh access token.

**Request:**
```http
POST /api/v1/auth/refresh
Content-Type: application/json

{
  "refreshToken": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."
}
```

**Response (200 OK):**
```json
{
  "success": true,
  "data": {
    "accessToken": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
    "expiresIn": 1800
  }
}
```

---

### 2.4 POST /auth/logout

User logout (invalidate tokens).

**Request:**
```http
POST /api/v1/auth/logout
Authorization: Bearer <token>
```

**Response (200 OK):**
```json
{
  "success": true,
  "message": "Logged out successfully"
}
```

---

### 2.5 POST /auth/forgot-password

Request password reset.

**Request:**
```http
POST /api/v1/auth/forgot-password
Content-Type: application/json

{
  "email": "student@example.com"
}
```

**Response (200 OK):**
```json
{
  "success": true,
  "message": "If an account exists, a password reset email has been sent"
}
```

---

### 2.6 POST /auth/reset-password

Reset password with token.

**Request:**
```http
POST /api/v1/auth/reset-password
Content-Type: application/json

{
  "token": "abc123def456...",
  "password": "NewSecurePassword123!",
  "confirmPassword": "NewSecurePassword123!"
}
```

**Response (200 OK):**
```json
{
  "success": true,
  "message": "Password reset successful"
}
```

---

### 2.7 PUT /auth/change-password

Change password (authenticated).

**Request:**
```http
PUT /api/v1/auth/change-password
Authorization: Bearer <token>
Content-Type: application/json

{
  "currentPassword": "OldPassword123!",
  "newPassword": "NewPassword456!",
  "confirmPassword": "NewPassword456!"
}
```

**Response (200 OK):**
```json
{
  "success": true,
  "message": "Password changed successfully"
}
```

---

## 3. USER ENDPOINTS

### 3.1 GET /users/me

Get current user profile.

**Request:**
```http
GET /api/v1/users/me
Authorization: Bearer <token>
```

**Response (200 OK):**
```json
{
  "success": true,
  "data": {
    "id": 1,
    "email": "student@example.com",
    "firstName": "Ahmet",
    "lastName": "Yılmaz",
    "phone": "+905551234567",
    "role": "STUDENT",
    "isActive": true,
    "kvkkConsent": true,
    "kvkkConsentDate": "2026-03-01T10:00:00Z",
    "lastLoginAt": "2026-03-13T08:00:00Z",
    "createdAt": "2026-03-01T10:00:00Z"
  }
}
```

---

### 3.2 PUT /users/me

Update current user profile.

**Request:**
```http
PUT /api/v1/users/me
Authorization: Bearer <token>
Content-Type: application/json

{
  "firstName": "Ahmet",
  "lastName": "Yılmaz",
  "phone": "+905559876543"
}
```

**Response (200 OK):**
```json
{
  "success": true,
  "data": {
    "id": 1,
    "email": "student@example.com",
    "firstName": "Ahmet",
    "lastName": "Yılmaz",
    "phone": "+905559876543",
    "updatedAt": "2026-03-13T10:30:00Z"
  },
  "message": "Profile updated successfully"
}
```

---

### 3.3 GET /users (Admin Only)

List all users with pagination and filtering.

**Request:**
```http
GET /api/v1/users?page=0&size=20&role=STUDENT&search=ahmet
Authorization: Bearer <admin_token>
```

**Query Parameters:**

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| page | integer | 0 | Page number (0-indexed) |
| size | integer | 20 | Page size (max 100) |
| sort | string | createdAt,desc | Sort field and direction |
| role | string | - | Filter by role |
| search | string | - | Search by name/email |
| isActive | boolean | - | Filter by status |

**Response (200 OK):**
```json
{
  "success": true,
  "data": {
    "content": [
      {
        "id": 1,
        "email": "student@example.com",
        "firstName": "Ahmet",
        "lastName": "Yılmaz",
        "role": "STUDENT",
        "isActive": true,
        "createdAt": "2026-03-01T10:00:00Z"
      }
    ],
    "page": 0,
    "size": 20,
    "totalElements": 150,
    "totalPages": 8,
    "first": true,
    "last": false
  }
}
```

---

## 4. APPLICATION ENDPOINTS

### 4.1 POST /applications

Create new application.

**Request:**
```http
POST /api/v1/applications
Authorization: Bearer <student_token>
Content-Type: application/json

{
  "departmentId": 1,
  "applicationTerm": "FALL",
  "sourceUniversity": "Ege Üniversitesi",
  "sourceFaculty": "Mühendislik Fakültesi",
  "sourceDepartment": "Bilgisayar Mühendisliği",
  "currentSemester": 4,
  "yksScore": 425.5,
  "yksRanking": 15000,
  "gpa": 3.45,
  "languageCertType": "YDS",
  "languageScore": 75
}
```

**Response (201 Created):**
```json
{
  "success": true,
  "data": {
    "id": 1,
    "applicationNumber": "2026-FALL-CENG-0001",
    "status": "DRAFT",
    "currentStage": "OIDB_INITIAL",
    "department": {
      "id": 1,
      "name": "Computer Engineering",
      "code": "CENG",
      "faculty": {
        "id": 1,
        "name": "Faculty of Engineering"
      }
    },
    "sourceUniversity": "Ege Üniversitesi",
    "sourceDepartment": "Bilgisayar Mühendisliği",
    "yksScore": 425.5,
    "yksRanking": 15000,
    "gpa": 3.45,
    "createdAt": "2026-03-13T10:30:00Z"
  },
  "message": "Application created as draft"
}
```

---

### 4.2 GET /applications

List applications (filtered by role).

**Request (Student):**
```http
GET /api/v1/applications
Authorization: Bearer <student_token>
```

**Request (Staff):**
```http
GET /api/v1/applications?stage=OIDB_INITIAL&status=SUBMITTED&page=0&size=20
Authorization: Bearer <staff_token>
```

**Query Parameters:**

| Parameter | Type | Description | Access |
|-----------|------|-------------|--------|
| page | integer | Page number | All |
| size | integer | Page size | All |
| status | string | Filter by status | Staff |
| stage | string | Filter by workflow stage | Staff |
| departmentId | integer | Filter by department | Staff |
| year | integer | Filter by year | Staff |
| term | string | Filter by term (FALL/SPRING) | Staff |
| search | string | Search by student name | Staff |
| assignedToMe | boolean | Show only assigned to current user | Staff |

**Response (200 OK):**
```json
{
  "success": true,
  "data": {
    "content": [
      {
        "id": 1,
        "applicationNumber": "2026-FALL-CENG-0001",
        "status": "SUBMITTED",
        "currentStage": "OIDB_INITIAL",
        "applicant": {
          "id": 1,
          "firstName": "Ahmet",
          "lastName": "Yılmaz",
          "email": "ahmet@example.com"
        },
        "department": {
          "id": 1,
          "name": "Computer Engineering",
          "code": "CENG"
        },
        "sourceUniversity": "Ege Üniversitesi",
        "gpa": 3.45,
        "transferGrade": null,
        "submittedAt": "2026-03-13T10:30:00Z",
        "documentsCount": 5,
        "pendingDocumentsCount": 2
      }
    ],
    "page": 0,
    "size": 20,
    "totalElements": 45,
    "totalPages": 3
  }
}
```

---

### 4.3 GET /applications/{id}

Get application details.

**Request:**
```http
GET /api/v1/applications/1
Authorization: Bearer <token>
```

**Response (200 OK):**
```json
{
  "success": true,
  "data": {
    "id": 1,
    "applicationNumber": "2026-FALL-CENG-0001",
    "applicationYear": 2026,
    "applicationTerm": "FALL",
    "status": "DOCUMENTS_UNDER_REVIEW",
    "currentStage": "OIDB_INITIAL",
    "applicant": {
      "id": 1,
      "firstName": "Ahmet",
      "lastName": "Yılmaz",
      "email": "ahmet@example.com",
      "phone": "+905551234567"
    },
    "department": {
      "id": 1,
      "name": "Computer Engineering",
      "code": "CENG",
      "quota": 5,
      "faculty": {
        "id": 1,
        "name": "Faculty of Engineering"
      }
    },
    "academicInfo": {
      "sourceUniversity": "Ege Üniversitesi",
      "sourceFaculty": "Mühendislik Fakültesi",
      "sourceDepartment": "Bilgisayar Mühendisliği",
      "currentSemester": 4,
      "yksScore": 425.5,
      "yksRanking": 15000,
      "gpa": 3.45,
      "transferGrade": 87.5432
    },
    "languageInfo": {
      "certificateType": "YDS",
      "score": 75,
      "approved": null,
      "evaluatedAt": null
    },
    "ranking": {
      "isAsil": null,
      "position": null
    },
    "assignedTo": {
      "id": 5,
      "firstName": "Mehmet",
      "lastName": "Demir"
    },
    "documents": [
      {
        "id": 1,
        "type": "TRANSCRIPT",
        "fileName": "transcript.pdf",
        "status": "APPROVED",
        "uploadedAt": "2026-03-13T10:35:00Z"
      },
      {
        "id": 2,
        "type": "YKS_RESULT",
        "fileName": "yks_sonuc.pdf",
        "status": "PENDING",
        "uploadedAt": "2026-03-13T10:36:00Z"
      }
    ],
    "workflowHistory": [
      {
        "id": 1,
        "fromStage": null,
        "toStage": "OIDB_INITIAL",
        "action": "APPLICATION_SUBMITTED",
        "performedBy": "Ahmet Yılmaz",
        "createdAt": "2026-03-13T10:30:00Z"
      }
    ],
    "timestamps": {
      "createdAt": "2026-03-13T10:00:00Z",
      "submittedAt": "2026-03-13T10:30:00Z",
      "updatedAt": "2026-03-13T11:00:00Z"
    }
  }
}
```

---

### 4.4 PUT /applications/{id}

Update application (draft only).

**Request:**
```http
PUT /api/v1/applications/1
Authorization: Bearer <student_token>
Content-Type: application/json

{
  "sourceUniversity": "Dokuz Eylül Üniversitesi",
  "sourceDepartment": "Yazılım Mühendisliği",
  "gpa": 3.50
}
```

**Response (200 OK):**
```json
{
  "success": true,
  "data": {
    "id": 1,
    "status": "DRAFT",
    "sourceUniversity": "Dokuz Eylül Üniversitesi",
    "sourceDepartment": "Yazılım Mühendisliği",
    "gpa": 3.50,
    "updatedAt": "2026-03-13T11:00:00Z"
  },
  "message": "Application updated"
}
```

---

### 4.5 POST /applications/{id}/submit

Submit application.

**Request:**
```http
POST /api/v1/applications/1/submit
Authorization: Bearer <student_token>
```

**Validation:**
- All required documents must be uploaded
- Application must be in DRAFT status
- Application period must be open

**Response (200 OK):**
```json
{
  "success": true,
  "data": {
    "id": 1,
    "applicationNumber": "2026-FALL-CENG-0001",
    "status": "SUBMITTED",
    "currentStage": "OIDB_INITIAL",
    "submittedAt": "2026-03-13T11:30:00Z"
  },
  "message": "Application submitted successfully"
}
```

---

### 4.6 POST /applications/{id}/withdraw

Withdraw application.

**Request:**
```http
POST /api/v1/applications/1/withdraw
Authorization: Bearer <student_token>
Content-Type: application/json

{
  "reason": "Personal reasons"
}
```

**Response (200 OK):**
```json
{
  "success": true,
  "data": {
    "id": 1,
    "status": "WITHDRAWN",
    "withdrawnAt": "2026-03-13T12:00:00Z"
  },
  "message": "Application withdrawn"
}
```

---

### 4.7 GET /applications/{id}/status

Get application status with timeline.

**Request:**
```http
GET /api/v1/applications/1/status
Authorization: Bearer <student_token>
```

**Response (200 OK):**
```json
{
  "success": true,
  "data": {
    "applicationId": 1,
    "applicationNumber": "2026-FALL-CENG-0001",
    "currentStatus": "DOCUMENTS_UNDER_REVIEW",
    "currentStage": "OIDB_INITIAL",
    "progress": 25,
    "timeline": [
      {
        "stage": "OIDB_INITIAL",
        "status": "IN_PROGRESS",
        "enteredAt": "2026-03-13T10:30:00Z",
        "completedAt": null,
        "description": "Initial document review by Registrar's Office"
      },
      {
        "stage": "YDYO_EVALUATION",
        "status": "PENDING",
        "enteredAt": null,
        "completedAt": null,
        "description": "Language proficiency evaluation"
      },
      {
        "stage": "OIDB_POST_LANGUAGE",
        "status": "PENDING",
        "enteredAt": null,
        "completedAt": null,
        "description": "Post-language review"
      },
      {
        "stage": "DEANS_OFFICE_REVIEW",
        "status": "PENDING",
        "enteredAt": null,
        "completedAt": null,
        "description": "Dean's Office review"
      },
      {
        "stage": "YGK_EVALUATION",
        "status": "PENDING",
        "enteredAt": null,
        "completedAt": null,
        "description": "Transfer Committee evaluation"
      },
      {
        "stage": "DEANS_OFFICE_FINAL",
        "status": "PENDING",
        "enteredAt": null,
        "completedAt": null,
        "description": "Faculty Board decision"
      },
      {
        "stage": "OIDB_FINAL",
        "status": "PENDING",
        "enteredAt": null,
        "completedAt": null,
        "description": "Final publication"
      }
    ],
    "estimatedCompletionDays": 14,
    "lastUpdated": "2026-03-13T11:00:00Z"
  }
}
```

---

## 5. DOCUMENT ENDPOINTS

### 5.1 POST /applications/{appId}/documents

Upload document.

**Request:**
```http
POST /api/v1/applications/1/documents
Authorization: Bearer <student_token>
Content-Type: multipart/form-data

file: <binary>
documentType: TRANSCRIPT
```

**Form Data:**

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| file | File | Yes | Document file (PDF, JPG, PNG) |
| documentType | String | Yes | Type of document |

**Allowed File Types:** PDF, JPG, JPEG, PNG  
**Max File Size:** 15 MB

**Response (201 Created):**
```json
{
  "success": true,
  "data": {
    "id": 1,
    "applicationId": 1,
    "documentType": "TRANSCRIPT",
    "fileName": "a1b2c3d4-transcript.pdf",
    "originalFileName": "not_belgesi.pdf",
    "fileSize": 1245678,
    "mimeType": "application/pdf",
    "status": "PENDING",
    "uploadedAt": "2026-03-13T10:35:00Z"
  },
  "message": "Document uploaded successfully"
}
```

---

### 5.2 GET /applications/{appId}/documents

List application documents.

**Request:**
```http
GET /api/v1/applications/1/documents
Authorization: Bearer <token>
```

**Response (200 OK):**
```json
{
  "success": true,
  "data": [
    {
      "id": 1,
      "documentType": "TRANSCRIPT",
      "originalFileName": "not_belgesi.pdf",
      "fileSize": 1245678,
      "status": "APPROVED",
      "reviewComments": null,
      "uploadedAt": "2026-03-13T10:35:00Z",
      "reviewedAt": "2026-03-13T14:00:00Z",
      "reviewedBy": {
        "id": 5,
        "firstName": "Mehmet",
        "lastName": "Demir"
      }
    },
    {
      "id": 2,
      "documentType": "YKS_RESULT",
      "originalFileName": "yks_sonuc.pdf",
      "fileSize": 856432,
      "status": "PENDING",
      "reviewComments": null,
      "uploadedAt": "2026-03-13T10:36:00Z",
      "reviewedAt": null,
      "reviewedBy": null
    }
  ]
}
```

---

### 5.3 GET /documents/{id}/download

Download document.

**Request:**
```http
GET /api/v1/documents/1/download
Authorization: Bearer <token>
```

**Response (200 OK):**
```http
Content-Type: application/pdf
Content-Disposition: attachment; filename="transcript.pdf"
Content-Length: 1245678

<binary data>
```

---

### 5.4 PUT /documents/{id}/review (Staff Only)

Review document.

**Request:**
```http
PUT /api/v1/documents/1/review
Authorization: Bearer <staff_token>
Content-Type: application/json

{
  "status": "APPROVED",
  "comments": "Document verified successfully"
}
```

**Allowed Status Values:** `APPROVED`, `REJECTED`, `REQUIRES_RESUBMISSION`

**Response (200 OK):**
```json
{
  "success": true,
  "data": {
    "id": 1,
    "status": "APPROVED",
    "reviewComments": "Document verified successfully",
    "reviewedAt": "2026-03-13T14:00:00Z",
    "reviewedBy": {
      "id": 5,
      "firstName": "Mehmet",
      "lastName": "Demir"
    }
  },
  "message": "Document reviewed"
}
```

---

### 5.5 DELETE /documents/{id}

Delete document (draft applications only).

**Request:**
```http
DELETE /api/v1/documents/1
Authorization: Bearer <student_token>
```

**Response (204 No Content)**

---

## 6. WORKFLOW ENDPOINTS

### 6.1 POST /applications/{id}/transfer (Staff Only)

Transfer application to next stage.

**Request:**
```http
POST /api/v1/applications/1/transfer
Authorization: Bearer <staff_token>
Content-Type: application/json

{
  "action": "APPROVE_AND_FORWARD",
  "toStage": "YDYO_EVALUATION",
  "notes": "All documents verified. Forwarding for language evaluation."
}
```

**Actions:**

| Action | Description |
|--------|-------------|
| APPROVE_AND_FORWARD | Approve and send to next stage |
| REJECT | Reject application |
| REQUEST_DOCUMENTS | Request additional documents |
| RETURN_TO_PREVIOUS | Return to previous stage |

**Response (200 OK):**
```json
{
  "success": true,
  "data": {
    "id": 1,
    "status": "PENDING_LANGUAGE_EVALUATION",
    "currentStage": "YDYO_EVALUATION",
    "previousStage": "OIDB_INITIAL",
    "transferredAt": "2026-03-13T15:00:00Z",
    "transferredBy": {
      "id": 5,
      "firstName": "Mehmet",
      "lastName": "Demir"
    }
  },
  "message": "Application transferred to YDYO"
}
```

---

### 6.2 PUT /applications/{id}/language-evaluation (YDYO Only)

Submit language evaluation result.

**Request:**
```http
PUT /api/v1/applications/1/language-evaluation
Authorization: Bearer <ydyo_token>
Content-Type: application/json

{
  "approved": true,
  "notes": "YDS score meets requirements"
}
```

**Response (200 OK):**
```json
{
  "success": true,
  "data": {
    "id": 1,
    "languageApproved": true,
    "status": "LANGUAGE_APPROVED",
    "evaluatedAt": "2026-03-14T10:00:00Z"
  },
  "message": "Language evaluation submitted"
}
```

---

### 6.3 GET /applications/{id}/workflow-history

Get workflow history.

**Request:**
```http
GET /api/v1/applications/1/workflow-history
Authorization: Bearer <token>
```

**Response (200 OK):**
```json
{
  "success": true,
  "data": [
    {
      "id": 1,
      "fromStage": null,
      "toStage": "OIDB_INITIAL",
      "action": "APPLICATION_SUBMITTED",
      "performedBy": {
        "id": 1,
        "firstName": "Ahmet",
        "lastName": "Yılmaz",
        "role": "STUDENT"
      },
      "notes": null,
      "createdAt": "2026-03-13T10:30:00Z"
    },
    {
      "id": 2,
      "fromStage": "OIDB_INITIAL",
      "toStage": "YDYO_EVALUATION",
      "action": "APPROVE_AND_FORWARD",
      "performedBy": {
        "id": 5,
        "firstName": "Mehmet",
        "lastName": "Demir",
        "role": "OIDB_STAFF"
      },
      "notes": "Documents verified. Forwarding for language evaluation.",
      "createdAt": "2026-03-13T15:00:00Z"
    }
  ]
}
```

---

## 7. EVALUATION ENDPOINTS

### 7.1 POST /applications/{id}/calculate-grade (Staff Only)

Calculate transfer grade.

**Request:**
```http
POST /api/v1/applications/1/calculate-grade
Authorization: Bearer <staff_token>
```

**Response (200 OK):**
```json
{
  "success": true,
  "data": {
    "applicationId": 1,
    "yksScore": 425.5,
    "minYksScore": 380.0,
    "gpa": 3.45,
    "transferGrade": 101.0526,
    "formula": "(425.5 / 380.0 * 100 * 0.9) + (0.1 * 3.45)",
    "calculatedAt": "2026-03-14T11:00:00Z"
  },
  "message": "Transfer grade calculated"
}
```

---

### 7.2 POST /departments/{id}/calculate-rankings (Staff Only)

Calculate rankings for a department.

**Request:**
```http
POST /api/v1/departments/1/calculate-rankings?year=2026&term=FALL
Authorization: Bearer <staff_token>
```

**Response (200 OK):**
```json
{
  "success": true,
  "data": {
    "departmentId": 1,
    "departmentName": "Computer Engineering",
    "applicationYear": 2026,
    "applicationTerm": "FALL",
    "quota": 5,
    "totalApplications": 12,
    "eligibleApplications": 10,
    "rankings": [
      {
        "position": 1,
        "applicationId": 5,
        "applicantName": "Ali Veli",
        "transferGrade": 105.2341,
        "isAsil": true
      },
      {
        "position": 2,
        "applicationId": 3,
        "applicantName": "Ayşe Kaya",
        "transferGrade": 102.1234,
        "isAsil": true
      },
      {
        "position": 6,
        "applicationId": 1,
        "applicantName": "Ahmet Yılmaz",
        "transferGrade": 98.5432,
        "isAsil": false
      }
    ],
    "calculatedAt": "2026-03-14T12:00:00Z"
  },
  "message": "Rankings calculated"
}
```

---

### 7.3 GET /departments/{id}/rankings

Get current rankings.

**Request:**
```http
GET /api/v1/departments/1/rankings?year=2026&term=FALL
Authorization: Bearer <token>
```

**Response (200 OK):**
```json
{
  "success": true,
  "data": {
    "departmentId": 1,
    "departmentName": "Computer Engineering",
    "quota": 5,
    "asil": [
      {
        "position": 1,
        "applicationId": 5,
        "applicantName": "Ali Veli",
        "transferGrade": 105.2341
      }
    ],
    "yedek": [
      {
        "position": 6,
        "applicationId": 1,
        "applicantName": "Ahmet Yılmaz",
        "transferGrade": 98.5432
      }
    ],
    "lastCalculatedAt": "2026-03-14T12:00:00Z"
  }
}
```

---

## 8. INTIBAK ENDPOINTS

### 8.1 GET /applications/{id}/intibak

Get intibak records for application.

**Request:**
```http
GET /api/v1/applications/1/intibak
Authorization: Bearer <token>
```

**Response (200 OK):**
```json
{
  "success": true,
  "data": {
    "applicationId": 1,
    "totalSourceCredits": 45,
    "totalAcceptedCredits": 38,
    "records": [
      {
        "id": 1,
        "sourceCourse": {
          "code": "BIL101",
          "name": "Programlamaya Giriş",
          "credits": 4,
          "grade": "AA",
          "gradePoint": 4.00
        },
        "targetCourse": {
          "code": "CENG111",
          "name": "Introduction to Programming",
          "credits": 4
        },
        "decision": "ACCEPTED",
        "notes": null
      },
      {
        "id": 2,
        "sourceCourse": {
          "code": "MAT101",
          "name": "Matematik I",
          "credits": 5,
          "grade": "BA",
          "gradePoint": 3.50
        },
        "targetCourse": {
          "code": "MATH101",
          "name": "Calculus I",
          "credits": 5
        },
        "decision": "ACCEPTED",
        "notes": null
      }
    ],
    "preparedBy": {
      "id": 10,
      "firstName": "Prof. Dr. Ali",
      "lastName": "Öztürk"
    },
    "preparedAt": "2026-03-15T10:00:00Z"
  }
}
```

---

### 8.2 POST /applications/{id}/intibak (YGK Only)

Add intibak record.

**Request:**
```http
POST /api/v1/applications/1/intibak
Authorization: Bearer <ygk_token>
Content-Type: application/json

{
  "sourceCourseCode": "BIL101",
  "sourceCourseName": "Programlamaya Giriş",
  "sourceCredits": 4,
  "sourceGrade": "AA",
  "sourceGradePoint": 4.00,
  "targetCourseCode": "CENG111",
  "targetCourseName": "Introduction to Programming",
  "targetCredits": 4,
  "decision": "ACCEPTED",
  "notes": null
}
```

**Response (201 Created):**
```json
{
  "success": true,
  "data": {
    "id": 1,
    "applicationId": 1,
    "decision": "ACCEPTED"
  },
  "message": "Intibak record added"
}
```

---

### 8.3 PUT /applications/{id}/intibak/{recordId} (YGK Only)

Update intibak record.

**Request:**
```http
PUT /api/v1/applications/1/intibak/1
Authorization: Bearer <ygk_token>
Content-Type: application/json

{
  "decision": "CONDITIONAL",
  "notes": "Student must take additional lab hours"
}
```

**Response (200 OK):**
```json
{
  "success": true,
  "data": {
    "id": 1,
    "decision": "CONDITIONAL",
    "notes": "Student must take additional lab hours",
    "updatedAt": "2026-03-15T11:00:00Z"
  },
  "message": "Intibak record updated"
}
```

---

### 8.4 POST /applications/{id}/intibak/complete (YGK Only)

Mark intibak as complete.

**Request:**
```http
POST /api/v1/applications/1/intibak/complete
Authorization: Bearer <ygk_token>
```

**Response (200 OK):**
```json
{
  "success": true,
  "data": {
    "applicationId": 1,
    "status": "INTIBAK_COMPLETED",
    "currentStage": "DEANS_OFFICE_FINAL",
    "totalRecords": 15,
    "acceptedRecords": 12,
    "totalAcceptedCredits": 38
  },
  "message": "Intibak completed and forwarded to Dean's Office"
}
```

---

## 9. NOTIFICATION ENDPOINTS

### 9.1 GET /notifications

Get user notifications.

**Request:**
```http
GET /api/v1/notifications?page=0&size=20&unreadOnly=false
Authorization: Bearer <token>
```

**Query Parameters:**

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| page | integer | 0 | Page number |
| size | integer | 20 | Page size |
| unreadOnly | boolean | false | Show only unread |

**Response (200 OK):**
```json
{
  "success": true,
  "data": {
    "content": [
      {
        "id": 1,
        "type": "DOCUMENT_APPROVED",
        "title": "Belge Onaylandı",
        "message": "Transkript belgeniz onaylandı.",
        "priority": "NORMAL",
        "isRead": false,
        "actionUrl": "/applications/1/documents",
        "createdAt": "2026-03-13T14:00:00Z"
      }
    ],
    "unreadCount": 5,
    "page": 0,
    "size": 20,
    "totalElements": 25
  }
}
```

---

### 9.2 PUT /notifications/{id}/read

Mark notification as read.

**Request:**
```http
PUT /api/v1/notifications/1/read
Authorization: Bearer <token>
```

**Response (200 OK):**
```json
{
  "success": true,
  "data": {
    "id": 1,
    "isRead": true,
    "readAt": "2026-03-13T15:30:00Z"
  }
}
```

---

### 9.3 PUT /notifications/read-all

Mark all notifications as read.

**Request:**
```http
PUT /api/v1/notifications/read-all
Authorization: Bearer <token>
```

**Response (200 OK):**
```json
{
  "success": true,
  "data": {
    "markedCount": 5
  },
  "message": "All notifications marked as read"
}
```

---

### 9.4 GET /notifications/unread-count

Get unread notification count.

**Request:**
```http
GET /api/v1/notifications/unread-count
Authorization: Bearer <token>
```

**Response (200 OK):**
```json
{
  "success": true,
  "data": {
    "count": 5
  }
}
```

---

## 10. PUBLIC ENDPOINTS

### 10.1 GET /public/faculties

List all faculties.

**Request:**
```http
GET /api/v1/public/faculties
```

**Response (200 OK):**
```json
{
  "success": true,
  "data": [
    {
      "id": 1,
      "name": "Faculty of Engineering",
      "code": "ENG",
      "departmentsCount": 8
    },
    {
      "id": 2,
      "name": "Faculty of Science",
      "code": "SCI",
      "departmentsCount": 5
    }
  ]
}
```

---

### 10.2 GET /public/departments

List all departments.

**Request:**
```http
GET /api/v1/public/departments?facultyId=1
```

**Response (200 OK):**
```json
{
  "success": true,
  "data": [
    {
      "id": 1,
      "name": "Computer Engineering",
      "code": "CENG",
      "facultyId": 1,
      "facultyName": "Faculty of Engineering",
      "quota": 5,
      "minGpa": 2.50,
      "maxRanking": 300000,
      "languageRequired": true,
      "minLanguageScore": 60
    }
  ]
}
```

---

### 10.3 GET /public/application-period

Get current application period info.

**Request:**
```http
GET /api/v1/public/application-period
```

**Response (200 OK):**
```json
{
  "success": true,
  "data": {
    "year": 2026,
    "term": "FALL",
    "startDate": "2026-03-01",
    "endDate": "2026-03-31",
    "isOpen": true,
    "daysRemaining": 18
  }
}
```

---

## 11. ADMIN ENDPOINTS

### 11.1 GET /admin/dashboard

Get admin dashboard statistics.

**Request:**
```http
GET /api/v1/admin/dashboard
Authorization: Bearer <admin_token>
```

**Response (200 OK):**
```json
{
  "success": true,
  "data": {
    "applications": {
      "total": 150,
      "draft": 20,
      "submitted": 45,
      "underReview": 35,
      "completed": 50
    },
    "byStage": {
      "OIDB_INITIAL": 25,
      "YDYO_EVALUATION": 10,
      "DEANS_OFFICE_REVIEW": 15,
      "YGK_EVALUATION": 8,
      "COMPLETED": 50
    },
    "users": {
      "totalStudents": 200,
      "totalStaff": 25,
      "activeToday": 45
    },
    "documents": {
      "total": 750,
      "pending": 120,
      "approved": 580,
      "rejected": 50
    }
  }
}
```

---

### 11.2 GET /admin/audit-logs

Get audit logs.

**Request:**
```http
GET /api/v1/admin/audit-logs?page=0&size=50&entityType=Application&action=UPDATE
Authorization: Bearer <admin_token>
```

**Response (200 OK):**
```json
{
  "success": true,
  "data": {
    "content": [
      {
        "id": 1,
        "userId": 5,
        "userName": "Mehmet Demir",
        "action": "UPDATE",
        "entityType": "Application",
        "entityId": 1,
        "changes": {
          "status": {
            "from": "SUBMITTED",
            "to": "DOCUMENTS_UNDER_REVIEW"
          }
        },
        "ipAddress": "192.168.1.100",
        "createdAt": "2026-03-13T14:00:00Z"
      }
    ],
    "page": 0,
    "size": 50,
    "totalElements": 1250
  }
}
```

---

## 12. REVISION HISTORY

| Version | Date | Author | Changes |
|---------|------|--------|---------|
| 1.0 | Mar 2026 | Group 05 | Initial version |
