# UTMS - Undergraduate Transfer Management System
## 09 - Frontend Types & Interfaces

**Version:** 1.0  
**Last Updated:** March 2026

---

## 1. ENUMERATIONS

### 1.1 User Roles

```typescript
enum UserRole {
  STUDENT = 'STUDENT',
  OIDB_STAFF = 'OIDB_STAFF',
  DEAN_STAFF = 'DEAN_STAFF',
  YGK_MEMBER = 'YGK_MEMBER',
  YDYO_STAFF = 'YDYO_STAFF',
  SYSTEM_ADMIN = 'SYSTEM_ADMIN'
}
```

### 1.2 Application Status

```typescript
enum ApplicationStatus {
  DRAFT = 'DRAFT',
  SUBMITTED = 'SUBMITTED',
  DOCUMENTS_UNDER_REVIEW = 'DOCUMENTS_UNDER_REVIEW',
  DOCUMENTS_APPROVED = 'DOCUMENTS_APPROVED',
  DOCUMENTS_REJECTED = 'DOCUMENTS_REJECTED',
  PENDING_LANGUAGE_EVALUATION = 'PENDING_LANGUAGE_EVALUATION',
  LANGUAGE_APPROVED = 'LANGUAGE_APPROVED',
  LANGUAGE_REJECTED = 'LANGUAGE_REJECTED',
  PENDING_ACADEMIC_EVALUATION = 'PENDING_ACADEMIC_EVALUATION',
  PENDING_INTIBAK = 'PENDING_INTIBAK',
  INTIBAK_COMPLETED = 'INTIBAK_COMPLETED',
  PENDING_FACULTY_DECISION = 'PENDING_FACULTY_DECISION',
  ASIL = 'ASIL',
  YEDEK = 'YEDEK',
  REJECTED = 'REJECTED',
  WITHDRAWN = 'WITHDRAWN'
}
```

### 1.3 Workflow Stage

```typescript
enum WorkflowStage {
  OIDB_INITIAL = 'OIDB_INITIAL',
  YDYO_EVALUATION = 'YDYO_EVALUATION',
  OIDB_POST_LANGUAGE = 'OIDB_POST_LANGUAGE',
  DEANS_OFFICE_REVIEW = 'DEANS_OFFICE_REVIEW',
  YGK_EVALUATION = 'YGK_EVALUATION',
  DEANS_OFFICE_FINAL = 'DEANS_OFFICE_FINAL',
  OIDB_FINAL = 'OIDB_FINAL',
  COMPLETED = 'COMPLETED'
}
```

### 1.4 Document Types

```typescript
enum DocumentType {
  TRANSCRIPT = 'TRANSCRIPT',
  YKS_RESULT = 'YKS_RESULT',
  LANGUAGE_CERTIFICATE = 'LANGUAGE_CERTIFICATE',
  COURSE_CONTENT = 'COURSE_CONTENT',
  ID_CARD = 'ID_CARD',
  PORTFOLIO = 'PORTFOLIO',
  DISCIPLINARY_CERTIFICATE = 'DISCIPLINARY_CERTIFICATE',
  STUDENT_CERTIFICATE = 'STUDENT_CERTIFICATE',
  OTHER = 'OTHER'
}
```

### 1.5 Document Status

```typescript
enum DocumentStatus {
  PENDING = 'PENDING',
  APPROVED = 'APPROVED',
  REJECTED = 'REJECTED',
  REQUIRES_RESUBMISSION = 'REQUIRES_RESUBMISSION'
}
```

### 1.6 Notification Types

```typescript
enum NotificationType {
  APPLICATION_SUBMITTED = 'APPLICATION_SUBMITTED',
  DOCUMENT_REQUESTED = 'DOCUMENT_REQUESTED',
  DOCUMENT_APPROVED = 'DOCUMENT_APPROVED',
  DOCUMENT_REJECTED = 'DOCUMENT_REJECTED',
  STATUS_CHANGED = 'STATUS_CHANGED',
  STAGE_CHANGED = 'STAGE_CHANGED',
  LANGUAGE_RESULT = 'LANGUAGE_RESULT',
  RANKING_RESULT = 'RANKING_RESULT',
  FINAL_DECISION = 'FINAL_DECISION',
  SYSTEM_ANNOUNCEMENT = 'SYSTEM_ANNOUNCEMENT',
  REMINDER = 'REMINDER'
}
```

