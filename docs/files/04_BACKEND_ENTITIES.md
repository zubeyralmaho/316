# UTMS - Undergraduate Transfer Management System
## 04 - Backend Entities

**Version:** 1.0  
**Last Updated:** March 2026  
**Package:** `com.iztech.utms.entity`

---

## 1. ENTITY OVERVIEW

```
Entity Classes:
├── BaseEntity.java          # Abstract base with common fields
├── User.java                # User accounts
├── Faculty.java             # Faculties
├── Department.java          # Departments
├── Application.java         # Transfer applications
├── Document.java            # Uploaded documents
├── WorkflowHistory.java     # Workflow audit trail
├── IntibakRecord.java       # Course equivalence records
├── Notification.java        # User notifications
├── AuditLog.java           # System audit logs
└── PasswordResetToken.java  # Password reset tokens
```

---

## 2. BASE ENTITY

```java
package com.iztech.utms.entity;

import jakarta.persistence.*;
import lombok.Getter;
import lombok.Setter;
import org.hibernate.annotations.CreationTimestamp;
import org.hibernate.annotations.UpdateTimestamp;

import java.io.Serializable;
import java.time.LocalDateTime;

@Getter
@Setter
@MappedSuperclass
public abstract class BaseEntity implements Serializable {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @CreationTimestamp
    @Column(name = "created_at", nullable = false, updatable = false)
    private LocalDateTime createdAt;

    @UpdateTimestamp
    @Column(name = "updated_at")
    private LocalDateTime updatedAt;

    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (o == null || getClass() != o.getClass()) return false;
        BaseEntity that = (BaseEntity) o;
        return id != null && id.equals(that.id);
    }

    @Override
    public int hashCode() {
        return getClass().hashCode();
    }
}
```

---

## 3. USER ENTITY

```java
package com.iztech.utms.entity;

import com.iztech.utms.enums.UserRole;
import jakarta.persistence.*;
import jakarta.validation.constraints.Email;
import jakarta.validation.constraints.NotBlank;
import jakarta.validation.constraints.Size;
import lombok.*;
import org.springframework.security.core.GrantedAuthority;
import org.springframework.security.core.authority.SimpleGrantedAuthority;
import org.springframework.security.core.userdetails.UserDetails;

import java.time.LocalDateTime;
import java.util.ArrayList;
import java.util.Collection;
import java.util.Collections;
import java.util.List;

@Entity
@Table(name = "users", indexes = {
    @Index(name = "idx_users_email", columnList = "email"),
    @Index(name = "idx_users_role", columnList = "role")
})
@Getter
@Setter
@NoArgsConstructor
@AllArgsConstructor
@Builder
public class User extends BaseEntity implements UserDetails {

    @NotBlank(message = "Email is required")
    @Email(message = "Invalid email format")
    @Column(nullable = false, unique = true, length = 255)
    private String email;

    @NotBlank(message = "Password is required")
    @Column(name = "password_hash", nullable = false, length = 255)
    private String passwordHash;

    @NotBlank(message = "First name is required")
    @Size(min = 2, max = 100)
    @Column(name = "first_name", nullable = false, length = 100)
    private String firstName;

    @NotBlank(message = "Last name is required")
    @Size(min = 2, max = 100)
    @Column(name = "last_name", nullable = false, length = 100)
    private String lastName;

    @Enumerated(EnumType.STRING)
    @Column(nullable = false, length = 50)
    private UserRole role;

    @Column(length = 20)
    private String phone;

    @Column(name = "is_active")
    @Builder.Default
    private Boolean isActive = true;

    @Column(name = "account_locked")
    @Builder.Default
    private Boolean accountLocked = false;

    @Column(name = "failed_attempts")
    @Builder.Default
    private Integer failedAttempts = 0;

    @Column(name = "kvkk_consent")
    @Builder.Default
    private Boolean kvkkConsent = false;

    @Column(name = "kvkk_consent_date")
    private LocalDateTime kvkkConsentDate;

    @Column(name = "last_login_at")
    private LocalDateTime lastLoginAt;

    // Relationships
    @OneToMany(mappedBy = "user", cascade = CascadeType.ALL, orphanRemoval = true)
    @Builder.Default
    private List<Application> applications = new ArrayList<>();

    @OneToMany(mappedBy = "user", cascade = CascadeType.ALL, orphanRemoval = true)
    @Builder.Default
    private List<Notification> notifications = new ArrayList<>();

    // UserDetails Implementation
    @Override
    public Collection<? extends GrantedAuthority> getAuthorities() {
        return Collections.singletonList(new SimpleGrantedAuthority("ROLE_" + role.name()));
    }

    @Override
    public String getPassword() {
        return passwordHash;
    }

    @Override
    public String getUsername() {
        return email;
    }

    @Override
    public boolean isAccountNonExpired() {
        return true;
    }

    @Override
    public boolean isAccountNonLocked() {
        return !accountLocked;
    }

    @Override
    public boolean isCredentialsNonExpired() {
        return true;
    }

    @Override
    public boolean isEnabled() {
        return isActive;
    }

    // Helper Methods
    public String getFullName() {
        return firstName + " " + lastName;
    }

    public void incrementFailedAttempts() {
        this.failedAttempts++;
        if (this.failedAttempts >= 5) {
            this.accountLocked = true;
        }
    }

    public void resetFailedAttempts() {
        this.failedAttempts = 0;
        this.accountLocked = false;
    }

    public void recordLogin() {
        this.lastLoginAt = LocalDateTime.now();
        resetFailedAttempts();
    }

    public void giveKvkkConsent() {
        this.kvkkConsent = true;
        this.kvkkConsentDate = LocalDateTime.now();
    }
}
```

