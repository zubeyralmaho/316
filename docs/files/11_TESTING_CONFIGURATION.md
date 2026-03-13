# UTMS - Undergraduate Transfer Management System
## 11 - Testing Strategy & Configuration Reference

**Version:** 1.0  
**Last Updated:** March 2026

---

# PART A: TESTING STRATEGY

## 1. TESTING OVERVIEW

### 1.1 Test Pyramid

```
                    ┌───────────────┐
                    │   E2E Tests   │  ~5% of tests
                    │   (Cypress)   │  Slowest, most brittle
                    └───────┬───────┘
                            │
                ┌───────────┴───────────┐
                │  Integration Tests    │  ~20% of tests
                │  (Spring Boot Test)   │  Test component interactions
                └───────────┬───────────┘
                            │
        ┌───────────────────┴───────────────────┐
        │           Unit Tests                   │  ~75% of tests
        │   (JUnit/Mockito, Vitest/RTL)         │  Fast, isolated
        └───────────────────────────────────────┘
```

### 1.2 Testing Goals

| Goal | Target |
|------|--------|
| Code Coverage | ≥ 80% |
| Unit Test Pass Rate | 100% |
| Integration Test Pass Rate | 100% |
| E2E Critical Path | 100% |
| Build Time (with tests) | < 10 minutes |

---

## 2. BACKEND TESTING

### 2.1 Unit Tests

**What to Test:**
- Service layer business logic
- Utility functions
- Validators
- Mappers
- Entity methods

**Test Structure:**
```
src/test/java/com/iztech/utms/
├── service/
│   ├── UserServiceTest.java
│   ├── ApplicationServiceTest.java
│   ├── DocumentServiceTest.java
│   ├── WorkflowServiceTest.java
│   ├── EvaluationServiceTest.java
│   └── NotificationServiceTest.java
├── util/
│   ├── TransferGradeCalculatorTest.java
│   └── ValidationUtilsTest.java
├── mapper/
│   ├── ApplicationMapperTest.java
│   └── UserMapperTest.java
└── entity/
    ├── ApplicationTest.java
    └── UserTest.java
```

**Example Test Scenarios:**

| Service | Test Case | Expected Outcome |
|---------|-----------|------------------|
| UserService | Login with valid credentials | Returns tokens |
| UserService | Login with invalid password | Throws InvalidCredentialsException |
| UserService | Login after 5 failed attempts | Throws AccountLockedException |
| ApplicationService | Create application | Returns application with DRAFT status |
| ApplicationService | Submit without documents | Throws ValidationException |
| EvaluationService | Calculate transfer grade | Correct formula applied |
| WorkflowService | Transfer to invalid stage | Throws WorkflowException |

### 2.2 Integration Tests

**What to Test:**
- Controller → Service → Repository flow
- Database operations
- API endpoints
- Security configuration

**Test Structure:**
```
src/test/java/com/iztech/utms/
├── controller/
│   ├── AuthControllerIT.java
│   ├── ApplicationControllerIT.java
│   └── DocumentControllerIT.java
├── repository/
│   ├── UserRepositoryIT.java
│   ├── ApplicationRepositoryIT.java
│   └── DocumentRepositoryIT.java
└── security/
    └── SecurityConfigIT.java
```

**Test Configuration:**
- Use @SpringBootTest
- Use TestContainers for PostgreSQL
- Use @MockBean for external services
- Use @WithMockUser for security tests

### 2.3 API Contract Tests

**Endpoints to Test:**

| Endpoint | Method | Test Cases |
|----------|--------|------------|
| /auth/login | POST | Valid login, invalid credentials, locked account |
| /auth/register | POST | Valid registration, duplicate email, weak password |
| /applications | POST | Create draft, missing fields, duplicate |
| /applications/{id} | GET | Owner access, staff access, not found |
| /applications/{id}/submit | POST | With documents, without documents |
| /documents | POST | Valid file, invalid type, too large |
| /documents/{id}/review | PUT | Approve, reject, already reviewed |

---

## 3. FRONTEND TESTING

### 3.1 Unit Tests (Vitest + React Testing Library)

**What to Test:**
- Component rendering
- User interactions
- Hook logic
- Utility functions

