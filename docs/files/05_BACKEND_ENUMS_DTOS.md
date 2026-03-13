# UTMS - Undergraduate Transfer Management System
## 05 - Backend Enums & DTOs

**Version:** 1.0  
**Last Updated:** March 2026

---

## PART A: ENUMERATIONS

### Package: `com.iztech.utms.enums`

---

### 1. UserRole

```java
package com.iztech.utms.enums;

import lombok.Getter;
import lombok.RequiredArgsConstructor;

@Getter
@RequiredArgsConstructor
public enum UserRole {
    
    STUDENT("Öğrenci", "Student applicant"),
    OIDB_STAFF("ÖİDB Personeli", "Registrar's Office staff"),
    DEAN_STAFF("Dekanlık Personeli", "Dean's Office staff"),
    YGK_MEMBER("YGK Üyesi", "Transfer Committee member"),
    YDYO_STAFF("YDYO Personeli", "Foreign Languages School staff"),
    SYSTEM_ADMIN("Sistem Yöneticisi", "System administrator");

    private final String displayNameTr;
    private final String displayNameEn;

    public boolean isStaff() {
        return this != STUDENT;
    }

    public boolean canReviewDocuments() {
        return this == OIDB_STAFF || this == SYSTEM_ADMIN;
    }

    public boolean canEvaluateLanguage() {
        return this == YDYO_STAFF || this == SYSTEM_ADMIN;
    }

    public boolean canPrepareIntibak() {
        return this == YGK_MEMBER || this == SYSTEM_ADMIN;
    }

    public boolean canMakeFinalDecision() {
        return this == DEAN_STAFF || this == SYSTEM_ADMIN;
    }

    public boolean canTransferApplication() {
        return this == OIDB_STAFF || this == DEAN_STAFF || 
               this == YGK_MEMBER || this == YDYO_STAFF || this == SYSTEM_ADMIN;
    }

    public boolean canViewAllApplications() {
        return this != STUDENT;
    }

    public boolean canManageUsers() {
        return this == SYSTEM_ADMIN;
    }
}
```

---

### 2. ApplicationStatus

```java
package com.iztech.utms.enums;

import lombok.Getter;
import lombok.RequiredArgsConstructor;

@Getter
@RequiredArgsConstructor
public enum ApplicationStatus {

    // Initial States
    DRAFT("Taslak", "Draft", false, true),
    SUBMITTED("Gönderildi", "Submitted", false, false),

    // Document Review
    DOCUMENTS_UNDER_REVIEW("Belgeler İnceleniyor", "Documents Under Review", false, false),
    DOCUMENTS_APPROVED("Belgeler Onaylandı", "Documents Approved", false, false),
    DOCUMENTS_REJECTED("Belgeler Reddedildi", "Documents Rejected", true, false),

    // Language Evaluation
    PENDING_LANGUAGE_EVALUATION("Dil Değerlendirmesi Bekleniyor", "Pending Language Evaluation", false, false),
    LANGUAGE_APPROVED("Dil Yeterliliği Onaylandı", "Language Approved", false, false),
    LANGUAGE_REJECTED("Dil Yeterliliği Reddedildi", "Language Rejected", true, false),

    // Academic Evaluation
    PENDING_ACADEMIC_EVALUATION("Akademik Değerlendirme Bekleniyor", "Pending Academic Evaluation", false, false),
    PENDING_INTIBAK("İntibak Bekleniyor", "Pending Intibak", false, false),
    INTIBAK_COMPLETED("İntibak Tamamlandı", "Intibak Completed", false, false),

    // Final Decision
    PENDING_FACULTY_DECISION("Fakülte Kararı Bekleniyor", "Pending Faculty Decision", false, false),
    
    // Terminal States
    ASIL("Asil (Kabul)", "Accepted (Primary)", true, false),
    YEDEK("Yedek", "Waitlisted", true, false),
    REJECTED("Reddedildi", "Rejected", true, false),
    WITHDRAWN("Geri Çekildi", "Withdrawn", true, false);

    private final String displayNameTr;
    private final String displayNameEn;
    private final boolean terminal;
    private final boolean editable;

    public boolean isActive() {
        return !terminal;
    }

    public boolean isAccepted() {
        return this == ASIL;
    }

    public boolean isRejected() {
        return this == REJECTED || this == DOCUMENTS_REJECTED || this == LANGUAGE_REJECTED;
    }

    public boolean isWaitlisted() {
        return this == YEDEK;
    }

    public boolean isWithdrawn() {
        return this == WITHDRAWN;
    }

    public boolean canBeWithdrawn() {
        return !terminal || this == YEDEK;
    }

    public int getProgressPercentage() {
        return switch (this) {
            case DRAFT -> 0;
            case SUBMITTED -> 10;
            case DOCUMENTS_UNDER_REVIEW -> 20;
            case DOCUMENTS_APPROVED -> 30;
            case PENDING_LANGUAGE_EVALUATION -> 40;
            case LANGUAGE_APPROVED -> 50;
            case PENDING_ACADEMIC_EVALUATION -> 60;
            case PENDING_INTIBAK -> 70;
            case INTIBAK_COMPLETED -> 80;
            case PENDING_FACULTY_DECISION -> 90;
            case ASIL, YEDEK, REJECTED, WITHDRAWN -> 100;
            default -> 0;
        };
    }
}
```

