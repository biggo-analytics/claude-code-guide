# Phase 3: Core Features — ฟีเจอร์หลัก

> **สัปดาห์ 3–4** — Authentication, business logic (order workflow),
> product search, test plan, unit tests, และ debugging

---

## ภาพรวม Phase นี้

Phase 3 เปลี่ยนโปรเจกต์จาก "CRUD พื้นฐาน" ไปเป็น "ระบบที่มี business value จริง":

1. **Authentication** — JWT-based auth ครบวงจร (register, login, refresh, middleware)
2. **Order Workflow** — Business logic: cart → checkout → payment → order status
3. **Product Search** — Full-text search ด้วย PostgreSQL tsvector
4. **Test Plan & Unit Tests** — QA เขียน test plan, Backend Dev เขียน table-driven tests
5. **Bug Discovery & Fix** — QA พบ race condition, Backend Dev debug และแก้ไข
6. **ADR** — SA บันทึก decision เรื่อง database-level locking

```
สัปดาห์ 3                              สัปดาห์ 4
├──────────────────────────────────────┼──────────────────────────────────────┤
│ Auth ──────►                         │                                      │
│     Order Workflow ──────────►       │                                      │
│               Search ─────►          │                                      │
│                                      │ Test Plan ──► Unit Tests ──────►     │
│                                      │        Bug Found ──► Debug ──► Fix   │
│                                      │                            ADR ──►   │
```

---

## ใครทำอะไรใน Phase นี้

| Role | งาน | วัน |
|------|-----|-----|
| 👤 **คุณแบ็ค** (Backend Dev) | Implement auth, order workflow, search, unit tests, debug | วันที่ 8–13 |
| 👤 **คุณคิว** (QA) | เขียน test plan, run tests, report bug | วันที่ 11–12 |
| 👤 **คุณเอส** (SA) | Review architecture decisions, เขียน ADR | วันที่ 13 |
| 👤 **คุณบี** (BA) | Clarify business rules สำหรับ order workflow | วันที่ 9 (support) |
| 👤 **คุณแพร** (PM) | Track progress, prioritize bug fix | วันที่ 12 (support) |

---

## Step 1: Backend Dev — Implement Authentication

### วันที่ 8: คุณแบ็ค เริ่มงาน authentication

👤 **คุณแบ็ค** → Claude Code:

```
ดูโครงสร้าง internal/user/ ที่มีอยู่แล้ว และสร้างระบบ authentication ให้ ShopFast:

1. สร้าง internal/auth/model.go — RegisterRequest, LoginRequest, TokenResponse, Claims struct
2. สร้าง internal/auth/service.go — AuthService ที่มี:
   - Register(req) → hash password ด้วย bcrypt, สร้าง user, return token pair
   - Login(req) → verify password, return JWT access token (15 min) + refresh token (7 days)
   - RefreshToken(refreshToken) → validate, issue new pair
   - token เก็บ user_id, role ใน claims
3. JWT secret อ่านจาก config (อย่า hardcode)
4. ใช้ pattern เดียวกับ service layer ที่มีอยู่ — inject repository ผ่าน constructor

ดู CLAUDE.md สำหรับ coding conventions ของโปรเจกต์
```

Claude Code สร้าง model structs ที่ consistent กับ codebase:

```go
// internal/auth/model.go (ย่อ)
type RegisterRequest struct {
    Email    string `json:"email" validate:"required,email"`
    Password string `json:"password" validate:"required,min=8"`
    Name     string `json:"name" validate:"required"`
}

type Claims struct {
    UserID uint   `json:"user_id"`
    Role   string `json:"role"`
    jwt.RegisteredClaims
}
```

💡 **เคล็ดลับ:** prompt บอก "ใช้ pattern เดียวกับที่มีอยู่" — ทำให้ Claude Code สร้าง code ที่ consistent กับ codebase โดยไม่ต้องอธิบาย pattern ซ้ำ

### สร้าง Middleware & Redis Refresh Token