### 1.7 Intibak Decision

```typescript
enum IntibakDecision {
  PENDING = 'PENDING',
  ACCEPTED = 'ACCEPTED',
  REJECTED = 'REJECTED',
  CONDITIONAL = 'CONDITIONAL'
}
```

---

## 2. DOMAIN TYPES

### 2.1 User Types

```typescript
interface User {
  id: number;
  email: string;
  firstName: string;
  lastName: string;
  phone?: string;
  role: UserRole;
  isActive: boolean;
  kvkkConsent: boolean;
  kvkkConsentDate?: string;
  lastLoginAt?: string;
  createdAt: string;
  updatedAt: string;
}

interface UserProfile extends User {
  applicationsCount?: number;
}

// For display in lists
interface UserSummary {
  id: number;
  email: string;
  firstName: string;
  lastName: string;
  role: UserRole;
  isActive: boolean;
}
```

### 2.2 Application Types

```typescript
interface Application {
  id: number;
  applicationNumber: string;
  applicationYear: number;
  applicationTerm: 'FALL' | 'SPRING';
  status: ApplicationStatus;
  currentStage: WorkflowStage;
  
  // Relations
  applicant: ApplicantInfo;
  department: DepartmentInfo;
  assignedTo?: AssigneeInfo;
  
  // Academic Info
  academicInfo: AcademicInfo;
  languageInfo: LanguageInfo;
  ranking: RankingInfo;
  
  // Collections
  documents: Document[];
  workflowHistory: WorkflowHistoryEntry[];
  intibakRecords?: IntibakRecord[];
  
  // Timestamps
  submittedAt?: string;
  evaluatedAt?: string;
  decidedAt?: string;
  createdAt: string;
  updatedAt: string;
}

interface ApplicantInfo {
  id: number;
  firstName: string;
  lastName: string;
  email: string;
  phone?: string;
}

interface AcademicInfo {
  sourceUniversity: string;
  sourceFaculty?: string;
  sourceDepartment: string;
  currentSemester: number;
  yksScore: number;
  yksRanking: number;
  gpa: number;
  transferGrade?: number;
}

interface LanguageInfo {
  certificateType?: string;
  score?: number;
  approved?: boolean;
  evaluatedAt?: string;
}

interface RankingInfo {
  isAsil?: boolean;
  position?: number;
}

interface AssigneeInfo {
  id: number;
  firstName: string;
  lastName: string;
}

// For list views
interface ApplicationListItem {
  id: number;
  applicationNumber: string;
  status: ApplicationStatus;
  currentStage: WorkflowStage;
  applicant: ApplicantInfo;
  department: DepartmentInfo;
  sourceUniversity: string;
  gpa: number;
  transferGrade?: number;
  submittedAt?: string;
  documentsCount: number;
  pendingDocumentsCount: number;
}
```

### 2.3 Department Types

```typescript
interface Faculty {
  id: number;
  name: string;
  code: string;
  isActive: boolean;
}

interface Department {
  id: number;
  name: string;
  code: string;
  faculty: Faculty;
  quota: number;
  minGpa: number;
  maxRanking?: number;
  minYksScore?: number;
  languageRequired: boolean;
  minLanguageScore: number;
  specialRequirements?: string;
  isActive: boolean;
}

interface DepartmentInfo {
  id: number;
  name: string;
  code: string;
  quota?: number;
  faculty?: {
    id: number;
    name: string;
  };
}
```

### 2.4 Document Types

```typescript
interface Document {
  id: number;
  applicationId: number;
  documentType: DocumentType;
  fileName: string;
  originalFileName: string;
  fileSize: number;
  mimeType: string;
  status: DocumentStatus;
  reviewComments?: string;
  uploadedAt: string;
  reviewedAt?: string;
  reviewedBy?: ReviewerInfo;
}

interface ReviewerInfo {
  id: number;
  firstName: string;
  lastName: string;
}
```

### 2.5 Workflow Types

