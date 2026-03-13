# UTMS - Undergraduate Transfer Management System
## 08 - Frontend Architecture

**Version:** 1.0  
**Last Updated:** March 2026

---

## 1. TECHNOLOGY STACK

### 1.1 Core Technologies

| Technology | Version | Purpose |
|------------|---------|---------|
| React | 18.x | UI framework |
| TypeScript | 5.x | Type-safe JavaScript |
| Vite | 5.x | Build tool & dev server |
| React Router | 6.x | Client-side routing |

### 1.2 State Management

| Technology | Purpose |
|------------|---------|
| Zustand | Global state (auth, notifications) |
| React Query | Server state, caching, sync |
| React Hook Form | Form state management |

### 1.3 UI Framework

| Technology | Purpose |
|------------|---------|
| Ant Design | Component library |
| Tailwind CSS | Utility styling |
| Framer Motion | Animations |

### 1.4 Supporting Libraries

| Library | Purpose |
|---------|---------|
| Axios | HTTP client |
| Zod | Schema validation |
| dayjs | Date handling |
| i18next | Internationalization |

---

## 2. PROJECT STRUCTURE

```
src/
├── main.tsx                    # Application entry point
├── App.tsx                     # Root component with providers
├── vite-env.d.ts              # Vite type definitions
│
├── api/                        # API layer
│   ├── client.ts              # Axios instance configuration
│   ├── auth.api.ts            # Authentication endpoints
│   ├── application.api.ts     # Application endpoints
│   ├── document.api.ts        # Document endpoints
│   ├── evaluation.api.ts      # Evaluation endpoints
│   ├── notification.api.ts    # Notification endpoints
│   ├── user.api.ts            # User management endpoints
│   └── admin.api.ts           # Admin endpoints
│
├── components/                 # Reusable components
│   ├── common/                # Generic components
│   ├── layout/                # Layout components
│   ├── auth/                  # Authentication components
│   ├── application/           # Application-related components
│   ├── document/              # Document components
│   ├── workflow/              # Workflow components
│   ├── intibak/               # Intibak components
│   └── notification/          # Notification components
│
├── pages/                      # Page components
│   ├── auth/                  # Authentication pages
│   ├── student/               # Student portal pages
│   ├── staff/                 # Staff portal pages
│   ├── admin/                 # Admin pages
│   └── error/                 # Error pages
│
├── hooks/                      # Custom React hooks
├── store/                      # Zustand stores
├── types/                      # TypeScript type definitions
├── utils/                      # Utility functions
├── routes/                     # Routing configuration
├── constants/                  # Application constants
├── services/                   # Business logic services
└── styles/                     # Global styles
```

---

## 3. ROUTING STRUCTURE

### 3.1 Public Routes

| Path | Page | Description |
|------|------|-------------|
| `/` | LandingPage | Public landing page |
| `/login` | LoginPage | User login |
| `/register` | RegisterPage | Student registration |
| `/forgot-password` | ForgotPasswordPage | Password reset request |
| `/reset-password` | ResetPasswordPage | Password reset form |

### 3.2 Student Routes (Protected)

| Path | Page | Description |
|------|------|-------------|
| `/student` | StudentDashboard | Student home |
| `/student/applications` | ApplicationListPage | My applications |
| `/student/applications/new` | NewApplicationPage | Create application |
| `/student/applications/:id` | ApplicationDetailPage | View application |
| `/student/applications/:id/documents` | DocumentsPage | Manage documents |
| `/student/applications/:id/status` | StatusPage | Track status |
| `/student/profile` | ProfilePage | User profile |
| `/student/notifications` | NotificationsPage | All notifications |

### 3.3 Staff Routes (Protected)

| Path | Page | Description |
|------|------|-------------|
| `/staff` | StaffDashboard | Staff home |
| `/staff/applications` | ApplicationListPage | Applications queue |
| `/staff/applications/:id` | ApplicationDetailPage | Application review |
| `/staff/applications/:id/documents` | DocumentReviewPage | Review documents |
| `/staff/applications/:id/intibak` | IntibakPage | Intibak management |
| `/staff/rankings` | RankingsPage | View/calculate rankings |

### 3.4 Admin Routes (Protected)

