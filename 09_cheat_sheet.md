# สรุปรวมและ Cheat Sheet สำหรับ Claude Code

> เอกสารอ้างอิงฉบับย่อ (Quick Reference) รวบรวมคำสั่ง เวิร์กโฟลว์ และแนวทางปฏิบัติที่ใช้บ่อยในการทำงานร่วมกับ Claude Code

---

## 9.1 Workflow Summary: "Single Source of Truth" Spec Approach

### Spec-First Workflow

แนวทางการทำงานแบบ Specification-Driven โดยใช้เอกสาร Spec เป็นแหล่งอ้างอิงหลัก (Single Source of Truth):

1. **BA** สร้าง Specification ในรูปแบบ Markdown
2. **SA** ตรวจสอบสถาปัตยกรรม (Architecture) และความเป็นไปได้ (Feasibility)
3. **Dev** พัฒนาโค้ดด้วย Claude Code โดยอ้างอิงจาก Spec
4. **QA** ตรวจสอบตามเกณฑ์การยอมรับ (Acceptance Criteria)
5. **PM** ติดตามความคืบหน้า (Progress) และค่าใช้จ่าย (Cost)

### แผนภาพเวิร์กโฟลว์

```
BA ──→ SA ──→ Dev ──→ QA ──→ PM
│       │       │       │       │
Spec  Review  Code   Test   Track
       ↓       ↓       ↓
     CLAUDE.md  PR    Report
```

### บทบาทในแต่ละขั้นตอน

| ขั้นตอน | BA | SA | Dev | QA | PM |
|---------|----|----|-----|----|----|
| สร้าง Spec | เขียน Spec | Review | - | Review criteria | กำหนด scope |
| Review | แก้ไขตาม feedback | ตรวจ architecture | ประเมิน effort | ตรวจ testability | อนุมัติ |
| Implement | ตอบคำถาม | แก้ปัญหา technical | เขียนโค้ด + test | เตรียม test plan | ติดตาม |
| Testing | ยืนยัน behavior | - | แก้ bug | ทดสอบ + รายงาน | ติดตาม |
| Delivery | - | - | Deploy support | Sign-off | ปิดงาน |

---

## 9.2 Essential Commands — Project Initialization & Context

คำสั่งพื้นฐานสำหรับเริ่มต้นโปรเจกต์และจัดการบริบท (Context):

| คำสั่ง | คำอธิบาย | ตัวอย่าง |
|--------|----------|---------|
| `claude` | เปิด Interactive Session ใหม่ | `claude` |
| `claude -c` | ต่อ session ล่าสุดที่เปิดค้างไว้ | `claude -c` |
| `claude --resume` | เลือก session เก่าจากรายการ | `claude --resume` |
| `/init` | สร้างไฟล์ CLAUDE.md สำหรับโปรเจกต์ | `/init` |
| `/context` | แสดงบริบทปัจจุบันที่ Claude มองเห็น | `/context` |
| `/compact` | บีบอัดบริบทเพื่อประหยัด token | `/compact` |
| `/clear` | ล้างบริบททั้งหมดในเซสชัน | `/clear` |
| `/memory` | แก้ไข Memory File (CLAUDE.md) | `/memory` |
| `/cost` | แสดงค่าใช้จ่าย token ของเซสชันปัจจุบัน | `/cost` |
| `/model` | เปลี่ยนโมเดลระหว่างเซสชัน | `/model sonnet` |
| `Shift+Tab` | สลับเข้า Plan Mode (วางแผนก่อนทำ) | - |
| `Esc` | ยกเลิกคำสั่งหรือหยุดการทำงาน | - |

### เคล็ดลับ: ลำดับการเริ่มงานแนะนำ

- เปิดเซสชันใหม่ที่ root ของโปรเจกต์
- ตรวจสอบว่ามี CLAUDE.md อยู่แล้วหรือยัง ถ้ายังไม่มีให้รัน `/init`
- ใช้ `/compact` เมื่อบริบทเริ่มยาวเกินไป
- ใช้ `/cost` ตรวจสอบค่าใช้จ่ายเป็นระยะ