---

### 3. WorkflowStage

```java
package com.iztech.utms.enums;

import lombok.Getter;
import lombok.RequiredArgsConstructor;

import java.util.Arrays;
import java.util.List;

@Getter
@RequiredArgsConstructor
public enum WorkflowStage {

    OIDB_INITIAL(
        "ÖİDB İlk İnceleme",
        "Initial OIDB Review",
        UserRole.OIDB_STAFF,
        1
    ),
    YDYO_EVALUATION(
        "YDYO Dil Değerlendirmesi",
        "Language Evaluation",
        UserRole.YDYO_STAFF,
        2
    ),
    OIDB_POST_LANGUAGE(
        "ÖİDB Dil Sonrası İnceleme",
        "Post-Language OIDB Review",
        UserRole.OIDB_STAFF,
        3
    ),
    DEANS_OFFICE_REVIEW(
        "Dekanlık İncelemesi",
        "Dean's Office Review",
        UserRole.DEAN_STAFF,
        4
    ),
    YGK_EVALUATION(
        "YGK Değerlendirmesi",
        "Transfer Committee Evaluation",
        UserRole.YGK_MEMBER,
        5
    ),
    DEANS_OFFICE_FINAL(
        "Fakülte Kurulu Kararı",
        "Faculty Board Decision",
        UserRole.DEAN_STAFF,
        6
    ),
    OIDB_FINAL(
        "ÖİDB Sonuç Yayını",
        "Final Publication",
        UserRole.OIDB_STAFF,
        7
    ),
    COMPLETED(
        "Tamamlandı",
        "Completed",
        null,
        8
    );

    private final String displayNameTr;
    private final String displayNameEn;
    private final UserRole responsibleRole;
    private final int order;

    public WorkflowStage getNextStage() {
        int nextOrder = this.order + 1;
        return Arrays.stream(values())
            .filter(stage -> stage.order == nextOrder)
            .findFirst()
            .orElse(COMPLETED);
    }

    public WorkflowStage getPreviousStage() {
        int prevOrder = this.order - 1;
        return Arrays.stream(values())
            .filter(stage -> stage.order == prevOrder)
            .findFirst()
            .orElse(this);
    }

    public boolean isFirst() {
        return this == OIDB_INITIAL;
    }

    public boolean isLast() {
        return this == COMPLETED;
    }

    public boolean isBeforeLanguageEvaluation() {
        return this.order < YDYO_EVALUATION.order;
    }

    public boolean isAfterLanguageEvaluation() {
        return this.order > YDYO_EVALUATION.order;
    }

    public boolean canBeHandledBy(UserRole role) {
        if (role == UserRole.SYSTEM_ADMIN) return true;
        return this.responsibleRole == role;
    }

    public static List<WorkflowStage> getOrderedStages() {
        return Arrays.stream(values())
            .sorted((a, b) -> Integer.compare(a.order, b.order))
            .toList();
    }

    public static WorkflowStage getFirstStage() {
        return OIDB_INITIAL;
    }
}
```

---

### 4. DocumentType

```java
package com.iztech.utms.enums;

import lombok.Getter;
import lombok.RequiredArgsConstructor;

@Getter
@RequiredArgsConstructor
public enum DocumentType {

    TRANSCRIPT(
        "Transkript",
        "Transcript",
        true,
        "Official transcript from current university"
    ),
    YKS_RESULT(
        "YKS Sonuç Belgesi",
        "YKS Result Document",
        true,
        "YKS exam result document"
    ),
    LANGUAGE_CERTIFICATE(
        "Dil Yeterlilik Belgesi",
        "Language Certificate",
        false,
        "Language proficiency certificate (YDS, TOEFL, etc.)"
    ),
    COURSE_CONTENT(
        "Ders İçerikleri",
        "Course Contents",
        true,
        "Syllabus and course content documents"
    ),
    ID_CARD(
        "Kimlik Kartı",
        "ID Card",
        true,
        "Copy of national ID card"
    ),
    PORTFOLIO(
        "Portfolyo",
        "Portfolio",
        false,
        "Portfolio for design/architecture departments"
    ),
    DISCIPLINARY_CERTIFICATE(
        "Disiplin Belgesi",
        "Disciplinary Certificate",
        true,
        "Certificate confirming no disciplinary actions"
    ),
    STUDENT_CERTIFICATE(
        "Öğrenci Belgesi",
        "Student Certificate",
        true,
        "Current student status certificate"
    ),
    OTHER(
        "Diğer",
        "Other",
        false,
        "Other supporting documents"
    );

    private final String displayNameTr;
    private final String displayNameEn;
    private final boolean required;
    private final String description;

    public boolean isOptional() {
        return !required;
    }

    public static DocumentType[] getRequiredTypes() {
        return new DocumentType[] {
            TRANSCRIPT, YKS_RESULT, COURSE_CONTENT, ID_CARD, 
            DISCIPLINARY_CERTIFICATE, STUDENT_CERTIFICATE
        };
    }
}
```