```typescript
interface WorkflowHistoryEntry {
  id: number;
  fromStage?: WorkflowStage;
  toStage: WorkflowStage;
  action: string;
  performedBy?: {
    id: number;
    firstName: string;
    lastName: string;
    role: UserRole;
  };
  notes?: string;
  createdAt: string;
}

interface StageInfo {
  stage: WorkflowStage;
  status: 'COMPLETED' | 'IN_PROGRESS' | 'PENDING';
  enteredAt?: string;
  completedAt?: string;
  description: string;
}

interface ApplicationStatusInfo {
  applicationId: number;
  applicationNumber: string;
  currentStatus: ApplicationStatus;
  currentStage: WorkflowStage;
  progress: number;
  timeline: StageInfo[];
  estimatedCompletionDays: number;
  lastUpdated: string;
}
```

### 2.6 Intibak Types

```typescript
interface IntibakRecord {
  id: number;
  applicationId: number;
  sourceCourse: SourceCourseInfo;
  targetCourse?: TargetCourseInfo;
  decision: IntibakDecision;
  notes?: string;
}

interface SourceCourseInfo {
  code: string;
  name: string;
  credits: number;
  ects?: number;
  grade: string;
  gradePoint?: number;
}

interface TargetCourseInfo {
  code: string;
  name: string;
  credits: number;
  ects?: number;
}

interface IntibakSummary {
  applicationId: number;
  totalSourceCredits: number;
  totalAcceptedCredits: number;
  records: IntibakRecord[];
  preparedBy?: {
    id: number;
    firstName: string;
    lastName: string;
  };
  preparedAt?: string;
}
```

### 2.7 Notification Types

```typescript
interface Notification {
  id: number;
  type: NotificationType;
  title: string;
  message: string;
  priority: 'LOW' | 'NORMAL' | 'HIGH' | 'URGENT';
  isRead: boolean;
  readAt?: string;
  actionUrl?: string;
  applicationId?: number;
  createdAt: string;
}
```

### 2.8 Ranking Types

```typescript
interface RankingEntry {
  position: number;
  applicationId: number;
  applicantName: string;
  transferGrade: number;
  isAsil: boolean;
}

interface DepartmentRankings {
  departmentId: number;
  departmentName: string;
  quota: number;
  asil: RankingEntry[];
  yedek: RankingEntry[];
  lastCalculatedAt?: string;
}
```

---

## 3. API REQUEST TYPES

### 3.1 Authentication Requests

```typescript
interface LoginRequest {
  email: string;
  password: string;
}

interface RegisterRequest {
  email: string;
  password: string;
  confirmPassword: string;
  firstName: string;
  lastName: string;
  phone?: string;
  kvkkConsent: boolean;
}

interface ForgotPasswordRequest {
  email: string;
}

interface ResetPasswordRequest {
  token: string;
  password: string;
  confirmPassword: string;
}

interface ChangePasswordRequest {
  currentPassword: string;
  newPassword: string;
  confirmPassword: string;
}
```

### 3.2 Application Requests

```typescript
interface CreateApplicationRequest {
  departmentId: number;
  applicationTerm: 'FALL' | 'SPRING';
  sourceUniversity: string;
  sourceFaculty?: string;
  sourceDepartment: string;
  currentSemester: number;
  yksScore: number;
  yksRanking: number;
  gpa: number;
  languageCertType?: string;
  languageScore?: number;
}

interface UpdateApplicationRequest {
  sourceUniversity?: string;
  sourceFaculty?: string;
  sourceDepartment?: string;
  currentSemester?: number;
  yksScore?: number;
  yksRanking?: number;
  gpa?: number;
  languageCertType?: string;
  languageScore?: number;
}

interface WithdrawRequest {
  reason?: string;
}
```

### 3.3 Workflow Requests

```typescript
interface TransferRequest {
  action: 'APPROVE_AND_FORWARD' | 'REJECT' | 'REQUEST_DOCUMENTS' | 'RETURN_TO_PREVIOUS';
  toStage?: WorkflowStage;
  notes?: string;
}

interface LanguageEvaluationRequest {
  approved: boolean;
  notes?: string;
}
```

### 3.4 Document Requests

```typescript
interface UploadDocumentRequest {
  file: File;
  documentType: DocumentType;
}

interface ReviewDocumentRequest {
  status: DocumentStatus;
  comments?: string;
}
```

### 3.5 Intibak Requests