---

## 9.3 Essential Commands — Multi-module Development

คำสั่งสำหรับการทำงานกับหลายโมดูล (Multi-module) และการรันแบบไม่โต้ตอบ (Non-interactive):

| คำสั่ง | คำอธิบาย | ตัวอย่าง |
|--------|----------|---------|
| `--add-dir` | เพิ่ม directory อื่นเข้ามาในบริบท | `claude --add-dir /path/to/frontend` |
| `claude -w` | เปิดในโหมด Worktree (แยก branch ทำงาน) | `claude -w feature-auth` |
| `claude -p` | รันแบบ Non-interactive / Pipe mode | `claude -p "fix tests"` |
| `--max-turns` | จำกัดจำนวนรอบการทำงานอัตโนมัติ | `claude -p "task" --max-turns 5` |
| `--model` | ระบุโมเดลที่ต้องการใช้ | `claude --model opus` |
| `--output-format` | กำหนดรูปแบบผลลัพธ์ | `claude -p "task" --output-format json` |

### ตัวอย่าง: ทำงานกับ Monorepo

```bash
# เปิด Claude Code พร้อม backend + frontend
claude --add-dir /path/to/frontend

# รันงานอัตโนมัติแบบจำกัดรอบ
claude -p "สร้าง API endpoint สำหรับ user profile" --max-turns 10

# ใช้ Worktree สำหรับ feature ใหม่
claude -w feature-user-profile
```

---

## 9.4 Essential Commands — Database & Schema

Prompt ตัวอย่างสำหรับงานฐานข้อมูล (Database) และ Schema:

| Task | Prompt ตัวอย่าง |
|------|----------------|
| สร้าง Migration | `"สร้าง SQL migration สำหรับ [table] ที่มี [columns]"` |
| GORM Model | `"สร้าง GORM model สำหรับ [entity] ที่มี relation กับ [other entity]"` |
| Query Optimization | `"วิเคราะห์ queries ใน [file] หา N+1 problems"` |
| Index Recommendation | `"แนะนำ indexes สำหรับ queries ที่ใช้บ่อยใน [repository]"` |
| Schema Review | `"review database schema ตรวจสอบ normalization และ naming"` |

### ตัวอย่าง Prompt แบบละเอียด

```
สร้าง SQL migration สำหรับตาราง "orders" ที่มีคอลัมน์:
- id (UUID, primary key)
- user_id (UUID, foreign key → users)
- total_amount (decimal)
- status (enum: pending, confirmed, shipped, delivered)
- created_at, updated_at (timestamps)

รวมถึง indexes ที่จำเป็นและ GORM model ที่ตรงกัน
```

---

## 9.5 Essential Commands — Testing & Validation

Prompt ตัวอย่างสำหรับงานทดสอบ (Testing) และการตรวจสอบ (Validation):

| Task | Prompt ตัวอย่าง |
|------|----------------|
| Unit Test | `"สร้าง unit tests สำหรับ [file] ด้วย table-driven tests"` |
| Integration Test | `"สร้าง integration test สำหรับ [endpoint]"` |
| Run Tests | `"รัน go test ./... แล้วสรุปผล"` |
| Coverage | `"วิเคราะห์ test coverage แล้วแนะนำ test ที่ควรเพิ่ม"` |
| Code Review | `"review code ใน [file] ตาม checklist"` |

### แนวทาง Testing ที่ดี

- ใช้ Table-Driven Tests เสมอสำหรับ Go
- ครอบคลุมทั้ง Happy Path และ Edge Cases
- ตรวจสอบ Error Messages ว่ามีความหมาย
- ทดสอบ Concurrent Access ถ้าเกี่ยวข้อง
- ให้ Claude วิเคราะห์ Coverage แล้วแนะนำ Test ที่ขาด

---

## 9.6 Essential Commands — Security & Review

การตั้งค่าสิทธิ์ (Permissions) และความปลอดภัย (Security):

