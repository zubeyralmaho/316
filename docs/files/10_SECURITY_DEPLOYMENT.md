# UTMS - Undergraduate Transfer Management System
## 10 - Security & Deployment Guide

**Version:** 1.0  
**Last Updated:** March 2026

---

# PART A: SECURITY

## 1. AUTHENTICATION

### 1.1 JWT-Based Authentication

**Token Types:**
| Token | Purpose | Expiry | Storage |
|-------|---------|--------|---------|
| Access Token | API authorization | 30 minutes | Memory |
| Refresh Token | Obtain new access token | 7 days | HttpOnly Cookie |

**Access Token Structure:**
```
Header:
{
  "alg": "HS256",
  "typ": "JWT"
}

Payload:
{
  "sub": "userId",
  "email": "user@example.com",
  "role": "STUDENT",
  "iat": 1234567890,
  "exp": 1234569690
}
```

**Security Measures:**
- Tokens signed with strong secret (256-bit minimum)
- Short access token lifetime (30 min)
- Refresh tokens are single-use
- Token blacklisting on logout
- Secure cookie flags for refresh token

### 1.2 Password Security

**Hashing:**
- Algorithm: BCrypt
- Work factor: 12 rounds
- Salt: Auto-generated per password

**Password Requirements:**
| Requirement | Value |
|-------------|-------|
| Minimum length | 8 characters |
| Uppercase | At least 1 |
| Lowercase | At least 1 |
| Digit | At least 1 |
| Special character | At least 1 |
| Password history | Cannot reuse last 5 |

**Account Protection:**
| Measure | Configuration |
|---------|---------------|
| Failed login threshold | 5 attempts |
| Account lockout | Automatic |
| Lockout duration | Until password reset |
| Login delay | Exponential backoff |

### 1.3 Session Management

**Session Security:**
- Session ID regeneration on login
- Session invalidation on logout
- Concurrent session limit (configurable)
- Idle timeout: 30 minutes
- Absolute timeout: 8 hours

---

## 2. AUTHORIZATION

### 2.1 Role-Based Access Control (RBAC)

**Role Hierarchy:**
```
SYSTEM_ADMIN
    └── Full system access

OIDB_STAFF
    └── Document review, workflow management, rankings

DEAN_STAFF
    └── Academic review, final decisions

YGK_MEMBER
    └── Intibak preparation, rankings

YDYO_STAFF
    └── Language evaluation

STUDENT
    └── Own application management only
```

### 2.2 Permission Matrix

| Permission | STUDENT | OIDB | DEAN | YGK | YDYO | ADMIN |
|------------|---------|------|------|-----|------|-------|
| View own application | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ |
| Create application | ✓ | - | - | - | - | ✓ |
| View all applications | - | ✓ | ✓ | ✓ | ✓ | ✓ |
| Review documents | - | ✓ | - | - | - | ✓ |
| Evaluate language | - | - | - | - | ✓ | ✓ |
| Prepare intibak | - | - | - | ✓ | - | ✓ |
| Final decision | - | - | ✓ | - | - | ✓ |
| Transfer workflow | - | ✓ | ✓ | ✓ | ✓ | ✓ |
| Manage users | - | - | - | - | - | ✓ |
| System configuration | - | - | - | - | - | ✓ |

### 2.3 Resource-Level Authorization

**Application Access Rules:**
```
IF user.role == STUDENT:
    ALLOW access IF application.userId == user.id
    
IF user.role == OIDB_STAFF:
    ALLOW access IF application.currentStage IN [OIDB_INITIAL, OIDB_POST_LANGUAGE, OIDB_FINAL]
    
IF user.role == YDYO_STAFF:
    ALLOW access IF application.currentStage == YDYO_EVALUATION
    
IF user.role == DEAN_STAFF:
    ALLOW access IF application.currentStage IN [DEANS_OFFICE_REVIEW, DEANS_OFFICE_FINAL]
    
IF user.role == YGK_MEMBER:
    ALLOW access IF application.currentStage == YGK_EVALUATION
    
IF user.role == SYSTEM_ADMIN:
    ALLOW all access
```

