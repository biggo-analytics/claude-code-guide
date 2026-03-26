# บทที่ 3: การใช้งานขั้นสูง

## Advanced Usage

> เอกสารภายในสำหรับทีมพัฒนา — ปรับปรุงล่าสุด: มีนาคม 2026
>
> ครอบคลุมฟีเจอร์ขั้นสูงของ Claude Code: ซับเอเจนต์ (Subagent), สกิล (Skill),
> โหมดวางแผน (Plan Mode), เวิร์คทรี (Worktree), ฮุค (Hook), MCP, ระบบสิทธิ์ และอื่น ๆ

---

## สารบัญ

- [3.1 Subagents (ซับเอเจนต์)](#31-subagents-ซับเอเจนต์)
- [3.2 Skills System](#32-skills-system)
- [3.3 Plan Mode (โหมดวางแผน)](#33-plan-mode-โหมดวางแผน)
- [3.4 Worktrees (เวิร์คทรี)](#34-worktrees-เวิร์คทรี)
- [3.5 Hooks System (ระบบฮุค)](#35-hooks-system-ระบบฮุค)
- [3.6 MCP (Model Context Protocol)](#36-mcp-model-context-protocol)
- [3.7 Permission System ขั้นสูง](#37-permission-system-ขั้นสูง)
- [3.8 Systems Thinking](#38-systems-thinking)
- [3.9 คำสั่งเพิ่มเติม](#39-คำสั่งเพิ่มเติม)
- [Cross-references](#cross-references)

---

## 3.1 Subagents (ซับเอเจนต์)

ซับเอเจนต์ (Subagent) คือกลไกที่ Claude Code สามารถ spawn เอเจนต์ (Agent) ย่อย
เพื่อทำงานแบบขนาน (parallel) หรือแบบแยกบริบท (isolated context) ได้
ช่วยให้งานที่ต้องค้นหาหลายจุดใน codebase หรือวิเคราะห์หลายแง่มุมพร้อมกัน
ทำได้เร็วขึ้นอย่างมาก

### 3.1.1 ประเภท Built-in Subagents

Claude Code มีซับเอเจนต์ในตัว 3 ประเภทหลัก:

#### Explore Subagent

ใช้สำหรับค้นหาข้อมูลใน codebase อย่างรวดเร็ว ทำงานแบบ read-only เท่านั้น:

- ค้นหาไฟล์หรือ patterns เฉพาะ
- ตอบคำถามเกี่ยวกับ architecture ของโปรเจกต์
- หา function signatures, struct definitions, interface implementations
- สำรวจ directory structure

> **เคล็ดลับ:** Explore subagent ใช้เฉพาะ read-only tools (`Glob`, `Grep`, `Read`)
> จึงปลอดภัยและรวดเร็ว ไม่มีการแก้ไขไฟล์ใด ๆ

#### Plan Subagent

ใช้สำหรับวางแผนการ implementation วิเคราะห์:

- ไฟล์ที่เกี่ยวข้องที่ต้องแก้ไข
- ลำดับขั้นตอนการทำงาน
- Architectural trade-offs ของแต่ละแนวทาง
- ผลกระทบต่อส่วนอื่นของระบบ

#### General-purpose Subagent

ซับเอเจนต์แบบทั่วไปที่สามารถใช้ tools ทั้งหมดได้ เหมาะกับงาน multi-step:

- ค้นหา + อ่าน + วิเคราะห์ ในขั้นตอนเดียว
- งานที่ต้องทั้งอ่านไฟล์หลายไฟล์แล้วสรุปผล
- การเปรียบเทียบ implementations ในหลายส่วนของโค้ด

### 3.1.2 Custom Agents

ทีมสามารถสร้างเอเจนต์เฉพาะทางได้โดยวางไฟล์ Markdown ใน `.claude/agents/` directory
แต่ละไฟล์กำหนด behavior ของเอเจนต์ตัวหนึ่ง:

```
.claude/
  agents/
    api-reviewer.md
    migration-helper.md
    test-generator.md
```

ตัวอย่างไฟล์ `.claude/agents/test-runner.md`:

```markdown
# Test Runner Agent

คุณเป็น agent สำหรับรัน tests

## Tools
- Bash
- Read
- Grep

## Instructions
1. หา test files ที่เกี่ยวข้องกับไฟล์ที่ระบุ
2. รัน tests ด้วย `go test` หรือ `npm test` ตาม project type
3. รายงานผลลัพธ์พร้อมสรุป pass/fail
4. ถ้ามี failure ให้วิเคราะห์สาเหตุเบื้องต้น
```

ตัวอย่างไฟล์ `.claude/agents/api-reviewer.md`:

```markdown
# API Reviewer

คุณคือ reviewer สำหรับ Go API endpoints ของทีม ให้ตรวจสอบตาม checklist:

1. ใช้ standard HTTP methods ถูกต้อง (GET, POST, PUT, DELETE)
2. Response format เป็น JSON ตาม RFC 7807 สำหรับ errors
3. มี input validation ครบทุก field
4. มี authorization middleware
5. มี unit tests ครอบคลุม happy path และ error cases
6. Naming convention: camelCase สำหรับ JSON fields
7. Pagination ใช้ cursor-based สำหรับ list endpoints
```

### 3.1.3 คำสั่ง `/agents`

ใช้คำสั่ง `/agents` เพื่อดูรายการเอเจนต์ทั้งหมดที่มีอยู่:

```bash
# ใน Claude Code interactive session
/agents
```

จะแสดงรายการ built-in agents และ custom agents ที่อยู่ใน `.claude/agents/`

### 3.1.4 เมื่อไหร่ควรใช้ Subagent

| สถานการณ์                                       | ควรใช้ Subagent? | เหตุผล                                |
|-------------------------------------------------|:----------------:|---------------------------------------|
| ค้นหาข้อมูลจากหลายไฟล์พร้อมกัน                    | ใช่              | Explore subagent ทำงานขนานได้           |
| วิเคราะห์ dependencies ข้าม packages             | ใช่              | ต้อง research หลายจุด                  |
| แก้ไข typo ใน 1 ไฟล์                             | ไม่จำเป็น        | งานเล็กไม่ต้อง spawn agent             |
| ทำ parallel development หลาย features            | ใช่              | แยก context ชัดเจน                    |
| วางแผน architecture change                      | ใช่              | Plan subagent วิเคราะห์ได้ละเอียด       |
| สร้าง CRUD ตาม pattern เดิม                      | ไม่จำเป็น        | pattern ชัดอยู่แล้ว                    |

### 3.1.5 ตัวอย่างการใช้งานจริง

**กรณี: ค้นหาทุก endpoint ที่เกี่ยวกับ authentication**

```
พรอมต์ (Prompt): "ค้นหาทุก endpoint ที่เกี่ยวกับ user authentication
แล้วสรุปว่าแต่ละ endpoint ทำอะไร ใช้ middleware อะไร
และมี test coverage หรือไม่"
```

Claude Code จะ spawn subagents แบบขนาน:

1. Subagent A: ค้นหาไฟล์ route definitions ที่มี "auth" หรือ "login"
2. Subagent B: ค้นหา middleware ที่เกี่ยวข้อง
3. Subagent C: ค้นหา test files ที่เกี่ยวข้อง

ผลลัพธ์จะถูกรวมเป็นสรุปครบถ้วนในการตอบเดียว

> **หมายเหตุ:** การใช้ซับเอเจนต์ไม่ต้องระบุเอง — Claude Code จะตัดสินใจเองว่า
> ควร spawn subagent เมื่อไหร่ตาม context ของงาน แต่การเขียนพรอมต์ที่บอกชัดว่า
> ต้องการข้อมูลจากหลายแหล่ง จะช่วยกระตุ้นให้ใช้ subagents ได้ดีขึ้น

---

## 3.2 Skills System

สกิล (Skill) คือ custom slash commands ที่ทีมสร้างขึ้นเพื่อให้เป็น reusable workflows
ทุกคนในทีมเรียกใช้ได้ผ่านคำสั่ง `/skill-name`

### 3.2.1 โครงสร้าง Skills Directory

Skills อยู่ใน `.claude/skills/` directory โดยแต่ละ skill เป็น directory ที่มีไฟล์
`SKILL.md` เป็นหลัก:

```
.claude/skills/
├── deploy/
│   └── SKILL.md
├── create-migration/
│   └── SKILL.md
└── review-pr/
    └── SKILL.md
```

### 3.2.2 โครงสร้างไฟล์ SKILL.md

```markdown
---
name: skill-name
description: คำอธิบายสั้น ๆ ว่า skill ทำอะไร
user_invocable: true
---

# Skill Title

คำสั่งโดยละเอียดที่ Claude Code จะทำตามเมื่อ skill ถูกเรียกใช้
สามารถอ้างอิง files, patterns, conventions ของโปรเจกต์ได้
```

ส่วน frontmatter (`---` block) รองรับ fields หลัก:

| Field            | Type    | คำอธิบาย                                     | จำเป็น |
|------------------|---------|----------------------------------------------|--------|
| `name`           | string  | ชื่อที่ใช้เรียก skill (ตรงกับชื่อ directory)     | ใช่    |
| `description`    | string  | คำอธิบายสั้นที่แสดงในรายการ skills              | ใช่    |
| `user_invocable` | boolean | กำหนดว่าผู้ใช้เรียกได้โดยตรงหรือไม่             | ไม่    |

### 3.2.3 ตัวอย่าง SKILL.md: สร้าง Database Migration

```markdown
---
name: create-migration
description: สร้าง database migration file ใหม่
user_invocable: true
---

# Create Migration Skill

## Instructions
1. รับชื่อ migration จาก argument
2. สร้างไฟล์ migration ใน /migrations directory
3. ใช้ format: YYYYMMDDHHMMSS_name.sql
4. สร้างทั้ง up และ down migration

## Template

### UP migration
```sql
-- UP: {timestamp}_{name}.up.sql
CREATE TABLE IF NOT EXISTS {table_name} (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    updated_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
);
```

### DOWN migration
```sql
-- DOWN: {timestamp}_{name}.down.sql
DROP TABLE IF EXISTS {table_name};
```
```

การเรียกใช้:

```bash
# ใน Claude Code interactive session
/create-migration add_users_table
```

### 3.2.4 ตัวอย่าง SKILL.md: Review PR

```markdown
---
name: review-pr
description: ตรวจสอบ PR ตาม checklist มาตรฐานของทีม
user_invocable: true
---

# PR Review Skill

## Checklist

### Security
- ไม่มี SQL injection (ใช้ parameterized queries)
- ไม่มี hardcoded credentials
- Input validation ครบทุก endpoint
- Authorization check ก่อนเข้าถึงข้อมูล

### Performance
- ไม่มี N+1 query
- Database queries มี proper indexes
- Pagination สำหรับ list endpoints

### Code Quality
- Error handling ครบถ้วน ไม่ swallow errors
- Logging มี context เพียงพอ (request ID, user ID)
- Function names สื่อความหมาย
- ไม่มี dead code

### Testing
- Unit tests สำหรับ business logic
- Error cases ถูก test
- Test data ไม่ hardcode ค่าจาก production
```

### 3.2.5 การสร้าง Custom Skill

ขั้นตอนการสร้าง skill ใหม่:

```bash
# 1. สร้าง directory
mkdir -p .claude/skills/my-skill

# 2. สร้างไฟล์ SKILL.md
touch .claude/skills/my-skill/SKILL.md

# 3. เขียน frontmatter + instructions ใน SKILL.md

# 4. ทดสอบโดยเรียก /my-skill ใน Claude Code
```

> **เคล็ดลับ:** ตั้งชื่อ skill ให้สั้น จำง่าย และสื่อความหมาย เช่น `/create-api`
> ดีกว่า `/create-new-api-endpoint-with-full-crud`

> **คำเตือน:** Skill instructions จะถูกส่งเป็นส่วนหนึ่งของ system prompt
> หาก instructions ยาวเกินไปจะใช้โทเค็น (Token) context มาก
> แนะนำให้เขียนกระชับ ชัดเจน ไม่เกิน 200 บรรทัด

---

## 3.3 Plan Mode (โหมดวางแผน)

Plan Mode คือโหมดที่ Claude Code จะวิเคราะห์ อ่านโค้ด และวางแผนเท่านั้น
**โดยไม่แก้ไขไฟล์ใด ๆ** เหมาะสำหรับงานที่ต้องคิดให้รอบคอบก่อนลงมือทำ

### 3.3.1 การเปิด/ปิด Plan Mode

```
Shift+Tab  →  สลับระหว่าง Normal Mode กับ Plan Mode
/plan      →  เปิด Plan Mode ด้วยคำสั่ง
```

สังเกตสถานะได้จากตัวบ่งชี้ (indicator) ที่มุมล่างของหน้าจอ:

- **Plan** = อยู่ใน Plan Mode (read-only)
- **Normal** = อยู่ใน Normal Mode (read + write)

### 3.3.2 Workflow การใช้ Plan Mode

```
┌──────────────────────────────────────────────────────┐
│  1. เปิด Plan Mode (Shift+Tab หรือ /plan)            │
│  2. อธิบาย task ที่ต้องการทำ                          │
│  3. Claude Code อ่านโค้ด วิเคราะห์ วางแผน             │
│  4. ทบทวนแผน — ถามเพิ่มเติมถ้าไม่แน่ใจ               │
│  5. ปิด Plan Mode (Shift+Tab)                        │
│  6. สั่งให้ implement ตามแผน                          │
└──────────────────────────────────────────────────────┘
```

### 3.3.3 เมื่อไหร่ควรใช้ Plan Mode

| สถานการณ์                                    | ควรใช้ Plan Mode? | เหตุผล                            |
|----------------------------------------------|:-----------------:|-----------------------------------|
| เพิ่ม field ง่าย ๆ ใน struct                   | ไม่จำเป็น         | งานเล็ก ทำได้เลย                   |
| Refactoring ใหญ่ข้าม 10+ ไฟล์                | ใช่               | ต้องวางแผนก่อนเพื่อไม่พลาด         |
| ตัดสินใจ architecture (event-driven?)         | ใช่               | ต้องวิเคราะห์ trade-offs           |
| แก้บั๊กง่าย ๆ                                 | ไม่จำเป็น         | fix ได้ทันที                       |
| เพิ่ม feature ใหม่ที่กระทบหลาย layers         | ใช่               | ต้องเห็นภาพรวมก่อน                |
| เปลี่ยน database schema                       | ใช่               | กระทบหลาย layers                  |
| สร้าง CRUD endpoint ตาม pattern เดิม          | ไม่จำเป็น         | pattern ชัดอยู่แล้ว                |
| ย้ายจาก REST เป็น gRPC                        | ใช่               | เปลี่ยนแปลงขนาดใหญ่               |

### 3.3.4 ตัวอย่าง Prompt ใน Plan Mode

**สถานการณ์:** ต้องการเพิ่ม role-based access control (RBAC)

```
วิเคราะห์ระบบ authentication ปัจจุบัน แล้ววางแผนเพิ่ม role-based access control (RBAC)
โดย:
1. วิเคราะห์โครงสร้างที่มี
2. ออกแบบ database schema สำหรับ roles/permissions
3. วางแผน middleware changes
4. ระบุ files ที่ต้องแก้ไข
5. เรียงลำดับการทำงาน
```

### 3.3.5 ตัวอย่าง Plan Mode Session

```
[Plan Mode ON]

User: ฉันต้องการเพิ่ม audit logging สำหรับทุก write operation
      (CREATE, UPDATE, DELETE) โดยบันทึกลง audit_logs table
      ใน PostgreSQL ต้องเก็บ: who, what, when, old_value, new_value

Claude Code: (อ่านโค้ดและวิเคราะห์)

## แผนการ Implementation

### 1. Database Layer
- สร้าง migration สำหรับ audit_logs table
- Columns: id, actor_id, action, resource_type, resource_id,
  old_value (JSONB), new_value (JSONB), ip_address, created_at

### 2. Repository Layer
- สร้าง internal/repository/audit_log.go
- สร้าง sqlc query ใน queries/audit_log.sql

### 3. Middleware Approach
- สร้าง audit middleware ที่ wrap service layer
- ใช้ Decorator pattern:
  UserService → AuditUserService → UserRepository

### 4. ไฟล์ที่ต้องแก้ไข
- migrations/20260310_create_audit_logs.up.sql (ใหม่)
- migrations/20260310_create_audit_logs.down.sql (ใหม่)
- internal/repository/audit_log.go (ใหม่)
- internal/service/audit.go (ใหม่)
- internal/middleware/audit.go (ใหม่)
- cmd/api/wire.go (แก้ไข — เพิ่ม DI)

### 5. Trade-offs
- Option A: Middleware pattern — ง่าย แต่ต้องทำทุก service
- Option B: Database trigger — ครบถ้วน แต่ debug ยาก
- แนะนำ: Option A เพราะ maintain ง่ายกว่า

User: ดี ใช้ Option A

[Plan Mode OFF — กด Shift+Tab]

User: implement ตามแผนที่วางไว้
```

> **เคล็ดลับ:** ใน Plan Mode สามารถถามคำถามต่อเนื่องเพื่อขุดลึกในแต่ละจุดได้
> เช่น "อธิบาย trade-off ของ Option B เพิ่มเติม" หรือ
> "ถ้าใช้ database trigger จะมีปัญหาเรื่อง performance ไหม"

---

## 3.4 Worktrees (เวิร์คทรี)

เวิร์คทรี (Worktree) คือฟีเจอร์ของ Git ที่ทำงานหลาย branch พร้อมกันได้
โดยแต่ละ branch อยู่ใน directory แยก Claude Code รองรับการทำงานกับ worktrees
ผ่าน flag `-w` ทำให้ไม่ต้อง stash/switch branch ไปมา

### 3.4.1 การใช้งานเบื้องต้น

```bash
# สร้าง worktree สำหรับ feature
claude -w feature-auth

# Claude จะสร้าง git worktree แยก
# ทำงานใน isolated copy ของ repo
```

เมื่อใช้ flag `-w` Claude Code จะ:

1. สร้าง git worktree ใหม่ (ถ้ายังไม่มี)
2. สร้าง branch ใหม่ชื่อตาม argument
3. เปิด session ใน worktree directory นั้น
4. ทำงานได้อิสระจาก main working directory

### 3.4.2 Use Cases

**ทำ feature branch ขณะ main branch ยังทำงานอื่น:**

```bash
# Terminal 1: ทำ feature ใหม่
claude -w feature-order-export

# Terminal 2: แก้ bug ด่วนบน main
claude   # ทำงานใน main directory ปกติ

# Terminal 3: ทำ feature อื่นพร้อมกัน
claude -w feature-notification-service
```

**ทดลอง approach ต่าง ๆ แบบขนาน:**

```bash
# ลอง approach A
claude -w experiment-event-driven

# ลอง approach B
claude -w experiment-saga-pattern

# เปรียบเทียบผลลัพธ์แล้วเลือก approach ที่ดีกว่า
```

**parallel development: หลาย feature พร้อมกัน:**

```bash
# Developer ทำงาน 3 features พร้อมกัน
claude -w feature-payment-gateway    # Terminal 1
claude -w feature-email-templates    # Terminal 2
claude -w feature-report-dashboard   # Terminal 3
```

### 3.4.3 การจัดการ Worktree

```bash
# ดู worktrees ทั้งหมด
git worktree list

# ผลลัพธ์ตัวอย่าง:
# /Users/dev/project              abc1234 [main]
# /Users/dev/project-feature-auth def5678 [feature-auth]
# /Users/dev/project-fix-login    ghi9012 [fix-login]
```

**Worktree Lifecycle:**

```
┌─────────┐     ┌─────────┐     ┌─────────┐     ┌─────────┐
│ Create  │ ──→ │  Work   │ ──→ │  Merge  │ ──→ │ Cleanup │
└─────────┘     └─────────┘     └─────────┘     └─────────┘
  claude -w       พัฒนาตาม       merge PR         ลบ worktree
  feature-x       ปกติ           เข้า main         + branch
```

ขั้นตอน cleanup เมื่อ merge แล้ว:

```bash
# ลบ worktree
git worktree remove .worktrees/feature-auth

# ลบ branch (ถ้า merge แล้ว)
git branch -d feature-auth
```

### 3.4.4 ข้อควรระวัง

> **คำเตือน:** แต่ละ worktree ใช้ disk space แยกเพราะมี working copy ของตัวเอง
> ถ้าโปรเจกต์ใหญ่ ควรลบ worktree ที่ไม่ใช้แล้วเพื่อประหยัดพื้นที่

- worktree จะถูกลบอัตโนมัติถ้าไม่มี changes ที่ยัง uncommitted
- ถ้ามี changes จะ return path และ branch name ให้จัดการเอง
- อย่าลืมรัน `npm install` หรือ `go mod download` ในแต่ละ worktree
  เพราะ `node_modules/` หรือ binary cache อาจไม่ถูก share
- ระวังเรื่อง database migration — worktree อาจ run migration ที่ branch หลักยังไม่มี
- ไฟล์ `.env` ต้อง copy หรือ symlink ไปยัง worktree ใหม่เอง

---

## 3.5 Hooks System (ระบบฮุค)

ระบบฮุค (Hook) ใน Claude Code คือ shell commands ที่ทำงานอัตโนมัติ
เมื่อเกิด events ที่กำหนดไว้ ช่วยให้ integrate กับ toolchain ของทีมได้อย่างราบรื่น

### 3.5.1 Event Types

| Event          | จังหวะที่ทำงาน                        | Use Case หลัก                    |
|----------------|---------------------------------------|----------------------------------|
| `PreToolUse`   | ก่อนเรียก tool (เช่น ก่อน write file) | Validation, ป้องกันการแก้ไขบางไฟล์ |
| `PostToolUse`  | หลังเรียก tool (เช่น หลัง write file) | Auto-format, lint, test          |
| `Notification` | เมื่อ Claude Code ส่ง notification    | แจ้งเตือนผ่าน Slack, webhook     |
| `Stop`         | เมื่อเอเจนต์หยุดทำงาน                | สรุปผล, cleanup, notification    |

### 3.5.2 Configuration ใน settings.json

Hooks ตั้งค่าใน settings file (`~/.claude/settings.json` สำหรับ user-level
หรือ `.claude/settings.json` สำหรับ project-level):

```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Write|Edit",
        "command": "prettier --write $CLAUDE_FILE_PATH"
      }
    ],
    "PreToolUse": [
      {
        "matcher": "Bash",
        "command": "echo 'About to run: $CLAUDE_TOOL_INPUT'"
      }
    ]
  }
}
```

### 3.5.3 Hook Object Structure

แต่ละ hook object มี fields ดังนี้:

| Field     | Type   | คำอธิบาย                                                          |
|-----------|--------|------------------------------------------------------------------|
| `matcher` | string | Regex pattern สำหรับ match tool name (ว่างเปล่า = match ทุก tool) |
| `command` | string | Shell command ที่จะรัน — รองรับ environment variables              |

### 3.5.4 Environment Variables ที่ Hooks ได้รับ

| Variable              | มีใน Event      | คำอธิบาย                         |
|-----------------------|-----------------|----------------------------------|
| `$CLAUDE_TOOL_NAME`   | Pre/PostToolUse | ชื่อ tool ที่ถูกเรียก              |
| `$CLAUDE_FILE_PATH`   | Pre/PostToolUse | path ของไฟล์ที่ถูกแก้ไข           |
| `$CLAUDE_TOOL_INPUT`  | Pre/PostToolUse | input ของ tool (JSON string)     |
| `$CLAUDE_NOTIFICATION`| Notification    | ข้อความ notification              |
| `$CLAUDE_EXIT_STATUS` | Stop            | สถานะการหยุด                     |

### 3.5.5 ตัวอย่างการใช้งาน Hooks

#### 1. Auto-format หลังแก้ไขไฟล์

```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Write|Edit",
        "command": "if echo $CLAUDE_FILE_PATH | grep -q '\\.go$'; then gofmt -w $CLAUDE_FILE_PATH && goimports -w $CLAUDE_FILE_PATH; fi"
      },
      {
        "matcher": "Write|Edit",
        "command": "if echo $CLAUDE_FILE_PATH | grep -qE '\\.(ts|tsx|js|jsx)$'; then npx prettier --write $CLAUDE_FILE_PATH; fi"
      }
    ]
  }
}
```

#### 2. Lint check ก่อน commit

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Bash",
        "command": "if echo $CLAUDE_TOOL_INPUT | grep -q 'git commit'; then golangci-lint run ./...; fi"
      }
    ]
  }
}
```

#### 3. Notification เมื่องานเสร็จ (macOS)

```json
{
  "hooks": {
    "Stop": [
      {
        "matcher": "",
        "command": "osascript -e 'display notification \"Claude Code task completed\" with title \"Claude Code\"'"
      }
    ]
  }
}
```

#### 4. Auto-test หลังแก้ไขไฟล์ test

```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Write|Edit",
        "command": "if echo $CLAUDE_FILE_PATH | grep -q '_test\\.go$'; then go test $(dirname $CLAUDE_FILE_PATH)/...; fi"
      }
    ]
  }
}
```

### 3.5.6 ข้อควรระวังสำหรับ Hooks

> **คำเตือน:** Hook commands จะรันด้วย shell ของผู้ใช้ ดังนั้นต้องมั่นใจว่า
> command ที่ระบุสามารถรันได้จาก path ปัจจุบัน และระวังเรื่อง security —
> อย่าใส่ secrets ใน hook commands

> **เคล็ดลับ:** ทดสอบ hook command ใน terminal ก่อนใส่ใน settings.json เสมอ
> เพื่อให้มั่นใจว่าทำงานถูกต้อง

---

## 3.6 MCP (Model Context Protocol)

MCP (Model Context Protocol) คือโปรโตคอลสำหรับเชื่อมต่อ Claude Code กับ
external services ต่าง ๆ ทำให้สามารถอ่าน/เขียนข้อมูลจากระบบภายนอกได้โดยตรง
จากภายใน Claude Code session

### 3.6.1 Configuration

MCP servers ตั้งค่าใน `.mcp.json` ที่ root ของโปรเจกต์:

```json
{
  "mcpServers": {
    "server-name": {
      "command": "command-to-start-server",
      "args": ["arg1", "arg2"],
      "env": {
        "ENV_VAR": "value"
      }
    }
  }
}
```

### 3.6.2 ตัวอย่าง MCP Servers

#### PostgreSQL Server

เชื่อมต่อกับ database โดยตรง (query, inspect schema):

```json
{
  "mcpServers": {
    "postgres": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-postgres"],
      "env": {
        "DATABASE_URL": "postgresql://user:pass@localhost:5432/mydb"
      }
    }
  }
}
```

เมื่อเชื่อมต่อแล้ว สามารถสั่ง Claude Code ได้เช่น:

```
"ดู schema ของ table orders"
"query ดู orders ที่มี status = 'pending' ย้อนหลัง 7 วัน"
"ดู indexes ทั้งหมดของ table users"
```

#### GitHub Server

เชื่อมต่อกับ GitHub เพื่อจัดการ issues, PRs, reviews:

```json
{
  "mcpServers": {
    "github": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-github"],
      "env": {
        "GITHUB_TOKEN": "${GITHUB_TOKEN}"
      }
    }
  }
}
```

#### Slack Server

เชื่อมต่อกับ Slack เพื่ออ่าน/ส่งข้อความ:

```json
{
  "mcpServers": {
    "slack": {
      "command": "npx",
      "args": ["-y", "@anthropic/mcp-server-slack"],
      "env": {
        "SLACK_BOT_TOKEN": "${SLACK_BOT_TOKEN}"
      }
    }
  }
}
```

#### Figma Server

เชื่อมต่อกับ Figma เพื่อสร้าง/อ่าน design, แปลง UI เป็น Figma design:

```json
{
  "mcpServers": {
    "figma": {
      "command": "npx",
      "args": ["-y", "@anthropic/figma-mcp-server"],
      "env": {
        "FIGMA_ACCESS_TOKEN": "${FIGMA_ACCESS_TOKEN}"
      }
    }
  }
}
```

> 💡 สำหรับ claude.ai (Web UI) — Figma MCP เป็น built-in integration
> เปิดใช้ที่ Settings → Integrations → Figma → Connect โดยไม่ต้องตั้งค่า `.mcp.json`

เมื่อเชื่อมต่อแล้ว สามารถสั่ง Claude ได้เช่น:

```
"สร้างไฟล์ Figma ใหม่ชื่อ Dashboard Redesign"
"ดูหน้าเว็บ https://example.com แล้วสร้าง design ใน Figma ตาม UI ที่เห็น"
"ดู screenshot นี้ แล้วแปลงเป็น Figma design พร้อม Auto Layout"
"ถ่ายภาพ design ใน Figma ไฟล์นี้ให้ดูหน่อย"
```

📌 **ดูตัวอย่างเต็ม:** [Use Case: Web URL / Screenshot → Figma](examples/08_figma_mcp_usecase.md)

### 3.6.3 ตัวอย่าง .mcp.json แบบเต็มสำหรับทีม

```json
{
  "mcpServers": {
    "postgres": {
      "command": "npx",
      "args": [
        "-y",
        "@modelcontextprotocol/server-postgres",
        "postgresql://dev_user:dev_pass@localhost:5432/myapp_dev"
      ]
    },
    "github": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-github"],
      "env": {
        "GITHUB_PERSONAL_ACCESS_TOKEN": "${GITHUB_TOKEN}"
      }
    },
    "slack": {
      "command": "npx",
      "args": ["-y", "@anthropic/mcp-server-slack"],
      "env": {
        "SLACK_BOT_TOKEN": "${SLACK_BOT_TOKEN}"
      }
    }
  }
}
```

### 3.6.4 คำสั่ง `/mcp`

ใช้คำสั่ง `/mcp` ใน Claude Code session เพื่อดูสถานะ MCP servers:

```
/mcp
```

จะแสดงรายการ servers, สถานะการเชื่อมต่อ, และ tools ที่แต่ละ server ให้มา

### 3.6.5 Use Cases สำหรับ MCP

| MCP Server   | Use Case                                                    |
|--------------|-------------------------------------------------------------|
| PostgreSQL   | Query data, inspect schema, ดู indexes, วิเคราะห์ slow queries |
| GitHub       | จัดการ PRs, ดู issues, สร้าง review comments                  |
| Slack        | อ่านข้อความจาก channel, ส่ง notification                      |
| Jira         | ดู tasks, update status, สร้าง subtasks                       |
| Figma        | สร้าง/อ่าน design, แปลง Web URL/screenshot เป็น Figma design   |
| Custom       | เชื่อมต่อ internal tools ของทีม                                |

### 3.6.6 Security Considerations

1. **อย่าใส่ credentials ตรง ๆ ใน .mcp.json**
   - ใช้ environment variables (`${VAR_NAME}`) แทน hardcoded values
   - เก็บ tokens ใน `.env` file ที่อยู่ใน `.gitignore`

2. **ใช้ least-privilege tokens**
   - GitHub token: ให้เฉพาะ scopes ที่จำเป็น (เช่น `repo`, `read:org`)
   - Slack token: ให้เฉพาะ channels ที่ต้องใช้งาน
   - Database: ใช้ read-only user สำหรับ production queries

3. **แยก environment ชัดเจน**
   - `.mcp.json` สำหรับ development
   - ไม่ควรเชื่อมต่อ production services จาก Claude Code โดยตรง

4. **ใส่ .mcp.json ใน .gitignore ถ้ามี secrets**
   - หรือใช้ template file (`.mcp.json.example`) ที่ commit ได้

> **คำเตือน:** อย่าใส่ production database credentials ใน `.mcp.json`
> ใช้เฉพาะ development/staging environment เท่านั้น

---

## 3.7 Permission System ขั้นสูง

ระบบสิทธิ์ (Permission System) ของ Claude Code ควบคุมว่าเอเจนต์สามารถใช้
tools ใดได้บ้าง มี 5 โหมดหลักในการจัดการสิทธิ์

### 3.7.1 โหมดทั้ง 5

| #  | โหมด                      | คำอธิบาย                                         |
|----|---------------------------|--------------------------------------------------|
| 1  | **Default**               | ถามก่อนทุก tool call ที่เป็น write operation      |
| 2  | **Approve Once**          | อนุมัติ tool เดียว ครั้งเดียว                      |
| 3  | **Approve for Session**   | อนุมัติ tool เดียว ตลอด session                   |
| 4  | **Allow All for Session** | อนุมัติทุก tool ตลอด session                      |
| 5  | **Deny**                  | ปฏิเสธ tool call                                 |

### 3.7.2 การตั้งค่า Permissions ใน settings.json

```json
{
  "permissions": {
    "allow": [
      "Read",
      "Glob",
      "Grep",
      "Edit(**/*.go)",
      "Write(.claude/**)",
      "Bash(go test*)",
      "Bash(go build*)"
    ],
    "deny": [
      "Bash(rm -rf*)",
      "Bash(git push --force*)",
      "Bash(curl*)",
      "Write(.env*)"
    ]
  }
}
```

### 3.7.3 Pattern Matching สำหรับ Permissions

`allow` และ `deny` รองรับ glob patterns:

```
"Read"                     → อนุญาตอ่านทุกไฟล์
"Edit(**/*.go)"            → อนุญาตแก้ไขเฉพาะ .go files
"Write(.claude/**)"        → อนุญาตเขียนเฉพาะใน .claude/ directory
"Bash(go test*)"           → อนุญาต go test, go test ./..., go test -v, etc.
"Bash(npm run*)"           → อนุญาต npm run test, npm run build, etc.
"Bash(rm -rf*)"            → ปฏิเสธ rm -rf ทุกรูปแบบ
"Bash(git push --force*)"  → ปฏิเสธ git push --force ทุกรูปแบบ
```

> **หมายเหตุ:** `deny` มีความสำคัญสูงกว่า `allow` เสมอ — ถ้า command match ทั้ง
> allow และ deny จะถูก deny

### 3.7.4 Permission Levels

Permissions สามารถตั้งค่าได้ 3 ระดับ (merge กันตามลำดับ):

| ระดับ   | ไฟล์                          | ขอบเขต                  |
|---------|-------------------------------|-------------------------|
| Project | `.claude/settings.json`       | เฉพาะโปรเจกต์นี้         |
| Local   | `.claude/settings.local.json` | เฉพาะเครื่องนี้ (gitignore)|
| User    | `~/.claude/settings.json`     | ทุกโปรเจกต์ของ user นี้  |

**การ merge:** settings ทั้ง 3 ระดับจะถูก merge โดย deny takes precedence
ถ้ามี deny ที่ระดับใดระดับหนึ่ง จะถูก deny เสมอ ไม่ว่าระดับอื่นจะ allow หรือไม่

### 3.7.5 ตัวอย่าง Settings แยกตามระดับ

**User-level** (`~/.claude/settings.json`) — safety rules ทั่วไป:

```json
{
  "permissions": {
    "deny": [
      "Bash(rm -rf /)",
      "Bash(rm -rf ~)",
      "Bash(*sudo*)",
      "Bash(git push --force*)",
      "Bash(git push -f*)",
      "Bash(git reset --hard*)",
      "Bash(git clean -f*)",
      "Bash(chmod 777*)"
    ]
  }
}
```

**Project-level** (`.claude/settings.json`) — allow patterns เฉพาะโปรเจกต์:

```json
{
  "permissions": {
    "allow": [
      "Read",
      "Glob",
      "Grep",
      "Edit(**/*.go)",
      "Write(.claude/**)",
      "Bash(go test*)",
      "Bash(go build*)",
      "Bash(go vet*)",
      "Bash(make*)",
      "Bash(git status*)",
      "Bash(git diff*)",
      "Bash(git log*)",
      "Bash(git add*)",
      "Bash(git commit*)"
    ],
    "deny": [
      "Bash(DROP TABLE*)",
      "Bash(DROP DATABASE*)",
      "Write(.env*)"
    ]
  }
}
```

### 3.7.6 Best Practices สำหรับ Permission

1. **เริ่มจาก restrictive แล้วค่อยเปิด**
   - เริ่มด้วย Default mode + deny patterns ที่ครบถ้วน
   - ค่อย ๆ เพิ่ม allow patterns ตามที่ต้องใช้จริง

2. **แยก settings ตาม scope**
   - User-level: deny patterns สำหรับ safety
   - Project-level: allow patterns เฉพาะโปรเจกต์

3. **อย่าลืมตั้ง deny สำหรับ destructive commands**
   - `rm -rf`, `git push --force`, `git reset --hard`
   - `DROP TABLE`, `DROP DATABASE`
   - `sudo`, `chmod 777`

> **คำเตือน:** ถ้าไม่ตั้ง deny patterns ไว้ Claude Code อาจรัน destructive commands
> ได้ถ้าผู้ใช้กด approve โดยไม่ตั้งใจ deny patterns ช่วยเป็น safety net

---

## 3.8 Systems Thinking

Claude Code มีความสามารถในการวิเคราะห์ระบบแบบองค์รวม (Systems Thinking)
ช่วยให้เข้าใจ codebase ที่ซับซ้อนได้ดีขึ้น

### 3.8.1 Data Flow Analysis

ให้ Claude Code ติดตาม data flow จาก API ถึง database:

```
พรอมต์: "Trace data flow ของ order creation ตั้งแต่
HTTP request เข้ามาจนถึงบันทึกลง database
แสดง flow ทุกขั้นตอน รวมถึง validation, transformation,
และ side effects ที่เกิดขึ้น"
```

ตัวอย่าง output ที่คาดหวัง:

```
HTTP POST /api/v1/orders
  → Chi Router (cmd/api/routes.go)
    → Auth Middleware (internal/middleware/auth.go)
      → OrderHandler.Create (internal/handler/order.go)
        → Input Validation (validateCreateInput)
        → OrderService.Create (internal/service/order.go)
          → Check inventory
          → Calculate pricing
          → OrderRepository.Create (internal/repository/order.go)
            → BEGIN transaction
            → INSERT INTO orders (...)
            → INSERT INTO order_items (...)
            → COMMIT
          → Publish event: order.created
        → Return OrderResponse
      → HTTP 201 Created + JSON body
