# Phase 1: Inception — โปรเจกต์เริ่มต้น

> **ระยะเวลา:** สัปดาห์ที่ 1
> **เป้าหมาย:** วางรากฐานทุกอย่างก่อนเขียนโค้ดแม้แต่บรรทัดเดียว —
> สถาปัตยกรรม, มาตรฐานโค้ด, specifications, dev environment, และ permission profiles

---

## ภาพรวม Phase นี้

Phase 1 คือสัปดาห์ที่สำคัญที่สุดของโปรเจกต์ ShopFast ทุกอย่างที่ทำใน phase นี้
จะกำหนดทิศทางของ phase ที่เหลือทั้งหมด ถ้า inception ดี — โค้ดจะสม่ำเสมอ
ทีมจะเข้าใจตรงกัน Claude Code จะทำงานได้แม่นยำ

เป้าหมายหลัก 4 ข้อ:

1. **Architecture ชัดเจน** — ทุกคนรู้ว่าระบบมีกี่ components, สื่อสารกันอย่างไร
2. **CLAUDE.md พร้อมใช้** — Claude Code ทุก session จะเข้าใจบริบทของ ShopFast ตั้งแต่เปิดมา
3. **Specifications ครบ** — BA เขียน spec สำหรับ core features, SA review แล้ว
4. **Dev environment ทำงานได้** — `make dev` แล้วทุกอย่างพร้อม

---

## ใครทำอะไร

| 👤 Role | คน | งานหลักใน Phase 1 |
|---------|------|-------------------|
| SA | คุณเอส | ออกแบบ architecture, สร้าง CLAUDE.md, เขียน ADR, ตั้ง permission profiles |
| BA | คุณบี | เขียน product spec, user stories, acceptance criteria, API contract draft |
| PM | คุณแพร | กำหนด scope, review specs + architecture, approve ก่อนเข้า Phase 2 |
| DevOps | คุณดีบี | สร้าง repo structure, Docker Compose, Makefile, GitHub repo setup |
| Backend Dev | คุณแบ็ค | (สังเกตการณ์ + ให้ feedback เรื่อง tech stack) |
| Frontend Dev | คุณฟร้อนท์ | (สังเกตการณ์ + ให้ feedback เรื่อง admin dashboard scope) |
| QA | คุณคิว | (สังเกตการณ์ + เตรียม test strategy draft) |

---

## Step 1: SA ออกแบบ High-Level Architecture

👤 **คุณเอส (SA)** เริ่มต้นวันแรกของ ShopFast

วันที่ 1 ของโปรเจกต์ คุณเอสเปิด terminal ที่โฟลเดอร์ว่างที่เพิ่ง `git init` มา
แล้วเปิด Claude Code ด้วย `claude --model opus` เพราะงาน architecture ต้องการ
ความคิดเชิงลึก

```
ออกแบบ high-level architecture สำหรับระบบ ShopFast E-commerce:

Components:
- REST API (Go 1.22 + Fiber v2 + GORM v2)
- Admin Dashboard (React 18 + TypeScript 5 + Tailwind CSS v3)
- PostgreSQL 16 เป็น primary database
- Redis 7 สำหรับ caching + session + rate limiting
- GitHub Actions สำหรับ CI/CD

Requirements:
- Three-layer architecture: Handler → Service → Repository
- ระบบรองรับ ~10,000 users, ~1,000 orders/day
- Background jobs สำหรับ order processing, email notifications
- RESTful API versioning (/api/v1/)

ผลลัพธ์ที่ต้องการ:
1. Component diagram แบบ C4 model (Level 1 + Level 2) เป็น ASCII
2. รายการ components พร้อม technology ที่เลือกและเหตุผล
3. Communication patterns ระหว่าง components
4. โครงสร้าง directory ของ repository
```

Claude Code จะตอบกลับด้วย component diagram, tech decisions, และ directory structure
ที่เหมาะกับขนาดโปรเจกต์ คุณเอส review ผลลัพธ์แล้วปรับแต่งตามประสบการณ์ของทีม

