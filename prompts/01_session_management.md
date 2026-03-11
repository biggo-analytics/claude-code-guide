# Session Management — เริ่มงาน, ทำต่อ, ปิดงาน, ส่งต่อ

> Prompt สำหรับจัดการ session การทำงานกับ Claude Code ตั้งแต่เปิดเครื่องจนปิดงาน
> ใช้ได้กับทุก role — ครอบคลุมทั้งการทำงานคนเดียวและทำงานเป็นทีม

---

## สารบัญ

- [1.1 เริ่มงานใหม่ (Start of Day / New Session)](#11-เริ่มงานใหม่-start-of-day--new-session)
- [1.2 ทำงานต่อจากเดิม (Resume)](#12-ทำงานต่อจากเดิม-resume)
- [1.3 ระหว่างทำงาน (During Work)](#13-ระหว่างทำงาน-during-work)
- [1.4 ปิดงาน (End of Day / End of Task)](#14-ปิดงาน-end-of-day--end-of-task)
- [1.5 ส่งต่องาน (Handoff)](#15-ส่งต่องาน-handoff)

---

## 1.1 เริ่มงานใหม่ (Start of Day / New Session)

**SM-01.** ตรวจสถานะโปรเจกต์
> ทำก่อนเริ่มงานทุกครั้ง เพื่อให้ Claude เข้าใจสถานะปัจจุบัน

```
อ่าน DEVLOG.md และ git log --oneline -20 แล้วสรุป:
1. งานล่าสุดที่ทำเสร็จ
2. งานที่ค้างอยู่ (ถ้ามี)
3. ไฟล์ที่มี uncommitted changes
4. branch ปัจจุบันคือ branch อะไร
```

💡 ใช้คู่กับ `claude -c` (ต่อ session ล่าสุด) หรือเปิด session ใหม่

---

**SM-02.** โหลด context จาก memory
> เริ่ม session ใหม่ เพื่อให้ Claude มี context เท่ากับเมื่อวาน

```
อ่าน CLAUDE.md, docs/specs/ (ถ้ามี), และ DEVLOG.md
แล้วสรุปสิ่งที่ต้องจำ:
- architecture decisions
- naming conventions
- งานที่ค้าง
- ข้อควรระวังหรือ known issues
```

---

**SM-03.** Quick health check
> ตรวจสอบว่าโปรเจกต์ยังทำงานได้ปกติก่อนเริ่มงาน

```
ช่วยตรวจสอบสุขภาพโปรเจกต์:
1. รัน [test command] — ผ่านทุก test ไหม?
2. รัน [lint command] — มี lint errors ไหม?
3. รัน [build command] — build ผ่านไหม?
สรุปผลเป็น checklist: ✅ ผ่าน / ❌ มีปัญหา
```

💡 แทนที่ `[test command]` เช่น `go test ./...`, `npm test`, `pytest`

---

**SM-04.** ตรวจสอบ PR/MR ที่รอ review
> ดู pending items ก่อนเริ่มงานใหม่

```
ตรวจสอบ:
1. git stash list — มี stash ค้างไหม?
2. git branch -a — มี branch ที่กำลังทำค้างไหม?
3. ดู PR/MR ที่เปิดอยู่ (ถ้ามี)
สรุปว่ามี pending items อะไรบ้างที่ต้องจัดการ
```

---

**SM-05.** เตรียม standup update
> สร้าง standup summary จาก git history

```
ดู git log --since="yesterday" --author="[ชื่อ]" และ DEVLOG.md
แล้วสรุปในรูปแบบ standup:
- Yesterday: [สิ่งที่ทำเมื่อวาน]
- Today: [แผนวันนี้]
- Blockers: [สิ่งที่ติดขัด]
เขียนให้กระชับ ไม่เกิน 5 บรรทัด
```

---

**SM-06.** เริ่มงานกับโปรเจกต์ใหม่ (ครั้งแรก)
> เปิด Claude Code กับ codebase ที่ไม่เคยเห็นมาก่อน

```
วิเคราะห์ codebase นี้แล้วสรุป:
1. Tech stack + versions (จาก config files)
2. Project structure (key directories)
3. Architecture pattern (layers, modules)
4. ขั้นตอน build/test/run (จาก Makefile, package.json, etc.)
5. Entry points ของ application
6. ไฟล์สำคัญที่ต้องรู้
```

💡 ใช้ตอน onboard โปรเจกต์ใหม่ หรือโปรเจกต์ที่ไม่คุ้นเคย

---

**SM-07.** เริ่มงาน feature ใหม่
> เตรียมตัวก่อนเริ่ม implement feature

```
กำลังจะเริ่มทำ [feature] ช่วยเตรียมก่อน:
1. อ่าน spec ใน [spec path] (ถ้ามี) สรุปสิ่งสำคัญ
2. ดู codebase — มี code ที่เกี่ยวข้องหรือ pattern คล้ายกันอยู่แล้วไหม?
3. ระบุไฟล์ที่จะต้องแก้ไข/สร้างใหม่
4. ระบุ dependencies กับ features อื่น
5. สรุปเป็น checklist ขั้นตอนที่ต้องทำ
```

---

**SM-08.** เริ่มงาน bug fix
> เตรียม context ก่อนเริ่มแก้ bug

```
มี bug report: [คำอธิบาย bug]
ก่อนเริ่มแก้ ช่วย:
1. หาไฟล์ที่เกี่ยวข้อง
2. ทำความเข้าใจ flow ที่เกี่ยวข้อง
3. ดูว่ามี test ที่ cover จุดนี้อยู่แล้วไหม
4. ตั้ง hypothesis สาเหตุที่น่าจะเป็นไปได้
```

---

## 1.2 ทำงานต่อจากเดิม (Resume)

**SM-09.** Resume จาก DEVLOG
> ทำงานต่อจาก session ก่อนหน้า

```
ดู DEVLOG.md section ล่าสุด แล้วทำงานต่อจากจุดที่ค้างไว้
ถ้ามี TODO items ใน DEVLOG ให้เริ่มจากข้อแรกที่ยังไม่เสร็จ
สรุปก่อนว่าจะทำอะไรต่อ แล้วค่อยเริ่ม
```

💡 ใช้กับ `claude -c` (continue last session) หรือ `claude --resume` (เลือก session)

---

**SM-10.** Resume จาก branch/PR ที่ค้าง
> กลับมาทำ feature ที่ทำไว้ครึ่งหนึ่ง

```
กำลังทำ [feature] ใน branch [branch name]
ดู git log และ git diff เทียบกับ main แล้วสรุป:
1. สิ่งที่ทำไปแล้ว (commits ที่มี)
2. สิ่งที่ยังไม่เสร็จ (จาก uncommitted changes + TODO in code)
3. ขั้นตอนถัดไปที่ควรทำ
```

---

**SM-11.** Resume หลังลาหยุดนาน
> กลับมาทำงานหลังจากหายไปหลายวัน/สัปดาห์

```
ห่างจากโปรเจกต์นี้มา [N] วัน ช่วยสรุป:
1. อ่าน git log --since="[วันที่ออกไป]" — มีอะไรเปลี่ยนบ้าง?
2. อ่าน DEVLOG.md — มี entries ใหม่อะไร?
3. ดู CLAUDE.md — มี rules/conventions ใหม่ไหม?
4. ดู branches ที่ค้าง — ยังมีงานเดิมที่ต้องทำต่อไหม?
5. สรุปสิ่งสำคัญที่ต้องรู้ก่อนเริ่มงาน
```

---

**SM-12.** รับงานต่อจากคนอื่น
> เพื่อนส่งต่องานให้ ต้องเข้าใจ context

```
รับงานต่อจาก [ชื่อเพื่อน] ใน branch [branch name]
ช่วย:
1. ดู commits ทั้งหมดใน branch นี้ — เล่าว่าทำอะไรไปแล้ว
2. ดู uncommitted changes (ถ้ามี)
3. อ่าน PR description / DEVLOG entries ที่เกี่ยวข้อง
4. ระบุสิ่งที่ยังเหลือ
5. ระบุ gotchas หรือข้อควรระวัง
```

---

**SM-13.** สร้าง context จาก git history (ไม่มี DEVLOG)
> โปรเจกต์ไม่มี DEVLOG ต้องเข้าใจจาก git

```
โปรเจกต์นี้ไม่มี DEVLOG — ช่วยสรุป context จาก:
1. git log --oneline -50 — เรื่องราวของโปรเจกต์
2. git log --all --graph --oneline -30 — branch structure
3. ไฟล์ที่แก้ไขบ่อยที่สุด (git log --pretty=format: --name-only | sort | uniq -c | sort -rg | head -20)
4. สรุปว่าตอนนี้โปรเจกต์อยู่ในสถานะไหน
```

---

**SM-14.** Resume งาน refactoring ที่ทำไว้ครึ่งทาง
> Refactoring มักใช้หลาย session

```
กำลัง refactor [module/component] — ทำไว้ครึ่งทางแล้ว
ดู:
1. commits ที่ทำไป (git log)
2. ไฟล์ที่เปลี่ยนไป (git diff main)
3. TODO/FIXME ใน code ที่ยังค้าง
4. tests ที่ยังไม่ผ่าน
สรุปว่าอยู่ step ไหนของ refactoring plan แล้ว step ถัดไปคืออะไร
```

---

## 1.3 ระหว่างทำงาน (During Work)

**SM-15.** Compact with context preservation
> Session ยาว context เริ่มหมด

```
/compact สรุปสิ่งที่ทำไปแล้วในเซสชันนี้ และเก็บ context สำคัญไว้:
- ไฟล์ที่แก้ไป: [list]
- decisions ที่ตัดสินใจ: [list]
- งานที่เหลือ: [list]
- patterns/conventions ที่ต้องจำ
```

---

**SM-16.** บันทึก decision ลง CLAUDE.md
> ตัดสินใจ convention ใหม่ ต้องบันทึกไว้

```
เพิ่ม rule ใหม่ใน CLAUDE.md:
- [rule description]
- เหตุผล: [rationale]
อย่าลบ rules เดิม ให้เพิ่มต่อท้ายในหมวดที่เหมาะสม
```

💡 ทำทุกครั้งที่ตัดสินใจ convention ใหม่ — Claude จะจำได้ในทุก session

---

**SM-17.** อัปเดต CLAUDE.md จาก codebase จริง
> CLAUDE.md อาจ outdated ไม่ตรงกับ code

```
อ่าน codebase ปัจจุบัน (โครงสร้างไฟล์, naming patterns, dependencies)
แล้วตรวจสอบว่า CLAUDE.md ยังตรงกับ reality หรือไม่:
- ส่วนที่ถูกต้อง ✅
- ส่วนที่ outdated ❌ พร้อมเสนอแก้ไข
- ส่วนที่ขาดหาย ➕ พร้อมเสนอเพิ่ม
```

---

**SM-18.** สลับ task ระหว่างทำงาน
> ต้องเปลี่ยนไปทำอย่างอื่นกลางทาง

```
กำลังทำ [task A] แต่ต้องสลับไปทำ [task B] ก่อน
ช่วย:
1. สรุปสถานะปัจจุบันของ task A (ทำถึงไหน, อะไรเหลือ)
2. บันทึกไว้ใน DEVLOG หรือ comment เพื่อกลับมาทำต่อ
3. Commit/stash work-in-progress ของ task A
4. เตรียม context สำหรับ task B
```

---

**SM-19.** จดบันทึก mid-session
> บันทึกสิ่งสำคัญระหว่างทำงานก่อนลืม

```
บันทึก note ระหว่าง session:
- สิ่งที่ค้นพบ: [finding]
- ปัญหาที่เจอ: [issue]
- วิธีแก้ที่ใช้: [solution]
- ข้อควรจำ: [notes]
เพิ่มใน DEVLOG.md section วันนี้
```

---

**SM-20.** Check-in ก่อน commit ใหญ่
> ตรวจสอบก่อนทำ commit สำคัญ

```
กำลังจะ commit งานชุดนี้ ช่วยตรวจสอบ:
1. ดู git diff --staged — มีอะไรที่ไม่ควร commit ไหม? (.env, debug logs, hardcoded values)
2. Test ผ่านหมดไหม?
3. ไม่มี TODO/FIXME ที่ควรแก้ก่อน commit?
4. Naming ตรงกับ conventions ใน CLAUDE.md?
สรุปเป็น ✅ พร้อม commit / ❌ ต้องแก้ก่อน
```

---

## 1.4 ปิดงาน (End of Day / End of Task)

**SM-21.** เขียน DEVLOG entry
> ทำก่อนปิด session ทุกครั้ง — "จดหมายถึงตัวเองในอนาคต"

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

---

**SM-22.** Self-review ก่อน commit
> จับ issues ก่อนเข้า PR

```
ดู git diff --staged แล้ว review:
1. มี code ที่ไม่ควร commit ไหม? (.env, debug logs, TODO hacks)
2. Naming conventions ตรงกับ CLAUDE.md ไหม?
3. มี test สำหรับ logic ใหม่ไหม?
4. มี error handling ที่ขาดไหม?
5. มี security concerns ไหม? (hardcoded secrets, SQL injection, etc.)
สรุปเป็น checklist: ✅ ผ่าน / ❌ ต้องแก้
```

---

**SM-23.** สรุป session สำหรับทีม
> Copy ไปวางใน Slack / standup meeting

```
สรุปงานที่ทำในเซสชันนี้ในรูปแบบ standup update:
- Done: [สิ่งที่เสร็จ]
- In Progress: [สิ่งที่กำลังทำ]
- Blocked: [สิ่งที่ติดขัด]
- Next: [สิ่งที่จะทำต่อ]
เขียนให้กระชับ ไม่เกิน 5 บรรทัด
```

---

**SM-24.** End-of-sprint summary
> สรุปงานทั้ง sprint

```
สรุปงานของ sprint นี้:
1. อ่าน git log --since="[sprint start date]" สรุป commits ทั้งหมด
2. จัดกลุ่มตาม feature/fix/refactor/test
3. ระบุ features ที่เสร็จ vs ที่ค้าง
4. สรุป technical decisions ที่สำคัญ
5. list known issues / tech debt ที่เพิ่มขึ้น
เขียนในรูปแบบ sprint review presentation
```

---

**SM-25.** Commit message จาก session ทั้งหมด
> รวมงานทั้ง session เป็น commit เดียว

```
ดู git diff --staged แล้วเขียน commit message:
- ใช้ conventional commits format (feat/fix/refactor/docs/test/chore)
- บรรทัดแรก < 72 chars
- Body อธิบายว่าทำไม (why) ไม่ใช่ทำอะไร (what)
- ถ้ามี breaking changes ระบุด้วย
```

---

**SM-26.** ปิดงาน + เตรียมสำหรับ session หน้า
> ครบวงจรก่อนปิดเครื่อง

```
กำลังจะปิดงาน ช่วยทำ:
1. ตรวจว่า uncommitted changes ที่ต้อง commit/stash
2. อัปเดต DEVLOG.md (สิ่งที่ทำ, decisions, next steps)
3. ตรวจว่า tests ยังผ่าน (ไม่ทิ้ง broken state)
4. สร้าง TODO list สำหรับวันพรุ่งนี้
5. ถ้ามีอะไรต้องบอกเพื่อนร่วมทีม ให้เตรียม message
```

---

**SM-27.** ปิด feature branch
> เสร็จ feature แล้ว เตรียม merge

```
Feature [feature name] เสร็จแล้ว ช่วยเตรียมก่อน merge:
1. ดู git diff main..HEAD — สรุปการเปลี่ยนแปลงทั้งหมด
2. ตรวจว่า tests ผ่านหมด
3. ตรวจว่า lint ผ่าน
4. อัปเดต docs/specs ถ้ามีการเปลี่ยน API
5. สร้าง PR description (Summary, Changes, Test Plan)
```

---

## 1.5 ส่งต่องาน (Handoff)

**SM-28.** ส่งต่อให้เพื่อนร่วมทีม
> ส่ง feature/branch ให้คนอื่นทำต่อ

```
สรุปสถานะของ [feature/branch] สำหรับ [ชื่อเพื่อน] ที่จะรับงานต่อ:
1. สิ่งที่ทำเสร็จแล้ว + ไฟล์ที่เกี่ยวข้อง
2. สิ่งที่ยังเหลือ + approach ที่แนะนำ
3. Gotchas / ข้อควรระวัง
4. วิธี test สิ่งที่ทำไปแล้ว
5. Decisions ที่ตัดสินใจไปแล้ว + เหตุผล
เขียนในรูปแบบที่ paste ลง PR description ได้
```

---

**SM-29.** ส่งต่อเพื่อ code review
> เตรียม context ให้ reviewer

```
สร้าง PR description สำหรับ review:
## Summary
- [bullet points สรุปสิ่งที่เปลี่ยน]
## Key Changes
- [ไฟล์สำคัญที่เปลี่ยน + เหตุผล]
## How to Test
- [ขั้นตอน test + expected results]
## Risks / Concerns
- [สิ่งที่ reviewer ควรดูเป็นพิเศษ]
## Related
- [link ไป spec, issue, หรือ PR อื่นที่เกี่ยวข้อง]
```

---

**SM-30.** ส่งต่อให้ on-call / operations
> ส่งต่อ context สำหรับ production support

```
สรุป deployment / operational context:
1. สิ่งที่ deploy ไปล่าสุด — features, changes, migrations
2. จุดที่ต้องระวัง — new components, config changes
3. Rollback plan — ขั้นตอนถ้าต้อง rollback
4. Monitoring — ดู metrics/logs อะไร เพื่อยืนยันว่าทำงานถูกต้อง
5. Contact — ติดต่อใครถ้ามีปัญหา
```

---

**SM-31.** Async handoff ข้าม timezone
> ทีมอยู่คนละ timezone ต้องส่งต่อแบบ async

```
สรุปสำหรับทีม [timezone] ที่จะรับงานต่อ:
## Status
- [สถานะปัจจุบัน]
## What I Did
- [รายการสิ่งที่ทำ + commits]
## Pending / Needs Attention
- [สิ่งที่รอ decision, blocked items]
## Context
- [ข้อมูลสำคัญที่ต้องรู้]
## Action Items
- [ ] [TODO สำหรับคนถัดไป]
เขียนให้ self-contained — คนอ่านต้องเข้าใจได้โดยไม่ต้องถาม
```

---

**SM-32.** สรุปให้ manager / stakeholder (non-technical)
> สรุปงาน technical ให้คนที่ไม่ใช่ dev

```
สรุปงานที่ทำในรูปแบบที่ non-technical person เข้าใจ:
1. สิ่งที่ทำเสร็จ — อธิบายเป็นภาษาคน ไม่ใช้ศัพท์ technical
2. ความคืบหน้า — เทียบกับแผนที่วางไว้ (ahead/on-track/behind)
3. ปัญหาที่พบ — อธิบายผลกระทบ ไม่ใช่ technical detail
4. ต้องการอะไรจาก stakeholder — decisions, approvals, resources
5. แผนถัดไป — timeline + milestones
```

---

**SM-33.** สร้าง CLAUDE.md จากศูนย์
> โปรเจกต์ที่ยังไม่มี CLAUDE.md

```
วิเคราะห์ codebase นี้แล้วสร้าง CLAUDE.md ที่ครอบคลุม:
1. Project Overview — tech stack, purpose
2. Architecture — layers, patterns, key directories
3. Coding Conventions — naming, formatting, imports
4. Commands — build, test, lint, run, migrate
5. Important Files — entry points, configs, key modules
6. Rules — ข้อห้าม, conventions ที่ต้องทำตาม
7. Testing — framework, patterns, how to run
```

💡 ให้ SA review CLAUDE.md ที่สร้าง ก่อน commit

---

**SM-34.** Daily standup prep (automated)
> สร้าง standup notes อัตโนมัติจาก data ที่มี

```
เตรียม standup notes จาก data ที่มี:
1. git log --since="yesterday" --author="[name]" — สิ่งที่ทำเมื่อวาน
2. git status + git stash list — งานที่ค้าง
3. DEVLOG.md entry ล่าสุด — plans/blockers
4. open PRs ที่รอ review
สรุปในรูปแบบ:
✅ Done: ...
🔄 Today: ...
🚫 Blocked: ...
```

---

**SM-35.** สลับระหว่างหลาย projects
> ทำงานหลายโปรเจกต์พร้อมกัน

```
สลับจาก [project A] มา [project B]
ช่วย:
1. ปิดงาน project A — stash/commit changes, note สถานะ
2. โหลด context ของ project B — อ่าน CLAUDE.md, DEVLOG
3. ดูสถานะล่าสุดของ project B
4. สรุปว่าจะทำอะไรต่อใน project B
```

💡 ใช้ `--add-dir` ถ้าต้อง reference code ข้ามโปรเจกต์

---

**SM-36.** เตรียม demo / presentation
> เตรียม demo จากงานที่ทำ

```
เตรียม demo สำหรับ [feature/sprint]:
1. สรุป features ที่พร้อม demo
2. ลำดับ demo flow ที่เหมาะสม (เรื่องราวต่อเนื่อง)
3. จุดที่ต้องระวัง / workarounds
4. backup plan ถ้า feature หลักมีปัญหา
5. talking points สำหรับแต่ละ feature
```

---

**SM-37.** Weekly review / retrospective prep
> เตรียมข้อมูลสำหรับ weekly review

```
สรุปงานสัปดาห์นี้สำหรับ review/retro:
1. git log --since="1 week ago" — สรุป commits ทั้งหมด
2. Features completed vs planned
3. Bugs fixed
4. Technical decisions made (อ้างอิง ADRs ถ้ามี)
5. What went well
6. What could be improved
7. Action items for next week
```

---

**SM-38.** ตรวจสอบก่อนเริ่ม long session
> เตรียมพร้อมก่อนทำงานชิ้นใหญ่

```
กำลังจะเริ่มทำ [large task] ที่อาจใช้เวลาทั้งวัน ช่วยเตรียม:
1. ตรวจว่า tests ผ่านหมด (baseline)
2. สร้าง branch ใหม่ (ถ้ายังไม่มี)
3. อ่าน spec/requirements ที่เกี่ยวข้อง
4. สรุปเป็น step-by-step plan
5. ประมาณจำนวน files ที่ต้องแก้
เริ่มจาก plan ก่อน ยังไม่ต้อง implement
```

💡 ใช้ Plan Mode (Shift+Tab) สำหรับงานซับซ้อน

---

**SM-39.** Recovery จาก session ที่ context หาย
> Claude ลืม context กลาง session (หลัง compact หรือ context window เต็ม)

```
ดูเหมือน context จะหายบางส่วน — ช่วย recover:
1. อ่าน CLAUDE.md ใหม่
2. ดู git diff (uncommitted changes) — นี่คืองานที่กำลังทำ
3. ดู git log -5 — commits ล่าสุด
4. อ่าน DEVLOG.md section ล่าสุด
สรุปว่าเรากำลังทำอะไรอยู่ แล้วทำงานต่อ
```

---

**SM-40.** เตรียมก่อน release
> Checklist ก่อน deploy production

```
เตรียมก่อน release v[X.Y.Z]:
1. ตรวจว่า tests ผ่านทั้งหมด
2. ตรวจว่า lint ผ่าน
3. ดู pending TODO/FIXME ที่สำคัญ
4. ตรวจ migrations ที่ต้อง run
5. ตรวจ config/env changes ที่ต้องอัปเดต
6. สร้าง release notes
7. ตรวจ breaking changes
สรุปเป็น release checklist
```

---

**SM-41.** Post-release monitoring session
> เฝ้าดูหลัง deploy

```
เพิ่ง deploy v[X.Y.Z] ไป ช่วย:
1. ดู code ที่เปลี่ยนไป — จุดไหนต้องเฝ้าดู?
2. สร้าง monitoring checklist:
   - endpoints ที่ต้องตรวจ
   - logs ที่ต้อง watch
   - metrics ที่ต้องดู
3. สร้าง rollback steps (ถ้ามีปัญหา)
```

---

**SM-42.** Multi-session task tracking
> งานที่ใช้หลาย session ต้อง track progress

```
งาน [large feature] ใช้หลาย sessions — ช่วย track:
1. อ่าน DEVLOG entries ที่เกี่ยวกับ [feature] ทั้งหมด
2. สรุป timeline: session ไหนทำอะไร
3. overall progress: ทำไปกี่ % (จาก planned work)
4. remaining items ที่ต้องทำ
5. estimated sessions ที่เหลือ
```

---

**SM-43.** End-of-project wrap-up
> ปิดโปรเจกต์ / ส่งมอบ

```
โปรเจกต์ [name] เสร็จแล้ว สร้าง wrap-up document:
1. Project summary — สิ่งที่ส่งมอบ
2. Architecture overview สุดท้าย
3. Known issues / tech debt ที่ยังมี
4. Maintenance guide — วิธีดูแล, deploy, debug
5. Key decisions log (อ้างอิง ADRs)
6. Lessons learned
7. Contact points สำหรับคำถาม
```

---

**SM-44.** Quick context dump
> สรุป context ปัจจุบันแบบเร็ว

```
สรุป context ปัจจุบันให้ฉัง:
- Branch: ?
- Last commit: ?
- Uncommitted changes: ?
- Current task: ? (จาก DEVLOG/TODO)
- Tests status: ?
เขียนสั้นๆ 5-10 บรรทัด
```

---

**SM-45.** Session planning
> วางแผนงานสำหรับ session นี้

```
วันนี้มีเวลา [N] ชั่วโมง ต้องการทำ:
1. [task 1]
2. [task 2]
3. [task 3]
ช่วยจัดลำดับ — อะไรควรทำก่อน, อะไรอาจตัดออกได้ถ้าเวลาไม่พอ
ประมาณ effort แต่ละ task
```

---

← [README](README.md) | [Developer Workflows →](02_developer_workflows.md)