---

## 4. FACULTY ENTITY

```java
package com.iztech.utms.entity;

import jakarta.persistence.*;
import jakarta.validation.constraints.NotBlank;
import jakarta.validation.constraints.Size;
import lombok.*;

import java.util.ArrayList;
import java.util.List;

@Entity
@Table(name = "faculties", indexes = {
    @Index(name = "idx_faculties_code", columnList = "code")
})
@Getter
@Setter
@NoArgsConstructor
@AllArgsConstructor
@Builder
public class Faculty extends BaseEntity {

    @NotBlank(message = "Faculty name is required")
    @Size(max = 255)
    @Column(nullable = false, length = 255)
    private String name;

    @NotBlank(message = "Faculty code is required")
    @Size(max = 20)
    @Column(nullable = false, unique = true, length = 20)
    private String code;

    @Column(name = "dean_name", length = 200)
    private String deanName;

    @Column(name = "contact_email", length = 255)
    private String contactEmail;

    @Column(name = "is_active")
    @Builder.Default
    private Boolean isActive = true;

    // Relationships
    @OneToMany(mappedBy = "faculty", cascade = CascadeType.ALL, fetch = FetchType.LAZY)
    @Builder.Default
    private List<Department> departments = new ArrayList<>();

    // Helper Methods
    public int getDepartmentsCount() {
        return departments != null ? departments.size() : 0;
    }

    public void addDepartment(Department department) {
        departments.add(department);
        department.setFaculty(this);
    }

    public void removeDepartment(Department department) {
        departments.remove(department);
        department.setFaculty(null);
    }
}
```

---

## 5. DEPARTMENT ENTITY

```java
package com.iztech.utms.entity;

import jakarta.persistence.*;
import jakarta.validation.constraints.*;
import lombok.*;

import java.math.BigDecimal;
import java.util.ArrayList;
import java.util.List;

@Entity
@Table(name = "departments", indexes = {
    @Index(name = "idx_departments_faculty", columnList = "faculty_id"),
    @Index(name = "idx_departments_code", columnList = "code")
})
@Getter
@Setter
@NoArgsConstructor
@AllArgsConstructor
@Builder
public class Department extends BaseEntity {

    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "faculty_id", nullable = false)
    private Faculty faculty;

    @NotBlank(message = "Department name is required")
    @Size(max = 255)
    @Column(nullable = false, length = 255)
    private String name;

    @NotBlank(message = "Department code is required")
    @Size(max = 20)
    @Column(nullable = false, unique = true, length = 20)
    private String code;

    @Min(value = 0, message = "Quota cannot be negative")
    @Column
    @Builder.Default
    private Integer quota = 0;

    @DecimalMin(value = "0.00", message = "GPA must be at least 0.00")
    @DecimalMax(value = "4.00", message = "GPA cannot exceed 4.00")
    @Column(name = "min_gpa", precision = 3, scale = 2)
    @Builder.Default
    private BigDecimal minGpa = new BigDecimal("2.00");

    @Column(name = "max_ranking")
    private Integer maxRanking;

    @Column(name = "min_yks_score", precision = 10, scale = 2)
    private BigDecimal minYksScore;

    @Column(name = "language_required")
    @Builder.Default
    private Boolean languageRequired = true;

    @Column(name = "min_language_score")
    @Builder.Default
    private Integer minLanguageScore = 60;

    @Column(name = "special_requirements", columnDefinition = "TEXT")
    private String specialRequirements;

    @Column(name = "is_active")
    @Builder.Default
    private Boolean isActive = true;

    // Relationships
    @OneToMany(mappedBy = "department", cascade = CascadeType.ALL, fetch = FetchType.LAZY)
    @Builder.Default
    private List<Application> applications = new ArrayList<>();

    // Helper Methods
    public boolean checkEligibility(BigDecimal applicantGpa, Integer applicantRanking, BigDecimal yksScore) {
        // Check GPA
        if (applicantGpa.compareTo(minGpa) < 0) {
            return false;
        }

        // Check ranking
        if (maxRanking != null && applicantRanking > maxRanking) {
            return false;
        }

        // Check YKS score
        if (minYksScore != null && yksScore.compareTo(minYksScore) < 0) {
            return false;
        }

        return true;
    }

    public int getAvailableSlots() {
        if (applications == null) return quota;
        
        long acceptedCount = applications.stream()
            .filter(app -> Boolean.TRUE.equals(app.getIsAsil()))
            .count();
        
        return Math.max(0, quota - (int) acceptedCount);
    }
}
```

