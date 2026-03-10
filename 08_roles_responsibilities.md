# บทบาทและความรับผิดชอบ

## Roles & Responsibilities

คู่มือการใช้งาน Claude Code สำหรับแต่ละบทบาทในทีมพัฒนา ครอบคลุมหน้าที่หลัก, prompt templates,
permission settings ที่แนะนำ, แนวทางการทำงานร่วมกันในทีม, และ onboarding checklist สำหรับสมาชิกใหม่

---

## 8.1 Developer (นักพัฒนา — FE/BE)

### หน้าที่หลัก

- **Implementation ตาม spec** — รับ specification แล้วเขียนโค้ดตามที่กำหนด
- **Bug fixing และ debugging** — วิเคราะห์ปัญหาและแก้ไขโดยใช้ Claude ช่วยหาสาเหตุ
- **Unit/Integration testing** — เขียนและรัน test เพื่อยืนยันความถูกต้อง
- **Code review** — review โค้ดของคนอื่น และ self-review ด้วย Claude ก่อนส่ง PR

### Prompt Templates สำหรับ Developer

```
# Implementation
"implement [feature] ตาม spec ใน docs/specs/[name].md
ใช้ three-layer architecture: handler → service → repository"

# Bug Fix
"วิเคราะห์ error นี้: [error message]
อยู่ใน [file path]
1. หาสาเหตุ
2. เสนอวิธีแก้ไข
3. ตรวจสอบว่าจะกระทบส่วนอื่นหรือไม่"

# Testing
"สร้าง unit tests สำหรับ [file]:
- Table-driven tests
- Mock dependencies
- Edge cases"

# Debugging
"รัน [command] แล้ววิเคราะห์ output
เน้นหาสาเหตุของ [specific issue]"
```

### Permission Settings ที่แนะนำ

```json
{
  "permissions": {
    "allow": [
      "Read", "Glob", "Grep",
      "Edit(**/*.go)", "Edit(**/*.ts)", "Edit(**/*.tsx)",
      "Write(internal/**)", "Write(web/src/**)",
      "Bash(go test*)", "Bash(go build*)",
      "Bash(npm test*)", "Bash(npm run dev*)"
    ],
    "deny": [
      "Bash(rm -rf*)",
      "Bash(git push --force*)",
      "Write(.env*)"
    ]
  }
}
```

---

## 8.2 SA (System Analyst)

### หน้าที่หลัก

- **Architecture analysis และ design consistency** — ตรวจสอบว่าโค้ดเป็นไปตามสถาปัตยกรรม
- **CLAUDE.md maintenance** — เป็นเจ้าของ CLAUDE.md ดูแลให้ถูกต้องและเป็นปัจจุบัน
- **Architecture Decision Records (ADR)** — บันทึกเหตุผลการตัดสินใจด้านสถาปัตยกรรม
- **Technical review and guidance** — ให้คำปรึกษาด้านเทคนิค

### Prompt Templates สำหรับ SA

```
# Architecture Analysis
"วิเคราะห์สถาปัตยกรรมของโปรเจกต์:
1. Package dependencies
2. Layer violations
3. Design pattern consistency
4. Separation of concerns"

# CLAUDE.md Review
"อ่าน CLAUDE.md แล้ว:
1. ข้อมูลยังถูกต้องหรือไม่
2. มีอะไรที่ขาดหายไป
3. มีอะไรที่ล้าสมัย
4. แนะนำการปรับปรุง"

# ADR Creation
"สร้าง ADR สำหรับ [decision]:
Context: [why this decision is needed]
Alternatives: [option A, B, C]
ใช้ format: Title, Status, Context, Decision, Consequences"

# Design Review
"review การออกแบบใน [file/module]:
1. ตรง conventions ที่กำหนดหรือไม่
2. มี anti-patterns หรือไม่
3. scalability concerns"
```

### Permission Settings ที่แนะนำ

```json
{
  "permissions": {
    "allow": [
      "Read", "Glob", "Grep",
      "Edit(CLAUDE.md)", "Edit(docs/**)",
      "Write(docs/adr/**)",
      "Bash(go vet*)", "Bash(staticcheck*)"
    ],
    "deny": [
      "Bash(rm -rf*)",
      "Bash(git push --force*)",
      "Write(.env*)"
    ]
  }
}
```

---

## 8.3 BA (Business Analyst)

### หน้าที่หลัก

