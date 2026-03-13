# UTMS - Undergraduate Transfer Management System
## 01 - System Architecture

**Version:** 1.0  
**Last Updated:** March 2026

---

## 1. ARCHITECTURE OVERVIEW

UTMS follows a **Monolithic Architecture** with a layered design pattern. This architecture was chosen for:

- **Simplicity**: Single codebase, easier development and debugging
- **ACID Transactions**: Full transactional integrity across operations
- **Lower Latency**: No network calls between components
- **Team Size**: Appropriate for small-medium development teams
- **Deployment**: Single deployable artifact

---

## 2. HIGH-LEVEL ARCHITECTURE

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                              CLIENT LAYER                                    │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│   ┌─────────────┐  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐       │
│   │   Student   │  │    OIDB     │  │   Dean's    │  │    YGK      │       │
│   │   Portal    │  │   Portal    │  │   Portal    │  │   Portal    │       │
│   │  (React.js) │  │  (React.js) │  │  (React.js) │  │  (React.js) │       │
│   └──────┬──────┘  └──────┬──────┘  └──────┬──────┘  └──────┬──────┘       │
│          │                │                │                │               │
│          └────────────────┴────────────────┴────────────────┘               │
│                                    │                                         │
│                              HTTPS/REST                                      │
│                                    │                                         │
└────────────────────────────────────┼─────────────────────────────────────────┘
                                     │
┌────────────────────────────────────┼─────────────────────────────────────────┐
│                              SERVER LAYER                                    │
├────────────────────────────────────┼─────────────────────────────────────────┤
│                                    ▼                                         │
│   ┌─────────────────────────────────────────────────────────────────────┐   │
│   │                         API GATEWAY / NGINX                          │   │
│   │                    (Load Balancing, SSL Termination)                 │   │
│   └─────────────────────────────────┬───────────────────────────────────┘   │
│                                     │                                        │
│   ┌─────────────────────────────────▼───────────────────────────────────┐   │
│   │                     SPRING BOOT APPLICATION                          │   │
│   │  ┌───────────────────────────────────────────────────────────────┐  │   │
│   │  │                    CONTROLLER LAYER                            │  │   │
│   │  │  AuthController | ApplicationController | DocumentController  │  │   │
│   │  │  EvaluationController | NotificationController | AdminController│ │   │
│   │  └───────────────────────────────┬───────────────────────────────┘  │   │
│   │                                  │                                   │   │
│   │  ┌───────────────────────────────▼───────────────────────────────┐  │   │
│   │  │                      SERVICE LAYER                             │  │   │
│   │  │  ┌─────────────┐ ┌─────────────┐ ┌─────────────┐              │  │   │
│   │  │  │ Application │ │  Document   │ │ Evaluation  │              │  │   │
│   │  │  │   Service   │ │   Service   │ │   Service   │              │  │   │
│   │  │  └─────────────┘ └─────────────┘ └─────────────┘              │  │   │
│   │  │  ┌─────────────┐ ┌─────────────┐ ┌─────────────┐              │  │   │
│   │  │  │  Workflow   │ │   Status    │ │Notification │              │  │   │
│   │  │  │   Service   │ │   Service   │ │   Service   │              │  │   │
│   │  │  └─────────────┘ └─────────────┘ └─────────────┘              │  │   │
│   │  │  ┌─────────────┐ ┌─────────────┐ ┌─────────────┐              │  │   │
│   │  │  │    User     │ │  Intibak    │ │   Email     │              │  │   │
│   │  │  │   Service   │ │   Service   │ │   Service   │              │  │   │
│   │  │  └─────────────┘ └─────────────┘ └─────────────┘              │  │   │
│   │  └───────────────────────────────┬───────────────────────────────┘  │   │
│   │                                  │                                   │   │
│   │  ┌───────────────────────────────▼───────────────────────────────┐  │   │
│   │  │                    REPOSITORY LAYER                            │  │   │
│   │  │  UserRepository | ApplicationRepository | DocumentRepository   │  │   │
│   │  │  NotificationRepository | IntibakRepository | AuditLogRepository│ │   │
│   │  └───────────────────────────────┬───────────────────────────────┘  │   │
│   │                                  │                                   │   │
│   └──────────────────────────────────┼──────────────────────────────────┘   │
│                                      │                                       │
└──────────────────────────────────────┼───────────────────────────────────────┘
                                       │