---

## 6. APPLICATION ENTITY

```java
package com.iztech.utms.entity;

import com.iztech.utms.enums.ApplicationStatus;
import com.iztech.utms.enums.WorkflowStage;
import jakarta.persistence.*;
import jakarta.validation.constraints.*;
import lombok.*;

import java.math.BigDecimal;
import java.time.LocalDateTime;
import java.util.ArrayList;
import java.util.List;

@Entity
@Table(name = "applications", indexes = {
    @Index(name = "idx_applications_user", columnList = "user_id"),
    @Index(name = "idx_applications_department", columnList = "department_id"),
    @Index(name = "idx_applications_status", columnList = "status"),
    @Index(name = "idx_applications_stage", columnList = "current_stage"),
    @Index(name = "idx_applications_year_term", columnList = "application_year, application_term")
})
@Getter
@Setter
@NoArgsConstructor
@AllArgsConstructor
@Builder
public class Application extends BaseEntity {

    // Relationships
    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "user_id", nullable = false)
    private User user;

    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "department_id", nullable = false)
    private Department department;

    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "assigned_to")
    private User assignedTo;

    // Application Period
    @NotNull(message = "Application year is required")
    @Column(name = "application_year", nullable = false)
    private Integer applicationYear;

    @NotBlank(message = "Application term is required")
    @Column(name = "application_term", nullable = false, length = 20)
    private String applicationTerm; // FALL, SPRING

    // Status and Workflow
    @Enumerated(EnumType.STRING)
    @Column(nullable = false, length = 50)
    @Builder.Default
    private ApplicationStatus status = ApplicationStatus.DRAFT;

    @Enumerated(EnumType.STRING)
    @Column(name = "current_stage", nullable = false, length = 50)
    @Builder.Default
    private WorkflowStage currentStage = WorkflowStage.OIDB_INITIAL;

    // Source Institution
    @NotBlank(message = "Source university is required")
    @Column(name = "source_university", nullable = false, length = 255)
    private String sourceUniversity;

    @NotBlank(message = "Source department is required")
    @Column(name = "source_department", nullable = false, length = 255)
    private String sourceDepartment;

    @Column(name = "source_faculty", length = 255)
    private String sourceFaculty;

    @Min(value = 1, message = "Semester must be at least 1")
    @Max(value = 14, message = "Semester cannot exceed 14")
    @Column(name = "current_semester", nullable = false)
    private Integer currentSemester;

    // Academic Scores
    @NotNull(message = "YKS score is required")
    @DecimalMin(value = "0.00", message = "YKS score must be positive")
    @Column(name = "yks_score", nullable = false, precision = 10, scale = 2)
    private BigDecimal yksScore;

    @NotNull(message = "YKS ranking is required")
    @Positive(message = "YKS ranking must be positive")
    @Column(name = "yks_ranking", nullable = false)
    private Integer yksRanking;

    @NotNull(message = "GPA is required")
    @DecimalMin(value = "0.00", message = "GPA must be at least 0.00")
    @DecimalMax(value = "4.00", message = "GPA cannot exceed 4.00")
    @Column(nullable = false, precision = 4, scale = 2)
    private BigDecimal gpa;

    // Calculated Fields
    @Column(name = "transfer_grade", precision = 10, scale = 4)
    private BigDecimal transferGrade;

    @Column(name = "is_asil")
    private Boolean isAsil;

    @Column(name = "ranking_position")
    private Integer rankingPosition;

    // Language Info
    @Column(name = "language_cert_type", length = 50)
    private String languageCertType;

    @Column(name = "language_score")
    private Integer languageScore;

    @Column(name = "language_approved")
    private Boolean languageApproved;

    // Timestamps
    @Column(name = "submitted_at")
    private LocalDateTime submittedAt;

    @Column(name = "evaluated_at")
    private LocalDateTime evaluatedAt;

    @Column(name = "decided_at")
    private LocalDateTime decidedAt;

    // Child Collections
    @OneToMany(mappedBy = "application", cascade = CascadeType.ALL, orphanRemoval = true)
    @Builder.Default
    @OrderBy("uploadedAt DESC")
    private List<Document> documents = new ArrayList<>();

    @OneToMany(mappedBy = "application", cascade = CascadeType.ALL, orphanRemoval = true)
    @Builder.Default
    @OrderBy("createdAt DESC")
    private List<WorkflowHistory> workflowHistory = new ArrayList<>();

    @OneToMany(mappedBy = "application", cascade = CascadeType.ALL, orphanRemoval = true)
    @Builder.Default
    @OrderBy("id ASC")
    private List<IntibakRecord> intibakRecords = new ArrayList<>();

    @OneToMany(mappedBy = "application", cascade = CascadeType.ALL, orphanRemoval = true)
    @Builder.Default
    private List<Notification> notifications = new ArrayList<>();

    // Helper Methods
    public String getApplicationNumber() {
        return String.format("%d-%s-%s-%04d", 
            applicationYear, 
            applicationTerm, 
            department.getCode(), 
            getId()
        );
    }

    public void submit() {
        this.status = ApplicationStatus.SUBMITTED;
        this.submittedAt = LocalDateTime.now();
    }

    public void withdraw() {
        this.status = ApplicationStatus.WITHDRAWN;
    }

    public boolean isDraft() {
        return this.status == ApplicationStatus.DRAFT;
    }

    public boolean isEditable() {
        return this.status == ApplicationStatus.DRAFT;
    }

    public boolean canWithdraw() {
        return this.status != ApplicationStatus.WITHDRAWN 
            && this.status != ApplicationStatus.REJECTED
            && this.status != ApplicationStatus.ASIL;
    }

    public void addDocument(Document document) {
        documents.add(document);
        document.setApplication(this);
    }

    public void removeDocument(Document document) {
        documents.remove(document);
        document.setApplication(null);
    }

    public void addWorkflowEntry(WorkflowHistory entry) {
        workflowHistory.add(entry);
        entry.setApplication(this);
    }

    public void addIntibakRecord(IntibakRecord record) {
        intibakRecords.add(record);
        record.setApplication(this);
    }

    public int getPendingDocumentsCount() {
        return (int) documents.stream()
            .filter(d -> d.getStatus().name().equals("PENDING"))
            .count();
    }

    public boolean hasAllRequiredDocuments() {
        // Check for required document types
        boolean hasTranscript = documents.stream()
            .anyMatch(d -> d.getDocumentType().name().equals("TRANSCRIPT"));
        boolean hasYksResult = documents.stream()
            .anyMatch(d -> d.getDocumentType().name().equals("YKS_RESULT"));
        boolean hasIdCard = documents.stream()
            .anyMatch(d -> d.getDocumentType().name().equals("ID_CARD"));
        
        return hasTranscript && hasYksResult && hasIdCard;
    }

    /**
     * Calculate transfer grade using the formula:
     * TG = (x/y * 100 * 0.9) + (0.1 * z)
     * where x = YKS score, y = min YKS score for department, z = GPA
     */
    public void calculateTransferGrade(BigDecimal minYksScore) {
        if (minYksScore == null || minYksScore.compareTo(BigDecimal.ZERO) == 0) {
            throw new IllegalArgumentException("Minimum YKS score must be positive");
        }

        // (yksScore / minYksScore) * 100 * 0.9
        BigDecimal yksComponent = yksScore
            .divide(minYksScore, 10, java.math.RoundingMode.HALF_UP)
            .multiply(new BigDecimal("100"))
            .multiply(new BigDecimal("0.9"));

        // 0.1 * gpa
        BigDecimal gpaComponent = gpa.multiply(new BigDecimal("0.1"));

        // Total
        this.transferGrade = yksComponent.add(gpaComponent)
            .setScale(4, java.math.RoundingMode.HALF_UP);
    }
}
```