- **แปลง business requirements เป็น Markdown specifications** — เขียน spec ที่ชัดเจน
- **สร้าง feature specs** — กำหนดรายละเอียด feature
- **ดูแลว่า spec ครบถ้วน** — ตรวจสอบว่าไม่มี requirement ตกหล่น

### Spec Template สำหรับ BA

```markdown
# Feature: [Feature Name]

## Objective
[ระบุวัตถุประสงค์ของ feature — ทำไมถึงต้องมี feature นี้]

## User Stories

### US-001: [Story Title]
- **As a** [role]
- **I want** [action]
- **So that** [benefit]

## Business Rules

| Rule ID | Description     | Condition           | Action          |
|---------|-----------------|---------------------|-----------------|
| BR-001  | [description]   | [when/if condition] | [then action]   |

## API Contract (High-Level)

### [Method] /api/v1/[resource]
- **Request:** [summary of request body/params]
- **Response:** [summary of response structure]
- **Error Cases:** [list of possible errors]

## Acceptance Criteria

- [ ] AC-001: [criteria description]
- [ ] AC-002: [criteria description]

## Edge Cases

| Case ID | Scenario     | Expected Behavior       |
|---------|--------------|-------------------------|
| EC-001  | [scenario]   | [expected behavior]     |

## Dependencies

- [dependency 1 — e.g., ต้องมี user authentication ก่อน]
- [dependency 2 — e.g., ต้องเชื่อมต่อกับ payment gateway]
```

### Prompt Templates สำหรับ BA

```
# Spec Creation
"ช่วยสร้าง specification สำหรับ feature [name]:
Business requirement: [description]
ใช้ format ตาม template ของทีม"

# Requirement Validation
"ตรวจสอบ spec ใน docs/specs/[name].md:
1. ครบถ้วนหรือไม่
2. มี ambiguity ตรงไหน
3. edge cases ที่ยังไม่ได้ระบุ
4. acceptance criteria ชัดเจนหรือไม่"

# User Story Generation
"จาก business requirement นี้: [description]
สร้าง user stories ในรูปแบบ:
As a [role], I want [action] so that [benefit]"
```

### Permission Settings ที่แนะนำ

```json
{
  "permissions": {
    "allow": [
      "Read", "Glob", "Grep",
      "Edit(docs/specs/**)", "Write(docs/specs/**)"
    ],
    "deny": [
      "Bash(rm -rf*)", "Write(.env*)",
      "Edit(internal/**)", "Edit(web/src/**)"
    ]
  }
}
```

---

## 8.4 PM (Project Manager)

### หน้าที่หลัก

- **ติดตามความคืบหน้า** — ตรวจสอบสถานะ feature ที่กำลังพัฒนา
- **Technical debt monitoring** — ติดตามหนี้ทางเทคนิคในโค้ด
- **Cost monitoring** — ดูแลค่าใช้จ่ายด้วยคำสั่ง `/cost`
- **Team coordination** — ประสานงานระหว่างสมาชิกในทีม

### Prompt Templates สำหรับ PM

```
# Progress Check
"สรุปสถานะของ feature [name]:
1. files ที่ถูกแก้ไข
2. tests ที่ผ่าน/ไม่ผ่าน
3. TODO items ที่เหลือ"

# Technical Debt Report
"วิเคราะห์ technical debt ในโปรเจกต์:
1. TODO/FIXME/HACK comments
2. Code complexity metrics
3. Outdated dependencies
4. Missing tests"

# Cost Monitoring
"/cost — ตรวจสอบค่าใช้จ่ายโทเค็น"
```

### Permission Settings ที่แนะนำ

PM ใช้ Claude Code ในโหมดอ่านอย่างเดียว (Read-Only):

```json
{
  "permissions": {
    "allow": [
      "Read", "Glob", "Grep",
      "Bash(git log*)", "Bash(git diff*)", "Bash(git status*)"
    ],
    "deny": [
      "Edit(**)", "Write(**)",
      "Bash(rm*)", "Bash(git push*)"
    ]
  }
}
```

---

## 8.5 DevOps

### หน้าที่หลัก

- **CI/CD workflows** — สร้างและดูแล GitHub Actions หรือ pipeline อื่นๆ
- **Docker configuration** — จัดการ Dockerfile และ docker-compose
- **Deployment automation** — สร้าง script สำหรับ deploy อัตโนมัติ

### Prompt Templates สำหรับ DevOps