┌──────────────────────────────────────┼───────────────────────────────────────┐
│                               DATA LAYER                                     │
├──────────────────────────────────────┼───────────────────────────────────────┤
│                                      ▼                                       │
│   ┌─────────────────────────────────────────────────────────────────────┐   │
│   │                         PostgreSQL Database                          │   │
│   │                      (Primary + Read Replica)                        │   │
│   └─────────────────────────────────────────────────────────────────────┘   │
│                                                                              │
│   ┌─────────────────────────────────────────────────────────────────────┐   │
│   │                         File Storage (NFS/S3)                        │   │
│   │                    /var/utms/documents (encrypted)                   │   │
│   └─────────────────────────────────────────────────────────────────────┘   │
│                                                                              │
└──────────────────────────────────────────────────────────────────────────────┘

┌──────────────────────────────────────────────────────────────────────────────┐
│                          EXTERNAL INTEGRATIONS                               │
├──────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│   ┌─────────────┐    ┌─────────────┐    ┌─────────────┐    ┌─────────────┐  │
│   │   YÖKSİS    │    │    ÖSYM     │    │    UBYS     │    │    SMTP     │  │
│   │     API     │    │     API     │    │     API     │    │   Server    │  │
│   │  (Academic) │    │   (Exams)   │    │  (Student)  │    │   (Email)   │  │
│   └─────────────┘    └─────────────┘    └─────────────┘    └─────────────┘  │
│                                                                              │
└──────────────────────────────────────────────────────────────────────────────┘
```

---

## 3. TECHNOLOGY STACK

### 3.1 Backend Technologies

| Component | Technology | Version | Purpose |
|-----------|------------|---------|---------|
| Runtime | Java | 17+ (LTS) | Application runtime |
| Framework | Spring Boot | 3.2.x | Application framework |
| Security | Spring Security | 6.x | Authentication & Authorization |
| ORM | Spring Data JPA | 3.x | Database access |
| ORM Implementation | Hibernate | 6.x | JPA implementation |
| Validation | Jakarta Validation | 3.x | Input validation |
| API Docs | SpringDoc OpenAPI | 2.x | API documentation |
| Build Tool | Maven | 3.9+ | Build & dependency management |
| Testing | JUnit 5 + Mockito | 5.x | Unit & integration testing |

### 3.2 Frontend Technologies

| Component | Technology | Version | Purpose |
|-----------|------------|---------|---------|
| Framework | React.js | 18.x | UI framework |
| Language | TypeScript | 5.x | Type-safe JavaScript |
| Routing | React Router | 6.x | Client-side routing |
| State | Zustand | 4.x | State management |
| HTTP Client | Axios | 1.x | API communication |
| UI Components | Ant Design | 5.x | Component library |
| Forms | React Hook Form | 7.x | Form management |
| Validation | Zod | 3.x | Schema validation |
| Build Tool | Vite | 5.x | Build & dev server |
| Testing | Vitest + RTL | - | Unit & component testing |

### 3.3 Database & Storage

| Component | Technology | Version | Purpose |
|-----------|------------|---------|---------|
| Database | PostgreSQL | 15+ | Primary data store |
| Connection Pool | HikariCP | 5.x | Connection pooling |
| File Storage | Local/NFS/S3 | - | Document storage |

### 3.4 Infrastructure

| Component | Technology | Purpose |
|-----------|------------|---------|
| Web Server | Nginx | Reverse proxy, SSL, load balancing |
| Containerization | Docker | Application packaging |
| Orchestration | Docker Compose | Multi-container management |
| CI/CD | GitHub Actions | Automated build & deploy |
| Monitoring | Prometheus + Grafana | Metrics & visualization |
| Logging | ELK Stack | Centralized logging |

---

## 4. LAYER DESCRIPTIONS

### 4.1 Controller Layer

**Responsibility:** Handle HTTP requests, validate input, delegate to services

```
Controllers:
├── AuthController          # Login, register, password reset
├── ApplicationController   # Application CRUD, submission
├── DocumentController      # Document upload, download, review
├── EvaluationController    # Eligibility check, rankings
├── WorkflowController      # Stage transfers, workflow actions
├── NotificationController  # User notifications
├── IntibakController       # Course equivalence management
├── AdminController         # System administration
└── PublicController        # Public endpoints (departments, quotas)
```

### 4.2 Service Layer

**Responsibility:** Business logic, transaction management, orchestration

```
Services:
├── UserService             # User management, authentication
├── ApplicationService      # Application lifecycle management
├── DocumentService         # Document processing, storage
├── DocumentStorageService  # File system operations
├── EvaluationService       # Eligibility, scoring, ranking
├── WorkflowService         # Workflow state machine
├── StatusService           # Application status management
├── NotificationService     # Notification creation & delivery
├── EmailService            # Email sending
├── IntibakService          # Course equivalence management
├── IntegrationService      # External API communication
├── AuditService            # Audit logging
└── ReportService           # Report generation
```

### 4.3 Repository Layer

**Responsibility:** Data access, CRUD operations, queries

```
Repositories:
├── UserRepository
├── ApplicationRepository
├── DocumentRepository
├── NotificationRepository
├── WorkflowHistoryRepository
├── IntibakRecordRepository
├── DepartmentRepository
├── FacultyRepository
├── AuditLogRepository
└── PasswordResetTokenRepository
```

### 4.4 Entity Layer

**Responsibility:** Domain model, JPA entities

```
Entities:
├── User                    # Base user entity
├── Application             # Transfer application
├── Document                # Uploaded document
├── Notification            # User notification
├── WorkflowHistory         # Workflow audit trail
├── IntibakRecord           # Course equivalence record
├── Department              # Academic department
├── Faculty                 # Faculty/college
├── AuditLog                # System audit log
└── PasswordResetToken      # Password reset tokens
```

---

## 5. PACKAGE STRUCTURE

### 5.1 Backend Package Structure

```
src/main/java/com/iztech/utms/
├── UtmsApplication.java                 # Spring Boot main class
│
├── config/                              # Configuration classes
│   ├── SecurityConfig.java
│   ├── WebConfig.java
│   ├── JpaConfig.java
│   ├── AsyncConfig.java
│   ├── CacheConfig.java
│   └── OpenApiConfig.java
│
├── controller/                          # REST controllers
│   ├── AuthController.java
│   ├── ApplicationController.java
│   ├── DocumentController.java
│   ├── EvaluationController.java
│   ├── WorkflowController.java
│   ├── NotificationController.java
│   ├── IntibakController.java
│   ├── AdminController.java
│   └── PublicController.java
│
├── service/                             # Business services
│   ├── UserService.java
│   ├── ApplicationService.java
│   ├── DocumentService.java
│   ├── DocumentStorageService.java
│   ├── EvaluationService.java
│   ├── WorkflowService.java
│   ├── StatusService.java
│   ├── NotificationService.java
│   ├── EmailService.java
│   ├── IntibakService.java
│   ├── IntegrationService.java
│   ├── AuditService.java
│   └── ReportService.java
│
├── repository/                          # JPA repositories
│   ├── UserRepository.java
│   ├── ApplicationRepository.java
│   ├── DocumentRepository.java
│   ├── NotificationRepository.java
│   ├── WorkflowHistoryRepository.java
│   ├── IntibakRecordRepository.java
│   ├── DepartmentRepository.java
│   ├── FacultyRepository.java
│   ├── AuditLogRepository.java
│   └── PasswordResetTokenRepository.java
│
├── entity/                              # JPA entities
│   ├── User.java
│   ├── Application.java
│   ├── Document.java
│   ├── Notification.java
│   ├── WorkflowHistory.java
│   ├── IntibakRecord.java
│   ├── Department.java
│   ├── Faculty.java
│   ├── AuditLog.java
│   └── PasswordResetToken.java
│
├── dto/                                 # Data Transfer Objects
│   ├── request/
│   │   ├── LoginRequest.java
│   │   ├── RegisterRequest.java
│   │   ├── CreateApplicationRequest.java
│   │   ├── UpdateApplicationRequest.java
│   │   ├── TransferRequest.java
│   │   ├── ReviewDocumentRequest.java
│   │   └── IntibakRequest.java
│   │
│   └── response/
│       ├── LoginResponse.java
│       ├── UserResponse.java
│       ├── ApplicationResponse.java
│       ├── ApplicationListResponse.java
│       ├── DocumentResponse.java
│       ├── EligibilityResponse.java
│       ├── RankingResponse.java
│       ├── NotificationResponse.java
│       └── ErrorResponse.java
│
├── enums/                               # Enumerations
│   ├── UserRole.java
│   ├── ApplicationStatus.java
│   ├── WorkflowStage.java
│   ├── DocumentType.java
│   ├── DocumentStatus.java
│   └── NotificationType.java
│
├── mapper/                              # Entity-DTO mappers
│   ├── UserMapper.java
│   ├── ApplicationMapper.java
│   ├── DocumentMapper.java
│   └── NotificationMapper.java
│
├── security/                            # Security components
│   ├── JwtTokenProvider.java
│   ├── JwtAuthenticationFilter.java
│   ├── CustomUserDetails.java
│   ├── CustomUserDetailsService.java
│   └── ApplicationSecurity.java
│
├── integration/                         # External API clients
│   ├── YoksisClient.java
│   ├── OsymClient.java
│   └── UbysClient.java
│
├── exception/                           # Exception handling
│   ├── GlobalExceptionHandler.java
│   ├── ApplicationNotFoundException.java
│   ├── DocumentNotFoundException.java
│   ├── UserNotFoundException.java
│   ├── InvalidCredentialsException.java
│   ├── AccountLockedException.java
│   ├── WorkflowException.java
│   └── ValidationException.java
│
├── util/                                # Utilities
│   ├── TransferGradeCalculator.java
│   ├── ValidationUtils.java
│   ├── FileUtils.java
│   └── DateUtils.java
│
└── event/                               # Application events
    ├── ApplicationSubmittedEvent.java
    ├── DocumentUploadedEvent.java
    ├── StatusChangedEvent.java
    └── EventListener.java
