# Phase 5: Integration & CI/CD

> **สัปดาห์ 5–6** — ประกอบร่างทุกชิ้นส่วน: integration tests ยืนยันว่า API ทำงานจริง,
> CI pipeline รันอัตโนมัติทุก push, production Docker image พร้อม deploy,
> และ PM sign-off ว่าทุก spec ถูก implement ครบ

---

## ภาพรวม Phase นี้

Phase 5 เปลี่ยนจาก "ทำงานคนเดียว" เป็น "ทำงานร่วมกัน" — backend, frontend, database
และ infrastructure ต้องทำงานร่วมกันได้จริง

```
┌────────────────────────────────────────────────────────┐
│                  Phase 5 Pipeline                       │
│                                                         │
│  QA               DevOps            SA         PM       │
│  ┌──────────┐    ┌────────────┐   ┌───────┐  ┌───────┐│
│  │Integration│    │CI Pipeline │   │Review │  │Release││
│  │  Tests   │───▶│Dockerfile  │──▶│ Arch  │─▶│Check- ││
│  │Security  │    │Compose     │   │       │  │ list  ││
│  └──────────┘    └────────────┘   └───────┘  └───────┘│
└────────────────────────────────────────────────────────┘
```

📌 **Cross-ref:** [บทที่ 4](../04_fullstack_collaboration.md) — CI/CD, Docker |
[บทที่ 7](../07_qa_security_review.md) — QA & Security Review

---

## ใครทำอะไร

| ลำดับ | Role | คน | งาน | Claude Code ช่วยเรื่อง |
|-------|------|-----|------|------------------------|
| 1 | QA | 👤 คุณคิว | เขียน integration tests | สร้าง test suite, test containers setup |
| 2 | QA | 👤 คุณคิว | Security review | ตรวจ OWASP Top 10, security test cases |
| 3 | DevOps | 👤 คุณดีบี | สร้าง CI pipeline | GitHub Actions workflow |
| 4 | DevOps | 👤 คุณดีบี | สร้าง production Dockerfile | Multi-stage build (Go + React) |
| 5 | DevOps | 👤 คุณดีบี | Docker Compose staging | Full stack environment |
| 6 | SA | 👤 คุณเอส | Review deployment architecture | ตรวจ Docker setup, แนะนำปรับปรุง |
| 7 | PM | 👤 คุณแพร | ตรวจ release readiness | Release checklist, verify specs |

---

## Step 1: QA เขียน Integration Tests

👤 **คุณคิว (QA)** — ยืนยันว่า API ทำงานได้จริงเมื่อต่อกับ database จริง ไม่ใช่แค่ mock

📌 **Cross-ref:** [บทที่ 7](../07_qa_security_review.md) — Integration Testing

### 1.1 Setup Test Containers

```
ดู test files ใน internal/handler/ และ internal/service/ ทั้งหมด
แล้วสร้าง integration test suite ใหม่ใน tests/integration/ ที่:
1. ใช้ testcontainers-go เพื่อ spin up PostgreSQL และ Redis containers
2. มี TestMain ที่ setup/teardown containers อัตโนมัติ
3. มี helper function สำหรับ seed test data
4. ใช้ separate test database ต่อ test suite เพื่อ avoid conflicts
```

Claude Code สร้าง `tests/integration/setup_test.go` — ใช้ testcontainers-go spin up
PostgreSQL 16 + Redis 7 ใน `TestMain`, connect ผ่าน GORM, run auto-migrate,
แล้ว `os.Exit(m.Run())` — containers ถูก terminate อัตโนมัติผ่าน `defer`

### 1.2 เขียน API Integration Tests

```
สร้าง integration tests สำหรับ order flow แบบ end-to-end:
1. POST /api/v1/auth/register — สร้าง user ใหม่
2. POST /api/v1/auth/login — login เพื่อได้ JWT token
3. POST /api/v1/orders — สร้าง order (ใช้ token)
4. GET /api/v1/orders/:id — ดึง order ที่สร้าง
5. PATCH /api/v1/orders/:id/status — เปลี่ยนสถานะ order

ทุก test ต้อง:
- ใช้ httptest กับ Fiber app จริง (ไม่ mock)
- ตรวจสอบทั้ง response body และ database state
- cleanup data หลังแต่ละ test
```