💡 **เคล็ดลับ:** ใช้ `Shift+Tab` เพื่อเข้า Plan Mode ก่อน — ให้ Claude Code วิเคราะห์
และเสนอแนวทาง โดยยังไม่สร้างไฟล์จริง จนกว่าจะ approve แผน

📌 **Cross-ref:** ดูรายละเอียดการใช้ Claude Code เป็น Architecture Assistant ใน
[บทที่ 6 — หัวข้อ 6.1](../06_architecture_design.md)

---

## Step 2: SA สร้าง CLAUDE.md

👤 **คุณเอส (SA)** — ยังอยู่ใน session เดิม

หลังจากได้ architecture draft แล้ว คุณเอสสร้าง CLAUDE.md ซึ่งเป็นไฟล์ที่ Claude Code
จะอ่านทุกครั้งที่เปิด session ไม่ว่าใครในทีมจะเปิดก็ตาม

```
สร้าง CLAUDE.md สำหรับโปรเจกต์ ShopFast โดยมีหัวข้อต่อไปนี้:

1. Project Overview — tech stack ทั้งหมด (Go/Fiber/GORM, React/TS/Tailwind,
   PostgreSQL, Redis)
2. Architecture Rules — three-layer (handler→service→repository),
   ข้อห้ามข้ามชั้น (handler ห้ามเรียก repository ตรง, service ห้าม import fiber)
3. Coding Standards — gofmt, error wrapping ด้วย %w, table-driven tests,
   React functional components only, TS strict mode
4. Directory Structure — internal/ (handler, service, repository, model, dto),
   web/src/, docs/ (specs, adr)
5. Commands — build, test, lint, frontend dev, docker compose
6. Conventions — /api/v1/, soft delete, response format, error format,
   conventional commits (feat:, fix:, refactor:, docs:, test:)
```

Claude Code จะสร้างไฟล์ `CLAUDE.md` ที่ root ของโปรเจกต์ คุณเอส review และปรับให้ตรง
กับ conventions ของทีม จากนั้น commit:

```
git add CLAUDE.md && git commit -m "docs: add CLAUDE.md with project conventions"
```

⚠️ **ข้อควรระวัง:** อย่าใส่ secrets, API keys, หรือ passwords ใน CLAUDE.md เด็ดขาด
เพราะไฟล์นี้ถูก commit ลง git — ทุกคนในทีมจะเห็น

💡 **เคล็ดลับ:** CLAUDE.md ที่ดีไม่ได้ยาวที่สุด แต่มีข้อมูลที่ "ทำให้ Claude Code
ไม่ต้องเดา" — architecture rules, naming conventions, คำสั่ง build/test เป็นสิ่งที่สำคัญที่สุด

📌 **Cross-ref:** ดูรายละเอียดเรื่อง CLAUDE.md, .claude/rules/, .claude/skills/ ใน
[บทที่ 1 — หัวข้อ 1.6](../01_intro.md)

---

## Step 3: BA เขียน Product Specification

👤 **คุณบี (BA)** — วันที่ 2

คุณแพร (PM) ส่ง high-level requirements มาให้คุณบีเป็น bullet points ใน Slack:
"อยากได้ระบบจัดการสินค้า, ระบบ user + auth, ระบบออเดอร์ + payment, admin dashboard"

คุณบีเปิด Claude Code แล้วเริ่มแปลง requirements เป็น structured spec:

```
ช่วยสร้าง feature specification สำหรับ ShopFast E-commerce
โดยใช้ template นี้สำหรับแต่ละ feature:

Template:
- Objective (1-2 ประโยค)
- User Stories (As a..., I want..., So that...)
- Business Rules (เงื่อนไขทางธุรกิจ)
- API Contract (method, URL, request/response JSON)
- Edge Cases
- Acceptance Criteria (checklist)
- Dependencies

Features ที่ต้องสร้าง spec:
1. User Registration & Authentication (JWT)
2. Product Management (CRUD + categories + search)
3. Order Processing (create order, payment, status tracking)
4. Admin Dashboard (user management, order management, product management)

เริ่มจาก Feature 1: User Registration & Authentication
บันทึกเป็นไฟล์ docs/specs/user-auth.md
```

