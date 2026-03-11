# Architecture & Design — สถาปัตยกรรม, ADR, Technical Debt

> Prompt สำหรับ SA / Tech Lead — วิเคราะห์สถาปัตยกรรม, บันทึก decisions, จัดการ CLAUDE.md, วางแผน technical evolution
> ใช้ได้กับทุก tech stack โดยแทนค่า `[placeholder]`

---

## สารบัญ

- [3.1 Architecture Analysis](#31-architecture-analysis)
- [3.2 Architecture Decision Records (ADR)](#32-architecture-decision-records-adr)
- [3.3 System Design](#33-system-design)
- [3.4 CLAUDE.md Management](#34-claudemd-management)
- [3.5 Technical Debt & Evolution](#35-technical-debt--evolution)

---

## 3.1 Architecture Analysis

**ARCH-01.** วิเคราะห์สถาปัตยกรรมปัจจุบัน
> ทำความเข้าใจ architecture ทั้งระบบ

```
วิเคราะห์สถาปัตยกรรมของโปรเจกต์นี้:
1. Architecture style: monolith / microservices / modular / hexagonal?
2. Layer structure: มีกี่ layers? แต่ละ layer มีบทบาทอะไร?
3. Communication patterns: sync/async, REST/gRPC/events
4. Data flow: request เข้ามา → ผ่านอะไรบ้าง → response ออกไป
5. External dependencies: services, databases, queues
วาดเป็น ASCII diagram ของ architecture
```

---

**ARCH-02.** ตรวจสอบ architecture violations
> ตรวจว่า code ยังตรงกับ architecture ที่ออกแบบ

```
ตรวจสอบ architecture violations ใน codebase:
1. Layer violations — handler เรียก repository ตรง? service เรียก handler?
2. Dependency direction — depend เข้าข้างในหรือออกข้างนอก?
3. Circular dependencies — modules ไหน depend กันวนรอบ?
4. Interface usage — ข้าม layer ผ่าน interface หรือ concrete type?
5. Package boundaries — มี import ข้ามกลุ่มที่ไม่ควรมีไหม?
รายงานเป็นตาราง: ไฟล์, violation type, severity, วิธีแก้
```

---

**ARCH-03.** วิเคราะห์ dependency graph
> เข้าใจ dependencies ระหว่าง modules

```
สร้าง dependency graph ของโปรเจกต์:
1. Module-level: แต่ละ module/package depend on อะไร
2. Depth: dependency chain ยาวแค่ไหน
3. Coupling: คู่ไหน coupling มากที่สุด
4. Stability: module ไหน "stable" (ถูก depend มาก), ไหน "unstable"
5. Abstractions: module ไหนมี interface มาก vs concrete class มาก
แนะนำจุดที่ควรปรับ dependency structure
```

---

**ARCH-04.** วิเคราะห์ coupling / cohesion
> หา modules ที่ coupling สูงหรือ cohesion ต่ำ

```
วิเคราะห์ coupling และ cohesion:
1. Afferent Coupling (Ca): module ไหนถูก depend มากที่สุด
2. Efferent Coupling (Ce): module ไหน depend คนอื่นมากที่สุด
3. Cohesion: module ไหนมี functions ที่ไม่เกี่ยวกัน (low cohesion)
4. God class/module: มี module ที่ทำหลายอย่างเกินไปไหม
5. Feature envy: มี code ที่ใช้ data ของ module อื่นมากกว่าตัวเอง
แนะนำ refactoring สำหรับจุดที่มีปัญหามากที่สุด
```

---

**ARCH-05.** เปรียบเทียบ current vs intended architecture
> ตรวจว่า code drift จาก design ไปแค่ไหน

```
เปรียบเทียบ:
- Intended architecture (จาก CLAUDE.md / ADRs / docs): [สรุป design ที่ตั้งใจ]
- Actual implementation (จาก code จริง)

สรุปเป็นตาราง:
| Aspect | Intended | Actual | Gap |
|--------|----------|--------|-----|
| ... | ... | ... | ... |

แนะนำว่าควรแก้ code ให้ตรง design หรือ update design ให้ตรง code
```

---

**ARCH-06.** Review code structure ตาม principles
> SA review ตาม architecture principles

```
Review โครงสร้าง project ตาม architecture principles:
1. Separation of Concerns — แต่ละ layer ทำหน้าที่ชัดเจน?
2. Dependency Inversion — depend on abstractions?
3. Single Responsibility — แต่ละ module มีเหตุผลเดียวที่ต้องเปลี่ยน?
4. Open/Closed — extend ได้โดยไม่ต้องแก้ code เดิม?
5. Interface Segregation — interface เล็กพอ?
อ่านไฟล์ทั้งหมดแล้วสรุปผลเป็นตาราง: principle, status (✅/⚠️/❌), หมายเหตุ
```

---

## 3.2 Architecture Decision Records (ADR)

**ARCH-07.** สร้าง ADR สำหรับ decision ใหม่
> บันทึก technical decision อย่างเป็นทางการ

```
สร้าง ADR-[NNN] เรื่อง [decision]:
- Title: [ชื่อ decision]
- Status: Proposed
- Context: [ทำไมต้องตัดสินใจเรื่องนี้ ปัญหาคืออะไร]
- Decision: [สิ่งที่ตัดสินใจ]
- Consequences:
  - Positive: [ข้อดี]
  - Negative: [ข้อเสีย/trade-offs]
  - Risks: [ความเสี่ยง]
- Alternatives Considered:
  - [Option A]: [pros/cons]
  - [Option B]: [pros/cons]
บันทึกที่ docs/adr/[filename].md
```

---

**ARCH-08.** สร้าง ADR จาก incident / bug
> บันทึก decision ที่เกิดจากปัญหาจริง

```
เขียน ADR จาก incident ที่เพิ่งเกิด:
- Context: [ปัญหาที่พบ — อาการ, ผลกระทบ]
- Root Cause: [สาเหตุ]
- Decision: [วิธีแก้ที่เลือก]
- Alternatives: [ทางเลือกอื่นที่พิจารณา + เหตุผลที่ไม่เลือก]
- Consequences:
  - Rules ใหม่ที่ต้องปฏิบัติ
  - Performance impact
  - Code changes ที่ต้องทำ
```

---

**ARCH-09.** Review และ update ADRs ที่มีอยู่
> ตรวจว่า ADRs ยัง current ไหม

```
อ่าน ADRs ทั้งหมดใน docs/adr/ แล้วตรวจสอบ:
1. ADR ไหนยังเป็นจริง (Active)
2. ADR ไหน outdated (ควรเป็น Superseded)
3. ADR ไหนไม่ได้ถูก implement จริง
4. Decision ไหนมี ADR แต่ code ทำไม่ตรง
5. Decision สำคัญที่ยังไม่มี ADR
สรุปเป็นตาราง + recommended actions
```

---

**ARCH-10.** Tradeoff analysis
> เปรียบเทียบ options สำหรับ technical decision

```
วิเคราะห์ tradeoffs สำหรับ [decision]:
Options:
- [Option A]: [brief description]
- [Option B]: [brief description]
- [Option C]: [brief description]

เปรียบเทียบในด้าน:
| Criteria | Option A | Option B | Option C |
|----------|----------|----------|----------|
| Complexity | | | |
| Performance | | | |
| Maintainability | | | |
| Team familiarity | | | |
| Migration effort | | | |
| Risk | | | |

แนะนำ option ที่เหมาะสมที่สุด + เหตุผล
```

---

## 3.3 System Design

**ARCH-11.** ออกแบบ feature/module ใหม่ (high-level)
> ออกแบบก่อน implement

```
ออกแบบ [feature/module] ใหม่:
- Requirements: [list requirements]
- Constraints: [technical constraints, timeline, etc.]

ผลลัพธ์ที่ต้องการ:
1. Component diagram (ASCII)
2. Key interfaces / contracts
3. Data model (entities + relationships)
4. API endpoints (if applicable)
5. ไฟล์ที่ต้องสร้าง/แก้ + directory structure
6. Dependencies กับ modules อื่น
7. Edge cases ที่ต้อง handle
ออกแบบก่อน ยังไม่ต้อง implement
```

---

**ARCH-12.** ออกแบบ API contract
> กำหนด API interface ก่อน implement

```
ออกแบบ API contract สำหรับ [feature]:
แต่ละ endpoint:
- Method + Path
- Request: headers, params, body (with types + validation)
- Response: success body + error bodies
- Status codes ที่เป็นไปได้
- Auth requirements
- Rate limiting (ถ้ามี)

รูปแบบ:
- JSON naming: [camelCase/snake_case]
- Pagination: [cursor/offset]
- Error format: [standard format]
- Versioning: [URL/header]
```

---

**ARCH-13.** ออกแบบ database schema
> ออกแบบ schema สำหรับ feature ใหม่

```
ออกแบบ database schema สำหรับ [feature]:
Requirements: [business requirements]

ผลลัพธ์:
1. ER diagram (text-based)
2. Tables: columns, types, constraints
3. Relationships: FK, cardinality
4. Indexes: ที่จำเป็นสำหรับ queries ที่ใช้บ่อย
5. Migration strategy: [zero-downtime?]
6. Data volume estimate + growth rate
7. Compatibility กับ schema ที่มีอยู่
```

---

**ARCH-14.** ออกแบบ caching strategy
> วางแผน cache เพื่อ performance

```
ออกแบบ caching strategy สำหรับ [module/feature]:
1. Cache targets: data อะไรควร cache?
2. Cache type: [in-memory / Redis / CDN / browser]
3. Key patterns: [naming convention]
4. TTL: แต่ละ type expire เมื่อไหร่?
5. Invalidation: cache update/clear เมื่อไหร่?
6. Cache miss handling: fallback behavior
7. Warm-up strategy: ต้อง pre-load ไหม?
8. Monitoring: ดู hit rate อย่างไร?
```

---

**ARCH-15.** ออกแบบ error handling strategy
> กำหนด error handling ทั้งระบบ

```
ออกแบบ error handling strategy สำหรับโปรเจกต์:
1. Error types: กำหนด error categories (validation, auth, not found, internal)
2. Error codes: numbering system
3. Error propagation: แต่ละ layer ส่ง error อย่างไร
4. Error wrapping: เพิ่ม context อย่างไร
5. Client-facing errors: format + ข้อมูลที่ส่งให้ client
6. Internal errors: format + ข้อมูลสำหรับ logging
7. Logging: error ไหน log level อะไร
8. Recovery: retry, circuit breaker, fallback
```

---

**ARCH-16.** ออกแบบ event/messaging architecture
> วาง event-driven patterns

```
ออกแบบ event system สำหรับ [use case]:
1. Events: list events ที่ต้องมี (name, payload, trigger)
2. Publishers: ใครส่ง event
3. Subscribers: ใครรับ event + ทำอะไร
4. Delivery guarantee: at-least-once / exactly-once
5. Ordering: ต้อง maintain order ไหม
6. Error handling: failed event processing
7. Dead letter queue: unprocessable events
8. Monitoring: event flow tracking
```

---

## 3.4 CLAUDE.md Management

**ARCH-17.** สร้าง CLAUDE.md จากศูนย์
> โปรเจกต์ใหม่ที่ยังไม่มี CLAUDE.md

```
วิเคราะห์ codebase แล้วสร้าง CLAUDE.md:
1. Project Overview — ชื่อ, purpose, tech stack + versions
2. Architecture — layers, patterns, key directories
3. Coding Conventions — naming (functions, variables, files), formatting
4. Commands — build, test, lint, run, migrate, seed
5. Directory Structure — อธิบายแต่ละ directory
6. Important Rules — ข้อห้าม, conventions ที่ต้องทำตาม
7. Testing — framework, patterns, how to run
8. API Conventions — response format, error codes, auth
9. Git Conventions — branch naming, commit format, PR process
```

---

**ARCH-18.** Audit CLAUDE.md กับ codebase จริง
> ตรวจว่า CLAUDE.md ยังตรงกับ reality

```
เปรียบเทียบ CLAUDE.md กับ codebase จริง:
สำหรับแต่ละ section ใน CLAUDE.md:
1. ข้อมูลยังถูกต้องไหม? (tech stack, versions, patterns)
2. Commands ที่ระบุยังทำงานได้ไหม?
3. Directory structure ยังตรงไหม?
4. Conventions ที่ระบุ ถูก follow จริงไหม?
5. มีอะไรใหม่ใน codebase ที่ยังไม่ได้เพิ่มใน CLAUDE.md?
สรุปเป็น: ✅ ถูกต้อง / ❌ Outdated (+ แก้อะไร) / ➕ ขาดหาย (+ เพิ่มอะไร)
```

---

**ARCH-19.** อัปเดต CLAUDE.md หลังการเปลี่ยนแปลงใหญ่
> หลัง refactor, migration, หรือ architecture change

```
เพิ่ง [อธิบาย change ที่เกิด] ช่วยอัปเดต CLAUDE.md:
1. ส่วนไหนต้องแก้ไข (เปลี่ยน pattern, rename, ย้าย directory)
2. ส่วนไหนต้องเพิ่ม (tools ใหม่, conventions ใหม่)
3. ส่วนไหนต้องลบ (deprecated patterns)
อัปเดตแล้วรัน build/test เพื่อยืนยันว่า commands ยังถูกต้อง
```

---

**ARCH-20.** ออกแบบ permission profiles
> สร้าง .claude/settings.json สำหรับทีม

```
ออกแบบ Claude Code permission profiles:

Shared (.claude/settings.json):
- Allow ทุกคน: [Read, search, build, test, lint]
- Deny ทุกคน: [destructive commands, secret files, force push]

Role-specific (.claude/settings.local.json):
- Backend Dev: allow [backend dirs], deny [frontend dirs]
- Frontend Dev: allow [frontend dirs], deny [backend dirs]
- BA: allow [docs/specs], deny [source code modification]
- PM: read-only mode
- QA: allow [test files], deny [production code]
- DevOps: allow [infra files], deny [application code]

สร้าง JSON ตัวอย่างสำหรับแต่ละ role
```

---

## 3.5 Technical Debt & Evolution

**ARCH-21.** Technical debt inventory
> สำรวจ tech debt ทั้งหมด

```
สำรวจ technical debt ในโปรเจกต์:
1. Code smells: long methods, god classes, duplicate code
2. TODO/FIXME/HACK comments ทั้งหมด
3. Outdated dependencies (major version behind)
4. Missing tests (critical paths without coverage)
5. Hard-coded values ที่ควรเป็น config
6. Deprecated APIs/patterns ที่ยังใช้อยู่
7. Known performance issues
8. Security concerns

จัดลำดับตาม: severity × likelihood × impact
สรุปเป็น tech debt backlog
```

---

**ARCH-22.** Impact analysis
> วิเคราะห์ผลกระทบก่อนทำ change ใหญ่

```
วิเคราะห์ impact ของการ [proposed change]:
1. Files/modules ที่ต้องแก้โดยตรง
2. Files/modules ที่ได้รับผลกระทบทางอ้อม (dependencies)
3. Tests ที่ต้องอัปเดต
4. Configurations ที่ต้องเปลี่ยน
5. Database migrations ที่ต้องทำ
6. API changes ที่กระทบ consumers
7. Documentation ที่ต้องอัปเดต
8. Estimated effort + risk level
```

---

**ARCH-23.** วางแผน migration
> Migration ระบบ (framework, DB, API version)

```
วางแผน migration จาก [current] ไป [target]:
1. Assessment: อะไรต้องเปลี่ยน? scope ใหญ่แค่ไหน?
2. Strategy: big bang / incremental / strangler fig?
3. Steps: ลำดับ migration ที่ปลอดภัย
4. Parallel running: ต้องรัน old + new พร้อมกันไหม?
5. Data migration: ข้อมูลต้องย้าย/แปลงอย่างไร?
6. Rollback plan: ถ้ามีปัญหา ถอยกลับอย่างไร?
7. Testing: test อะไร ณ แต่ละ step?
8. Timeline: ประมาณ effort
```

---

**ARCH-24.** Deprecation plan
> วางแผนเลิกใช้ API/feature

```
วางแผน deprecation สำหรับ [API/feature/pattern]:
1. Current usage: ใช้ที่ไหนบ้าง? กี่จุด?
2. Replacement: ใช้อะไรแทน?
3. Migration path: consumers ต้องทำอะไรเพื่อย้าย?
4. Timeline:
   - Phase 1: Announce deprecation + add warnings
   - Phase 2: Provide migration tools/docs
   - Phase 3: Remove deprecated code
5. Communication: แจ้งใครบ้าง? ช่องทางไหน?
6. Breaking change handling: versioning strategy
```

---

**ARCH-25.** Architecture review checklist
> Review architecture ก่อน major milestone

```
Architecture review checklist:
1. Functionality: ตอบ requirements ครบไหม?
2. Reliability: มี error handling, retry, failover?
3. Performance: มี bottleneck ที่รู้ไหม? capacity plan?
4. Scalability: ถ้า load เพิ่ม 10x รองรับไหม?
5. Security: auth, input validation, data protection
6. Maintainability: code readable, well-structured, documented?
7. Testability: test coverage, test infrastructure
8. Operability: monitoring, logging, alerting, runbooks
9. Deployability: CI/CD, rollback, feature flags
สรุปเป็น: ✅ Ready / ⚠️ Concerns / ❌ Blockers
```

---

**ARCH-26.** Scalability analysis
> วิเคราะห์ว่าระบบ scale ได้แค่ไหน

```
วิเคราะห์ scalability ของระบบ:
1. Current bottleneck: จุดไหนจะเป็นปัญหาแรกเมื่อ load เพิ่ม?
2. Database: query patterns ที่จะช้าเมื่อ data โต
3. API: endpoints ที่ resource-intensive
4. Memory: ส่วนไหนที่ memory usage จะโตตาม data
5. Stateful components: session, cache, file storage
6. External dependencies: ส่วนไหนที่ third-party จะเป็น bottleneck
แนะนำ scaling strategy สำหรับแต่ละจุด
```

---

**ARCH-27.** Code review จากมุม architecture
> SA review PR ตาม architecture lens

```
Review PR/code changes จากมุม architecture:
1. ตรงกับ architecture patterns ที่กำหนดไหม?
2. Dependency direction ถูกต้องไหม?
3. มี layer violations ไหม?
4. Naming conventions ตรง CLAUDE.md?
5. ใช้ interfaces/abstractions ถูกต้องไหม?
6. Error handling ตาม strategy ที่กำหนด?
7. มี tech debt ใหม่ที่เพิ่มเข้ามาไหม?
8. ต้อง update CLAUDE.md/ADR ไหม?
สรุปเป็น ✅ Approve / 🔧 Request changes
```

---

**ARCH-28.** ออกแบบ module boundary
> กำหนด boundary สำหรับ module ใหม่

```
ออกแบบ boundary สำหรับ [new module]:
1. Responsibility: module นี้รับผิดชอบอะไรเท่านั้น
2. Public API: interfaces/functions ที่ expose
3. Internal: สิ่งที่ซ่อนไว้ข้างใน
4. Dependencies: import อะไรได้ อะไรไม่ได้
5. Data ownership: module นี้ own data อะไร
6. Communication: คุยกับ module อื่นผ่านอะไร (interface, event, API)
7. Testing boundary: test อะไรเป็น unit vs integration
```

---

**ARCH-29.** Technology evaluation
> ประเมิน technology ใหม่ก่อนนำมาใช้

```
ประเมิน [technology/library/tool] สำหรับ [use case]:
1. Problem it solves: แก้ปัญหาอะไร
2. Alternatives: ทางเลือกอื่นมีอะไรบ้าง
3. Maturity: version, community size, maintenance status
4. Learning curve: ทีมต้องเรียนรู้แค่ไหน
5. Integration: เข้ากับ stack ปัจจุบันได้ง่ายไหม
6. Performance: benchmark vs current solution
7. License: compatible กับโปรเจกต์ไหม
8. Risk: ถ้าเลือกแล้วไม่ดี ถอยออกยากไหม
สรุปเป็น recommendation: ✅ Adopt / ⚠️ Trial / ❌ Avoid
```

---

**ARCH-30.** ออกแบบ API versioning strategy
> กำหนดวิธี version API

```
ออกแบบ API versioning strategy:
1. Versioning method: URL path (/v1/) vs header vs query param
2. Breaking change policy: อะไรนับเป็น breaking change
3. Deprecation process: แจ้ง deprecation อย่างไร + timeline
4. Backward compatibility: support กี่ versions
5. Migration guide: สร้าง guide สำหรับ consumers อย่างไร
6. Testing: test ว่า old versions ยังทำงาน
7. Documentation: document แต่ละ version อย่างไร
```

---

**ARCH-31.** Design review for new feature
> Review design ก่อน implement

```
Review design ของ [feature]:
Design doc/proposal: [path or paste]

ตรวจสอบ:
1. ตอบ requirements ครบไหม?
2. เข้ากับ architecture ปัจจุบันไหม?
3. มี edge cases ที่ไม่ได้ handle?
4. Performance implications?
5. Security implications?
6. Testing strategy เหมาะสม?
7. Operational concerns (monitoring, logging)?
8. ต้อง update ADR / CLAUDE.md?
ให้ feedback + suggestions
```

---

**ARCH-32.** Refactoring strategy recommendation
> แนะนำ strategy สำหรับ refactor ขนาดใหญ่

```
ต้อง refactor [module/system] เพราะ [reasons]:
ปัจจุบัน: [current state + problems]
เป้าหมาย: [desired state]

แนะนำ refactoring strategy:
1. Approach: [big bang / incremental / strangler fig / branch by abstraction]
2. เหตุผล: ทำไมเลือก approach นี้
3. Risks: ความเสี่ยงของแต่ละ approach
4. Milestones: แบ่งงานเป็น milestones ที่ deliver value ได้
5. Testing strategy: ยืนยัน behavior ไม่เปลี่ยนอย่างไร
6. Rollback plan: ถ้ามีปัญหาระหว่าง refactor
```

---

← [Developer Workflows](02_developer_workflows.md) | [Project & Business →](04_project_and_business.md)