```

### 5.2 Frontend Directory Structure

```
src/
├── main.tsx                             # Application entry point
├── App.tsx                              # Root component
├── vite-env.d.ts                        # Vite types
│
├── api/                                 # API layer
│   ├── client.ts                        # Axios instance
│   ├── auth.api.ts
│   ├── application.api.ts
│   ├── document.api.ts
│   ├── evaluation.api.ts
│   ├── notification.api.ts
│   └── admin.api.ts
│
├── components/                          # Reusable components
│   ├── common/
│   │   ├── Button/
│   │   ├── Input/
│   │   ├── Modal/
│   │   ├── Table/
│   │   ├── Card/
│   │   ├── Loading/
│   │   └── ErrorBoundary/
│   │
│   ├── layout/
│   │   ├── Header/
│   │   ├── Sidebar/
│   │   ├── Footer/
│   │   ├── StudentLayout/
│   │   └── AdminLayout/
│   │
│   ├── auth/
│   │   ├── LoginForm/
│   │   ├── RegisterForm/
│   │   └── ResetPasswordForm/
│   │
│   ├── application/
│   │   ├── ApplicationForm/
│   │   ├── ApplicationCard/
│   │   ├── ApplicationList/
│   │   ├── ApplicationDetail/
│   │   └── StatusBadge/
│   │
│   ├── document/
│   │   ├── DocumentUploader/
│   │   ├── DocumentList/
│   │   ├── DocumentViewer/
│   │   └── DocumentReview/
│   │
│   ├── workflow/
│   │   ├── WorkflowTimeline/
│   │   ├── StageIndicator/
│   │   └── TransferButton/
│   │
│   ├── intibak/
│   │   ├── IntibakForm/
│   │   ├── CourseMapping/
│   │   └── IntibakTable/
│   │
│   └── notification/
│       ├── NotificationBell/
│       ├── NotificationList/
│       └── NotificationItem/
│
├── pages/                               # Page components
│   ├── auth/
│   │   ├── LoginPage.tsx
│   │   ├── RegisterPage.tsx
│   │   └── ResetPasswordPage.tsx
│   │
│   ├── student/
│   │   ├── DashboardPage.tsx
│   │   ├── NewApplicationPage.tsx
│   │   ├── ApplicationStatusPage.tsx
│   │   └── DocumentsPage.tsx
│   │
│   ├── staff/
│   │   ├── DashboardPage.tsx
│   │   ├── ApplicationListPage.tsx
│   │   ├── ApplicationDetailPage.tsx
│   │   ├── DocumentReviewPage.tsx
│   │   └── IntibakPage.tsx
│   │
│   ├── admin/
│   │   ├── DashboardPage.tsx
│   │   ├── UsersPage.tsx
│   │   ├── DepartmentsPage.tsx
│   │   ├── ReportsPage.tsx
│   │   └── SettingsPage.tsx
│   │
│   └── error/
│       ├── NotFoundPage.tsx
│       └── ErrorPage.tsx
│
├── hooks/                               # Custom hooks
│   ├── useAuth.ts
│   ├── useApplication.ts
│   ├── useDocuments.ts
│   ├── useNotifications.ts
│   ├── useDebounce.ts
│   └── usePagination.ts
│
├── store/                               # State management
│   ├── authStore.ts
│   ├── applicationStore.ts
│   ├── notificationStore.ts
│   └── uiStore.ts
│
├── types/                               # TypeScript types
│   ├── auth.types.ts
│   ├── application.types.ts
│   ├── document.types.ts
│   ├── user.types.ts
│   ├── notification.types.ts
│   └── api.types.ts
│
├── utils/                               # Utilities
│   ├── constants.ts
│   ├── validators.ts
│   ├── formatters.ts
│   ├── storage.ts
│   └── helpers.ts
│
├── routes/                              # Routing
│   ├── index.tsx
│   ├── PrivateRoute.tsx
│   ├── RoleRoute.tsx
│   └── routes.config.ts
│
└── styles/                              # Global styles
    ├── global.css
    ├── variables.css
    └── antd-overrides.css