**Test Structure:**
```
src/
├── components/
│   ├── common/
│   │   ├── Button/
│   │   │   ├── Button.tsx
│   │   │   └── Button.test.tsx
│   │   └── Input/
│   │       ├── Input.tsx
│   │       └── Input.test.tsx
│   └── application/
│       ├── ApplicationCard/
│       │   ├── ApplicationCard.tsx
│       │   └── ApplicationCard.test.tsx
│       └── StatusBadge/
│           ├── StatusBadge.tsx
│           └── StatusBadge.test.tsx
├── hooks/
│   ├── useAuth.ts
│   └── useAuth.test.ts
└── utils/
    ├── validators.ts
    └── validators.test.ts
```

**Example Test Cases:**

| Component | Test Case | Verification |
|-----------|-----------|--------------|
| LoginForm | Render | Shows email, password inputs |
| LoginForm | Submit valid | Calls onSubmit with values |
| LoginForm | Submit invalid | Shows validation errors |
| ApplicationCard | Render | Shows status, department, date |
| StatusBadge | DRAFT status | Renders gray badge |
| StatusBadge | ASIL status | Renders green badge |
| useAuth | Login success | Sets user state |
| useAuth | Logout | Clears user state |

### 3.2 Integration Tests

**What to Test:**
- Page rendering with mocked API
- Form submission flows
- Navigation
- Error handling

### 3.3 E2E Tests (Cypress)

**Critical User Flows:**

| Flow | Steps |
|------|-------|
| Student Registration | Navigate → Fill form → Submit → Verify redirect |
| Student Login | Navigate → Enter credentials → Submit → Dashboard visible |
| Create Application | Login → New app → Fill form → Save → Verify created |
| Upload Document | Select app → Upload tab → Select file → Upload → Verify |
| Submit Application | With documents → Submit → Confirm → Verify status |
| Check Status | Login → My apps → Select → View timeline |
| Staff Login | Navigate → Enter staff credentials → Staff dashboard |
| Review Document | Login → Pending docs → Select → Review → Submit |

---

## 4. TEST DATA

### 4.1 Test Users

| Email | Password | Role | Status |
|-------|----------|------|--------|
| student@test.com | Test123! | STUDENT | Active |
| oidb@test.com | Test123! | OIDB_STAFF | Active |
| dean@test.com | Test123! | DEAN_STAFF | Active |
| ygk@test.com | Test123! | YGK_MEMBER | Active |
| ydyo@test.com | Test123! | YDYO_STAFF | Active |
| admin@test.com | Test123! | SYSTEM_ADMIN | Active |
| locked@test.com | Test123! | STUDENT | Locked |
| inactive@test.com | Test123! | STUDENT | Inactive |

### 4.2 Test Applications

| ID | Status | Stage | Description |
|----|--------|-------|-------------|
| 1 | DRAFT | OIDB_INITIAL | Not submitted |
| 2 | SUBMITTED | OIDB_INITIAL | Pending review |
| 3 | DOCUMENTS_APPROVED | YDYO_EVALUATION | At language review |
| 4 | PENDING_INTIBAK | YGK_EVALUATION | At intibak |
| 5 | ASIL | COMPLETED | Accepted |
| 6 | REJECTED | COMPLETED | Rejected |

### 4.3 Test Documents

| Type | File | Size | Status |
|------|------|------|--------|
| TRANSCRIPT | test_transcript.pdf | 500 KB | PENDING |
| YKS_RESULT | test_yks.pdf | 200 KB | APPROVED |
| ID_CARD | test_id.jpg | 1 MB | REJECTED |
| LANGUAGE_CERTIFICATE | test_lang.pdf | 300 KB | PENDING |

---

# PART B: CONFIGURATION REFERENCE

## 5. BACKEND CONFIGURATION

### 5.1 application.yml Structure