---

### 5. DocumentStatus

```java
package com.iztech.utms.enums;

import lombok.Getter;
import lombok.RequiredArgsConstructor;

@Getter
@RequiredArgsConstructor
public enum DocumentStatus {

    PENDING("Beklemede", "Pending", "#faad14"),
    APPROVED("Onaylandı", "Approved", "#52c41a"),
    REJECTED("Reddedildi", "Rejected", "#f5222d"),
    REQUIRES_RESUBMISSION("Yeniden Yüklenmeli", "Requires Resubmission", "#fa8c16");

    private final String displayNameTr;
    private final String displayNameEn;
    private final String colorCode;

    public boolean isFinal() {
        return this == APPROVED || this == REJECTED;
    }

    public boolean needsAction() {
        return this == PENDING || this == REQUIRES_RESUBMISSION;
    }
}
```

---

### 6. NotificationType

```java
package com.iztech.utms.enums;

import lombok.Getter;
import lombok.RequiredArgsConstructor;

@Getter
@RequiredArgsConstructor
public enum NotificationType {

    APPLICATION_SUBMITTED("Başvuru Alındı", "Application Submitted"),
    DOCUMENT_REQUESTED("Belge Talep Edildi", "Document Requested"),
    DOCUMENT_APPROVED("Belge Onaylandı", "Document Approved"),
    DOCUMENT_REJECTED("Belge Reddedildi", "Document Rejected"),
    STATUS_CHANGED("Durum Değişti", "Status Changed"),
    STAGE_CHANGED("Aşama Değişti", "Stage Changed"),
    LANGUAGE_RESULT("Dil Sonucu", "Language Result"),
    RANKING_RESULT("Sıralama Sonucu", "Ranking Result"),
    FINAL_DECISION("Nihai Karar", "Final Decision"),
    SYSTEM_ANNOUNCEMENT("Sistem Duyurusu", "System Announcement"),
    REMINDER("Hatırlatma", "Reminder");

    private final String displayNameTr;
    private final String displayNameEn;

    public boolean isImportant() {
        return this == DOCUMENT_REJECTED || this == FINAL_DECISION || 
               this == LANGUAGE_RESULT || this == RANKING_RESULT;
    }
}
```

---

### 7. NotificationPriority

```java
package com.iztech.utms.enums;

import lombok.Getter;
import lombok.RequiredArgsConstructor;

@Getter
@RequiredArgsConstructor
public enum NotificationPriority {

    LOW("Düşük", "Low", 0),
    NORMAL("Normal", "Normal", 1),
    HIGH("Yüksek", "High", 2),
    URGENT("Acil", "Urgent", 3);

    private final String displayNameTr;
    private final String displayNameEn;
    private final int level;

    public boolean isHighPriority() {
        return level >= HIGH.level;
    }
}
```

---

### 8. IntibakDecision

```java
package com.iztech.utms.enums;

import lombok.Getter;
import lombok.RequiredArgsConstructor;

@Getter
@RequiredArgsConstructor
public enum IntibakDecision {

    PENDING("Beklemede", "Pending", "#faad14"),
    ACCEPTED("Kabul Edildi", "Accepted", "#52c41a"),
    REJECTED("Reddedildi", "Rejected", "#f5222d"),
    CONDITIONAL("Koşullu Kabul", "Conditional", "#1890ff");

    private final String displayNameTr;
    private final String displayNameEn;
    private final String colorCode;

    public boolean countsAsAccepted() {
        return this == ACCEPTED || this == CONDITIONAL;
    }
}
```

---

## PART B: DATA TRANSFER OBJECTS (DTOs)

### Package: `com.iztech.utms.dto`

---

### 1. REQUEST DTOs

#### 1.1 Authentication Requests