👤 **คุณแบ็ค** → Claude Code:

```
สร้าง auth middleware สำหรับ Fiber:

1. internal/auth/middleware.go — JWTMiddleware: ดึง token จาก Authorization header,
   validate JWT, เก็บ user_id/role ใน c.Locals, return 401 ถ้า invalid
2. RequireRole(roles ...string) middleware ที่ check role จาก claims
3. internal/auth/handler.go — POST /auth/register, /auth/login, /auth/refresh
4. เพิ่ม refresh token storage ด้วย Redis:
   - key pattern "refresh:{user_id}:{token_id}", TTL = 7 days
   - token rotation: refresh → ลบ token เก่า, สร้างใหม่
   - logout → ลบ refresh token ของ user
```

⚠️ **ข้อควรระวัง:** คุณแบ็คตรวจ output เจอว่า Claude Code ลืม handle Redis connection error — ต้อง prompt เพิ่มให้ return 503 แทน 500

📌 **Cross-ref:** ดู [บทที่ 4](../04_fullstack_collaboration.md) สำหรับ pattern การ iterate กับ Claude Code เมื่อ output ยังไม่ครบ

✅ **จบวันที่ 8:** ระบบ auth ครบ — register, login, refresh, logout, middleware, role-based access

---

## Step 2: Backend Dev — Implement Order Workflow

### วันที่ 9: คุณแบ็ค เริ่มงาน business logic ที่ซับซ้อนที่สุด

คุณแบ็คปรึกษาคุณบี (BA) เพื่อ clarify rules ก่อน prompt:

```
Business Rules (จากคุณบี):
1. Cart → Checkout: ตรวจ stock ทุก item, reject ทั้ง order ถ้า out of stock,
   คำนวณราคา ณ เวลา checkout
2. Checkout → Payment: reserve stock, status="pending_payment",
   auto-cancel ถ้าไม่จ่ายใน 30 นาที
3. Payment → Confirmed: status="confirmed", finalize stock, ส่ง event
4. Statuses: pending_payment → confirmed → processing → shipped → delivered
   Cancel ได้เฉพาะ pending_payment และ confirmed
```

👤 **คุณแบ็ค** → Claude Code:

```
สร้าง order workflow ตาม business rules ต่อไปนี้ (อ่านให้ครบก่อนเริ่ม):

## Business Rules
[paste business rules จากคุณบีทั้งหมดข้างบน]

## Implementation Requirements
1. internal/order/model.go — Order, OrderItem, OrderStatus enum, CartItem
2. internal/order/service.go — OrderService: Checkout, ConfirmPayment,
   CancelOrder, GetOrder, ListOrders
3. Stock operations ต้องอยู่ใน database transaction (db.Transaction)
4. Status transitions ต้อง validate — ห้ามข้าม status

แยกไฟล์ชัดเจน: model, repository, service, handler, routes
```

📌 **Cross-ref:** ดู [บทที่ 5](../05_spec_driven_dev.md) สำหรับ pattern "Spec → Prompt → Code"

Claude Code สร้าง status transition validation (state machine):

```go
// internal/order/model.go (ย่อ)
var validTransitions = map[OrderStatus][]OrderStatus{
    StatusPendingPayment: {StatusConfirmed, StatusCancelled},
    StatusConfirmed:      {StatusProcessing, StatusCancelled},
    StatusProcessing:     {StatusShipped},
    StatusShipped:        {StatusDelivered},
}

func (s OrderStatus) CanTransitionTo(next OrderStatus) bool {
    for _, valid := range validTransitions[s] {
        if valid == next { return true }
    }
    return false
}
```

และ checkout logic ที่ใช้ `SELECT FOR UPDATE` เพื่อ lock stock:

```go
// internal/order/service.go (ย่อ — Checkout)
err := s.db.Transaction(func(tx *gorm.DB) error {
    for _, item := range items {
        var p product.Product
        tx.Clauses(clause.Locking{Strength: "UPDATE"}).First(&p, item.ProductID)
        if p.Stock < item.Quantity {
            return fmt.Errorf("insufficient stock for %s", p.Name)
        }
        tx.Model(&p).Update("stock", gorm.Expr("stock - ?", item.Quantity))
    }
    // ... create order with StatusPendingPayment, ExpiresAt = +30 min
})
```

⚠️ **ข้อควรระวัง:** version แรกที่ Claude Code สร้าง **ไม่มี locking** — คุณแบ็คต้อง prompt เพิ่มให้ใช้ `clause.Locking{Strength: "UPDATE"}`

💡 **เคล็ดลับ:** เมื่อ business logic ซับซ้อน ให้แบ่ง prompt ทีละ function — prompt Checkout ก่อน แล้ว ConfirmPayment ทีหลัง จะได้ review ทีละส่วน การ paste business rules ตรง ๆ จาก BA ดีกว่าย่อเอง

✅ **จบวันที่ 9–10:** Order workflow ครบ — checkout, payment, cancellation, status transitions

---

## Step 3: Backend Dev — Product Search

### วันที่ 10: คุณแบ็ค เพิ่มระบบ search ด้วย PostgreSQL tsvector

👤 **คุณแบ็ค** → Claude Code:

```
เพิ่มระบบ product search โดยใช้ PostgreSQL full-text search:

1. Migration: เพิ่ม tsvector column, trigger auto-update เมื่อ name/description เปลี่ยน,
   GIN index, setweight A สำหรับ name, B สำหรับ description
2. internal/product/search.go — SearchProducts รับ:
   query (full-text), category_id, min/max_price, sort_by, page, limit
3. ถ้า query ว่าง → return all products ตาม filters
4. Pagination ใช้ OFFSET/LIMIT (< 100K products)
```

Claude Code สร้าง migration ที่มี `setweight` + trigger + GIN index:

```sql
-- migrations/005_add_product_search_vector.up.sql (ย่อ)
ALTER TABLE products ADD COLUMN search_vector tsvector;

CREATE INDEX idx_products_search_vector ON products USING GIN(search_vector);

CREATE TRIGGER trigger_products_search_vector
    BEFORE INSERT OR UPDATE OF name, description ON products
    FOR EACH ROW EXECUTE FUNCTION products_search_vector_update();
```

และ search function ที่ support filtering + sort by relevance:

```go
// internal/product/search.go (ย่อ)
if params.Query != "" {
    query = query.Where("search_vector @@ plainto_tsquery('english', ?)", params.Query)
}
// ... category, price range filters, sort, pagination
```

💡 **เคล็ดลับ:** ระบุ `setweight` ใน prompt ช่วยให้ search ให้น้ำหนัก product name > description — ถ้าไม่ระบุจะได้ weight เท่ากัน

✅ **จบวันที่ 10:** Product search พร้อม — full-text search + filters + pagination

---

## Step 4: QA — เขียน Test Plan

### วันที่ 11: คุณคิว ใช้ Claude Code สร้าง test plan จาก specs

👤 **คุณคิว** → Claude Code:

```
อ่านไฟล์: internal/order/service.go, internal/order/model.go,
internal/auth/service.go, docs/specs/order_workflow.md

สร้าง test plan ในรูปแบบ markdown table ครอบคลุม:
1. Auth — register, login, refresh, invalid cases
2. Order Checkout — happy path, out of stock, concurrent checkout
3. Payment — success, timeout, double payment
4. Cancellation — valid/invalid states
5. Edge cases — empty cart, negative quantity, expired token

Priority: P0 (must test), P1 (should test), P2 (nice to test)
```

Claude Code สร้าง test plan (~23 test cases) — ตัวอย่างบางส่วน:

| Test ID | Category | Test Case | Expected Result | Priority |
|---------|----------|-----------|-----------------|----------|
| AUTH-01 | Auth | Register with valid data | 201, token pair | P0 |
| AUTH-07 | Auth | Refresh token rotation | New pair, old invalidated | P0 |
| ORD-01 | Checkout | Happy path | 201, stock reserved | P0 |
| ORD-02 | Checkout | Insufficient stock | 400, stock unchanged | P0 |
| **ORD-03** | **Checkout** | **Concurrent checkout same item** | **One succeeds, one fails** | **P0** |
| PAY-02 | Payment | Payment timeout (>30 min) | Auto-cancelled, stock restored | P0 |
| CAN-03 | Cancel | Cancel shipped order | 400, cannot cancel | P0 |

📌 **Cross-ref:** ดู [บทที่ 7](../07_qa_security_review.md) สำหรับ pattern การสร้าง test plan จาก code + specs

💡 **เคล็ดลับ:** ให้ Claude Code อ่าน source code + specs พร้อมกัน → test plan ครอบคลุมทั้ง requirements และ implementation details เช่น ORD-03 มาจากการเห็น `SELECT FOR UPDATE` ใน code

---

## Step 5: Backend Dev — เขียน Unit Tests

### วันที่ 11: คุณแบ็ค เขียน table-driven tests

👤 **คุณแบ็ค** → Claude Code:

```
เขียน unit tests สำหรับ internal/order/service.go:

1. Table-driven test pattern
2. Mock repository ด้วย testify/mock
3. Test cases ตาม test plan ORD-01 ถึง ORD-05
4. ใช้ testify/assert

ไฟล์: internal/order/service_test.go
```

```go
// internal/order/service_test.go (ย่อ)
func TestOrderService_Checkout(t *testing.T) {
    tests := []struct {
        name        string
        items       []CartItem
        mockSetup   func(*MockProductRepo, *MockOrderRepo)
        wantErr     bool
        errContains string
    }{
        {
            name:  "happy path - 2 items",
            items: []CartItem{{ProductID: 1, Quantity: 2}, {ProductID: 2, Quantity: 1}},
            // ... mock setup: stock sufficient
            wantErr: false,
        },
        {
            name:  "insufficient stock",
            items: []CartItem{{ProductID: 1, Quantity: 100}},
            // ... mock: stock=5
            wantErr: true, errContains: "insufficient stock",
        },
        {
            name: "empty cart", items: []CartItem{},
            wantErr: true, errContains: "cart is empty",
        },
    }
    // ... run tests with assert
}
```

เพิ่ม tests สำหรับ status transitions ด้วย table-driven pattern เช่นกัน — test ทุก valid/invalid transition

📌 **Cross-ref:** ดู [บทที่ 4](../04_fullstack_collaboration.md) สำหรับ pattern table-driven tests + mocking strategy

✅ **จบวันที่ 11:** Unit tests สำหรับ order service, status transitions — coverage ~80%

---

## Step 6: QA — Run Tests & พบ Bug

### วันที่ 12: คุณคิว run concurrent checkout test แล้วพบปัญหา

คุณคิวเขียน integration test สำหรับ ORD-03 ด้วย database จริง:

```go
// Scenario: product stock=1, 2 goroutines checkout พร้อมกัน
// Expected: 1 succeed, 1 fail, final stock=0
```

ผลลัพธ์:

```
$ go test -run TestCheckout_ConcurrentAccess -count=10

--- FAIL: TestCheckout_ConcurrentAccess (0.23s)
    expected 1 success, got 2
    expected stock=0, got stock=-1

FAIL (7/10 passed, 3 failed)
```

**ปัญหา:** ทั้ง 2 checkouts สำเร็จ! stock ติดลบ! — **race condition**

คุณคิวรายงาน bug:

```
Bug Report:
- Test ID: ORD-03 | Severity: Critical (P0)
- Description: Concurrent checkout ทำให้ stock ติดลบ
- Reproduce: run TestCheckout_ConcurrentAccess 10 ครั้ง, fail ~30%
- Root Cause (suspected): race condition ใน stock reservation
```

📌 **Cross-ref:** ดู [บทที่ 7](../07_qa_security_review.md) สำหรับ pattern bug report ที่ดี

---