---

## 7. DOCUMENT ENTITY

```java
package com.iztech.utms.entity;

import com.iztech.utms.enums.DocumentStatus;
import com.iztech.utms.enums.DocumentType;
import jakarta.persistence.*;
import jakarta.validation.constraints.NotBlank;
import jakarta.validation.constraints.NotNull;
import jakarta.validation.constraints.Positive;
import lombok.*;

import java.time.LocalDateTime;

@Entity
@Table(name = "documents", indexes = {
    @Index(name = "idx_documents_application", columnList = "application_id"),
    @Index(name = "idx_documents_type", columnList = "document_type"),
    @Index(name = "idx_documents_status", columnList = "status")
})
@Getter
@Setter
@NoArgsConstructor
@AllArgsConstructor
@Builder
public class Document extends BaseEntity {

    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "application_id", nullable = false)
    private Application application;

    @NotNull(message = "Document type is required")
    @Enumerated(EnumType.STRING)
    @Column(name = "document_type", nullable = false, length = 50)
    private DocumentType documentType;

    @NotBlank(message = "File name is required")
    @Column(name = "file_name", nullable = false, length = 255)
    private String fileName;

    @NotBlank(message = "Original file name is required")
    @Column(name = "original_file_name", nullable = false, length = 255)
    private String originalFileName;

    @NotBlank(message = "File path is required")
    @Column(name = "file_path", nullable = false, length = 500)
    private String filePath;

    @NotNull(message = "File size is required")
    @Positive(message = "File size must be positive")
    @Column(name = "file_size", nullable = false)
    private Long fileSize;

    @NotBlank(message = "MIME type is required")
    @Column(name = "mime_type", nullable = false, length = 100)
    private String mimeType;

    @Column(length = 64)
    private String checksum;

    @Enumerated(EnumType.STRING)
    @Column(nullable = false, length = 50)
    @Builder.Default
    private DocumentStatus status = DocumentStatus.PENDING;

    @Column(name = "review_comments", columnDefinition = "TEXT")
    private String reviewComments;

    @Column(name = "uploaded_at")
    @Builder.Default
    private LocalDateTime uploadedAt = LocalDateTime.now();

    @Column(name = "reviewed_at")
    private LocalDateTime reviewedAt;

    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "reviewed_by")
    private User reviewedBy;

    // Helper Methods
    public void approve(User reviewer, String comments) {
        this.status = DocumentStatus.APPROVED;
        this.reviewedBy = reviewer;
        this.reviewComments = comments;
        this.reviewedAt = LocalDateTime.now();
    }

    public void reject(User reviewer, String comments) {
        this.status = DocumentStatus.REJECTED;
        this.reviewedBy = reviewer;
        this.reviewComments = comments;
        this.reviewedAt = LocalDateTime.now();
    }

    public void requestResubmission(User reviewer, String comments) {
        this.status = DocumentStatus.REQUIRES_RESUBMISSION;
        this.reviewedBy = reviewer;
        this.reviewComments = comments;
        this.reviewedAt = LocalDateTime.now();
    }

    public boolean isPending() {
        return this.status == DocumentStatus.PENDING;
    }

    public boolean isApproved() {
        return this.status == DocumentStatus.APPROVED;
    }

    public boolean isRejected() {
        return this.status == DocumentStatus.REJECTED;
    }

    public String getFileSizeFormatted() {
        if (fileSize < 1024) {
            return fileSize + " B";
        } else if (fileSize < 1024 * 1024) {
            return String.format("%.1f KB", fileSize / 1024.0);
        } else {
            return String.format("%.1f MB", fileSize / (1024.0 * 1024.0));
        }
    }

    public boolean isValidFileType() {
        return mimeType != null && (
            mimeType.equals("application/pdf") ||
            mimeType.equals("image/jpeg") ||
            mimeType.equals("image/png") ||
            mimeType.equals("image/jpg")
        );
    }

    public static final long MAX_FILE_SIZE = 15 * 1024 * 1024; // 15 MB

    public boolean isFileSizeValid() {
        return fileSize != null && fileSize <= MAX_FILE_SIZE;
    }
}
```

