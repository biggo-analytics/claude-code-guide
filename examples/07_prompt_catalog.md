# Prompt Catalog — รวม Prompt ทั้งหมดจาก ShopFast Walkthrough

> copy-paste ready — ปรับชื่อโปรเจกต์/module แล้วใช้ได้เลย
> ดูคำสั่ง Claude Code เพิ่มเติมที่ [บทที่ 9](../09_cheat_sheet.md)

## สารบัญ

- [Part 1: จัดตาม Role](#part-1-จัดตาม-role) — PM / BA / SA / Backend / Frontend / DBA-DevOps / QA
- [Part 2: จัดตาม Phase](#part-2-จัดตาม-phase) — Phase 1–6 ตาราง quick-ref
- [สถิติ](#สถิติ)

---

## Part 1: จัดตาม Role

### PM (คุณแพร)

**P1.** Review specs — Phase 1
```
อ่าน specs ทั้งหมดใน docs/specs/ แล้วสรุป: แต่ละ spec ครอบคลุมอะไร, acceptance criteria ครบไหม, มี overlap/dependency อย่างไร, ลำดับ implement ที่เหมาะสม
```
> สรุป specs + หา gaps ก่อน approve — [01_phase1_inception.md](01_phase1_inception.md)

**P2.** Project brief — Phase 1
```
อ่าน docs/specs/ และ docs/adr/ ทั้งหมด แล้วสร้าง project brief: Scope, Tech decisions สรุปจาก ADRs, ลำดับ implement, Open questions
```
> สร้าง project brief สำหรับ kickoff — [01_phase1_inception.md](01_phase1_inception.md)

**P3.** Sprint report — Phase 2
```
สรุป progress ของ [โปรเจกต์] Phase [N] ในรูปแบบ sprint report: Done, Blockers, Next, Risks
```
> สรุป sprint status — [02_phase2_database_backend.md](02_phase2_database_backend.md)

**P4.** Release checklist — Phase 5
```
อ่าน specs ทั้งหมด แล้วสร้าง release checklist: แต่ละ spec list acceptance criteria, ตรวจว่ามี implementation + test coverage + API docs, สรุปเป็นตาราง, ระบุ blockers
```
> ตรวจ spec coverage ก่อน release — [05_phase5_integration_cicd.md](05_phase5_integration_cicd.md)

**P5.** Verify spec vs tests — Phase 5
```
เปรียบเทียบ acceptance criteria จาก docs/specs/[feature].md กับ test cases ใน tests/ — criteria ไหนมี test แล้ว? สร้างรายการ gaps
```
> Cross-ref specs กับ tests — [05_phase5_integration_cicd.md](05_phase5_integration_cicd.md)

**P6.** Release notes — Phase 5
```
จาก git log และ specs ที่ implement แล้ว สร้าง release notes v[X.Y.Z]: Summary, New Features, API Endpoints, Known Issues, Deployment Notes
```
> Draft release notes — [05_phase5_integration_cicd.md](05_phase5_integration_cicd.md)

---

### BA (คุณบี)

**B1.** Feature spec (template) — Phase 1
```
สร้าง feature specification โดยใช้ template: Objective, User Stories (As a... I want... So that...), Business Rules, API Contract (method/URL/JSON), Edge Cases, Acceptance Criteria, Dependencies — เริ่มจาก Feature 1: [ชื่อ] บันทึกที่ docs/specs/[name].md
```
> สร้าง spec ครบ template — [01_phase1_inception.md](01_phase1_inception.md)

**B2.** Spec ต่อ (อ้างอิง format เดิม) — Phase 1
```
สร้าง spec สำหรับ Feature [N]: [ชื่อ] อ้างอิง format เดียวกับ docs/specs/[feature-ก่อนหน้า].md — Business requirements: [list]
```
> สร้าง spec ต่อ ใช้ format เดิม — [01_phase1_inception.md](01_phase1_inception.md)

**B3.** New feature spec (maintenance) — Phase 6
```
เขียน spec สำหรับระบบ [feature ใหม่]: requirement หลัก [list] — เขียนเป็น markdown ที่มี data model, API endpoints, business rules
```
> เขียน spec สำหรับ feature ใหม่ — [06_phase6_maintenance.md](06_phase6_maintenance.md)

---

### SA (คุณเอส)

**S1.** High-level architecture — Phase 1
```
ออกแบบ high-level architecture: Components [list + tech], Requirements [pattern, scale, jobs, versioning] — ผลลัพธ์: C4 diagram (ASCII), tech decisions + เหตุผล, communication patterns, directory structure
```
> ออกแบบ architecture ทั้งระบบ — [01_phase1_inception.md](01_phase1_inception.md)

**S2.** สร้าง CLAUDE.md — Phase 1
```
สร้าง CLAUDE.md: Project Overview (tech stack), Architecture Rules (layer ข้อห้าม), Coding Standards, Directory Structure, Commands (build/test/lint), Conventions (API format, commits)
```
> สร้าง CLAUDE.md ให้ Claude Code มี context — [01_phase1_inception.md](01_phase1_inception.md)

**S3.** ADR — Phase 1
```
สร้าง ADR-[NNN] เรื่อง [decision] — Context: [บริบท], Alternatives: [options] — ใช้ format: Status, Context, Decision, Consequences, Alternatives Considered — บันทึกที่ docs/adr/[filename].md
```
> สร้าง ADR บันทึก technical decisions — [01_phase1_inception.md](01_phase1_inception.md)

**S4.** Permission profiles (shared) — Phase 1
```
สร้าง .claude/settings.json: Allow ทุกคน [Read/Glob/Grep, build, test, lint] — Deny ทุกคน [rm -rf, git push --force, Write .env, docker prune]
```
> ตั้ง shared permissions — [01_phase1_inception.md](01_phase1_inception.md)

**S5.** Role-specific permissions — Phase 1
```
สร้างตัวอย่าง .claude/settings.local.json สำหรับ Backend (allow internal/), Frontend (allow web/src/), BA (allow docs/), PM (read-only), QA (allow test files)
```
> สร้าง permission profiles แยกตาม role — [01_phase1_inception.md](01_phase1_inception.md)

**S6.** Review code structure — Phase 2
```
Review โครงสร้าง project ตาม [architecture] principles: layer separation, dependency direction, interface usage, naming, error handling, circular dependency — อ่านไฟล์ทั้งหมดแล้วสรุปผล
```
> SA review ตาม architecture principles — [02_phase2_database_backend.md](02_phase2_database_backend.md)

**S7.** ADR จาก incident — Phase 3
```
เขียน ADR: Title [decision], Context [ปัญหาที่พบ], Decision [solution], Alternatives [options + tradeoffs], Consequences [rules, risks, performance]
```
> ADR สำหรับ decisions จาก bugs — [03_phase3_core_features.md](03_phase3_core_features.md)

**S8.** Review deployment — Phase 5
```
Review docker-compose + Dockerfiles: Security (secret leak, root?), Networking (expose?), Persistence (data loss?), Health checks, Resource limits — รายงานเป็นตาราง
```
> Review Docker setup ก่อน go-live — [05_phase5_integration_cicd.md](05_phase5_integration_cicd.md)

**S9.** Impact analysis — Phase 6
```
ดูสถาปัตยกรรมปัจจุบัน แล้ววิเคราะห์ว่าการเพิ่ม [feature ใหม่] จะกระทบส่วนไหนบ้าง ต้องแก้อะไรใน services, schema, API
```
> วิเคราะห์ impact ก่อน implement — [06_phase6_maintenance.md](06_phase6_maintenance.md)

**S10.** วิเคราะห์ responsibilities — Phase 6
```
วิเคราะห์ไฟล์ [path] ที่มีกว่า [N] บรรทัด — ระบุ responsibilities, จำนวน methods ต่อกลุ่ม, แนะนำการแยก service ตาม Single Responsibility Principle
```
> วิเคราะห์ก่อน refactor — [06_phase6_maintenance.md](06_phase6_maintenance.md)

**S11.** วางแผน refactoring — Phase 6
```
วางแผน refactor [file]: แยกเป็น [N] services, เรียกกันผ่าน interface, ไม่เปลี่ยน API contract — เขียน step-by-step plan ก่อน ยังไม่ implement
```
> วางแผน refactoring — [06_phase6_maintenance.md](06_phase6_maintenance.md)

**S12.** Onboarding checklist — Phase 6
```
เพิ่ม "Onboarding Checklist" ใน CLAUDE.md: วิธี setup, คำสั่ง test, architecture overview, conventions, ไฟล์สำคัญ
```
> อัปเดต CLAUDE.md สำหรับ onboarding — [06_phase6_maintenance.md](06_phase6_maintenance.md)

---

### Backend Dev (คุณแบ็ค)

**D1.** Authentication service — Phase 3
```
สร้างระบบ authentication: model (RegisterRequest, LoginRequest, TokenResponse, Claims), service (Register + bcrypt, Login + JWT, RefreshToken), JWT secret จาก config, ใช้ pattern เดียวกับ service ที่มีอยู่
```
> สร้าง auth service ตาม existing pattern — [03_phase3_core_features.md](03_phase3_core_features.md)

**D2.** Auth middleware — Phase 3
```
สร้าง auth middleware: ดึง token จาก Authorization header, validate JWT, เก็บ user_id/role ใน context, RequireRole middleware, HTTP handlers, routes
```
> JWT middleware + role-based access — [03_phase3_core_features.md](03_phase3_core_features.md)

**D3.** Refresh token (Redis) — Phase 3
```
เพิ่ม refresh token storage ด้วย Redis: key pattern "refresh:{user_id}:{token_id}", TTL ตรงกับ expiry, token rotation เมื่อ refresh, ลบเมื่อ logout
```
> Refresh token storage + rotation — [03_phase3_core_features.md](03_phase3_core_features.md)

**D4.** Order workflow — Phase 3
```
สร้าง order workflow ตาม business rules [paste rules จาก BA]: Checkout (validate stock, reserve, create order), ConfirmPayment (finalize stock), CancelOrder (restore stock) — ใช้ DB transaction, validate status transitions
```
> Complex business logic จาก BA specs — [03_phase3_core_features.md](03_phase3_core_features.md)

**D5.** Product search (tsvector) — Phase 3
```
เพิ่ม product search ด้วย PostgreSQL full-text search: tsvector column + trigger + GIN index, SearchProducts รับ query/filters/sort/pagination
```
> Full-text search ด้วย PostgreSQL — [03_phase3_core_features.md](03_phase3_core_features.md)

**D6.** Unit tests (table-driven) — Phase 3
```
เขียน unit tests สำหรับ [service]: table-driven pattern, mock repository ด้วย interface, test cases [list], ใช้ testify/assert + testify/mock
```
> Table-driven tests + mocks — [03_phase3_core_features.md](03_phase3_core_features.md)

**D7.** Status transition tests — Phase 3
```
เพิ่ม tests สำหรับ CanTransitionTo(): test valid transitions [list ✓], test invalid transitions [list ✗] — table-driven
```
> Test state machine — [03_phase3_core_features.md](03_phase3_core_features.md)

**D8.** Debug race condition — Phase 3
```
มี bug: [symptom] — อ่าน [file] ฟังก์ชัน [name] วิเคราะห์: flow ของ operation, หา race condition, อธิบายว่าทำไม [mechanism] ไม่ทำงาน
```
> วิเคราะห์ race condition — [03_phase3_core_features.md](03_phase3_core_features.md)

**D9.** Fix race condition — Phase 3
```
แก้ race condition: เปลี่ยน repository methods ให้รับ tx *gorm.DB, pass tx จาก Transaction callback, ตรวจว่าทุก operation ใช้ tx ไม่ใช่ s.db
```
> Fix ด้วย transaction parameter — [03_phase3_core_features.md](03_phase3_core_features.md)

**D10.** Project structure — Phase 2
```
สร้างโครงสร้าง Go project ตาม three-layer: cmd/ (entry + migrate), internal/ (handlers, service, repository, models, dto, middleware, config), pkg/ (response, validator, pagination) — interface ทุก layer, standard response format
```
> Scaffold project structure — [02_phase2_database_backend.md](02_phase2_database_backend.md)

**D11.** Base CRUD — Phase 2
```
สร้าง [Resource] CRUD ครบ 3 layers: Repository (CRUD + pagination + filter), Service (business logic + validation), Handler (HTTP endpoints), DTOs — dependency injection ทุก layer
```
> Reference CRUD implementation — [02_phase2_database_backend.md](02_phase2_database_backend.md)

**D12.** Input validation — Phase 2
```
สร้างระบบ validation: ใช้ go-playground/validator, custom validator แปลง errors เป็น field-level format, validation tags ใน DTOs, error response มาตรฐาน
```
> Validation + error format — [02_phase2_database_backend.md](02_phase2_database_backend.md)

**D13.** Apply review feedback — Phase 2
```
ตามผล review: แยก filter building เป็น method, เพิ่ม error wrapping ทุก method, เพิ่ม structured logger (inject ผ่าน constructor), เพิ่ม request ID middleware
```
> Apply SA feedback — [02_phase2_database_backend.md](02_phase2_database_backend.md)

**D14.** วิเคราะห์ production bug — Phase 6
```
ดูโค้ดที่ [จุดที่มีปัญหา] มี [user report] ช่วยหา root cause
```
> หา root cause จาก customer report — [06_phase6_maintenance.md](06_phase6_maintenance.md)

**D15.** Regression test ก่อน fix — Phase 6
```
เขียน test case สำหรับ bug นี้ก่อน fix — ต้องครอบคลุม [boundary cases]
```
> Test-first debugging — [06_phase6_maintenance.md](06_phase6_maintenance.md)

**D16.** Fix + verify — Phase 6
```
fix bug โดยเปลี่ยนเงื่อนไขให้ตรงตาม business rule "[rule]" แล้วรัน test ให้ผ่าน
```
> Fix แล้ว verify — [06_phase6_maintenance.md](06_phase6_maintenance.md)

**D17.** DEVLOG entry — Phase 6
```
เพิ่ม DEVLOG entry: ปัญหา [X], สาเหตุ [Y], วิธีแก้ [Z], เพิ่ม regression test [N] cases
```
> บันทึก DEVLOG — [06_phase6_maintenance.md](06_phase6_maintenance.md)

**D18.** New feature model — Phase 6
```
สร้าง [Feature] model ตาม spec [paste spec] — สร้างใน [path] พร้อม migration
```
> Model + migration จาก spec — [06_phase6_maintenance.md](06_phase6_maintenance.md)

**D19.** New feature service — Phase 6
```
สร้าง [Feature]Service: [Method1], [Method2], [ValidateMethod]
```
> Service layer — [06_phase6_maintenance.md](06_phase6_maintenance.md)

**D20.** New feature endpoints — Phase 6
```
สร้าง API endpoints: POST (admin create), GET (admin list), POST /apply (user action) — เชื่อมกับ existing service
```
> API endpoints เชื่อมกับ existing code — [06_phase6_maintenance.md](06_phase6_maintenance.md)

**D21.** Refactor step-by-step — Phase 6
```
Step [N]: สร้าง [Service] interface + implementation, ย้าย methods จาก [source], เขียน test
```
> Refactor ทีละ step — [06_phase6_maintenance.md](06_phase6_maintenance.md)

**D22.** แก้ N+1 query — Phase 6
```
แก้ N+1 query ใน [repository] — ใช้ Preload ดึง [relation] มาพร้อมกัน
```
> Fix N+1 — [06_phase6_maintenance.md](06_phase6_maintenance.md)

**D23.** เพิ่ม pagination — Phase 6
```
เพิ่ม pagination: query parameter page/limit, default=[N], max=[M]
```
> Pagination — [06_phase6_maintenance.md](06_phase6_maintenance.md)

**D24.** เพิ่ม indexes — Phase 6
```
เพิ่ม database index สำหรับ [fields ที่ค้นหาบ่อย] — เขียนเป็น migration
```
> Indexes สำหรับ performance — [06_phase6_maintenance.md](06_phase6_maintenance.md)

**D25.** Redis cache — Phase 6
```
เพิ่ม Redis cache: key pattern [resource]:page:{page}:limit:{limit}, TTL [N] นาที, invalidate เมื่อ data เปลี่ยน
```
> Cache + invalidation — [06_phase6_maintenance.md](06_phase6_maintenance.md)

**D26.** Performance analysis — Phase 6
```
API [endpoint] ช้า [N] วินาที เมื่อมี [N] records — วิเคราะห์ code ตั้งแต่ handler ถึง repository หา bottleneck
```
> วิเคราะห์ bottleneck — [06_phase6_maintenance.md](06_phase6_maintenance.md)

---

### Frontend Dev (คุณฟร้อนท์)

**F1.** Scaffold project — Phase 4
```
สร้าง React admin dashboard ด้วย Vite + TS + Tailwind: feature-based folders, path aliases, Tailwind custom theme, React Router v6, React Query v5, .env.example
```
> Scaffold frontend — [04_phase4_frontend.md](04_phase4_frontend.md)

**F2.** TS types จาก Go DTOs — Phase 4
```
อ่าน Go DTOs ใน [path] แล้วสร้าง TypeScript interfaces: แปลง types (uint→number, time.Time→string), แยกไฟล์ตาม domain, สร้าง PaginatedResponse<T>
```
> Cross-project type generation — [04_phase4_frontend.md](04_phase4_frontend.md)

**F3.** Sync types — Phase 4
```
เปรียบเทียบ Go struct กับ TypeScript interface — หา fields ที่ไม่ตรงกัน แล้วอัพเดท
```
> Sync types หลัง backend เปลี่ยน — [04_phase4_frontend.md](04_phase4_frontend.md)

**F4.** API client — Phase 4
```
สร้าง Axios instance: baseURL จาก env, request interceptor (JWT), response interceptor (401→login, 500→toast), generic type-safe methods
```
> API client + interceptors — [04_phase4_frontend.md](04_phase4_frontend.md)

**F5.** React Query hooks — Phase 4
```
สร้าง hooks: useList(params), useOne(id), useCreate() mutation + invalidate, useUpdate(), useDelete() — ใช้ types + api client ที่มี
```
> CRUD hooks — [04_phase4_frontend.md](04_phase4_frontend.md)

**F6.** List page — Phase 4
```
สร้าง ListPage: ตาราง, search (debounced), filter dropdown, pagination, ปุ่ม add/edit/delete, loading skeleton, empty state, responsive (table→card)
```
> List page ครบทุก state — [04_phase4_frontend.md](04_phase4_frontend.md)

**F7.** Form (create/edit) — Phase 4
```
สร้าง Form component: React Hook Form + Zod validation, ใช้ได้ทั้ง create/edit, inline validation errors ภาษาไทย, toast notification
```
> Reusable form component — [04_phase4_frontend.md](04_phase4_frontend.md)

**F8.** Detail page — Phase 4
```
สร้าง DetailPage: useOne(id) hook, แสดงรายละเอียด, ปุ่มแก้ไข/ลบ, loading skeleton, 404 handling
```
> Detail page + error states — [04_phase4_frontend.md](04_phase4_frontend.md)

**F9.** Order detail + status timeline — Phase 4
```
สร้าง OrderDetailPage: Status Timeline แนวตั้ง (icon + status + timestamp, current highlight, future greyed), ปุ่ม update status dropdown
```
> Status timeline component — [04_phase4_frontend.md](04_phase4_frontend.md)

**F10.** Admin layout — Phase 4
```
สร้าง AdminLayout: Sidebar (logo, nav, active link, collapsible), Header (breadcrumbs, search, notifications, user), responsive (desktop/tablet/mobile)
```
> Responsive admin layout — [04_phase4_frontend.md](04_phase4_frontend.md)

**F11.** Breadcrumbs — Phase 4
```
สร้าง Breadcrumbs: auto-generate จาก React Router useMatches(), route handle.breadcrumb, clickable links
```
> Auto breadcrumbs — [04_phase4_frontend.md](04_phase4_frontend.md)

**F12.** แก้ responsive — Phase 4
```
ตาราง [Component] ล้นจอบน [N]px — แก้ให้ mobile เปลี่ยนเป็น card layout, ใช้ Tailwind responsive classes
```
> Responsive table→card — [04_phase4_frontend.md](04_phase4_frontend.md)

**F13.** แก้ a11y — Phase 4
```
เพิ่ม aria-label ให้ [elements], แก้ dialog ให้ trap focus (focus on open, tab cycle, escape close, restore focus)
```
> Accessibility fixes — [04_phase4_frontend.md](04_phase4_frontend.md)

**F14.** Admin page (new feature) — Phase 6
```
สร้างหน้า admin จัดการ [feature]: ตาราง list, form สร้างใหม่, แสดงสถานะ — ใช้ pattern เดียวกับหน้าที่มีอยู่
```
> Admin page ตาม existing pattern — [06_phase6_maintenance.md](06_phase6_maintenance.md)

**F15.** Checkout feature — Phase 6
```
เพิ่ม [feature] input ในหน้า checkout: ช่อง input, ปุ่ม action, แสดงผลลัพธ์/error
```
> เพิ่ม feature ใน checkout — [06_phase6_maintenance.md](06_phase6_maintenance.md)

---

### DBA / DevOps (คุณดีบี)

**O1.** ER diagram — Phase 2
```
ออกแบบ ER diagram: entities [list + descriptions], relationships + constraints, soft delete ทุกตาราง — แสดงเป็น text-based พร้อม field types
```
> ER diagram จาก specs — [02_phase2_database_backend.md](02_phase2_database_backend.md)

**O2.** GORM models — Phase 2
```
จาก ER diagram สร้าง GORM models: GORM tags (column, type, not null, unique, index), relationships, json tags, แต่ละ model คนละไฟล์
```
> GORM models — [02_phase2_database_backend.md](02_phase2_database_backend.md)

**O3.** Migration strategy — Phase 2
```
เปรียบเทียบ migration strategy: AutoMigrate vs manual SQL — ด้าน dev convenience, production safety, rollback, team conflicts, seed data
```
> เลือก migration strategy — [02_phase2_database_backend.md](02_phase2_database_backend.md)

**O4.** Migration setup — Phase 2
```
สร้าง migration setup: db connection, AutoMigrate (dev), SQL migrations dir (prod), CLI tool, seed data (admin user, sample data)
```
> Migration infrastructure — [02_phase2_database_backend.md](02_phase2_database_backend.md)

**O5.** Database indexing — Phase 2
```
วิเคราะห์ schema แนะนำ indexes: queries ที่ใช้บ่อย [list] — แนะนำ index type, CREATE INDEX, เหตุผล, trade-off
```
> Index recommendations — [02_phase2_database_backend.md](02_phase2_database_backend.md)

**O6.** Docker Compose (dev) — Phase 1
```
สร้าง Docker Compose: api (hot reload), database (volume), cache, web (dev server) — .env file + .env.example
```
> Dev environment — [01_phase1_inception.md](01_phase1_inception.md)

**O7.** Makefile — Phase 1
```
สร้าง Makefile: dev/down/logs, build/test/lint, migrate-up/down, web-dev/web-build, seed/clean + .PHONY
```
> Common commands — [01_phase1_inception.md](01_phase1_inception.md)

**O8.** Directory structure — Phase 1
```
สร้าง directory structure ตาม CLAUDE.md: [directories] — สร้าง .gitignore + .gitkeep
```
> Repo structure — [01_phase1_inception.md](01_phase1_inception.md)

**O9.** CI pipeline — Phase 5
```
สร้าง GitHub Actions: trigger push/PR to main, jobs: lint → test-unit → test-integration (ต้อง DB+cache services) → build-and-push (main only), caching
```
> CI/CD pipeline — [05_phase5_integration_cicd.md](05_phase5_integration_cicd.md)

**O10.** Dockerfile API — Phase 5
```
Multi-stage Dockerfile: builder (build static binary), runner (distroless), non-root user
```
> Optimized API image — [05_phase5_integration_cicd.md](05_phase5_integration_cicd.md)

**O11.** Dockerfile Frontend — Phase 5
```
Multi-stage Dockerfile: builder (npm ci + build), runner (nginx:alpine + SPA routing config)
```
> Frontend image — [05_phase5_integration_cicd.md](05_phase5_integration_cicd.md)

**O12.** Docker Compose (staging) — Phase 5
```
สร้าง docker-compose.staging.yml: api + frontend + database + cache + nginx reverse proxy — named volumes, health checks, .env
```
> Staging environment — [05_phase5_integration_cicd.md](05_phase5_integration_cicd.md)

**O13.** Production hardening — Phase 5
```
เพิ่ม hardening: resource limits, restart policy, logging driver + rotation, read_only + no-new-privileges
```
> Security hardening — [05_phase5_integration_cicd.md](05_phase5_integration_cicd.md)

---

### QA (คุณคิว)

**Q1.** Test plan — Phase 3
```
อ่าน [service code] + [spec] แล้วสร้าง test plan: markdown table (Test ID, Category, Test Case, Input, Expected, Priority P0/P1/P2) ครอบคลุม [categories + edge cases]
```
> Test plan จาก code + specs — [03_phase3_core_features.md](03_phase3_core_features.md)

**Q2.** Concurrent integration test — Phase 3
```
เขียน integration test: [N] goroutines [action] พร้อมกัน, resource มี [state], expected: 1 succeed + 1 fail — ใช้ database จริง ไม่ mock
```
> Concurrency test — [03_phase3_core_features.md](03_phase3_core_features.md)

**Q3.** Test container setup — Phase 5
```
สร้าง integration test suite: testcontainers-go spin up DB+cache, TestMain setup/teardown, seed helpers, separate test database
```
> Test infrastructure — [05_phase5_integration_cicd.md](05_phase5_integration_cicd.md)

**Q4.** E2E integration tests — Phase 5
```
สร้าง integration tests: register → login → create resource → get → update status — ใช้ httptest + real app, ตรวจ response + DB state, cleanup หลังแต่ละ test
```
> End-to-end tests — [05_phase5_integration_cicd.md](05_phase5_integration_cicd.md)

**Q5.** Test fixtures — Phase 5
```
สร้าง fixture helpers: Seed[Resource](db, count), SeedUser(db, role), CleanAll(db) ตาม FK order, faker data
```
> Reusable fixtures — [05_phase5_integration_cicd.md](05_phase5_integration_cicd.md)

**Q6.** OWASP review — Phase 5
```
Review codebase ตาม OWASP Top 10: Injection, Auth, Sensitive Data, Access Control, CORS/headers, XSS, Dependencies, Logging — รายงานเป็นตาราง PASS/FAIL/WARN
```
> Security review — [05_phase5_integration_cicd.md](05_phase5_integration_cicd.md)

**Q7.** Security test cases — Phase 5
```
สร้าง security tests: SQL Injection (search=' OR 1=1), Auth Bypass (no token→401, expired→401, wrong role→403, tampered JWT→401, access other user→403)
```
> Security tests — [05_phase5_integration_cicd.md](05_phase5_integration_cicd.md)

**Q8.** UI flow testing — Phase 4
```
วิเคราะห์ UI flows สร้าง test cases: CRUD flow, management flow, navigation flow, search/filter flow — แต่ละ flow: steps, expected results, edge cases
```
> UI test cases — [04_phase4_frontend.md](04_phase4_frontend.md)

**Q9.** Accessibility testing — Phase 4
```
ตรวจ accessibility: aria labels, keyboard navigation, color contrast WCAG AA, screen reader roles, focus management — รายงานปัญหา + วิธีแก้
```
> A11y audit — [04_phase4_frontend.md](04_phase4_frontend.md)

**Q10.** Cross-browser testing — Phase 4
```
ตรวจ CSS/JS compatibility: Chrome, Firefox, Safari, Edge — CSS grid/flexbox, backdrop-filter, transitions — รายงาน properties ที่ไม่รองรับ
```
> Browser compatibility — [04_phase4_frontend.md](04_phase4_frontend.md)

**Q11.** Reproduce bug test — Phase 6
```
เขียน test ที่ reproduce bug นี้ก่อน: [bug description] — test ต้อง fail กับ code ปัจจุบัน
```
> Test-first bug reproduction — [06_phase6_maintenance.md](06_phase6_maintenance.md)

---

### New Developer (คุณนิว) — Phase 6

**N1.** เรียนรู้ architecture
```
อธิบายสถาปัตยกรรมของโปรเจกต์นี้: กี่ layer, แต่ละ layer ทำอะไร, framework + library อะไรบ้าง
```

**N2.** Trace request flow
```
flow ของ [operation] เป็นยังไง ตั้งแต่ request เข้ามาจนถึง response — ผ่าน file ไหนบ้าง
```

**N3.** เข้าใจ business rules
```
อธิบาย business rules ของระบบ [feature] ดูจาก code แล้วสรุปว่ามี rule อะไรบ้าง
```

**N4.** Task แรกตาม pattern
```
ดูตัวอย่าง endpoint ที่มีอยู่ แล้วช่วยเขียน endpoint [METHOD] [path] ทำตาม pattern เดียวกัน
```

> ทั้ง 4 prompts ใช้ Claude Code เป็น onboarding buddy — [06_phase6_maintenance.md](06_phase6_maintenance.md)

---

## Part 2: จัดตาม Phase

### Phase 1: Inception — 14 prompts

| ID | Role | Prompt สั้นๆ |
|----|------|-------------|
| S1 | SA | ออกแบบ high-level architecture + C4 diagram |
| S2 | SA | สร้าง CLAUDE.md |
| B1 | BA | Feature spec ด้วย template |
| B2 | BA | Spec ต่อ อ้างอิง format เดิม |
| S3 | SA | ADR (x3: framework, ORM, database) |
| O6 | DevOps | Docker Compose dev |
| O7 | DevOps | Makefile |
| O8 | DevOps | Directory structure |
| P1 | PM | Review specs ทั้งหมด |
| P2 | PM | Project brief |
| S4 | SA | Permission profiles (shared) |
| S5 | SA | Role-specific permissions |

### Phase 2: Database & Backend — 11 prompts

| ID | Role | Prompt สั้นๆ |
|----|------|-------------|
| O1 | DBA | ER diagram |
| O2 | DBA | GORM models |
| O3 | DBA | Migration strategy comparison |
| O4 | DBA | Migration setup (hybrid) |
| D10 | Backend | Project structure scaffold |
| D11 | Backend | Base CRUD (Product) |
| D12 | Backend | Input validation |
| S6 | SA | Review code structure |
| D13 | Backend | Apply SA review feedback |
| O5 | DBA | Database indexing |
| P3 | PM | Sprint report |

### Phase 3: Core Features — 12 prompts

| ID | Role | Prompt สั้นๆ |
|----|------|-------------|
| D1 | Backend | Authentication service |
| D2 | Backend | Auth middleware |
| D3 | Backend | Refresh token (Redis) |
| D4 | Backend | Order workflow |
| D5 | Backend | Product search (tsvector) |
| Q1 | QA | Test plan จาก specs + code |
| D6 | Backend | Unit tests (table-driven) |
| D7 | Backend | Status transition tests |
| Q2 | QA | Integration test (concurrent) |
| D8 | Backend | Debug race condition |
| D9 | Backend | Fix race condition |
| S7 | SA | ADR สำหรับ locking strategy |

### Phase 4: Frontend — 16 prompts

| ID | Role | Prompt สั้นๆ |
|----|------|-------------|
| F1 | Frontend | Scaffold Vite + React + TS + Tailwind |
| F2 | Frontend | TS types จาก Go DTOs |
| F3 | Frontend | Sync types กับ backend |
| F4 | Frontend | API client + interceptors |
| F5 | Frontend | React Query hooks |
| F6 | Frontend | List page |
| F7 | Frontend | Form (create/edit) |
| F8 | Frontend | Detail page |
| F9 | Frontend | Order detail + status timeline |
| F10 | Frontend | Admin layout + navigation |
| F11 | Frontend | Breadcrumbs |
| Q8 | QA | UI flow testing |
| Q9 | QA | Accessibility testing |
| Q10 | QA | Cross-browser testing |
| F12 | Frontend | แก้ responsive bug |
| F13 | Frontend | แก้ accessibility issues |

### Phase 5: Integration & CI/CD — 14 prompts

| ID | Role | Prompt สั้นๆ |
|----|------|-------------|
| Q3 | QA | Test container setup |
| Q4 | QA | E2E integration tests |
| Q5 | QA | Test fixtures |
| Q6 | QA | OWASP security review |
| Q7 | QA | Security test cases |
| O9 | DevOps | CI pipeline (GitHub Actions) |
| O10 | DevOps | Dockerfile API (multi-stage) |
| O11 | DevOps | Dockerfile Frontend (multi-stage) |
| O12 | DevOps | Docker Compose staging |
| S8 | SA | Review deployment architecture |
| O13 | DevOps | Production hardening |
| P4 | PM | Release checklist |
| P5 | PM | Verify spec vs tests |
| P6 | PM | Release notes |

### Phase 6: Maintenance — 25 prompts

| ID | Role | Prompt สั้นๆ |
|----|------|-------------|
| D14 | Backend | วิเคราะห์ production bug |
| D15 | Backend | Regression test ก่อน fix |
| D16 | Backend | Fix + verify |
| D17 | Backend | DEVLOG entry |
| B3 | BA | New feature spec |
| S9 | SA | Impact analysis |
| D18 | Backend | New feature model |
| D19 | Backend | New feature service |
| D20 | Backend | New feature endpoints |
| F14 | Frontend | Admin page (new feature) |
| F15 | Frontend | Checkout feature |
| S10 | SA | วิเคราะห์ responsibilities |
| S11 | SA | วางแผน refactoring |
| D21 | Backend | Refactor step-by-step |
| N1 | New Dev | เรียนรู้ architecture |
| N2 | New Dev | Trace request flow |
| N3 | New Dev | เข้าใจ business rules |
| S12 | SA | Onboarding checklist |
| N4 | New Dev | Task แรกตาม pattern |
| D26 | Backend | Performance analysis |
| D22 | Backend | แก้ N+1 query |
| D23 | Backend | เพิ่ม pagination |
| D24 | Backend | เพิ่ม indexes |
| D25 | Backend | เพิ่ม Redis cache |
| Q11 | QA | Reproduce bug test |

---

## สถิติ

**Total: 80 prompts** จาก 6 phases, 8 roles

| Role | จำนวน | % |
|------|-------|---|
| Backend Dev (คุณแบ็ค) | 26 | 32% |
| Frontend Dev (คุณฟร้อนท์) | 15 | 19% |
| SA (คุณเอส) | 12 | 15% |
| QA (คุณคิว) | 11 | 14% |
| DBA/DevOps (คุณดีบี) | 8 | 10% |
| PM (คุณแพร) | 6 | 8% |
| New Dev (คุณนิว) | 4 | 5% |
| BA (คุณบี) | 3 | 4% |

| Phase | จำนวน | หมายเหตุ |
|-------|-------|----------|
| Phase 1: Inception | 14 | Architecture, specs, environment |
| Phase 2: Database & Backend | 11 | Schema, models, CRUD |
| Phase 3: Core Features | 12 | Auth, orders, tests, debugging |
| Phase 4: Frontend | 16 | Components, pages, QA |
| Phase 5: Integration & CI/CD | 14 | Security, CI, Docker |
| Phase 6: Maintenance | 25 | Bugs, features, refactoring, onboarding |

---

## Part 3: Dev Workflow Prompts — คำสั่งที่ Dev ใช้ทุกวัน

> Prompt ในส่วนนี้ไม่ผูกกับ phase ใด — เป็นคำสั่งที่ developer ใช้เป็นประจำสำหรับจัดการ memory,
> เริ่มงาน/ปิดงาน, sync code, และทำงานร่วมกันในทีม

### 3.1 ก่อนเริ่มงาน (Start of Day / New Session)

**W1.** ตรวจสอบสถานะโปรเจกต์
```
อ่าน DEVLOG.md และ git log --oneline -20 แล้วสรุป:
1. งานล่าสุดที่ทำเสร็จ
2. งานที่ค้างอยู่ (ถ้ามี)
3. ไฟล์ที่มี uncommitted changes
```
> ทำก่อนเริ่มงานทุกครั้ง เพื่อให้ Claude เข้าใจ context ปัจจุบัน

**W2.** โหลด context จาก memory
```
อ่าน CLAUDE.md, docs/specs/ (ถ้ามี), และ DEVLOG.md
แล้วสรุปสิ่งที่ต้องจำ: architecture decisions, naming conventions, งานที่ค้าง
```
> ใช้ตอนเริ่ม session ใหม่ เพื่อให้ Claude Code มี context เท่ากับเมื่อวาน

**W3.** Resume งานจากเซสชันที่แล้ว
```
ดู DEVLOG.md section ล่าสุด แล้วทำงานต่อจากจุดที่ค้างไว้
ถ้ามี TODO items ใน DEVLOG ให้เริ่มจากข้อแรกที่ยังไม่เสร็จ
```
> ใช้กับ `claude -c` (continue last session) หรือ `claude --resume` (เลือก session)

### 3.2 จัดการ Memory และ Context

**W4.** บันทึก decisions ลง CLAUDE.md
```
เพิ่ม rule ใหม่ใน CLAUDE.md:
- [rule description]
- เหตุผล: [rationale]
อย่าลบ rules เดิม ให้เพิ่มต่อท้ายในหมวดที่เหมาะสม
```
> ใช้ทุกครั้งที่ตัดสินใจ convention ใหม่ เพื่อให้ Claude จำได้ในทุก session

**W5.** Compact context เมื่อ session ยาว
```
/compact สรุปสิ่งที่ทำไปแล้วในเซสชันนี้ และเก็บ context สำคัญไว้:
- ไฟล์ที่แก้ไป: [list]
- decisions ที่ตัดสินใจ: [list]
- งานที่เหลือ: [list]
```
> ใช้เมื่อ session ยาวเกิน context window — Claude จะบีบอัดแต่เก็บข้อมูลสำคัญ

**W6.** อัพเดท CLAUDE.md จาก codebase จริง
```
อ่าน codebase ปัจจุบัน (โครงสร้างไฟล์, naming patterns, dependencies)
แล้วตรวจสอบว่า CLAUDE.md ยังตรงกับ reality หรือไม่
ถ้ามีส่วนที่ outdated ให้เสนอการแก้ไข
```
> ใช้เป็นระยะ — CLAUDE.md ที่ outdated ทำให้ Claude สร้าง code ที่ไม่ตรง convention

**W7.** สร้าง project memory จากศูนย์
```
วิเคราะห์ codebase นี้แล้วสร้าง CLAUDE.md ที่ครอบคลุม:
1. Tech stack + versions
2. Project structure (key directories)
3. Architecture pattern (layer, naming)
4. Coding conventions ที่เห็นจาก code
5. Commands สำหรับ build, test, run
6. Important files/paths
```
> ใช้ตอน onboard โปรเจกต์ใหม่ หรือโปรเจกต์ที่ยังไม่มี CLAUDE.md

### 3.3 หลังทำงานเสร็จ (End of Day / End of Task)

**W8.** เขียน DEVLOG entry
```
เขียน DEVLOG entry สำหรับงานวันนี้:
## [วันที่] — [หัวข้อสั้นๆ]
### What Was Done
- [list สิ่งที่ทำเสร็จ]
### Technical Decisions
- [decisions + rationale]
### Known Issues
- [issues ที่รู้แต่ยังไม่ได้แก้]
### Next Steps
- [ ] [TODO items สำหรับ session หน้า]
```
> ทำก่อนปิด session ทุกครั้ง — เป็น "จดหมายถึงตัวเองในอนาคต"

**W9.** Self-review ก่อน commit
```
ดู git diff --staged แล้ว review:
1. มี code ที่ไม่ควร commit ไหม? (.env, debug logs, TODO hacks)
2. Naming conventions ตรงกับ CLAUDE.md ไหม?
3. มี test สำหรับ logic ใหม่ไหม?
4. มี error handling ที่ขาดไหม?
สรุปเป็น checklist: ✅ ผ่าน / ❌ ต้องแก้
```
> ทำก่อน `git commit` ทุกครั้ง — จับ issues ก่อนเข้า PR

**W10.** สรุป session สำหรับทีม
```
สรุปงานที่ทำในเซสชันนี้ในรูปแบบ standup update:
- Done: [สิ่งที่เสร็จ]
- In Progress: [สิ่งที่กำลังทำ]
- Blocked: [สิ่งที่ติดขัด]
- Next: [สิ่งที่จะทำต่อ]
เขียนให้กระชับ ไม่เกิน 5 บรรทัด
```
> Copy ไปวางใน Slack / standup meeting

### 3.4 Sync Code และทำงานร่วมกัน

**W11.** Review code ของเพื่อนร่วมทีม
```
ดู git diff main...[branch-name] แล้ว review:
1. โครงสร้างตาม architecture ที่กำหนดไหม?
2. Error handling ครบไหม?
3. มี test ไหม?
4. มี security concerns ไหม?
5. Performance: มี N+1 query, missing index, unnecessary loop ไหม?
สรุปเป็น comment style: ✅ Approve / 🔧 Request Changes พร้อมเหตุผล
```
> ใช้ review PR — Claude อ่าน diff แล้ว review ตาม coding standards

**W12.** Resolve merge conflicts
```
ดู git diff --name-only --diff-filter=U (conflicted files)
แล้วช่วย resolve conflicts:
- เข้าใจว่าแต่ละฝั่ง (ours/theirs) ต้องการอะไร
- เลือกวิธี merge ที่ถูกต้องตาม business logic
- ตรวจว่า resolved code ยังทำงานถูกต้อง
```
> ใช้ตอน merge/rebase แล้วมี conflicts — Claude เข้าใจ context ทั้งสองฝั่ง

**W13.** Sync types ระหว่าง Backend ↔ Frontend
```
อ่าน Go DTOs ใน internal/dto/ แล้วตรวจสอบว่า TypeScript interfaces ใน web/src/types/
ยังตรงกันอยู่ไหม:
- field ที่เพิ่ม/ลบ/เปลี่ยนชื่อ
- type ที่เปลี่ยน
สรุปความต่าง และอัพเดท TS types ให้ตรงกับ Go DTOs
```
> ใช้หลัง Backend เปลี่ยน DTO — ป้องกัน type mismatch ระหว่าง FE/BE

**W14.** ตรวจสอบ API contract
```
เปรียบเทียบ API spec ใน docs/api/ กับ implementation จริงใน internal/handlers/:
- Endpoints ที่มีใน spec แต่ยังไม่ implement
- Endpoints ที่ implement แล้วแต่ไม่ตรงกับ spec (path, method, request/response shape)
- Endpoints ที่ implement แล้วแต่ไม่มีใน spec
สรุปเป็นตาราง
```
> ใช้เป็นระยะ เพื่อให้ spec กับ code sync กัน

**W15.** Handoff งานให้เพื่อนร่วมทีม
```
สรุปสถานะของ [feature/branch] สำหรับ [ชื่อเพื่อน] ที่จะรับงานต่อ:
1. สิ่งที่ทำเสร็จแล้ว + ไฟล์ที่เกี่ยวข้อง
2. สิ่งที่ยังเหลือ + approach ที่แนะนำ
3. Gotchas / ข้อควรระวัง
4. วิธี test สิ่งที่ทำไปแล้ว
เขียนในรูปแบบที่ paste ลง PR description ได้
```
> ใช้ตอนต้องส่งต่องาน — ลด context loss ระหว่างคน

### 3.5 Debugging & Troubleshooting Workflows

**W16.** วิเคราะห์ error log
```
วิเคราะห์ error นี้:
[paste error/stack trace]

1. สาเหตุที่น่าจะเป็นไปได้ (เรียงจากน่าจะใช่ที่สุด)
2. ไฟล์ที่เกี่ยวข้อง
3. วิธีแก้ไข
4. วิธีป้องกันไม่ให้เกิดอีก
```
> Copy-paste error message จาก terminal / log ได้เลย

**W17.** ตรวจสอบ dependency issues
```
อ่าน go.mod (หรือ package.json) แล้วตรวจสอบ:
1. Dependencies ที่ outdated มาก (major version behind)
2. Dependencies ที่มี known vulnerabilities
3. Dependencies ที่ไม่ได้ใช้แล้ว
4. Conflicts หรือ version incompatibilities
สรุปเป็น action items
```
> ใช้เป็นระยะ หรือตอน build/test fail จาก dependency issues

**W18.** Performance quick check
```
อ่าน [file/function] แล้ววิเคราะห์ performance:
1. N+1 queries
2. Missing database indexes
3. Unnecessary loops/allocations
4. Missing caching opportunities
5. Unbounded queries (no LIMIT)
เสนอ improvements เรียงตาม impact
```
> Quick audit สำหรับ code ที่ไม่มั่นใจ

### 3.6 Git Workflow Prompts

**W19.** สร้าง commit message
```
ดู git diff --staged แล้วเขียน commit message:
- ใช้ conventional commits format (feat/fix/refactor/docs/test/chore)
- บรรทัดแรก < 72 chars
- Body อธิบายว่าทำไม (why) ไม่ใช่ทำอะไร (what)
```
> ให้ Claude เขียน commit message จาก staged changes

**W20.** สร้าง PR description
```
ดู git log main..HEAD และ git diff main...HEAD แล้วสร้าง PR description:
## Summary
- [bullet points]
## Changes
- [file-by-file summary]
## Test Plan
- [ ] [test checklist]
## Breaking Changes
- [ถ้ามี]
```
> Auto-generate PR description จาก commits + diff

**W21.** Branch cleanup
```
ดู git branch --merged main แล้วแนะนำ branches ที่ปลอดภัยที่จะลบ
แยกระหว่าง: local only, has remote, ไม่แน่ใจ (ต้อง check)
```
> Housekeeping — ลบ branches ที่ merge แล้ว

---

## สถิติ (อัพเดท)

**Total: 101 prompts** — 80 project prompts + 21 dev workflow prompts

### Project Prompts (by Role)

| Role | จำนวน | % |
|------|-------|---|
| Backend Dev (คุณแบ็ค) | 26 | 32% |
| Frontend Dev (คุณฟร้อนท์) | 15 | 19% |
| SA (คุณเอส) | 12 | 15% |
| QA (คุณคิว) | 11 | 14% |
| DBA/DevOps (คุณดีบี) | 8 | 10% |
| PM (คุณแพร) | 6 | 8% |
| New Dev (คุณนิว) | 4 | 5% |
| BA (คุณบี) | 3 | 4% |

### Dev Workflow Prompts (by Category)

| หมวด | จำนวน |
|------|-------|
| ก่อนเริ่มงาน (W1–W3) | 3 |
| จัดการ Memory (W4–W7) | 4 |
| หลังทำงาน (W8–W10) | 3 |
| Sync & Collaboration (W11–W15) | 5 |
| Debugging (W16–W18) | 3 |
| Git Workflow (W19–W21) | 3 |

### Project Prompts (by Phase)

| Phase | จำนวน | หมายเหตุ |
|-------|-------|----------|
| Phase 1: Inception | 14 | Architecture, specs, environment |
| Phase 2: Database & Backend | 11 | Schema, models, CRUD |
| Phase 3: Core Features | 12 | Auth, orders, tests, debugging |
| Phase 4: Frontend | 16 | Components, pages, QA |
| Phase 5: Integration & CI/CD | 14 | Security, CI, Docker |
| Phase 6: Maintenance | 25 | Bugs, features, refactoring, onboarding |

---

← [Phase 6](06_phase6_maintenance.md) | [Overview](00_overview.md)