```yaml
# ============================================
# UTMS Backend Configuration
# ============================================

spring:
  application:
    name: utms-backend
    
  profiles:
    active: ${SPRING_PROFILES_ACTIVE:dev}

# -------- Database Configuration --------
  datasource:
    url: ${DB_URL:jdbc:postgresql://localhost:5432/utms}
    username: ${DB_USERNAME:utms_user}
    password: ${DB_PASSWORD:utms_password}
    driver-class-name: org.postgresql.Driver
    hikari:
      maximum-pool-size: ${DB_POOL_SIZE:10}
      minimum-idle: 2
      connection-timeout: 30000
      idle-timeout: 600000
      max-lifetime: 1800000

  jpa:
    hibernate:
      ddl-auto: ${JPA_DDL_AUTO:validate}
    show-sql: ${JPA_SHOW_SQL:false}
    properties:
      hibernate:
        dialect: org.hibernate.dialect.PostgreSQLDialect
        format_sql: true
        default_schema: public

# -------- Security Configuration --------
  security:
    jwt:
      secret: ${JWT_SECRET}
      access-expiration: ${JWT_ACCESS_EXPIRATION:1800000}  # 30 minutes
      refresh-expiration: ${JWT_REFRESH_EXPIRATION:604800000}  # 7 days

# -------- File Upload Configuration --------
  servlet:
    multipart:
      enabled: true
      max-file-size: 15MB
      max-request-size: 20MB

# -------- Email Configuration --------
  mail:
    host: ${MAIL_HOST:smtp.gmail.com}
    port: ${MAIL_PORT:587}
    username: ${MAIL_USERNAME}
    password: ${MAIL_PASSWORD}
    properties:
      mail:
        smtp:
          auth: true
          starttls:
            enable: true

# -------- Server Configuration --------
server:
  port: ${SERVER_PORT:8080}
  compression:
    enabled: true
    mime-types: application/json,application/xml,text/html,text/plain
  error:
    include-message: always
    include-binding-errors: always

# -------- Application Configuration --------
app:
  storage:
    documents:
      path: ${STORAGE_PATH:/var/utms/documents}
  mail:
    from: ${MAIL_FROM:noreply@utms.iyte.edu.tr}
    from-name: ${MAIL_FROM_NAME:UTMS}
  frontend:
    url: ${FRONTEND_URL:http://localhost:3000}
  cors:
    allowed-origins: ${CORS_ORIGINS:http://localhost:3000}

# -------- Actuator Configuration --------
management:
  endpoints:
    web:
      exposure:
        include: health,info,metrics,prometheus
  endpoint:
    health:
      show-details: when_authorized

# -------- Logging Configuration --------
logging:
  level:
    root: INFO
    com.iztech.utms: ${LOG_LEVEL:DEBUG}
    org.springframework.security: WARN
    org.hibernate.SQL: ${SQL_LOG_LEVEL:WARN}
  pattern:
    console: "%d{yyyy-MM-dd HH:mm:ss} [%thread] %-5level %logger{36} - %msg%n"
```

### 5.2 Profile-Specific Configurations

**application-dev.yml:**
```yaml
spring:
  jpa:
    hibernate:
      ddl-auto: update
    show-sql: true
    
logging:
  level:
    com.iztech.utms: DEBUG
    org.hibernate.SQL: DEBUG
```

**application-prod.yml:**
```yaml
spring:
  jpa:
    hibernate:
      ddl-auto: validate
    show-sql: false
    
server:
  tomcat:
    accesslog:
      enabled: true
      
logging:
  level:
    root: WARN
    com.iztech.utms: INFO
```

**application-test.yml:**
```yaml
spring:
  datasource:
    url: jdbc:tc:postgresql:15:///utms_test
    
  jpa:
    hibernate:
      ddl-auto: create-drop
```

---

## 6. FRONTEND CONFIGURATION

### 6.1 Vite Configuration (vite.config.ts)