```java
package com.iztech.utms.dto.request;

import jakarta.validation.constraints.*;
import lombok.*;

public class AuthRequests {

    @Data
    @NoArgsConstructor
    @AllArgsConstructor
    @Builder
    public static class LoginRequest {
        
        @NotBlank(message = "Email is required")
        @Email(message = "Invalid email format")
        private String email;

        @NotBlank(message = "Password is required")
        private String password;
    }

    @Data
    @NoArgsConstructor
    @AllArgsConstructor
    @Builder
    public static class RegisterRequest {
        
        @NotBlank(message = "Email is required")
        @Email(message = "Invalid email format")
        @Size(max = 255)
        private String email;

        @NotBlank(message = "Password is required")
        @Size(min = 8, max = 100, message = "Password must be between 8 and 100 characters")
        @Pattern(
            regexp = "^(?=.*[a-z])(?=.*[A-Z])(?=.*\\d)(?=.*[@$!%*?&])[A-Za-z\\d@$!%*?&]+$",
            message = "Password must contain at least one uppercase, one lowercase, one digit, and one special character"
        )
        private String password;

        @NotBlank(message = "Confirm password is required")
        private String confirmPassword;

        @NotBlank(message = "First name is required")
        @Size(min = 2, max = 100)
        private String firstName;

        @NotBlank(message = "Last name is required")
        @Size(min = 2, max = 100)
        private String lastName;

        @Pattern(regexp = "^\\+90[0-9]{10}$", message = "Invalid Turkish phone number")
        private String phone;

        @AssertTrue(message = "KVKK consent is required")
        private Boolean kvkkConsent;
    }

    @Data
    @NoArgsConstructor
    @AllArgsConstructor
    public static class ForgotPasswordRequest {
        
        @NotBlank(message = "Email is required")
        @Email(message = "Invalid email format")
        private String email;
    }

    @Data
    @NoArgsConstructor
    @AllArgsConstructor
    public static class ResetPasswordRequest {
        
        @NotBlank(message = "Token is required")
        private String token;

        @NotBlank(message = "Password is required")
        @Size(min = 8, max = 100)
        @Pattern(
            regexp = "^(?=.*[a-z])(?=.*[A-Z])(?=.*\\d)(?=.*[@$!%*?&])[A-Za-z\\d@$!%*?&]+$",
            message = "Password must contain at least one uppercase, one lowercase, one digit, and one special character"
        )
        private String password;

        @NotBlank(message = "Confirm password is required")
        private String confirmPassword;
    }

    @Data
    @NoArgsConstructor
    @AllArgsConstructor
    public static class ChangePasswordRequest {
        
        @NotBlank(message = "Current password is required")
        private String currentPassword;

        @NotBlank(message = "New password is required")
        @Size(min = 8, max = 100)
        @Pattern(
            regexp = "^(?=.*[a-z])(?=.*[A-Z])(?=.*\\d)(?=.*[@$!%*?&])[A-Za-z\\d@$!%*?&]+$",
            message = "Password must contain at least one uppercase, one lowercase, one digit, and one special character"
        )
        private String newPassword;

        @NotBlank(message = "Confirm password is required")
        private String confirmPassword;
    }

    @Data
    @NoArgsConstructor
    @AllArgsConstructor
    public static class RefreshTokenRequest {
        
        @NotBlank(message = "Refresh token is required")
        private String refreshToken;
    }
}
```

#### 1.2 Application Requests

```java
package com.iztech.utms.dto.request;

import jakarta.validation.constraints.*;
import lombok.*;

import java.math.BigDecimal;

public class ApplicationRequests {

    @Data
    @NoArgsConstructor
    @AllArgsConstructor
    @Builder
    public static class CreateApplicationRequest {
        
        @NotNull(message = "Department ID is required")
        private Long departmentId;

        @NotBlank(message = "Application term is required")
        @Pattern(regexp = "^(FALL|SPRING)$", message = "Term must be FALL or SPRING")
        private String applicationTerm;

        @NotBlank(message = "Source university is required")
        @Size(max = 255)
        private String sourceUniversity;

        @Size(max = 255)
        private String sourceFaculty;

        @NotBlank(message = "Source department is required")
        @Size(max = 255)
        private String sourceDepartment;

        @NotNull(message = "Current semester is required")
        @Min(value = 1, message = "Semester must be at least 1")
        @Max(value = 14, message = "Semester cannot exceed 14")
        private Integer currentSemester;

        @NotNull(message = "YKS score is required")
        @DecimalMin(value = "0.00", message = "YKS score must be positive")
        private BigDecimal yksScore;

        @NotNull(message = "YKS ranking is required")
        @Positive(message = "YKS ranking must be positive")
        private Integer yksRanking;

        @NotNull(message = "GPA is required")
        @DecimalMin(value = "0.00", message = "GPA must be at least 0.00")
        @DecimalMax(value = "4.00", message = "GPA cannot exceed 4.00")
        private BigDecimal gpa;

        @Size(max = 50)
        private String languageCertType;

        private Integer languageScore;
    }

    @Data
    @NoArgsConstructor
    @AllArgsConstructor
    @Builder
    public static class UpdateApplicationRequest {
        
        @Size(max = 255)
        private String sourceUniversity;

        @Size(max = 255)
        private String sourceFaculty;

        @Size(max = 255)
        private String sourceDepartment;

        @Min(value = 1)
        @Max(value = 14)
        private Integer currentSemester;

        @DecimalMin(value = "0.00")
        private BigDecimal yksScore;

        @Positive
        private Integer yksRanking;

        @DecimalMin(value = "0.00")
        @DecimalMax(value = "4.00")
        private BigDecimal gpa;

        @Size(max = 50)
        private String languageCertType;

        private Integer languageScore;
    }

    @Data
    @NoArgsConstructor
    @AllArgsConstructor
    public static class WithdrawRequest {
        
        @Size(max = 500)
        private String reason;
    }
}
```

#### 1.3 Workflow Requests