---

## 3. DATA PROTECTION

### 3.1 Data Classification

| Classification | Examples | Protection Level |
|----------------|----------|------------------|
| Public | Department info, quotas | None required |
| Internal | Application statistics | Authentication required |
| Confidential | Personal data, grades | Role-based access |
| Restricted | Passwords, tokens | Encryption required |

### 3.2 Encryption

**Data at Rest:**
| Data | Encryption | Key Management |
|------|------------|----------------|
| Database | AES-256 | AWS KMS / Vault |
| Documents | AES-256 | Per-application key |
| Backups | AES-256 | Separate backup key |

**Data in Transit:**
| Protocol | Version | Configuration |
|----------|---------|---------------|
| TLS | 1.2+ minimum | Strong ciphers only |
| HTTPS | Required | HSTS enabled |

### 3.3 Personal Data (KVKK Compliance)

**KVKK Requirements:**
- Explicit consent required before data collection
- Purpose limitation for data processing
- Data minimization principle
- Right to access personal data
- Right to request deletion
- Data breach notification (72 hours)

**Implementation:**
```
User Registration:
├── Display KVKK information text
├── Require explicit checkbox consent
├── Record consent timestamp
├── Store consent record separately
└── Allow consent withdrawal

Data Requests:
├── Provide data export functionality
├── Allow account deletion request
├── Anonymize data on deletion
└── Retain audit logs (anonymized)
```

---

## 4. INPUT VALIDATION

### 4.1 Validation Strategy

**Defense in Depth:**
```
Client-side validation (UX)
    └── React Hook Form + Zod schemas
    
Server-side validation (Security)
    └── Jakarta Validation + Custom validators
    
Database constraints (Integrity)
    └── CHECK constraints + Foreign keys
```

### 4.2 Common Validation Rules

| Input Type | Validation | Sanitization |
|------------|------------|--------------|
| Email | RFC 5322 format | Lowercase, trim |
| Phone | Turkish format | Digits only |
| Text | Length limits | HTML escape |
| Numbers | Range checks | Type coercion |
| Files | Type + size | Filename sanitization |

### 4.3 Protection Against

| Attack | Protection |
|--------|------------|
| SQL Injection | Parameterized queries (JPA) |
| XSS | Content Security Policy, output encoding |
| CSRF | CSRF tokens, SameSite cookies |
| File Upload | Type validation, virus scan |
| Path Traversal | Filename sanitization |

---

## 5. API SECURITY

### 5.1 Rate Limiting

| Endpoint Type | Limit | Window |
|---------------|-------|--------|
| Authentication | 5 requests | 1 minute |
| General API | 100 requests | 1 minute |
| File upload | 10 requests | 1 minute |
| Search | 30 requests | 1 minute |

### 5.2 Request Validation

**Headers Required:**
```
Content-Type: application/json
Accept: application/json
Authorization: Bearer <token>
X-Request-ID: <uuid>
```

**CORS Configuration:**
```
Allowed Origins: [frontend domain only]
Allowed Methods: GET, POST, PUT, DELETE, PATCH
Allowed Headers: Content-Type, Authorization, X-Request-ID
Credentials: true
Max Age: 3600
```

### 5.3 Response Security Headers

```
Strict-Transport-Security: max-age=31536000; includeSubDomains
X-Content-Type-Options: nosniff
X-Frame-Options: DENY
X-XSS-Protection: 1; mode=block
Content-Security-Policy: default-src 'self'
Referrer-Policy: strict-origin-when-cross-origin
```

---

## 6. AUDIT LOGGING

### 6.1 Events to Log

| Event Category | Events |
|----------------|--------|
| Authentication | Login, logout, failed login, password reset |
| Authorization | Access denied, role change |
| Data Access | View, create, update, delete |
| Workflow | Status change, stage transfer |
| Administration | User management, configuration |

### 6.2 Log Format