```

---

## 6. DATA FLOW

### 6.1 Request Flow

```
┌─────────┐    ┌─────────┐    ┌────────────┐    ┌─────────┐    ┌────────────┐
│ Browser │───▶│  Nginx  │───▶│ Controller │───▶│ Service │───▶│ Repository │
│         │    │         │    │            │    │         │    │            │
└─────────┘    └─────────┘    └────────────┘    └─────────┘    └────────────┘
                                    │                │               │
                                    │                │               ▼
                                    │                │         ┌──────────┐
                                    │                │         │ Database │
                                    │                │         └──────────┘
                                    │                │
                                    │                ▼
                                    │         ┌─────────────┐
                                    │         │  External   │
                                    │         │    APIs     │
                                    │         └─────────────┘
                                    │
                                    ▼
                              ┌──────────┐
                              │   DTO    │
                              │ Response │
                              └──────────┘
```

### 6.2 Authentication Flow

```
┌──────────┐     ┌──────────────┐     ┌─────────────┐     ┌──────────────┐
│  Client  │────▶│ POST /login  │────▶│ UserService │────▶│ UserRepository│
└──────────┘     └──────────────┘     └─────────────┘     └──────────────┘
                                             │
                                             ▼
                                    ┌─────────────────┐
                                    │ Validate Password│
                                    │   (BCrypt)       │
                                    └────────┬────────┘
                                             │
                                             ▼
                                    ┌─────────────────┐
                                    │ Generate JWT    │
                                    │ Access + Refresh│
                                    └────────┬────────┘
                                             │
                                             ▼
                                    ┌─────────────────┐
                                    │ Return Tokens   │
                                    │ + User Info     │
                                    └─────────────────┘
