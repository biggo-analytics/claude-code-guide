# Team Collaboration — Onboarding, Knowledge Management, Cross-role

> Prompt สำหรับทุก Role — ครอบคลุมการ onboard คนใหม่, จัดการ knowledge, ทำงานข้าม role
> ใช้ได้กับทุก tech stack โดยแทนค่า `[placeholder]`

---

## สารบัญ

- [7.1 Onboarding (เข้าใจโปรเจกต์ใหม่)](#71-onboarding-เข้าใจโปรเจกต์ใหม่)
- [7.2 First Tasks (งานแรก)](#72-first-tasks-งานแรก)
- [7.3 Knowledge Management](#73-knowledge-management)
- [7.4 Cross-role Collaboration](#74-cross-role-collaboration)
- [7.5 Learning Claude Code](#75-learning-claude-code)

---

## 7.1 Onboarding (เข้าใจโปรเจกต์ใหม่)

**TEAM-01.** ทำความเข้าใจ architecture ภาพรวม
> สิ่งแรกที่ควรทำเมื่อเข้าโปรเจกต์ใหม่

```
ฉันเป็นนักพัฒนาใหม่ในโปรเจกต์นี้
ช่วยอธิบาย architecture ภาพรวม:
1. โปรเจกต์นี้ทำอะไร? (purpose)
2. Tech stack อะไร? (ภาษา, frameworks, databases)
3. โครงสร้าง directory เป็นยังไง? (อธิบายแต่ละ folder)
4. Architecture style? (monolith/microservices/modular)
5. ไฟล์สำคัญที่ต้องอ่านก่อน (top 10)
6. วาด ASCII diagram ของ architecture
```

---

**TEAM-02.** Trace request flow
> เข้าใจว่า request ไหลอย่างไรในระบบ

```
ช่วย trace request flow ของ [endpoint / feature]:
เริ่มจาก HTTP request → ผ่าน layers ทุก layer → ถึง database/response

แสดง:
1. Entry point (route/handler)
2. Middleware chain
3. Business logic (service layer)
4. Data access (repository/model)
5. External calls (ถ้ามี)
6. Response construction

วาดเป็น sequence diagram (ASCII) พร้อมบอกชื่อไฟล์แต่ละ step
```

---

**TEAM-03.** เข้าใจ business rules จาก code
> อ่าน code เข้าใจ business logic

```
อ่าน code ใน [file/module] แล้วอธิบาย business rules:
1. Feature นี้ทำอะไร (ในภาษาที่ non-technical เข้าใจ)
2. Business rules / เงื่อนไข:
   - กฎอะไรบ้างที่ enforce ใน code
   - edge cases ที่ handle
   - validations ที่ทำ
3. ข้อจำกัด / limitations
4. Dependencies กับส่วนอื่น
```

---

**TEAM-04.** สำรวจ conventions ของโปรเจกต์
> เรียนรู้ว่าโปรเจกต์นี้เขียน code style ยังไง

```
วิเคราะห์ coding conventions ของโปรเจกต์นี้:
1. **Naming:** functions, variables, files ตั้งชื่อแบบไหน?
2. **Structure:** แต่ละ feature/module organize ยังไง?
3. **Error handling:** pattern ที่ใช้ handle errors
4. **Testing:** test files อยู่ไหน? naming? pattern (AAA, table-driven)?
5. **Imports:** จัดลำดับ imports ยังไง?
6. **Comments:** style ของ comments/docs
7. **Git:** commit message format? branch naming?

ดูจาก CLAUDE.md (ถ้ามี) + ตัวอย่างจาก code จริง
ให้ตัวอย่าง code ประกอบแต่ละข้อ
```

---

**TEAM-05.** Map dependencies ของ module
> เข้าใจว่า module ที่จะทำงานด้วยเชื่อมต่อกับอะไร

```
วิเคราะห์ dependency map ของ [module]:
1. **Depends on:** module นี้เรียกใช้อะไรบ้าง?
   - Internal modules
   - External packages
   - Database tables
   - External services

2. **Depended by:** module อื่นไหนเรียกใช้ module นี้?

3. **Data flow:** ข้อมูลไหลเข้า-ออก module นี้อย่างไร?

วาดเป็น dependency diagram (ASCII)
ระบุจุดที่ coupling สูง (อาจเป็นปัญหาตอนแก้ code)
```

---

**TEAM-06.** รวมรายการ "ต้องรู้" สำหรับคนใหม่
> สร้าง reading list

```
สร้าง "Must Know" list สำหรับนักพัฒนาใหม่ในโปรเจกต์นี้:

1. **ไฟล์ที่ต้องอ่านก่อน** (เรียงตามลำดับ)
2. **Concepts ที่ต้องเข้าใจ** (architecture patterns, business domain)
3. **Commands ที่ต้องรู้** (dev setup, test, build, deploy)
4. **Common gotchas** (สิ่งที่มักทำผิด)
5. **Key contacts / resources** (ถ้าดูจาก docs/README)

Format เป็น checklist ที่ tick ได้เมื่อเรียนรู้แล้ว:
- [ ] อ่าน README
- [ ] ...
```

---

## 7.2 First Tasks (งานแรก)

**TEAM-07.** Implement feature แรกตาม pattern
> งานแรกที่ทำให้เข้าใจ codebase

```
ฉันเป็นนักพัฒนาใหม่ ต้องการ implement [feature]:
1. หา module/feature ที่คล้ายกันในโปรเจกต์เป็น reference
2. อธิบาย step-by-step ว่าต้องสร้าง/แก้ไฟล์อะไรบ้าง
3. ในแต่ละ step ให้:
   - แสดง code จาก reference ที่ใช้เป็นตัวอย่าง
   - อธิบายว่าต้องปรับอะไร
   - อธิบาย "ทำไม" ถึงทำแบบนี้
4. เขียน tests ด้วย
5. ตรวจว่าตรงกับ conventions ของโปรเจกต์
```

💡 เหมาะสำหรับ first PR — ได้เรียนรู้ pattern + contribute code จริง

---

**TEAM-08.** แก้ bug แรก
> เริ่มจาก bug เล็กๆ เพื่อเข้าใจ codebase

```
ฉันเป็นนักพัฒนาใหม่ ต้องการแก้ bug นี้:
Bug: [อธิบาย bug]

ช่วยเป็น mentor:
1. อธิบายว่า bug นี้น่าจะอยู่ในส่วนไหนของ code
2. ช่วย trace ไปหา root cause
3. อธิบาย context ของ code ที่เกี่ยวข้อง
4. แนะนำวิธีแก้ (อธิบาย "ทำไม" ด้วย)
5. เขียน test ที่ reproduce bug + verify fix
6. ตรวจว่า fix ไม่ break อย่างอื่น
```

---

**TEAM-09.** เขียน test แรก
> เรียนรู้ test patterns ของโปรเจกต์

```
ฉันเป็นนักพัฒนาใหม่ ต้องการเขียน test สำหรับ [function/module]:
1. หา test file ที่มีอยู่ในโปรเจกต์เป็นตัวอย่าง
2. อธิบาย test pattern ที่โปรเจกต์ใช้:
   - Test runner อะไร?
   - Naming convention?
   - Setup/teardown pattern?
   - Mock strategy?
3. ช่วยเขียน test cases:
   - Happy path
   - Error cases
   - Edge cases
4. อธิบายว่าทำไมถึงเลือก test แต่ละ case
```

---

**TEAM-10.** เพิ่ม documentation สำหรับ feature ที่ไม่มี docs
> contribute แบบที่ไม่ต้องเปลี่ยน code

```
ฉันเป็นนักพัฒนาใหม่ ต้องการเพิ่ม documentation สำหรับ [module/feature]:
1. อ่านและเข้าใจ code ของ [module]
2. สร้าง documentation:
   - Purpose — module นี้ทำอะไร
   - Architecture — ส่วนประกอบหลัก
   - API — public functions/methods + parameters + return values
   - Examples — ตัวอย่างการใช้งาน
   - Configuration — settings ที่ปรับได้
3. เขียนตาม documentation style ที่โปรเจกต์ใช้
```

💡 วิธีนี้ได้เรียนรู้ code + contribute ได้ทันทีแม้ยังไม่คุ้น codebase

---

## 7.3 Knowledge Management

**TEAM-11.** สร้าง Architecture Decision Record (ADR)
> บันทึก decisions สำหรับทีม

```
สร้าง ADR สำหรับ decision:
เรื่อง: [decision topic]
Context: [ทำไมต้อง decide เรื่องนี้]

Template:
# ADR-[N]: [Title]
## Status: [Proposed / Accepted / Deprecated]
## Context
[ปัญหา/สถานการณ์ที่ต้อง decide]
## Options Considered
1. [Option A] — pros, cons
2. [Option B] — pros, cons
3. [Option C] — pros, cons
## Decision
[เลือกอะไร ทำไม]
## Consequences
- ✅ ข้อดี
- ⚠️ ข้อเสีย/trade-offs
- 📋 สิ่งที่ต้องทำตาม
```

---

**TEAM-12.** สร้าง Changelog
> บันทึกการเปลี่ยนแปลงสำหรับทีม

```
สร้าง changelog จาก git history:
ตั้งแต่ [tag/date/commit] ถึง HEAD

จัดกลุ่ม:
### ✨ Features
- [feature description] ([commit hash])

### 🐛 Bug Fixes
- [fix description] ([commit hash])

### ♻️ Refactoring
- [refactor description]

### 📝 Documentation
- [doc changes]

### ⚠️ Breaking Changes
- [breaking change + migration guide]

เขียนให้คนอ่านเข้าใจ (ไม่ใช่ copy commit messages ตรงๆ)
```

---

**TEAM-13.** สร้าง API Documentation จาก code
> generate docs จาก implementation

```
สร้าง API documentation จาก code:
อ่าน handlers/controllers ใน [path]

สำหรับแต่ละ endpoint:
| Field | Detail |
|-------|--------|
| Method | GET/POST/PUT/DELETE |
| Path | /api/v1/... |
| Auth | Required/Optional/None |
| Request | body/params/query |
| Response 200 | success shape |
| Response 4xx | error shape |
| Example | curl command |

จัดกลุ่มตาม resource/module
```

---

**TEAM-14.** สรุป module/feature สำหรับ knowledge sharing
> เตรียม content สำหรับ tech talk / wiki

```
สรุป [module/feature] สำหรับ knowledge sharing:
Audience: [ทีม dev / ทุกคน / คนใหม่]

1. **What:** Feature นี้ทำอะไร (1-2 ประโยค)
2. **Why:** ทำไมถึง design แบบนี้
3. **How:** Architecture + key components (พร้อม diagram)
4. **Code walkthrough:** ไฟล์สำคัญ + flow
5. **Gotchas:** สิ่งที่ต้องระวัง
6. **Future:** แผนที่จะทำต่อ (ถ้ามี)

เขียนเป็น format ที่พร้อมใส่ wiki / Notion / Confluence
```

---

**TEAM-15.** Update CLAUDE.md สำหรับทีม
> ให้ CLAUDE.md เป็น single source of truth

```
Update CLAUDE.md ให้เป็นปัจจุบัน:
1. อ่าน CLAUDE.md ที่มีอยู่
2. สแกน codebase จริงเทียบกับ CLAUDE.md:
   - Directory structure ยังตรงไหม?
   - Commands ยังใช้ได้ไหม?
   - Conventions ยังตรงกับ code จริงไหม?
   - Dependencies ยังถูกต้องไหม?
3. เพิ่มสิ่งที่ขาด:
   - Conventions ใหม่ที่เกิดขึ้น
   - Known issues ปัจจุบัน
   - ข้อตกลงที่ทีม decide ล่าสุด
4. ลบสิ่งที่ outdated
```

---

## 7.4 Cross-role Collaboration

**TEAM-16.** Translate ภาษา technical ↔ business
> สื่อสารข้ามทีม

```
แปลง [technical/business] content เป็นภาษาที่ [audience] เข้าใจ:

Content:
[paste content]

Target audience: [PM / BA / Developer / Stakeholder / Management]

Requirements:
- ใช้คำศัพท์ที่ audience เข้าใจ
- ถ้าเป็น technical → business: เน้น impact, benefit, risk
- ถ้าเป็น business → technical: เน้น requirements, constraints, acceptance criteria
- เพิ่มตัวอย่างที่ช่วยให้เข้าใจ
```

---

**TEAM-17.** สร้าง Sprint review summary
> สรุปงานใน sprint สำหรับทุก role

```
สร้าง sprint review summary:
Sprint: [sprint name/number]
ระยะเวลา: [start date] - [end date]

อ่าน:
- git log ในช่วง sprint
- specs ที่ completed
- PRs ที่ merged

สรุป:
1. **Delivered:** features/fixes ที่เสร็จ (เขียนให้ non-technical เข้าใจ)
2. **In Progress:** งานที่ยังทำอยู่
3. **Demo-ready:** features ที่พร้อม demo
4. **Blockers:** ปัญหาที่ค้าง
5. **Metrics:** PRs merged, commits, test coverage change
```

---

**TEAM-18.** Sync types/interfaces ระหว่าง frontend-backend
> ให้ FE/BE ใช้ types เดียวกัน

```
Sync types ระหว่าง frontend และ backend:
1. อ่าน API response types จาก backend: [backend path]
2. อ่าน types/interfaces จาก frontend: [frontend path]
3. เปรียบเทียบ:
   - Field names ตรงกัน?
   - Types ตรงกัน? (e.g., string vs number, nullable vs required)
   - Enum values ตรงกัน?
   - Response format เหมือนกัน?
4. สร้าง/update frontend types ให้ตรงกับ backend
5. แนะนำ strategy สำหรับ keep types in sync ต่อไป
```

---

**TEAM-19.** Prepare handoff document
> เตรียมส่งต่องานให้คนอื่น

```
สร้าง handoff document สำหรับ [feature/project]:
1. **Current state:** สถานะปัจจุบัน (% done, what works, what doesn't)
2. **Architecture:** overview + key decisions ที่ทำไป
3. **Key files:** ไฟล์สำคัญที่ต้องเข้าใจ + อธิบายสั้นๆ
4. **In progress:** งานที่ทำค้าง + approach ที่ใช้
5. **Known issues:** ปัญหาที่รู้แล้วแต่ยังไม่ได้แก้
6. **Gotchas:** สิ่งที่ต้องระวัง (non-obvious things)
7. **Next steps:** สิ่งที่ต้องทำต่อ (เรียงตาม priority)
8. **Environment setup:** ขั้นตอน setup สำหรับคนรับงาน
9. **Contacts:** ถ้ามีคำถาม ถามใคร
```

---

**TEAM-20.** Review spec ร่วมกัน (BA + Dev + QA)
> ตรวจ spec ก่อนเริ่ม implement

```
Review spec ใน [spec path] จากมุมมอง 3 roles:

**Developer perspective:**
- Implement ได้จริงหมือ? มี technical constraint อะไร?
- ต้องเปลี่ยน architecture ไหม?
- ประมาณ effort เท่าไหร่?

**QA perspective:**
- Acceptance criteria ครบและ testable?
- Edge cases ที่ spec ไม่ได้ระบุ?
- Test strategy แนะนำ?

**BA perspective:**
- Requirements ครบตาม business needs?
- User stories สมเหตุสมผล?
- มี gap ระหว่าง business need กับ spec?

สรุปเป็น action items ที่ต้องแก้ใน spec ก่อน implement
```

---

## 7.5 Learning Claude Code

**TEAM-21.** แนะนำ Claude Code สำหรับคนใหม่
> เรียนรู้ features พื้นฐาน

```
ฉันเพิ่งเริ่มใช้ Claude Code
ช่วยแนะนำ features พื้นฐาน:
1. Commands ที่ใช้บ่อย (claude, claude -c, claude -r)
2. Slash commands ที่สำคัญ (/init, /compact, /memory)
3. Permission modes (ask, auto, plan)
4. Plan mode vs normal mode — ใช้เมื่อไหร่?
5. CLAUDE.md — คืออะไร สำคัญยังไง?
6. Tips สำหรับเขียน prompt ที่ได้ผลดี
อธิบายพร้อมตัวอย่างจริง
```

---

**TEAM-22.** ตั้งค่า Claude Code สำหรับโปรเจกต์
> setup ให้พร้อมใช้กับโปรเจกต์

```
ช่วย setup Claude Code สำหรับโปรเจกต์นี้:
1. สร้าง/update CLAUDE.md:
   - Project overview
   - Architecture & conventions
   - Commands (test, lint, build)
   - File structure
   - Known issues

2. สร้าง .claude/settings.json:
   - Allowed commands
   - Denied commands
   - Permission defaults

3. แนะนำ custom slash commands ที่เหมาะกับโปรเจกต์

4. แนะนำ hooks ที่ควรตั้ง (pre-commit checks, etc.)
```

---

**TEAM-23.** Effective prompting patterns
> เขียน prompt ให้ได้ผลดี

```
ช่วยแนะนำ effective prompting patterns สำหรับ Claude Code:

1. **Be specific:** ตัวอย่าง prompt ที่ดี vs ไม่ดี
2. **Give context:** วิธีให้ context ที่เพียงพอ
3. **Iterative approach:** ทำทีละ step vs ทำทีเดียว
4. **Plan first:** ใช้ Plan mode เมื่อไหร่
5. **Reference patterns:** วิธีชี้ไปที่ code ตัวอย่าง
6. **Verify results:** วิธีให้ Claude ตรวจงานตัวเอง

ให้ตัวอย่าง before/after ของ prompt แต่ละแบบ
```

---

**TEAM-24.** สร้าง team conventions สำหรับใช้ Claude Code
> ตั้งข้อตกลงสำหรับทีม

```
ช่วยสร้าง team conventions สำหรับการใช้ Claude Code ร่วมกัน:

1. **CLAUDE.md management:**
   - ใครเป็นคน update?
   - Review process?
   - Sections ที่จำเป็น?

2. **Session management:**
   - DEVLOG format?
   - Handoff protocol?
   - Branch naming?

3. **Code quality:**
   - ต้อง self-review ก่อน commit?
   - Test requirements?
   - PR checklist?

4. **Security:**
   - Permission settings?
   - Commands ที่ห้ามใช้?
   - Sensitive data handling?

สรุปเป็นเอกสาร 1 หน้าที่ทุกคนในทีมอ้างอิงได้
```

---

**TEAM-25.** Guided practice session
> ฝึกใช้ Claude Code แบบมี guide

```
สร้าง guided practice session สำหรับเรียนรู้ Claude Code:
ใช้ codebase ปัจจุบันเป็นสนามฝึก

Exercise 1: อ่านและเข้าใจ code
- ให้ Claude อธิบาย [module]
- ลอง trace request flow

Exercise 2: แก้ bug เล็กๆ
- หา TODO/FIXME ในโปรเจกต์
- ลองแก้ 1 อัน

Exercise 3: เขียน test
- เลือก function ที่ยังไม่มี test
- ให้ Claude ช่วยเขียน

Exercise 4: Refactoring
- หา code smell 1 อัน
- ลอง refactor ทีละ step

Exercise 5: Session management
- ลอง compact + update CLAUDE.md
- เขียน DEVLOG entry

แต่ละ exercise ให้ prompt ตัวอย่าง + expected outcome
```

---

> **ดูเพิ่มเติม:**
> - [01_session_management.md](01_session_management.md) — prompts สำหรับ handoff และ session management
> - [04_project_and_business.md](04_project_and_business.md) — stakeholder communication prompts
> - [examples/07_prompt_catalog.md](../examples/07_prompt_catalog.md) — ตัวอย่าง onboarding prompts เฉพาะโปรเจกต์ ShopFast (N1-N4)
> - [08_roles_responsibilities.md](../08_roles_responsibilities.md) — บทบาทและ permissions สำหรับแต่ละ role
