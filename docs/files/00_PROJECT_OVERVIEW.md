# UTMS - Undergraduate Transfer Management System
## 00 - Project Overview

**Version:** 1.0  
**Last Updated:** March 2026  
**Organization:** İzmir Institute of Technology (IZTECH)  
**Team:** Group 05

---

## 1. EXECUTIVE SUMMARY

UTMS (Undergraduate Transfer Management System) is a web-based platform designed to digitalize and streamline the horizontal transfer (yatay geçiş) application process at İzmir Institute of Technology (IZTECH). The system replaces manual, paper-based processes with an automated digital workflow.

---

## 2. PROBLEM STATEMENT

### 2.1 Current Process Issues (AS-IS)

| Problem | Impact | Frequency |
|---------|--------|-----------|
| Manual data verification from YÖKSİS/ÖSYM | Significant delays (days) | Every application |
| Manual eligibility checks | Human errors, inconsistency | Every application |
| Spreadsheet-based scoring/ranking | Error-prone, inefficient | Per evaluation period |
| Email/physical document transfer | Lost documents, delays | Multiple times per app |
| No real-time tracking | Student uncertainty, support load | Continuous |
| Phone-based feedback | Inefficient, no audit trail | Per application |

### 2.2 Stakeholder Pain Points

**Students:**
- Cannot track application progress
- Uncertain about requirements
- No visibility into decision process
- Long waiting times

**OIDB Staff:**
- Manual document verification
- Repetitive data entry
- Paper-based workflows
- High error rates

**YGK Members:**
- Manual intibak preparation
- No standardized interface
- Difficult coordination

---

## 3. SOLUTION (TO-BE)

### 3.1 Key Features

| Feature | Description | Benefit |
|---------|-------------|---------|
| Online Application | Digital submission portal | 24/7 access, no physical visit |
| Automated Verification | Integration with YÖKSİS/ÖSYM | Instant validation, no manual entry |
| Rule-based Eligibility | Automatic checks against criteria | Consistent, error-free evaluation |
| Auto Scoring/Ranking | Formula-based calculation | Fair, transparent rankings |
| Digital Workflow | Electronic routing between units | Fast, traceable transfers |
| Real-time Tracking | Student dashboard with status | Transparency, reduced inquiries |
| Automated Notifications | Email/in-app alerts | Timely communication |

### 3.2 Process Improvements

```
AS-IS Process Time: ~4-6 weeks
TO-BE Process Time: ~1-2 weeks (60-75% reduction)

AS-IS Error Rate: ~15-20%
TO-BE Error Rate: <2% (90% reduction)

AS-IS Manual Steps: 25+
TO-BE Manual Steps: 5 (intibak only)
```

---

## 4. SYSTEM BOUNDARIES

### 4.1 In Scope

- ✅ Digital application submission
- ✅ Document upload and management
- ✅ Automated data validation via external APIs
- ✅ Rule-based eligibility checking
- ✅ Automatic scoring and ranking
- ✅ Digital workflow routing
- ✅ Real-time status tracking
- ✅ Automated notifications
- ✅ Manual intibak interface (digital workspace)
- ✅ Result publication

### 4.2 Out of Scope

- ❌ Replacing academic judgment in intibak
- ❌ Non-IZTECH transfer applications
- ❌ Automatic course equivalence decisions
- ❌ Mobile native applications (v1.0)
- ❌ Digital signature (e-imza) integration (v1.0)

---

## 5. STAKEHOLDERS

### 5.1 Primary Stakeholders

| Stakeholder | Role | Interest Level |
|-------------|------|----------------|
| Applicant Students | End users | High |
| OIDB Staff | Administrators | High |
| Dean's Office Staff | Reviewers | High |
| YGK Members | Evaluators | High |
| YDYO Staff | Language evaluators | Medium |

### 5.2 Secondary Stakeholders

| Stakeholder | Role | Interest Level |
|-------------|------|----------------|
| IT Department | Technical support | Medium |
| University Administration | Oversight | Low |
| YÖK | Regulatory compliance | Low |

---

## 6. SUCCESS CRITERIA

### 6.1 Functional Success

| Metric | Target | Measurement |
|--------|--------|-------------|
| Application completion rate | >95% | Submitted/Started |
| Document rejection rate | <5% | Rejected/Uploaded |
| Workflow completion time | <14 days | Average processing time |
| System availability | 99% | Uptime during application period |