| Setting | คำอธิบาย |
|---------|----------|
| Permission Modes | Default, Approve Once, Approve Session, Allow All, Deny |
| `/sandbox` | เปิดโหมด Sandbox (จำกัดการเข้าถึงระบบ) |
| Allow Patterns | `"Read"`, `"Edit(**/*.go)"`, `"Bash(go test*)"` |
| Deny Patterns | `"Bash(rm -rf*)"`, `"Write(.env*)"`, `"Bash(git push --force*)"` |
| `settings.json` | `.claude/settings.json` — สิทธิ์ระดับโปรเจกต์ (แชร์กับทีม) |
| `settings.local.json` | `.claude/settings.local.json` — สิทธิ์ส่วนตัว (อยู่ใน .gitignore) |

### ตัวอย่างไฟล์ settings.json

```json
{
  "permissions": {
    "allow": [
      "Read",
      "Edit(**/*.go)",
      "Edit(**/*.ts)",
      "Bash(go test*)",
      "Bash(go build*)",
      "Bash(npm test*)",
      "Bash(git status)",
      "Bash(git diff*)",
      "Bash(git log*)"
    ],
    "deny": [
      "Bash(rm -rf*)",
      "Write(.env*)",
      "Bash(git push --force*)",
      "Bash(curl*)",
      "Bash(wget*)"
    ]
  }
}
```

### แนวทางความปลอดภัย

- ตั้ง Deny Patterns สำหรับคำสั่งอันตรายเสมอ
- ใช้ `settings.json` สำหรับกฎระดับทีม
- ใช้ `settings.local.json` สำหรับกฎส่วนตัว
- เปิด Sandbox Mode เมื่อทำงานกับโค้ดที่ไม่คุ้นเคย

---

## 9.7 Conflict Resolution

### สถานการณ์ที่ FE/BE Changes Conflict

ปัญหาที่พบบ่อยเมื่อ Frontend และ Backend พัฒนาพร้อมกัน:

1. **API Response Shape เปลี่ยน** — โครงสร้าง JSON ไม่ตรงกัน
2. **Field Naming Inconsistency** — ชื่อ field ไม่ตรงกัน (เช่น camelCase vs snake_case)
3. **Missing/Extra Fields** — field หายหรือเกินมา
4. **Type Mismatches** — ชนิดข้อมูลไม่ตรงกัน (เช่น string vs number)

### วิธีแก้: Spec เป็น Single Source of Truth

| ขั้นตอน | การกระทำ | ผู้รับผิดชอบ |
|---------|----------|-------------|
| 1. Detect | ค้นพบ conflict (test fail, type error) | Dev / QA |
| 2. Spec Review | กลับไปดู spec — spec ถูกต้องหรือไม่? | BA + SA |
| 3. Update Spec | ถ้า spec ต้องเปลี่ยน → อัพเดท spec | BA + SA |
| 4. Resolve | Dev implement ตาม spec ที่อัพเดท | Dev |
| 5. Verify | ทั้ง FE และ BE ตรงกับ spec | QA |

### ตัวอย่าง: API Response Shape เปลี่ยน

```
Scenario: BE เปลี่ยน field "name" → "full_name"

1. อัพเดท spec: docs/specs/user.md
2. BE: แก้ Go struct, response mapping
3. FE: อัพเดท TypeScript interface, components
4. Test: ทั้ง BE และ FE tests pass

Claude Prompt:
"API field 'name' เปลี่ยนเป็น 'full_name' ตาม spec ใหม่
อัพเดททุกที่ที่ใช้ field นี้: types, components, tests"
```

### Type Synchronization Workflow

```
Go Struct ──→ Spec ──→ TypeScript Interface
   ↓                        ↓
 Handler                 Component
   ↓                        ↓
   API ─────────────────→ Client
```

### แนวทางป้องกัน Conflict

- กำหนด API Contract ใน spec ก่อนเริ่มพัฒนา
- ใช้ชื่อ field ตาม convention ที่ตกลงกัน (เช่น snake_case สำหรับ JSON)
- เขียน Integration Test ที่ตรวจสอบ response shape
- Review spec เมื่อมีการเปลี่ยนแปลง schema

---