```

### 3.8.2 Dependency Mapping

ใช้ Claude Code วิเคราะห์ dependencies ระหว่าง packages:

```
พรอมต์: "วิเคราะห์ dependency graph ของ internal/service/ package
แต่ละ service พึ่งพา (depend on) อะไรบ้าง
มี circular dependencies หรือไม่"
```

ตัวอย่าง output:

```
OrderService
  ├── OrderRepository (interface)
  ├── InventoryService (interface)
  │   └── InventoryRepository (interface)
  ├── PricingService (interface)
  │   ├── DiscountRepository (interface)
  │   └── TaxService (interface)
  ├── EventPublisher (interface)
  └── Logger (*slog.Logger)
```

### 3.8.3 Impact Analysis

ก่อนแก้ไข ให้ Claude Code วิเคราะห์ว่าจะกระทบอะไรบ้าง:

```
พรอมต์: "ถ้าฉันเปลี่ยน User struct เพิ่ม field 'department_id'
จะกระทบอะไรบ้าง แสดงทุกไฟล์ที่ต้องแก้ไข
แยกเป็น direct impact กับ indirect impact"
```

ตัวอย่าง output:

```
## Impact Analysis: เพิ่ม department_id ใน User struct

### Direct Impact (ต้องแก้ไข)
1. internal/model/user.go — เพิ่ม field ใน struct
2. internal/repository/user.go — แก้ SQL queries
3. queries/user.sql — เพิ่ม column ใน sqlc queries
4. migrations/ — สร้าง migration เพิ่ม column
5. internal/handler/user.go — แก้ request/response DTOs