## Step 7: Backend Dev — Debug Race Condition

### วันที่ 12: คุณแบ็ค วิเคราะห์ด้วย Claude Code

👤 **คุณแบ็ค** → Claude Code:

```
มี bug: concurrent checkout ทำให้ stock ติดลบ

อ่าน internal/order/service.go ฟังก์ชัน Checkout และวิเคราะห์:
1. ดู flow ของ stock reservation ใน transaction
2. หา race condition ที่เป็นไปได้
3. อธิบายว่าทำไม SELECT FOR UPDATE อาจไม่ทำงาน

Bug: 2 goroutines checkout stock=1 พร้อมกัน, ทั้ง 2 สำเร็จ, stock=-1
```

**Claude Code วิเคราะห์:** transaction ใช้ `db.Transaction()` แต่ repository methods ใช้ `s.db` (service-level) แทน `tx` (transaction) — `SELECT FOR UPDATE` จึงทำงานบน connection คนละตัว ไม่ได้ block กัน:

```
Timeline:
1. Goroutine A: BEGIN TX, SELECT FOR UPDATE (connection 1)
2. Goroutine B: BEGIN TX, SELECT FOR UPDATE (connection 2 — ไม่ถูก block!)
3. ทั้งคู่เห็น stock=1, ทั้งคู่ UPDATE → stock = -1
```

### คุณแบ็ค แก้ไข

👤 **คุณแบ็ค** → Claude Code:

```
แก้ race condition: เปลี่ยน repository methods ให้รับ tx *gorm.DB เป็น parameter
ตรวจสอบว่าทุก db operation ใน Transaction callback ใช้ tx ไม่ใช่ s.db
```

```go
// ✅ แก้ไข: repository รับ tx, Checkout ใช้ tx ทุกที่
err := s.db.Transaction(func(tx *gorm.DB) error {
    for _, item := range items {
        p, _ := s.productRepo.FindByIDForUpdate(tx, item.ProductID) // ✅ tx
        // ...
        s.productRepo.UpdateStock(tx, p.ID, -item.Quantity) // ✅ tx
    }
    return s.orderRepo.Create(tx, order) // ✅ tx
})
```

```
$ go test -run TestCheckout_ConcurrentAccess -count=20
ok   shopfast/internal/order  4.521s (20/20 passed)

$ go test ./...
PASS (all packages)
```

📌 **Cross-ref:** ดู [บทที่ 4](../04_fullstack_collaboration.md) สำหรับ pattern การ debug ด้วย Claude Code

⚠️ **ข้อควรระวัง:** นี่เป็น bug ที่พบบ่อยใน GORM — ใช้ `db` แทน `tx` ภายใน `db.Transaction` callback ทุกครั้งที่เขียน transaction ต้อง review ว่า code ข้างในใช้ `tx`

✅ **จบวันที่ 12:** Bug fixed, คุณคิว verify — confirmed fixed

---

## Step 8: SA — เขียน ADR สำหรับ Locking Strategy

### วันที่ 13: คุณเอส บันทึก architectural decision

👤 **คุณเอส** → Claude Code:

```
เขียน ADR สำหรับ ShopFast:

Title: Use Database-Level Pessimistic Locking for Order Creation
Context: พบ race condition ใน concurrent checkout
Decision: ใช้ PostgreSQL SELECT FOR UPDATE ภายใน GORM transaction
Alternatives:
1. Optimistic locking (version column) — UX ไม่ดีถ้า conflict บ่อย
2. Application-level mutex — ไม่ scale กับ multiple instances
3. Redis distributed lock — เพิ่ม complexity
4. DB pessimistic locking (เลือก) — ตรงกับ PostgreSQL strengths

Consequences: repository ต้องรับ tx, ต้อง lock ตาม ID ASC เพื่อป้องกัน deadlock
```

Claude Code สร้าง ADR-004 ครบถ้วน: Status, Context, Decision (พร้อม implementation rules), Alternatives Considered (4 ตัวเลือก + pros/cons), Consequences (positive/negative/performance)

