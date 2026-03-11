# Developer Workflows — Implement, Debug, Refactor, Test, Git

> Prompt สำหรับนักพัฒนา (FE/BE) ครอบคลุมทุกขั้นตอนการทำงานประจำวัน
> Stack-agnostic — ใช้ได้กับทุก tech stack โดยแทนค่า `[placeholder]`

---

## สารบัญ

- [2.1 Implementation](#21-implementation)
- [2.2 Debugging](#22-debugging)
- [2.3 Refactoring](#23-refactoring)
- [2.4 Testing](#24-testing)
- [2.5 Git Workflow](#25-git-workflow)
- [2.6 Code Reading & Understanding](#26-code-reading--understanding)

---

## 2.1 Implementation

**DEV-01.** Implement feature จาก spec
> มี spec พร้อมแล้ว ต้องการ implement

```
อ่าน spec ใน [spec path] แล้ว implement [feature]:
- ใช้ architecture pattern เดียวกับที่มีอยู่ในโปรเจกต์
- สร้างไฟล์ใหม่ตาม directory structure ที่กำหนดใน CLAUDE.md
- เขียน tests ด้วย
- ถ้า spec ไม่ชัดตรงไหน ถามก่อน implement
```

---

**DEV-02.** Implement ตาม pattern ที่มีอยู่
> ดูตัวอย่างที่มีแล้วทำตาม

```
ดูตัวอย่าง [existing file/module] แล้วสร้าง [new feature] ตาม pattern เดียวกัน:
- ใช้โครงสร้างเดียวกัน
- ใช้ naming conventions เดียวกัน
- ใช้ error handling เดียวกัน
- เขียน tests ตาม pattern เดียวกับ tests ที่มี
```

💡 วิธีนี้ช่วยรักษา consistency ในโปรเจกต์

---

**DEV-03.** Implement แบบ TDD (Test-Driven)
> เขียน test ก่อน แล้วค่อย implement

```
ทำ [feature] แบบ TDD:
1. เขียน test cases ก่อน (ยังไม่ต้อง implement) ตาม requirements:
   - [requirement 1]
   - [requirement 2]
   - [edge case 1]
2. รัน test — ต้อง fail ทุกตัว
3. Implement code ให้ test ผ่านทีละตัว
4. Refactor ถ้าจำเป็น
```

---

**DEV-04.** สร้าง API endpoint
> สร้าง REST API endpoint ใหม่

```
สร้าง API endpoint:
- Method: [GET/POST/PUT/DELETE]
- Path: [/api/v1/resource]
- Request: [body/params description]
- Response: [expected response structure]
- Auth: [required/optional/none]
- Validation: [rules]
สร้างครบทุก layer ตาม architecture (handler/controller → service → repository)
รวม input validation, error responses, และ tests
```

---

**DEV-05.** สร้าง data model + migration
> สร้าง model และ database migration

```
สร้าง data model สำหรับ [entity]:
- Fields: [list fields + types]
- Relations: [belongs_to, has_many, etc.]
- Constraints: [unique, not null, default values]
- Indexes: [fields ที่ต้อง index]
สร้าง:
1. Model/struct definition
2. Migration file
3. Repository layer (CRUD operations)
```

---

**DEV-06.** สร้าง background job / scheduled task
> งานที่รันเบื้องหลัง

```
สร้าง background job สำหรับ [task description]:
- Trigger: [cron schedule / event-based / manual]
- Input: [data ที่ต้องใช้]
- Process: [ขั้นตอนการทำงาน]
- Output: [ผลลัพธ์ / side effects]
- Error handling: [retry strategy, dead letter queue]
- Logging: [สิ่งที่ต้อง log]
```

---

**DEV-07.** เพิ่ม feature เข้า codebase ที่มีอยู่
> ต่อเติม feature ใน code ที่คนอื่นเขียน

```
ต้องเพิ่ม [feature] เข้าไปใน [existing module]:
1. อ่าน code ปัจจุบันใน [path] เข้าใจ flow ก่อน
2. ระบุจุดที่ต้องเพิ่ม/แก้ไข
3. เสนอแผนก่อน (ไม่ implement เลย)
4. ตรวจว่าไม่ break existing functionality
5. เพิ่ม tests สำหรับ feature ใหม่
```

---

**DEV-08.** Implement validation rules
> สร้างระบบ validation

```
สร้าง input validation สำหรับ [resource/endpoint]:
- Field rules: [field1: required, min 3 chars], [field2: email format], etc.
- Business rules: [custom validation logic]
- Error response format: [field-level errors]
- Validation ต้องทำก่อน business logic
- Error messages ต้องชัดเจน อ่านแล้วรู้ว่าต้องแก้อะไร
```

---

**DEV-09.** สร้าง middleware / interceptor
> สร้าง cross-cutting concern

```
สร้าง middleware สำหรับ [purpose]:
- ทำอะไร: [logging / auth / rate-limit / CORS / etc.]
- Apply กับ: [all routes / specific routes / specific methods]
- Configuration: [parameters ที่ต้องตั้ง]
- Error handling: [response เมื่อ middleware reject]
ดูตัวอย่าง middleware ที่มีอยู่แล้วทำตาม pattern เดียวกัน
```

---

**DEV-10.** Implement ตาม interface ที่กำหนด
> Interface/contract ถูกกำหนดไว้แล้ว

```
Interface [name] ถูกกำหนดไว้ใน [path]:
[paste interface definition]

Implement interface นี้:
- ทุก method ต้อง implement ครบ
- error handling ตามที่ interface กำหนด
- เขียน unit tests ที่ test ผ่าน interface (ไม่ใช่ concrete type)
```

---

## 2.2 Debugging

**DEV-11.** วิเคราะห์ error message / stack trace
> Copy-paste error แล้วให้วิเคราะห์

```
วิเคราะห์ error นี้:
[paste error/stack trace]

1. สาเหตุที่น่าจะเป็นไปได้ (เรียงจากน่าจะใช่ที่สุด)
2. ไฟล์ที่เกี่ยวข้อง
3. วิธีแก้ไข
4. วิธีป้องกันไม่ให้เกิดอีก
```

---

**DEV-12.** Debug failing test
> Test ที่เคย pass กลับ fail

```
Test นี้ fail:
- Test file: [test file path]
- Test name: [test name]
- Error: [error message]
- เมื่อก่อนเคย pass

ช่วย:
1. อ่าน test code + production code ที่เกี่ยวข้อง
2. หาว่าอะไรเปลี่ยนไป (ดู git log ไฟล์ที่เกี่ยว)
3. ระบุ root cause
4. เสนอวิธีแก้
```

---

**DEV-13.** Debug race condition / concurrency issue
> ปัญหาที่เกิดไม่แน่นอน

```
มี bug: [symptom] — เกิดไม่สม่ำเสมอ สงสัยว่าเป็น race condition
อ่าน [file] ฟังก์ชัน [name] วิเคราะห์:
1. flow ของ operation
2. shared state ที่อาจถูก access พร้อมกัน
3. locking mechanism ที่ใช้อยู่ (ถ้ามี)
4. จุดที่อาจเกิด race
5. วิธีแก้ + วิธี test race condition
```

---

**DEV-14.** Debug performance issue
> API ช้า หรือ app กิน memory มาก

```
[endpoint/function] ช้า: [N] วินาที เมื่อมี [N] records
วิเคราะห์ code ตั้งแต่ [entry point] ถึง [data layer]:
1. Database queries — มี N+1 ไหม? query ช้าตรงไหน?
2. Loops — มี nested loops ที่ไม่จำเป็นไหม?
3. Memory — มี allocation ที่มากเกินไปไหม?
4. External calls — มี blocking calls ไหม?
5. Caching — ตรงไหนควร cache?
เรียง bottleneck ตาม impact จากมากไปน้อย
```

---

**DEV-15.** Debug data inconsistency
> Data ไม่ตรงกับที่ควรเป็น

```
มี data inconsistency: [อธิบายปัญหา]
Expected: [expected state]
Actual: [actual state]
ช่วยวิเคราะห์:
1. Code paths ที่อาจทำให้ data เพี้ยน
2. Race conditions ที่อาจเกิด
3. Missing transactions
4. ลำดับ operations ที่อาจผิด
5. Validation ที่ขาดหาย
```

---

**DEV-16.** Debug integration issue
> ปัญหาเมื่อ connect กับ service อื่น

```
ต่อกับ [service/API] แล้วมีปัญหา: [error/symptom]
Config ที่ใช้: [relevant config]
ช่วยตรวจ:
1. Connection config ถูกต้องไหม?
2. Request format ตรงกับที่ service ต้องการไหม?
3. Auth/credentials ถูกต้องไหม?
4. Response ที่ได้กลับมาเป็นอย่างไร?
5. Timeout / retry settings เหมาะสมไหม?
```

---

**DEV-17.** Debug แบบ systematic
> ปัญหาซับซ้อน ต้องหาอย่างเป็นระบบ

```
มี bug: [symptom description]
Reproduce steps: [steps]
ช่วย debug อย่างเป็นระบบ:
1. ตั้ง hypothesis 3 ข้อ ว่าสาเหตุน่าจะเป็นอะไร
2. สำหรับแต่ละ hypothesis — ระบุวิธีทดสอบ
3. เริ่มจาก hypothesis ที่มีโอกาสมากที่สุด
4. ทดสอบทีละข้อ จนหา root cause
```

---

**DEV-18.** Fix bug + regression test
> แก้ bug แล้วเขียน test ป้องกัน

```
Bug: [description]
Root cause: [cause ถ้ารู้แล้ว]
1. เขียน test ที่ reproduce bug ก่อน (test ต้อง fail กับ code ปัจจุบัน)
2. Fix bug
3. รัน test — ต้อง pass
4. ตรวจว่า existing tests ยังผ่าน (ไม่ break อะไรอื่น)
5. เพิ่ม DEVLOG entry สรุปปัญหาและวิธีแก้
```

---

## 2.3 Refactoring

**DEV-19.** วิเคราะห์ไฟล์เพื่อ refactor
> ดูก่อนว่าควร refactor อะไร

```
วิเคราะห์ [file path] ที่มีกว่า [N] บรรทัด:
1. ระบุ responsibilities ทั้งหมด (แต่ละกลุ่ม function ทำอะไร)
2. จำนวน methods/functions ต่อกลุ่ม
3. Dependencies กับ modules อื่น
4. จุดที่ละเมิด Single Responsibility Principle
5. แนะนำการแยกที่เหมาะสม
ยังไม่ต้อง implement — แค่วิเคราะห์
```

---

**DEV-20.** วางแผน refactoring
> วาง plan ก่อน เพื่อ refactor อย่างปลอดภัย

```
วางแผน refactor [module/file]:
- เป้าหมาย: [เหตุผลที่ต้อง refactor]
- แยกเป็น [N] steps ที่แต่ละ step:
  1. ทำอะไร
  2. ไฟล์ที่กระทบ
  3. วิธี verify ว่า step นี้ไม่ break อะไร
- ทุก step ต้อง commit ได้ + tests ผ่าน
- ไม่เปลี่ยน behavior (public API / external interface เดิม)
เขียนเป็น step-by-step plan ก่อน ยังไม่ implement
```

---

**DEV-21.** Extract method / service / module
> แยก code ออกมาเป็น unit ใหม่

```
Extract [function/method name] จาก [source file]:
1. ระบุ code block ที่จะ extract
2. กำหนด interface / function signature
3. ย้าย code + ปรับ dependencies
4. อัปเดต caller ทั้งหมดให้ใช้ unit ใหม่
5. ย้าย/เพิ่ม tests สำหรับ unit ใหม่
ตรวจว่า behavior ไม่เปลี่ยน
```

---

**DEV-22.** Rename ข้าม codebase
> เปลี่ยนชื่อ function / variable / type ทุกที่

```
Rename [old name] เป็น [new name] ทั้งโปรเจกต์:
1. หาทุกที่ที่ใช้ [old name] (code, tests, configs, docs)
2. เปลี่ยนชื่อทุกจุด
3. ตรวจว่า build/test ผ่าน
4. ตรวจว่า imports/references อัปเดตครบ
5. ดูว่ามี string references (config, env) ที่ต้องเปลี่ยนด้วยไหม
```

---

**DEV-23.** ลบ dead code
> ทำความสะอาด code ที่ไม่ได้ใช้

```
หา dead code ในโปรเจกต์:
1. Functions/methods ที่ไม่มี caller
2. Imports ที่ไม่ได้ใช้
3. Variables ที่ assign แล้วไม่ใช้
4. Files ที่ไม่มีใคร import
5. Feature flags / configs ที่ไม่ได้ใช้แล้ว
สำหรับแต่ละจุด — ยืนยันว่าไม่ได้ใช้จริง (ไม่ใช่ dynamic reference)
```

---

**DEV-24.** Migrate pattern
> เปลี่ยนจาก pattern A ไป pattern B

```
ต้องการเปลี่ยนจาก [pattern A] เป็น [pattern B] ใน [module]:
ตัวอย่าง:
- จาก: [code snippet ของ pattern เดิม]
- เป็น: [code snippet ของ pattern ใหม่]

1. หาทุกจุดที่ใช้ pattern เดิม
2. เปลี่ยนทีละจุด + verify
3. อัปเดต tests
4. อัปเดต CLAUDE.md ถ้าเป็น convention change
```

---

**DEV-25.** Simplify complex function
> ฟังก์ชันซับซ้อนเกิน ต้อง simplify

```
ฟังก์ชัน [name] ใน [file] ซับซ้อนเกิน ([N] บรรทัด, [N] levels of nesting):
ช่วย simplify:
1. แยก early returns (guard clauses)
2. Extract helper functions
3. ลด nesting levels
4. ทำให้ logic อ่านง่ายขึ้น
ต้องไม่เปลี่ยน behavior — tests ที่มีต้องยังผ่าน
```

---

## 2.4 Testing

**DEV-26.** สร้าง unit tests
> เขียน unit tests สำหรับ code ที่มี

```
เขียน unit tests สำหรับ [file/function]:
- ใช้ [test framework] (ถ้ามี preference)
- Table-driven / parameterized tests
- Mock dependencies ด้วย interface
- Cover cases:
  - Happy path
  - Edge cases: [list]
  - Error cases: [list]
- ทุก test ต้องมีชื่อที่อธิบายว่า test อะไร
```

---

**DEV-27.** สร้าง integration tests
> Test ที่ต่อกับ dependencies จริง

```
เขียน integration test สำหรับ [endpoint/flow]:
- ใช้ [test setup approach - testcontainers/docker/in-memory DB]
- Test full flow: [request → processing → response → side effects]
- ตรวจสอบทั้ง response และ DB state
- Setup: seed data ที่จำเป็น
- Cleanup: reset state หลังแต่ละ test
- Test ต้อง run แยกกันได้ (ไม่ depend on order)
```

---

**DEV-28.** เพิ่ม edge case tests
> ดู code แล้วหา edge cases ที่ยังไม่มี test

```
อ่าน [file/function] + existing tests
แล้วหา edge cases ที่ยังไม่มี test:
1. Boundary values (0, 1, max, negative)
2. Empty/null inputs
3. Concurrent access
4. Timeout / slow responses
5. Invalid state transitions
6. Unicode / special characters
เขียน tests สำหรับ edge cases ที่หา
```

---

**DEV-29.** ปรับปรุง test coverage
> เพิ่ม coverage ในจุดที่ขาด

```
วิเคราะห์ test coverage ของ [module]:
1. รัน coverage report
2. ระบุ functions/branches ที่ยังไม่มี test
3. จัดลำดับตาม:
   - Criticality (business logic สำคัญ)
   - Risk (error-prone code)
   - Complexity (code ที่ซับซ้อน)
4. เขียน tests สำหรับ top 5 ที่สำคัญที่สุด
```

---

**DEV-30.** สร้าง test fixtures / helpers
> สร้าง utilities สำหรับ test

```
สร้าง test helpers สำหรับ [module]:
1. Factory functions — สร้าง test data:
   - Create[Entity](overrides) — default values + override
   - CreateValid[Entity]() — valid data
   - CreateInvalid[Entity]() — invalid data
2. Setup helpers — เตรียม test environment
3. Assert helpers — custom assertions ที่ใช้บ่อย
4. Cleanup helpers — reset state หลัง test
ใช้ naming convention เดียวกับ test helpers ที่มี
```

---

**DEV-31.** สร้าง test data
> Generate realistic test data

```
สร้าง test data สำหรับ [entity/table]:
- [N] records ที่มี variations:
  - ค่าปกติ
  - Boundary values
  - Special cases (unicode, long strings, etc.)
- Relationships ระหว่าง entities ต้องถูกต้อง
- Data ต้อง realistic (ไม่ใช่ "test1", "test2")
สร้างเป็น [format: SQL/JSON/code fixtures]
```

---

**DEV-32.** Mock external dependencies
> สร้าง mocks สำหรับ dependencies ที่ test ไม่ได้

```
สร้าง mock สำหรับ [interface/service]:
- Methods: [list methods ที่ต้อง mock]
- Default behavior: [return อะไรปกติ]
- Error scenarios: [return errors สำหรับ test error paths]
- ใช้ [mock framework] ถ้ามี preference
- Mock ต้อง verify ว่าถูก call ด้วย arguments ที่ถูกต้อง
```

---

## 2.5 Git Workflow

**DEV-33.** เขียน commit message
> สร้าง commit message จาก staged changes

```
ดู git diff --staged แล้วเขียน commit message:
- ใช้ conventional commits format (feat/fix/refactor/docs/test/chore)
- บรรทัดแรก < 72 chars
- Body อธิบายว่าทำไม (why) ไม่ใช่ทำอะไร (what)
- ถ้ามีหลาย changes แยกเป็น bullet points
- Reference issue number ถ้ามี
```

---

**DEV-34.** สร้าง PR description
> Auto-generate จาก commits + diff

```
ดู git log main..HEAD และ git diff main...HEAD แล้วสร้าง PR description:
## Summary
- [bullet points]
## Changes
- [file-by-file summary ของ changes สำคัญ]
## Test Plan
- [ ] [test checklist]
## Breaking Changes
- [ถ้ามี]
## Screenshots
- [ถ้าเกี่ยวกับ UI]
```

---

**DEV-35.** Self-review ก่อน submit PR
> Review code ตัวเองก่อนส่งให้คนอื่น

```
ดู git diff main..HEAD แล้ว self-review:
1. Logic ถูกต้องไหม?
2. Naming conventions ตรง CLAUDE.md?
3. Error handling ครบไหม?
4. มี test สำหรับ logic ใหม่?
5. มี hardcoded values ที่ควรเป็น config?
6. มี security concerns? (SQL injection, XSS, etc.)
7. มี debug code หลุด? (console.log, print, TODO)
8. Performance — มี N+1, missing index?
สรุปเป็น ✅/❌ พร้อม suggestions
```

---

**DEV-36.** Resolve merge conflicts
> ช่วย resolve conflicts อย่างถูกต้อง

```
ดูไฟล์ที่มี merge conflicts แล้วช่วย resolve:
- เข้าใจว่าแต่ละฝั่ง (ours/theirs) ต้องการอะไร
- เลือกวิธี merge ที่ถูกต้องตาม business logic
- ตรวจว่า resolved code ยัง compile/build ได้
- รัน tests หลัง resolve
ถ้าไม่แน่ใจว่าควรเลือกฝั่งไหน ให้บอกก่อน อย่าเดา
```

---

**DEV-37.** Branch cleanup
> ลบ branches ที่ไม่ได้ใช้แล้ว

```
ดู git branch --merged main แล้วแนะนำ branches ที่ปลอดภัยจะลบ:
แยกเป็น:
- ✅ ปลอดภัยลบได้ (merged + no unique commits)
- ⚠️ ไม่แน่ใจ (ต้องตรวจเพิ่ม)
- ❌ ยังไม่ควรลบ (มี unmerged work)
อย่าลบเอง — แค่แนะนำ
```

---

**DEV-38.** Cherry-pick / backport
> นำ fix จาก branch หนึ่งไปอีก branch

```
ต้องนำ [commit/feature] จาก [source branch] ไป [target branch]:
1. ระบุ commits ที่ต้อง cherry-pick
2. ตรวจ dependencies — commits เหล่านี้ depend on commits อื่นไหม?
3. แนะนำลำดับ cherry-pick ที่ถูกต้อง
4. ตรวจว่า tests ผ่านหลัง cherry-pick
5. ระบุ potential conflicts
```

---

**DEV-39.** สร้าง gitignore
> สร้าง/อัปเดต .gitignore

```
ตรวจสอบ .gitignore ว่าครอบคลุม:
1. Build artifacts ([build dir], [dist dir])
2. Dependencies ([node_modules], [vendor], etc.)
3. IDE files (.vscode, .idea, etc.)
4. OS files (.DS_Store, Thumbs.db)
5. Environment files (.env, .env.local)
6. Logs (*.log)
7. Temp files
เทียบกับ template ของ [language/framework] แนะนำสิ่งที่ขาด
```

---

## 2.6 Code Reading & Understanding

**DEV-40.** อธิบาย architecture ของ codebase
> ต้องเข้าใจ architecture ทั้งหมด

```
อธิบายสถาปัตยกรรมของโปรเจกต์นี้:
1. กี่ layer? แต่ละ layer ทำอะไร?
2. Framework + libraries หลักอะไรบ้าง?
3. Data flow ตั้งแต่ request เข้าจนถึง response
4. Patterns ที่ใช้ (Repository, Service, MVC, etc.)
5. Key abstractions / interfaces
6. โครงสร้าง directories
อธิบายเป็น high-level ก่อน แล้วค่อยลง detail
```

---

**DEV-41.** Trace request flow
> ติดตาม request ตั้งแต่เข้าจนออก

```
Trace flow ของ [operation/endpoint] ตั้งแต่ request เข้ามาจนถึง response:
- ผ่าน files ไหนบ้าง? ตามลำดับ
- แต่ละ layer ทำอะไร?
- มี middleware / interceptors อะไรบ้าง?
- Query database อะไรบ้าง?
- External services ที่ call?
วาดเป็น sequence diagram (text-based)
```

---

**DEV-42.** เข้าใจ business rules จาก code
> อ่าน code แล้วสรุป business rules

```
อ่าน code ใน [path/module] แล้วสรุป business rules:
1. เงื่อนไข (conditions) ทั้งหมดที่มี
2. State transitions ที่เป็นไปได้
3. Constraints / limitations
4. Validation rules
5. Edge cases ที่ handle อยู่
สรุปเป็น business language ไม่ใช่ technical language
```

---

**DEV-43.** Map dependencies ระหว่าง modules
> เข้าใจความสัมพันธ์ระหว่าง modules

```
วิเคราะห์ dependencies ระหว่าง modules ในโปรเจกต์:
1. Module ไหน depend on module ไหน
2. มี circular dependencies ไหม?
3. Module ไหนเป็น "hub" (ถูก depend มากที่สุด)
4. Module ไหน "ฟรี" (ไม่ depend ใคร)
วาดเป็น dependency graph (text-based)
```

---

**DEV-44.** หาตัวอย่างการใช้ pattern/interface
> ดูว่า pattern ถูกใช้ที่ไหนบ้าง

```
หาตัวอย่างการใช้ [interface/pattern/function] ในโปรเจกต์:
1. ที่ไหนบ้างที่ implement [interface]?
2. ใช้ pattern นี้ในรูปแบบไหนบ้าง?
3. มี variations ของ pattern นี้ไหม?
4. มี anti-pattern ที่ใช้ผิดวิธีไหม?
สรุปตัวอย่างที่ดีที่สุด เพื่อใช้เป็น reference
```

---

**DEV-45.** เข้าใจ error handling strategy
> ดูว่า project จัดการ errors อย่างไร

```
วิเคราะห์ error handling strategy ของโปรเจกต์:
1. Custom error types ที่กำหนด
2. Error wrapping / propagation pattern
3. Error logging — log ที่ไหน, format อะไร
4. Client-facing errors — format อะไร, มี error codes ไหม
5. Recovery strategy — retry, fallback, circuit breaker
6. ส่วนไหนที่ error handling ยังไม่ดี
```

---

**DEV-46.** เข้าใจ config & environment setup
> ดูว่า project configure อย่างไร

```
อธิบาย configuration setup ของโปรเจกต์:
1. Config files ที่ใช้ (และ format)
2. Environment variables ที่ต้องตั้ง
3. ค่า default ของแต่ละ config
4. Config ที่ต่างกันระหว่าง dev/staging/production
5. Sensitive configs ที่ต้องเป็น secret
6. วิธีเพิ่ม config ใหม่
```

---

**DEV-47.** เข้าใจ authentication & authorization flow
> ดูว่า auth ทำงานอย่างไร

```
อธิบาย authentication + authorization flow ของโปรเจกต์:
1. Auth method: [JWT/session/OAuth/etc.]
2. Login flow ตั้งแต่ request จนได้ token
3. Token refresh flow (ถ้ามี)
4. Middleware ที่ตรวจ auth
5. Role-based access control (ถ้ามี)
6. Protected vs public routes
7. Logout / token revocation
```

---

**DEV-48.** Quick code explanation
> อธิบาย code snippet เฉพาะจุด

```
อธิบาย code ใน [file]:[line start]-[line end]:
1. ทำอะไร (high-level)
2. ทำไมถึงเขียนแบบนี้
3. Edge cases ที่ handle
4. จุดที่อาจทำให้สับสน
อธิบายให้คนที่ไม่รู้จัก codebase เข้าใจได้
```

---

**DEV-49.** Compare implementation approaches
> เปรียบเทียบวิธีที่ต่างกัน

```
เปรียบเทียบ approach สำหรับ [task]:
Approach A: [description]
Approach B: [description]

เทียบกันในด้าน:
1. Complexity — เขียน + maintain
2. Performance — speed, memory
3. Testability — ง่ายต่อการ test
4. Extensibility — รองรับ future changes
5. Consistency — ตรงกับ patterns ที่มีในโปรเจกต์
แนะนำว่าควรใช้อันไหน + เหตุผล
```

---

**DEV-50.** อ่าน external library source code
> เข้าใจ library ที่ใช้อยู่

```
อธิบายวิธีใช้ [library/package] ที่ใช้ในโปรเจกต์:
1. import อยู่ใน files ไหนบ้าง
2. ใช้ features อะไร
3. Configuration ที่ตั้งไว้
4. Patterns การใช้ที่ถูกต้องตาม library
5. Common pitfalls ของ library นี้
6. Version ที่ใช้ + breaking changes ที่ควรรู้
```

---

**DEV-51.** สร้าง development environment
> Setup local dev environment

```
สร้าง development environment setup guide จาก codebase:
1. Prerequisites — tools, versions ที่ต้องติดตั้ง
2. Clone + initial setup steps
3. Environment variables — ต้องตั้งอะไรบ้าง
4. Database setup — migration, seed data
5. Start development — commands ที่ต้องรัน
6. Verify — วิธีตรวจว่า setup สำเร็จ
7. Common issues + solutions
```

---

**DEV-52.** Document API ที่สร้าง
> สร้าง API documentation

```
ดู handlers/controllers ทั้งหมดแล้วสร้าง API documentation:
แต่ละ endpoint:
- Method + Path
- Description
- Headers ที่ต้องส่ง (auth, content-type)
- Request body / query params (with types)
- Response body (success + error)
- Example request + response
- Status codes ที่เป็นไปได้
```

---

← [Session Management](01_session_management.md) | [Architecture & Design →](03_architecture_and_design.md)