### Indirect Impact (อาจต้องแก้ไข)
6. internal/handler/user_test.go — update test cases
7. internal/service/user_test.go — update test cases
8. frontend/src/types/user.ts — update TypeScript types
```

### 3.8.4 ตัวอย่าง Prompts สำหรับแต่ละ Analysis Type

**Architecture Overview:**

```
"อธิบาย architecture ของโปรเจกต์นี้ในภาพรวม
ใช้ patterns อะไร (layered, hexagonal, etc.)
แต่ละ layer รับผิดชอบอะไร"
```

**Error Flow Analysis:**

```
"Trace error handling flow เมื่อ database connection fail
ตั้งแต่ repository layer ขึ้นมาจน response ถึง client
error ถูก wrap, log, transform อย่างไรในแต่ละ layer"
```

**Performance Bottleneck Analysis:**

```
"วิเคราะห์ potential performance bottlenecks ใน order listing endpoint
ดู SQL queries, N+1 problems, missing indexes, unnecessary data loading"
```

**Security Surface Analysis:**

```
"วิเคราะห์ attack surface ของ API ทั้งหมด
- endpoints ไหนไม่มี authentication
- input validation ครบหรือไม่
- มี SQL injection vectors ไหม
- sensitive data ถูก expose ใน response หรือไม่"
```

> **เคล็ดลับ:** Systems thinking prompts ที่ดีที่สุดคือแบบที่ระบุ scope ชัดเจน
> และบอก output format ที่ต้องการ เช่น "แสดงเป็น tree", "สรุปเป็นตาราง",
> "เรียงตามลำดับ impact มากไปน้อย"

---

## 3.9 คำสั่งเพิ่มเติม

### 3.9.1 /rewind — ย้อนกลับ Conversation

คำสั่ง `/rewind` ย้อน conversation กลับไปจุดก่อนหน้า เหมือน git reset สำหรับ
Claude Code session ใช้เมื่อ:

- Claude Code ไปผิดทาง (wrong approach)
- อยากลองวิธีอื่นโดยไม่เริ่มใหม่ทั้งหมด
- ทำพลาดและอยากย้อนกลับ

**วิธีใช้:**

```
/rewind
```

Claude Code จะแสดง checkpoints ที่สามารถย้อนกลับไปได้ เลือก checkpoint
ที่ต้องการแล้ว conversation จะถูก rewind กลับไปจุดนั้น

**Workflow ที่แนะนำ:**

```
┌──────────────────────────────────────────────────────────┐
│  1. สั่งงาน Claude Code                                  │
│  2. ดูผลลัพธ์ — ไม่ใช่สิ่งที่ต้องการ                       │
│  3. /rewind → ย้อนกลับไปก่อน step ที่ผิดพลาด              │
│  4. สั่งใหม่ด้วยพรอมต์ที่ชัดเจนขึ้น                        │
└──────────────────────────────────────────────────────────┘
```

**ตัวอย่างสถานการณ์:**

```
User: "refactor UserService ให้ใช้ repository pattern"