```typescript
interface CreateIntibakRequest {
  sourceCourseCode: string;
  sourceCourseName: string;
  sourceCredits: number;
  sourceEcts?: number;
  sourceGrade: string;
  sourceGradePoint?: number;
  targetCourseCode?: string;
  targetCourseName?: string;
  targetCredits?: number;
  targetEcts?: number;
  decision: IntibakDecision;
  notes?: string;
}

interface UpdateIntibakRequest {
  targetCourseCode?: string;
  targetCourseName?: string;
  targetCredits?: number;
  targetEcts?: number;
  decision?: IntibakDecision;
  notes?: string;
}
```

---

## 4. API RESPONSE TYPES

### 4.1 Common Response Types

```typescript
interface ApiResponse<T> {
  success: boolean;
  data: T;
  message?: string;
  timestamp: string;
}

interface ApiError {
  success: false;
  error: {
    code: string;
    message: string;
    details?: FieldError[];
  };
  timestamp: string;
}

interface FieldError {
  field: string;
  message: string;
}

interface PageResponse<T> {
  content: T[];
  page: number;
  size: number;
  totalElements: number;
  totalPages: number;
  first: boolean;
  last: boolean;
}
```

### 4.2 Auth Responses

```typescript
interface LoginResponse {
  accessToken: string;
  refreshToken: string;
  tokenType: string;
  expiresIn: number;
  user: {
    id: number;
    email: string;
    firstName: string;
    lastName: string;
    role: UserRole;
    lastLoginAt?: string;
  };
}

interface RefreshResponse {
  accessToken: string;
  expiresIn: number;
}
```

### 4.3 Evaluation Responses

```typescript
interface TransferGradeResponse {
  applicationId: number;
  yksScore: number;
  minYksScore: number;
  gpa: number;
  transferGrade: number;
  formula: string;
  calculatedAt: string;
}

interface RankingResponse {
  departmentId: number;
  departmentName: string;
  applicationYear: number;
  applicationTerm: string;
  quota: number;
  totalApplications: number;
  eligibleApplications: number;
  rankings: RankingEntry[];
  calculatedAt: string;
}
```

---

## 5. FORM TYPES

### 5.1 Form State Types

```typescript
// Login Form
interface LoginFormValues {
  email: string;
  password: string;
  remember?: boolean;
}

// Registration Form
interface RegisterFormValues {
  email: string;
  password: string;
  confirmPassword: string;
  firstName: string;
  lastName: string;
  phone?: string;
  kvkkConsent: boolean;
}

// Application Form
interface ApplicationFormValues {
  departmentId: number;
  applicationTerm: 'FALL' | 'SPRING';
  sourceUniversity: string;
  sourceFaculty?: string;
  sourceDepartment: string;
  currentSemester: number;
  yksScore: number;
  yksRanking: number;
  gpa: number;
  languageCertType?: string;
  languageScore?: number;
}

// Document Upload Form
interface DocumentUploadFormValues {
  documentType: DocumentType;
  file: File | null;
}

// Document Review Form
interface DocumentReviewFormValues {
  status: DocumentStatus;
  comments: string;
}

// Intibak Form
interface IntibakFormValues {
  sourceCourseCode: string;
  sourceCourseName: string;
  sourceCredits: number;
  sourceGrade: string;
  targetCourseCode?: string;
  targetCourseName?: string;
  targetCredits?: number;
  decision: IntibakDecision;
  notes?: string;
}
```

---

## 6. UTILITY TYPES

### 6.1 Filter Types

```typescript
interface ApplicationFilters {
  status?: ApplicationStatus;
  stage?: WorkflowStage;
  departmentId?: number;
  year?: number;
  term?: 'FALL' | 'SPRING';
  search?: string;
  assignedToMe?: boolean;
}

interface UserFilters {
  role?: UserRole;
  isActive?: boolean;
  search?: string;
}

interface AuditLogFilters {
  userId?: number;
  action?: string;
  entityType?: string;
  startDate?: string;
  endDate?: string;
}
```

### 6.2 Sort Types

```typescript
interface SortConfig {
  field: string;
  direction: 'asc' | 'desc';
}

type SortDirection = 'ascend' | 'descend' | null;
```

### 6.3 Pagination Types

```typescript
interface PaginationConfig {
  page: number;
  size: number;
  sort?: SortConfig;
}

interface PaginatedRequest {
  page?: number;
  size?: number;
  sort?: string;
}
```

