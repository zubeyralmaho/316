# Documentation Gap Analysis Report

Based on a comprehensive review of the markdown files in `docs/files`, several logical gaps, missing endpoints, and inconsistencies have been identified across the documentation.

## 1. Missing API Endpoints (`03_API_SPECIFICATION.md`)

Several endpoints are missing from the API specifications, despite the existence of corresponding DTOs, Controllers, and Frontend Pages in other documents.

**Missing Authentication Endpoints:**
- While `ForgotPasswordRequest`, `ResetPasswordRequest`, and `ChangePasswordRequest` DTOs are defined in `05_BACKEND_ENUMS_DTOS.md`, there are no corresponding endpoints for these actions in the API specification (e.g., `POST /auth/forgot-password`, `POST /auth/reset-password`, `POST /auth/change-password`).
- The `PasswordResetToken` database table and backend entity exist to support these missing operations.

**Missing Application Endpoints:**
- A `WithdrawRequest` DTO is defined in `05_BACKEND_ENUMS_DTOS.md`, and `WITHDRAWN` is a valid application status, but there is no endpoint defined to withdraw an application (e.g., `POST /applications/{id}/withdraw`).

**Missing Admin Endpoints:**
- `01_SYSTEM_ARCHITECTURE.md` explicitly lists an `AdminController`.
- `08_FRONTEND_ARCHITECTURE.md` lists admin pages such as `UsersPage.tsx`, `DepartmentsPage.tsx`, `ReportsPage.tsx`, and `SettingsPage.tsx`.
- However, `03_API_SPECIFICATION.md` only documents `GET /admin/dashboard` and `GET /admin/audit-logs`.
- **Gaps:** Missing CRUD operations for Users, Departments, and Faculties. Missing endpoints for generating/downloading reports. Missing endpoints for system settings.

## 2. Inconsistencies in Database Schema (`02_DATABASE_SCHEMA.md`)

There are discrepancies between the ER diagram and the actual SQL table definitions/backend entities.

**`intibak_records` Table Discrepancies:**
- The ER diagram in Section 1 is missing several important fields that are present in the SQL definition (Section 2.7) and the `IntibakRecord` entity (`04_BACKEND_ENTITIES.md`).
- **Missing from ER Diagram:** `source_ects`, `source_grade_point`, `target_ects`, `decision_notes`, `approved_by`.
- **Truncated in ER Diagram:** Fields like `source_course_code` and `source_course_name` are truncated to `source_course_co` and `source_course_na`.

## 3. Summary

To ensure the documentation is fully consistent:
1. Update `03_API_SPECIFICATION.md` to include endpoints for forgotten password, password reset, application withdrawal, admin CRUD operations (users/departments), and reports.
2. Update the ER diagram diagram in `02_DATABASE_SCHEMA.md` to reflect all fields present in the `intibak_records` SQL table.