```
{
  "timestamp": "2026-03-13T10:30:00Z",
  "level": "INFO",
  "event": "LOGIN_SUCCESS",
  "userId": 123,
  "userEmail": "user@example.com",
  "ipAddress": "192.168.1.100",
  "userAgent": "Mozilla/5.0...",
  "sessionId": "abc123",
  "details": {
    "method": "password"
  }
}
```

### 6.3 Log Retention

| Log Type | Retention | Storage |
|----------|-----------|---------|
| Security events | 7 years | Immutable storage |
| Audit trail | 5 years | Database + archive |
| Application logs | 90 days | Log aggregator |
| Debug logs | 7 days | Local + rotation |

---

# PART B: DEPLOYMENT

## 7. INFRASTRUCTURE

### 7.1 Architecture Overview

```
┌─────────────────────────────────────────────────────────────┐
│                        INTERNET                              │
└───────────────────────────┬─────────────────────────────────┘
                            │
                    ┌───────▼───────┐
                    │   Firewall    │
                    │   (WAF)       │
                    └───────┬───────┘
                            │
                    ┌───────▼───────┐
                    │ Load Balancer │
                    │   (Nginx)     │
                    └───────┬───────┘
                            │
            ┌───────────────┼───────────────┐
            │               │               │
    ┌───────▼───────┐ ┌─────▼─────┐ ┌───────▼───────┐
    │  Frontend     │ │ Backend   │ │  Backend      │
    │  (Static)     │ │ Instance 1│ │  Instance 2   │
    │  CDN/Nginx    │ │ (Docker)  │ │  (Docker)     │
    └───────────────┘ └─────┬─────┘ └───────┬───────┘
                            │               │
                    ┌───────┴───────────────┴───────┐
                    │                               │
            ┌───────▼───────┐           ┌───────────▼─────┐
            │  PostgreSQL   │           │  File Storage   │
            │  Primary      │           │  (NFS/S3)       │
            └───────┬───────┘           └─────────────────┘
                    │
            ┌───────▼───────┐
            │  PostgreSQL   │
            │  Replica      │
            └───────────────┘
```

### 7.2 Server Requirements

**Application Server (per instance):**
| Resource | Minimum | Recommended |
|----------|---------|-------------|
| CPU | 2 cores | 4 cores |
| RAM | 4 GB | 8 GB |
| Storage | 20 GB SSD | 50 GB SSD |
| Network | 100 Mbps | 1 Gbps |

**Database Server:**
| Resource | Minimum | Recommended |
|----------|---------|-------------|
| CPU | 2 cores | 4 cores |
| RAM | 8 GB | 16 GB |
| Storage | 100 GB SSD | 500 GB SSD |
| IOPS | 3000 | 10000 |

### 7.3 Network Configuration

**Ports:**
| Service | Port | Access |
|---------|------|--------|
| HTTPS | 443 | Public |
| HTTP (redirect) | 80 | Public |
| Application | 8080 | Internal |
| PostgreSQL | 5432 | Internal |
| SSH | 22 | Admin VPN only |

---

## 8. DOCKER CONFIGURATION

### 8.1 Backend Dockerfile

```dockerfile
# Build stage
FROM eclipse-temurin:17-jdk-alpine AS build
WORKDIR /app
COPY pom.xml .
COPY src ./src
RUN ./mvnw clean package -DskipTests

# Runtime stage
FROM eclipse-temurin:17-jre-alpine
WORKDIR /app

# Security: Run as non-root user
RUN addgroup -S appgroup && adduser -S appuser -G appgroup
USER appuser

COPY --from=build /app/target/*.jar app.jar

# Health check
HEALTHCHECK --interval=30s --timeout=10s --retries=3 \
  CMD wget -q --spider http://localhost:8080/actuator/health || exit 1

EXPOSE 8080
ENTRYPOINT ["java", "-jar", "app.jar"]
```

### 8.2 Frontend Dockerfile

```dockerfile
# Build stage
FROM node:20-alpine AS build
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build

# Runtime stage
FROM nginx:alpine
COPY --from=build /app/dist /usr/share/nginx/html
COPY nginx.conf /etc/nginx/conf.d/default.conf
EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
```

### 8.3 Docker Compose (Production)