## 9.8 Quick Reference: Model Selection

เลือกโมเดลให้เหมาะกับลักษณะงาน:

| Model | ใช้เมื่อ | ค่าใช้จ่าย | ตัวอย่างงาน |
|-------|---------|-----------|------------|
| Sonnet | งานทั่วไป, code generation, testing | ปานกลาง | สร้าง CRUD, เขียน test, refactor |
| Opus | งานซับซ้อน, architecture, difficult bugs | สูง | ออกแบบ system, debug ปัญหาซับซ้อน |
| Haiku | งานเล็กๆ, quick questions, formatting | ต่ำ | ถามคำถาม, จัด format, สร้าง comment |

### แนวทางการเลือกโมเดล

- **เริ่มต้นด้วย Sonnet** สำหรับงานส่วนใหญ่
- **สลับไป Opus** เมื่อ Sonnet ไม่สามารถแก้ปัญหาได้ หรือต้องการคุณภาพสูงสุด
- **ใช้ Haiku** สำหรับงานที่ไม่ซับซ้อนเพื่อประหยัดค่าใช้จ่าย
- สลับโมเดลระหว่างเซสชันด้วย `/model [model-name]`

---

## 9.9 Quick Reference: Keyboard Shortcuts

| คีย์ | การทำงาน |
|------|----------|
| `Esc` | ยกเลิกคำสั่ง / หยุดการทำงานปัจจุบัน |
| `Esc` + `Esc` | ยกเลิกและย้อนกลับไป prompt ก่อนหน้า |
| `Shift+Tab` | สลับเข้า/ออก Plan Mode |
| `Ctrl+B` | บุ๊กมาร์กตำแหน่งปัจจุบัน |
| `Ctrl+C` | ยกเลิกคำสั่งที่กำลังพิมพ์ |
| `Up/Down` | เลื่อนดูประวัติคำสั่ง |

---

## 9.10 Quick Reference: Prompt Patterns ที่ใช้บ่อย

### Pattern 1: สร้างโค้ดจาก Spec

```
อ่าน spec ใน docs/specs/[feature].md
แล้วสร้าง [component] ตามที่ระบุ
รวมถึง unit tests และ error handling
```

### Pattern 2: Review และปรับปรุง

```
review code ใน [path]
ตรวจสอบ: error handling, naming, performance
แนะนำการปรับปรุงพร้อมเหตุผล
```

### Pattern 3: Debug

```
test [command] ล้มเหลวด้วย error: [error message]
วิเคราะห์สาเหตุและแก้ไข
```

### Pattern 4: Multi-file Change

```
เปลี่ยน [สิ่งที่ต้องเปลี่ยน] ตาม spec ใหม่
อัพเดททุกไฟล์ที่เกี่ยวข้อง:
- types/interfaces
- handlers/controllers
- tests
- documentation
```

---

## 9.11 Quick Reference: Cost Management

### การประหยัดค่าใช้จ่าย

| วิธีการ | คำอธิบาย |
|---------|----------|
| ใช้ `/compact` บ่อยๆ | ลดจำนวน token ในบริบท |
| เลือก model ให้เหมาะ | ใช้ Haiku สำหรับงานเล็ก, Sonnet สำหรับงานทั่วไป |
| เขียน prompt ให้ชัดเจน | ลดการถาม-ตอบหลายรอบ |
| ใช้ `--max-turns` | จำกัดรอบสำหรับงาน automation |
| ใช้ `/cost` ตรวจสอบ | ติดตามค่าใช้จ่ายเป็นระยะ |

### การประมาณค่าใช้จ่าย

| ขนาดงาน | Sonnet (โดยประมาณ) | Opus (โดยประมาณ) |
|---------|-------------------|-----------------|
| งานเล็ก (prompt เดียว) | ~$0.01-0.05 | ~$0.05-0.20 |
| งานกลาง (หลาย turns) | ~$0.10-0.50 | ~$0.50-2.00 |
| งานใหญ่ (full feature) | ~$0.50-2.00 | ~$2.00-10.00 |