| Path | Page | Description |
|------|------|-------------|
| `/admin` | AdminDashboard | Admin home |
| `/admin/users` | UsersPage | User management |
| `/admin/departments` | DepartmentsPage | Department management |
| `/admin/audit-logs` | AuditLogsPage | System logs |
| `/admin/reports` | ReportsPage | Generate reports |
| `/admin/settings` | SettingsPage | System settings |

### 3.5 Route Protection Logic

```
Route Access Rules:
├── Public routes: No authentication required
├── Student routes: role === 'STUDENT'
├── Staff routes: role IN ['OIDB_STAFF', 'DEAN_STAFF', 'YGK_MEMBER', 'YDYO_STAFF']
├── Admin routes: role === 'SYSTEM_ADMIN'
└── 403 Forbidden: If role doesn't match
```

---

## 4. COMPONENT ARCHITECTURE

### 4.1 Component Categories

**Common Components** - Reusable across the application
```
common/
├── Button/              # Customized button variants
├── Input/               # Form inputs with validation
├── Modal/               # Modal dialogs
├── Table/               # Data tables with sorting/filtering
├── Card/                # Content cards
├── Loading/             # Loading spinners/skeletons
├── ErrorBoundary/       # Error handling wrapper
├── Pagination/          # Pagination controls
├── Badge/               # Status badges
├── Alert/               # Alert messages
├── Tooltip/             # Tooltips
├── Dropdown/            # Dropdown menus
├── FileUpload/          # File upload with preview
├── DatePicker/          # Date selection
└── ConfirmDialog/       # Confirmation dialogs
```

**Layout Components** - Page structure
```
layout/
├── Header/              # Top navigation bar
├── Sidebar/             # Side navigation menu
├── Footer/              # Page footer
├── PageContainer/       # Page wrapper with breadcrumbs
├── StudentLayout/       # Layout for student pages
├── StaffLayout/         # Layout for staff pages
├── AdminLayout/         # Layout for admin pages
└── AuthLayout/          # Layout for auth pages
```

**Feature Components** - Domain-specific
```
application/
├── ApplicationForm/     # Application creation form
├── ApplicationCard/     # Application summary card
├── ApplicationList/     # List of applications
├── ApplicationDetail/   # Full application view
├── ApplicationSummary/  # Brief application info
└── StatusBadge/         # Application status display

document/
├── DocumentUploader/    # Drag-drop file upload
├── DocumentList/        # List of documents
├── DocumentViewer/      # PDF/image viewer
├── DocumentReview/      # Review interface
└── DocumentStatusIcon/  # Document status indicator

workflow/
├── WorkflowTimeline/    # Visual timeline
├── StageIndicator/      # Current stage display
├── StageProgress/       # Progress bar
├── TransferButton/      # Stage transfer action
└── WorkflowHistory/     # History log

intibak/
├── IntibakForm/         # Course equivalence form
├── IntibakTable/        # Course equivalence table
├── CourseComparison/    # Side-by-side comparison
├── IntibakSummary/      # Summary statistics
└── CourseSearch/        # Course search

notification/
├── NotificationBell/    # Header notification icon
├── NotificationList/    # Notification list
├── NotificationItem/    # Single notification
└── NotificationBadge/   # Unread count badge
```

### 4.2 Component Design Patterns

**Container/Presenter Pattern:**
```
ApplicationDetailPage (Container)
├── Fetches data using React Query
├── Manages local state
├── Handles user interactions
└── Renders ApplicationDetail (Presenter)
    └── Pure UI component, receives props
```

**Compound Components:**
```
<ApplicationForm>
  <ApplicationForm.BasicInfo />
  <ApplicationForm.AcademicInfo />
  <ApplicationForm.LanguageInfo />
  <ApplicationForm.Actions />
</ApplicationForm>
```

**Render Props / Hooks:**
```
const { applications, isLoading, error } = useApplications();
```

---

## 5. STATE MANAGEMENT

### 5.1 Auth Store (Zustand)

```typescript
interface AuthState {
  user: User | null;
  accessToken: string | null;
  isAuthenticated: boolean;
  isLoading: boolean;
  
  // Actions
  login: (credentials: LoginRequest) => Promise<void>;
  logout: () => void;
  refreshToken: () => Promise<void>;
  updateUser: (user: User) => void;
}
```

**State Persistence:**
- Access token: Memory only (security)
- Refresh token: HttpOnly cookie (handled by server)
- User info: LocalStorage (optional, for UX)

### 5.2 Notification Store (Zustand)