### 6.2 User Satisfaction

| Metric | Target | Measurement |
|--------|--------|-------------|
| Student satisfaction | >4.0/5.0 | Post-process survey |
| Staff efficiency | +50% | Applications processed per day |
| Support ticket reduction | -70% | Tickets vs previous year |

---

## 7. CONSTRAINTS

### 7.1 Regulatory Constraints

- Must comply with YÖK horizontal transfer regulations
- Must comply with KVKK (Personal Data Protection Law)
- Must follow IZTECH internal procedures

### 7.2 Technical Constraints

- Must integrate with existing UBYS system
- Must support all major browsers (Chrome, Firefox, Safari, Edge)
- Must be accessible on mobile devices (responsive design)

### 7.3 Operational Constraints

- Intibak decisions must remain manual (regulatory requirement)
- Application period dates set by YÖK calendar
- Quota limits set by faculties

---

## 8. ASSUMPTIONS

| ID | Assumption | Risk if False |
|----|------------|---------------|
| A1 | YÖKSİS/ÖSYM APIs remain available | Cannot validate data |
| A2 | Students have internet access | Cannot submit applications |
| A3 | Students have digital documents | Cannot complete applications |
| A4 | Staff trained before go-live | System underutilization |
| A5 | UBYS integration supported | Cannot publish results |

---

## 9. DEPENDENCIES

| ID | Dependency | Type | Owner |
|----|------------|------|-------|
| D1 | YÖKSİS API access | External | YÖK |
| D2 | ÖSYM API access | External | ÖSYM |
| D3 | UBYS integration endpoint | Internal | IT Dept |
| D4 | University network infrastructure | Internal | IT Dept |
| D5 | Email server (SMTP) | Internal | IT Dept |

---

## 10. PROJECT TIMELINE

### 10.1 Phases

| Phase | Duration | Deliverables |
|-------|----------|--------------|
| Phase 1: Analysis | 4 weeks | Requirements, Design docs |
| Phase 2: Development | 12 weeks | Working system |
| Phase 3: Testing | 4 weeks | Test reports, bug fixes |
| Phase 4: Deployment | 2 weeks | Production system |
| Phase 5: Support | Ongoing | Maintenance, updates |

### 10.2 Milestones

| Milestone | Date | Criteria |
|-----------|------|----------|
| M1: Requirements Complete | Week 4 | SRS approved |
| M2: Backend Complete | Week 10 | APIs functional |
| M3: Frontend Complete | Week 14 | UI complete |
| M4: UAT Complete | Week 18 | User acceptance |
| M5: Go-Live | Week 20 | Production deployment |

---

## 11. GLOSSARY

| Term | Definition |
|------|------------|
| **UTMS** | Undergraduate Transfer Management System |
| **IZTECH/İYTE** | İzmir Institute of Technology |
| **ÖİDB** | Registrar's Office (Öğrenci İşleri Daire Başkanlığı) |
| **YDYO** | School of Foreign Languages (Yabancı Diller Yüksekokulu) |
| **YGK** | Transfer Committee (Yatay Geçiş Komisyonu) |
| **UBYS** | University Student Information System |
| **YÖKSİS** | Higher Education Council Information System |
| **ÖSYM** | Student Selection and Placement Center |
| **YKS** | Higher Education Institutions Exam |
| **Intibak** | Course equivalence evaluation (manual process) |
| **Asil** | Primary/accepted candidate |
| **Yedek** | Waitlist/reserve candidate |
| **KVKK** | Turkish Personal Data Protection Law |
| **Yatay Geçiş** | Horizontal/lateral transfer between universities |

---

## 12. DOCUMENT REFERENCES

| ID | Document | Version | Date |
|----|----------|---------|------|
| REF-01 | Problem Analysis Report | 1.0 | Oct 2025 |
| REF-02 | Software Requirements Specification | 1.0 | Nov 2025 |
| REF-03 | İYTE Kurumlar Arası Yatay Geçiş Yönergesi | - | Dec 2023 |
| REF-04 | IEEE Std 830 | - | - |
| REF-05 | ISO/IEC 25010:2011 | - | - |

---

## 13. REVISION HISTORY

| Version | Date | Author | Changes |
|---------|------|--------|---------|
| 1.0 | Mar 2026 | Group 05 | Initial version |