---

## 8. WORKFLOW HISTORY ENTITY

```java
package com.iztech.utms.entity;

import com.iztech.utms.enums.WorkflowStage;
import jakarta.persistence.*;
import jakarta.validation.constraints.NotBlank;
import jakarta.validation.constraints.NotNull;
import lombok.*;
import org.hibernate.annotations.JdbcTypeCode;
import org.hibernate.type.SqlTypes;

import java.time.LocalDateTime;
import java.util.Map;

@Entity
@Table(name = "workflow_history", indexes = {
    @Index(name = "idx_workflow_application", columnList = "application_id"),
    @Index(name = "idx_workflow_performed_by", columnList = "performed_by"),
    @Index(name = "idx_workflow_created", columnList = "created_at")
})
@Getter
@Setter
@NoArgsConstructor
@AllArgsConstructor
@Builder
public class WorkflowHistory {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "application_id", nullable = false)
    private Application application;

    @Enumerated(EnumType.STRING)
    @Column(name = "from_stage", length = 50)
    private WorkflowStage fromStage;

    @NotNull(message = "To stage is required")
    @Enumerated(EnumType.STRING)
    @Column(name = "to_stage", nullable = false, length = 50)
    private WorkflowStage toStage;

    @NotBlank(message = "Action is required")
    @Column(nullable = false, length = 100)
    private String action;

    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "performed_by")
    private User performedBy;

    @Column(columnDefinition = "TEXT")
    private String notes;

    @JdbcTypeCode(SqlTypes.JSON)
    @Column(columnDefinition = "jsonb")
    private Map<String, Object> metadata;

    @Column(name = "created_at", nullable = false, updatable = false)
    @Builder.Default
    private LocalDateTime createdAt = LocalDateTime.now();

    // Helper Methods
    public boolean isTransition() {
        return fromStage != null && !fromStage.equals(toStage);
    }

    public String getActionDescription() {
        return switch (action) {
            case "APPLICATION_SUBMITTED" -> "Başvuru gönderildi";
            case "APPROVE_AND_FORWARD" -> "Onaylandı ve iletildi";
            case "REJECT" -> "Reddedildi";
            case "REQUEST_DOCUMENTS" -> "Belge talep edildi";
            case "RETURN_TO_PREVIOUS" -> "Önceki aşamaya iade edildi";
            case "LANGUAGE_APPROVED" -> "Dil yeterliliği onaylandı";
            case "LANGUAGE_REJECTED" -> "Dil yeterliliği reddedildi";
            case "INTIBAK_COMPLETED" -> "İntibak tamamlandı";
            case "FINAL_DECISION" -> "Nihai karar verildi";
            default -> action;
        };
    }

    // Factory Methods
    public static WorkflowHistory createSubmissionEntry(Application application, User submitter) {
        return WorkflowHistory.builder()
            .application(application)
            .fromStage(null)
            .toStage(WorkflowStage.OIDB_INITIAL)
            .action("APPLICATION_SUBMITTED")
            .performedBy(submitter)
            .build();
    }

    public static WorkflowHistory createTransferEntry(
            Application application,
            WorkflowStage from,
            WorkflowStage to,
            String action,
            User performer,
            String notes) {
        return WorkflowHistory.builder()
            .application(application)
            .fromStage(from)
            .toStage(to)
            .action(action)
            .performedBy(performer)
            .notes(notes)
            .build();
    }
}
```