Claude Code จะสร้าง spec ที่มี user stories, business rules, API contracts พร้อม
request/response JSON, edge cases, และ acceptance criteria ครบถ้วน

คุณบีทำซ้ำกับ feature ที่เหลือ:

```
สร้าง spec สำหรับ Feature 2: Product Management
อ้างอิง format เดียวกับ docs/specs/user-auth.md
บันทึกเป็น docs/specs/product-management.md

Business requirements:
- สินค้ามี categories แบบ hierarchical (parent → child)
- รองรับ product variants (size, color)
- search ด้วยชื่อสินค้า + filter ตาม category, price range
- pagination สำหรับ listing
- soft delete (admin ลบแล้วยังกู้คืนได้)
```

💡 **เคล็ดลับ:** ให้ Claude Code อ้างอิง spec ก่อนหน้าเสมอ ("อ้างอิง format เดียวกับ...")
เพื่อให้ได้ format ที่สม่ำเสมอ ไม่ต้องปรับ template ทุกรอบ

📌 **Cross-ref:** ดู spec template, ตัวอย่าง simple/complex spec, และ workflow จาก
spec สู่ code ใน [บทที่ 5 — Specification-Driven Development](../05_spec_driven_dev.md)

---

## Step 4: SA เขียน Architecture Decision Records (ADRs)

👤 **คุณเอส (SA)** — วันที่ 2–3

หลังจากทีมถกกันเรื่อง tech stack ในวัน kickoff คุณเอสต้องบันทึกเหตุผลของการตัดสินใจ
เป็น ADR เพื่อให้คนที่มาทีหลัง (หรือตัวเองใน 6 เดือน) เข้าใจว่า "ทำไมถึงเลือก X แทน Y"

คุณเอสเปิด Claude Code แล้วสร้าง ADR แรก:

```
สร้าง ADR-001 เรื่องการเลือก Fiber แทน Gin สำหรับ Go web framework

Context:
- ทีมมี 2 backend devs คุ้นเคย Node.js/Express
- ต้องการ framework ที่ API คล้าย Express
- Performance สำคัญ (target: <100ms response time p95)
- ต้องรองรับ middleware chain + route grouping

Alternatives:
1. Fiber v2 (Express-inspired, fasthttp based)
2. Gin (most popular Go framework, net/http based)
3. Echo (minimalist, similar to Gin)

ใช้ ADR format: Status, Context, Decision, Consequences (positive/negative/risks),
Alternatives Considered

บันทึกที่ docs/adr/001-fiber-over-gin.md
```

คุณเอสทำซ้ำสำหรับ decisions อื่นด้วย pattern เดียวกัน — ระบุ Context, Alternatives,
แล้วให้ Claude Code สร้าง ADR format เต็ม:

```
สร้าง ADR-002: เลือก GORM เป็น ORM หลัก
Context: ทีมคุ้นเคย ORM จากภาษาอื่น, CRUD ~80%, ต้องการ auto-migration + soft delete
Alternatives: GORM v2 / sqlx + raw SQL / Ent / SQLBoiler
บันทึกที่ docs/adr/002-gorm-as-orm.md
```

```
สร้าง ADR-003: เลือก PostgreSQL แทน MySQL
Context: ต้องการ JSONB fields, full-text search, ทีม DBA คุ้นเคย PostgreSQL
Alternatives: PostgreSQL 16 / MySQL 8 / CockroachDB
บันทึกที่ docs/adr/003-postgresql-over-mysql.md
```

💡 **เคล็ดลับ:** ส่วนที่สำคัญที่สุดของ ADR คือ "Context" และ "Alternatives Considered"
ไม่ใช่ตัว Decision — เพราะคนที่มาอ่านทีหลังต้องการรู้ว่า "มีทางเลือกอะไรบ้าง
และทำไมถึงไม่เลือกทางอื่น"