> หมายเหตุ: ค่าใช้จ่ายจริงขึ้นอยู่กับปริมาณ token ที่ใช้ ตรวจสอบด้วย `/cost` เสมอ

---

## 9.12 Checklist ก่อนเริ่มงาน

- [ ] มีไฟล์ CLAUDE.md ในโปรเจกต์แล้ว (ถ้ายังไม่มีให้รัน `/init`)
- [ ] ตั้งค่า `.claude/settings.json` สำหรับ permissions ระดับทีม
- [ ] มี Spec พร้อมสำหรับ feature ที่จะพัฒนา
- [ ] ตกลง API Contract กับทีม FE/BE แล้ว
- [ ] กำหนด branch naming convention
- [ ] ตั้ง Deny Patterns สำหรับคำสั่งอันตราย

---

## 9.13 Troubleshooting Quick Fixes

10 ปัญหาที่พบบ่อยพร้อมวิธีแก้แบบสั้น:

| # | ปัญหา | วิธีแก้ |
|---|--------|---------|
| 1 | Claude สร้าง code ไม่ตรง convention | อัพเดท CLAUDE.md → ใส่ convention ที่ชัดเจน → ให้ Claude อ่าน CLAUDE.md ก่อนทำงาน |
| 2 | Context window เต็ม / ตอบช้า | `/compact` สรุป context → ถ้ายังไม่พอให้เริ่ม session ใหม่ |
| 3 | Claude แก้ไฟล์ผิด | `git diff` ตรวจสอบ → `git restore <file>` กู้คืน → ระบุไฟล์ให้ชัดในครั้งต่อไป |
| 4 | Permission ถูก block | ตรวจ `.claude/settings.json` → เพิ่ม allow pattern ที่ต้องการ |
| 5 | Claude hallucinate / ตอบไม่ตรง | เริ่ม session ใหม่ → ให้ context ที่ชัดเจนกว่าเดิม |
| 6 | Test ที่ generate ไม่ครอบคลุม | ระบุ edge cases ที่ต้องการ: `"ต้องครอบคลุม: nil input, empty list, boundary values"` |
| 7 | Code compile ไม่ผ่าน | ให้ Claude รัน `go build ./...` หรือ `npm run build` เองแล้วแก้ |
| 8 | npm install / go mod ล้มเหลว | ตรวจ network + API key → ลอง `npm cache clean --force` หรือ `go clean -modcache` |
| 9 | Claude ทำงานช้ามาก | สลับ model เป็น Sonnet/Haiku → ลด context ด้วย `/compact` |
| 10 | DEVLOG/CLAUDE.md conflict | กำหนดเจ้าของแต่ละ section → review ก่อน merge → ใช้ git merge strategy |

📌 รายละเอียดเพิ่มเติมที่ [บทที่ 10: Troubleshooting & FAQ](./10_troubleshooting_faq.md)

---

## 9.14 Session Management Cheat Sheet

เมื่อไรควรทำอะไร:

| สถานการณ์ | Action | คำสั่ง |
|-----------|--------|--------|
| Session ยาว, Claude เริ่มลืม context | Compact | `/compact` |
| จบ task ใหญ่, จะเริ่ม task ใหม่ | Clear | `/clear` |
| ต้องการ context จาก session เก่า | Continue | `claude -c` |
| เลือก session เฉพาะจากรายการ | Resume | `claude --resume` |
| เปลี่ยน task ที่ไม่เกี่ยวข้องกันเลย | Session ใหม่ | `claude` (เปิดใหม่) |
| ทำงานหลาย feature พร้อมกัน | Worktree | `claude -w <branch>` |
| รัน task อัตโนมัติ | Pipe mode | `claude -p "task" --max-turns 5` |

### Decision Tree: จัดการ Session

```
Claude ตอบช้า / ลืม context?
├─ ยังอยู่ task เดิม → /compact แล้วทำต่อ
└─ เปลี่ยน task แล้ว
   ├─ ต้องการ context เดิม → /clear แล้วให้ context ใหม่
   └─ ไม่ต้องการ context → เปิด session ใหม่

จะเริ่มงานวันใหม่?
├─ ต่อจากเมื่อวาน → claude -c หรือ claude --resume
└─ งานใหม่ → claude (session ใหม่) + อ่าน DEVLOG ก่อน
```