```typescript
interface NotificationState {
  notifications: Notification[];
  unreadCount: number;
  
  // Actions
  fetchNotifications: () => Promise<void>;
  markAsRead: (id: number) => Promise<void>;
  markAllAsRead: () => Promise<void>;
  addNotification: (notification: Notification) => void;
}
```

### 5.3 UI Store (Zustand)

```typescript
interface UIState {
  sidebarCollapsed: boolean;
  theme: 'light' | 'dark';
  language: 'tr' | 'en';
  
  // Actions
  toggleSidebar: () => void;
  setTheme: (theme: string) => void;
  setLanguage: (lang: string) => void;
}
```

### 5.4 Server State (React Query)

```typescript
// Queries
useApplications(filters)      // List applications
useApplication(id)            // Single application
useDocuments(applicationId)   // Application documents
useNotifications()            // User notifications
useUser()                     // Current user

// Mutations
useCreateApplication()
useUpdateApplication()
useSubmitApplication()
useUploadDocument()
useReviewDocument()
useTransferApplication()
```

**Caching Strategy:**
| Query | Stale Time | Cache Time |
|-------|------------|------------|
| Applications List | 30 seconds | 5 minutes |
| Single Application | 1 minute | 10 minutes |
| User Profile | 5 minutes | 30 minutes |
| Departments | 1 hour | 24 hours |
| Notifications | 10 seconds | 1 minute |

---

## 6. PAGE SPECIFICATIONS

### 6.1 Student Dashboard

**Purpose:** Overview of student's transfer application status

**Layout:**
```
┌─────────────────────────────────────────────────────┐
│ Header (Logo, User Menu, Notifications)             │
├──────────┬──────────────────────────────────────────┤
│          │                                          │
│ Sidebar  │  Welcome Message                         │
│          │  ─────────────────────────────────       │
│ - Home   │                                          │
│ - Apps   │  ┌─────────────────────────────────┐    │
│ - Docs   │  │ Current Application Status      │    │
│ - Profile│  │ [Progress Bar: 65%]             │    │
│          │  │ Stage: Academic Evaluation      │    │
│          │  │ Last Update: 2 days ago         │    │
│          │  └─────────────────────────────────┘    │
│          │                                          │
│          │  ┌─────────┐ ┌─────────┐ ┌─────────┐   │
│          │  │Pending  │ │Documents│ │Notifica-│   │
│          │  │Actions:2│ │Status   │ │tions: 3 │   │
│          │  └─────────┘ └─────────┘ └─────────┘   │
│          │                                          │
│          │  Recent Activity                         │
│          │  ─────────────────────────────────       │
│          │  • Document approved - 1 day ago         │
│          │  • Status changed - 3 days ago           │
│          │                                          │
└──────────┴──────────────────────────────────────────┘
```

**Components Used:**
- PageContainer
- StatsCard (x3)
- ApplicationStatusCard
- ActivityTimeline
- NotificationPreview

**Data Required:**
- Current application (if exists)
- Application status
- Recent notifications
- Pending actions count

---

### 6.2 New Application Page

**Purpose:** Create a new transfer application

**Layout:**
```
┌─────────────────────────────────────────────────────┐
│ Header                                              │
├──────────┬──────────────────────────────────────────┤
│          │  New Application                         │
│ Sidebar  │  ─────────────────────────────────       │
│          │                                          │
│          │  Step 1 of 4: Department Selection       │
│          │  ═══════════════════════════════════     │
│          │                                          │
│          │  ┌─────────────────────────────────┐    │
│          │  │ Select Faculty:                  │    │
│          │  │ [Engineering ▼]                  │    │
│          │  │                                  │    │
│          │  │ Select Department:               │    │
│          │  │ [Computer Engineering ▼]         │    │
│          │  │                                  │    │
│          │  │ Quota: 5 | Min GPA: 2.50         │    │
│          │  └─────────────────────────────────┘    │
│          │                                          │
│          │  ┌─────────────────────────────────┐    │
│          │  │ [Back]              [Next Step] │    │
│          │  └─────────────────────────────────┘    │
│          │                                          │
└──────────┴──────────────────────────────────────────┘
```

**Steps:**
1. Department Selection
2. Source Institution Information
3. Academic Scores
4. Language Information (if required)

**Form Validation:**
- Real-time validation using React Hook Form + Zod
- Step validation before proceeding
- Final validation before save