📌 **Cross-ref:** ดู ADR template, ตัวอย่าง ADR เต็มรูปแบบ, และ prompt สำหรับสร้าง ADR
ใน [บทที่ 6 — หัวข้อ 6.6 Architecture Decision Records](../06_architecture_design.md)

---

## Step 5: DevOps ตั้ง Dev Environment

👤 **คุณดีบี (DevOps)** — วันที่ 3–4

คุณดีบีรับ architecture diagram จากคุณเอส แล้วเริ่มสร้าง development environment
ที่ทุกคนในทีมจะใช้ เป้าหมาย: ใครก็ตามที่ clone repo แล้ว `make dev`
ต้องได้ทุกอย่างพร้อมภายใน 2 นาที

```
สร้าง Docker Compose สำหรับ ShopFast development environment:

Services:
1. api - Go API (hot reload ด้วย air)
   - port: 8080
   - mount: ./cmd, ./internal, ./pkg → container
   - depends on: postgres, redis

2. postgres - PostgreSQL 16
   - port: 5432
   - volume: pgdata สำหรับ persist data
   - init script: สร้าง database "shopfast_dev"

3. redis - Redis 7
   - port: 6379

4. web - React Admin Dashboard (Vite dev server)
   - port: 5173
   - mount: ./web/src → container

Environment variables ใช้ .env file
สร้าง .env.example ด้วย (ไม่มี secrets จริง)

บันทึกที่ docker-compose.yml และ .env.example
```

จากนั้นคุณดีบีสร้าง Makefile:

```
สร้าง Makefile สำหรับ ShopFast:
- Development: make dev (compose up), make down, make logs
- Backend: make build, make test, make lint, make migrate-up/down
- Frontend: make web-dev, make web-build
- Utility: make seed (sample data), make clean
รวม .PHONY declarations — บันทึกที่ Makefile
```

สุดท้ายคุณดีบีสร้างโครงสร้าง repository ที่สอดคล้องกับ architecture ใน CLAUDE.md:

```
สร้าง directory structure สำหรับ ShopFast repository ตาม CLAUDE.md:
- cmd/api/main.go (placeholder), internal/ (handler, service, repository,
  model, dto, middleware, config), web/src/ (components, pages, hooks,
  types, lib), docs/ (specs, adr), migrations/, scripts/, .github/workflows/
สร้าง .gitignore (Go + Node.js + Docker) และ .gitkeep ในแต่ละ directory
```

💡 **เคล็ดลับ:** ให้ Claude Code สร้าง `.env.example` แยกจาก `.env` เสมอ —
commit `.env.example` ลง git แต่ใส่ `.env` ใน `.gitignore`

📌 **Cross-ref:** ดู Docker multi-stage build, docker-compose patterns, และ CI/CD setup
ใน [บทที่ 4 — หัวข้อ 4.4 CI/CD และ 4.5 Docker & Deployment](../04_fullstack_collaboration.md)

---

## Step 6: PM Review และ Approve

👤 **คุณแพร (PM)** — วันที่ 4–5

คุณแพรไม่ได้เขียนโค้ด แต่ต้อง review ทุกอย่างก่อนที่ทีมจะเริ่ม Phase 2
คุณแพรเปิด Claude Code ในโหมดอ่านอย่างเดียว (read-only permissions)

```
อ่าน specs ทั้งหมดใน docs/specs/ แล้วสรุปให้หน่อย:

1. แต่ละ spec ครอบคลุม feature อะไรบ้าง
2. acceptance criteria ครบถ้วนหรือยัง
3. มี feature ไหนที่ overlap กัน
4. มี dependency ระหว่าง features อย่างไร
5. ประเมินลำดับการ implement ที่เหมาะสม
```

คุณแพรยังใช้ Claude Code ช่วย review architecture decisions และสร้างสรุปสำหรับ kickoff:

```
อ่าน docs/specs/ และ docs/adr/ ทั้งหมด แล้วสร้าง project brief:
- Scope: features ที่จะทำ พร้อม dependencies ระหว่าง features
- Tech decisions: สรุปจาก ADRs พร้อมความเสี่ยงที่ระบุไว้
- ลำดับ implement ที่เหมาะสม
- Open questions: สิ่งที่ยังต้องตัดสินใจ
```

