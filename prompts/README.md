# Prompt Library — รวม Prompt สำเร็จรูปสำหรับทุกสถานการณ์

> **Generic, copy-paste ready** — ไม่ผูกกับโปรเจกต์ใดโปรเจกต์หนึ่ง
> ปรับ `[placeholder]` แล้วใช้ได้เลยกับทุก tech stack

---

## สารบัญ

| # | ไฟล์ | หัวข้อ | จำนวน Prompt | เหมาะกับ |
|---|------|--------|-------------|---------|
| 01 | [01_session_management.md](01_session_management.md) | Session Management — เริ่มงาน, ทำต่อ, ปิดงาน, ส่งต่อ | ~45 | ทุก Role |
| 02 | [02_developer_workflows.md](02_developer_workflows.md) | Developer Workflows — Implement, Debug, Refactor, Test, Git | ~55 | Dev (FE/BE) |
| 03 | [03_architecture_and_design.md](03_architecture_and_design.md) | Architecture & Design — สถาปัตยกรรม, ADR, Technical Debt | ~32 | SA / Tech Lead |
| 04 | [04_project_and_business.md](04_project_and_business.md) | Project & Business — Spec, Planning, Reporting, Stakeholders | ~38 | PM / BA |
| 05 | [05_quality_and_security.md](05_quality_and_security.md) | Quality & Security — Review, Testing, Security, Performance | ~38 | QA / Security |
| 06 | [06_devops_and_infrastructure.md](06_devops_and_infrastructure.md) | DevOps & Infrastructure — CI/CD, Docker, Monitoring, Incidents | ~33 | DevOps / SRE |
| 07 | [07_team_collaboration.md](07_team_collaboration.md) | Team Collaboration — Onboarding, Knowledge Mgmt, Cross-role | ~37 | ทุก Role |

**รวมทั้งหมด: ~278 prompts**

---

## วิธีใช้

### 1. Placeholder Convention

Prompt ทุกตัวใช้ `[placeholder]` สำหรับส่วนที่ต้องแทนค่า:

| Placeholder | ความหมาย | ตัวอย่าง |
|-------------|----------|---------|
| `[project]` | ชื่อโปรเจกต์ | ShopFast, MyApp |
| `[feature]` | ชื่อ feature | user authentication, shopping cart |
| `[file]` / `[path]` | path ของไฟล์ | `internal/service/order.go` |
| `[module]` | ชื่อ module/package | `auth`, `payment` |
| `[service]` | ชื่อ service | `OrderService`, `UserService` |
| `[endpoint]` | API endpoint | `POST /api/v1/orders` |
| `[error message]` | ข้อความ error | `null pointer exception at line 42` |
| `[spec]` | path ไปยัง spec file | `docs/specs/auth.md` |
| `[branch]` | ชื่อ branch | `feature/user-auth` |
| `[N]` | ตัวเลข | จำนวนบรรทัด, จำนวน records, etc. |

### 2. วิธีเลือก Prompt

1. **ตาม Role** — เลือกไฟล์ที่ตรงกับบทบาท
2. **ตามสถานการณ์** — ดูชื่อ section ในแต่ละไฟล์
3. **ตามช่วงเวลา** — เริ่มงาน → `01`, ทำงาน → `02-06`, ปิดงาน → `01`
4. **ข้าม Role** — ไฟล์ `07` รวม prompt สำหรับทำงานร่วมกันระหว่าง role

### 3. Format ของแต่ละ Prompt

```
**ID.** ชื่อ Prompt
> คำอธิบายสั้นๆ — ใช้เมื่อไหร่/ทำไม
‎```
ตัว prompt ที่ copy-paste ได้
‎```
💡 Tips (ถ้ามี)
```

---

## เกี่ยวข้องกับเอกสารอื่น

| เอกสาร | ความสัมพันธ์ |
|--------|-------------|
| [examples/07_prompt_catalog.md](../examples/07_prompt_catalog.md) | Prompt เฉพาะโปรเจกต์ ShopFast — ดูเป็นตัวอย่างการใช้งานจริงในบริบทโปรเจกต์ |
| [09_cheat_sheet.md](../09_cheat_sheet.md) | Quick reference คำสั่ง Claude Code + prompt patterns สั้นๆ |
| [08_roles_responsibilities.md](../08_roles_responsibilities.md) | บทบาทและ permission settings สำหรับแต่ละ role |
| [02_agentic_mastery.md](../02_agentic_mastery.md) | เทคนิคเขียน prompt ที่ดี (CTCO pattern) |

---

> **Tip:** Prompt ในโฟลเดอร์นี้ออกแบบให้ **stack-agnostic** — ใช้ได้กับทั้ง Go, Python, Node.js, Java หรือ stack อื่นๆ
> ถ้าต้องการตัวอย่างเจาะจง Go/Fiber + React/TS ดูที่ [Prompt Catalog](../examples/07_prompt_catalog.md)