Claude Code: (ทำการ refactor แบบ Active Record pattern แทน)

User: /rewind

(ย้อนกลับไปก่อน refactor)

User: "refactor UserService ให้ใช้ repository pattern
       โดย repository เป็น interface ที่ inject ผ่าน constructor
       ตาม dependency injection pattern ของโปรเจกต์"
```

> **หมายเหตุ:** `/rewind` ย้อนเฉพาะ conversation state เท่านั้น
> ไม่ได้ย้อนการเปลี่ยนแปลงไฟล์ที่ทำไปแล้ว ถ้า Claude Code แก้ไฟล์ไปแล้ว
> ต้อง undo ด้วย git เอง:

```bash
# ย้อนไฟล์ที่ถูกแก้กลับเป็น version ล่าสุดใน git
git checkout -- path/to/modified/file.go

# หรือ ย้อนทั้งหมด (ระวัง — จะลบทุก uncommitted changes)
git checkout -- .
```

### 3.9.2 /sandbox — เปิดโหมดแซนด์บ็อกซ์

คำสั่ง `/sandbox` เปิดโหมดแซนด์บ็อกซ์ (Sandbox) ที่จำกัดการเข้าถึง filesystem
ปลอดภัยสำหรับทดลอง commands หรือ approach ใหม่ ๆ

**วิธีใช้:**

```
/sandbox
```

เมื่อเข้า sandbox mode:

- การแก้ไขไฟล์ทั้งหมดจะเกิดขึ้นใน isolated environment
- ไฟล์ต้นฉบับไม่ถูกกระทบ
- เหมาะสำหรับทดลอง approach ที่ไม่แน่ใจ

**Use Cases:**

```
# ทดลอง refactoring approach
/sandbox
"ลองย้าย business logic จาก handler layer
 ลงมาที่ service layer ดูว่าโค้ดจะหน้าตาเป็นอย่างไร"