```

### 6.3 Application Submission Flow

```
┌──────────┐
│ Student  │
└────┬─────┘
     │ 1. Fill form & upload docs
     ▼
┌────────────────────┐
│ ApplicationService │
└────────┬───────────┘
         │ 2. Validate data
         ▼
┌────────────────────┐     ┌─────────────┐
│ EvaluationService  │────▶│ YÖKSİS/ÖSYM │
└────────┬───────────┘     └─────────────┘
         │ 3. Check eligibility
         ▼
┌────────────────────┐
│   StatusService    │
└────────┬───────────┘
         │ 4. Update status
         ▼
┌────────────────────┐
│ NotificationService│
└────────┬───────────┘
         │ 5. Send confirmation
         ▼
┌────────────────────┐
│   WorkflowService  │
└────────────────────┘
         │ 6. Initialize workflow
         ▼
    [OIDB_INITIAL]
```

---

## 7. SECURITY ARCHITECTURE

### 7.1 Authentication Mechanism

```
┌─────────────────────────────────────────────────────────────┐
│                    JWT AUTHENTICATION                        │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  Access Token:                                               │
│  ┌─────────────────────────────────────────────────────┐    │
│  │ Header: { alg: HS256, typ: JWT }                     │    │
│  │ Payload: {                                           │    │
│  │   sub: userId,                                       │    │
│  │   email: "user@example.com",                         │    │
│  │   role: "STUDENT",                                   │    │
│  │   iat: 1234567890,                                   │    │
│  │   exp: 1234569690  (30 min)                         │    │
│  │ }                                                    │    │
│  │ Signature: HMACSHA256(...)                           │    │
│  └─────────────────────────────────────────────────────┘    │
│                                                              │
│  Refresh Token:                                              │
│  ┌─────────────────────────────────────────────────────┐    │
│  │ Expiry: 7 days                                       │    │
│  │ Single-use, stored in httpOnly cookie               │    │
│  └─────────────────────────────────────────────────────┘    │
│                                                              │
└─────────────────────────────────────────────────────────────┘
```

### 7.2 Role-Based Access Control

```
┌─────────────────────────────────────────────────────────────┐
│                    ROLE HIERARCHY                            │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│                    ┌─────────────┐                          │
│                    │ SYSTEM_ADMIN│                          │
│                    └──────┬──────┘                          │
│                           │                                  │
│         ┌─────────────────┼─────────────────┐               │
│         ▼                 ▼                 ▼               │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐         │
│  │ OIDB_STAFF  │  │ DEAN_STAFF  │  │ YGK_MEMBER  │         │
│  └─────────────┘  └─────────────┘  └─────────────┘         │
│         │                                   │               │
│         └───────────────┬───────────────────┘               │
│                         ▼                                    │
│                  ┌─────────────┐                            │
│                  │ YDYO_STAFF  │                            │
│                  └─────────────┘                            │
│                         │                                    │
│                         ▼                                    │
│                  ┌─────────────┐                            │
│                  │   STUDENT   │                            │
│                  └─────────────┘                            │
│                                                              │
└─────────────────────────────────────────────────────────────┘
```

---

## 8. DEPLOYMENT ARCHITECTURE

### 8.1 Production Deployment

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                         PRODUCTION ENVIRONMENT                               │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│                           ┌───────────────┐                                 │
│                           │   Internet    │                                 │
│                           └───────┬───────┘                                 │
│                                   │                                          │
│                           ┌───────▼───────┐                                 │
│                           │   Firewall    │                                 │
│                           └───────┬───────┘                                 │
│                                   │                                          │
│   ┌───────────────────────────────▼───────────────────────────────┐        │
│   │                        Load Balancer                           │        │
│   │                     (Nginx / HAProxy)                          │        │
│   │              SSL Termination, Health Checks                    │        │
│   └───────────────┬───────────────────────────────┬───────────────┘        │
│                   │                               │                         │
│           ┌───────▼───────┐               ┌───────▼───────┐                │
│           │  App Server 1 │               │  App Server 2 │                │
│           │   (Docker)    │               │   (Docker)    │                │
│           │  UTMS:8080    │               │  UTMS:8080    │                │
│           └───────┬───────┘               └───────┬───────┘                │
│                   │                               │                         │
│                   └───────────────┬───────────────┘                         │
│                                   │                                          │
│           ┌───────────────────────▼───────────────────────┐                │
│           │                                               │                │
│   ┌───────▼───────┐                             ┌────────▼────────┐       │
│   │  PostgreSQL   │                             │   File Storage  │       │
│   │   Primary     │──────────────────────────▶  │     (NFS/S3)    │       │
│   │               │       Replication           │                 │       │
│   └───────────────┘                             └─────────────────┘       │
│           │                                                                 │
│   ┌───────▼───────┐                                                        │
│   │  PostgreSQL   │                                                        │
│   │    Replica    │                                                        │
│   │  (Read-only)  │                                                        │
│   └───────────────┘                                                        │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

### 8.2 Docker Compose Configuration

```yaml
version: '3.8'