**Key decision points ใน ADR:**
- ทุก repository method ใน transaction ต้องรับ `tx *gorm.DB`
- Lock products ตาม `product_id ASC` เพื่อป้องกัน deadlock
- Acceptable สำหรับ < 1,000 concurrent checkouts/sec
- ถ้า scale เกินนี้ → พิจารณา optimistic locking + retry

📌 **Cross-ref:** ดู [บทที่ 6](../06_architecture_design.md) สำหรับ ADR template และ best practices

💡 **เคล็ดลับ:** ให้ Claude Code เขียน ADR พร้อม alternatives + เหตุผล — ช่วยให้ทีมเข้าใจ trade-offs และไม่ revisit decision ซ้ำ

---

## ✅ Checkpoint — จบ Phase 3

| รายการ | สถานะ |
|--------|-------|
| ระบบ Auth ครบ (register, login, refresh, logout, middleware, RBAC) | ✅ |
| Refresh token storage ใน Redis + token rotation | ✅ |
| Order workflow: checkout → payment → confirm/cancel | ✅ |
| Stock reservation + status transition validation (state machine) | ✅ |
| Product search ด้วย tsvector + filters + pagination | ✅ |
| Test plan ครอบคลุม P0, P1, P2 (23 test cases) | ✅ |
| Unit tests (table-driven) + integration test (concurrent checkout) | ✅ |
| Race condition bug fixed (tx แทน db) | ✅ |
| ADR-004 บันทึก locking strategy | ✅ |

### Deliverables

```
internal/
├── auth/
│   ├── model.go, service.go, middleware.go, handler.go, routes.go, service_test.go
├── order/
│   ├── model.go, repository.go, service.go, handler.go, routes.go
│   ├── service_test.go, service_integration_test.go
├── product/
│   ├── search.go, repository.go (updated: accepts tx)
migrations/005_add_product_search_vector.up.sql
docs/adr/ADR-004-pessimistic-locking.md
docs/test-plan/phase3-test-plan.md
```

---

## เคล็ดลับ & ข้อผิดพลาดที่พบบ่อย

### 💡 เคล็ดลับ

1. **Prompt ทีละ function** — แบ่ง prompt ทีละ function (Checkout, ConfirmPayment, CancelOrder) แล้ว review ทีละชิ้น ดีกว่า prompt ทั้ง workflow ในครั้งเดียว
2. **Paste specs ตรง ๆ** — อย่าย่อ business rules จาก BA ให้ paste ทั้งหมดใน prompt ได้เลย
3. **Debug ด้วย code + error + context** — ระบุไฟล์ที่เกี่ยวข้อง + symptom + expected behavior ในครั้งเดียว
4. **ระบุ test cases ชัดเจน** — ทั้ง positive และ negative ไม่ปล่อยให้ Claude Code เดาเอง
5. **ใช้ test plan เป็น input** — QA สร้าง test plan → Backend Dev อ้างอิง test IDs ใน prompt → traceable
6. **เขียน ADR ทันที** — อย่ารอจนลืม context

### ⚠️ ข้อผิดพลาดที่พบบ่อย

1. **ลืมส่ง `tx` เข้า repository** — bug #1 ของ GORM transactions, Claude Code ก็อาจสร้าง code ที่มี bug นี้
2. **JWT secret hardcode** — ตรวจสอบว่าอ่านจาก config/env เสมอ
3. **ไม่ validate status transition** — client อาจ skip status เช่น pending → delivered
4. **Pagination ไม่มี upper limit** — `limit=1000000` ทำให้ server OOM, ต้อง cap (max 100)
5. **`to_tsquery` ไม่ sanitize** — ใช้ `plainto_tsquery` ที่ safe กว่า
6. **Integration test ไม่ cleanup** — test state leak ข้าม test cases

---

*← [Phase 2: Database & Backend](02_phase2_database_backend.md) | [Phase 4: Frontend →](04_phase4_frontend.md)*