```typescript
import { defineConfig } from 'vite';
import react from '@vitejs/plugin-react';
import path from 'path';

export default defineConfig({
  plugins: [react()],
  
  resolve: {
    alias: {
      '@': path.resolve(__dirname, './src'),
      '@components': path.resolve(__dirname, './src/components'),
      '@pages': path.resolve(__dirname, './src/pages'),
      '@hooks': path.resolve(__dirname, './src/hooks'),
      '@store': path.resolve(__dirname, './src/store'),
      '@api': path.resolve(__dirname, './src/api'),
      '@types': path.resolve(__dirname, './src/types'),
      '@utils': path.resolve(__dirname, './src/utils'),
    },
  },
  
  server: {
    port: 3000,
    proxy: {
      '/api': {
        target: 'http://localhost:8080',
        changeOrigin: true,
      },
    },
  },
  
  build: {
    outDir: 'dist',
    sourcemap: true,
    rollupOptions: {
      output: {
        manualChunks: {
          vendor: ['react', 'react-dom', 'react-router-dom'],
          ui: ['antd'],
        },
      },
    },
  },
  
  test: {
    globals: true,
    environment: 'jsdom',
    setupFiles: './src/test/setup.ts',
    coverage: {
      reporter: ['text', 'json', 'html'],
      exclude: ['node_modules/', 'src/test/'],
    },
  },
});
```

### 6.2 Environment Files

**.env.development:**
```bash
VITE_API_URL=http://localhost:8080/api/v1
VITE_APP_NAME=UTMS (Development)
VITE_ENVIRONMENT=development
VITE_ENABLE_MOCK=false
```

**.env.production:**
```bash
VITE_API_URL=https://api.utms.iyte.edu.tr/api/v1
VITE_APP_NAME=UTMS
VITE_ENVIRONMENT=production
VITE_ENABLE_MOCK=false
```

### 6.3 TypeScript Configuration (tsconfig.json)

```json
{
  "compilerOptions": {
    "target": "ES2020",
    "useDefineForClassFields": true,
    "lib": ["ES2020", "DOM", "DOM.Iterable"],
    "module": "ESNext",
    "skipLibCheck": true,
    "moduleResolution": "bundler",
    "allowImportingTsExtensions": true,
    "resolveJsonModule": true,
    "isolatedModules": true,
    "noEmit": true,
    "jsx": "react-jsx",
    "strict": true,
    "noUnusedLocals": true,
    "noUnusedParameters": true,
    "noFallthroughCasesInSwitch": true,
    "baseUrl": ".",
    "paths": {
      "@/*": ["src/*"],
      "@components/*": ["src/components/*"],
      "@pages/*": ["src/pages/*"],
      "@hooks/*": ["src/hooks/*"],
      "@store/*": ["src/store/*"],
      "@api/*": ["src/api/*"],
      "@types/*": ["src/types/*"],
      "@utils/*": ["src/utils/*"]
    }
  },
  "include": ["src"],
  "references": [{ "path": "./tsconfig.node.json" }]
}
```

### 6.4 ESLint Configuration (.eslintrc.cjs)

```javascript
module.exports = {
  root: true,
  env: { browser: true, es2020: true },
  extends: [
    'eslint:recommended',
    'plugin:@typescript-eslint/recommended',
    'plugin:react-hooks/recommended',
    'plugin:react/recommended',
    'prettier',
  ],
  ignorePatterns: ['dist', '.eslintrc.cjs'],
  parser: '@typescript-eslint/parser',
  plugins: ['react-refresh'],
  rules: {
    'react-refresh/only-export-components': [
      'warn',
      { allowConstantExport: true },
    ],
    'react/react-in-jsx-scope': 'off',
    '@typescript-eslint/explicit-function-return-type': 'off',
    '@typescript-eslint/no-unused-vars': ['error', { argsIgnorePattern: '^_' }],
  },
  settings: {
    react: {
      version: 'detect',
    },
  },
};
```

---

## 7. NGINX CONFIGURATION

### 7.1 Production nginx.conf