services:
  app:
    build: .
    ports:
      - "8080:8080"
    environment:
      - SPRING_PROFILES_ACTIVE=prod
      - DB_URL=jdbc:postgresql://db:5432/utms
      - DB_USERNAME=${DB_USERNAME}
      - DB_PASSWORD=${DB_PASSWORD}
      - JWT_SECRET=${JWT_SECRET}
    depends_on:
      - db
    volumes:
      - document-storage:/var/utms/documents
    networks:
      - utms-network

  db:
    image: postgres:15
    environment:
      - POSTGRES_DB=utms
      - POSTGRES_USER=${DB_USERNAME}
      - POSTGRES_PASSWORD=${DB_PASSWORD}
    volumes:
      - postgres-data:/var/lib/postgresql/data
    networks:
      - utms-network

  nginx:
    image: nginx:latest
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - ./nginx.conf:/etc/nginx/nginx.conf
      - ./ssl:/etc/nginx/ssl
    depends_on:
      - app
    networks:
      - utms-network

volumes:
  postgres-data:
  document-storage:

networks:
  utms-network:
    driver: bridge
```

---

## 9. MONITORING & OBSERVABILITY

### 9.1 Health Endpoints

| Endpoint | Purpose |
|----------|---------|
| `/actuator/health` | Overall health status |
| `/actuator/health/db` | Database connectivity |
| `/actuator/health/diskSpace` | Disk space availability |
| `/actuator/metrics` | Application metrics |
| `/actuator/info` | Application info |

### 9.2 Logging Strategy

```
Log Levels:
├── ERROR   # Errors requiring immediate attention
├── WARN    # Warning conditions
├── INFO    # Informational messages (default)
├── DEBUG   # Debug information
└── TRACE   # Most detailed logging

Log Format:
timestamp | level | thread | logger | message | context
```

---

## 10. REVISION HISTORY

| Version | Date | Author | Changes |
|---------|------|--------|---------|
| 1.0 | Mar 2026 | Group 05 | Initial version |