---

## 7. COMPONENT PROP TYPES

### 7.1 Common Props

```typescript
interface BaseProps {
  className?: string;
  style?: React.CSSProperties;
}

interface LoadingProps {
  loading?: boolean;
}

interface ErrorProps {
  error?: string | null;
}
```

### 7.2 Table Props

```typescript
interface TableProps<T> extends BaseProps, LoadingProps {
  data: T[];
  columns: ColumnDef<T>[];
  pagination?: PaginationConfig;
  onPaginationChange?: (config: PaginationConfig) => void;
  onRowClick?: (record: T) => void;
  rowKey: keyof T | ((record: T) => string);
}
```

### 7.3 Form Props

```typescript
interface FormProps<T> extends BaseProps {
  initialValues?: Partial<T>;
  onSubmit: (values: T) => Promise<void>;
  onCancel?: () => void;
  submitText?: string;
  cancelText?: string;
  loading?: boolean;
}
```

### 7.4 Modal Props

```typescript
interface ModalProps extends BaseProps {
  open: boolean;
  onClose: () => void;
  title?: string;
  footer?: React.ReactNode;
  width?: number | string;
  closable?: boolean;
}

interface ConfirmModalProps extends ModalProps {
  onConfirm: () => void;
  confirmText?: string;
  cancelText?: string;
  confirmLoading?: boolean;
  danger?: boolean;
}
```

---

## 8. STORE TYPES

### 8.1 Auth Store

```typescript
interface AuthState {
  user: User | null;
  accessToken: string | null;
  isAuthenticated: boolean;
  isLoading: boolean;
  error: string | null;
}

interface AuthActions {
  login: (request: LoginRequest) => Promise<void>;
  logout: () => void;
  refreshToken: () => Promise<void>;
  updateUser: (user: Partial<User>) => void;
  clearError: () => void;
}

type AuthStore = AuthState & AuthActions;
```

### 8.2 Notification Store

```typescript
interface NotificationState {
  notifications: Notification[];
  unreadCount: number;
  isLoading: boolean;
}

interface NotificationActions {
  fetchNotifications: () => Promise<void>;
  markAsRead: (id: number) => Promise<void>;
  markAllAsRead: () => Promise<void>;
  addNotification: (notification: Notification) => void;
  removeNotification: (id: number) => void;
}

type NotificationStore = NotificationState & NotificationActions;
```

### 8.3 UI Store

```typescript
interface UIState {
  sidebarCollapsed: boolean;
  theme: 'light' | 'dark';
  language: 'tr' | 'en';
  breadcrumbs: Breadcrumb[];
}

interface Breadcrumb {
  label: string;
  path?: string;
}

interface UIActions {
  toggleSidebar: () => void;
  setSidebarCollapsed: (collapsed: boolean) => void;
  setTheme: (theme: 'light' | 'dark') => void;
  setLanguage: (lang: 'tr' | 'en') => void;
  setBreadcrumbs: (breadcrumbs: Breadcrumb[]) => void;
}

type UIStore = UIState & UIActions;
```

---

## 9. HOOK RETURN TYPES

```typescript
// useAuth hook
interface UseAuthReturn {
  user: User | null;
  isAuthenticated: boolean;
  isLoading: boolean;
  login: (request: LoginRequest) => Promise<void>;
  logout: () => void;
  hasRole: (role: UserRole) => boolean;
  hasAnyRole: (roles: UserRole[]) => boolean;
}

// useApplications hook
interface UseApplicationsReturn {
  applications: ApplicationListItem[];
  isLoading: boolean;
  error: Error | null;
  pagination: PaginationConfig;
  setPagination: (config: PaginationConfig) => void;
  filters: ApplicationFilters;
  setFilters: (filters: ApplicationFilters) => void;
  refetch: () => void;
}

// useNotifications hook
interface UseNotificationsReturn {
  notifications: Notification[];
  unreadCount: number;
  isLoading: boolean;
  markAsRead: (id: number) => Promise<void>;
  markAllAsRead: () => Promise<void>;
}
```

---

## 10. REVISION HISTORY

| Version | Date | Author | Changes |
|---------|------|--------|---------|
| 1.0 | Mar 2026 | Group 05 | Initial version |