---

## 9. INTIBAK RECORD ENTITY

```java
package com.iztech.utms.entity;

import com.iztech.utms.enums.IntibakDecision;
import jakarta.persistence.*;
import jakarta.validation.constraints.NotBlank;
import jakarta.validation.constraints.NotNull;
import jakarta.validation.constraints.Positive;
import lombok.*;

import java.math.BigDecimal;

@Entity
@Table(name = "intibak_records", indexes = {
    @Index(name = "idx_intibak_application", columnList = "application_id"),
    @Index(name = "idx_intibak_decision", columnList = "decision")
})
@Getter
@Setter
@NoArgsConstructor
@AllArgsConstructor
@Builder
public class IntibakRecord extends BaseEntity {

    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "application_id", nullable = false)
    private Application application;

    // Source Course Information
    @NotBlank(message = "Source course code is required")
    @Column(name = "source_course_code", nullable = false, length = 50)
    private String sourceCourseCode;

    @NotBlank(message = "Source course name is required")
    @Column(name = "source_course_name", nullable = false, length = 255)
    private String sourceCourseName;

    @NotNull(message = "Source credits is required")
    @Positive(message = "Source credits must be positive")
    @Column(name = "source_credits", nullable = false)
    private Integer sourceCredits;

    @Column(name = "source_ects", precision = 4, scale = 1)
    private BigDecimal sourceEcts;

    @NotBlank(message = "Source grade is required")
    @Column(name = "source_grade", nullable = false, length = 10)
    private String sourceGrade;

    @Column(name = "source_grade_point", precision = 3, scale = 2)
    private BigDecimal sourceGradePoint;

    // Target Course Information
    @Column(name = "target_course_code", length = 50)
    private String targetCourseCode;

    @Column(name = "target_course_name", length = 255)
    private String targetCourseName;

    @Column(name = "target_credits")
    private Integer targetCredits;

    @Column(name = "target_ects", precision = 4, scale = 1)
    private BigDecimal targetEcts;

    // Decision
    @NotNull(message = "Decision is required")
    @Enumerated(EnumType.STRING)
    @Column(nullable = false, length = 50)
    @Builder.Default
    private IntibakDecision decision = IntibakDecision.PENDING;

    @Column(name = "decision_notes", columnDefinition = "TEXT")
    private String decisionNotes;

    // Metadata
    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "prepared_by")
    private User preparedBy;

    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "approved_by")
    private User approvedBy;

    // Helper Methods
    public void accept(User preparer) {
        this.decision = IntibakDecision.ACCEPTED;
        this.preparedBy = preparer;
    }

    public void reject(User preparer, String notes) {
        this.decision = IntibakDecision.REJECTED;
        this.preparedBy = preparer;
        this.decisionNotes = notes;
    }

    public void setConditional(User preparer, String notes) {
        this.decision = IntibakDecision.CONDITIONAL;
        this.preparedBy = preparer;
        this.decisionNotes = notes;
    }

    public boolean isAccepted() {
        return this.decision == IntibakDecision.ACCEPTED;
    }

    public boolean isPending() {
        return this.decision == IntibakDecision.PENDING;
    }

    public boolean hasTargetCourse() {
        return targetCourseCode != null && !targetCourseCode.isBlank();
    }

    public int getAcceptedCredits() {
        if (isAccepted() && targetCredits != null) {
            return targetCredits;
        }
        return 0;
    }
}
```

---

## 10. NOTIFICATION ENTITY