### 1.3 Test Data Seeding

```
สร้าง test fixture helper ที่:
- SeedProducts(db, count) สร้าง products พร้อม categories
- SeedUser(db, role) สร้าง user พร้อม hashed password
- CleanAll(db) truncate ทุก table ตาม foreign key order
- ใช้ faker library สำหรับ realistic test data
```

💡 **เคล็ดลับ:** บอก Claude Code ให้ truncate tables ตาม foreign key dependency order
เพื่อหลีกเลี่ยง constraint violations — หลายทีมเจอปัญหานี้ตอนรัน parallel tests

---

## Step 2: QA รัน Security Review

👤 **คุณคิว (QA)** — ก่อน deploy ต้องตรวจ security ตาม OWASP Top 10

📌 **Cross-ref:** [บทที่ 7](../07_qa_security_review.md) — Security Review

### 2.1 OWASP Top 10 Review

```
Review codebase ตาม OWASP Top 10 สำหรับ Go/Fiber API:
1. Injection — ตรวจว่าใช้ parameterized queries ทุกที่
2. Broken Authentication — ตรวจ JWT implementation, password hashing
3. Sensitive Data Exposure — ไม่มี secret ใน code, ไม่ return password
4. Broken Access Control — ตรวจ middleware authorization ทุก route
5. Security Misconfiguration — ตรวจ CORS, headers, error messages
6. XSS — ตรวจ output encoding ทั้ง API และ frontend
7. Components with Known Vulnerabilities — ตรวจ go.mod dependencies
8. Insufficient Logging — ตรวจว่า log auth events

รายงานเป็นตาราง: หมวด | สถานะ (PASS/FAIL/WARN) | รายละเอียด | แนวทางแก้ไข
```

### 2.2 SQL Injection & Auth Bypass Tests

```
สร้าง security test cases:

SQL Injection:
- GET /api/v1/products?search=' OR 1=1 --
- GET /api/v1/products?sort=name; DROP TABLE products
- Assert: ไม่มี SQL error, data ไม่ leak, app ไม่ crash

Auth Bypass:
1. เรียก protected endpoint โดยไม่มี token → 401
2. ใช้ expired token → 401
3. User role เข้า admin endpoint → 403
4. แก้ JWT payload (tamper) → 401
5. User A token เข้าถึง resource ของ user B → 403
6. Brute force login → rate limit หลัง 5 attempts
```

⚠️ **ข้อควรระวัง:** อย่าลืมทดสอบ horizontal privilege escalation (user A เข้าถึง data ของ user B)
— เป็น bug ที่พบบ่อยมากและ unit test จับไม่ได้

---

## Step 3: DevOps สร้าง CI Pipeline

👤 **คุณดีบี (DevOps)** — สร้าง CI pipeline ที่รันอัตโนมัติทุก push

📌 **Cross-ref:** [บทที่ 4](../04_fullstack_collaboration.md) — หัวข้อ CI/CD

```
สร้าง GitHub Actions workflow ใน .github/workflows/ci.yml:
1. Trigger: push to main, pull_request to main
2. Jobs (ตาม dependency order):
   a. lint — golangci-lint + eslint
   b. test-unit — go test ./internal/...
   c. test-integration — ต้อง PostgreSQL + Redis services
   d. build-and-push — Docker images to ghcr.io (เฉพาะ main)
ใช้ caching สำหรับ Go modules และ npm dependencies
```

Claude Code สร้าง workflow ที่สำคัญ — โดยเฉพาะ integration test job ที่ใช้ GitHub Actions services:

```yaml
# .github/workflows/ci.yml (ย่อเฉพาะ integration test job)
  test-integration:
    needs: lint
    runs-on: ubuntu-latest
    services:
      postgres:
        image: postgres:16-alpine
        env:
          POSTGRES_DB: shopfast_test
          POSTGRES_USER: test
          POSTGRES_PASSWORD: test
        ports: ["5432:5432"]
        options: >-
          --health-cmd pg_isready
          --health-interval 10s
          --health-timeout 5s
          --health-retries 5
      redis:
        image: redis:7-alpine
        ports: ["6379:6379"]
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-go@v5
        with:
          go-version: "1.22"
          cache: true
      - run: go test -race -v ./tests/integration/...

  build-and-push:
    needs: [test-unit, test-integration]
    if: github.ref == 'refs/heads/main'
    # ... docker/build-push-action สำหรับ API + frontend images
```

💡 **เคล็ดลับ:** เพิ่ม `concurrency` group เพื่อ cancel CI runs เก่าเมื่อ push commit ใหม่
— ประหยัด GitHub Actions minutes

---

## Step 4: DevOps สร้าง Production Dockerfile

👤 **คุณดีบี (DevOps)** — สร้าง optimized Docker images สำหรับ production

📌 **Cross-ref:** [บทที่ 4](../04_fullstack_collaboration.md) — หัวข้อ Docker

### 4.1 Go API — Multi-stage Dockerfile

```
สร้าง multi-stage Dockerfile สำหรับ Go API (Dockerfile.api):
- Stage 1 (builder): golang:1.22-alpine, build static binary ด้วย ldflags
- Stage 2 (runner): gcr.io/distroless/static-debian12
- non-root user, copy เฉพาะ binary + configs
```

```dockerfile
# Dockerfile.api
FROM golang:1.22-alpine AS builder
WORKDIR /build
COPY go.mod go.sum ./
RUN go mod download
COPY . .
ARG VERSION=dev
RUN CGO_ENABLED=0 GOOS=linux go build \
    -ldflags="-s -w -X main.version=${VERSION}" \
    -o /build/api ./cmd/api

FROM gcr.io/distroless/static-debian12
COPY --from=builder /build/api /api
COPY --from=builder /build/configs /configs
EXPOSE 8080
USER nonroot:nonroot
ENTRYPOINT ["/api"]
```

### 4.2 React Frontend — Multi-stage Dockerfile

```
สร้าง multi-stage Dockerfile สำหรับ React (frontend/Dockerfile):
- Stage 1: node:20-alpine, npm ci, npm run build
- Stage 2: nginx:alpine, custom nginx.conf สำหรับ SPA routing
```

```dockerfile
# frontend/Dockerfile
FROM node:20-alpine AS builder
WORKDIR /app
COPY package.json package-lock.json ./
RUN npm ci --ignore-scripts
COPY . .
RUN npm run build

FROM nginx:1.25-alpine
COPY --from=builder /app/dist /usr/share/nginx/html
COPY nginx.conf /etc/nginx/conf.d/default.conf
EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
```

⚠️ **ข้อควรระวัง:** สร้าง `.dockerignore` ก่อน build — ถ้าไม่มี Docker จะ copy `.git/`,
`node_modules/`, test files เข้า build context ทำให้ build ช้ามาก

---

## Step 5: DevOps ตั้ง Docker Compose สำหรับ Staging

👤 **คุณดีบี (DevOps)** — สร้าง full stack environment ที่เหมือน production

```
สร้าง docker-compose.staging.yml ที่มี:
1. api — Go API, ต่อ postgres + redis, health check
2. frontend — React app (nginx)
3. postgres — PostgreSQL 16 with volume persistence
4. redis — Redis 7 with AOF persistence
5. nginx — reverse proxy, route /api/* → api, /* → frontend
ใช้ named volumes, custom network, health checks, .env file
```