💡 **เคล็ดลับ:** PM ควรใช้ Claude Code ในโหมด read-only เพื่อป้องกันการแก้ไขโค้ด
โดยไม่ตั้งใจ — ตั้ง permissions ให้มีแค่ Read, Glob, Grep และ git log/diff/status

📌 **Cross-ref:** ดู permission settings สำหรับ PM ใน
[บทที่ 8 — หัวข้อ 8.4 PM](../08_roles_responsibilities.md)

---

## Step 7: SA ตั้ง Permission Profiles

👤 **คุณเอส (SA)** — วันที่ 5

ก่อนจบ Phase 1 คุณเอสต้องตั้ง permission profiles สำหรับแต่ละ role
เพื่อให้ Claude Code ของแต่ละคนทำได้เฉพาะสิ่งที่ role นั้นรับผิดชอบ

```
สร้าง .claude/settings.json สำหรับโปรเจกต์ ShopFast

ตั้ง permissions กลางที่ทุก role ใช้ร่วมกัน:

Allow ทุกคน:
- Read, Glob, Grep (อ่านไฟล์ได้หมด)
- Bash(go build ./...) — build ได้
- Bash(go test ./...) — test ได้
- Bash(npm run lint) — lint ได้

Deny ทุกคน:
- Bash(rm -rf*) — ห้ามลบแบบ recursive
- Bash(git push --force*) — ห้าม force push
- Write(.env*) — ห้ามแก้ .env files
- Bash(docker system prune*) — ห้ามลบ Docker resources

บันทึกที่ .claude/settings.json
```

จากนั้นคุณเอสสร้างเอกสารแนะนำ `settings.local.json` สำหรับแต่ละ role:

```
สร้างตัวอย่าง .claude/settings.local.json สำหรับ 5 roles:
- Backend Dev: allow Edit/Write(internal/**), deny Edit(web/src/**)
- Frontend Dev: allow Edit/Write(web/src/**), deny Edit(internal/**)
- BA: allow Edit/Write(docs/specs/**), deny Edit(internal/** + web/src/**)
- PM: deny Edit(**), Write(**) — read-only ทั้งหมด
- QA: allow Edit/Write(**/*_test.go + **/*.test.ts), deny Edit production code
สร้างเป็นไฟล์ตัวอย่างใน docs/ (ให้แต่ละคน copy ไป .claude/settings.local.json เอง)
```

⚠️ **ข้อควรระวัง:** `.claude/settings.json` ถูก commit ลง git (ใช้ร่วมกันทั้งทีม)
แต่ `.claude/settings.local.json` ต้องอยู่ใน `.gitignore` (เฉพาะคน)
อย่าสลับกัน!

📌 **Cross-ref:** ดู permission settings แนะนำสำหรับแต่ละ role ใน
[บทที่ 8 — Roles & Responsibilities](../08_roles_responsibilities.md) ทุกหัวข้อ (8.1–8.6)

---

## ✅ Checkpoint — ก่อนไป Phase 2

ก่อนเริ่ม Phase 2 (Database & Backend) ทีมต้องตรวจสอบว่าทุกข้อด้านล่างเสร็จแล้ว:

**Architecture & Standards:**
- [ ] Architecture diagram (C4 Level 1 + Level 2) ถูก review โดยทีม
- [ ] CLAUDE.md อยู่ที่ root ของ repo — ครอบคลุม tech stack, architecture rules, coding standards, commands
- [ ] ADR อย่างน้อย 3 ฉบับ: web framework, ORM, database
- [ ] Directory structure ตรงกับที่ระบุใน CLAUDE.md

**Specifications:**
- [ ] Feature specs อย่างน้อย 3 features หลัก (user auth, products, orders)
- [ ] แต่ละ spec มี: user stories, business rules, API contract, edge cases, acceptance criteria
- [ ] PM review และ approve specs แล้ว