```
# CI/CD
"สร้าง GitHub Actions workflow สำหรับ:
1. Build and test on PR
2. Deploy to staging on merge to develop
3. Deploy to production on release tag"

# Docker
"สร้าง Dockerfile + docker-compose.yml สำหรับ development environment:
- Go API
- PostgreSQL
- Redis
- Frontend (React)"

# Infrastructure Review
"review Dockerfile สำหรับ production:
1. Multi-stage build ถูกต้องหรือไม่
2. Security best practices
3. Image size optimization
4. Non-root user"
```

### Permission Settings ที่แนะนำ

```json
{
  "permissions": {
    "allow": [
      "Read", "Glob", "Grep",
      "Edit(.github/**)", "Edit(Dockerfile*)",
      "Edit(docker-compose*)", "Edit(Makefile)",
      "Write(.github/workflows/**)",
      "Bash(docker*)", "Bash(make*)"
    ],
    "deny": [
      "Bash(rm -rf*)",
      "Bash(git push --force*)",
      "Write(.env*)",
      "Bash(kubectl delete*)"
    ]
  }
}
```

---

## 8.6 QA

### หน้าที่หลัก

- **Test generation and execution** — สร้าง test cases ที่ครอบคลุม
- **Coverage analysis** — วิเคราะห์ test coverage
- **Regression testing** — ตรวจสอบว่า changes ใหม่ไม่ break functionality เดิม

### Prompt Templates สำหรับ QA

```
# Test Generation
"สร้าง comprehensive test suite สำหรับ [module]:
1. Unit tests
2. Integration tests
3. Edge case tests"

# Regression Testing
"ตรวจสอบว่า changes ใน PR นี้:
1. ไม่ break existing tests
2. มี test coverage สำหรับ new code
3. Edge cases ถูก handle"

# Coverage Analysis
"วิเคราะห์ test coverage ของ [module]:
1. รัน coverage report
2. ระบุส่วนที่ยังไม่มี test
3. แนะนำ test cases ที่ควรเพิ่ม"
```

### Permission Settings ที่แนะนำ

```json
{
  "permissions": {
    "allow": [
      "Read", "Glob", "Grep",
      "Edit(**/*_test.go)", "Edit(**/*.test.ts)", "Edit(**/*.spec.ts)",
      "Write(**/*_test.go)", "Write(**/*.test.ts)",
      "Bash(go test*)", "Bash(npm test*)", "Bash(npm run test*)"
    ],
    "deny": [
      "Bash(rm -rf*)",
      "Bash(git push --force*)",
      "Write(.env*)",
      "Edit(internal/service/**)",
      "Edit(internal/handler/**)"
    ]
  }
}
```

---

## 8.7 แนวทางจัดการ Claude Code ในทีม

### Shared Settings

- **`.claude/settings.json`** — commit ใน git เพื่อให้ทุกคนใช้ rules เดียวกัน
- **`.claude/settings.local.json`** — settings ส่วนตัว (อยู่ใน `.gitignore`)
- **`CLAUDE.md`** — SA เป็นผู้ดูแลหลัก ทุกคนเสนอแก้ไขผ่าน PR ได้

### Cost Monitoring

- ใช้ `/cost` เป็นระยะ — ตรวจสอบทุกสัปดาห์ หรือหลัง session ยาว
- ตั้ง budget per developer — กำหนดวงเงินต่อ developer ต่อเดือน
- เลือก model ให้เหมาะสม — Sonnet สำหรับงานทั่วไป, Opus สำหรับงานซับซ้อน, Haiku สำหรับงานเล็กๆ
- ใช้ `/compact` สม่ำเสมอ — ลดขนาด context window เพื่อประหยัดโทเค็น (Token)
- ใช้ `/model` สลับ model ระหว่างทำงานตามความเหมาะสม

### Knowledge Sharing

- **แชร์ effective prompts** — prompt ที่ให้ผลลัพธ์ดีควรแชร์ให้ทีม
- **บันทึก prompt patterns** — รวบรวมไว้ใน wiki หรือ shared doc
- **Review CLAUDE.md ร่วมกัน** — ทุก sprint
- **Retrospective** — รวม Claude Code เข้าใน sprint retrospective

### Workflow แนะนำสำหรับทีม

```
BA สร้าง spec → SA review + ออกแบบ architecture → SA อัปเดต CLAUDE.md
→ Developer implement ตาม spec → Developer self-review → QA สร้าง tests
→ PM ติดตามความคืบหน้า → DevOps deploy ด้วย Claude-assisted CI/CD
```

---

## 8.8 Onboarding Checklist สำหรับ Developer ใหม่

