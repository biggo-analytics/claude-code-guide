# Project & Business — Spec, Planning, Reporting, Stakeholders

> Prompt สำหรับ PM และ BA — ครอบคลุมการเขียน spec, วางแผนโปรเจกต์, ติดตามงาน, สื่อสารกับ stakeholders
> ใช้ได้กับทุกโปรเจกต์ โดยแทนค่า `[placeholder]`

---

## สารบัญ

- [4.1 Specification Writing (BA)](#41-specification-writing-ba)
- [4.2 Project Planning (PM)](#42-project-planning-pm)
- [4.3 Tracking & Reporting (PM)](#43-tracking--reporting-pm)
- [4.4 Stakeholder Communication](#44-stakeholder-communication)

---

## 4.1 Specification Writing (BA)

**BIZ-01.** สร้าง Feature Specification
> เขียน spec ครบ template สำหรับ feature ใหม่

```
สร้าง feature specification สำหรับ [feature name]:
Business requirement: [อธิบาย requirement]

ใช้ template:
1. Objective — ทำไมต้องมี feature นี้
2. User Stories — As a [role], I want [action], So that [benefit]
3. Business Rules — เงื่อนไข/กฎที่ต้องปฏิบัติ
4. API Contract — method, URL, request/response JSON
5. Edge Cases — กรณีพิเศษที่ต้อง handle
6. Acceptance Criteria — เงื่อนไขที่ต้องผ่านเพื่อถือว่าเสร็จ
7. Dependencies — ต้องมีอะไรก่อน

บันทึกที่ docs/specs/[feature-name].md
```

---

**BIZ-02.** สร้าง spec ต่อจากตัวอย่าง
> ใช้ format เดียวกับ spec ที่มีอยู่

```
สร้าง spec สำหรับ [feature name]
อ้างอิง format เดียวกับ [existing spec file path]
Business requirements:
- [requirement 1]
- [requirement 2]
- [requirement 3]
ใช้ structure, depth, และ level of detail เดียวกัน
```

---

**BIZ-03.** สร้าง User Stories
> แปลง requirements เป็น user stories

```
จาก business requirement นี้: [description]
สร้าง user stories:
- ใช้ format: As a [role], I want [action], So that [benefit]
- แยก role ให้ชัด (end user, admin, system)
- แต่ละ story ต้องมี acceptance criteria
- จัด priority: Must Have / Should Have / Nice to Have
- ระบุ story ไหนมี dependency กับ story อื่น
```

---

**BIZ-04.** สร้าง API contract specification
> กำหนด API interface ให้ FE/BE ตรงกัน

```
สร้าง API contract สำหรับ [feature]:
แต่ละ endpoint:
- Method + Path (RESTful)
- Request: body/params with types + validation rules
- Response (success): status code + body structure
- Response (error): error codes + messages
- Authentication: required? what type?
- Examples: request/response ตัวอย่าง

Conventions:
- JSON field naming: [camelCase/snake_case]
- Date format: [ISO 8601]
- Pagination: [cursor/offset + format]
```

---

**BIZ-05.** เขียน acceptance criteria ให้ครบ
> ตรวจว่า criteria ครอบคลุม

```
สร้าง acceptance criteria สำหรับ [feature]:
ใช้ Given-When-Then format:
- Given [precondition]
- When [action]
- Then [expected result]

ครอบคลุม:
1. Happy path — flow ปกติ
2. Validation — input ไม่ถูกต้อง
3. Edge cases — boundary values, empty data
4. Error cases — system failures, timeouts
5. Authorization — different roles
6. Performance — response time expectations
```

---

**BIZ-06.** เขียน edge cases / business rules
> ระบุ edge cases ที่อาจลืม

```
วิเคราะห์ [feature] แล้วหา edge cases ที่ต้อง handle:
1. Boundary values — min/max, empty, zero
2. Concurrent actions — 2 คนทำพร้อมกัน
3. Timing — expired, timeout, out-of-order
4. Data — unicode, very long input, special chars
5. State — invalid transitions, already processed
6. Integration — external service down, slow response
7. Business — ข้อยกเว้นจากกฎปกติ

แต่ละ edge case: scenario, expected behavior, priority
```

---

**BIZ-07.** Spec review และ gap analysis
> ตรวจสอบ spec ว่าครบถ้วน

```
ตรวจสอบ spec ใน [spec file path]:
1. Completeness — ครบทุก section ของ template ไหม?
2. Clarity — มี ambiguity ตรงไหน? ตีความได้หลายแบบ?
3. Consistency — ข้อมูลใน spec ขัดกันเองไหม?
4. Testability — acceptance criteria เขียนแบบ test ได้จริงไหม?
5. Missing scenarios — edge cases ที่ยังไม่ได้ระบุ
6. Dependencies — ระบุ dependencies ครบไหม?
7. API contract — request/response ชัดเจนพอให้ dev implement ไหม?
สรุปเป็น: ✅ พร้อม / 🔧 ต้องแก้ (+ list สิ่งที่ต้องแก้)
```

---

**BIZ-08.** เปรียบเทียบ spec versions
> เข้าใจว่า spec เปลี่ยนไปอย่างไร

```
เปรียบเทียบ spec เวอร์ชันเดิมกับเวอร์ชันใหม่:
- เวอร์ชันเดิม: [path หรือ paste]
- เวอร์ชันใหม่: [path หรือ paste]

สรุป:
1. อะไรเพิ่ม (new requirements)
2. อะไรเปลี่ยน (modified requirements)
3. อะไรลบ (removed requirements)
4. Impact ต่อ code ที่ implement ไปแล้ว
5. ต้องอัปเดตอะไรบ้าง (code, tests, docs)
```

---

**BIZ-09.** แปลง meeting notes เป็น requirements
> เอา notes จากการประชุมมาจัดเป็น spec

```
จาก meeting notes:
[paste meeting notes]

แปลงเป็น structured requirements:
1. Features ที่ตกลงกัน
2. User stories
3. Business rules
4. Acceptance criteria
5. Open questions (ยังไม่ได้ตกลง)
6. Decisions made (ตัดสินใจแล้ว)
7. Action items + owner
```

---

**BIZ-10.** สร้าง data dictionary
> อธิบาย terms และ data fields

```
สร้าง data dictionary สำหรับ [module/feature]:
| Term | Definition | Type | Constraints | Example |
|------|-----------|------|-------------|---------|
| [field] | [meaning] | [type] | [rules] | [value] |

รวม:
1. Entity definitions
2. Field descriptions + types
3. Validation rules
4. Relationships ระหว่าง entities
5. Business terminology ที่ใช้ (glossary)
```

---

## 4.2 Project Planning (PM)

**BIZ-11.** สร้าง project brief
> สรุปโปรเจกต์สำหรับ kickoff

```
อ่าน specs ทั้งหมดใน docs/specs/ และ docs/adr/ (ถ้ามี) แล้วสร้าง project brief:
1. Project Overview — ชื่อ, เป้าหมาย, scope
2. Tech Stack — technology ที่เลือก + เหตุผล
3. Features — list features จาก specs + priority
4. Architecture — สรุปจาก ADRs
5. Implementation Order — ลำดับที่แนะนำ + dependencies
6. Team — roles ที่ต้องมี
7. Risks — ความเสี่ยงที่เห็น
8. Open Questions — สิ่งที่ยังไม่ได้ตัดสินใจ
```

---

**BIZ-12.** Sprint planning
> เตรียม sprint plan จาก backlog

```
ช่วยวางแผน sprint:
Capacity: [N] developer-days
Backlog items:
- [item 1]: [brief description]
- [item 2]: [brief description]
- [item 3]: [brief description]

ช่วย:
1. ประเมิน effort แต่ละ item (S/M/L/XL)
2. ระบุ dependencies ระหว่าง items
3. แนะนำ items ที่ควรเลือกเข้า sprint (ตาม priority + capacity)
4. ลำดับการทำ (order of execution)
5. Risks + mitigation
```

---

**BIZ-13.** Effort estimation จาก specs
> ประมาณ effort จาก spec files

```
อ่าน spec ใน [spec path] แล้วประมาณ effort:
1. จำนวนไฟล์ที่ต้องสร้าง/แก้ (ดูจาก architecture)
2. Complexity ของ business logic (simple/moderate/complex)
3. จำนวน API endpoints
4. จำนวน test cases ที่ต้องเขียน
5. Database changes (migrations, indexes)
6. Integration points กับ modules อื่น

สรุปเป็น T-shirt size (S/M/L/XL) + breakdown by task
⚠️ หมายเหตุ: นี่เป็นการประมาณจาก code complexity เท่านั้น
```

---

**BIZ-14.** Risk assessment
> ระบุความเสี่ยงของโปรเจกต์

```
วิเคราะห์ความเสี่ยงของโปรเจกต์/feature [name]:
สำหรับแต่ละ risk:
| Risk | Likelihood | Impact | Mitigation |
|------|-----------|--------|------------|
| [description] | H/M/L | H/M/L | [plan] |

Categories:
1. Technical — complexity, new technology, integration
2. Resource — availability, skill gaps
3. Schedule — dependencies, external blockers
4. Scope — requirement changes, scope creep
5. Quality — testing gaps, technical debt
```

---

**BIZ-15.** Dependency mapping
> หา dependencies ระหว่าง features

```
อ่าน specs ทั้งหมดใน [specs directory] แล้ว map dependencies:
1. Feature ไหน depend on feature ไหน
2. Shared components ที่หลาย features ใช้
3. Database tables/schemas ที่ share กัน
4. External service dependencies
5. Critical path — ลำดับ implementation ที่สั้นที่สุด
วาดเป็น dependency diagram (text-based)
```

---

**BIZ-16.** Roadmap generation
> สร้าง roadmap จาก features

```
สร้าง roadmap สำหรับ [project/product]:
Features:
- [feature 1]: [priority, complexity]
- [feature 2]: [priority, complexity]
- [feature 3]: [priority, complexity]

จัดลำดับเป็น phases/milestones:
- Phase 1 (MVP): [features ที่ต้องมี]
- Phase 2: [features ถัดไป]
- Phase 3: [nice-to-have features]
ระบุ dependencies ระหว่าง phases + estimated duration
```

---

## 4.3 Tracking & Reporting (PM)

**BIZ-17.** Sprint report
> สรุปสถานะ sprint

```
สรุป progress ของ [project] Sprint [N]:
1. Done: [features/tasks ที่เสร็จ]
   - ดูจาก git log, merged PRs
2. In Progress: [features ที่กำลังทำ]
   - ดูจาก open branches, open PRs
3. Blocked: [features ที่ติด]
4. Carry Over: [features ที่ไม่เสร็จ]
5. Unplanned Work: [bugs, urgent requests]
6. Metrics:
   - Planned vs Completed: [N/M items]
   - Test Coverage: [%]
   - Open Bugs: [N]
```

---

**BIZ-18.** Release checklist
> ตรวจสอบก่อน release

```
อ่าน specs ทั้งหมดแล้วสร้าง release checklist:
| Spec | Implemented | Tests | Docs | Status |
|------|------------|-------|------|--------|
| [name] | ✅/❌ | ✅/❌ | ✅/❌ | Ready/Blocked |

สำหรับแต่ละ spec:
1. Acceptance criteria ผ่านครบไหม?
2. มี test coverage ไหม?
3. API docs อัปเดตไหม?
4. Migration scripts พร้อมไหม?
ระบุ blockers ที่ต้องแก้ก่อน release
```

---

**BIZ-19.** Spec vs implementation coverage
> เทียบ spec กับ code จริง

```
เปรียบเทียบ acceptance criteria จาก [spec path] กับ tests + code:
| Criteria | Has Implementation | Has Test | Gap |
|----------|-------------------|----------|-----|
| [AC-001] | ✅/❌ | ✅/❌ | [description] |

สรุป:
1. Criteria ที่ implement + test ครบ
2. Criteria ที่ implement แต่ไม่มี test
3. Criteria ที่ยังไม่ implement
4. Code ที่ implement แต่ไม่มีใน spec (scope creep?)
```

---

**BIZ-20.** Release notes
> สร้าง release notes จาก git history

```
สร้าง release notes v[X.Y.Z] จาก git log และ specs:
## Release Notes v[X.Y.Z]
### Summary
- [1-2 sentences overview]
### New Features
- [feature 1]: [user-facing description]
### Improvements
- [improvement 1]
### Bug Fixes
- [fix 1]
### API Changes
- [new/changed/removed endpoints]
### Breaking Changes
- [breaking change + migration guide]
### Known Issues
- [issue 1]
### Deployment Notes
- [special deployment steps]
```

---

**BIZ-21.** Cost tracking (Claude Code usage)
> ติดตามค่าใช้จ่าย Claude Code

```
สรุปการใช้งาน Claude Code ในสัปดาห์/sprint นี้:
1. ดูจาก /cost ของ sessions ที่มี
2. แยกตาม task type:
   - Code generation
   - Code review
   - Debugging
   - Documentation
   - Planning
3. Model ที่ใช้ (Sonnet vs Opus vs Haiku)
4. Session ไหนใช้ token มากที่สุด? ทำไม?
5. แนะนำวิธีลดค่าใช้จ่าย
```

---

**BIZ-22.** Technical debt report สำหรับ management
> สรุป tech debt ภาษาคน

```
สร้าง technical debt report สำหรับ management:
1. อ่าน codebase หา: TODO/FIXME, outdated deps, missing tests, code smells
2. จัดลำดับตาม business impact:
   - 🔴 Critical: กระทบ stability / security
   - 🟡 Important: กระทบ velocity / maintenance
   - 🟢 Minor: cosmetic / nice-to-have
3. สำหรับแต่ละ item:
   - อธิบายเป็นภาษาคน (ไม่ใช่ technical jargon)
   - ผลกระทบถ้าไม่แก้
   - ประมาณ effort ที่ต้องใช้แก้
4. แนะนำ items ที่ควรจัดการใน sprint ถัดไป
```

---

**BIZ-23.** Progress visualization
> สร้าง visual summary ของ progress

```
สร้าง progress overview ของ [project/sprint]:
1. Feature progress bar:
   [Feature A] ████████░░ 80%
   [Feature B] ██████░░░░ 60%
   [Feature C] ██░░░░░░░░ 20%
2. Overall metrics:
   - Features: [done/total]
   - Tests: [passing/total]
   - Coverage: [%]
   - Open issues: [N]
3. Burndown: planned vs actual (text-based chart)
```

---

## 4.4 Stakeholder Communication

**BIZ-24.** Non-technical summary
> อธิบาย technical decisions ให้คนไม่ใช่ dev

```
สรุป technical decision เรื่อง [decision] ในภาษาที่ non-technical person เข้าใจ:
1. ปัญหาที่เจอ — อธิบายเป็น business impact
2. ทางเลือกที่พิจารณา — ข้อดี/ข้อเสียในมุม business
3. สิ่งที่ตัดสินใจ — ทำอะไร + ทำไม
4. ผลกระทบ — ต่อ timeline, cost, features
5. ต้องการอะไรจาก stakeholder — approval, resources, decision
ไม่ใช้ศัพท์ technical ยกเว้นจำเป็นจริงๆ (อธิบายประกอบ)
```

---

**BIZ-25.** Executive summary
> สรุปสถานะให้ผู้บริหาร

```
สร้าง executive summary สำหรับ [project]:
## Status: [🟢 On Track / 🟡 At Risk / 🔴 Off Track]
## Summary
- [1-2 sentences สถานะปัจจุบัน]
## Key Achievements
- [bullet points]
## Risks & Issues
- [with mitigation plan]
## Upcoming Milestones
- [milestone 1]: [date] — [status]
## Budget
- [planned vs actual usage]
## Decisions Needed
- [decisions ที่รอจาก management]

เขียนให้อ่านจบใน 2 นาที
```

---

**BIZ-26.** Change request impact analysis
> วิเคราะห์ impact ของ change request

```
มี change request: [อธิบาย change]
วิเคราะห์ impact:
1. Scope: กระทบ features ไหนบ้าง?
2. Code: กี่ files ต้องแก้? complexity?
3. Tests: test ไหนต้องเขียนใหม่/แก้?
4. Timeline: เพิ่มงานกี่วัน?
5. Risk: ความเสี่ยงถ้าทำ change นี้
6. Dependencies: block งานอื่นไหม?
7. Alternative: มีวิธีอื่นที่ impact น้อยกว่าไหม?
สรุปเป็น: ✅ แนะนำทำ / ⚠️ ทำได้แต่มี risk / ❌ ไม่แนะนำ
```

---

**BIZ-27.** Meeting notes → action items
> แปลง meeting notes เป็น action items

```
จาก meeting notes:
[paste notes]

สรุป:
1. Decisions Made:
   - [decision 1] — ตัดสินใจโดย [who]
2. Action Items:
   | # | Action | Owner | Deadline | Priority |
   |---|--------|-------|----------|----------|
   | 1 | [task] | [name] | [date] | H/M/L |
3. Open Questions:
   - [question] — ต้องได้คำตอบจาก [who] ภายใน [when]
4. Follow-up Meeting:
   - [date/topic ถ้ามี]
```

---

**BIZ-28.** Decision log
> บันทึก decisions ที่เกิดขึ้น

```
อัปเดต decision log:
| # | Date | Decision | Rationale | Made By | Status |
|---|------|----------|-----------|---------|--------|
| [N] | [date] | [decision] | [why] | [who] | Active/Superseded |

เพิ่ม entry ใหม่:
- Decision: [อะไรที่ตัดสินใจ]
- Context: [ทำไมต้องตัดสินใจ]
- Alternatives: [ทางเลือกอื่นที่พิจารณา]
- Rationale: [เหตุผลที่เลือกทางนี้]
```

---

**BIZ-29.** Status update template
> สร้าง weekly status update

```
สร้าง weekly status update จาก data ที่มี:
1. อ่าน git log --since="1 week ago" — สรุป changes
2. ดู open PRs — งานที่รอ review
3. ดู DEVLOG — issues และ decisions

Format:
## Week [N] Status Update
### Highlights
- [top 3 achievements]
### Progress
- [feature progress by %]
### Issues
- [issues + severity + owner]
### Plan Next Week
- [priorities]
### Needs from Management
- [decisions, resources, approvals]
```

---

**BIZ-30.** Retrospective preparation
> เตรียมข้อมูลสำหรับ retrospective

```
เตรียมข้อมูลสำหรับ sprint retrospective:
1. Metrics:
   - Velocity: planned vs actual
   - Bug count: found vs fixed
   - PR turnaround time
2. What went well:
   - ดูจาก features ที่เสร็จ, tests ที่ pass
3. What could be improved:
   - ดูจาก carry-over items, bugs, delays
4. Claude Code effectiveness:
   - งานไหนที่ Claude ช่วยได้ดี?
   - งานไหนที่ต้องแก้ไข output ของ Claude มาก?
5. Suggested experiments for next sprint:
   - [experiment + expected outcome]
```

---

**BIZ-31.** Feature prioritization matrix
> จัดลำดับ features ตามหลายมิติ

```
ช่วยจัดลำดับ features เหล่านี้:
[list features]

ประเมินแต่ละ feature ตาม:
| Feature | Business Value | User Impact | Technical Effort | Risk | Score |
|---------|---------------|-------------|-----------------|------|-------|
| [name] | H/M/L | H/M/L | S/M/L/XL | H/M/L | [calculated] |

แนะนำลำดับ implementation:
1. Quick wins (high value, low effort)
2. Strategic (high value, high effort)
3. Nice-to-have (low value, low effort)
4. Avoid for now (low value, high effort)
```

---

**BIZ-32.** Competitor/alternative analysis
> เปรียบเทียบ approach กับ alternatives

```
เปรียบเทียบ approach สำหรับ [feature/solution]:
| Aspect | Our Approach | Alternative A | Alternative B |
|--------|-------------|---------------|---------------|
| Cost | | | |
| Time to market | | | |
| Scalability | | | |
| Maintenance | | | |
| User experience | | | |
| Risk | | | |

แนะนำ approach + เหตุผล
```

---

**BIZ-33.** Handoff document (PM → new PM)
> ส่งต่อ project ให้ PM คนใหม่

```
สร้าง project handoff document:
1. Project Overview: scope, goals, status
2. Team: members, roles, responsibilities
3. Architecture: high-level (ดูจาก CLAUDE.md + code)
4. Process: sprint cadence, ceremonies, tools
5. Current Status: features done/in-progress/planned
6. Risks & Issues: active issues + mitigation
7. Stakeholders: who, expectations, communication cadence
8. Key Decisions: link to decision log / ADRs
9. Budget: current usage, projections
10. Upcoming Milestones: dates, deliverables
```

---

**BIZ-34.** Requirement validation with code
> ตรวจว่า code ตรง requirements ไหม

```
เปรียบเทียบ requirements ใน [spec path] กับ implementation:
1. อ่าน spec — list ทุก requirement
2. อ่าน code — หา implementation ของแต่ละ requirement
3. สรุป:
   - ✅ Implemented correctly
   - ⚠️ Implemented but not matching spec
   - ❌ Not implemented yet
4. สำหรับ ⚠️: ระบุ gap + ว่า spec หรือ code ที่ควรแก้
5. สำหรับ ❌: ระบุ effort ที่ต้องใช้ implement
```

---

**BIZ-35.** Scope management
> ช่วยจัดการ scope เมื่อมี change requests

```
Scope เดิม: [list original features]
Change requests ที่เข้ามา:
- [CR-1]: [description]
- [CR-2]: [description]

วิเคราะห์:
1. Effort ของแต่ละ CR
2. Impact ต่อ timeline
3. ถ้าเพิ่ม CR แล้ว capacity ไม่พอ — feature ไหนควร defer?
4. แนะนำ trade-offs:
   - Option A: เพิ่ม CR, defer [features]
   - Option B: เพิ่ม CR, ขยาย timeline
   - Option C: ปฏิเสธ CR, อธิบายเหตุผล
```

---

**BIZ-36.** สร้าง project charter
> เอกสารเริ่มโปรเจกต์อย่างเป็นทางการ

```
สร้าง project charter สำหรับ [project]:
1. Project Name & Description
2. Business Objectives — ทำไมต้องทำ
3. Scope — ทำอะไร / ไม่ทำอะไร (in-scope / out-of-scope)
4. Key Stakeholders — ใครเกี่ยวข้อง
5. Success Criteria — วัดความสำเร็จอย่างไร
6. Constraints — budget, timeline, resources
7. Assumptions — สิ่งที่สมมติ
8. Risks — ความเสี่ยงเบื้องต้น
9. High-level Timeline — milestones
10. Approval — sign-off from
```

---

**BIZ-37.** Feature acceptance review
> ตรวจรับงาน feature

```
ตรวจรับ feature [name]:
1. อ่าน spec ใน [spec path] — list acceptance criteria
2. สำหรับแต่ละ criteria:
   - หา implementation ใน code
   - หา test ที่ cover criteria นี้
   - Status: ✅ Pass / ❌ Fail / ⚠️ Partial
3. ตรวจเพิ่ม:
   - Error handling ครบไหม?
   - Edge cases handled?
   - Documentation updated?
4. Verdict: ✅ Accept / 🔧 Accept with conditions / ❌ Reject
```

---

**BIZ-38.** Capacity planning
> วางแผนกำลังคน

```
ช่วยวางแผน capacity:
Available team: [N developers, M days per sprint]
Backlog:
- [item 1]: size [S/M/L/XL]
- [item 2]: size [S/M/L/XL]
- [item 3]: size [S/M/L/XL]

คำนวณ:
1. Total capacity (dev-days)
2. Buffer (20% for bugs, meetings, etc.)
3. Net capacity
4. How many items fit?
5. Recommended sprint scope
6. Items to defer to next sprint
```

---

← [Architecture & Design](03_architecture_and_design.md) | [Quality & Security →](05_quality_and_security.md)