**User Experience:**
- Progress indicator
- Save as draft at any step
- Confirmation before leaving with unsaved changes
- Auto-save draft every 30 seconds

---

### 6.3 Application Status Page

**Purpose:** Track application progress through workflow

**Layout:**
```
┌─────────────────────────────────────────────────────┐
│ Header                                              │
├──────────┬──────────────────────────────────────────┤
│          │  Application Status                      │
│ Sidebar  │  #2026-FALL-CENG-0001                    │
│          │  ─────────────────────────────────       │
│          │                                          │
│          │  Current Status: PENDING_INTIBAK        │
│          │  Progress: ████████████░░░░ 70%         │
│          │                                          │
│          │  Timeline                                │
│          │  ─────────────────────────────────       │
│          │                                          │
│          │  ✓ OIDB Initial Review                   │
│          │    Completed: Mar 15, 2026              │
│          │    By: Mehmet D.                         │
│          │                                          │
│          │  ✓ Language Evaluation                   │
│          │    Completed: Mar 17, 2026              │
│          │    Result: Approved                      │
│          │                                          │
│          │  ● Intibak Preparation (Current)        │
│          │    Started: Mar 18, 2026                │
│          │    Estimated: 5-7 days                  │
│          │                                          │
│          │  ○ Faculty Decision                      │
│          │    Pending                               │
│          │                                          │
│          │  ○ Result Publication                    │
│          │    Pending                               │
│          │                                          │
└──────────┴──────────────────────────────────────────┘
```

**Components Used:**
- StatusHeader
- ProgressBar
- WorkflowTimeline
- StageCard

**Legend:**
- ✓ = Completed
- ● = In Progress
- ○ = Pending

---

### 6.4 Document Review Page (Staff)

**Purpose:** Review and approve/reject documents

**Layout:**
```
┌─────────────────────────────────────────────────────┐
│ Header                                              │
├──────────┬──────────────────────────────────────────┤
│          │  Document Review                         │
│ Sidebar  │  Application: #2026-FALL-CENG-0001      │
│          │  ─────────────────────────────────       │
│          │                                          │
│          │  ┌────────────┬──────────────────────┐  │
│          │  │ Documents  │ Document Viewer       │  │
│          │  │            │                       │  │
│          │  │ ✓ Transcript│ ┌─────────────────┐ │  │
│          │  │ ⏳ YKS Result│ │                 │ │  │
│          │  │ ✓ ID Card   │ │   [PDF Viewer]  │ │  │
│          │  │ ⏳ Language │ │                 │ │  │
│          │  │ ⏳ Courses  │ │                 │ │  │
│          │  │            │ └─────────────────┘ │  │
│          │  │            │                       │  │
│          │  │            │ Review Actions:       │  │
│          │  │            │ [Approve] [Reject]   │  │
│          │  │            │ [Request Resubmit]   │  │
│          │  │            │                       │  │
│          │  │            │ Comments:             │  │
│          │  │            │ [________________]    │  │
│          │  │            │                       │  │
│          │  └────────────┴──────────────────────┘  │
│          │                                          │
└──────────┴──────────────────────────────────────────┘
```

**Features:**
- Split view: Document list + Viewer
- PDF/Image preview
- Quick actions
- Comment input
- Batch approve option

---

### 6.5 Intibak Page (YGK)

**Purpose:** Prepare course equivalence records

**Layout:**
```
┌─────────────────────────────────────────────────────┐
│ Header                                              │
├──────────┬──────────────────────────────────────────┤
│          │  Intibak Preparation                     │
│ Sidebar  │  Student: Ahmet Yılmaz                   │
│          │  ─────────────────────────────────       │
│          │                                          │
│          │  Summary: 38/45 credits accepted (84%)   │
│          │                                          │
│          │  ┌───────────────────────────────────┐  │
│          │  │ Source Course │ Target │ Decision │  │
│          │  ├───────────────┼────────┼──────────┤  │
│          │  │ BIL101 Prog I │ CENG111│ ✓ Accept │  │
│          │  │ BIL102 Prog II│ CENG112│ ✓ Accept │  │
│          │  │ MAT101 Calc I │ MATH101│ ✓ Accept │  │
│          │  │ FIZ101 Physics│ PHYS101│ ⚠ Cond.  │  │
│          │  │ ENG101 English│ ---    │ ✗ Reject │  │
│          │  │ ...           │        │          │  │
│          │  └───────────────────────────────────┘  │
│          │                                          │
│          │  [Add New] [Complete Intibak]            │
│          │                                          │
└──────────┴──────────────────────────────────────────┘
```