```java
package com.iztech.utms.entity;

import com.iztech.utms.enums.NotificationPriority;
import com.iztech.utms.enums.NotificationType;
import jakarta.persistence.*;
import jakarta.validation.constraints.NotBlank;
import jakarta.validation.constraints.NotNull;
import lombok.*;

import java.time.LocalDateTime;

@Entity
@Table(name = "notifications", indexes = {
    @Index(name = "idx_notifications_user", columnList = "user_id"),
    @Index(name = "idx_notifications_application", columnList = "application_id"),
    @Index(name = "idx_notifications_unread", columnList = "user_id, is_read"),
    @Index(name = "idx_notifications_created", columnList = "created_at")
})
@Getter
@Setter
@NoArgsConstructor
@AllArgsConstructor
@Builder
public class Notification {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "user_id", nullable = false)
    private User user;

    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "application_id")
    private Application application;

    @NotNull(message = "Notification type is required")
    @Enumerated(EnumType.STRING)
    @Column(nullable = false, length = 50)
    private NotificationType type;

    @NotBlank(message = "Title is required")
    @Column(nullable = false, length = 255)
    private String title;

    @NotBlank(message = "Message is required")
    @Column(nullable = false, columnDefinition = "TEXT")
    private String message;

    @Enumerated(EnumType.STRING)
    @Column(length = 20)
    @Builder.Default
    private NotificationPriority priority = NotificationPriority.NORMAL;

    @Column(name = "is_read")
    @Builder.Default
    private Boolean isRead = false;

    @Column(name = "read_at")
    private LocalDateTime readAt;

    @Column(name = "action_url", length = 500)
    private String actionUrl;

    @Column(name = "created_at", nullable = false, updatable = false)
    @Builder.Default
    private LocalDateTime createdAt = LocalDateTime.now();

    // Helper Methods
    public void markAsRead() {
        if (!this.isRead) {
            this.isRead = true;
            this.readAt = LocalDateTime.now();
        }
    }

    public boolean isUnread() {
        return !this.isRead;
    }

    // Factory Methods
    public static Notification createApplicationSubmitted(User user, Application application) {
        return Notification.builder()
            .user(user)
            .application(application)
            .type(NotificationType.APPLICATION_SUBMITTED)
            .title("Başvurunuz Alındı")
            .message("Başvurunuz başarıyla alınmıştır. Başvuru numaranız: " + application.getApplicationNumber())
            .priority(NotificationPriority.HIGH)
            .actionUrl("/applications/" + application.getId())
            .build();
    }

    public static Notification createDocumentApproved(User user, Application application, String documentType) {
        return Notification.builder()
            .user(user)
            .application(application)
            .type(NotificationType.DOCUMENT_APPROVED)
            .title("Belge Onaylandı")
            .message(documentType + " belgeniz onaylanmıştır.")
            .actionUrl("/applications/" + application.getId() + "/documents")
            .build();
    }

    public static Notification createDocumentRejected(User user, Application application, String documentType, String reason) {
        return Notification.builder()
            .user(user)
            .application(application)
            .type(NotificationType.DOCUMENT_REJECTED)
            .title("Belge Reddedildi")
            .message(documentType + " belgeniz reddedilmiştir. Sebep: " + reason)
            .priority(NotificationPriority.HIGH)
            .actionUrl("/applications/" + application.getId() + "/documents")
            .build();
    }

    public static Notification createStatusChanged(User user, Application application, String newStatus) {
        return Notification.builder()
            .user(user)
            .application(application)
            .type(NotificationType.STATUS_CHANGED)
            .title("Başvuru Durumu Güncellendi")
            .message("Başvurunuzun durumu güncellendi: " + newStatus)
            .actionUrl("/applications/" + application.getId() + "/status")
            .build();
    }

    public static Notification createFinalDecision(User user, Application application, boolean accepted) {
        String title = accepted ? "Tebrikler! Başvurunuz Kabul Edildi" : "Başvuru Sonucu";
        String message = accepted 
            ? "Yatay geçiş başvurunuz kabul edilmiştir. Kayıt işlemleri için ÖİDB ile iletişime geçiniz."
            : "Yatay geçiş başvurunuz değerlendirme sonucunda kabul edilememiştir.";
        
        return Notification.builder()
            .user(user)
            .application(application)
            .type(NotificationType.FINAL_DECISION)
            .title(title)
            .message(message)
            .priority(NotificationPriority.URGENT)
            .actionUrl("/applications/" + application.getId())
            .build();
    }
}
```

---

## 11. AUDIT LOG ENTITY

