# การพัฒนาแบบขับเคลื่อนด้วยสเปค

## Specification-Driven Development with Claude Code

---

> **บทที่ 5** -- คู่มือภายในสำหรับทีมพัฒนา
>
> บทนี้ครอบคลุมแนวทางการเขียนสเปค (Specification) เพื่อใช้ร่วมกับ Claude Code
> ตั้งแต่โครงสร้างเอกสาร, ตัวอย่างจริง, ไปจนถึง workflow การนำ spec ไปสู่ code

---

## สารบัญ

- [5.1 แนวคิด Spec-First Development](#51-แนวคิด-spec-first-development)
- [5.2 โครงสร้าง Specification](#52-โครงสร้าง-specification)
- [5.3 ตัวอย่าง Spec -- Simple Feature](#53-ตัวอย่าง-spec--simple-feature)
- [5.4 ตัวอย่าง Spec -- Complex Feature](#54-ตัวอย่าง-spec--complex-feature)
- [5.5 Workflow: จาก Spec สู่ Code ด้วย Claude Code](#55-workflow-จาก-spec-สู่-code-ด้วย-claude-code)
- [5.6 API-First Development](#56-api-first-development)
- [5.7 Test-Driven Specification](#57-test-driven-specification)
- [Cross-References](#cross-references)

---

## 5.1 แนวคิด Spec-First Development

### ปัญหาของ "Code First"

การพัฒนาแบบ "code first" คือการเริ่มเขียนโค้ดทันทีโดยไม่มีสเปค (Specification)
ที่ชัดเจน วิธีนี้มีปัญหาหลายประการ:

| ปัญหา | ผลกระทบ |
|--------|----------|
| ขอบเขตคลาดเคลื่อน (Scope Creep) | ฟีเจอร์ขยายเกินที่ตกลงกันไว้ ใช้เวลามากขึ้น |
| ความไม่สอดคล้อง (Inconsistency) | แต่ละคนเข้าใจ requirement ต่างกัน API ไม่เป็นมาตรฐาน |
| งานซ้ำซ้อน (Rework) | implement แล้วพบว่าไม่ตรง requirement ต้องทำใหม่ |
| ทดสอบได้ยาก | ไม่มีเกณฑ์ชัดเจนว่า "สำเร็จ" คืออะไร |

เมื่อใช้ AI อย่าง Claude Code ปัญหาเหล่านี้จะยิ่งทวีความรุนแรง เพราะ AI จะ "เดา"
ส่วนที่ขาดหายไปจากบริบท ทำให้ผลลัพธ์ไม่ตรงกับที่ต้องการ

### Spec-First Workflow

```
Spec  -->  Plan  -->  Implement  -->  Verify  -->  Iterate
 |          |            |              |             |
 BA       Claude       Claude         Tests        Claude
         Code          Code          + QA          Code
```

1. **Spec** -- BA (Business Analyst) หรือ Product Owner เขียนสเปคเป็น Markdown
2. **Plan** -- Developer ป้อน spec ให้ Claude Code วิเคราะห์และวางแผน
3. **Implement** -- Claude Code สร้างโค้ดตาม plan ทีละ layer
4. **Verify** -- รัน test suite เทียบกับ acceptance criteria (เกณฑ์การยอมรับ) ใน spec
5. **Iterate** -- หากไม่ผ่าน กลับไปแก้ไขและ verify ซ้ำ

### ข้อดีของ Spec-First

- **ความต้องการชัดเจน (Clear Requirements):** ทุกคนในทีมเข้าใจตรงกัน
- **ติดตามการเปลี่ยนแปลงได้ (Traceable Changes):** spec อยู่ใน git, review ผ่าน PR
- **ผลลัพธ์สม่ำเสมอ (Consistent Output):** Claude Code ทำงานได้แม่นยำเมื่อบริบทครบ
- **ทดสอบได้ (Testable):** acceptance criteria เป็นพื้นฐานของ test cases

### Spec ในบริบทของ Claude Code

ในการทำงานกับ Claude Code นั้น spec คือ structured prompt ที่ดีที่สุด
เพราะ Claude Code ประมวลผล Markdown ได้ดีเยี่ยม การเขียน spec เป็น Markdown
ที่มีโครงสร้างชัดเจนจึงเป็นวิธีสื่อสารที่มีประสิทธิภาพสูงสุดระหว่างทีมกับ AI

> **Tip:** เก็บ spec ไว้ในโฟลเดอร์ `docs/specs/` ของโปรเจกต์ เพื่อให้ Claude Code
> อ่านได้โดยตรงจาก codebase โดยไม่ต้อง copy-paste

---

## 5.2 โครงสร้าง Specification

### Template มาตรฐาน

ใช้โครงสร้างต่อไปนี้เป็น template สำหรับทุก feature spec:

````markdown
# Feature: [Feature Name]

## Objective
[1-2 ประโยคอธิบายเป้าหมายของ feature นี้]

## Technical Requirements
- Language/Framework: [เช่น Go 1.22, Fiber, GORM]
- Database: [เช่น PostgreSQL 15]
- Dependencies: [packages/modules ที่ต้องใช้]

## API Contract
### POST /api/v1/[resource]
**Request:**
```json
{
  "field1": "string",
  "field2": 123
}
```
**Response (201):**
```json
{
  "id": "uuid",
  "field1": "string",
  "created_at": "timestamp"
}
```
**Error Responses:**
- 400: validation error
- 409: duplicate resource
- 500: internal server error

## Business Rules
1. [Rule 1: คำอธิบาย]
2. [Rule 2: คำอธิบาย]
3. [Rule 3: คำอธิบาย]

## Edge Cases
- [Edge case 1]
- [Edge case 2]

## Acceptance Criteria
- [ ] [Criterion 1]
- [ ] [Criterion 2]
- [ ] [Criterion 3]

## Dependencies
- [Depends on Feature X]
- [Requires table Y to exist]
````

### คำอธิบายแต่ละส่วน

| ส่วน | จุดประสงค์ |
|------|-----------|
| **Objective** | สรุปสั้น ๆ ว่า feature ทำอะไร ทำไมถึงต้องมี |
| **Technical Requirements** | ระบุ stack ที่ใช้ เพื่อให้ Claude Code สร้างโค้ดถูก framework |
| **API Contract** | กำหนด request/response ชัดเจน รวม error codes |
| **Business Rules** | เงื่อนไขทางธุรกิจที่ต้อง implement |
| **Edge Cases** | สถานการณ์พิเศษที่ต้องรองรับ |
| **Acceptance Criteria** | checklist สำหรับ verify ว่า feature สมบูรณ์ |
| **Dependencies** | feature หรือ resource อื่นที่ต้องมีก่อน |

> **Warning:** อย่าข้าม Edge Cases และ Acceptance Criteria เพราะทั้งสองส่วนนี้
> เป็นสิ่งที่ทำให้ Claude Code สร้างโค้ดที่ robust มากขึ้น และเป็นพื้นฐานของ test cases

---

## 5.3 ตัวอย่าง Spec -- Simple Feature

### Feature: User Registration

````markdown
# Feature: User Registration

## Objective
ให้ผู้ใช้สามารถสมัครสมาชิกผ่าน API โดยใช้อีเมลและรหัสผ่าน
เพื่อเข้าถึงระบบได้อย่างปลอดภัย

## Technical Requirements
- Language/Framework: Go 1.22, Fiber v2
- ORM: GORM v2
- Database: PostgreSQL 15
- Dependencies:
  - `golang.org/x/crypto/bcrypt` สำหรับ hash password
  - `github.com/google/uuid` สำหรับสร้าง UUID
  - `github.com/go-playground/validator/v10` สำหรับ validation

## API Contract

### POST /api/v1/auth/register

**Request:**
```json
{
  "email": "user@example.com",
  "password": "SecurePass123!",
  "first_name": "สมชาย",
  "last_name": "ใจดี"
}
```

**Response (201 Created):**
```json
{
  "id": "550e8400-e29b-41d4-a716-446655440000",
  "email": "user@example.com",
  "first_name": "สมชาย",
  "last_name": "ใจดี",
  "created_at": "2026-03-10T10:30:00Z"
}
```

**Error Responses:**
- 400 Bad Request: ข้อมูลไม่ครบหรือรูปแบบไม่ถูกต้อง
```json
{
  "error": "validation_error",
  "message": "email: must be a valid email address"
}
```
- 409 Conflict: อีเมลซ้ำในระบบ
```json
{
  "error": "duplicate_email",
  "message": "email already registered"
}
```
- 500 Internal Server Error: ข้อผิดพลาดภายในระบบ

## Business Rules
1. **Email Validation:** อีเมลต้องอยู่ในรูปแบบ RFC 5322 ที่ถูกต้อง
2. **Password Strength:** รหัสผ่านต้องมีอย่างน้อย 8 ตัวอักษร ประกอบด้วย
   ตัวพิมพ์ใหญ่อย่างน้อย 1 ตัว, ตัวเลขอย่างน้อย 1 ตัว, อักขระพิเศษอย่างน้อย 1 ตัว
3. **Duplicate Check:** ตรวจสอบว่าอีเมลยังไม่ถูกใช้งานในระบบ
4. **Password Hashing:** เก็บ password เป็น bcrypt hash เท่านั้น ห้ามเก็บ plaintext
5. **UUID Generation:** สร้าง UUID v4 สำหรับ user ID

## Edge Cases
- ส่งอีเมลที่มีช่องว่างด้านหน้า/ท้าย -> trim ก่อน validate
- ส่งอีเมลตัวพิมพ์ใหญ่ -> แปลงเป็นตัวเล็กก่อนบันทึก
- ส่ง password ที่มีแต่ช่องว่าง -> reject
- ส่ง first_name / last_name ที่มีอักขระพิเศษ (เช่น ภาษาไทย) -> อนุญาต
- ส่ง request ซ้ำพร้อมกัน (concurrent) ด้วยอีเมลเดียวกัน -> ป้องกัน duplicate

## Acceptance Criteria
- [ ] POST /api/v1/auth/register สร้าง user ใหม่ได้สำเร็จ
- [ ] Response มี id, email, first_name, last_name, created_at
- [ ] Password ไม่อยู่ใน response
- [ ] Password ถูก hash ด้วย bcrypt ในฐานข้อมูล
- [ ] อีเมลที่ซ้ำ return 409 Conflict
- [ ] อีเมลรูปแบบผิด return 400 Bad Request
- [ ] Password ที่ไม่ผ่านเกณฑ์ return 400 Bad Request
- [ ] ช่องที่จำเป็นไม่ครบ return 400 Bad Request

## Dependencies
- ตาราง `users` ต้องมีอยู่ในฐานข้อมูล (สร้างผ่าน migration)
- Migration file สำหรับสร้างตาราง users
````

> **Note:** สังเกตว่า spec นี้มีรายละเอียดเพียงพอที่ Claude Code จะสร้าง model,
> repository, service, handler และ test ได้ครบถ้วนโดยไม่ต้องถามเพิ่ม

---

## 5.4 ตัวอย่าง Spec -- Complex Feature

### Feature: Order Processing with Payment

````markdown
# Feature: Order Processing with Payment

## Objective
ระบบจัดการคำสั่งซื้อ (Order) ที่รองรับการชำระเงิน (Payment)
ตั้งแต่สร้างคำสั่งซื้อ ตรวจสอบสินค้าคงคลัง ประมวลผลการชำระเงิน
จนถึงอัพเดทสถานะคำสั่งซื้อ

## Technical Requirements
- Language/Framework: Go 1.22, Fiber v2
- ORM: GORM v2
- Database: PostgreSQL 15
- Message Queue: RabbitMQ (สำหรับ async processing)
- Dependencies:
  - Payment Gateway SDK
  - Notification Service client

## API Contracts

### POST /api/v1/orders
สร้างคำสั่งซื้อใหม่

**Request:**
```json
{
  "items": [
    { "product_id": "uuid", "quantity": 2 },
    { "product_id": "uuid", "quantity": 1 }
  ],
  "shipping_address_id": "uuid",
  "payment_method": "credit_card",
  "coupon_code": "SAVE10"
}
```

**Response (201 Created):**
```json
{
  "id": "uuid",
  "order_number": "ORD-20260310-0001",
  "status": "pending_payment",
  "items": [
    {
      "product_id": "uuid",
      "product_name": "สินค้า A",
      "quantity": 2,
      "unit_price": 500.00,
      "subtotal": 1000.00
    }
  ],
  "subtotal": 1500.00,
  "discount": 150.00,
  "shipping_fee": 50.00,
  "total": 1400.00,
  "created_at": "2026-03-10T10:30:00Z"
}
```

### POST /api/v1/orders/:id/pay
ชำระเงินสำหรับคำสั่งซื้อ

**Request:**
```json
{
  "payment_token": "tok_xxxxxxxxxxxx"
}
```

**Response (200 OK):**
```json
{
  "order_id": "uuid",
  "payment_id": "uuid",
  "status": "paid",
  "paid_at": "2026-03-10T10:35:00Z"
}
```

### GET /api/v1/orders/:id
ดูรายละเอียดคำสั่งซื้อ

**Response (200 OK):**
```json
{
  "id": "uuid",
  "order_number": "ORD-20260310-0001",
  "status": "paid",
  "items": [...],
  "total": 1400.00,
  "payment": {
    "id": "uuid",
    "method": "credit_card",
    "status": "completed",
    "paid_at": "2026-03-10T10:35:00Z"
  },
  "created_at": "2026-03-10T10:30:00Z",
  "updated_at": "2026-03-10T10:35:00Z"
}
```

### PATCH /api/v1/orders/:id/cancel
ยกเลิกคำสั่งซื้อ

**Response (200 OK):**
```json
{
  "id": "uuid",
  "status": "cancelled",
  "cancelled_at": "2026-03-10T11:00:00Z"
}
```

## Order Status State Machine

```
                    +-----------+
                    |  created  |
                    +-----+-----+
                          |
                    (inventory reserved)
                          |
                    +-----v---------+
                    | pending_payment|
                    +-----+---------+
                          |
              +-----------+-----------+
              |                       |
        (payment success)       (payment failed)
              |                       |
        +-----v-----+          +-----v--------+
        |   paid     |          | payment_failed|
        +-----+-----+          +-----+--------+
              |                       |
        (ship order)            (retry/cancel)
              |                       |
        +-----v-----+          +-----v-----+
        |  shipped   |          | cancelled  |
        +-----+-----+          +-----------+
              |
        (delivery confirmed)
              |
        +-----v-----+
        | completed  |
        +-----------+
```

## Business Rules
1. **Inventory Check:** ตรวจสอบสินค้าคงคลังก่อนสร้าง order
   ถ้าสินค้าไม่พอ -> reject ทั้ง order
2. **Inventory Reserve:** เมื่อสร้าง order สำเร็จ ต้อง reserve สินค้า
   ถ้า reserve ไม่สำเร็จ -> rollback order
3. **Coupon Validation:** ตรวจสอบ coupon code:
   ยังไม่หมดอายุ, ยังไม่ถูกใช้ครบจำนวน, ใช้ได้กับสินค้าที่สั่ง
4. **Payment Flow:** ส่ง payment request ไป payment gateway
   รอผลลัพธ์ (synchronous) แล้วอัพเดทสถานะ
5. **Status Transitions:** สถานะต้องเปลี่ยนตาม state machine เท่านั้น
   เช่น จาก "paid" ไป "cancelled" ได้ แต่จาก "shipped" ไป "cancelled" ไม่ได้
6. **Notification:** ส่ง notification เมื่อเปลี่ยนสถานะ
   (email สำหรับ paid, shipped, completed)
7. **Idempotency:** การชำระเงินต้อง idempotent
   ส่ง request ซ้ำด้วย payment_token เดิม ต้องไม่เรียกเก็บเงินซ้ำ

## Edge Cases
- สินค้าหมดระหว่างสร้าง order (race condition) -> ใช้ pessimistic lock
- ชำระเงินบางส่วน (partial payment) -> ไม่รองรับในเวอร์ชันแรก return 400
- Payment gateway timeout -> ตั้งสถานะเป็น "payment_failed" ให้ retry ได้
- ผู้ใช้สั่งซื้อพร้อมกันหลาย order (concurrent orders) -> อนุญาต แต่ตรวจ stock แยกกัน
- ยกเลิก order ที่ชำระเงินแล้ว -> ต้อง trigger refund flow
- coupon code ถูกใช้พร้อมกันเกิน limit -> ใช้ database lock ป้องกัน

## Integration Points
- **Payment Gateway:** POST /v1/charges (external API)
- **Notification Service:** publish event ไป RabbitMQ queue `order.status.changed`
- **Inventory Service:** internal gRPC call ไป inventory service

## Acceptance Criteria
- [ ] สร้าง order ใหม่ได้เมื่อสินค้าเพียงพอ
- [ ] สินค้าถูก reserve เมื่อสร้าง order สำเร็จ
- [ ] ชำระเงินสำเร็จเปลี่ยนสถานะเป็น "paid"
- [ ] ชำระเงินล้มเหลวเปลี่ยนสถานะเป็น "payment_failed"
- [ ] สถานะเปลี่ยนตาม state machine เท่านั้น
- [ ] ยกเลิก order ได้เฉพาะสถานะที่อนุญาต
- [ ] Coupon ถูกตรวจสอบและคำนวณส่วนลดถูกต้อง
- [ ] Notification ถูกส่งเมื่อเปลี่ยนสถานะ
- [ ] Payment idempotency ทำงานถูกต้อง
- [ ] Concurrent order ไม่ทำให้ stock ติดลบ

## Dependencies
- ตาราง `orders`, `order_items`, `payments` ต้องมีอยู่ในฐานข้อมูล
- Inventory Service ต้อง deploy แล้ว
- Payment Gateway credentials ต้องตั้งค่าใน environment
- RabbitMQ ต้อง running
````

> **Note:** spec ที่ซับซ้อนอย่างนี้จะช่วยให้ Claude Code เข้าใจ "ภาพรวม" ของ feature
> ทั้งหมด ลดโอกาสที่จะ implement ผิดพลาดในส่วนที่เชื่อมโยงกัน

---

## 5.5 Workflow: จาก Spec สู่ Code ด้วย Claude Code

### ขั้นตอนที่ 1: สร้าง/อ่าน Spec

BA สร้าง spec --> SA (Solution Architect) review --> ทีม approve ผ่าน PR

```
docs/
  specs/
    user-registration.md
    order-processing.md
    payment-integration.md
```

### ขั้นตอนที่ 2: Plan Mode

ใช้ Claude Code อ่าน spec แล้ววางแผน implementation:

```
อ่าน spec ใน docs/specs/user-registration.md
แล้ววางแผน implementation:
1. Database schema / migration
2. Repository layer
3. Service layer (business logic)
4. Handler layer (API endpoints)
5. Tests
```

Claude Code จะวิเคราะห์ spec แล้วเสนอแผนงานเป็นขั้นตอน ทีมสามารถ review
และปรับแผนก่อนเริ่ม implement ได้

> **Tip:** ใช้ Plan Mode (Shift+Tab สลับโหมด) เพื่อให้ Claude Code วิเคราะห์
> โดยไม่เขียนโค้ดทันที ดูรายละเอียดเพิ่มเติมในบทที่ 3

### ขั้นตอนที่ 3: Implement ทีละ Layer

สั่ง Claude Code implement ทีละ layer เพื่อควบคุมคุณภาพ:

**Layer 1 -- Model & Migration:**

```
implement ตาม plan ที่วางไว้ เริ่มจาก:
Step 1: สร้าง GORM model และ migration สำหรับตาราง users
(อ้างอิง spec ใน docs/specs/user-registration.md)
```

**Layer 2 -- Repository:**

```
Step 2: สร้าง repository layer สำหรับ users
- CreateUser
- GetUserByEmail
- GetUserByID
(อ้างอิง spec ใน docs/specs/user-registration.md)
```

**Layer 3 -- Service:**

```
Step 3: สร้าง service layer สำหรับ user registration
- implement business rules ทุกข้อตาม spec
- email validation, password strength, duplicate check
(อ้างอิง spec ใน docs/specs/user-registration.md)
```

**Layer 4 -- Handler:**

```
Step 4: สร้าง handler สำหรับ POST /api/v1/auth/register
- request validation
- call service
- response format ตาม spec
(อ้างอิง spec ใน docs/specs/user-registration.md)
```

### ขั้นตอนที่ 4: Test ตาม Spec

สร้าง test cases โดยอ้างอิง acceptance criteria ใน spec:

```
สร้าง test cases ตาม acceptance criteria ใน spec:
- ทดสอบ happy path: สร้าง user สำเร็จ
- ทดสอบ edge cases ที่ระบุใน spec: email trim, lowercase, concurrent
- ทดสอบ error responses: 400, 409, 500
```

### ขั้นตอนที่ 5: Verify & Iterate

ตรวจสอบ implementation เทียบกับ spec:

```
ตรวจสอบ implementation กับ spec:
1. ทุก API endpoint ตรงกับ spec หรือไม่
2. Business rules ครบหรือไม่
3. Edge cases ถูก handle หรือไม่
4. Test coverage เพียงพอหรือไม่
```

ถ้าพบส่วนที่ขาด ให้กลับไป implement เพิ่มแล้ว verify อีกครั้ง

> **Warning:** อย่าข้ามขั้นตอน Verify เด็ดขาด เพราะ Claude Code อาจ implement
> ได้ถูกต้องในระดับ syntax แต่อาจพลาด business logic บางข้อที่ระบุไว้ใน spec

---

## 5.6 API-First Development

### เริ่มจาก OpenAPI/Swagger Spec

แนวทาง API-First (การออกแบบ API ก่อน) คือการเขียน OpenAPI specification
ก่อนเริ่ม implement โค้ด ข้อดีคือทั้ง frontend และ backend ตกลง contract ร่วมกัน
ก่อนเริ่มเขียนโค้ด

ตัวอย่าง OpenAPI spec:

```yaml
openapi: 3.0.3
info:
  title: User Service API
  version: 1.0.0
paths:
  /api/v1/auth/register:
    post:
      summary: Register a new user
      requestBody:
        required: true
        content:
          application/json:
            schema:
              type: object
              required: [email, password, first_name, last_name]
              properties:
                email:
                  type: string
                  format: email
                password:
                  type: string
                  minLength: 8
                first_name:
                  type: string
                last_name:
                  type: string
      responses:
        '201':
          description: User created successfully
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/UserResponse'
        '400':
          description: Validation error
        '409':
          description: Email already exists
components:
  schemas:
    UserResponse:
      type: object
      properties:
        id:
          type: string
          format: uuid
        email:
          type: string
        first_name:
          type: string
        last_name:
          type: string
        created_at:
          type: string
          format: date-time
```

### ให้ Claude Code อ่าน OpenAPI Spec แล้วสร้างโค้ด

```
อ่าน OpenAPI spec ใน docs/api/openapi.yaml
แล้วสร้าง:
1. Go struct สำหรับ request/response
2. Fiber route definitions
3. Handler stubs
4. Validation middleware
```

ตัวอย่างโค้ดที่ Claude Code จะสร้าง:

```go
// สร้างจาก OpenAPI spec โดยอัตโนมัติ

// RegisterRequest -- request body สำหรับ POST /api/v1/auth/register
type RegisterRequest struct {
    Email     string `json:"email" validate:"required,email"`
    Password  string `json:"password" validate:"required,min=8"`
    FirstName string `json:"first_name" validate:"required"`
    LastName  string `json:"last_name" validate:"required"`
}

// UserResponse -- response body สำหรับ user endpoints
type UserResponse struct {
    ID        string    `json:"id"`
    Email     string    `json:"email"`
    FirstName string    `json:"first_name"`
    LastName  string    `json:"last_name"`
    CreatedAt time.Time `json:"created_at"`
}
```

### Sync Spec กับ Code

เมื่อ OpenAPI spec เปลี่ยน ให้ Claude Code อัพเดทโค้ดให้สอดคล้อง:

```
OpenAPI spec ใน docs/api/openapi.yaml ถูกอัพเดท:
- เพิ่ม field "phone" ใน register request (optional)
- เพิ่ม field "phone" ใน user response

อัพเดท code ให้ตรงกับ spec ใหม่:
1. เพิ่ม field ใน Go struct
2. อัพเดท validation
3. อัพเดท migration (ถ้าจำเป็น)
4. อัพเดท test cases
```

> **Tip:** ตั้ง CLAUDE.md ให้ Claude Code รู้ว่า OpenAPI spec อยู่ที่ไหน:
> ```
> # API Specification
> OpenAPI spec: docs/api/openapi.yaml
> ทุกครั้งที่สร้างหรือแก้ไข API ต้องตรวจสอบกับ OpenAPI spec
> ```

---

## 5.7 Test-Driven Specification

### แนวคิด TDD กับ Spec

Test-Driven Specification (การทดสอบที่ขับเคลื่อนด้วยสเปค) คือการเขียน test cases
จาก spec ก่อนเริ่ม implement จริง เป็นการผสมผสานระหว่าง TDD (Test-Driven Development)
กับ Spec-Driven Development

### เขียน Test ก่อน Implementation

สั่ง Claude Code สร้าง test จาก spec:

```
จาก spec ใน docs/specs/user-registration.md
สร้าง test file ก่อน:
1. TestCreateUser_Success
2. TestCreateUser_DuplicateEmail
3. TestCreateUser_InvalidPassword
4. TestCreateUser_MissingFields
(ยังไม่ต้อง implement -- แค่ test cases)
```

### ตัวอย่าง Go Test Code

```go
package service_test

import (
    "context"
    "testing"

    "github.com/stretchr/testify/assert"
    "github.com/stretchr/testify/require"

    "myapp/internal/model"
    "myapp/internal/service"
)

func TestCreateUser_Success(t *testing.T) {
    // Arrange
    svc := setupTestService(t)
    req := &model.RegisterRequest{
        Email:     "test@example.com",
        Password:  "SecurePass123!",
        FirstName: "สมชาย",
        LastName:  "ใจดี",
    }

    // Act
    user, err := svc.CreateUser(context.Background(), req)

    // Assert
    require.NoError(t, err)
    assert.NotEmpty(t, user.ID)
    assert.Equal(t, "test@example.com", user.Email)
    assert.Equal(t, "สมชาย", user.FirstName)
    assert.Equal(t, "ใจดี", user.LastName)
    assert.NotZero(t, user.CreatedAt)
}

func TestCreateUser_DuplicateEmail(t *testing.T) {
    // Arrange
    svc := setupTestService(t)
    req := &model.RegisterRequest{
        Email:     "duplicate@example.com",
        Password:  "SecurePass123!",
        FirstName: "ทดสอบ",
        LastName:  "ซ้ำ",
    }

    // สร้าง user แรก
    _, err := svc.CreateUser(context.Background(), req)
    require.NoError(t, err)

    // Act -- สร้าง user ซ้ำด้วยอีเมลเดียวกัน
    _, err = svc.CreateUser(context.Background(), req)

    // Assert
    assert.Error(t, err)
    assert.ErrorIs(t, err, service.ErrDuplicateEmail)
}

func TestCreateUser_InvalidPassword(t *testing.T) {
    svc := setupTestService(t)

    testCases := []struct {
        name     string
        password string
    }{
        {"too short", "Ab1!"},
        {"no uppercase", "securepass123!"},
        {"no number", "SecurePass!!!"},
        {"no special char", "SecurePass123"},
        {"only spaces", "        "},
    }

    for _, tc := range testCases {
        t.Run(tc.name, func(t *testing.T) {
            req := &model.RegisterRequest{
                Email:     "test@example.com",
                Password:  tc.password,
                FirstName: "Test",
                LastName:  "User",
            }

            _, err := svc.CreateUser(context.Background(), req)
            assert.Error(t, err)
            assert.ErrorIs(t, err, service.ErrInvalidPassword)
        })
    }
}

func TestCreateUser_MissingFields(t *testing.T) {
    svc := setupTestService(t)

    testCases := []struct {
        name string
        req  *model.RegisterRequest
    }{
        {"missing email", &model.RegisterRequest{
            Password: "SecurePass123!", FirstName: "Test", LastName: "User",
        }},
        {"missing password", &model.RegisterRequest{
            Email: "test@example.com", FirstName: "Test", LastName: "User",
        }},
        {"missing first_name", &model.RegisterRequest{
            Email: "test@example.com", Password: "SecurePass123!", LastName: "User",
        }},
        {"missing last_name", &model.RegisterRequest{
            Email: "test@example.com", Password: "SecurePass123!", FirstName: "Test",
        }},
    }

    for _, tc := range testCases {
        t.Run(tc.name, func(t *testing.T) {
            _, err := svc.CreateUser(context.Background(), tc.req)
            assert.Error(t, err)
        })
    }
}

func TestCreateUser_EmailTrimAndLowercase(t *testing.T) {
    // Edge case จาก spec: trim whitespace และแปลงเป็น lowercase
    svc := setupTestService(t)
    req := &model.RegisterRequest{
        Email:     "  Test@Example.COM  ",
        Password:  "SecurePass123!",
        FirstName: "Test",
        LastName:  "User",
    }

    user, err := svc.CreateUser(context.Background(), req)

    require.NoError(t, err)
    assert.Equal(t, "test@example.com", user.Email)
}
```

### ขั้นตอน TDD กับ Claude Code

1. **สร้าง test จาก spec** (test ต้อง fail ทั้งหมดเพราะยังไม่มี implementation):

```
สร้าง test file จาก spec -- ทุก test ต้อง compile ได้
แต่ fail เมื่อรัน เพราะยังไม่มี implementation
```

2. **ให้ Claude Code implement จนทุก test ผ่าน:**

```
implement service layer ให้ทุก test ใน
internal/service/user_service_test.go ผ่าน
อ้างอิง business rules จาก docs/specs/user-registration.md
```

3. **ตรวจสอบว่าทุก test ผ่าน:**

```bash
go test ./internal/service/... -v -count=1
```

> **Tip:** การเขียน test ก่อนช่วยให้ Claude Code มี "เป้าหมาย" ที่ชัดเจน
> AI จะ implement ได้ตรงจุดกว่าเมื่อรู้ว่าต้องทำให้ test อะไรผ่านบ้าง

---

## Cross-References

- [บทที่ 1: บทนำและพื้นฐาน](./01_intro.md)
- [บทที่ 2: Agentic Mastery](./02_agentic_mastery.md)
- [บทที่ 3: การใช้งานขั้นสูง](./03_advanced_usage.md)
- [บทที่ 4: Full-Stack Collaboration](./04_fullstack_collaboration.md)
- [บทที่ 6: Architecture & Design](./06_architecture_design.md)
- [บทที่ 7: QA, Security, and Review](./07_qa_security_review.md)
- [บทที่ 8: บทบาทและความรับผิดชอบ](./08_roles_responsibilities.md)
- [บทที่ 9: Cheat Sheet](./09_cheat_sheet.md)

---

> **สรุปท้ายบท:** Specification-Driven Development เป็นแนวทางที่ทำให้การทำงานกับ
> Claude Code มีประสิทธิภาพสูงสุด เมื่อ spec ชัดเจน AI จะสร้างโค้ดที่ตรงตาม
> requirement ได้ตั้งแต่รอบแรก ลดเวลาในการ iterate และทำให้ทีมทำงานร่วมกันได้ดีขึ้น