**Features:**
- Course comparison
- Inline editing
- Decision dropdown
- Bulk actions
- Summary statistics
- Complete button (when all decided)

---

## 7. UI/UX GUIDELINES

### 7.1 Color Scheme

| Purpose | Color | Hex Code | Usage |
|---------|-------|----------|-------|
| Primary | Blue | #1890ff | Actions, links |
| Success | Green | #52c41a | Approved, success |
| Warning | Orange | #faad14 | Pending, caution |
| Error | Red | #f5222d | Rejected, error |
| Info | Cyan | #13c2c2 | Information |
| Text Primary | Dark | #262626 | Main text |
| Text Secondary | Gray | #8c8c8c | Secondary text |
| Background | Light | #f0f2f5 | Page background |

### 7.2 Status Colors

| Status | Color | Badge Style |
|--------|-------|-------------|
| DRAFT | Gray | default |
| SUBMITTED | Blue | processing |
| DOCUMENTS_UNDER_REVIEW | Orange | warning |
| DOCUMENTS_APPROVED | Green | success |
| DOCUMENTS_REJECTED | Red | error |
| PENDING_LANGUAGE_EVALUATION | Orange | warning |
| LANGUAGE_APPROVED | Green | success |
| LANGUAGE_REJECTED | Red | error |
| PENDING_INTIBAK | Orange | warning |
| INTIBAK_COMPLETED | Blue | processing |
| PENDING_FACULTY_DECISION | Orange | warning |
| ASIL | Green | success |
| YEDEK | Cyan | default |
| REJECTED | Red | error |
| WITHDRAWN | Gray | default |

### 7.3 Typography

| Element | Font | Size | Weight |
|---------|------|------|--------|
| H1 | Inter | 30px | 600 |
| H2 | Inter | 24px | 600 |
| H3 | Inter | 20px | 500 |
| H4 | Inter | 16px | 500 |
| Body | Inter | 14px | 400 |
| Small | Inter | 12px | 400 |
| Label | Inter | 14px | 500 |

### 7.4 Spacing

| Size | Value | Usage |
|------|-------|-------|
| xs | 4px | Tight spacing |
| sm | 8px | Small gaps |
| md | 16px | Default spacing |
| lg | 24px | Section spacing |
| xl | 32px | Large sections |
| xxl | 48px | Page margins |

### 7.5 Responsive Breakpoints

| Breakpoint | Min Width | Target |
|------------|-----------|--------|
| xs | 0 | Mobile |
| sm | 576px | Tablet portrait |
| md | 768px | Tablet landscape |
| lg | 992px | Desktop |
| xl | 1200px | Large desktop |
| xxl | 1600px | Extra large |

### 7.6 Loading States

| Scenario | Component | Behavior |
|----------|-----------|----------|
| Page load | Skeleton | Show skeleton of content |
| Button action | Spinner | Show spinner inside button |
| Table loading | Table skeleton | Row placeholders |
| Form submit | Button disabled | Disable with spinner |
| Background refresh | None | Silent refresh |

### 7.7 Error Handling

| Error Type | Display | Action |
|------------|---------|--------|
| Form validation | Inline | Show below field |
| API error | Toast | Show notification |
| Network error | Modal | Retry option |
| 404 | Full page | Go back option |
| 403 | Full page | Go to home |
| 500 | Full page | Contact support |

---

## 8. ACCESSIBILITY

### 8.1 Requirements

- WCAG 2.1 Level AA compliance
- Keyboard navigation support
- Screen reader compatibility
- Color contrast ratios ≥ 4.5:1
- Focus indicators
- Alt text for images
- ARIA labels where needed

### 8.2 Implementation

- Use semantic HTML
- Provide skip links
- Ensure form labels
- Test with screen readers
- Support reduced motion

---

## 9. INTERNATIONALIZATION

### 9.1 Supported Languages

| Language | Code | Status |
|----------|------|--------|
| Turkish | tr | Primary |
| English | en | Secondary |

### 9.2 Implementation

- i18next for translations
- Separate JSON files per language
- Date/number formatting per locale
- RTL support (future)

---

## 10. REVISION HISTORY

| Version | Date | Author | Changes |
|---------|------|--------|---------|
| 1.0 | Mar 2026 | Group 05 | Initial version |