```nginx
upstream backend {
    server backend:8080;
    keepalive 32;
}

server {
    listen 80;
    server_name utms.iyte.edu.tr;
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    server_name utms.iyte.edu.tr;

    # SSL Configuration
    ssl_certificate /etc/nginx/ssl/cert.pem;
    ssl_certificate_key /etc/nginx/ssl/key.pem;
    ssl_session_timeout 1d;
    ssl_session_cache shared:SSL:50m;
    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_ciphers ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256;
    ssl_prefer_server_ciphers off;

    # Security Headers
    add_header Strict-Transport-Security "max-age=31536000; includeSubDomains" always;
    add_header X-Content-Type-Options nosniff;
    add_header X-Frame-Options DENY;
    add_header X-XSS-Protection "1; mode=block";

    # Gzip Compression
    gzip on;
    gzip_types text/plain text/css application/json application/javascript;
    gzip_min_length 1000;

    # Frontend (Static files)
    location / {
        root /usr/share/nginx/html;
        index index.html;
        try_files $uri $uri/ /index.html;
        
        # Cache static assets
        location ~* \.(js|css|png|jpg|jpeg|gif|ico|svg|woff2)$ {
            expires 1y;
            add_header Cache-Control "public, immutable";
        }
    }

    # Backend API
    location /api/ {
        proxy_pass http://backend;
        proxy_http_version 1.1;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_set_header Connection "";
        
        # Timeouts
        proxy_connect_timeout 60s;
        proxy_send_timeout 60s;
        proxy_read_timeout 60s;
        
        # Buffer settings
        proxy_buffering on;
        proxy_buffer_size 128k;
        proxy_buffers 4 256k;
    }

    # File uploads
    location /api/v1/applications/*/documents {
        client_max_body_size 20M;
        proxy_pass http://backend;
        proxy_http_version 1.1;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_request_buffering off;
    }

    # Health check endpoint
    location /health {
        proxy_pass http://backend/actuator/health;
        proxy_http_version 1.1;
    }
}
```

---

## 8. CI/CD CONFIGURATION

### 8.1 GitHub Actions Workflow

```yaml
name: CI/CD Pipeline

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main]

env:
  REGISTRY: ghcr.io
  IMAGE_NAME: ${{ github.repository }}

jobs:
  # Backend Tests
  backend-test:
    runs-on: ubuntu-latest
    services:
      postgres:
        image: postgres:15
        env:
          POSTGRES_DB: utms_test
          POSTGRES_USER: test
          POSTGRES_PASSWORD: test
        ports:
          - 5432:5432
        options: >-
          --health-cmd pg_isready
          --health-interval 10s
          --health-timeout 5s
          --health-retries 5

    steps:
      - uses: actions/checkout@v4
      
      - name: Set up JDK 17
        uses: actions/setup-java@v4
        with:
          java-version: '17'
          distribution: 'temurin'
          cache: maven

      - name: Run tests
        working-directory: ./backend
        run: mvn test -Dspring.profiles.active=test

      - name: Generate coverage report
        working-directory: ./backend
        run: mvn jacoco:report

  # Frontend Tests
  frontend-test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - name: Set up Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'npm'
          cache-dependency-path: frontend/package-lock.json

      - name: Install dependencies
        working-directory: ./frontend
        run: npm ci

      - name: Run tests
        working-directory: ./frontend
        run: npm run test:coverage

  # Build and Push
  build:
    needs: [backend-test, frontend-test]
    runs-on: ubuntu-latest
    if: github.ref == 'refs/heads/main'
    
    steps:
      - uses: actions/checkout@v4

      - name: Login to GitHub Container Registry
        uses: docker/login-action@v3
        with:
          registry: ${{ env.REGISTRY }}
          username: ${{ github.actor }}
          password: ${{ secrets.GITHUB_TOKEN }}

      - name: Build and push backend
        uses: docker/build-push-action@v5
        with:
          context: ./backend
          push: true
          tags: ${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}-backend:${{ github.sha }}

      - name: Build and push frontend
        uses: docker/build-push-action@v5
        with:
          context: ./frontend
          push: true
          tags: ${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}-frontend:${{ github.sha }}

  # Deploy
  deploy:
    needs: build
    runs-on: ubuntu-latest
    if: github.ref == 'refs/heads/main'
    
    steps:
      - name: Deploy to production
        uses: appleboy/ssh-action@v1.0.0
        with:
          host: ${{ secrets.PROD_HOST }}
          username: ${{ secrets.PROD_USER }}
          key: ${{ secrets.PROD_SSH_KEY }}
          script: |
            cd /opt/utms
            docker-compose pull
            docker-compose up -d
```

---

## 9. REVISION HISTORY

| Version | Date | Author | Changes |
|---------|------|--------|---------|
| 1.0 | Mar 2026 | Group 05 | Initial version |
