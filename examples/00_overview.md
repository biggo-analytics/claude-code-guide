# ShopFast — ตัวอย่างการใช้ Claude Code ตลอดวงจรชีวิตโปรเจกต์

> **End-to-End Project Walkthrough** — จำลองโปรเจกต์ E-commerce ขนาดกลาง
> ตั้งแต่ inception จนถึง maintenance โดยแสดงว่าแต่ละ role ทำอะไร ใช้ prompt อะไร ในแต่ละช่วง

---

## สารบัญ (Table of Contents)

| ไฟล์ | Phase | เนื้อหาหลัก |
|------|-------|-------------|
| [00_overview.md](00_overview.md) | — | แนะนำโปรเจกต์, ทีม, tech stack, ไกด์การอ่าน |
| [01_phase1_inception.md](01_phase1_inception.md) | Inception | สถาปัตยกรรม, CLAUDE.md, specs, dev env setup |
| [02_phase2_database_backend.md](02_phase2_database_backend.md) | Database & Backend | ER design, GORM models, migrations, base CRUD |
| [03_phase3_core_features.md](03_phase3_core_features.md) | Core Features | Auth, business logic, search, tests, debugging |
| [04_phase4_frontend.md](04_phase4_frontend.md) | Frontend | React admin panel, types from Go, API hooks, pages |
| [05_phase5_integration_cicd.md](05_phase5_integration_cicd.md) | Integration & CI/CD | Integration tests, CI pipeline, production Docker |
| [06_phase6_maintenance.md](06_phase6_maintenance.md) | Maintenance | 5 scenarios: bug fix, new feature, refactor, onboarding, perf |
| [07_prompt_catalog.md](07_prompt_catalog.md) | — | Prompt ทั้งหมด (~60+) จัดตาม role และ phase |
| [08_figma_mcp_usecase.md](08_figma_mcp_usecase.md) | Use Case | แปลง Web URL / Screenshot → Figma design ด้วย MCP |

---

## โปรเจกต์ ShopFast คืออะไร?

**ShopFast** เป็นระบบ E-commerce ขนาดกลาง ประกอบด้วย:

- **REST API** สำหรับจัดการสินค้า, ออเดอร์, ผู้ใช้, และ payment
- **Admin Dashboard** สำหรับทีมงานภายในจัดการระบบ
- **Background Jobs** สำหรับ order processing, email notifications, inventory sync

### ขอบเขตที่จำลอง

```
┌─────────────────────────────────────────────┐
│                 ShopFast                      │
│                                               │
│  ┌──────────┐  ┌──────────┐  ┌────────────┐ │
│  │ REST API │  │  Admin   │  │ Background │ │
│  │ (Go/     │  │Dashboard │  │   Jobs     │ │
│  │  Fiber)  │  │(React/TS)│  │  (Go)      │ │
│  └────┬─────┘  └────┬─────┘  └─────┬──────┘ │
│       │              │              │         │
│       └──────────────┼──────────────┘         │
│                      │                         │
│              ┌───────┴───────┐                │
│              │  PostgreSQL   │                │
│              └───────────────┘                │
└─────────────────────────────────────────────┘
```

> **หมายเหตุ:** ตัวอย่างนี้ไม่ได้สร้างแอปจริง แต่จำลอง prompts, workflows, และ decision-making
> ที่เกิดขึ้นในแต่ละ phase เพื่อให้เห็นภาพการใช้ Claude Code ในบริบทจริง

---

## Tech Stack

| Layer | Technology | หมายเหตุ |
|-------|-----------|----------|
| **Backend** | Go 1.22 + Fiber v2 | REST API, three-layer architecture |
| **ORM** | GORM v2 | Auto-migration, hooks, preloading |
| **Frontend** | React 18 + TypeScript 5 | Vite, React Query, React Router |
| **Styling** | Tailwind CSS v3 | Admin Dashboard UI |
| **Database** | PostgreSQL 16 | Primary data store |
| **Cache** | Redis 7 | Session, rate limiting, product cache |
| **CI/CD** | GitHub Actions | Lint → Test → Build → Deploy |
| **Container** | Docker + Docker Compose | Local dev & production |
| **Monitoring** | Prometheus + Grafana | Metrics, alerting |

---

## ทีม ShopFast (7 คน)

| Role | ชื่อ (สมมติ) | หน้าที่หลักใน walkthrough |
|------|-------------|--------------------------|
| **PM** | คุณแพร | กำหนด scope, ติดตาม progress, review milestones |
| **BA** | คุณบี | เขียน specs, acceptance criteria, user stories |
| **SA** | คุณเอส | ออกแบบ architecture, review technical decisions, เขียน ADR |
| **Backend Dev** | คุณแบ็ค | เขียน Go API, business logic, unit tests |
| **Frontend Dev** | คุณฟร้อนท์ | เขียน React admin dashboard, API integration |
| **DBA / DevOps** | คุณดีบี | Database design, migrations, CI/CD pipeline, Docker |
| **QA** | คุณคิว | Test plans, integration tests, security review |

> **สังเกต:** ทีมนี้ตรงกับ roles ที่อธิบายใน [บทที่ 8: Roles & Responsibilities](../08_roles_responsibilities.md)
> แต่ละ phase จะแสดงว่า role ไหนใช้ Claude Code ทำอะไร พร้อม prompt จริง

---

## Timeline ของโปรเจกต์