```java
package com.iztech.utms.entity;

import jakarta.persistence.*;
import jakarta.validation.constraints.NotBlank;
import lombok.*;
import org.hibernate.annotations.JdbcTypeCode;
import org.hibernate.type.SqlTypes;

import java.time.LocalDateTime;
import java.util.Map;

@Entity
@Table(name = "audit_logs", indexes = {
    @Index(name = "idx_audit_user", columnList = "user_id"),
    @Index(name = "idx_audit_entity", columnList = "entity_type, entity_id"),
    @Index(name = "idx_audit_action", columnList = "action"),
    @Index(name = "idx_audit_created", columnList = "created_at")
})
@Getter
@Setter
@NoArgsConstructor
@AllArgsConstructor
@Builder
public class AuditLog {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "user_id")
    private User user;

    @NotBlank(message = "Action is required")
    @Column(nullable = false, length = 100)
    private String action;

    @NotBlank(message = "Entity type is required")
    @Column(name = "entity_type", nullable = false, length = 100)
    private String entityType;

    @Column(name = "entity_id")
    private Long entityId;

    @JdbcTypeCode(SqlTypes.JSON)
    @Column(name = "old_value", columnDefinition = "jsonb")
    private Map<String, Object> oldValue;

    @JdbcTypeCode(SqlTypes.JSON)
    @Column(name = "new_value", columnDefinition = "jsonb")
    private Map<String, Object> newValue;

    @Column(name = "ip_address", length = 50)
    private String ipAddress;

    @Column(name = "user_agent", length = 500)
    private String userAgent;

    @Column(name = "session_id", length = 100)
    private String sessionId;

    @Column(name = "created_at", nullable = false, updatable = false)
    @Builder.Default
    private LocalDateTime createdAt = LocalDateTime.now();

    // Factory Methods
    public static AuditLog createEntry(
            User user,
            String action,
            String entityType,
            Long entityId,
            Map<String, Object> oldValue,
            Map<String, Object> newValue,
            String ipAddress,
            String userAgent) {
        
        return AuditLog.builder()
            .user(user)
            .action(action)
            .entityType(entityType)
            .entityId(entityId)
            .oldValue(oldValue)
            .newValue(newValue)
            .ipAddress(ipAddress)
            .userAgent(userAgent)
            .build();
    }

    public static AuditLog createLoginEntry(User user, String ipAddress, String userAgent) {
        return AuditLog.builder()
            .user(user)
            .action("LOGIN")
            .entityType("User")
            .entityId(user.getId())
            .ipAddress(ipAddress)
            .userAgent(userAgent)
            .build();
    }

    public static AuditLog createLogoutEntry(User user, String ipAddress) {
        return AuditLog.builder()
            .user(user)
            .action("LOGOUT")
            .entityType("User")
            .entityId(user.getId())
            .ipAddress(ipAddress)
            .build();
    }
}
```

---

## 12. PASSWORD RESET TOKEN ENTITY

```java
package com.iztech.utms.entity;

import jakarta.persistence.*;
import jakarta.validation.constraints.NotBlank;
import jakarta.validation.constraints.NotNull;
import lombok.*;

import java.time.LocalDateTime;
import java.util.UUID;

@Entity
@Table(name = "password_reset_tokens", indexes = {
    @Index(name = "idx_reset_tokens_user", columnList = "user_id"),
    @Index(name = "idx_reset_tokens_token", columnList = "token"),
    @Index(name = "idx_reset_tokens_expires", columnList = "expires_at")
})
@Getter
@Setter
@NoArgsConstructor
@AllArgsConstructor
@Builder
public class PasswordResetToken {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "user_id", nullable = false)
    private User user;

    @NotBlank(message = "Token is required")
    @Column(nullable = false, unique = true, length = 255)
    private String token;

    @NotNull(message = "Expiry date is required")
    @Column(name = "expires_at", nullable = false)
    private LocalDateTime expiresAt;

    @Column
    @Builder.Default
    private Boolean used = false;

    @Column(name = "used_at")
    private LocalDateTime usedAt;

    @Column(name = "created_at", nullable = false, updatable = false)
    @Builder.Default
    private LocalDateTime createdAt = LocalDateTime.now();

    // Helper Methods
    public boolean isExpired() {
        return LocalDateTime.now().isAfter(expiresAt);
    }

    public boolean isValid() {
        return !used && !isExpired();
    }

    public void markAsUsed() {
        this.used = true;
        this.usedAt = LocalDateTime.now();
    }

    // Factory Methods
    public static PasswordResetToken create(User user, int expirationMinutes) {
        return PasswordResetToken.builder()
            .user(user)
            .token(UUID.randomUUID().toString())
            .expiresAt(LocalDateTime.now().plusMinutes(expirationMinutes))
            .build();
    }

    public static final int DEFAULT_EXPIRATION_MINUTES = 60;

    public static PasswordResetToken createDefault(User user) {
        return create(user, DEFAULT_EXPIRATION_MINUTES);
    }
}
```

---

## 13. REVISION HISTORY

| Version | Date | Author | Changes |
|---------|------|--------|---------|
| 1.0 | Mar 2026 | Group 05 | Initial version |