**Dev Environment:**
- [ ] `docker compose up -d` ทำงานได้ — PostgreSQL, Redis พร้อม
- [ ] `make build` compile สำเร็จ (แม้จะเป็น placeholder)
- [ ] `make test` รันได้ (แม้ยังไม่มี test จริง)
- [ ] `.env.example` มีครบทุก environment variables ที่ต้องการ
- [ ] `.gitignore` ครอบคลุม: `.env`, `settings.local.json`, build artifacts, `node_modules`

**Permissions:**
- [ ] `.claude/settings.json` ถูก commit ลง git
- [ ] ตัวอย่าง `settings.local.json` สำหรับแต่ละ role มีอยู่ใน docs/
- [ ] ทุกคนในทีม clone repo แล้วเปิด Claude Code ได้ — CLAUDE.md ถูกอ่านอัตโนมัติ

---

## เคล็ดลับ & ข้อผิดพลาดที่พบบ่อย

### 💡 เคล็ดลับ

1. **CLAUDE.md ไม่ต้องสมบูรณ์แบบตั้งแต่วันแรก** — เขียนสิ่งที่รู้แน่ๆ ก่อน
   แล้วค่อยอัปเดตทุก sprint คุณเอสควร review CLAUDE.md ทุกสิ้นสัปดาห์

2. **ใช้ Opus สำหรับงาน architecture, Sonnet สำหรับงานที่เหลือ** —
   `claude --model opus` ตอนออกแบบ architecture และเขียน ADR
   สลับกลับ `claude --model sonnet` ตอนสร้าง directory structure หรือ Dockerfile

3. **Spec ก่อน Code เสมอ** — อย่าให้ developer เริ่ม implement ก่อนที่ spec จะถูก
   approve เพราะ spec ที่เปลี่ยนทีหลังจะทำให้เสียเวลา rework มากกว่าการรอ 1–2 วัน

4. **ADR ช่วยเวลา onboard คนใหม่** — คนที่เข้ามาทีหลังจะถามว่า "ทำไมไม่ใช้ Gin?"
   ถ้ามี ADR จะตอบได้ทันทีโดยไม่ต้องถาม SA

5. **ให้ Claude Code อ่าน spec ก่อนทำทุกอย่าง** — เมื่อเริ่ม session ใหม่
   พิมพ์ "อ่าน docs/specs/[feature].md ก่อน" เสมอ เพื่อให้ Claude Code มี context ครบ

### ⚠️ ข้อผิดพลาดที่พบบ่อย

1. **CLAUDE.md ไม่ตรงกับ reality** — เขียนว่า "ใช้ three-layer" แต่โค้ดจริง handler
   เรียก repository ตรง ทำให้ Claude Code สร้างโค้ดที่ขัดกับ codebase จริง
   **แก้:** อัปเดต CLAUDE.md ทุกครั้งที่เปลี่ยน convention

2. **Spec ขาด Edge Cases** — spec เขียนแค่ happy path ทำให้ Claude Code
   ไม่ handle error cases
   **แก้:** ใช้ prompt "ตรวจสอบ spec นี้ — มี edge cases ที่ยังไม่ได้ระบุหรือไม่?"

3. **Permission เปิดกว้างเกินไป** — ทุกคนใช้ settings เดียวกัน QA แก้ production code ได้
   **แก้:** ตั้ง `settings.local.json` แยกตาม role ตั้งแต่วันแรก

4. **ข้าม ADR** — ทีมเลือก tech stack โดยไม่บันทึกเหตุผล พอมีคนถามทีหลังจำไม่ได้
   **แก้:** ทำ ADR ทุกครั้งที่มี decision ที่กระทบ architecture

5. **Docker Compose ไม่ทำงานข้ามเครื่อง** — ใช้ absolute path หรือ OS-specific config
   **แก้:** ใช้ relative paths, `.env` file, และทดสอบบน macOS + Linux

---

*[← ก่อนหน้า: Overview](00_overview.md) | [ต่อไป: Phase 2 — Database & Backend →](02_phase2_database_backend.md)*
