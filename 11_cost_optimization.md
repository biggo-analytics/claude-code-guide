# บทที่ 11: การจัดการต้นทุนและประสิทธิภาพ

> Last updated: 2026-03-11 | Claude Code compatible version: latest

คู่มือเชิงปฏิบัติสำหรับการจัดการค่าใช้จ่ายในการใช้งาน Claude Code อย่างมีประสิทธิภาพ
ครอบคลุมตั้งแต่การเข้าใจ pricing model, การเลือก model ให้เหมาะกับงาน, การจัดการ session,
การเขียน prompt ที่ประหยัด ไปจนถึงการวางแผนงบประมาณสำหรับทีม

---

## สารบัญ

- [11.1 เข้าใจ Pricing Model](#111-เข้าใจ-pricing-model)
- [11.2 เลือก Model ให้เหมาะกับงาน](#112-เลือก-model-ให้เหมาะกับงาน)
- [11.3 Session Management เพื่อประหยัด Token](#113-session-management-เพื่อประหยัด-token)
- [11.4 Prompt Optimization](#114-prompt-optimization)
- [11.5 Team Budget Planning](#115-team-budget-planning)
- [11.6 เมื่อไรไม่ควรใช้ Claude Code](#116-เมื่อไรไม่ควรใช้-claude-code)

---

## 11.1 เข้าใจ Pricing Model

### Token Counting Basics

Token คือหน่วยพื้นฐานที่ Claude ใช้ในการประมวลผลข้อความ โดยมีหลักการนับดังนี้:

- **1 token** ประมาณ 3-4 ตัวอักษรภาษาอังกฤษ หรือ 1-2 ตัวอักษรภาษาไทย
- **ภาษาไทยใช้ token มากกว่าภาษาอังกฤษ** — เขียน prompt เป็นภาษาอังกฤษจะประหยัดกว่า
- **โค้ดนับเป็น token ด้วย** — ทั้งโค้ดที่ Claude อ่านและโค้ดที่ Claude สร้าง
- **System prompt + CLAUDE.md** ถูกส่งเป็น input token ทุกครั้งที่มี interaction

### Input vs Output Tokens

| ประเภท | คืออะไร | ค่าใช้จ่าย |
|--------|---------|------------|
| **Input tokens** | ข้อความที่คุณส่ง + context ทั้งหมด (history, CLAUDE.md, ไฟล์ที่อ่าน) | ถูกกว่า |
| **Output tokens** | ข้อความที่ Claude ตอบกลับ + โค้ดที่ Claude สร้าง | **แพงกว่า 3-5 เท่า** |

> **สิ่งสำคัญ:** output token แพงกว่า input token อย่างมาก ดังนั้นการขอให้ Claude เขียนโค้ดยาวๆ
> ที่ไม่จำเป็นจะทำให้ค่าใช้จ่ายสูงขึ้นอย่างรวดเร็ว

### ราคาต่อ Model (ประมาณการ ณ มีนาคม 2026)

| Model | Input (per 1M tokens) | Output (per 1M tokens) | Context Window | จุดเด่น |
|-------|----------------------|------------------------|----------------|---------|
| **Haiku** | ~$0.25 | ~$1.25 | 200K | เร็ว ถูก เหมาะกับงานง่าย |
| **Sonnet** | ~$3.00 | ~$15.00 | 200K | สมดุลระหว่างคุณภาพและราคา |
| **Opus** | ~$15.00 | ~$75.00 | 200K | ฉลาดที่สุด เหมาะกับงานซับซ้อน |

> **หมายเหตุ:** ราคาอาจเปลี่ยนแปลง ตรวจสอบราคาล่าสุดได้ที่
> [anthropic.com/pricing](https://anthropic.com/pricing)

### Context Window ส่งผลต่อ Cost อย่างไร

ยิ่ง session ยาวขึ้น ค่าใช้จ่ายจะเพิ่มแบบ **ทวีคูณ (quadratic)** ไม่ใช่แบบเส้นตรง:

```
Turn 1: ส่ง prompt 500 tokens          → จ่าย input 500 tokens
Turn 2: ส่ง prompt + history 1,500     → จ่าย input 1,500 tokens
Turn 3: ส่ง prompt + history 3,000     → จ่าย input 3,000 tokens
Turn 4: ส่ง prompt + history 5,000     → จ่าย input 5,000 tokens
...
Turn 10: ส่ง prompt + history 20,000+  → จ่าย input 20,000+ tokens
```

ทุก turn จะส่ง **conversation history ทั้งหมด** กลับไปเป็น input tokens ดังนั้น:
- Turn ที่ 10 จ่ายแพงกว่า turn ที่ 1 หลายสิบเท่า
- session ยาว 50 turns อาจมีค่าใช้จ่ายรวมเทียบเท่า session สั้น 10 ครั้ง

### /cost Command — ตรวจสอบค่าใช้จ่าย Real-time

```bash
# ระหว่าง session ให้พิมพ์
/cost

# ผลลัพธ์ตัวอย่าง:
# Session cost: $0.45
# Input tokens: 125,000
# Output tokens: 8,500
# Model: claude-sonnet-4-20250514
```

**แนะนำ:** ตรวจ `/cost` ทุก 10-15 นาที หรือหลังจบ task ใหญ่แต่ละอัน

---

## 11.2 เลือก Model ให้เหมาะกับงาน

### ตารางเลือก Model ตามประเภทงาน

| งาน | Model ที่แนะนำ | เหตุผล | ประมาณ cost/task |
|-----|---------------|--------|-----------------|
| เขียน commit message | Haiku | งาน routine ที่มี pattern ชัดเจน | $0.001-0.005 |
| แก้ formatting / lint | Haiku | mechanical task ไม่ต้องคิดเยอะ | $0.002-0.01 |
| เขียน doc comment | Haiku | อ่านโค้ดแล้วสรุป ไม่ซับซ้อน | $0.005-0.02 |
| Simple refactoring (rename) | Haiku | find & replace ที่ต้องเข้าใจ scope | $0.005-0.02 |
| ตอบคำถามง่ายๆ เกี่ยวกับโค้ด | Haiku | Q&A ที่ไม่ต้องวิเคราะห์ลึก | $0.002-0.01 |
| สร้าง CRUD endpoint | Sonnet | ต้องเข้าใจ pattern และ convention | $0.05-0.20 |
| เขียน unit test | Sonnet | ต้องคิด edge case และ mock | $0.05-0.15 |
| Debug error ทั่วไป | Sonnet | ต้องวิเคราะห์ stack trace + context | $0.10-0.30 |
| Code review | Sonnet | ต้องเข้าใจ best practice | $0.05-0.20 |
| สร้าง feature ใหม่ | Sonnet | ต้องเขียนโค้ดหลายไฟล์ | $0.10-0.50 |
| React component + hook | Sonnet | ต้องจัดการ state และ TypeScript types | $0.05-0.25 |
| Architecture design | **Opus** | ต้องมองภาพรวมและ trade-offs | $0.50-2.00 |
| Complex debugging (race condition) | **Opus** | ต้องวิเคราะห์ลึกหลายชั้น | $0.50-3.00 |
| Security audit | **Opus** | ต้องคิดถึง attack vectors ทั้งหมด | $0.50-2.50 |
| Major refactoring | **Opus** | ต้องเข้าใจ dependency graph ทั้งระบบ | $1.00-5.00 |
| System design / ER design | **Opus** | ต้องพิจารณา scalability, normalization | $0.50-3.00 |

### Flowchart เลือก Model

```
งานที่จะทำ
    │
    ├── ต้องคิดเชิงสถาปัตยกรรม / วิเคราะห์เชิงลึก?
    │   └── ✓ → Opus
    │         เช่น: architecture design, security audit,
    │              complex debugging, system design
    │
    ├── งาน development ทั่วไป / ต้องเขียนโค้ดใหม่?
    │   └── ✓ → Sonnet
    │         เช่น: feature implementation, CRUD, unit test,
    │              code review, debugging ทั่วไป
    │
    └── งานง่ายๆ / repetitive / mechanical?
        └── ✓ → Haiku
              เช่น: formatting, doc comments, rename,
                   commit messages, simple Q&A
```

### วิธีเปลี่ยน Model ระหว่าง Session

```bash
# เปลี่ยน model ระหว่าง session
/model sonnet
/model opus
/model haiku

# ระบุ model ตั้งแต่เริ่ม session
claude --model haiku

# ใน CLAUDE.md กำหนด default model
# (ใส่ใน .claude/settings.json)
```

**เทคนิค:** เริ่ม session ด้วย Haiku สำหรับงานง่ายๆ แล้วสลับไป Sonnet เมื่อต้องเขียนโค้ดจริง
ไม่จำเป็นต้องเปิด session ใหม่

---

## 11.3 Session Management เพื่อประหยัด Token

### เมื่อไรควร /compact

สัญญาณที่บอกว่า context ใหญ่เกินไปและควรใช้ `/compact`:

| สัญญาณ | สิ่งที่เกิดขึ้น | ควรทำอะไร |
|--------|----------------|----------|
| Claude เริ่มลืมสิ่งที่คุยกันก่อนหน้า | context window เกือบเต็ม | `/compact` ทันที |
| `/cost` แสดง input tokens สูงผิดปกติ | history สะสมเยอะ | `/compact` |
| Claude ตอบช้าลงอย่างเห็นได้ชัด | ประมวลผล context ใหญ่ | `/compact` |
| ผ่านไป 20+ turns แล้ว | history สะสมจาก conversation | `/compact` |
| `/cost` เกิน budget ที่ตั้งไว้ต่อ session | ค่าใช้จ่ายสะสม | `/compact` หรือ session ใหม่ |

```bash
# compact พร้อมบอก Claude ว่าจะทำอะไรต่อ
/compact ต่อไปจะทำ pagination สำหรับ product listing
```

### เมื่อไรควรเริ่ม Session ใหม่

| สถานการณ์ | เริ่ม session ใหม่ | ใช้ /compact | เหตุผล |
|----------|------------------|-------------|--------|
| เปลี่ยน task ที่ไม่เกี่ยวกัน | ✓ | | context เก่าไม่มีประโยชน์ แต่เปลือง token |
| จบ feature หนึ่ง เริ่มอีก feature | ✓ | | ลด context ที่ไม่จำเป็น |
| debug ไม่สำเร็จ ลองแนวทางใหม่ | ✓ | | context เดิมอาจ bias Claude |
| ทำ task เดิมต่อ แต่ context ยาว | | ✓ | ยังต้องการ context เดิมบางส่วน |
| กลับมาทำงานต่อหลังพักเบรก | ✓ | | เริ่มต้นสะอาด ประหยัดกว่า |

### claude -c vs claude --resume vs New Session

| คำสั่ง | ทำอะไร | เมื่อไรควรใช้ | ผลต่อ cost |
|--------|--------|-------------|-----------|
| `claude` | เริ่ม session ใหม่ทั้งหมด | เริ่ม task ใหม่ เปลี่ยนเรื่อง | ถูกที่สุด (เริ่มจาก 0) |
| `claude -c` | ต่อ session ล่าสุด | กลับมาทำงานเดิมต่อ | ส่ง history ทั้งหมด |
| `claude --resume` | เลือก session เก่าจากรายการ | กลับไปหา session เฉพาะ | ส่ง history ของ session นั้น |

> **กฎง่ายๆ:** ถ้าไม่แน่ใจ ให้เริ่ม session ใหม่เสมอ ถูกกว่าและสะอาดกว่า

### ใช้ /clear หลังจบ Task ใหญ่

ถ้ายังอยู่ใน session เดิมแต่จะเปลี่ยน task:

```bash
# จบ task เรื่อง auth แล้ว จะไปทำ product listing
/clear

# เริ่ม task ใหม่ใน session เดิม — context เริ่มจาก 0
อ่าน internal/handler/product_handler.go แล้วสร้าง endpoint สำหรับ product listing พร้อม pagination
```

### ตัวอย่าง: วัน Typical ของ Developer กับ Session Management

```
09:00  เปิด session ใหม่ (claude)
       → ทำ feature: user registration API
       → 15 turns, ใช้ Sonnet
       → /cost → $0.35
       → จบ feature, commit code

09:45  /clear (ล้าง context ใน session เดิม)
       → เขียน unit test สำหรับ feature ที่เพิ่งทำ
       → 8 turns, ยังคงใช้ Sonnet
       → /cost → $0.15 (นับจาก clear)

10:15  เปิด session ใหม่ (claude)
       → /model haiku
       → แก้ lint errors 5 ไฟล์
       → 3 turns
       → /cost → $0.02

10:30  เปิด session ใหม่ (claude)
       → Debug payment integration (ปัญหาซับซ้อน)
       → /model opus (ต้องวิเคราะห์ลึก)
       → 10 turns
       → /compact (turn ที่ 10 context เริ่มใหญ่)
       → อีก 5 turns
       → /cost → $2.10

12:00  พักเที่ยง

13:00  เปิด session ใหม่ (claude)
       → งาน code review PR ของเพื่อนร่วมทีม
       → ใช้ Sonnet
       → /cost → $0.20

รวมค่าใช้จ่ายทั้งวัน: ~$2.82
```

---

## 11.4 Prompt Optimization

### ให้ Context เฉพาะที่จำเป็น

Claude Code อ่านไฟล์ได้เอง แต่ถ้าบอกให้อ่านทุกไฟล์ จะเปลือง token มาก:

| แนวทาง | ตัวอย่าง | ผลต่อ cost |
|--------|---------|-----------|
| **ชี้ไปที่ไฟล์เฉพาะ** | "อ่าน `internal/handler/product_handler.go`" | ประหยัด |
| **ระบุ function ที่เกี่ยวข้อง** | "ดู function `CreateProduct` ใน product_handler.go" | ประหยัดมาก |
| **บอกให้อ่านทุกไฟล์** | "อ่านทุกไฟล์ในโปรเจกต์" | แพงมาก |
| **ไม่ให้ context เลย** | "สร้าง API endpoint" | Claude ต้องค้นหาเอง อาจแพงกว่า |

### ใช้ CLAUDE.md ลดการอธิบายซ้ำ

สิ่งที่ควรใส่ใน CLAUDE.md เพื่อไม่ต้องพิมพ์ซ้ำทุก session:

```markdown
# CLAUDE.md
## Project Conventions
- ใช้ Go Fiber framework
- Handler pattern: internal/handler/<resource>_handler.go
- Service pattern: internal/service/<resource>_service.go
- Repository pattern: internal/repository/<resource>_repository.go
- Error handling: ใช้ custom error types ใน pkg/errors/
- Testing: ใช้ testify/assert, mock ด้วย mockery

## Code Style
- Go: follow project .golangci.yml
- React: functional components + hooks, TypeScript strict mode
- SQL: snake_case, timestamps ทุกตาราง (created_at, updated_at)
```

**ผลลัพธ์:** แทนที่จะพิมพ์ convention ทุก session (เช่น 200 tokens x 20 sessions = 4,000 tokens)
CLAUDE.md ถูกโหลดอัตโนมัติแต่ **ไม่ต้องพิมพ์ซ้ำในทุก prompt**

### One-shot vs Multi-turn Trade-offs

| แนวทาง | ข้อดี | ข้อเสีย | เหมาะกับ |
|--------|------|--------|---------|
| **One-shot** (prompt เดียวจบ) | ไม่มี history สะสม ถูกกว่า | ต้องเขียน prompt ละเอียดมาก อาจได้ผลไม่ตรง | งานที่ชัดเจน มี pattern เดิม |
| **Multi-turn** (ค่อยๆ refine) | ปรับแต่งได้ทีละนิด ผลลัพธ์แม่นยำกว่า | history สะสม ยิ่งเยอะยิ่งแพง | งานที่ต้อง iterate เช่น design, debug |

**กฎทั่วไป:** ถ้าเขียน prompt ให้ชัดเจนตั้งแต่แรก one-shot จะถูกกว่าเสมอ

### Prompt Length Sweet Spot

| ความยาว prompt | เหมาะกับ | หมายเหตุ |
|---------------|---------|---------|
| 1-3 บรรทัด | งานง่ายที่มี context ชัดเจน | "อ่าน X แล้วทำ Y" |
| 3-8 บรรทัด | งาน development ส่วนใหญ่ | ระบุ input/output/constraint |
| 8-15 บรรทัด | งานที่ต้องการ spec ละเอียด | ระบุ requirements ครบถ้วน |
| 15+ บรรทัด | ไม่แนะนำ — ใช้ spec file แทน | เขียนเป็นไฟล์ .md แล้วให้ Claude อ่าน |

### ตัวอย่าง: Prompt ที่แพง vs Prompt ที่ประหยัด

**งาน:** สร้าง API endpoint สำหรับ order management

**Prompt ที่แพง (ประมาณ $0.50-1.00):**
```
อ่านทุกไฟล์ในโปรเจกต์แล้วสร้าง API endpoint สำหรับจัดการ order
ต้องมี CRUD ทั้งหมด ทำให้เหมือนกับ endpoint อื่นในโปรเจกต์
ดูทุก handler, service, repository แล้วทำตาม pattern เดียวกัน
```

**ทำไมถึงแพง:**
- "อ่านทุกไฟล์" → Claude อ่านไฟล์ทุกไฟล์ = input tokens พุ่ง
- ไม่ระบุ pattern ชัดเจน → Claude ต้องค้นหาเอง = หลาย turns
- กว้างเกินไป → อาจได้ output ที่ไม่ตรงใจ ต้อง iterate

**Prompt ที่ประหยัด (ประมาณ $0.10-0.20):**
```
อ่าน internal/handler/product_handler.go และ internal/service/product_service.go
แล้วสร้าง order handler และ service ตาม pattern เดียวกัน

Order fields: id, user_id, items (jsonb), total_price, status, created_at, updated_at
Endpoints: POST /api/v1/orders, GET /api/v1/orders/:id, GET /api/v1/orders (list with pagination)
Status enum: pending, confirmed, shipped, delivered, cancelled
```

**ทำไมถึงถูก:**
- ชี้ไปที่ 2 ไฟล์เฉพาะ → input tokens น้อย
- ระบุ fields และ endpoints ชัดเจน → Claude ทำได้ใน 1-2 turns
- ผลลัพธ์ตรงใจตั้งแต่แรก → ไม่ต้อง iterate

---

## 11.5 Team Budget Planning

### ประมาณการ Token ต่อ Task Type

| ประเภทงาน | Input Tokens (avg) | Output Tokens (avg) | Cost (Sonnet) | Cost (Opus) |
|-----------|-------------------|---------------------|---------------|-------------|
| Simple Q&A | 2,000-5,000 | 500-1,000 | $0.01-0.02 | $0.05-0.10 |
| Commit message | 3,000-8,000 | 200-500 | $0.01-0.03 | $0.05-0.10 |
| Unit test (1 function) | 10,000-30,000 | 2,000-5,000 | $0.05-0.15 | $0.20-0.50 |
| CRUD endpoint | 15,000-50,000 | 5,000-15,000 | $0.10-0.40 | $0.50-1.50 |
| Feature implementation | 30,000-100,000 | 10,000-30,000 | $0.20-0.75 | $1.00-3.00 |
| Debug session | 20,000-80,000 | 5,000-20,000 | $0.10-0.50 | $0.50-2.00 |
| Code review (1 PR) | 15,000-40,000 | 3,000-8,000 | $0.05-0.25 | $0.30-1.00 |
| Architecture design | 30,000-100,000 | 10,000-30,000 | $0.20-0.75 | $1.00-3.00 |

### Budget Allocation ต่อ Role

| Role | การใช้งานหลัก | Model ที่ใช้บ่อย | ประมาณ cost/วัน | ประมาณ cost/เดือน |
|------|-------------|----------------|----------------|-----------------|
| **Senior Dev** | Feature dev, debugging, review | Sonnet + Opus | $3-8 | $60-160 |
| **Junior Dev** | CRUD, unit test, Q&A | Sonnet + Haiku | $2-5 | $40-100 |
| **SA** | Architecture, design review | Opus + Sonnet | $2-5 | $40-100 |
| **QA** | Test generation, review | Sonnet + Haiku | $1-3 | $20-60 |
| **BA** | Spec review, documentation | Haiku + Sonnet | $0.50-2 | $10-40 |
| **PM** | Status check, report | Haiku | $0.20-1 | $4-20 |

### Monthly Cost Projection — สูตรคำนวณ

```
Monthly Cost = จำนวนคน x ค่าเฉลี่ยต่อวันต่อคน x วันทำงานต่อเดือน

ตัวอย่าง:
  3 Devs x $5/day x 22 days    = $330
  1 SA   x $3/day x 22 days    = $66
  1 QA   x $2/day x 22 days    = $44
                          รวม   = $440/เดือน

  + Buffer 20% สำหรับงานไม่คาดคิด = $88
                     รวมทั้งหมด   = $528/เดือน
```

### Alert & Cap Strategies

| กลยุทธ์ | วิธีการ | เหมาะกับ |
|---------|--------|---------|
| **Daily cap per person** | กำหนด budget สูงสุดต่อวัน (เช่น $10/day) | ป้องกันใช้เกิน |
| **Weekly review** | PM ตรวจสอบ usage report ทุกสัปดาห์ | ติดตาม trend |
| **Model restriction** | จำกัด Opus ให้เฉพาะ SA/Senior Dev | ควบคุม cost per task |
| **Anthropic dashboard** | ตั้ง alert ใน Console เมื่อ spending ถึง threshold | แจ้งเตือนอัตโนมัติ |
| **Team policy** | กำหนด guidelines ว่างานไหนใช้ model ไหน | สร้างวินัยการใช้งาน |

### ROI Calculation

```
ROI = (เวลาที่ประหยัดได้ x ค่าแรงต่อชั่วโมง) - ค่า Claude Code
                        ค่า Claude Code

ตัวอย่าง: Developer ใช้ Claude Code สร้าง CRUD endpoint
  - ทำเอง: 3 ชั่วโมง
  - ใช้ Claude Code: 30 นาที (รวมเวลา review + แก้ไข)
  - เวลาที่ประหยัด: 2.5 ชั่วโมง
  - ค่าแรง developer: 500 บาท/ชั่วโมง
  - มูลค่าเวลาที่ประหยัด: 2.5 x 500 = 1,250 บาท
  - ค่า Claude Code (Sonnet, 1 session): ~$0.30 ≈ 11 บาท
  - ROI: (1,250 - 11) / 11 = 11,263%
```

> **สรุป:** แม้ค่า Claude Code จะดูเป็นต้นทุนเพิ่ม แต่เมื่อเทียบกับเวลาที่ประหยัดได้
> ROI สูงมากในเกือบทุกกรณี

### ตัวอย่าง: ทีม 5 คน ใช้ Claude Code เต็มรูปแบบ

```
ทีม ShopFast E-commerce (Go/React/PostgreSQL):

สมาชิก:
  2 x Senior Dev (Sonnet 70% + Opus 30%)   → $6/day each → $264/month
  1 x Junior Dev (Sonnet 80% + Haiku 20%)  → $3/day      → $66/month
  1 x SA/QA (Opus 40% + Sonnet 60%)        → $4/day      → $88/month
  1 x BA/PM (Haiku 60% + Sonnet 40%)       → $1.50/day   → $33/month

รวม:        $451/เดือน
+ Buffer:   $90 (20%)
ทั้งหมด:    ~$541/เดือน (~19,000 บาท)

เทียบกับ: ค่าแรง developer 1 คน/เดือน = ~60,000-120,000 บาท
ถ้า Claude Code ช่วยเพิ่ม productivity 20% ของทีม 5 คน
= ประหยัดได้ ~1 FTE = 60,000-120,000 บาท/เดือน
```

---

## 11.6 เมื่อไรไม่ควรใช้ Claude Code

### ข้อจำกัดที่ควรรู้

Claude Code ไม่ได้เหมาะกับทุกงาน บางกรณีเครื่องมืออื่นทำได้ดีกว่า เร็วกว่า และฟรี:

| สถานการณ์ | ใช้ Claude Code | ใช้เครื่องมืออื่น | เหตุผล |
|----------|:--------------:|:----------------:|--------|
| Find & replace ข้ามไฟล์ | | ✓ `grep`/`sed` | เร็วกว่า ฟรี แม่นยำ 100% |
| จัดการ credentials / signing keys | | ✓ vault, manual | Claude ไม่ควรเห็น secrets |
| แก้ไข production database | | ✓ migration tool | อันตรายเกินไป ต้องมี audit trail |
| Repetitive mechanical tasks (1000 files) | | ✓ script/automation | `for` loop เร็วกว่าและถูกกว่า |
| Real-time debugging (breakpoint) | | ✓ delve, Chrome DevTools | Claude ไม่ได้ attach debugger |
| Generate ไฟล์ขนาดใหญ่ (1000+ lines) | | ✓ template/code generator | Claude อาจ truncate หรือ hallucinate |
| ดึงข้อมูล real-time (API, web) | | ✓ curl, Postman, browser | Claude ไม่ access internet ระหว่าง coding |
| ย้าย/rename ไฟล์จำนวนมาก | | ✓ `mv`, script | `mv` เร็วกว่าและไม่ผิดพลาด |
| สร้าง boilerplate ซ้ำๆ | ✓ (ครั้งแรก) | ✓ template (ครั้งต่อไป) | ให้ Claude สร้าง template แล้วใช้ template ต่อ |
| เข้าใจ codebase ใหม่ | ✓ | | Claude อ่านและสรุปได้ดี |
| วิเคราะห์ design pattern | ✓ | | ต้องการความเข้าใจเชิงลึก |
| เขียน test ที่ต้องคิด edge case | ✓ | | Claude คิด edge case ได้ดี |
| Refactor ที่ต้องเข้าใจ business logic | ✓ | | ต้องเข้าใจ context ไม่ใช่แค่ text replace |

### คำแนะนำเชิงปฏิบัติ

**ใช้ Claude Code เมื่อ:**
- งานต้องการ **ความเข้าใจ** (understanding) ไม่ใช่แค่ **การแปลง** (transformation)
- งานมี **ความซับซ้อน** ที่ทำด้วยมือจะเสียเวลามาก
- งานต้อง **สร้างโค้ดใหม่** ที่ต้องเข้าใจ pattern และ convention

**ใช้เครื่องมืออื่นเมื่อ:**
- งาน **deterministic** — ผลลัพธ์ต้องเหมือนกัน 100% ทุกครั้ง
- งานเกี่ยวกับ **security-sensitive data**
- งาน **mechanical** ที่ทำซ้ำโดยไม่ต้องคิด
- ต้องการ **real-time interaction** กับระบบที่กำลังทำงาน

---

## การอ้างอิงข้ามบท

| บท | หัวข้อที่เกี่ยวข้อง | ลิงก์ |
|----|-------------------|------|
| บทที่ 1 | พื้นฐาน Claude Code, คำสั่ง /cost, /compact, /clear | [01_intro.md](./01_intro.md) |
| บทที่ 2 | เทคนิค Prompt ที่มีประสิทธิภาพ, Agentic workflow | [02_agentic_mastery.md](./02_agentic_mastery.md) |
| บทที่ 3 | Multi-repo workflow, CI/CD integration | [03_advanced_usage.md](./03_advanced_usage.md) |
| บทที่ 8 | บทบาทของแต่ละตำแหน่ง (ใช้ร่วมกับ budget allocation) | [08_roles_responsibilities.md](./08_roles_responsibilities.md) |
| บทที่ 9 | Cheat Sheet — คำสั่งทั้งหมดแบบย่อ | [09_cheat_sheet.md](./09_cheat_sheet.md) |
| บทที่ 10 | Troubleshooting — แก้ปัญหาที่พบบ่อย | [10_troubleshooting_faq.md](./10_troubleshooting_faq.md) |

---

> **สรุป**: การจัดการต้นทุน Claude Code ไม่ใช่แค่เรื่องของเงิน แต่เป็นเรื่องของ **การใช้เครื่องมือให้ถูกที่ถูกเวลา**
> เลือก model ให้ตรงกับความซับซ้อนของงาน จัดการ session ไม่ให้ context บวม เขียน prompt ให้ชัดเจน
> และรู้ว่าเมื่อไรควรใช้เครื่องมืออื่นแทน ทีมที่มีวินัยในการใช้งานจะได้ ROI สูงสุดจาก Claude Code

---

← [บทที่ 10: Troubleshooting & FAQ](10_troubleshooting_faq.md) | [บทที่ 12: The Agency →](12_agency_agents_guide.md)