```
สัปดาห์  1     2     3     4     5     6     7     8+
        ├─────┼─────┼─────┼─────┼─────┼─────┼─────┼────→
Phase 1 │█████│     │     │     │     │     │     │
        │Incep│     │     │     │     │     │     │
Phase 2 │     │█████│█████│     │     │     │     │
        │     │DB & Backend  │     │     │     │     │
Phase 3 │     │     │█████│█████│     │     │     │
        │     │     │Core Features  │     │     │     │
Phase 4 │     │     │     │█████│█████│     │     │
        │     │     │     │Frontend    │     │     │
Phase 5 │     │     │     │     │█████│█████│     │
        │     │     │     │     │Integration  │     │
Phase 6 │     │     │     │     │     │     │█████│████→
        │     │     │     │     │     │     │Maintenance
```

| Phase | ระยะเวลา | ไฟล์ |
|-------|----------|------|
| **Phase 1: Inception** | สัปดาห์ 1 | [01_phase1_inception.md](01_phase1_inception.md) |
| **Phase 2: Database & Backend** | สัปดาห์ 2–3 | [02_phase2_database_backend.md](02_phase2_database_backend.md) |
| **Phase 3: Core Features** | สัปดาห์ 3–4 | [03_phase3_core_features.md](03_phase3_core_features.md) |
| **Phase 4: Frontend** | สัปดาห์ 4–5 | [04_phase4_frontend.md](04_phase4_frontend.md) |
| **Phase 5: Integration & CI/CD** | สัปดาห์ 5–6 | [05_phase5_integration_cicd.md](05_phase5_integration_cicd.md) |
| **Phase 6: Maintenance** | สัปดาห์ 7+ (ongoing) | [06_phase6_maintenance.md](06_phase6_maintenance.md) |

---

## วิธีอ่านตัวอย่างนี้

### สำหรับผู้ที่อยากอ่านทั้งหมด
อ่านตามลำดับ `01` → `06` แล้วจบด้วย `07` (prompt catalog) เพื่อดูภาพรวมทั้งหมด

### สำหรับผู้ที่สนใจเฉพาะ role
1. ดูตาราง "ใครทำอะไร" ในแต่ละ phase
2. อ่านเฉพาะส่วนที่ตรงกับ role ของตัวเอง
3. ไปที่ [07_prompt_catalog.md](07_prompt_catalog.md) → ค้นหาตาม role ของตัวเอง

### สำหรับผู้ที่ต้องการ prompt สำเร็จรูป
ไปที่ [07_prompt_catalog.md](07_prompt_catalog.md) โดยตรง — รวม prompt ทั้งหมดที่ปรากฏใน walkthrough จัดหมวดตาม role และ phase พร้อม copy-paste ได้เลย

---

## Cross-References กับบทหลัก

ตัวอย่างนี้เชื่อมโยงกลับไปยังเนื้อหาหลักของ guide ตลอดทั้งเอกสาร:

| Phase | บทที่อ้างอิง |
|-------|-------------|
| Phase 1 (Inception) | [บทที่ 1](../01_intro.md) — CLAUDE.md, [บทที่ 5](../05_spec_driven_dev.md) — Specs, [บทที่ 6](../06_architecture_design.md) — Architecture, [บทที่ 8](../08_roles_responsibilities.md) — Permissions |
| Phase 2 (DB & Backend) | [บทที่ 4](../04_fullstack_collaboration.md) — GORM/CRUD, [บทที่ 6](../06_architecture_design.md) — DB Design |
| Phase 3 (Core Features) | [บทที่ 4](../04_fullstack_collaboration.md) — Debugging/Business Logic, [บทที่ 5](../05_spec_driven_dev.md) — Spec-to-Code, [บทที่ 6](../06_architecture_design.md) — ADR, [บทที่ 7](../07_qa_security_review.md) — Testing |
| Phase 4 (Frontend) | [บทที่ 4](../04_fullstack_collaboration.md) — React/Cross-project/React Query |
| Phase 5 (Integration) | [บทที่ 4](../04_fullstack_collaboration.md) — CI/CD/Docker, [บทที่ 7](../07_qa_security_review.md) — QA |
| Phase 6 (Maintenance) | [บทที่ 4](../04_fullstack_collaboration.md) — DEVLOG, [บทที่ 6](../06_architecture_design.md) — Refactoring, [บทที่ 8](../08_roles_responsibilities.md) — Onboarding |
| Prompt Catalog | ทุก phase + [บทที่ 9](../09_cheat_sheet.md) — Cheat Sheet |

---

## สัญลักษณ์ที่ใช้ในตัวอย่าง

| สัญลักษณ์ | ความหมาย |
|-----------|----------|
| `👤 คุณแบ็ค:` | ระบุว่า role ไหนเป็นผู้ใช้ Claude Code |
| ` ``` ` (code block ไม่มี lang) | Prompt ที่พิมพ์ให้ Claude Code |
| ` ```go `, ` ```tsx ` | Code output ที่ Claude Code สร้าง (ย่อ) |
| `📌 Cross-ref:` | ลิงก์ไปยังบทหลักที่มีรายละเอียดเพิ่มเติม |
| `💡 เคล็ดลับ:` | Tips ที่ได้จากประสบการณ์จริง |
| `⚠️ ข้อควรระวัง:` | สิ่งที่ต้องตรวจสอบ / gotcha |
| `✅ Checkpoint` | สิ่งที่ต้องเสร็จก่อนไป phase ถัดไป |

---

*ต่อไป: [Phase 1 — Inception →](01_phase1_inception.md)*