```yaml
version: '3.8'

services:
  backend:
    build:
      context: ./backend
      dockerfile: Dockerfile
    restart: always
    ports:
      - "8080:8080"
    environment:
      - SPRING_PROFILES_ACTIVE=prod
      - DB_URL=jdbc:postgresql://db:5432/utms
      - DB_USERNAME=${DB_USERNAME}
      - DB_PASSWORD=${DB_PASSWORD}
      - JWT_SECRET=${JWT_SECRET}
      - STORAGE_PATH=/var/utms/documents
    volumes:
      - document-storage:/var/utms/documents
    depends_on:
      db:
        condition: service_healthy
    networks:
      - backend-network
    deploy:
      replicas: 2
      resources:
        limits:
          cpus: '1'
          memory: 2G

  frontend:
    build:
      context: ./frontend
      dockerfile: Dockerfile
    restart: always
    ports:
      - "3000:80"
    networks:
      - frontend-network

  db:
    image: postgres:15-alpine
    restart: always
    environment:
      - POSTGRES_DB=utms
      - POSTGRES_USER=${DB_USERNAME}
      - POSTGRES_PASSWORD=${DB_PASSWORD}
    volumes:
      - postgres-data:/var/lib/postgresql/data
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U ${DB_USERNAME}"]
      interval: 10s
      timeout: 5s
      retries: 5
    networks:
      - backend-network

  nginx:
    image: nginx:alpine
    restart: always
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - ./nginx/nginx.conf:/etc/nginx/nginx.conf
      - ./nginx/ssl:/etc/nginx/ssl
    depends_on:
      - backend
      - frontend
    networks:
      - frontend-network
      - backend-network

volumes:
  postgres-data:
  document-storage:

networks:
  frontend-network:
  backend-network:
```

---

## 9. ENVIRONMENT CONFIGURATION

### 9.1 Environment Variables

**Backend:**
```bash
# Application
SPRING_PROFILES_ACTIVE=prod
SERVER_PORT=8080

# Database
DB_URL=jdbc:postgresql://db:5432/utms
DB_USERNAME=utms_user
DB_PASSWORD=<secure-password>
DB_POOL_SIZE=20

# Security
JWT_SECRET=<256-bit-secret>
JWT_ACCESS_EXPIRATION=1800000
JWT_REFRESH_EXPIRATION=604800000

# Storage
STORAGE_PATH=/var/utms/documents

# Email
MAIL_HOST=smtp.example.com
MAIL_PORT=587
MAIL_USERNAME=<email>
MAIL_PASSWORD=<password>

# External APIs (if applicable)
YOKSIS_API_URL=https://api.yoksis.gov.tr
YOKSIS_API_KEY=<api-key>
```

**Frontend:**
```bash
VITE_API_URL=https://api.utms.iyte.edu.tr
VITE_APP_NAME=UTMS
VITE_ENVIRONMENT=production
```

### 9.2 Configuration Files

**application-prod.yml:**
```yaml
spring:
  datasource:
    url: ${DB_URL}
    username: ${DB_USERNAME}
    password: ${DB_PASSWORD}
    hikari:
      maximum-pool-size: 20
      minimum-idle: 5
      
  jpa:
    hibernate:
      ddl-auto: validate
    show-sql: false
    
  mail:
    host: ${MAIL_HOST}
    port: ${MAIL_PORT}
    username: ${MAIL_USERNAME}
    password: ${MAIL_PASSWORD}
    
server:
  port: 8080
  compression:
    enabled: true
    
logging:
  level:
    root: INFO
    com.iztech.utms: INFO
```

---

## 10. DEPLOYMENT PROCEDURES

### 10.1 Pre-Deployment Checklist

```
□ All tests passing
□ Security scan completed
□ Database migrations prepared
□ Environment variables configured
□ SSL certificates valid
□ Backup completed
□ Rollback plan documented
□ Monitoring alerts configured
□ Stakeholders notified
```

### 10.2 Deployment Steps