```java
package com.iztech.utms.dto.request;

import com.iztech.utms.enums.WorkflowStage;
import jakarta.validation.constraints.*;
import lombok.*;

public class WorkflowRequests {

    @Data
    @NoArgsConstructor
    @AllArgsConstructor
    @Builder
    public static class TransferRequest {
        
        @NotBlank(message = "Action is required")
        @Pattern(
            regexp = "^(APPROVE_AND_FORWARD|REJECT|REQUEST_DOCUMENTS|RETURN_TO_PREVIOUS)$",
            message = "Invalid action"
        )
        private String action;

        private WorkflowStage toStage;

        @Size(max = 2000)
        private String notes;
    }

    @Data
    @NoArgsConstructor
    @AllArgsConstructor
    public static class LanguageEvaluationRequest {
        
        @NotNull(message = "Approval decision is required")
        private Boolean approved;

        @Size(max = 1000)
        private String notes;
    }
}
```

#### 1.4 Document Requests

```java
package com.iztech.utms.dto.request;

import com.iztech.utms.enums.DocumentStatus;
import jakarta.validation.constraints.*;
import lombok.*;

public class DocumentRequests {

    @Data
    @NoArgsConstructor
    @AllArgsConstructor
    @Builder
    public static class ReviewDocumentRequest {
        
        @NotNull(message = "Status is required")
        private DocumentStatus status;

        @Size(max = 1000)
        private String comments;
    }
}
```

#### 1.5 Intibak Requests

```java
package com.iztech.utms.dto.request;

import com.iztech.utms.enums.IntibakDecision;
import jakarta.validation.constraints.*;
import lombok.*;

import java.math.BigDecimal;

public class IntibakRequests {

    @Data
    @NoArgsConstructor
    @AllArgsConstructor
    @Builder
    public static class CreateIntibakRequest {
        
        @NotBlank(message = "Source course code is required")
        @Size(max = 50)
        private String sourceCourseCode;

        @NotBlank(message = "Source course name is required")
        @Size(max = 255)
        private String sourceCourseName;

        @NotNull(message = "Source credits is required")
        @Positive
        private Integer sourceCredits;

        private BigDecimal sourceEcts;

        @NotBlank(message = "Source grade is required")
        @Size(max = 10)
        private String sourceGrade;

        private BigDecimal sourceGradePoint;

        @Size(max = 50)
        private String targetCourseCode;

        @Size(max = 255)
        private String targetCourseName;

        private Integer targetCredits;

        private BigDecimal targetEcts;

        @NotNull(message = "Decision is required")
        private IntibakDecision decision;

        @Size(max = 1000)
        private String notes;
    }

    @Data
    @NoArgsConstructor
    @AllArgsConstructor
    public static class UpdateIntibakRequest {
        
        @Size(max = 50)
        private String targetCourseCode;

        @Size(max = 255)
        private String targetCourseName;

        private Integer targetCredits;

        private BigDecimal targetEcts;

        private IntibakDecision decision;

        @Size(max = 1000)
        private String notes;
    }
}
```

---

### 2. RESPONSE DTOs

#### 2.1 Common Responses

```java
package com.iztech.utms.dto.response;

import com.fasterxml.jackson.annotation.JsonInclude;
import lombok.*;

import java.time.LocalDateTime;
import java.util.List;

@Data
@NoArgsConstructor
@AllArgsConstructor
@Builder
@JsonInclude(JsonInclude.Include.NON_NULL)
public class ApiResponse<T> {
    
    private boolean success;
    private T data;
    private String message;
    private LocalDateTime timestamp;

    public static <T> ApiResponse<T> success(T data) {
        return ApiResponse.<T>builder()
            .success(true)
            .data(data)
            .timestamp(LocalDateTime.now())
            .build();
    }

    public static <T> ApiResponse<T> success(T data, String message) {
        return ApiResponse.<T>builder()
            .success(true)
            .data(data)
            .message(message)
            .timestamp(LocalDateTime.now())
            .build();
    }

    public static <T> ApiResponse<T> error(String message) {
        return ApiResponse.<T>builder()
            .success(false)
            .message(message)
            .timestamp(LocalDateTime.now())
            .build();
    }
}

@Data
@NoArgsConstructor
@AllArgsConstructor
@Builder
public class ErrorResponse {
    
    private boolean success;
    private ErrorDetails error;
    private LocalDateTime timestamp;

    @Data
    @NoArgsConstructor
    @AllArgsConstructor
    @Builder
    public static class ErrorDetails {
        private String code;
        private String message;
        private List<FieldError> details;
    }

    @Data
    @NoArgsConstructor
    @AllArgsConstructor
    public static class FieldError {
        private String field;
        private String message;
    }
}

@Data
@NoArgsConstructor
@AllArgsConstructor
@Builder
public class PageResponse<T> {
    
    private List<T> content;
    private int page;
    private int size;
    private long totalElements;
    private int totalPages;
    private boolean first;
    private boolean last;
}
```

#### 2.2 Authentication Responses