---

## 9.15 Cost-Saving Tips

5 เทคนิคประหยัด token ที่ใช้ได้ทันที:

### 1. ใช้ CLAUDE.md แทนการอธิบายซ้ำ
```
# แทนที่จะพิมพ์ทุกครั้งว่า "ใช้ camelCase สำหรับ JSON, snake_case สำหรับ DB"
# เขียนใน CLAUDE.md ครั้งเดียว → Claude จำได้ทุก session
```

### 2. ชี้ไฟล์เฉพาะ ไม่ต้องให้อ่านทั้งโปรเจกต์
```
# ❌ แพง: "อ่านทุกไฟล์แล้วสร้าง API endpoint"
# ✅ ประหยัด: "ดูตัวอย่างใน internal/handler/product_handler.go แล้วสร้าง endpoint ใหม่ตาม pattern เดียวกัน"
```

### 3. /compact เมื่อ session ยาวเกิน 30 turns
```
/compact สรุปสิ่งที่ทำไปแล้ว เก็บ: ไฟล์ที่แก้, decisions, งานที่เหลือ
```

### 4. ใช้ Haiku สำหรับงาน routine
```
/model haiku
# ใช้สำหรับ: commit messages, format code, simple questions, documentation
/model sonnet
# สลับกลับเมื่อต้องทำงานจริง
```

### 5. Batch งานที่เกี่ยวข้องกัน
```
# ❌ แพง: เปิด 5 sessions สำหรับ 5 endpoints ของ feature เดียวกัน
# ✅ ประหยัด: ทำทั้ง 5 endpoints ใน session เดียว (Claude มี context อยู่แล้ว)
```

📌 รายละเอียดเพิ่มเติมที่ [บทที่ 11: Cost Optimization](./11_cost_optimization.md)

---

## การอ้างอิงข้ามบท (Cross-references)

| บท | หัวข้อ | ลิงก์ |
|----|--------|------|
| บทที่ 1 | บทนำและพื้นฐาน Claude Code | [01_intro.md](./01_intro.md) |
| บทที่ 2 | Agentic Mastery — เทคนิคการใช้งานแบบ Agent | [02_agentic_mastery.md](./02_agentic_mastery.md) |
| บทที่ 3 | การใช้งานขั้นสูง | [03_advanced_usage.md](./03_advanced_usage.md) |
| บทที่ 4 | Full-Stack Collaboration — การทำงานร่วม FE/BE | [04_fullstack_collaboration.md](./04_fullstack_collaboration.md) |
| บทที่ 5 | Specification-Driven Development | [05_spec_driven_dev.md](./05_spec_driven_dev.md) |
| บทที่ 6 | Architecture & Design — สถาปัตยกรรมและการออกแบบ | [06_architecture_design.md](./06_architecture_design.md) |
| บทที่ 7 | QA, Security, and Review — การทดสอบและความปลอดภัย | [07_qa_security_review.md](./07_qa_security_review.md) |
| บทที่ 8 | บทบาทและความรับผิดชอบของแต่ละตำแหน่ง | [08_roles_responsibilities.md](./08_roles_responsibilities.md) |
| บทที่ 10 | Troubleshooting & FAQ | [10_troubleshooting_faq.md](./10_troubleshooting_faq.md) |
| บทที่ 11 | การจัดการต้นทุนและประสิทธิภาพ | [11_cost_optimization.md](./11_cost_optimization.md) |

---

> **สรุป**: เอกสารฉบับนี้รวบรวมคำสั่ง เวิร์กโฟลว์ และแนวทางปฏิบัติที่สำคัญทั้งหมดไว้ในที่เดียว
> ใช้เป็น Quick Reference ระหว่างทำงานจริงกับ Claude Code
> สำหรับรายละเอียดเชิงลึกของแต่ละหัวข้อ ให้อ้างอิงจากบทที่เกี่ยวข้องข้างต้น