**1. Database Migration:**
```bash
# Backup current database
pg_dump -h localhost -U utms_user -d utms > backup_$(date +%Y%m%d).sql

# Apply migrations
flyway -url=${DB_URL} -user=${DB_USERNAME} -password=${DB_PASSWORD} migrate
```

**2. Build and Push Images:**
```bash
# Build images
docker build -t utms-backend:v1.0 ./backend
docker build -t utms-frontend:v1.0 ./frontend

# Push to registry
docker push registry.example.com/utms-backend:v1.0
docker push registry.example.com/utms-frontend:v1.0
```

**3. Deploy:**
```bash
# Rolling update
docker-compose pull
docker-compose up -d --no-deps --scale backend=2 backend
docker-compose up -d --no-deps frontend

# Verify health
curl https://api.utms.iyte.edu.tr/actuator/health
```

### 10.3 Rollback Procedure

```bash
# Rollback to previous version
docker-compose down
docker-compose pull utms-backend:v0.9 utms-frontend:v0.9
docker-compose up -d

# Rollback database if needed
psql -h localhost -U utms_user -d utms < backup_YYYYMMDD.sql
```

---

## 11. MONITORING

### 11.1 Health Checks

**Endpoints:**
| Endpoint | Purpose | Expected |
|----------|---------|----------|
| /actuator/health | Overall health | {"status": "UP"} |
| /actuator/health/db | Database | {"status": "UP"} |
| /actuator/health/diskSpace | Storage | {"status": "UP"} |

### 11.2 Metrics to Monitor

**Application Metrics:**
- Request rate (requests/second)
- Response time (p50, p95, p99)
- Error rate (4xx, 5xx)
- Active users
- JVM memory usage
- Thread pool utilization

**Infrastructure Metrics:**
- CPU utilization
- Memory usage
- Disk I/O
- Network throughput
- Database connections

### 11.3 Alerting Rules

| Metric | Warning | Critical |
|--------|---------|----------|
| CPU Usage | > 70% for 5 min | > 90% for 2 min |
| Memory Usage | > 80% | > 95% |
| Response Time (p95) | > 2s | > 5s |
| Error Rate | > 1% | > 5% |
| DB Connections | > 80% pool | > 95% pool |
| Disk Space | < 20% free | < 10% free |

---

## 12. BACKUP & RECOVERY

### 12.1 Backup Strategy

| Data | Frequency | Retention | Type |
|------|-----------|-----------|------|
| Database | Daily | 30 days | Full |
| Database | Hourly | 24 hours | WAL |
| Documents | Daily | 90 days | Incremental |
| Configuration | On change | Forever | Version control |

### 12.2 Backup Procedures

**Database Backup:**
```bash
#!/bin/bash
# Daily backup script
DATE=$(date +%Y%m%d)
BACKUP_DIR=/backups

pg_dump -h localhost -U utms_user -d utms -F c -f ${BACKUP_DIR}/utms_${DATE}.dump

# Encrypt
gpg --encrypt --recipient backup@example.com ${BACKUP_DIR}/utms_${DATE}.dump

# Upload to remote storage
aws s3 cp ${BACKUP_DIR}/utms_${DATE}.dump.gpg s3://utms-backups/db/

# Clean old backups (keep 30 days)
find ${BACKUP_DIR} -name "utms_*.dump*" -mtime +30 -delete
```

### 12.3 Recovery Procedures

**Database Recovery:**
```bash
# Download backup
aws s3 cp s3://utms-backups/db/utms_YYYYMMDD.dump.gpg .

# Decrypt
gpg --decrypt utms_YYYYMMDD.dump.gpg > utms_YYYYMMDD.dump

# Restore
pg_restore -h localhost -U utms_user -d utms -c utms_YYYYMMDD.dump
```

**Recovery Time Objectives:**
| Scenario | RTO | RPO |
|----------|-----|-----|
| Minor failure | 15 min | 0 |
| Database corruption | 1 hour | 1 hour |
| Complete disaster | 4 hours | 24 hours |

---

## 13. REVISION HISTORY

| Version | Date | Author | Changes |
|---------|------|--------|---------|
| 1.0 | Mar 2026 | Group 05 | Initial version |