```java
package com.iztech.utms.dto.response;

import com.iztech.utms.enums.UserRole;
import lombok.*;

import java.time.LocalDateTime;

public class AuthResponses {

    @Data
    @NoArgsConstructor
    @AllArgsConstructor
    @Builder
    public static class LoginResponse {
        
        private String accessToken;
        private String refreshToken;
        private String tokenType;
        private long expiresIn;
        private UserInfo user;
    }

    @Data
    @NoArgsConstructor
    @AllArgsConstructor
    @Builder
    public static class UserInfo {
        
        private Long id;
        private String email;
        private String firstName;
        private String lastName;
        private UserRole role;
        private LocalDateTime lastLoginAt;
    }

    @Data
    @NoArgsConstructor
    @AllArgsConstructor
    @Builder
    public static class RefreshResponse {
        
        private String accessToken;
        private long expiresIn;
    }
}
```

#### 2.3 User Responses

```java
package com.iztech.utms.dto.response;

import com.iztech.utms.enums.UserRole;
import lombok.*;

import java.time.LocalDateTime;

public class UserResponses {

    @Data
    @NoArgsConstructor
    @AllArgsConstructor
    @Builder
    public static class UserResponse {
        
        private Long id;
        private String email;
        private String firstName;
        private String lastName;
        private String phone;
        private UserRole role;
        private Boolean isActive;
        private Boolean kvkkConsent;
        private LocalDateTime kvkkConsentDate;
        private LocalDateTime lastLoginAt;
        private LocalDateTime createdAt;
    }

    @Data
    @NoArgsConstructor
    @AllArgsConstructor
    @Builder
    public static class UserListResponse {
        
        private Long id;
        private String email;
        private String firstName;
        private String lastName;
        private UserRole role;
        private Boolean isActive;
        private LocalDateTime createdAt;
    }
}
```

#### 2.4 Application Responses

```java
package com.iztech.utms.dto.response;

import com.iztech.utms.enums.ApplicationStatus;
import com.iztech.utms.enums.WorkflowStage;
import lombok.*;

import java.math.BigDecimal;
import java.time.LocalDateTime;
import java.util.List;

public class ApplicationResponses {

    @Data
    @NoArgsConstructor
    @AllArgsConstructor
    @Builder
    public static class ApplicationResponse {
        
        private Long id;
        private String applicationNumber;
        private Integer applicationYear;
        private String applicationTerm;
        private ApplicationStatus status;
        private WorkflowStage currentStage;
        
        private ApplicantInfo applicant;
        private DepartmentInfo department;
        private AcademicInfo academicInfo;
        private LanguageInfo languageInfo;
        private RankingInfo ranking;
        private AssigneeInfo assignedTo;
        
        private List<DocumentResponse> documents;
        private List<WorkflowHistoryResponse> workflowHistory;
        
        private TimestampInfo timestamps;
    }

    @Data
    @NoArgsConstructor
    @AllArgsConstructor
    @Builder
    public static class ApplicationListResponse {
        
        private Long id;
        private String applicationNumber;
        private ApplicationStatus status;
        private WorkflowStage currentStage;
        private ApplicantInfo applicant;
        private DepartmentInfo department;
        private String sourceUniversity;
        private BigDecimal gpa;
        private BigDecimal transferGrade;
        private LocalDateTime submittedAt;
        private int documentsCount;
        private int pendingDocumentsCount;
    }

    @Data
    @NoArgsConstructor
    @AllArgsConstructor
    @Builder
    public static class ApplicantInfo {
        private Long id;
        private String firstName;
        private String lastName;
        private String email;
        private String phone;
    }

    @Data
    @NoArgsConstructor
    @AllArgsConstructor
    @Builder
    public static class DepartmentInfo {
        private Long id;
        private String name;
        private String code;
        private Integer quota;
        private FacultyInfo faculty;
    }

    @Data
    @NoArgsConstructor
    @AllArgsConstructor
    @Builder
    public static class FacultyInfo {
        private Long id;
        private String name;
    }

    @Data
    @NoArgsConstructor
    @AllArgsConstructor
    @Builder
    public static class AcademicInfo {
        private String sourceUniversity;
        private String sourceFaculty;
        private String sourceDepartment;
        private Integer currentSemester;
        private BigDecimal yksScore;
        private Integer yksRanking;
        private BigDecimal gpa;
        private BigDecimal transferGrade;
    }

    @Data
    @NoArgsConstructor
    @AllArgsConstructor
    @Builder
    public static class LanguageInfo {
        private String certificateType;
        private Integer score;
        private Boolean approved;
        private LocalDateTime evaluatedAt;
    }

    @Data
    @NoArgsConstructor
    @AllArgsConstructor
    @Builder
    public static class RankingInfo {
        private Boolean isAsil;
        private Integer position;
    }

    @Data
    @NoArgsConstructor
    @AllArgsConstructor
    @Builder
    public static class AssigneeInfo {
        private Long id;
        private String firstName;
        private String lastName;
    }

    @Data
    @NoArgsConstructor
    @AllArgsConstructor
    @Builder
    public static class TimestampInfo {
        private LocalDateTime createdAt;
        private LocalDateTime submittedAt;
        private LocalDateTime updatedAt;
    }

    @Data
    @NoArgsConstructor
    @AllArgsConstructor
    @Builder
    public static class ApplicationStatusResponse {
        private Long applicationId;
        private String applicationNumber;
        private ApplicationStatus currentStatus;
        private WorkflowStage currentStage;
        private int progress;
        private List<StageInfo> timeline;
        private int estimatedCompletionDays;
        private LocalDateTime lastUpdated;
    }

    @Data
    @NoArgsConstructor
    @AllArgsConstructor
    @Builder
    public static class StageInfo {
        private WorkflowStage stage;
        private String status; // COMPLETED, IN_PROGRESS, PENDING
        private LocalDateTime enteredAt;
        private LocalDateTime completedAt;
        private String description;
    }
}
```

