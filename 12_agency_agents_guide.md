# บทที่ 12: The Agency — AI Agent Personalities สำหรับ Claude Code

> Last updated: 2026-03-13 | Claude Code compatible version: latest

คู่มือการใช้งาน **The Agency** (คอลเลกชัน AI Agent Personalities แบบ open source) ร่วมกับ Claude Code
ครอบคลุมการติดตั้ง, โครงสร้าง agent, NEXUS framework สำหรับประสานงานหลาย agent,
พร้อมคู่มือเฉพาะสำหรับ 4 บทบาท: Software Engineer, PM, SA, Tester

---

## สารบัญ

- [12.1 The Agency คืออะไร](#121-the-agency-คืออะไร)
- [12.2 การติดตั้งและตั้งค่า](#122-การติดตั้งและตั้งค่า)
- [12.3 โครงสร้าง Agent และแนวคิดหลัก](#123-โครงสร้าง-agent-และแนวคิดหลัก)
- [12.4 NEXUS Framework](#124-nexus-framework)
- [12.5 คู่มือสำหรับ Software Engineer](#125-คู่มือสำหรับ-software-engineer)
- [12.6 คู่มือสำหรับ Project Manager](#126-คู่มือสำหรับ-project-manager)
- [12.7 คู่มือสำหรับ Solution Architect / SA](#127-คู่มือสำหรับ-solution-architect--sa)
- [12.8 คู่มือสำหรับ Tester / QA](#128-คู่มือสำหรับ-tester--qa)
- [12.9 Multi-Agent Orchestration ตัวอย่างจริง](#129-multi-agent-orchestration-ตัวอย่างจริง)
- [12.10 เคล็ดลับและแนวปฏิบัติที่ดี](#1210-เคล็ดลับและแนวปฏิบัติที่ดี)

---

## 12.1 The Agency คืออะไร

**The Agency** ([github.com/msitarzewski/agency-agents](https://github.com/msitarzewski/agency-agents)) คือคอลเลกชัน AI agent personalities มากกว่า 100 ตัว ที่ออกแบบมาเพื่อใช้ร่วมกับ Claude Code และ AI coding tools อื่นๆ แต่ละ agent มี identity, mission, rules, และ workflow ที่เฉพาะเจาะจง ทำให้ Claude Code ทำงานเหมือน "ผู้เชี่ยวชาญเฉพาะด้าน" แทนที่จะเป็น "AI ทั่วไป"

### เปรียบเทียบ: ไม่ใช้ vs ใช้ Agency

| ด้าน | ไม่ใช้ Agency | ใช้ Agency |
|------|:------------:|:---------:|
| **Prompt** | เขียนเอง ทุกครั้ง | เลือก agent ที่เหมาะ agent มี rules ในตัว |
| **ความสม่ำเสมอ** | ขึ้นกับว่าเขียน prompt ดีแค่ไหน | Agent มี standards ตายตัว เช่น naming, quality gates |
| **Quality Gates** | ต้องบอก Claude เอง ทุกครั้ง | Agent มี built-in checklist และ criteria |
| **บทบาท** | Claude เป็น "generalist" | Claude เป็น "specialist" — Backend Architect, API Tester ฯลฯ |
| **Multi-agent** | ต้องออกแบบ workflow เอง | NEXUS framework จัด pipeline ให้ |
| **Context passing** | ไม่มีโครงสร้าง | Handoff protocol ระหว่าง agents |

### Division Overview

Agent ทั้งหมดจัดเป็น division ตามความเชี่ยวชาญ:

| Division | จำนวน (โดยประมาณ) | ตัวอย่าง Agents |
|----------|:---------:|----------------|
| **Engineering** | 23 | Frontend Developer, Backend Architect, Senior Developer, DevOps Automator, Security Engineer |
| **Design** | 8 | UI Designer, UX Researcher, UX Architect, Brand Guardian |
| **Testing** | 8 | API Tester, Reality Checker, Evidence Collector, Performance Benchmarker, Accessibility Auditor |
| **Product** | 4 | Sprint Prioritizer, Trend Researcher, Feedback Synthesizer |
| **Project Management** | 6 | Senior Project Manager, Project Shepherd, Experiment Tracker |
| **Marketing** | 26+ | Growth Hacker, Content Creator, SEO Specialist, Social Media Strategist |
| **Sales** | 8 | Outbound Strategist, Sales Engineer, Pipeline Analyst |
| **Support** | 6 | Support Responder, Analytics Reporter, Finance Tracker, Legal Compliance Checker |
| **Spatial Computing** | 6 | XR Interface Architect, visionOS Spatial Engineer |
| **Game Development** | 13+ | Game Designer, Level Designer, Technical Artist (Unity/Unreal/Godot) |
| **Specialized** | 17+ | Agents Orchestrator, MCP Builder, Compliance Auditor, Document Generator |

### ทำไมควรใช้ Agency

1. **Consistency** — ทุกคนในทีมได้ผลลัพธ์แบบเดียวกัน เพราะ agent มี rules ตายตัว
2. **Specialization** — Claude ตอบในมุมมองของผู้เชี่ยวชาญ ไม่ใช่ generalist
3. **Quality Gates** — Agent มี built-in checklist ป้องกันงานหลุด
4. **Reusability** — ไม่ต้องเขียน prompt ใหม่ทุกครั้ง เลือก agent แล้วบอกงาน
5. **Team Scaling** — Onboard คนใหม่ง่ายขึ้น เลือก agent ที่เหมาะกับ role แล้วเริ่มทำงาน

---

## 12.2 การติดตั้งและตั้งค่า

### ติดตั้งทั้งหมด

```bash
# Clone repo
git clone https://github.com/msitarzewski/agency-agents.git

# Copy agent files ไปยัง Claude Code agents directory
cp -r agency-agents/agents/ ~/.claude/agents/
```

### ติดตั้งเฉพาะ Division ที่ต้องการ

ถ้าไม่ต้องการ agents ทั้งหมด เลือก copy เฉพาะ division:

```bash
# เฉพาะ Engineering + Testing (สำหรับ developer)
cp -r agency-agents/agents/engineering/ ~/.claude/agents/engineering/
cp -r agency-agents/agents/testing/ ~/.claude/agents/testing/

# เฉพาะ Project Management + Product (สำหรับ PM)
cp -r agency-agents/agents/project-management/ ~/.claude/agents/project-management/
cp -r agency-agents/agents/product/ ~/.claude/agents/product/
```

### ใช้ install script ของ repo

```bash
# ใช้ script ที่มากับ repo
cd agency-agents
./scripts/install.sh
```

### Agents แนะนำตามบทบาท (Quick Reference)

| บทบาท | Agents ที่ควรติดตั้ง | Division |
|-------|---------------------|----------|
| **Software Engineer** | Frontend Developer, Backend Architect, Senior Developer, Code Reviewer, DevOps Automator, Git Workflow Master | engineering |
| **Project Manager** | Senior PM, Sprint Prioritizer, Project Shepherd, Experiment Tracker | project-management, product |
| **Solution Architect** | Software Architect, Backend Architect, SRE, Database Optimizer | engineering |
| **Tester / QA** | API Tester, Reality Checker, Evidence Collector, Performance Benchmarker, Accessibility Auditor | testing |
| **ทุกบทบาท (แนะนำ)** | Agents Orchestrator | specialized |

### วิธีเรียกใช้ Agent ใน Claude Code

เมื่อติดตั้งแล้ว เรียกใช้ผ่าน `/agent` command:

```
# เรียก agent เฉพาะ
/agent:backend-architect

# หรือพิมพ์ /agent แล้วเลือกจากรายการ
/agent
```

อีกวิธีหนึ่งคือ copy เนื้อหาของ agent file แล้ววางเป็น system prompt ใน session:

```
# วิธี manual — copy agent file เข้ามาเป็น context
อ่านไฟล์ ~/.claude/agents/engineering/backend-architect.md
แล้วทำตามบทบาทและ rules ที่กำหนด

จากนั้น: [งานที่ต้องทำ]
```

> **Tip:** ถ้าใช้ agent บ่อยๆ สร้าง custom slash command ใน `.claude/commands/` เพื่อเรียกใช้ได้เร็วขึ้น
> ดูรายละเอียดที่ [บทที่ 3: Custom Slash Commands](03_advanced_usage.md)

---

## 12.3 โครงสร้าง Agent และแนวคิดหลัก

### กายวิภาคของ Agent File

ทุก agent file เป็น Markdown ที่มีโครงสร้างเดียวกัน:

```markdown
---
name: "Backend Architect"
description: "System architecture & backend design"
color: "#2563EB"
emoji: "🏗️"
vibe: "methodical, scalable, secure"
---

# Identity & Memory
คุณคือ Backend Architect ผู้เชี่ยวชาญด้าน...

# Core Mission
ออกแบบและสร้าง backend systems ที่...

# Critical Rules
1. ทุก API ต้องมี input validation
2. ใช้ parameterized queries เท่านั้น
3. ...

# Technical Deliverables
- Architecture diagrams
- API specifications
- Database schemas

# Workflow Process
1. Analyze requirements
2. Design architecture
3. Implement core components
4. Review and validate

# Communication Style
- ตอบด้วยเหตุผลทางเทคนิค
- ให้ trade-offs เสมอ

# Success Metrics
- Code coverage > 80%
- API response time < 200ms
- Zero critical vulnerabilities
```

### ส่วนประกอบ Agent และจุดประสงค์

| ส่วนประกอบ | จุดประสงค์ | ตัวอย่าง |
|-----------|-----------|---------|
| **Frontmatter** | metadata สำหรับ UI/tooling | `name`, `description`, `color`, `emoji`, `vibe` |
| **Identity & Memory** | บอก Claude ว่า "คุณคือใคร" | "คุณคือ Backend Architect ผู้เชี่ยวชาญด้าน scalable systems" |
| **Core Mission** | เป้าหมายหลักของ agent | "ออกแบบ backend ที่ปลอดภัย, scalable, maintainable" |
| **Critical Rules** | กฎที่ห้ามละเมิด | "ห้ามใช้ string concatenation ใน SQL", "ต้อง validate input ทุกครั้ง" |
| **Technical Deliverables** | output ที่คาดหวัง | "Architecture diagrams, API specs, migration scripts" |
| **Workflow Process** | ขั้นตอนการทำงาน | "1. Analyze → 2. Design → 3. Implement → 4. Validate" |
| **Communication Style** | น้ำเสียงและรูปแบบการตอบ | "ตอบตรงประเด็น ให้ trade-offs เสมอ ใช้ technical evidence" |
| **Success Metrics** | เกณฑ์วัดผลสำเร็จ | "Coverage > 80%, latency < 200ms, zero critical CVEs" |

### Quality Gates Concept

Agent หลายตัว (โดยเฉพาะใน Testing division) มี **Quality Gates** ในตัว — เป็นเงื่อนไขที่ต้องผ่านก่อนจะถือว่างานเสร็จ:

```
Quality Gate ของ Evidence Collector:
┌─────────────────────────────────────┐
│  ☐ ทุก test case มีผลลัพธ์จริง     │
│  ☐ Screenshots / logs แนบครบ        │
│  ☐ Edge cases ถูก test แล้ว         │
│  ☐ Performance baseline ถูกบันทึก   │
│  ☐ Verdict: PASS / NEEDS WORK       │
└─────────────────────────────────────┘
```

> **หลักการสำคัญ:** "Evidence Over Claims" — ไม่ว่า agent ไหนจะบอกว่า "เสร็จแล้ว"
> ต้องมีหลักฐาน (test results, screenshots, metrics) สนับสนุนเสมอ

---

## 12.4 NEXUS Framework

**NEXUS** (Network of EXperts, Unified in Strategy) คือ framework สำหรับประสานงานหลาย agents ข้ามเฟส ออกแบบมาให้ทำงานเป็น pipeline จาก Discovery ไปจนถึง Operate

### 7 Phases ของ NEXUS

```
Phase 0       Phase 1       Phase 2        Phase 3       Phase 4       Phase 5       Phase 6
DISCOVERY ──→ STRATEGY ──→ FOUNDATION ──→  BUILD   ──→  HARDEN  ──→  LAUNCH  ──→  OPERATE
   │             │             │              │             │             │             │
Research      Planning      Setup          Dev+QA        Security     Go-live      Monitor
& Analysis   & Design      Infra          Loop          & Perf       & Market     & Maintain
   │             │             │              │             │             │             │
Trend        Senior PM     DevOps        Developers    Reality      Growth       Analytics
Researcher   Sprint        Automator     Evidence      Checker      Hacker       Reporter
Feedback     Prioritizer   Frontend      Collector     Performance  Content      Infra
Synthesizer  UX Architect  Developer     API Tester    Benchmarker  Creator      Maintainer
UX           Backend       Backend                     Legal        DevOps       Support
Researcher   Architect     Architect                   Compliance   Automator    Responder
```

### รายละเอียดแต่ละ Phase

| Phase | ชื่อ | เป้าหมาย | Agents หลัก |
|:-----:|------|----------|-------------|
| 0 | **Discovery** | วิจัยตลาด ฟัง feedback วิเคราะห์ technology | Trend Researcher, Feedback Synthesizer, UX Researcher, Analytics Reporter |
| 1 | **Strategy** | วางแผน กำหนด scope ออกแบบ architecture | Senior PM, Sprint Prioritizer, UX Architect, Backend Architect, Brand Guardian |
| 2 | **Foundation** | ตั้ง infra, CI/CD, project structure | DevOps Automator, Frontend Developer, Backend Architect, Infrastructure Maintainer |
| 3 | **Build** | พัฒนา + ทดสอบแบบ loop | Developers (FE/BE) + Evidence Collector (QA loop) |
| 4 | **Harden** | ตรวจคุณภาพรอบสุดท้าย security, performance | Reality Checker, Performance Benchmarker, API Tester, Legal Compliance Checker |
| 5 | **Launch** | Deploy, marketing, go-live | Growth Hacker, Content Creator, DevOps Automator, Marketing agents |
| 6 | **Operate** | Monitor, support, iterate | Analytics Reporter, Infrastructure Maintainer, Support Responder |

### 3 Deployment Modes

NEXUS ไม่จำเป็นต้องใช้ทั้ง 7 phases เสมอ เลือก mode ตามขนาดงาน:

| Mode | จำนวน Agents | ระยะเวลา | เหมาะกับ | Phases ที่ใช้ |
|------|:-----------:|:--------:|---------|:----------:|
| **NEXUS-Full** | ทั้งหมด | 12-24 สัปดาห์ | Product lifecycle เต็มรูปแบบ | 0-6 ทุก phase |
| **NEXUS-Sprint** | 15-25 | 2-6 สัปดาห์ | Feature ใหม่ หรือ MVP | 1-4 (Strategy → Harden) |
| **NEXUS-Micro** | 5-10 | 1-5 วัน | Bug fix, audit, investigation | เลือกเฉพาะ phase ที่จำเป็น |

### Dev↔QA Loop (Phase 3)

ในเฟส Build, NEXUS ใช้ Dev-QA Loop ที่ทำซ้ำจนกว่าจะผ่าน:

```
┌────────────────────────────────────────────────────┐
│                  Dev-QA Loop                        │
│                                                     │
│   Developer ──→ Implement ──→ Evidence Collector    │
│       ↑                            │                │
│       │                      Run Tests              │
│       │                            │                │
│       │                     ┌──────┴──────┐         │
│       │                     │             │         │
│       │                   PASS          FAIL        │
│       │                     │             │         │
│       │                 ✓ Done      Feedback        │
│       │                             to Dev          │
│       │                               │             │
│       └───────────────────────────────┘             │
│                                                     │
│   สูงสุด 3 รอบ — ถ้าไม่ผ่าน escalate to lead       │
└────────────────────────────────────────────────────┘
```

**สถิติจาก Agency:**
- Dev-QA Loop จับ defect ได้ 95% ก่อน integration
- Multi-agent projects ล้มเหลวที่ handoff boundaries 73% ของเวลา (ถ้าไม่มี structured coordination)
- Parallel execution ลดเวลาได้ 40-60%

### Handoff Protocol

เมื่อส่งต่องานระหว่าง agents ให้ใช้ structured handoff:

```markdown
## Handoff: [Agent ต้นทาง] → [Agent ปลายทาง]

### Context
- สิ่งที่ทำเสร็จแล้ว: [สรุป]
- ไฟล์ที่เกี่ยวข้อง: [รายชื่อ]
- Decisions ที่ตัดสินใจแล้ว: [รายการ]

### Task
- สิ่งที่ต้องทำต่อ: [รายละเอียด]
- Acceptance criteria: [เกณฑ์]

### Constraints
- ข้อจำกัด: [รายการ]
- Dependencies: [รายการ]
```

> **Tip:** ใน Claude Code ทำ handoff ง่ายๆ ด้วยการ copy output จาก agent หนึ่ง
> แล้ว paste เป็น context ให้ agent ถัดไป ใน session ใหม่

---

## 12.5 คู่มือสำหรับ Software Engineer

### Agents แนะนำ

| Agent | Division | ใช้ทำอะไร |
|-------|----------|----------|
| **Frontend Developer** | engineering | สร้าง UI components, responsive design, CSS systems |
| **Backend Architect** | engineering | ออกแบบ API, database schema, system architecture |
| **Senior Developer** | engineering | Implementation ที่ซับซ้อน, refactoring, code patterns |
| **Code Reviewer** | engineering | Review code quality, best practices, security |
| **DevOps Automator** | engineering | CI/CD pipeline, Docker, deployment automation |
| **Git Workflow Master** | engineering | Git branching strategy, merge conflicts, workflow |

### ตัวอย่าง Prompt

**1. Implement Feature ด้วย Backend Architect**

```
/agent:backend-architect

ออกแบบและ implement Order Management API:

Requirements:
- POST /api/v1/orders — สร้าง order ใหม่
- GET /api/v1/orders/:id — ดู order
- GET /api/v1/orders — list พร้อม pagination + filter by status
- PATCH /api/v1/orders/:id/status — เปลี่ยน status

Constraints:
- ใช้ Go Fiber + GORM ตาม pattern ใน internal/handler/product_handler.go
- Status: pending → confirmed → shipped → delivered | cancelled
- ต้อง validate state transitions (เช่น delivered → pending ไม่ได้)
- Optimistic locking สำหรับ concurrent updates

สร้าง handler, service, repository, migration ตาม three-layer architecture
แล้วรัน go build ./... และ go test ./... เพื่อยืนยัน
```

**2. Code Review ด้วย Code Reviewer**

```
/agent:code-reviewer

Review PR changes ใน branch feature/payment-gateway:

ดู git diff main...feature/payment-gateway แล้ว review ตามเกณฑ์:
1. Security: SQL injection, input validation, authentication
2. Performance: N+1 queries, missing indexes, unnecessary allocations
3. Maintainability: naming conventions, error handling, code duplication
4. Testing: test coverage, edge cases, mock appropriateness

ให้ feedback เป็น 3 ระดับ:
- 🔴 MUST FIX — ต้องแก้ก่อน merge
- 🟡 SHOULD FIX — แนะนำให้แก้
- 🟢 NICE TO HAVE — ปรับปรุงได้
```

**3. Debug ด้วย Senior Developer**

```
/agent:senior-developer

Debug: API endpoint GET /api/v1/products ตอบช้ามาก (>3 วินาที)
เฉพาะเมื่อมี filter category + price range พร้อมกัน

1. อ่าน internal/handler/product_handler.go
   และ internal/repository/product_repository.go
2. วิเคราะห์ query ที่ generate จาก GORM
3. ตรวจ database indexes ที่มีอยู่
4. เสนอวิธีแก้ไข พร้อมประมาณการ improvement
5. Implement แล้วรัน benchmark เปรียบเทียบ before/after
```

**4. CI/CD Setup ด้วย DevOps Automator**

```
/agent:devops-automator

สร้าง GitHub Actions pipeline สำหรับ Go + React monorepo:

Stages:
1. Lint: golangci-lint + eslint
2. Test: go test + vitest (parallel)
3. Build: go build + npm run build
4. Docker: build + push to registry
5. Deploy: staging auto, production manual approval

Requirements:
- Cache go modules + node_modules
- Run tests in parallel
- Fail fast on lint errors
- Notify Slack on failure
- ใช้ matrix strategy สำหรับ Go 1.21 + 1.22
```

### Workflow: Backend Architect → Senior Developer → Code Reviewer

```
                           ตัวอย่าง: สร้าง Payment Gateway Feature

Session 1: Backend Architect                    Session 2: Senior Developer
┌──────────────────────────┐                   ┌──────────────────────────┐
│ • ออกแบบ API spec        │                   │ • Implement ตาม spec     │
│ • กำหนด database schema  │ ── handoff ──→    │ • สร้าง handler/service  │
│ • เลือก payment provider │                   │ • เขียน unit tests       │
│ • ระบุ security reqs     │                   │ • รัน build + test       │
└──────────────────────────┘                   └──────────┬───────────────┘
                                                          │
                                                       handoff
                                                          │
                                               ┌──────────▼───────────────┐
                                               │ Session 3: Code Reviewer │
                                               │ • Review security        │
                                               │ • Review performance     │
                                               │ • Review test coverage   │
                                               │ • Verdict: PASS / FIX    │
                                               └──────────────────────────┘
```

### NEXUS-Micro ตัวอย่าง: Bug Fix Flow

```
/agent:backend-architect

NEXUS-Micro: Bug Fix

Bug: Payment webhook ไม่ update order status เมื่อ payment สำเร็จ
Error log: "record not found" ใน UpdateOrderStatus

Phase 1 — Diagnose:
- อ่าน internal/handler/webhook_handler.go
- อ่าน internal/service/order_service.go
- ตรวจ log ที่เกี่ยวข้อง

Phase 2 — Fix:
- Implement fix
- เขียน regression test

Phase 3 — Validate:
- รัน go test ./internal/service/... -v
- ตรวจว่า fix ไม่กระทบ endpoint อื่น

ให้ root cause analysis + fix ในรอบเดียว
```

---

## 12.6 คู่มือสำหรับ Project Manager

### Agents แนะนำ

| Agent | Division | ใช้ทำอะไร |
|-------|----------|----------|
| **Senior Project Manager** | project-management | วางแผนโปรเจกต์, risk management, resource allocation |
| **Sprint Prioritizer** | product | จัด priority ด้วย RICE scoring, MoSCoW classification |
| **Project Shepherd** | project-management | ติดตาม progress, ดูแลงานให้เดินหน้า, remove blockers |
| **Experiment Tracker** | project-management | ติดตาม A/B tests, experiments, hypothesis validation |
| **Executive Summary Generator** | support | สร้างรายงานสรุปสำหรับผู้บริหาร |

### ตัวอย่าง Prompt

**1. Sprint Planning ด้วย Sprint Prioritizer**

```
/agent:sprint-prioritizer

ช่วยจัด priority สำหรับ Sprint 14 (2 สัปดาห์):

Backlog items:
1. Payment gateway integration (Stripe)
2. User profile page redesign
3. Order history export to CSV
4. Push notification system
5. Admin dashboard — sales analytics
6. Password reset via SMS
7. Product image optimization (WebP)
8. API rate limiting

Team capacity: 3 developers, 40 story points

ให้:
- RICE score สำหรับแต่ละ item
- MoSCoW classification
- แนะนำ sprint scope ที่เหมาะสม
- ระบุ dependencies ระหว่าง items
- Risk assessment สำหรับ items ที่เลือก
```

**2. Progress Report ด้วย Executive Summary Generator**

```
/agent:executive-summary-generator

สร้าง Weekly Status Report สำหรับ Sprint 13:

ข้อมูล:
- Planned: 35 story points
- Completed: 28 story points (80%)
- Carry-over: 7 points (2 items: search optimization, email templates)
- Blockers: Payment provider API documentation ยังไม่ครบ
- Risks: Performance regression หลัง deploy feature X

ให้:
- Executive summary (ไม่เกิน 200 คำ)
- ใช้ SCQA framework (Situation, Complication, Question, Answer)
- Traffic light status (Green/Amber/Red)
- Action items พร้อม owner
- Next sprint preview
```

**3. Risk Assessment ด้วย Senior Project Manager**

```
/agent:senior-project-manager

ประเมิน risk สำหรับ Q2 Release (go-live: 30 เม.ย.):

Scope:
- Payment gateway (Stripe + PromptPay)
- Multi-tenant architecture migration
- Mobile app v2.0 launch

Current status:
- Payment: 70% done, ติด 3rd party API delay
- Multi-tenant: 40% done, schema migration ยังไม่เสร็จ
- Mobile: 60% done, design ยังไม่ sign-off

ให้:
- Risk matrix (probability x impact)
- Mitigation plan สำหรับ top 3 risks
- Go/No-go recommendation
- Contingency plan ถ้า delay 2 สัปดาห์
```

**4. Backlog Prioritization ด้วย Sprint Prioritizer**

```
/agent:sprint-prioritizer

อ่าน docs/backlog.md แล้ว prioritize ตามเกณฑ์:

Criteria weights:
- Business value: 40%
- User impact: 30%
- Technical risk: 20%
- Effort: 10%

Constraints:
- ต้องทำ security items ก่อน Q2 audit
- Mobile features ต้องเสร็จก่อน app store submission (15 เม.ย.)
- Infrastructure items ไม่ควรเกิน 20% ของ sprint capacity

ให้ output เป็นตารางพร้อม ranked list
```

### ตัวอย่าง: Startup MVP Workflow (NEXUS-Sprint)

จาก Agency repo examples — สร้าง RetroBoard MVP ใน 4 สัปดาห์:

```
สัปดาห์ 1: Strategy + Foundation
├── Senior PM: กำหนด scope, create project plan
├── Sprint Prioritizer: จัด priority MVP features
├── UX Architect: wireframe core flows
└── DevOps Automator: setup CI/CD + staging env

สัปดาห์ 2-3: Build (Dev-QA Loop)
├── Backend Architect: API design + implementation
├── Frontend Developer: UI components + pages
├── Evidence Collector: ทดสอบทุก feature ที่เสร็จ
└── Project Shepherd: track progress daily

สัปดาห์ 4: Harden + Launch
├── Reality Checker: final quality check
├── Performance Benchmarker: load test
├── DevOps Automator: production deploy
└── Executive Summary Generator: launch report
```

---

## 12.7 คู่มือสำหรับ Solution Architect / SA

### Agents แนะนำ

| Agent | Division | ใช้ทำอะไร |
|-------|----------|----------|
| **Software Architect** | engineering | ออกแบบ system architecture, ตัดสินใจ technology |
| **Backend Architect** | engineering | API design, database schema, service patterns |
| **SRE** | engineering | Reliability, monitoring, incident management, SLO/SLI |
| **Database Optimizer** | engineering | Query optimization, indexing strategy, schema design |
| **Security Engineer** | engineering | Security architecture, threat modeling, vulnerability assessment |

### ตัวอย่าง Prompt

**1. Architecture Design ด้วย Software Architect**

```
/agent:software-architect

ออกแบบ architecture สำหรับ E-commerce Platform:

Requirements:
- รองรับ 10,000 concurrent users
- Multi-tenant (แยก data per merchant)
- Payment: Stripe + local (PromptPay, TrueMoney)
- Real-time: order status updates, inventory sync
- CDN: product images, static assets

Tech constraints:
- Backend: Go (existing team expertise)
- Frontend: React + TypeScript
- Database: PostgreSQL (primary) + Redis (cache)
- Infrastructure: Docker + Kubernetes

ให้:
1. High-level architecture diagram (ASCII)
2. Component diagram — services และ communication
3. Data flow diagram — order lifecycle
4. Technology decisions พร้อม trade-offs
5. Scalability strategy (horizontal vs vertical)
6. ระบุ single points of failure และวิธี mitigate
```

**2. Architecture Decision Record (ADR) ด้วย Software Architect**

```
/agent:software-architect

สร้าง ADR สำหรับ: เลือก message queue สำหรับ async processing

Context:
- ระบบปัจจุบันใช้ synchronous API calls ทุกอย่าง
- ปัญหา: payment webhook processing ช้า ทำให้ response time สูง
- ต้องการแยก heavy processing ออกจาก request cycle
- ทีมมี 3 developers ไม่เคยใช้ message queue มาก่อน

Options ที่พิจารณา:
1. RabbitMQ
2. Apache Kafka
3. Redis Streams
4. NATS

ให้ ADR format:
- Title, Status, Context
- Decision + Rationale
- Consequences (positive + negative)
- Alternatives considered พร้อมเหตุผลที่ไม่เลือก
```

**3. Production Readiness Review ด้วย SRE**

```
/agent:sre

ทำ Production Readiness Review สำหรับ Payment Service:

ตรวจสอบ:
1. Reliability
   - SLO/SLI ที่กำหนด (availability, latency, error rate)
   - Circuit breaker สำหรับ external dependencies
   - Retry strategy + backoff
   - Graceful degradation plan

2. Observability
   - Logging: structured logs, correlation IDs
   - Metrics: business + technical metrics
   - Tracing: distributed tracing setup
   - Alerting: alert rules + runbooks

3. Security
   - Secrets management
   - TLS configuration
   - Rate limiting
   - Input validation

4. Operational
   - Deployment rollback plan
   - Database migration strategy
   - Capacity planning
   - Disaster recovery

อ่าน code ใน internal/service/payment/ แล้วให้ checklist
พร้อม status (✅ Ready / ⚠️ Needs Work / ❌ Missing) สำหรับแต่ละข้อ
```

**4. Database Optimization ด้วย Database Optimizer**

```
/agent:database-optimizer

วิเคราะห์ database performance สำหรับ orders table:

ปัญหา:
- Query เวลา list orders with filters ช้า (>2s)
- Table มี ~5M rows
- ใช้ GORM กับ PostgreSQL

ให้:
1. อ่าน migration files ใน migrations/
2. วิเคราะห์ indexes ที่มีอยู่
3. ตรวจ GORM queries ใน internal/repository/order_repository.go
4. แนะนำ indexes ที่ควรเพิ่ม
5. ตรวจว่ามี N+1 query problem ไหม
6. แนะนำ query optimization
7. พิจารณา partitioning strategy ถ้าจำเป็น
```

### Workflow: Software Architect → Backend Architect → SRE

```
Session 1: Software Architect          Session 2: Backend Architect
┌─────────────────────────────┐       ┌─────────────────────────────┐
│ • High-level architecture   │       │ • API specification         │
│ • Technology decisions      │  ──→  │ • Database schema           │
│ • Component boundaries      │       │ • Service implementation    │
│ • ADR documentation         │       │ • Integration patterns      │
└─────────────────────────────┘       └──────────┬──────────────────┘
                                                  │
                                               handoff
                                                  │
                                      ┌───────────▼──────────────────┐
                                      │ Session 3: SRE               │
                                      │ • Production readiness check │
                                      │ • Monitoring setup           │
                                      │ • SLO/SLI definition         │
                                      │ • Runbook creation           │
                                      └──────────────────────────────┘
```

---

## 12.8 คู่มือสำหรับ Tester / QA

### Agents แนะนำ

| Agent | Division | ใช้ทำอะไร |
|-------|----------|----------|
| **API Tester** | testing | ทดสอบ API endpoints 7 จุด: status, headers, body, auth, validation, edge cases, performance |
| **Reality Checker** | testing | Final integration testing, ค่า default = "NEEDS WORK" ต้องพิสูจน์ว่าผ่าน |
| **Evidence Collector** | testing | รวบรวมหลักฐานการทดสอบ: screenshots, logs, test results |
| **Performance Benchmarker** | testing | Load testing, benchmark, performance profiling |
| **Accessibility Auditor** | testing | ตรวจ WCAG compliance, keyboard navigation, screen reader |

### ตัวอย่าง Prompt

**1. API Test Suite ด้วย API Tester**

```
/agent:api-tester

ทดสอบ Order API ครบ 7 จุด:

Endpoints:
- POST /api/v1/orders
- GET /api/v1/orders/:id
- GET /api/v1/orders?status=pending&page=1
- PATCH /api/v1/orders/:id/status

ตรวจสอบแต่ละ endpoint:
1. Status code — ถูกต้องตาม spec (200, 201, 400, 401, 404)
2. Response headers — Content-Type, pagination headers
3. Response body — schema validation, required fields
4. Authentication — ต้อง 401 เมื่อไม่มี token
5. Input validation — invalid data ต้อง 400 พร้อม error message
6. Edge cases — empty body, extra fields, SQL injection attempts
7. Performance — response time < 200ms

สร้าง curl commands สำหรับทุก test case
รัน test แล้วรายงานผลเป็นตาราง
```

**2. Reality Check ด้วย Reality Checker**

```
/agent:reality-checker

ทำ Final Reality Check สำหรับ Payment Feature ก่อน release:

Default verdict: NEEDS WORK (ต้องพิสูจน์ว่าผ่าน)

ตรวจสอบ:
1. Functional: ทำ payment ได้จริง end-to-end
2. Error handling: payment fail, timeout, duplicate, refund
3. Data integrity: order status ตรงกับ payment status
4. Concurrency: 2 users ซื้อ item สุดท้ายพร้อมกัน
5. Security: PCI compliance basics, no card data in logs
6. UX: error messages เข้าใจง่าย, loading states

สำหรับแต่ละข้อ:
- ต้องมีหลักฐาน (test output, screenshot, log) ถึงจะ PASS
- ถ้าไม่มีหลักฐาน = NEEDS WORK โดยอัตโนมัติ
- ถ้ามี 1 ข้อ FAIL = verdict ทั้งหมดเป็น NEEDS WORK
```

**3. Performance Benchmark ด้วย Performance Benchmarker**

```
/agent:performance-benchmarker

Benchmark Order API ภายใต้ load:

Test scenarios:
1. Baseline: 1 concurrent user, 100 requests
2. Normal load: 50 concurrent users, 1000 requests
3. Peak load: 200 concurrent users, 5000 requests
4. Stress test: เพิ่ม users จนกว่า error rate > 5%

Metrics ที่ต้องเก็บ:
- p50, p95, p99 response time
- Throughput (requests/sec)
- Error rate
- CPU / Memory usage

Acceptance criteria:
- p95 < 500ms ที่ normal load
- p99 < 1s ที่ peak load
- Error rate < 1% ที่ normal load
- Zero errors ที่ baseline

สร้าง test script แล้วรัน benchmark
รายงานผลพร้อม comparison table
```

**4. Accessibility Audit ด้วย Accessibility Auditor**

```
/agent:accessibility-auditor

ตรวจ WCAG 2.1 Level AA compliance สำหรับ checkout flow:

Pages:
1. /cart — ตะกร้าสินค้า
2. /checkout — กรอกข้อมูลจัดส่ง
3. /checkout/payment — เลือกวิธีชำระเงิน
4. /checkout/confirm — ยืนยันคำสั่งซื้อ

ตรวจ:
- Semantic HTML: headings hierarchy, landmarks, labels
- Keyboard navigation: tab order, focus management, skip links
- Color contrast: text vs background ratio ≥ 4.5:1
- Screen reader: alt text, aria-labels, live regions
- Forms: error messages, required field indicators
- Responsive: accessible ที่ทุก breakpoint

รายงานเป็น severity level:
- Critical: ใช้งานไม่ได้ด้วย assistive technology
- Major: ใช้งานยาก แต่ยังพอทำได้
- Minor: ปรับปรุงได้ แต่ไม่กระทบ usability
```

### ปรัชญาของ Reality Checker

Reality Checker เป็น agent ที่เข้มงวดที่สุดใน Testing division:

```
┌──────────────────────────────────────────────────────────────┐
│  Reality Checker Philosophy                                   │
│                                                               │
│  Default verdict:  ❌ NEEDS WORK                              │
│                                                               │
│  ต้องพิสูจน์ 3 อย่างถึงจะ ✅ PASS:                            │
│  1. Evidence — มีหลักฐาน (test results, logs, screenshots)   │
│  2. Coverage — ครอบคลุมทุก requirement ใน spec                │
│  3. Edge Cases — ทดสอบ edge cases แล้วไม่พัง                  │
│                                                               │
│  ❌ "มันน่าจะใช้ได้" → ไม่นับ                                  │
│  ❌ "ผม test แล้วมันผ่าน" (ไม่มีหลักฐาน) → ไม่นับ             │
│  ✅ "นี่คือ test output ที่แสดงว่าผ่านทุก case" → นับ          │
└──────────────────────────────────────────────────────────────┘
```

### Workflow: API Tester → Reality Checker → Evidence Collector

```
Session 1: API Tester                  Session 2: Reality Checker
┌───────────────────────────┐         ┌───────────────────────────┐
│ • ทดสอบ 7 จุดต่อ endpoint │         │ • Integration test        │
│ • สร้าง curl commands     │  ──→    │ • End-to-end scenarios    │
│ • รายงาน PASS/FAIL       │         │ • Verdict: PASS/NEEDS WORK│
│ • รวม test output         │         │ • ต้องมีหลักฐานทุกข้อ     │
└───────────────────────────┘         └──────────┬────────────────┘
                                                  │
                                               handoff
                                                  │
                                      ┌───────────▼────────────────┐
                                      │ Session 3: Evidence        │
                                      │ Collector                  │
                                      │ • รวบรวมหลักฐานทั้งหมด     │
                                      │ • จัด format เป็นรายงาน    │
                                      │ • Checklist สมบูรณ์        │
                                      │ • Final PASS/FAIL verdict  │
                                      └────────────────────────────┘
```

---

## 12.9 Multi-Agent Orchestration ตัวอย่างจริง

### NEXUS-Full Prompt (จาก QUICKSTART.md)

สำหรับ product lifecycle เต็มรูปแบบ:

```
/agent:agents-orchestrator

NEXUS-Full Pipeline สำหรับ RetroBoard — Sprint retrospective tool

Product vision:
- Web app ให้ทีมทำ sprint retrospective ออนไลน์
- Real-time collaboration (หลายคนพิมพ์พร้อมกัน)
- Anonymous mode สำหรับ feedback ที่ sensitive
- Export to Markdown/PDF

เริ่ม Phase 0 — Discovery:
1. Trend Researcher: วิเคราะห์ retro tools ในตลาด (Miro, FunRetro, EasyRetro)
2. UX Researcher: ระบุ pain points ของ retro แบบเดิม
3. Feedback Synthesizer: สรุป user needs จาก research

แล้วต่อด้วย Phase 1 — Strategy:
4. Senior PM: สร้าง project plan, define MVP scope
5. Sprint Prioritizer: prioritize features ด้วย RICE
6. Backend Architect: เลือก tech stack, ออกแบบ architecture

ดำเนินการ phase ต่อไปตาม NEXUS pipeline
ส่ง handoff ที่ structured ระหว่าง phases
```

### NEXUS-Sprint ตัวอย่าง: 4-Week RetroBoard MVP

```
/agent:agents-orchestrator

NEXUS-Sprint: RetroBoard MVP — 4 สัปดาห์

Week 1 — Strategy + Foundation:
- Senior PM: scope + timeline
- UX Architect: wireframe 3 core screens (board, create, results)
- Backend Architect: API design (WebSocket + REST)
- DevOps Automator: Docker + CI/CD + staging

Week 2-3 — Build (Dev-QA Loop):
- Backend Architect: implement API + WebSocket server
- Frontend Developer: React components + real-time UI
- Evidence Collector: test ทุก feature ที่ complete
  → Dev-QA loop สูงสุด 3 รอบต่อ feature

Week 4 — Harden + Launch:
- Reality Checker: final quality check
- Performance Benchmarker: concurrent users test
- API Tester: endpoint security + validation
- DevOps Automator: production deploy

Handoff format ระหว่าง agents:
สรุป (สิ่งที่ทำเสร็จ) → Task (สิ่งที่ต้องทำต่อ) → Constraints
```

### NEXUS-Micro ตัวอย่าง

**Bug Fix:**

```
NEXUS-Micro: Fix duplicate order bug

Agents: Backend Architect → API Tester → Evidence Collector

1. Backend Architect:
   - วิเคราะห์ race condition ใน POST /api/v1/orders
   - Implement idempotency key + database constraint
   - เขียน regression test

2. API Tester:
   - ส่ง 10 requests พร้อมกันด้วย same idempotency key
   - ยืนยันว่าได้ order เดียว (ไม่ duplicate)
   - ทดสอบ error response เมื่อ key ซ้ำ

3. Evidence Collector:
   - รวม test output + before/after comparison
   - Verdict: PASS/NEEDS WORK
```

**Compliance Audit:**

```
NEXUS-Micro: PDPA Compliance Audit

Agents: Legal Compliance Checker → Executive Summary Generator

1. Legal Compliance Checker:
   - ตรวจ data collection points (forms, APIs)
   - ตรวจ consent management
   - ตรวจ data retention policy ใน code
   - ตรวจ data export/delete capability (right to be forgotten)
   - สร้าง findings list พร้อม severity

2. Executive Summary Generator:
   - สรุปผลเป็น executive report
   - Traffic light status per area
   - Priority action items for remediation
```

**Performance Investigation:**

```
NEXUS-Micro: API Latency Investigation

Agents: Performance Benchmarker → Infrastructure Maintainer → DevOps Automator

1. Performance Benchmarker:
   - รัน benchmark ทุก critical endpoint
   - ระบุ endpoint ที่ช้ากว่า SLO
   - Profile เพื่อหา bottleneck

2. Infrastructure Maintainer:
   - ตรวจ resource utilization (CPU, memory, connections)
   - ตรวจ database connection pool
   - ตรวจ cache hit rate

3. DevOps Automator:
   - Implement fixes (scaling, caching, query optimization)
   - Deploy + validate improvement
```

### Multi-Agent Coordination Flow

```
┌──────────────────────────────────────────────────────────────────────┐
│                    Multi-Agent Coordination                          │
│                                                                      │
│  User                                                                │
│   │                                                                  │
│   ▼                                                                  │
│  ┌──────────────────┐                                                │
│  │ Orchestrator     │  ← เลือก agents + กำหนด sequence               │
│  │ (Agents          │                                                │
│  │  Orchestrator)   │                                                │
│  └────────┬─────────┘                                                │
│           │                                                          │
│     ┌─────┼─────────────────┐                                        │
│     ▼     ▼                 ▼                                        │
│  Agent A    Agent B      Agent C                                     │
│  (Design)   (Build)      (Test)                                      │
│     │         │            │                                         │
│     │    ┌────┴────┐       │                                         │
│     │    ▼         ▼       │                                         │
│     │  Code     Code       │                                         │
│     │    │         │       │                                         │
│     │    └────┬────┘       │                                         │
│     │         │            │                                         │
│     │    ┌────▼────┐       │                                         │
│     │    │  Build  │───────┘                                         │
│     │    │  Test   │     Test results                                 │
│     │    └────┬────┘       │                                         │
│     │         │       ┌────▼────┐                                    │
│     │         │       │ Verdict │                                    │
│     │         │       │PASS/FAIL│                                    │
│     │         │       └────┬────┘                                    │
│     │         │            │                                         │
│     └────┬────┴────────────┘                                         │
│          ▼                                                           │
│       Handoff                                                        │
│       Summary                                                        │
│          │                                                           │
│          ▼                                                           │
│        User                                                          │
└──────────────────────────────────────────────────────────────────────┘
```

### Quick Reference: สถานการณ์ → Agent

| สถานการณ์ | Primary Agent | Support Agents |
|----------|---------------|----------------|
| เริ่มโปรเจกต์ใหม่ | Agents Orchestrator (Full Pipeline) | — |
| สร้าง feature | Agents Orchestrator (Dev-QA Loop) | Developer + Evidence Collector |
| แก้ bug | Backend/Frontend Developer | API Tester, Evidence Collector |
| Code review | Code Reviewer | Security Engineer |
| Sprint planning | Sprint Prioritizer | Senior PM |
| Architecture design | Software Architect | Backend Architect |
| Production deploy | DevOps Automator | SRE |
| Performance issue | Performance Benchmarker | Infrastructure Maintainer, DevOps Automator |
| Security audit | Security Engineer | Legal Compliance Checker |
| Compliance check | Legal Compliance Checker | Executive Summary Generator |
| Monthly reporting | Executive Summary Generator | Analytics Reporter, Finance Tracker |
| UX improvement | UX Researcher → UX Architect | Frontend Developer, Evidence Collector |

---

## 12.10 เคล็ดลับและแนวปฏิบัติที่ดี

### ข้อผิดพลาดที่พบบ่อย

| ข้อผิดพลาด | ปัญหาที่เกิด | วิธีแก้ |
|-----------|-------------|---------|
| ใช้ agent ไม่ตรงกับงาน | ได้ผลลัพธ์ที่ไม่ตรงจุด เช่น ให้ API Tester ออกแบบ architecture | ดูตาราง Quick Reference ใน 12.9 เลือก agent ที่ตรง |
| ไม่ส่ง context ในการ handoff | Agent ถัดไปไม่รู้ว่างาน agent ก่อนหน้าทำอะไรไป | ใช้ Handoff Protocol template ใน 12.4 |
| ใช้ agent ทุกงาน แม้งานง่าย | เปลือง token, ช้ากว่าทำตรง | งานง่ายเช่น rename, format ใช้ Claude ตรง ไม่ต้องเรียก agent |
| Dev-QA loop ไม่จำกัดรอบ | วนแก้ไม่จบ เสีย token มาก | ตั้ง max 3 รอบ ถ้าไม่ผ่านให้ escalate |
| ใช้ NEXUS-Full กับงานเล็ก | Overhead สูง ได้ไม่คุ้มเสีย | bug fix ใช้ NEXUS-Micro, feature ใช้ NEXUS-Sprint |
| ไม่ review output ของ agent | Agent อาจ hallucinate หรือไม่ตรง requirements | ตรวจ output ทุกครั้ง โดยเฉพาะ code ที่ generate มา |

### เมื่อไรไม่ควรใช้ Agent

| สถานการณ์ | ทำไม | ทำอะไรแทน |
|----------|------|----------|
| คำถามเร็วๆ (1 บรรทัด) | Agent เพิ่ม system prompt tokens โดยไม่จำเป็น | ถาม Claude ตรงๆ |
| Rename / find-replace | Deterministic task ไม่ต้องการ "personality" | ใช้ IDE หรือ `sed` |
| อ่านโค้ดเพื่อทำความเข้าใจ | Claude ตรงๆ ก็อ่านได้ดี | `claude -p "อ่าน X แล้วอธิบาย"` |
| งานที่มี spec ชัดเจนและสั้น | Agent overhead ไม่คุ้ม | ให้ task ตรง + spec |
| Debug ง่ายๆ (typo, syntax error) | ไม่ต้องการ specialist | Claude ตรงๆ + `go build` |

### การ Customize Agent Files สำหรับทีม

สามารถแก้ไข agent files ใน `~/.claude/agents/` เพื่อให้ตรงกับ conventions ของทีม:

```markdown
# ตัวอย่าง: เพิ่ม team-specific rules ใน Backend Architect

# Critical Rules (เพิ่มจากของเดิม)
- ใช้ Go Fiber framework (ไม่ใช่ Gin หรือ Echo)
- Database: PostgreSQL + GORM เท่านั้น
- Three-layer architecture: handler → service → repository
- Error handling: ใช้ custom error types ใน pkg/errors/
- Naming: ตาม project .golangci.yml
- Testing: testify/assert + mockery
```

> **Tip:** เก็บ customized agents ใน repo ของทีม (เช่น `team-agents/`)
> แล้ว symlink ไปที่ `~/.claude/agents/` เพื่อให้ทุกคนใช้ version เดียวกัน

### Context Passing ระหว่าง Agents

เนื่องจากแต่ละ agent ทำงานใน session แยกกัน การส่ง context ต้องทำ manual:

```
วิธีที่ 1: Copy-paste output
─────────────────────────────
1. รัน agent A → ได้ output
2. Copy output ที่เกี่ยวข้อง
3. เปิด session ใหม่ → เรียก agent B
4. Paste output ของ A เป็น context ให้ B

วิธีที่ 2: ใช้ไฟล์ intermediate
─────────────────────────────────
1. รัน agent A → ให้ save output เป็นไฟล์ (เช่น docs/api-spec.md)
2. เปิด session ใหม่ → เรียก agent B
3. บอก agent B ให้อ่านไฟล์ docs/api-spec.md

วิธีที่ 3: DEVLOG
──────────────────
1. รัน agent A → บันทึกสรุปใน DEVLOG.md
2. รัน agent B → ให้อ่าน DEVLOG.md เพื่อรับ context
```

### Cost Considerations

Agent files เพิ่ม system prompt tokens ใน context:

| ขนาด Agent File | Tokens เพิ่ม (โดยประมาณ) | Cost เพิ่มต่อ session (Sonnet) |
|:---------------:|:----------------------:|:----------------------------:|
| เล็ก (1-2 KB) | ~500-800 | ~$0.002 |
| กลาง (3-5 KB) | ~1,000-2,000 | ~$0.005 |
| ใหญ่ (8-15 KB) | ~3,000-5,000 | ~$0.015 |

> **กฎทั่วไป:** cost ที่เพิ่มจาก agent file น้อยมากเมื่อเทียบกับ code context
> แต่ถ้าใช้หลาย agents ใน session เดียวกัน tokens จะสะสม — ใช้ `/compact` เป็นระยะ
>
> ดูรายละเอียดการจัดการต้นทุนที่ [บทที่ 11: Cost Optimization](11_cost_optimization.md)

---

## การอ้างอิงข้ามบท

| บท | หัวข้อที่เกี่ยวข้อง | ลิงก์ |
|----|-------------------|------|
| บทที่ 1 | พื้นฐาน Claude Code, CLAUDE.md, คำสั่งเบื้องต้น | [01_intro.md](01_intro.md) |
| บทที่ 2 | Prompt patterns, agentic workflow, Plan Mode | [02_agentic_mastery.md](02_agentic_mastery.md) |
| บทที่ 3 | Custom slash commands, MCP servers, multi-repo | [03_advanced_usage.md](03_advanced_usage.md) |
| บทที่ 5 | Spec-driven development (ใช้ร่วมกับ agent workflows) | [05_spec_driven_dev.md](05_spec_driven_dev.md) |
| บทที่ 6 | Architecture design (ใช้ร่วมกับ Software Architect agent) | [06_architecture_design.md](06_architecture_design.md) |
| บทที่ 7 | QA, Security, Review (ใช้ร่วมกับ Testing agents) | [07_qa_security_review.md](07_qa_security_review.md) |
| บทที่ 8 | Roles & Responsibilities (เปรียบเทียบ role-based usage) | [08_roles_responsibilities.md](08_roles_responsibilities.md) |
| บทที่ 9 | Cheat Sheet — คำสั่งอ้างอิงเร็ว | [09_cheat_sheet.md](09_cheat_sheet.md) |
| บทที่ 11 | Cost Optimization (agent เพิ่ม token cost) | [11_cost_optimization.md](11_cost_optimization.md) |

---

> **สรุป**: The Agency ช่วยให้ Claude Code ทำงานเป็น "ผู้เชี่ยวชาญเฉพาะด้าน" แทนที่จะเป็น AI ทั่วไป
> เลือก agent ที่ตรงกับงาน ใช้ NEXUS framework สำหรับงานที่ต้องประสานหลาย agents
> และอย่าลืม — agent เป็นเครื่องมือ ต้อง review output ทุกครั้งก่อนนำไปใช้จริง
> สำหรับงานง่ายๆ ไม่จำเป็นต้องใช้ agent — ถาม Claude ตรงๆ เร็วกว่าและถูกกว่า

---

← [บทที่ 11: การจัดการต้นทุนและประสิทธิภาพ](11_cost_optimization.md) | [README →](README.md)
