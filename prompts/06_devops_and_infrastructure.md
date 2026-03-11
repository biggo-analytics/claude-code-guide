# DevOps & Infrastructure — CI/CD, Docker, Monitoring, Incidents

> Prompt สำหรับ DevOps / SRE — ครอบคลุม CI/CD pipeline, containerization, monitoring, incident response
> ใช้ได้กับทุก tech stack โดยแทนค่า `[placeholder]`

---

## สารบัญ

- [6.1 CI/CD Pipeline](#61-cicd-pipeline)
- [6.2 Docker & Containers](#62-docker--containers)
- [6.3 Infrastructure & Configuration](#63-infrastructure--configuration)
- [6.4 Monitoring & Observability](#64-monitoring--observability)
- [6.5 Incident Response](#65-incident-response)

---

## 6.1 CI/CD Pipeline

**OPS-01.** สร้าง CI Pipeline (GitHub Actions)
> สร้าง pipeline สำหรับ build + test + lint

```
สร้าง GitHub Actions workflow สำหรับโปรเจกต์นี้:
ดูจาก:
- ภาษา/framework ที่ใช้ (อ่านจาก [package.json / go.mod / etc.])
- Test command ที่ใช้
- Lint command ที่ใช้
- Build command ที่ใช้

สร้าง `.github/workflows/ci.yml`:
1. Trigger: push to main, pull_request
2. Jobs:
   - lint — run linter
   - test — run tests (พร้อม DB ถ้าต้องการ)
   - build — build project
3. ใช้ caching สำหรับ dependencies
4. ใส่ badge สำหรับ README
```

---

**OPS-02.** สร้าง CD Pipeline (Deploy)
> สร้าง deployment pipeline

```
สร้าง deployment workflow:
- Environment: [staging / production]
- Deploy target: [Docker / K8s / Cloud Run / EC2 / etc.]
- Trigger: [manual / on tag / on merge to main]

สร้าง `.github/workflows/deploy.yml`:
1. Build Docker image (ถ้าใช้ Docker)
2. Push to registry [registry URL]
3. Deploy to [target]
4. Health check after deploy
5. Rollback step (ถ้า health check fail)
6. Notification (Slack/email)
ใส่ environment secrets ที่ต้องตั้งค่า
```

---

**OPS-03.** Debug CI ที่ fail
> วิเคราะห์ CI failure

```
CI pipeline fail — ช่วยวิเคราะห์:
Error log:
```
[paste error log]
```

1. Root cause คืออะไร?
2. เป็น flaky test, dependency issue, หรือ real bug?
3. วิธีแก้ (fix code, fix pipeline, หรือ retry?)
4. วิธีป้องกันไม่ให้เกิดซ้ำ
```

---

**OPS-04.** เพิ่ม step ใน pipeline
> ต่อเติม pipeline ที่มีอยู่

```
เพิ่ม step ใน CI/CD pipeline:
อ่าน workflow file ใน [.github/workflows/ci.yml]
เพิ่ม:
- [อธิบาย step ที่ต้องการเพิ่ม]
- ตำแหน่ง: [ก่อน/หลัง step ไหน]
- เงื่อนไข: [run เมื่อไหร่]
ให้ integrate เข้ากับ pipeline ที่มีอยู่อย่างเหมาะสม
```

---

**OPS-05.** สร้าง Release automation
> automate versioning และ release notes

```
สร้าง release automation workflow:
1. Versioning strategy: [semantic versioning / calver]
2. Trigger: [manual / on merge to main]
3. Steps:
   - Bump version number
   - Generate changelog จาก git commits
   - Create GitHub release with notes
   - Build & publish artifacts
   - Tag the release
4. Optional: publish to [npm / PyPI / Docker Hub / etc.]
```

---

## 6.2 Docker & Containers

**OPS-06.** สร้าง Dockerfile (Multi-stage)
> สร้าง production-ready Dockerfile

```
สร้าง multi-stage Dockerfile สำหรับโปรเจกต์นี้:
อ่าน tech stack จาก [package.json / go.mod / etc.]

Requirements:
1. Build stage — compile/build (ใช้ full image)
2. Production stage — run (ใช้ minimal image เช่น alpine/distroless)
3. Non-root user
4. .dockerignore file
5. Health check instruction
6. Optimize layer caching (copy dependency files ก่อน copy source)
7. Set proper labels (version, maintainer)

ต้อง image size เล็กที่สุดเท่าที่ได้
```

---

**OPS-07.** สร้าง Docker Compose สำหรับ development
> dev environment ที่ครบในคำสั่งเดียว

```
สร้าง docker-compose.yml สำหรับ development:
Services ที่ต้องการ:
- [app] — application (mount source code, hot reload)
- [db] — [PostgreSQL / MySQL / MongoDB] with seed data
- [cache] — [Redis / Memcached] (ถ้าใช้)
- [queue] — [RabbitMQ / Kafka] (ถ้าใช้)

Requirements:
1. Volume mounts สำหรับ source code (live reload)
2. Volume สำหรับ DB data (persist ข้าม restart)
3. Environment variables จาก .env file
4. Health checks สำหรับ dependencies
5. Network configuration
6. สร้าง .env.example ด้วย
```

---

**OPS-08.** สร้าง Docker Compose สำหรับ staging/production
> production-like environment

```
สร้าง docker-compose.prod.yml:
อ้างอิงจาก docker-compose.yml (dev) ที่มีอยู่
ปรับสำหรับ production:
1. ใช้ built image แทน volume mount
2. Resource limits (memory, CPU)
3. Restart policy (always/unless-stopped)
4. Logging configuration
5. ไม่เปิด port ที่ไม่จำเป็น
6. Production environment variables
7. SSL/TLS configuration (ถ้ามี reverse proxy)
```

---

**OPS-09.** Debug Docker issues
> แก้ปัญหา Docker

```
Docker มีปัญหา — ช่วยวิเคราะห์:
อ่าน [Dockerfile / docker-compose.yml]

ปัญหา: [อธิบายปัญหา]
Error: [paste error message]

ช่วย:
1. วิเคราะห์ root cause
2. แนะนำวิธีแก้
3. ตรวจ best practices ที่อาจขาด
4. ป้องกันไม่ให้เกิดซ้ำ
```

---

## 6.3 Infrastructure & Configuration

**OPS-10.** สร้าง Makefile
> รวมคำสั่งที่ใช้บ่อย

```
สร้าง Makefile สำหรับโปรเจกต์นี้:
ดูจาก tech stack และ commands ที่ใช้

Targets ที่ต้องมี:
- make dev — start development environment
- make test — run all tests
- make lint — run linter
- make build — build for production
- make clean — clean build artifacts
- make docker-build — build Docker image
- make docker-up — start with docker-compose
- make docker-down — stop containers
- make migrate — run database migrations
- make seed — seed database
- make help — show available targets

เพิ่ม .PHONY declarations และ help target ที่แสดงทุก targets
```

---

**OPS-11.** จัดการ Environment variables
> document และจัดระเบียบ env vars

```
จัดการ environment variables ของโปรเจกต์:
1. สแกน codebase หา env vars ที่ใช้ทั้งหมด
   (os.Getenv, process.env, os.environ, etc.)
2. สร้าง/update `.env.example` ครบทุกตัว:
   - จัดกลุ่มตามหมวด (DB, Auth, External Services, etc.)
   - ใส่ comment อธิบายแต่ละตัว
   - ใส่ default values ที่ปลอดภัย (ไม่ใช่ค่าจริง)
   - ระบุว่าตัวไหน required vs optional
3. ตรวจว่า .env อยู่ใน .gitignore
```

---

**OPS-12.** Security hardening
> ตรวจและปรับปรุง security ของ infrastructure

```
ตรวจ infrastructure security:
อ่าน [Dockerfile, docker-compose.yml, CI configs, nginx config]

Checklist:
1. Docker:
   - Non-root user?
   - No unnecessary capabilities?
   - Image from trusted source?
   - No secrets in image layers?

2. Network:
   - Unnecessary ports exposed?
   - Internal services accessible from outside?
   - HTTPS configured?

3. CI/CD:
   - Secrets stored properly (not in code)?
   - Minimal permissions for CI?
   - Dependency pinning (hash verification)?

4. Configuration:
   - Debug mode disabled in production?
   - CORS configured properly?
   - Rate limiting enabled?

รายงาน findings + fixes
```

---

## 6.4 Monitoring & Observability

**OPS-13.** เพิ่ม Structured logging
> ปรับ logging ให้ search และ analyze ได้

```
เพิ่ม structured logging ใน [module/path]:
1. อ่าน logging ที่มีอยู่
2. ปรับเป็น structured format (JSON):
   - timestamp
   - level (info/warn/error)
   - message
   - context fields (request_id, user_id, etc.)
   - error details (stack trace สำหรับ errors)
3. เพิ่ม log ที่จุดสำคัญ:
   - Request start/end (with duration)
   - External service calls (with duration)
   - Business events (order created, payment processed)
   - Error occurrences
4. ไม่ log sensitive data (passwords, tokens, PII)
```

---

**OPS-14.** สร้าง Health check endpoints
> endpoint สำหรับ monitoring

```
สร้าง health check endpoints:

1. **GET /health** (basic liveness)
   - Return 200 OK ถ้า service ยังทำงาน

2. **GET /health/ready** (readiness)
   - ตรวจ dependencies:
     - Database connection
     - Cache connection
     - External services (optional)
   - Return 200 ถ้าพร้อมรับ traffic
   - Return 503 ถ้ายังไม่พร้อม

3. **GET /health/detail** (detailed, internal only)
   - Version / build info
   - Uptime
   - Memory usage
   - Dependency status
   - ป้องกัน access จาก public

ตาม pattern ที่โปรเจกต์ใช้ (handler → service)
```

---

**OPS-15.** เพิ่ม Metrics collection
> เตรียม metrics สำหรับ monitoring dashboard

```
เพิ่ม metrics collection ใน [module/service]:
1. **Request metrics:**
   - request_count (by method, path, status)
   - request_duration_seconds (histogram)
   - request_size_bytes

2. **Business metrics:**
   - [business event]_total (e.g., orders_created_total)
   - [business event]_duration_seconds

3. **Resource metrics:**
   - db_connection_pool_size
   - cache_hit_ratio
   - goroutine/thread count

4. **Error metrics:**
   - error_total (by type, severity)

ใช้ [Prometheus / StatsD / OpenTelemetry] format
สร้าง middleware สำหรับ automatic request metrics
```

---

**OPS-16.** วิเคราะห์ logs
> หา patterns และ anomalies จาก logs

```
วิเคราะห์ log patterns ใน [log file / log output]:
1. Error frequency — errors ที่เกิดบ่อยที่สุด
2. Error patterns — errors เกิดเป็น cluster ช่วงเวลาไหน?
3. Slow requests — requests ที่ช้ากว่าปกติ
4. Unusual patterns — traffic patterns ที่ผิดปกติ
5. Correlation — errors ที่เกิดพร้อมกันมีความเกี่ยวข้องไหม?

สรุป:
- Top 5 issues ที่ต้องแก้
- Root cause hypothesis
- Recommended actions
```

---

## 6.5 Incident Response

**OPS-17.** วิเคราะห์ production incident
> แก้ปัญหา production เร่งด่วน

```
🚨 Production incident:
ปัญหา: [อธิบายอาการ]
Impact: [กระทบ users/features อะไร]
เริ่มเมื่อ: [เวลา]

ช่วยวิเคราะห์:
1. ดู error logs / recent changes (git log -10)
2. หา root cause — deploy ล่าสุดเปลี่ยนอะไร?
3. แนะนำ:
   - Immediate fix (hotfix/rollback?)
   - สิ่งที่ต้องตรวจเพิ่ม
   - Communication template สำหรับแจ้งทีม/users
```

💡 ใช้กับ `claude -c` เพื่อ iterate ได้ต่อเนื่อง

---

**OPS-18.** สร้าง Hotfix
> แก้ไขด่วนสำหรับ production

```
สร้าง hotfix สำหรับ production issue:
ปัญหา: [อธิบาย bug/issue]

1. สร้าง branch `hotfix/[description]` จาก [production branch]
2. หา root cause ใน code
3. แก้ไขแบบ minimal — แก้เฉพาะ bug ไม่ refactor
4. เขียน test ที่ reproduce bug + verify fix
5. สร้าง PR description:
   - Root cause
   - Fix approach
   - Risk assessment
   - Rollback plan
```

---

**OPS-19.** สร้าง Post-Incident Review
> เรียนรู้จาก incident เพื่อป้องกัน

```
สร้าง Post-Incident Review (PIR) สำหรับ incident:
Incident: [อธิบายสั้นๆ]

Template:
1. **Timeline:**
   - [time] — detected
   - [time] — investigation started
   - [time] — root cause identified
   - [time] — fix deployed
   - [time] — resolved

2. **Root Cause:** (อะไรเป็นสาเหตุจริงๆ)

3. **Contributing Factors:** (อะไรทำให้ impact แย่ลง)

4. **What Went Well:** (อะไรช่วยให้แก้ได้เร็ว)

5. **What Could Be Improved:**

6. **Action Items:**
   | Action | Owner | Priority | Deadline |
   |--------|-------|----------|----------|
   | ... | ... | ... | ... |
```

---

**OPS-20.** สร้าง Runbook
> คู่มือ step-by-step สำหรับ common operations

```
สร้าง runbook สำหรับ [operation/scenario]:
ตัวอย่าง scenarios:
- Deploy to production
- Rollback deployment
- Database migration
- Scale up/down services
- Rotate secrets/credentials
- Handle [specific incident type]

Format:
1. **Trigger:** เมื่อไหร่ต้องทำ
2. **Prerequisites:** ต้องมี access/tools อะไร
3. **Steps:** (numbered, ทำตามได้ทันที)
4. **Verification:** ตรวจสอบว่าสำเร็จยังไง
5. **Rollback:** ถ้าพังจะ rollback ยังไง
6. **Contacts:** ถ้าติดปัญหาถามใคร
```

---

**OPS-21.** สร้าง Rollback plan
> เตรียมแผนถอยกลับก่อน deploy

```
สร้าง rollback plan สำหรับ deployment [version/feature]:
1. **Pre-deployment checklist:**
   - Database backup?
   - Current version noted?
   - Rollback tested?

2. **Rollback triggers:** (เมื่อไหร่ต้อง rollback)
   - Error rate > [N]%
   - Response time > [N]ms
   - Health check fails
   - [business-specific trigger]

3. **Rollback steps:**
   - [step-by-step rollback procedure]
   - Database rollback needed? (migration down)
   - Cache invalidation needed?
   - Feature flag to disable?

4. **Post-rollback verification:**
   - Health check passes?
   - Key flows working?
   - Data integrity check?

5. **Communication:**
   - Who to notify
   - Status page update template
```

---

> **ดูเพิ่มเติม:**
> - [05_quality_and_security.md](05_quality_and_security.md) — Security audit และ performance analysis
> - [02_developer_workflows.md](02_developer_workflows.md) — Git workflows สำหรับ developers
> - [examples/07_prompt_catalog.md](../examples/07_prompt_catalog.md) — ตัวอย่าง DevOps prompts เฉพาะโปรเจกต์ ShopFast (O1-O13)