#### 2.5 Document Responses

```java
package com.iztech.utms.dto.response;

import com.iztech.utms.enums.DocumentStatus;
import com.iztech.utms.enums.DocumentType;
import lombok.*;

import java.time.LocalDateTime;

@Data
@NoArgsConstructor
@AllArgsConstructor
@Builder
public class DocumentResponse {
    
    private Long id;
    private Long applicationId;
    private DocumentType documentType;
    private String fileName;
    private String originalFileName;
    private Long fileSize;
    private String mimeType;
    private DocumentStatus status;
    private String reviewComments;
    private LocalDateTime uploadedAt;
    private LocalDateTime reviewedAt;
    private ReviewerInfo reviewedBy;

    @Data
    @NoArgsConstructor
    @AllArgsConstructor
    @Builder
    public static class ReviewerInfo {
        private Long id;
        private String firstName;
        private String lastName;
    }
}
```

#### 2.6 Workflow Responses

```java
package com.iztech.utms.dto.response;

import com.iztech.utms.enums.UserRole;
import com.iztech.utms.enums.WorkflowStage;
import lombok.*;

import java.time.LocalDateTime;

@Data
@NoArgsConstructor
@AllArgsConstructor
@Builder
public class WorkflowHistoryResponse {
    
    private Long id;
    private WorkflowStage fromStage;
    private WorkflowStage toStage;
    private String action;
    private PerformerInfo performedBy;
    private String notes;
    private LocalDateTime createdAt;

    @Data
    @NoArgsConstructor
    @AllArgsConstructor
    @Builder
    public static class PerformerInfo {
        private Long id;
        private String firstName;
        private String lastName;
        private UserRole role;
    }
}
```

#### 2.7 Evaluation Responses

```java
package com.iztech.utms.dto.response;

import lombok.*;

import java.math.BigDecimal;
import java.time.LocalDateTime;
import java.util.List;

public class EvaluationResponses {

    @Data
    @NoArgsConstructor
    @AllArgsConstructor
    @Builder
    public static class TransferGradeResponse {
        
        private Long applicationId;
        private BigDecimal yksScore;
        private BigDecimal minYksScore;
        private BigDecimal gpa;
        private BigDecimal transferGrade;
        private String formula;
        private LocalDateTime calculatedAt;
    }

    @Data
    @NoArgsConstructor
    @AllArgsConstructor
    @Builder
    public static class RankingResponse {
        
        private Long departmentId;
        private String departmentName;
        private Integer applicationYear;
        private String applicationTerm;
        private Integer quota;
        private Integer totalApplications;
        private Integer eligibleApplications;
        private List<RankingEntry> rankings;
        private LocalDateTime calculatedAt;
    }

    @Data
    @NoArgsConstructor
    @AllArgsConstructor
    @Builder
    public static class RankingEntry {
        
        private Integer position;
        private Long applicationId;
        private String applicantName;
        private BigDecimal transferGrade;
        private Boolean isAsil;
    }

    @Data
    @NoArgsConstructor
    @AllArgsConstructor
    @Builder
    public static class DepartmentRankingsResponse {
        
        private Long departmentId;
        private String departmentName;
        private Integer quota;
        private List<RankingEntry> asil;
        private List<RankingEntry> yedek;
        private LocalDateTime lastCalculatedAt;
    }
}
```

#### 2.8 Intibak Responses