```markdown
## Onboarding Checklist — Claude Code

### Day 1: ติดตั้งและทำความรู้จัก
- [ ] ติดตั้ง Claude Code: `curl -fsSL https://claude.ai/install.sh | bash`
- [ ] Login: `claude` → Login
- [ ] อ่าน CLAUDE.md ของโปรเจกต์
- [ ] ลอง Interactive Mode: `claude`
- [ ] ลอง Slash Commands: /help, /model, /cost
- [ ] อ่านบทที่ 1-2 ของคู่มือนี้

### Day 2: ฝึกใช้งาน
- [ ] ลอง Plan Mode (Shift+Tab)
- [ ] ลอง code generation จาก spec
- [ ] ลอง code review ด้วย Claude
- [ ] ลอง debugging workflow
- [ ] อ่านบทที่ 3-4

### Day 3: Advanced
- [ ] ตั้งค่า .claude/settings.local.json
- [ ] ลอง --add-dir สำหรับ cross-project
- [ ] ลอง /memory
- [ ] อ่านบทที่ 5-8

### Week 1: Integration
- [ ] ใช้ Claude Code ใน daily work
- [ ] แชร์ feedback กับทีม
- [ ] Contribute prompt patterns
```

---

## 8.9 ข้อควรระวังและข้อห้าม

### ข้อห้าม

> **Warning:** ข้อห้ามเหล่านี้มีความสำคัญมาก การฝ่าฝืนอาจนำไปสู่ปัญหาร้ายแรง

| ข้อห้าม                                        | เหตุผล                                               |
|------------------------------------------------|------------------------------------------------------|
| ห้าม approve โดยไม่อ่าน code                    | Claude อาจสร้างโค้ดที่ไม่ถูกต้อง หรือมี security flaw |
| ห้ามให้ Claude push code โดยไม่ review           | ต้องตรวจสอบก่อน push เสมอ                             |
| ห้ามใส่ secrets ใน CLAUDE.md                     | CLAUDE.md commit ใน git — ทุกคนเห็น                  |
| ห้ามใช้ Claude กับ production database โดยตรง    | อาจเกิด data loss หรือ corruption                     |
| ห้ามพึ่ง Claude 100% — ต้อง review เสมอ         | Claude เป็นเครื่องมือช่วย ไม่ใช่ทดแทนมนุษย์           |

### ข้อควรระวัง

| ข้อควรระวัง                                     | แนวทางป้องกัน                                         |
|------------------------------------------------|------------------------------------------------------|
| Claude อาจ hallucinate (สร้างข้อมูลเท็จ)        | ตรวจสอบความถูกต้องของข้อมูลเสมอ                       |
| Context window (หน้าต่างบริบท) มีจำกัด           | ใช้ `/compact` เมื่อ session ยาว                      |
| Cost สะสมได้เร็ว                                | ตรวจสอบด้วย `/cost` เป็นประจำ                         |
| Claude ไม่รู้ข้อมูลหลัง knowledge cutoff         | ตรวจสอบข้อมูลที่เกี่ยวกับ library versions             |
| Bash commands ที่ destructive                    | ตั้ง deny list ใน permissions                          |

### แนวปฏิบัติที่ดี

1. **Review ก่อน approve เสมอ** — อ่านทุก diff ที่ Claude สร้าง
2. **ใช้ Plan Mode ก่อน implement** — วางแผนก่อนลงมือทำ
3. **ตั้ง permissions ให้เหมาะสม** — จำกัดสิทธิ์ตาม role
4. **ใช้ `/compact` สม่ำเสมอ** — เพื่อรักษาคุณภาพของ context
5. **แชร์ความรู้ในทีม** — prompt patterns ที่ได้ผลดีควรแชร์ให้ทุกคน
6. **อัปเดต CLAUDE.md** — ทุกครั้งที่มีการเปลี่ยนแปลง conventions
7. **ตรวจสอบ cost** — monitor ค่าใช้จ่ายเป็นระยะ

---

## Cross-references

- [บทที่ 1: บทนำและพื้นฐาน](./01_intro.md)
- [บทที่ 2: Agentic Mastery](./02_agentic_mastery.md)
- [บทที่ 3: การใช้งานขั้นสูง](./03_advanced_usage.md)
- [บทที่ 4: Full-Stack Collaboration](./04_fullstack_collaboration.md)
- [บทที่ 5: Specification-Driven Development](./05_spec_driven_dev.md)
- [บทที่ 6: Architecture & Design](./06_architecture_design.md)
- [บทที่ 7: QA, Security, and Review](./07_qa_security_review.md)
- [บทที่ 9: Cheat Sheet](./09_cheat_sheet.md)