```yaml
# docker-compose.staging.yml (ย่อ)
services:
  nginx:
    image: nginx:1.25-alpine
    ports: ["443:443", "80:80"]
    volumes:
      - ./deploy/nginx/staging.conf:/etc/nginx/conf.d/default.conf
    depends_on:
      api: { condition: service_healthy }

  api:
    build: { context: ., dockerfile: Dockerfile.api }
    environment:
      - DB_HOST=postgres
      - DB_PASSWORD=${DB_PASSWORD}
      - REDIS_HOST=redis
      - JWT_SECRET=${JWT_SECRET}
    healthcheck:
      test: ["CMD", "/api", "health"]
      interval: 10s
    depends_on:
      postgres: { condition: service_healthy }
      redis: { condition: service_healthy }

  frontend:
    build: { context: ./frontend }

  postgres:
    image: postgres:16-alpine
    environment:
      POSTGRES_DB: ${DB_NAME:-shopfast}
      POSTGRES_PASSWORD: ${DB_PASSWORD}
    volumes: ["pgdata:/var/lib/postgresql/data"]
    healthcheck:
      test: ["CMD-SHELL", "pg_isready"]

  redis:
    image: redis:7-alpine
    command: redis-server --appendonly yes --requirepass ${REDIS_PASSWORD}
    volumes: ["redisdata:/data"]

volumes:
  pgdata:
  redisdata:

networks:
  default:
    name: shopfast
```

💡 **เคล็ดลับ:** ใช้ `docker compose --profile` เพื่อแยก dev-only services
(pgadmin, redis-commander) ออกจาก production services

---

## Step 6: SA Review Deployment Architecture

👤 **คุณเอส (SA)** — ตรวจสอบ Docker setup ก่อน go-live

```
Review docker-compose.staging.yml และ Dockerfiles ทั้งหมด:
1. Security: secret leak, container run as root?
2. Networking: service ไหนควร/ไม่ควร expose port?
3. Persistence: data หายไหมถ้า container restart?
4. Health checks: ครบทุก service? timeout เหมาะสม?
5. Resource limits: ควร set memory/cpu limits?
รายงานเป็นตาราง: หมวด | สถานะ | findings | recommendations
```

SA Findings:

| หมวด | สถานะ | Findings | Recommendations |
|------|--------|----------|-----------------|
| Security | ⚠️ WARN | API ยังไม่มี read-only filesystem | เพิ่ม `read_only: true` + tmpfs |
| Networking | ✅ PASS | เฉพาะ nginx expose port 80/443 | — |
| Persistence | ✅ PASS | Named volumes สำหรับ postgres + redis | — |
| Resources | ⚠️ WARN | ไม่มี memory limits | เพิ่ม `mem_limit` ทุก service |
| Logging | ⚠️ WARN | ใช้ default logging driver | json-file กับ max-size: 10m |

คุณเอสให้ Claude Code เพิ่ม production hardening:

```
ตาม findings เพิ่ม hardening:
- deploy.resources.limits สำหรับทุก service
- restart: unless-stopped
- logging driver กับ rotation
- read_only: true + security_opt: no-new-privileges
```

---

## Step 7: PM ตรวจ Release Readiness

👤 **คุณแพร (PM)** — ยืนยันว่าทุก spec ถูก implement ครบก่อนปล่อย production

### 7.1 สร้าง Release Checklist

```
อ่าน specs ทั้งหมดใน docs/specs/ แล้วสร้าง release checklist:
1. แต่ละ spec — list acceptance criteria ทั้งหมด
2. ตรวจว่าแต่ละ criteria มี: implementation, test coverage, API docs
3. สรุปเป็นตาราง: Feature | Code | Unit Test | Integration Test | Status
4. ระบุ blockers
```

### 7.2 Verify Spec Coverage

```
เปรียบเทียบ acceptance criteria จาก docs/specs/order_management.md
กับ test cases ใน tests/integration/ และ internal/service/*_test.go:
- criteria ไหนมี test แล้ว? ไหนยังไม่มี?
- สร้างรายการ gaps ที่ต้องเพิ่ม test ก่อน release
```

ผลลัพธ์ที่คุณแพรได้:

| Feature | Criteria | Code | Unit Test | Integration Test | Status |
|---------|----------|------|-----------|-----------------|--------|
| User Registration | Email + password validation | ✅ | ✅ | ✅ | READY |
| User Login | JWT token issuance | ✅ | ✅ | ✅ | READY |
| Create Order | Stock validation | ✅ | ✅ | ✅ | READY |
| Create Order | Price calculation | ✅ | ✅ | ⬜ | NEED TEST |
| Order Status | State machine transitions | ✅ | ✅ | ✅ | READY |
| Search Products | Full-text search | ✅ | ✅ | ⬜ | NEED TEST |
| Admin Dashboard | Order list + filter | ✅ | ⬜ | ⬜ | NEED TEST |

### 7.3 สร้าง Release Notes Draft

```
จาก git log และ specs ที่ implement แล้ว สร้าง release notes v1.0.0:
- Summary, New Features, API Endpoints (table)
- Known Issues, Deployment Notes (env vars ที่ต้อง set)
```

💡 **เคล็ดลับ:** ให้ Claude Code เปรียบเทียบ spec กับ code โดยตรง — จะพบ gaps ที่
manual review พลาด เช่น edge cases ที่ spec กำหนดแต่ code ไม่ได้ handle

---

## ✅ Checkpoint — Phase 5 เสร็จสมบูรณ์เมื่อ

| # | รายการ | ผู้รับผิดชอบ |
|---|--------|------------|
| 1 | Integration tests ผ่านทั้งหมด (order flow, auth flow) | 👤 คุณคิว |
| 2 | Security review ไม่พบ critical/high vulnerabilities | 👤 คุณคิว |
| 3 | CI pipeline รันผ่าน (lint → test → build → push) | 👤 คุณดีบี |
| 4 | Production Dockerfiles build สำเร็จ, image size ตาม target | 👤 คุณดีบี |
| 5 | Docker Compose staging — `docker compose up` ใช้งานได้จริง | 👤 คุณดีบี |
| 6 | SA review ผ่าน — ไม่มี critical findings ค้าง | 👤 คุณเอส |
| 7 | Release checklist ไม่มี blocker items | 👤 คุณแพร |
| 8 | ทุก spec acceptance criteria มี test coverage | 👤 คุณคิว + คุณแพร |

---

## เคล็ดลับ & ข้อผิดพลาดที่พบบ่อย

### 💡 เคล็ดลับ

1. **ใช้ testcontainers แทน mock database** — จับ bugs ที่ mock จับไม่ได้
   (constraint violations, migration issues)

2. **CI ควรมี cache strategy** — Cache Go modules + npm dependencies
   ลดเวลา CI จาก 10+ นาทีเหลือ 3-4 นาที

3. **Multi-stage Dockerfile ลด image size 10x** — Go API: ~1GB → ~20MB (distroless),
   React: ~1GB → ~30MB (nginx:alpine)

4. **ให้ PM verify specs ด้วย Claude Code** — cross-reference spec กับ code
   ได้เร็วกว่า manual review มาก

### ⚠️ ข้อผิดพลาดที่พบบ่อย

1. **ลืม health check** — API start ก่อน database พร้อม → connection refused → restart loop

2. **Hardcode secrets ใน docker-compose.yml** — ใช้ `.env` file และใส่ `.env` ใน `.gitignore`

3. **Integration test ไม่ cleanup data** — tests ทีหลัง fail เพราะ data เก่ายังอยู่
   — ใช้ transaction rollback หรือ truncate

4. **CI ไม่ fail fast** — lint fail แล้วยัง run tests → เสีย CI minutes ฟรี
   — ใช้ `needs: lint` ใน GitHub Actions

5. **Docker build context ใหญ่เกินไป** — ไม่มี `.dockerignore` ทำให้ build ช้ามาก

6. **ไม่ set resource limits** — container กิน memory ไม่จำกัด → OOM kill

7. **Skip security review** — "เดี๋ยวค่อยทำ" แล้วไม่ทำ → ช่องโหว่ขึ้น production

---

*← [Phase 4: Frontend](04_phase4_frontend.md) | [Phase 6: Maintenance →](06_phase6_maintenance.md)*