```java
package com.iztech.utms.dto.response;

import com.iztech.utms.enums.IntibakDecision;
import lombok.*;

import java.math.BigDecimal;
import java.time.LocalDateTime;
import java.util.List;

public class IntibakResponses {

    @Data
    @NoArgsConstructor
    @AllArgsConstructor
    @Builder
    public static class IntibakSummaryResponse {
        
        private Long applicationId;
        private Integer totalSourceCredits;
        private Integer totalAcceptedCredits;
        private List<IntibakRecordResponse> records;
        private PreparerInfo preparedBy;
        private LocalDateTime preparedAt;
    }

    @Data
    @NoArgsConstructor
    @AllArgsConstructor
    @Builder
    public static class IntibakRecordResponse {
        
        private Long id;
        private SourceCourseInfo sourceCourse;
        private TargetCourseInfo targetCourse;
        private IntibakDecision decision;
        private String notes;
    }

    @Data
    @NoArgsConstructor
    @AllArgsConstructor
    @Builder
    public static class SourceCourseInfo {
        private String code;
        private String name;
        private Integer credits;
        private String grade;
        private BigDecimal gradePoint;
    }

    @Data
    @NoArgsConstructor
    @AllArgsConstructor
    @Builder
    public static class TargetCourseInfo {
        private String code;
        private String name;
        private Integer credits;
    }

    @Data
    @NoArgsConstructor
    @AllArgsConstructor
    @Builder
    public static class PreparerInfo {
        private Long id;
        private String firstName;
        private String lastName;
    }
}
```

#### 2.9 Notification Responses

```java
package com.iztech.utms.dto.response;

import com.iztech.utms.enums.NotificationPriority;
import com.iztech.utms.enums.NotificationType;
import lombok.*;

import java.time.LocalDateTime;

@Data
@NoArgsConstructor
@AllArgsConstructor
@Builder
public class NotificationResponse {
    
    private Long id;
    private NotificationType type;
    private String title;
    private String message;
    private NotificationPriority priority;
    private Boolean isRead;
    private LocalDateTime readAt;
    private String actionUrl;
    private LocalDateTime createdAt;
}
```

---

## 3. MAPPERS

### Package: `com.iztech.utms.mapper`

```java
package com.iztech.utms.mapper;

import com.iztech.utms.dto.response.*;
import com.iztech.utms.entity.*;
import org.mapstruct.*;

import java.util.List;

@Mapper(componentModel = "spring", unmappedTargetPolicy = ReportingPolicy.IGNORE)
public interface ApplicationMapper {

    @Mapping(target = "applicant", source = "user")
    @Mapping(target = "academicInfo", source = "application")
    @Mapping(target = "languageInfo", source = "application")
    @Mapping(target = "ranking", source = "application")
    @Mapping(target = "timestamps", source = "application")
    ApplicationResponses.ApplicationResponse toResponse(Application application);

    @Mapping(target = "applicant", source = "user")
    @Mapping(target = "documentsCount", expression = "java(application.getDocuments().size())")
    @Mapping(target = "pendingDocumentsCount", expression = "java(application.getPendingDocumentsCount())")
    ApplicationResponses.ApplicationListResponse toListResponse(Application application);

    ApplicationResponses.ApplicantInfo toApplicantInfo(User user);
    ApplicationResponses.DepartmentInfo toDepartmentInfo(Department department);
    ApplicationResponses.FacultyInfo toFacultyInfo(Faculty faculty);

    @Mapping(target = "certificateType", source = "languageCertType")
    @Mapping(target = "score", source = "languageScore")
    @Mapping(target = "approved", source = "languageApproved")
    ApplicationResponses.LanguageInfo toLanguageInfo(Application application);

    @Mapping(target = "sourceUniversity", source = "sourceUniversity")
    @Mapping(target = "sourceFaculty", source = "sourceFaculty")
    @Mapping(target = "sourceDepartment", source = "sourceDepartment")
    ApplicationResponses.AcademicInfo toAcademicInfo(Application application);

    @Mapping(target = "isAsil", source = "isAsil")
    @Mapping(target = "position", source = "rankingPosition")
    ApplicationResponses.RankingInfo toRankingInfo(Application application);

    List<ApplicationResponses.ApplicationListResponse> toListResponses(List<Application> applications);
}

@Mapper(componentModel = "spring", unmappedTargetPolicy = ReportingPolicy.IGNORE)
public interface UserMapper {

    UserResponses.UserResponse toResponse(User user);
    UserResponses.UserListResponse toListResponse(User user);
    AuthResponses.UserInfo toUserInfo(User user);

    List<UserResponses.UserListResponse> toListResponses(List<User> users);
}

@Mapper(componentModel = "spring", unmappedTargetPolicy = ReportingPolicy.IGNORE)
public interface DocumentMapper {

    @Mapping(target = "applicationId", source = "application.id")
    DocumentResponse toResponse(Document document);

    @Mapping(target = "id", source = "reviewedBy.id")
    @Mapping(target = "firstName", source = "reviewedBy.firstName")
    @Mapping(target = "lastName", source = "reviewedBy.lastName")
    DocumentResponse.ReviewerInfo toReviewerInfo(Document document);

    List<DocumentResponse> toResponses(List<Document> documents);
}

@Mapper(componentModel = "spring", unmappedTargetPolicy = ReportingPolicy.IGNORE)
public interface NotificationMapper {

    NotificationResponse toResponse(Notification notification);
    List<NotificationResponse> toResponses(List<Notification> notifications);
}
```

---

## 4. REVISION HISTORY

| Version | Date | Author | Changes |
|---------|------|--------|---------|
| 1.0 | Mar 2026 | Group 05 | Initial version |
