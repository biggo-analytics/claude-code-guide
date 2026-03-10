# บทที่ 2: การเป็นผู้เชี่ยวชาญ Agentic Coding และแนวปฏิบัติที่ดี

## Agentic Mastery & Best Practices

> เอกสารภายในสำหรับทีมพัฒนา -- ปรับปรุงล่าสุด: มีนาคม 2026

---

## สารบัญ

- [2.1 หลักการ Agentic Coding](#21-หลักการ-agentic-coding)
- [2.2 เทคนิคการเขียนพรอมต์](#22-เทคนิคการเขียนพรอมต์-prompt-engineering)
- [2.3 CLAUDE.md -- สมองของโปรเจกต์](#23-claudemd----สมองของโปรเจกต์)
- [2.4 .claude/ Directory Structure](#24-claude-directory-structure)
- [2.5 Memory System](#25-memory-system)
- [2.6 Context Management](#26-context-management)
- [2.7 Best Practices รวม](#27-best-practices-รวม)

---

## 2.1 หลักการ Agentic Coding

### Think -> Plan -> Execute -> Verify

Agentic Coding = ให้เอเจนต์ (Agent) ของ AI ทำงานเป็นขั้นตอนตามวงจร **TPEV Cycle**:

1. **Think** -- วิเคราะห์คำสั่ง ทำความเข้าใจบริบท อ่านไฟล์ที่เกี่ยวข้อง
2. **Plan** -- กำหนดลำดับขั้นตอน เลือกเครื่องมือ ประเมินผลกระทบ
3. **Execute** -- สร้าง/แก้ไขโค้ด รันคำสั่ง ทำตามแผน
4. **Verify** -- รัน test ตรวจสอบผลลัพธ์ ยืนยันว่าตรงเป้าหมาย

```
┌──────────┐     ┌──────────┐     ┌──────────┐     ┌──────────┐
│  Think   │────>│   Plan   │────>│ Execute  │────>│  Verify  │
└──────────┘     └──────────┘     └──────────┘     └────┬─────┘
      ^                                                  │
      └──────────────── feedback loop ───────────────────┘
```

### Claude Code ไม่ใช่แค่ Autocomplete

Claude Code เป็นเอเจนต์ที่อ่านโค้ด เข้าใจบริบท วางแผน ลงมือทำ และตรวจสอบผลลัพธ์ด้วยตัวเอง:

| ด้าน | Autocomplete | Agentic (Claude Code) |
|------|-------------|----------------------|
| ขอบเขต | เติมโค้ดทีละบรรทัด | ทำงานข้ามไฟล์ ข้ามขั้นตอน |
| บริบท | ไฟล์ปัจจุบัน | ทั้งโปรเจกต์ + คำสั่งระบบ |
| เครื่องมือ | ไม่ใช้ | อ่านไฟล์, รันคำสั่ง, ค้นหา, แก้ไข |
| การตรวจสอบ | ไม่ตรวจ | รัน test, ดู output, แก้ไขถ้าผิด |

### Mental Model

คิดเหมือนทำงานกับ junior developer ที่เก่งมาก -- ต้องให้ context ชัด, task ชัด, ตรวจงานก่อน approve

**ข้อดี:** เร็วขึ้น, ลด boilerplate, ลด context switching, เพิ่ม consistency

**ข้อควรระวัง:** ต้อง review ผลลัพธ์เสมอ, ไม่ approve โดยไม่อ่าน, ระวัง hallucination

---

## 2.2 เทคนิคการเขียนพรอมต์ (Prompt Engineering)

### โครงสร้าง CTCO

พรอมต์ (Prompt) ที่มีประสิทธิภาพมี 4 องค์ประกอบ:

- **C**ontext: บริบทโปรเจกต์, ไฟล์ที่เกี่ยวข้อง
- **T**ask: สิ่งที่ต้องทำ ชัดเจน
- **C**onstraints: ข้อจำกัด, conventions, rules
- **O**utput Format: รูปแบบผลลัพธ์ที่ต้องการ

### ตัวอย่างที่ 1: Simple Code Generation

**BAD:** `สร้าง API ให้หน่อย`

**GOOD:**
```
สร้าง REST API endpoint สำหรับ CRUD ของ entity "Device"
Context: Go + Fiber + GORM, ดู pattern จาก internal/domain/user/
Task: สร้าง handler, service, repository
      Endpoints: GET /api/v1/devices, GET /:id, POST, PUT, DELETE
Constraints: ตาม Clean Architecture, ใส่ input validation
Output: สร้างไฟล์ตาม folder structure เดิม + unit test สำหรับ service
```

### ตัวอย่างที่ 2: Bug Fixing with Context

**BAD:** `fix the bug`

**GOOD:**
```
แก้ bug ใน internal/service/user_service.go function GetUserByID
Actual: user ไม่เจอ return nil, nil -> handler return 200
Expected: return nil, ErrNotFound -> handler return 404
Constraints: ไม่เปลี่ยน function signature, เพิ่ม test case
```

### ตัวอย่างที่ 3: Refactoring with Constraints

**BAD:** `refactor โค้ด`

**GOOD:**
```
refactor internal/service/order_service.go
Context: function ProcessOrder ยาว 200 บรรทัด
Task: แยกเป็น function ย่อยที่รับผิดชอบเรื่องเดียว
Constraints: ไม่เปลี่ยน public interface, ไม่เปลี่ยน behavior
Output: function ย่อยไม่เกิน 30 บรรทัด + unit test ใหม่
```

### ตัวอย่างที่ 4: Multi-file Changes

**BAD:** `เพิ่ม middleware`

**GOOD:**
```
เพิ่ม rate limiting middleware สำหรับ public API endpoints
Context: Fiber framework, ดูตัวอย่างใน internal/middleware/
         Redis connection อยู่ใน internal/infrastructure/redis.go
Task: สร้าง middleware + เพิ่มใน router + เพิ่ม config
Constraints: 100 req/min per IP, ใช้ Redis, return 429 + Retry-After
```

### ตัวอย่างที่ 5: Code Review Prompt

**BAD:** `review โค้ดให้หน่อย`

**GOOD:**
```
Review internal/handler/payment_handler.go เน้น:
1. Security vulnerabilities  2. Error handling ที่ขาด
3. Input validation          4. Performance issues
5. Code conventions ที่ไม่ตรง
Output: รายการ findings + severity (Critical/High/Medium/Low) + วิธีแก้
```

### ตัวอย่างที่ 6: Architecture Analysis

**BAD:** `วิเคราะห์โปรเจกต์ให้หน่อย`

**GOOD:**
```
วิเคราะห์ architecture ของ internal/service/ ทั้ง directory
Context: Clean Architecture, 15 service files
Task: ตรวจ violations, dependency ผิดทิศทาง, circular dependencies
Output: dependency diagram (text) + รายการ violations + priority
```

### เทคนิคเพิ่มเติม

- ใช้ภาษาไทยหรืออังกฤษก็ได้ในการเขียน prompt (Claude เข้าใจทั้งคู่)
- อ้างอิงไฟล์โดยตรง: "ดูไฟล์ `handler/user.go` แล้ว..."
- ใช้ multi-turn: ถามทีละขั้น ไม่ต้องรวมทุกอย่างใน prompt เดียว
- ใช้ "think step by step" หรือ "วิเคราะห์ทีละขั้น" สำหรับ complex tasks

```bash
# Multi-turn ที่แนะนำ:
# รอบ 1: "วิเคราะห์ internal/domain/ บอกว่าใช้ pattern อะไร"
# รอบ 2: "สร้าง domain entity สำหรับ AuditLog ตาม pattern นั้น"
# รอบ 3: "สร้าง repository ตาม pattern เดียวกับ repository อื่น"
# รอบ 4: "สร้าง service layer พร้อม unit test"
# รอบ 5: "สร้าง HTTP handler + เพิ่ม routes"
```

> **เคล็ดลับ:** ถ้า task ใหญ่มาก ใช้ Plan Mode (กด `Shift+Tab`) ให้ Claude วางแผนก่อน

---

## 2.3 CLAUDE.md -- สมองของโปรเจกต์

### CLAUDE.md คืออะไร

ไฟล์ที่ Claude Code อ่านทุกครั้งเมื่อเริ่มเซสชัน (Session) ในโปรเจกต์ เหมือน onboarding doc
สำหรับ Claude -- บอก context, rules, conventions ไม่ควรเกิน 200 บรรทัด

### โครงสร้างที่แนะนำ

```markdown
# Project Name

## Overview
Brief project description

## Tech Stack
- Backend: Go 1.22, Fiber, GORM, PostgreSQL
- Frontend: React 18, TypeScript, TailwindCSS
- Infrastructure: Docker, GitHub Actions

## Project Structure
- /cmd — entry points
- /internal — business logic
- /pkg — shared packages
- /web — frontend application

## Conventions
- Go: follow standard Go conventions, use gofmt
- Naming: camelCase for Go, snake_case for DB columns
- Error handling: wrap errors with context

## Commands
- Build: `go build ./cmd/api`
- Test: `go test ./...`
- Lint: `golangci-lint run`
- Frontend: `cd web && npm run dev`

## Important Notes
- Always run tests before committing
- Database migrations in /migrations directory
- API versioning: /api/v1/
```

### สิ่งที่ควรใส่

- Tech stack, project structure -- Claude เลือก pattern ถูก สร้างไฟล์ถูกที่
- Build/test/lint commands -- ใช้คำสั่งที่ถูกต้อง
- Naming conventions, coding rules -- โค้ดที่สร้างตรงกับทีม
- Important patterns, business context -- เข้าใจ domain ตั้งชื่อถูก

### สิ่งที่ไม่ควรใส่

- ข้อมูลส่วนตัว, secrets, API keys -- ไฟล์อยู่ใน repo ข้อมูลรั่วได้
- เนื้อหาเกิน 200 บรรทัด -- เปลืองโทเค็น (Token) context เหลือน้อย
- ข้อมูลที่เปลี่ยนบ่อย -- ต้อง maintain บ่อย ข้อมูลเก่าทำ output ผิด
- โค้ดยาวเป็นตัวอย่าง -- อ้างอิงไฟล์แทน

> **คำเตือน:** อย่าใส่ API keys, database passwords, หรือ secret ใดๆ ใน CLAUDE.md
> เพราะไฟล์นี้จะถูก commit เข้า repository

### การสร้าง CLAUDE.md

```bash
# ให้ Claude สร้างให้อัตโนมัติ
claude /init

# หรือสร้างเอง แล้วให้ Claude ช่วย refine
claude -p "อ่านโครงสร้างโปรเจกต์แล้วช่วยปรับปรุง CLAUDE.md"
```

### Hierarchy ของ CLAUDE.md

```
~/.claude/CLAUDE.md              <-- ระดับ Global (ใช้กับทุกโปรเจกต์)
    v
project-root/CLAUDE.md           <-- ระดับ Project (ใช้กับโปรเจกต์นี้)
    v
project-root/subdir/CLAUDE.md    <-- ระดับ Subdirectory (override ได้)
```

ไฟล์ที่เฉพาะเจาะจงกว่าจะ override ไฟล์ที่กว้างกว่า

> **เคล็ดลับ:** ใช้ `~/.claude/CLAUDE.md` สำหรับ preferences ส่วนตัว เช่น
> "ตอบเป็นภาษาไทย", "ใช้ conventional commits" -- มีผลกับทุกโปรเจกต์อัตโนมัติ

---

## 2.4 .claude/ Directory Structure

### ภาพรวม

```
.claude/
├── settings.json          <-- การตั้งค่าระดับโปรเจกต์ (commit ได้)
├── settings.local.json    <-- การตั้งค่าส่วนตัว (gitignored)
├── rules/                 <-- กฎเงื่อนไข (Conditional Rules)
│   ├── coding-style.md
│   └── api-conventions.md
└── skills/                <-- Custom Skills
    └── create-endpoint/
        └── SKILL.md
```

### settings.json -- project-level settings (shared via git)

กำหนด permissions, allowed tools, model preferences:

```json
{
  "permissions": {
    "allow": [
      "Read", "Glob", "Grep",
      "Bash(make *)", "Bash(go test *)", "Bash(go build *)",
      "Bash(npm run *)", "Bash(docker compose *)",
      "Bash(git status)", "Bash(git diff *)", "Bash(git log *)"
    ],
    "deny": [
      "Bash(rm -rf *)", "Bash(git push --force *)",
      "Bash(git reset --hard *)", "Bash(curl * | bash)"
    ]
  }
}
```

> **หมายเหตุ:** `settings.json` ควร commit เข้า repo เพื่อให้ทีมใช้ค่าเดียวกัน

### settings.local.json -- local overrides (gitignored)

personal preferences, local paths:

```json
{
  "permissions": {
    "allow": ["Bash(code *)", "Bash(open *)"]
  }
}
```

> **คำเตือน:** ต้องเพิ่ม `.claude/settings.local.json` ใน `.gitignore` เสมอ

### rules/ -- additional instruction files

กฎที่ Claude Code โหลดอัตโนมัติตามบริบท:

**`.claude/rules/coding-style.md`:**

```markdown
# Go Coding Style
- ใช้ gofmt, ตรวจ error ทุกครั้ง (ห้ามใช้ _ สำหรับ error)
- camelCase (private), PascalCase (public)
- ทุก exported function ต้องมี godoc comment
- Imports แยก 3 กลุ่ม (stdlib, external, internal)
- ห้ามใช้ panic() ใน production code
```

**`.claude/rules/api-conventions.md`:**

```markdown
# API Conventions
- Versioning: /api/v1/ prefix
- Response: ใช้ pkg/response.JSON()
- Error: { "error": { "code": "...", "message": "..." } }
- Pagination: page + per_page query params
```

### skills/ -- custom skills

แต่ละ skill เป็น directory ที่มี `SKILL.md` เรียกใช้ด้วย slash commands:

**`.claude/skills/create-endpoint/SKILL.md`:**

```markdown
# Skill: Create REST Endpoint
1. สร้าง domain model ใน internal/domain/{entity}/model.go
2. สร้าง repository interface + implementation
3. สร้าง service ใน internal/service/{entity}_service.go
4. สร้าง handler ใน internal/handler/{entity}_handler.go
5. เพิ่ม routes ใน router.go
6. สร้าง unit test + รัน `make test`
ดูตัวอย่างจาก internal/domain/user/
```

---

## 2.5 Memory System

### MEMORY.md

ถูกสร้างที่ `~/.claude/projects/<project-hash>/memory/MEMORY.md` เก็บข้อมูลข้ามเซสชัน
ถูกโหลดอัตโนมัติทุกเซสชัน (แต่ truncate หลังบรรทัดที่ 200)

### การใช้งาน

```bash
# แก้ไข memory
/memory

# บอกให้ Claude จำ
"จำไว้ว่าเราใช้ bun แทน npm ในโปรเจกต์นี้"

# Auto-memory: Claude จะจดจำ patterns สำคัญเอง (ถามก่อนบันทึกเสมอ)
```

### ตัวอย่าง MEMORY.md

```markdown
# Memory

## Preferences
- ตอบเป็นภาษาไทย ศัพท์เทคนิคใช้ภาษาอังกฤษ
- ใช้ Conventional Commits
- Prefer table-driven tests ใน Go

## Project Patterns
- bgrimm-aggregator: Clean Architecture, handler -> service -> repository
- bgrimm-aggregator: ทุก API response ใช้ pkg/response.JSON()

## Recurring Solutions
- GORM ต้อง Preload associations ก่อนใช้ ไม่งั้นได้ nil
- Docker Compose v2 ใช้ "docker compose" ไม่ใช่ "docker-compose"
```

### ควรจำ vs ไม่ควรจำ

| ควรจำ | ไม่ควรจำ |
|-------|---------|
| Stable patterns, conventions | Session-specific context |
| Important file paths | Temporary state |
| User preferences | สิ่งที่ duplicate กับ CLAUDE.md |
| Solutions to recurring problems | ข้อมูลที่เปลี่ยนบ่อย |

---

## 2.6 Context Management

### ปัญหา Context Window

Claude มี context window จำกัด เมื่อบริบทเต็มจะเริ่ม compress อัตโนมัติ ข้อมูลเก่าอาจหาย
อาการ: ตอบช้าลง, ลืมบริบทช่วงแรก, ค่าโทเค็นสูง

### คำสั่งจัดการ

| คำสั่ง | ใช้เมื่อ | ผลลัพธ์ |
|--------|---------|---------|
| `/compact` | บริบทเริ่มเยอะ, ประหยัดโทเค็น | บีบอัดบริบท รักษาข้อมูลสำคัญ |
| `/context` | ต้องการดูว่ามีอะไรในบริบท | แสดงรายการไฟล์และข้อมูล |
| `/clear` | เริ่มใหม่, เปลี่ยน task | ล้างบริบททั้งหมด |

**`/compact` ใส่คำสั่งเพิ่มเติมได้:**

```bash
/compact เก็บข้อมูลเกี่ยวกับ API endpoints ที่สร้างไปแล้วด้วย
```

### Best Practices

- ทำ task เดียวต่อ session (single-task session)
- ใช้ `/compact` เมื่อเริ่มรู้สึกว่า Claude "ลืม" สิ่งที่คุยไว้
- ใช้ `/clear` เมื่อเปลี่ยน task ไปเลย
- ใช้ `claude -c` เพื่อต่อ session เดิมถ้ายังทำ task เดิมอยู่
- อ้างอิงไฟล์แทน copy-paste โค้ด
- ใช้ CLAUDE.md แทนการอธิบายซ้ำทุก session

---

## 2.7 Best Practices รวม

### 1. ทำทีละ task

**DO:** 1 session = 1 task (เช่น "สร้าง User CRUD", "เพิ่ม auth middleware", "แก้ bug")

**DON'T:** ยัดทุกอย่างใน session เดียว "CRUD + auth + bug fix + refactor + ..."

### 2. ตรวจสอบก่อน approve

**DO:** อ่าน diff ทุกบรรทัด ตรวจ logic ตรวจ sensitive data ก่อน approve

**DON'T:** approve 5 file edits โดยไม่อ่านเลย

### 3. ใช้ /cost ตรวจสอบเป็นระยะ

**DO:** ตรวจ `/cost` ทุก 15-20 นาที ประเมินว่าควร `/compact` หรือยัง

**DON'T:** ปล่อยให้ session ยาวโดยไม่เคยตรวจ cost

### 4. Plan Mode ก่อน implement

**DO:** กด `Shift+Tab` เข้า Plan Mode ก่อนงานซับซ้อน Claude เสนอแผนก่อนลงมือ

**DON'T:** ให้ Claude ลุยงานใหญ่โดยไม่มีแผน

### 5. CLAUDE.md เป็นปัจจุบัน

**DO:** review ทุก sprint -- tech stack, conventions, commands ยังถูกต้องมั้ย?

**DON'T:** สร้างครั้งเดียวไม่เคยแก้ ปล่อยให้ข้อมูลเก่า

### 6. อย่า approve โดยไม่อ่าน

**DO:** ตรวจ bash command ว่าถูกต้อง ไม่มี flag แปลก ก่อน approve

**DON'T:** approve `rm -rf` โดยไม่อ่านว่าจะลบอะไร

### 7. ใช้ git -- commit ก่อนให้ Claude ทำงานใหญ่

**DO:**
```bash
git add -A && git commit -m "checkpoint: before refactor"
# ให้ Claude ทำงาน ... ถ้าไม่พอใจ -> git checkout -- .
```

**DON'T:** ให้ Claude แก้ 20 ไฟล์โดยไม่มี checkpoint

### 8. ระวัง secrets

**DO:** บอกข้อมูลที่จำเป็นโดยตรง "database อยู่ที่ localhost:5432 ชื่อ myapp"

**DON'T:** "อ่านไฟล์ .env แล้วช่วยตั้งค่า" -- credentials จะเข้าไปใน context window

> **คำเตือน:** ตรวจสอบว่า `.gitignore` ครอบคลุม `.env`, `*.pem`, `*.key`,
> `credentials.json`, `.claude/settings.local.json`

### สรุป Do / Don't

```
DO                                       DON'T
──────────────────────────────────────────────────────────────
1 task = 1 session                       ยัดทุกอย่างใน session เดียว
อ่าน diff ก่อน approve                   approve โดยไม่อ่าน
ตรวจ /cost เป็นระยะ                      ปล่อย session ยาวไม่ตรวจ cost
Plan Mode ก่อนงานใหญ่                    ให้ Claude ลุยโดยไม่มีแผน
อัพเดท CLAUDE.md ทุก sprint              สร้างครั้งเดียวไม่เคยแก้
git commit ก่อนงานใหญ่                   แก้ 20 ไฟล์ไม่มี checkpoint
ใช้ /compact เมื่อ token สูง              ปล่อยให้ context เต็ม
อ้างอิงไฟล์แทน paste โค้ด                paste โค้ด 500 บรรทัดใน prompt
ตรวจ security / edge cases               trust output 100%
ใช้ .gitignore ป้องกัน secrets           ให้ Claude อ่าน .env โดยตรง
```

---

## Cross-references

- [บทที่ 1: บทนำและพื้นฐาน](./01_intro.md)
- [บทที่ 3: การใช้งานขั้นสูง](./03_advanced_usage.md)
- [บทที่ 4: Full-Stack Collaboration](./04_fullstack_collaboration.md)
- [บทที่ 5: Specification-Driven Development](./05_spec_driven_dev.md)
- [บทที่ 6: Architecture & Design](./06_architecture_design.md)
- [บทที่ 7: QA, Security, and Review](./07_qa_security_review.md)
- [บทที่ 8: บทบาทและความรับผิดชอบ](./08_roles_responsibilities.md)
- [บทที่ 9: Cheat Sheet](./09_cheat_sheet.md)

---

> **หมายเหตุท้ายบท:** เนื้อหาในบทนี้เป็นแนวปฏิบัติที่แนะนำ (Best Practices)
> ซึ่งอาจปรับเปลี่ยนได้ตามบริบทของโปรเจกต์และทีม สิ่งสำคัญที่สุดคือทำความเข้าใจ
> หลักการ แล้วปรับใช้ให้เหมาะกับสถานการณ์จริง