# ทดลอง architecture change
/sandbox
"ลองเปลี่ยนจาก synchronous calls เป็น event-driven
 สำหรับ notification module"

# ทดสอบ impact ก่อนทำจริง
/sandbox
"ลองเปลี่ยน User struct ให้ใช้ embedded struct
 ดูว่ามี compile error ตรงไหนบ้าง"
```

### 3.9.3 การใช้ /rewind กับ /sandbox ร่วมกัน

Workflow ที่ทรงพลังคือใช้ทั้ง 2 คำสั่งร่วมกัน:

```
1. /sandbox → ทดลองวิธี A
2. ถ้าไม่ดี → /rewind ใน sandbox
3. ทดลองวิธี B
4. ถ้าดี → apply ลงจริง
5. ถ้าทั้ง A และ B ไม่ดี → ออกจาก sandbox โดยไม่มีอะไรเปลี่ยน
```

> **เคล็ดลับ:** ใช้ `/sandbox` ทุกครั้งที่จะทดลองอะไรที่ไม่แน่ใจ
> ดีกว่าต้องมานั่ง `git checkout` ทีหลัง

---

## Cross-references

เอกสารที่เกี่ยวข้องกับบทนี้:

| บท   | เอกสาร                                                              | เนื้อหาที่เกี่ยวข้อง                      |
|------|---------------------------------------------------------------------|------------------------------------------|
| บทที่ 1 | [บทนำและพื้นฐาน](./01_intro.md)                                    | การติดตั้ง, ตั้งค่าเบื้องต้น, คำสั่งพื้นฐาน  |
| บทที่ 2 | [Agentic Mastery](./02_agentic_mastery.md)                        | Workflow หลัก, Prompt Engineering, CLAUDE.md |
| บทที่ 4 | [Full-Stack Collaboration](./04_fullstack_collaboration.md)       | การทำงานร่วมกันข้าม stack                 |
| บทที่ 5 | [Specification-Driven Development](./05_spec_driven_dev.md)       | การพัฒนาจากสเปค (Specification)           |
| บทที่ 6 | [Architecture & Design](./06_architecture_design.md)              | การออกแบบระบบ, design patterns            |
| บทที่ 7 | [QA, Security, and Review](./07_qa_security_review.md)            | ความปลอดภัย, code review, testing         |
| บทที่ 8 | [บทบาทและความรับผิดชอบ](./08_roles_responsibilities.md)            | บทบาทของทีม, ขอบเขตความรับผิดชอบ          |
| บทที่ 9 | [Cheat Sheet](./09_cheat_sheet.md)                                | สรุปคำสั่ง, quick reference               |

### การอ้างอิงข้ามบทที่สำคัญ

- **Subagents + Full-Stack:** ดูบทที่ 4 สำหรับการใช้ subagents ทำงานข้าม frontend/backend
- **Skills + Spec-Driven:** ดูบทที่ 5 สำหรับ skills ที่สร้างจากสเปค
- **Hooks + QA:** ดูบทที่ 7 สำหรับ hooks ที่ใช้ใน quality assurance workflow
- **MCP + Security:** ดูบทที่ 7 สำหรับแนวทางจัดการ secrets ใน MCP configuration
- **Permissions + Security:** ดูบทที่ 7 สำหรับ permission policies ระดับองค์กร
- **Plan Mode + Agentic Mastery:** ดูบทที่ 2 สำหรับ basic workflow ก่อนใช้ Plan Mode
- **Worktrees + Agentic Mastery:** ดูบทที่ 2 สำหรับ Git workflow พื้นฐาน
- **Architecture + Systems Thinking:** ดูบทที่ 6 สำหรับ design patterns ที่ใช้ร่วมกับ analysis

---

> **สรุปบทที่ 3:** บทนี้ครอบคลุมฟีเจอร์ขั้นสูงของ Claude Code ที่ช่วยเพิ่มประสิทธิภาพ
> การทำงาน ตั้งแต่ซับเอเจนต์สำหรับงานขนาน สกิลสำหรับ reusable workflows
> Plan Mode สำหรับวางแผน เวิร์คทรีสำหรับทำงานหลาย branch ฮุคสำหรับ automation
> MCP สำหรับเชื่อมต่อระบบภายนอก ระบบสิทธิ์สำหรับความปลอดภัย
> Systems Thinking สำหรับวิเคราะห์ระบบ และคำสั่ง `/rewind` + `/sandbox`
> สำหรับทดลองอย่างปลอดภัย แนะนำให้เริ่มจากฟีเจอร์ที่ใช้บ่อยที่สุด
> (Plan Mode, Hooks) แล้วค่อยขยายไปฟีเจอร์อื่น ๆ ตามความต้องการ
