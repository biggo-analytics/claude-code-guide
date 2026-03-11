# Quality & Security — Review, Testing Strategy, Security, Performance

> Prompt สำหรับ QA / Security Engineer — ครอบคลุม code review, test strategy, security audit, performance analysis
> ใช้ได้กับทุก tech stack โดยแทนค่า `[placeholder]`

---

## สารบัญ

- [5.1 Code Review](#51-code-review)
- [5.2 Test Strategy & Planning](#52-test-strategy--planning)
- [5.3 Security Audit](#53-security-audit)
- [5.4 Performance Analysis](#54-performance-analysis)
- [5.5 Quality Metrics & Reporting](#55-quality-metrics--reporting)

---

## 5.1 Code Review

**QA-01.** Review PR / Branch แบบครบวงจร
> review diff ทั้งหมดของ branch ที่กำลังจะ merge

```
Review branch [branch] เทียบกับ [base branch]:
1. อ่าน diff ทั้งหมด (`git diff [base]...[branch]`)
2. ตรวจสอบแต่ละไฟล์:
   - ✅ Correctness — logic ถูกต้องตาม requirements ไหม?
   - ✅ Error handling — ครอบคลุมทุก error case?
   - ✅ Security — มี injection, XSS, hardcoded secrets ไหม?
   - ✅ Performance — มี N+1, unbounded query, missing index?
   - ✅ Style — ตรงกับ conventions ในโปรเจกต์?
3. สรุปเป็นตาราง: ไฟล์, severity (critical/warning/info), รายละเอียด, suggestion
4. ให้ verdict: Approve / Request Changes พร้อมเหตุผล
```

---

**QA-02.** Self-review ก่อนสร้าง PR
> ตรวจงานตัวเองก่อน push

```
ช่วย review staged changes ของฉัน (git diff --staged):
1. มี code ที่ไม่ควร commit ไหม? (debug logs, TODO hacks, hardcoded values)
2. commit message เหมาะสมไหม?
3. tests ครอบคลุม changes ทั้งหมดไหม?
4. มี breaking changes ที่ต้องแจ้งทีมไหม?
5. documentation ต้อง update ไหม?
แนะนำสิ่งที่ควรแก้ก่อน push
```

---

**QA-03.** Review เฉพาะด้าน Security
> focus เฉพาะ security concerns

```
Review [file/module/PR] เฉพาะด้าน security:
- Input validation — ทุก user input ผ่าน validation ก่อนใช้?
- SQL/NoSQL injection — ใช้ parameterized queries?
- XSS — output ทำ encoding/escaping?
- Authentication — เช็ค authn ทุก endpoint ที่ต้องการ?
- Authorization — เช็ค authz ว่า user มีสิทธิ์?
- Secrets — มี hardcoded credentials/API keys ไหม?
- Data exposure — response ส่ง sensitive data เกินจำเป็นไหม?
รายงาน: ไฟล์, บรรทัด, severity, CWE ID (ถ้ามี), วิธีแก้
```

---

**QA-04.** Review เฉพาะด้าน Performance
> focus เฉพาะ performance concerns

```
Review [file/module/PR] เฉพาะด้าน performance:
- Database: N+1 queries? missing indexes? unbounded SELECT?
- Memory: large allocations? memory leaks? unbounded caching?
- Concurrency: race conditions? deadlocks? missing locks?
- I/O: blocking operations? missing timeouts? retry storms?
- Algorithm: complexity เหมาะสม? มี O(n²) ที่ซ่อนอยู่?
แนะนำ optimization พร้อม expected impact
```

---

**QA-05.** Review API contract consistency
> ตรวจว่า API spec ตรงกับ implementation

```
ตรวจสอบ API contract consistency:
1. อ่าน API spec/documentation ใน [spec path]
2. อ่าน implementation ใน [handler/controller path]
3. เปรียบเทียบ:
   - HTTP methods & paths ตรงกัน?
   - Request body / query params ตรงกัน?
   - Response shape & status codes ตรงกัน?
   - Error format ตรงกัน?
   - Field naming (camelCase/snake_case) consistent?
รายงาน mismatches เป็นตาราง
```

---

**QA-06.** Review ตาม checklist เฉพาะทาง
> ใช้ checklist ที่กำหนดเอง

```
Review [file/module] ตาม checklist นี้:
- [ ] [checklist item 1]
- [ ] [checklist item 2]
- [ ] [checklist item 3]
- [ ] [checklist item 4]
- [ ] [checklist item 5]
ตรวจทีละข้อ รายงานผล pass/fail พร้อมหลักฐาน
```

💡 ปรับ checklist ตามมาตรฐานทีม เช่น error handling checklist, accessibility checklist

---

**QA-07.** Generate review comments แบบ structured
> สร้าง review comments ที่พร้อม post

```
จาก diff ของ [branch/PR]:
สร้าง review comments ในรูปแบบ:

**[severity]** `[file]:[line]`
> [คำอธิบาย]
```suggestion
[code suggestion ถ้ามี]
```

จัดกลุ่มตาม severity:
1. 🔴 Must Fix — bugs, security issues
2. 🟡 Should Fix — improvements, best practices
3. 🔵 Nice to Have — style, readability
```

---

## 5.2 Test Strategy & Planning

**QA-08.** วางแผน test strategy สำหรับ feature
> กำหนดว่า feature นี้ต้องทดสอบอะไร อย่างไร

```
วางแผน test strategy สำหรับ [feature]:
อ่าน spec ใน [spec path] แล้วออกแบบ:

1. **Unit Tests** — functions/methods ไหนต้องมี unit test?
   - Happy path cases
   - Edge cases
   - Error cases

2. **Integration Tests** — ต้อง test อะไรข้ามระบบ?
   - API endpoint tests
   - Database integration
   - External service interaction

3. **E2E Tests** — user flows ไหนต้อง test?
   - Critical path
   - Error recovery

4. **Test Data** — ต้องเตรียม data อะไร?

สรุปเป็นตาราง: test type, test case, priority, estimated effort
```

---

**QA-09.** วิเคราะห์ test coverage gaps
> หา code ที่ยังไม่มี test ครอบคลุม

```
วิเคราะห์ test coverage ของ [module/path]:
1. อ่านทุกไฟล์ใน [source path]
2. อ่าน tests ที่เกี่ยวข้องใน [test path]
3. ระบุ:
   - Functions ที่ไม่มี test เลย
   - Functions ที่มี test แต่ไม่ครอบคลุม edge cases
   - Error paths ที่ไม่มี test
   - Business rules ที่ไม่มี test verify
4. เรียงตาม priority (critical business logic ก่อน)
5. แนะนำ test cases ที่ควรเพิ่ม
```

---

**QA-10.** สร้าง test cases จาก spec
> แปลง spec เป็น test cases

```
อ่าน spec ใน [spec path] แล้วสร้าง test cases:
1. แปลง acceptance criteria เป็น test cases
2. เพิ่ม edge cases ที่ spec ไม่ได้ระบุ:
   - boundary values
   - null/empty inputs
   - concurrent access
   - timeout scenarios
   - permission/authorization
3. เขียนในรูปแบบ:
   - Test ID: TC-[N]
   - Description
   - Precondition
   - Steps
   - Expected Result
   - Priority (P0-P3)
```

---

**QA-11.** ออกแบบ test data
> สร้างชุด test data ที่ครอบคลุม

```
ออกแบบ test data สำหรับ [feature/module]:
1. **Valid data** — ข้อมูลปกติที่ถูกต้อง (happy path)
2. **Boundary values** — ค่าขอบ (min, max, min-1, max+1)
3. **Invalid data** — ข้อมูลไม่ถูกต้อง (wrong type, too long, negative)
4. **Edge cases** — empty string, null, Unicode, special characters
5. **Business-specific** — ข้อมูลที่ trigger business rules เฉพาะ

Output เป็น table-driven test format ที่ใส่ใน code ได้เลย
```

---

**QA-12.** สร้าง Regression test plan
> วางแผนทดสอบก่อน release

```
สร้าง regression test plan สำหรับ release [version]:
1. อ่าน git log ตั้งแต่ [last release tag] ถึง HEAD
2. ระบุ:
   - Features ใหม่ที่ต้อง verify
   - Bug fixes ที่ต้อง verify
   - Modules ที่ถูกเปลี่ยน → areas ที่อาจ impact
3. สร้าง regression checklist:
   - [ ] Critical paths (order, payment, auth)
   - [ ] Changed features
   - [ ] Integration points
   - [ ] Performance baselines
เรียงตาม risk level
```

---

**QA-13.** ตรวจ test quality
> review ว่า tests ที่มีอยู่มีคุณภาพดีหรือไม่

```
Review test quality ของ [test file/directory]:
1. Test naming — อ่านชื่อแล้วเข้าใจว่า test อะไร?
2. Assertions — assert ครบทุกสิ่งที่ควร check?
3. Independence — test แต่ละตัว run แยกกันได้?
4. Deterministic — ผลลัพธ์เหมือนกันทุกครั้ง? (no flaky tests)
5. Readability — Arrange-Act-Assert ชัดเจน?
6. Coverage — ครอบคลุม happy path + error path?
แนะนำ improvements ที่เป็นรูปธรรม
```

---

## 5.3 Security Audit

**QA-14.** OWASP Top 10 Audit
> ตรวจ codebase ตาม OWASP Top 10

```
ตรวจสอบ codebase ตาม OWASP Top 10 (2021):
สแกนไฟล์ใน [path] แล้วตรวจ:

A01: Broken Access Control
A02: Cryptographic Failures
A03: Injection
A04: Insecure Design
A05: Security Misconfiguration
A06: Vulnerable and Outdated Components
A07: Identification and Authentication Failures
A08: Software and Data Integrity Failures
A09: Security Logging and Monitoring Failures
A10: Server-Side Request Forgery (SSRF)

รายงานเป็นตาราง: category, severity, ไฟล์:บรรทัด, description, remediation
```

---

**QA-15.** ตรวจ Authentication & Authorization
> audit ระบบ authn/authz

```
Audit authentication & authorization ใน [module/path]:
1. **Authentication:**
   - Password hashing algorithm? (bcrypt/argon2?)
   - Token generation & validation (JWT? expiry?)
   - Session management (timeout? refresh?)
   - Brute force protection? (rate limiting?)

2. **Authorization:**
   - มี middleware/guard ตรวจ permission ทุก endpoint?
   - RBAC/ABAC model ชัดเจน?
   - Endpoint ไหนเปิดเป็น public? ถูกต้องไหม?
   - Privilege escalation paths?

รายงาน findings + severity + recommended fix
```

---

**QA-16.** ตรวจ Secrets & Configuration
> หา secrets ที่ hardcoded หรือ exposed

```
สแกน codebase หา secrets & sensitive configuration:
1. Hardcoded credentials (passwords, API keys, tokens)
2. .env files ที่อาจถูก commit (.gitignore ครอบคลุม?)
3. Configuration files ที่มี sensitive values
4. Connection strings ที่มี credentials
5. Private keys หรือ certificates
6. Debug/dev settings ที่เปิดอยู่ใน production config

ตรวจ:
- .gitignore มีครบ?
- .env.example มี placeholder (ไม่ใช่ค่าจริง)?
- Docker/CI files ใช้ secrets/env vars ไม่ใช่ hardcoded?

รายงาน: ไฟล์, บรรทัด, ประเภท secret, risk level, วิธีแก้
```

---

**QA-17.** สร้าง Security test cases
> เขียน test cases เฉพาะด้าน security

```
สร้าง security test cases สำหรับ [feature/module]:
1. **Injection tests:**
   - SQL injection payloads
   - XSS payloads (reflected, stored)
   - Command injection
   - Path traversal

2. **Authentication tests:**
   - Invalid/expired tokens
   - Missing auth header
   - Replay attacks

3. **Authorization tests:**
   - Access resource ของ user อื่น
   - Escalate privileges
   - Access admin endpoints as normal user

4. **Input validation tests:**
   - Oversized payloads
   - Unexpected content types
   - Unicode edge cases

เขียนเป็น test code ที่ run ได้เลย
```

---

**QA-18.** ตรวจ Dependency vulnerabilities
> ตรวจ third-party dependencies

```
ตรวจ dependency vulnerabilities:
1. อ่าน [package.json / go.mod / requirements.txt / pom.xml]
2. ระบุ dependencies ที่:
   - มี known CVEs
   - เป็น version เก่ามาก (unmaintained?)
   - มี license ที่อาจมีปัญหา
3. แนะนำ:
   - Dependencies ที่ควร update ด่วน (security fixes)
   - Dependencies ที่ควร update (new features, performance)
   - Dependencies ที่ควร replace (deprecated/unmaintained)
รายงานเป็นตาราง: package, current version, latest version, severity, action
```

💡 ใช้คู่กับ `npm audit`, `go vuln check`, `pip-audit`, `snyk`

---

## 5.4 Performance Analysis

**QA-19.** วิเคราะห์ slow endpoint
> หาสาเหตุที่ API endpoint ช้า

```
วิเคราะห์ performance ของ [endpoint]:
1. Trace request flow: handler → service → repository → DB
2. ตรวจแต่ละ layer:
   - Database: query ยังไง? มี index? N+1?
   - Service: มี loop ที่ call DB ซ้ำ? unnecessary processing?
   - I/O: external API calls? file operations?
   - Memory: large object allocations? unnecessary copies?
3. แนะนำ optimization:
   - Quick wins (ทำง่าย impact สูง)
   - Medium effort (ต้อง refactor เล็กน้อย)
   - Long term (ต้อง redesign)
```

---

**QA-20.** Database query optimization
> ปรับปรุง database queries

```
วิเคราะห์และ optimize database queries ใน [file/module]:
1. หาทุก query ที่เรียกจาก module นี้
2. ตรวจแต่ละ query:
   - มี index รองรับ? (แนะนำ index ที่ควรสร้าง)
   - SELECT * หรือ select เฉพาะ columns ที่ใช้?
   - N+1 problem? (ควรใช้ JOIN/preload?)
   - Pagination ถูกต้อง? (cursor vs offset)
   - WHERE clause efficient?
3. แนะนำ query ที่ optimize แล้ว พร้อม explain ความแตกต่าง
```

---

**QA-21.** ตรวจ Memory & Resource usage
> หา memory leaks และ resource ที่ไม่ถูกปิด

```
ตรวจ memory & resource management ใน [module/path]:
1. **Resource leaks:**
   - DB connections ที่เปิดแล้วไม่ปิด
   - File handles ที่ไม่ได้ close
   - HTTP clients ที่ไม่ได้ reuse
   - Goroutines/threads ที่ไม่มี lifecycle management
2. **Memory issues:**
   - Large buffers ที่ allocate ซ้ำ
   - Unbounded caches/maps
   - Closures ที่ capture ของใหญ่
3. **Timeout & limits:**
   - External calls มี timeout?
   - Rate limiting มี?
   - Request body size limit?
รายงาน findings พร้อม code fix
```

---

**QA-22.** Load testing plan
> วางแผน load test

```
สร้าง load testing plan สำหรับ [service/feature]:
1. **Identify critical endpoints:**
   - ดูจาก routes/handlers ว่ามี endpoints อะไรบ้าง
   - จัดลำดับตาม business criticality

2. **Define scenarios:**
   - Normal load: [N] concurrent users
   - Peak load: [N] concurrent users
   - Stress test: gradually increase until failure
   - Spike test: sudden burst of [N] requests

3. **Success criteria:**
   - Response time p95 < [N]ms
   - Error rate < [N]%
   - Throughput > [N] req/s

4. **Test data & setup requirements**

Output เป็น script template สำหรับ [k6 / Artillery / JMeter / wrk]
```

---

## 5.5 Quality Metrics & Reporting

**QA-23.** สรุป code quality ของโปรเจกต์
> ภาพรวม quality metrics

```
วิเคราะห์ code quality ภาพรวมของโปรเจกต์:
1. **Code metrics:**
   - จำนวนไฟล์, บรรทัด code, test files
   - ไฟล์ที่ใหญ่ที่สุด (อาจต้อง split)
   - Functions ที่ยาวที่สุด (อาจต้อง refactor)

2. **Test health:**
   - Test files vs source files ratio
   - Test naming consistency
   - Flaky test candidates (tests ที่อาจ fail สุ่ม)

3. **Code smells:**
   - TODO/FIXME/HACK comments
   - Dead code (unused functions/imports)
   - Duplicate code patterns
   - Deep nesting (> 4 levels)

4. **Dependency health:**
   - จำนวน dependencies
   - Outdated packages

สรุปเป็น report card: A-F rating per category
```

---

**QA-24.** สร้าง quality gate checklist
> checklist สำหรับ PR merge

```
สร้าง quality gate checklist สำหรับ PR ของโปรเจกต์นี้:
ดูจาก:
- CLAUDE.md (coding conventions)
- CI pipeline (ถ้ามี)
- Test patterns ที่ใช้

สร้าง checklist:
1. **Code Quality**
   - [ ] ผ่าน linter
   - [ ] ไม่มี TODO/FIXME ใหม่ที่ไม่มี ticket
   - [ ] ...

2. **Testing**
   - [ ] มี unit tests สำหรับ code ใหม่
   - [ ] tests ผ่านทุกตัว
   - [ ] ...

3. **Security**
   - [ ] ไม่มี hardcoded secrets
   - [ ] input validation ครบ
   - [ ] ...

4. **Documentation**
   - [ ] API docs updated (ถ้าเปลี่ยน API)
   - [ ] CLAUDE.md updated (ถ้าเปลี่ยน conventions)
   - [ ] ...
```

---

**QA-25.** สรุปผล QA สำหรับ release
> QA sign-off report

```
สร้าง QA summary report สำหรับ release [version]:
1. อ่าน git log ตั้งแต่ [last release]
2. อ่าน test results
3. สรุป:

**Release: [version]**
**Date: [date]**

| Category | Status | Details |
|----------|--------|---------|
| Unit Tests | ✅/❌ | x passed, y failed |
| Integration Tests | ✅/❌ | ... |
| Security Scan | ✅/❌ | ... |
| Performance | ✅/❌ | ... |
| Code Review | ✅/❌ | all PRs reviewed? |

**Known Issues:** (ถ้ามี)
**Risk Assessment:** Low / Medium / High
**Recommendation:** Go / No-Go
```

---

**QA-26.** เปรียบเทียบ quality ระหว่าง versions
> track quality trends

```
เปรียบเทียบ code quality ระหว่าง [branch/tag A] กับ [branch/tag B]:
1. จำนวน files/lines เพิ่ม-ลด
2. Test coverage เปลี่ยนไงบ้าง
3. TODO/FIXME จำนวนเพิ่ม-ลด
4. Dependencies เพิ่ม-ลด
5. Code complexity เปลี่ยนไหม (ไฟล์ใหญ่ขึ้น/เล็กลง)
สรุปเป็น: ✅ ดีขึ้น / ⚠️ เท่าเดิม / ❌ แย่ลง พร้อมเหตุผล
```

---

> **ดูเพิ่มเติม:**
> - [02_developer_workflows.md](02_developer_workflows.md) — prompts สำหรับเขียน tests (DEV section 2.4)
> - [06_devops_and_infrastructure.md](06_devops_and_infrastructure.md) — CI/CD pipeline ที่ run tests อัตโนมัติ
> - [examples/07_prompt_catalog.md](../examples/07_prompt_catalog.md) — ตัวอย่าง QA prompts เฉพาะโปรเจกต์ ShopFast (Q1-Q7)
