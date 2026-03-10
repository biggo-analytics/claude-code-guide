# Phase 6: Maintenance — ดูแลระบบหลัง Production

## ภาพรวม

ShopFast เปิดใช้งานจริงแล้ว ลูกค้าเริ่มสั่งซื้อสินค้า ทีมเข้าสู่โหมด maintenance ซึ่งไม่ได้หมายความว่า "ไม่มีอะไรทำ" — ตรงกันข้าม งานจะมาจากหลายทิศทางพร้อมกัน

Phase นี้ **ไม่ได้เป็นเส้นตรง** แต่เป็น 5 สถานการณ์อิสระที่เกิดขึ้นได้ในลำดับใดก็ได้:

| Scenario | สถานการณ์ | ใครเกี่ยวข้อง |
|----------|-----------|---------------|
| 1 | Production Bug — ส่วนลดคำนวณผิด | 👤 คุณคิว (QA), 👤 คุณแบ็ค (Backend) |
| 2 | New Feature — ระบบ coupon | 👤 คุณแพร (PM), 👤 คุณบี (BA), 👤 คุณเอส (SA), 👤 คุณแบ็ค, 👤 คุณฟร้อนท์ (Frontend) |
| 3 | Refactoring — แยก service ใหญ่ | 👤 คุณเอส, 👤 คุณแบ็ค |
| 4 | Onboarding — สมาชิกใหม่ | 👤 คุณนิว (New Dev), 👤 คุณเอส |
| 5 | Performance Issue — API ช้า | 👤 คุณแบ็ค |

---

## Scenario 1: Production Bug — ออเดอร์คำนวณส่วนลดผิด

### รายงานปัญหา

👤 คุณคิว ได้รับรายงานจากฝ่าย Customer Support ว่าลูกค้าสั่งสินค้า 6 ชิ้นขึ้นไป ส่วนลดที่ได้ไม่ตรงตามที่โปรโมชันระบุ คุณคิวสร้าง bug ticket พร้อม reproduction steps:

> "สั่งสินค้า 6 ชิ้น ราคาชิ้นละ 100 บาท ควรได้ส่วนลด 10% (รวม 540 บาท) แต่ระบบคิด 600 บาท — ส่วนลดไม่ถูก apply"

### วิเคราะห์ปัญหาด้วย Claude Code

👤 คุณแบ็ค เริ่มสืบหาสาเหตุ:

```
ดูโค้ดที่คำนวณส่วนลดของ order ใน OrderService
มีลูกค้ารายงานว่าสั่งสินค้า 6 ชิ้นขึ้นไปแล้วส่วนลดไม่ถูก apply
ช่วยหา root cause ให้หน่อย
```

Claude Code วิเคราะห์ไฟล์ `internal/service/order_service.go` และพบปัญหา — off-by-one error ในเงื่อนไข:

```go
// ❌ โค้ดที่มีปัญหา — ใช้ > แทน >=
if len(items) > 5 {
    discount = 0.10
}
```

เงื่อนไขกำหนดว่า "มากกว่า 5 ชิ้น" แต่ business rule คือ "5 ชิ้นขึ้นไป" ดังนั้นออเดอร์ที่มีสินค้าพอดี 5 หรือ 6 ชิ้นจึงไม่ได้ส่วนลด (6 > 5 เป็น true แต่ 5 > 5 เป็น false — ลูกค้าที่สั่ง 5 ชิ้นก็โดนด้วย)

### เขียน regression test ก่อน fix

```
เขียน test case สำหรับ bug นี้ก่อนที่จะ fix
ต้องครอบคลุม: สั่ง 4 ชิ้น (ไม่ได้ส่วนลด), 5 ชิ้น (ได้ส่วนลด), 6 ชิ้น (ได้ส่วนลด), 10 ชิ้น (ได้ส่วนลด)
```

Claude Code สร้าง test ใน `internal/service/order_service_test.go` ที่ fail กับโค้ดปัจจุบัน — ยืนยันว่า bug มีจริง

### Fix และ verify

```
fix bug นี้โดยเปลี่ยนเงื่อนไขให้ตรงตาม business rule
"สั่งสินค้า 5 ชิ้นขึ้นไปได้ส่วนลด 10%"
แล้วรัน test ให้ผ่าน
```

```go
// ✅ โค้ดที่ถูกต้อง
if len(items) >= 5 {
    discount = 0.10
}
```

✅ **Checkpoint:** Test ทั้งหมดผ่าน รวมถึง regression test ที่เพิ่งเขียน

### บันทึก DEVLOG

👤 คุณแบ็ค เขียน DEVLOG entry:

```
เพิ่ม DEVLOG entry สำหรับ bug fix วันนี้
- ปัญหา: off-by-one error ในการคำนวณส่วนลด
- สาเหตุ: ใช้ > แทน >= ทำให้ออเดอร์ที่มีสินค้าพอดี 5 ชิ้นไม่ได้ส่วนลด
- วิธีแก้: เปลี่ยนเงื่อนไขเป็น >= 5
- เพิ่ม regression test 4 cases
```

📌 ดูแนวทางการเขียน DEVLOG เพิ่มเติมที่ [บทที่ 4](../04_fullstack_collaboration.md) หัวข้อ DEVLOG

💡 **Tip:** เมื่อ fix production bug ให้เขียน test ก่อนเสมอ — ถ้า test ไม่ fail แสดงว่าคุณยังไม่ได้ reproduce bug จริง

---

## Scenario 2: New Feature — เพิ่มระบบ coupon

### เริ่มจาก requirement

👤 คุณแพร (PM) ได้รับ request จาก business team ว่าต้องการระบบ coupon เพื่อใช้ในแคมเปญการตลาด:

> "ลูกค้ากรอกโค้ด coupon ตอน checkout แล้วได้ส่วนลดเพิ่ม เช่น SUMMER20 ลด 20%"

### เขียน spec ด้วย Claude Code

👤 คุณบี (BA) เขียน spec:

```
ช่วยเขียน spec สำหรับระบบ coupon ของ e-commerce
โดยมี requirement หลัก:
- coupon มีรหัส เช่น SUMMER20
- กำหนดประเภทส่วนลดได้ (เปอร์เซ็นต์ หรือ จำนวนเงิน)
- กำหนดวันหมดอายุได้
- จำกัดจำนวนครั้งที่ใช้ได้
- ลูกค้า 1 คนใช้ coupon เดียวกันได้ 1 ครั้ง

เขียนเป็น markdown ที่มี data model, API endpoints, business rules
```

Claude Code สร้าง spec ที่ครอบคลุม data model (Coupon, CouponUsage), API 4 endpoints (CRUD + apply), และ validation rules

### ตรวจสอบ architecture impact

👤 คุณเอส (SA) review spec แล้วใช้ Claude Code ตรวจสอบ:

```
ดูสถาปัตยกรรมปัจจุบันของ ShopFast แล้ววิเคราะห์ว่า
การเพิ่มระบบ coupon จะกระทบส่วนไหนบ้าง
ต้องแก้ไขอะไรใน OrderService, database schema, และ API layer
```

Claude Code ชี้ให้เห็นว่าต้องเพิ่ม CouponService ใหม่ แก้ไข OrderService ให้ apply coupon ตอนคำนวณราคา และเพิ่ม migration สำหรับ table ใหม่

### Implement backend

👤 คุณแบ็ค implement ทีละส่วน:

```
สร้าง Coupon model ตาม spec นี้ [วาง spec]
สร้างใน internal/model/coupon.go
พร้อม database migration
```

```
สร้าง CouponService ที่มี method:
- CreateCoupon, GetCoupon, ListCoupons
- ApplyCoupon (validate แล้ว return ส่วนลด)
- ValidateCoupon (เช็คว่ายังใช้ได้อยู่ไหม)
```

```
สร้าง coupon API endpoints:
- POST /api/coupons (admin สร้าง coupon)
- GET /api/coupons (admin list coupons)
- POST /api/coupons/apply (ลูกค้า apply coupon ตอน checkout)
เชื่อมกับ OrderService ที่มีอยู่
```

### Implement frontend

👤 คุณฟร้อนท์ เพิ่มหน้า coupon management:

```
สร้างหน้า admin สำหรับจัดการ coupon
- ตาราง list coupons ที่มีอยู่
- form สร้าง coupon ใหม่
- แสดงสถานะ (active/expired/used up)
ใช้ component pattern เดียวกับหน้า product management ที่มีอยู่
```

```
เพิ่ม coupon input field ในหน้า checkout
- ช่อง input กรอกรหัส coupon
- ปุ่ม "ใช้ coupon"
- แสดงส่วนลดที่ได้ หรือ error message ถ้าโค้ดไม่ถูกต้อง
```

✅ **Checkpoint:** Coupon system ทำงานครบ — สร้าง, ใช้, validate, แสดงผลบนหน้า checkout

⚠️ **Warning:** Feature ใหม่ใน production ควรใช้ feature flag เปิด/ปิดได้ เผื่อมีปัญหาจะได้ rollback โดยไม่ต้อง deploy ใหม่

---

## Scenario 3: Refactoring — แยก monolith service ที่ใหญ่เกิน

### ปัญหา

`internal/service/order_service.go` โตขึ้นเรื่อย ๆ จนมีกว่า 800 บรรทัด รวมทุกอย่างตั้งแต่สร้าง order, คำนวณราคา, จัดการ payment, ส่ง notification — ทุกครั้งที่แก้ไขมีโอกาส break ส่วนอื่น

### วิเคราะห์ responsibilities

👤 คุณเอส ใช้ Claude Code วิเคราะห์:

```
วิเคราะห์ไฟล์ internal/service/order_service.go
ไฟล์นี้มีกว่า 800 บรรทัด ช่วยระบุว่ามี responsibilities อะไรบ้าง
แต่ละกลุ่มมีกี่ method และควรแยกเป็น service ใหม่อย่างไร
ให้ยึดหลัก Single Responsibility Principle
```

Claude Code วิเคราะห์และสรุปว่า OrderService มี 3 กลุ่ม responsibility หลัก:

1. **Order Management** (~300 lines) — สร้าง/แก้ไข/ยกเลิก order
2. **Payment Processing** (~250 lines) — คำนวณราคา, เรียก payment gateway, จัดการ refund
3. **Notification** (~200 lines) — ส่ง email/SMS ตาม order status
4. **Shared utilities** (~50 lines) — validation, formatting

### วางแผน refactoring

```
วางแผนการ refactor order_service.go โดย:
1. แยกเป็น 3 services: OrderService, PaymentService, NotificationService
2. ให้ OrderService เรียก PaymentService และ NotificationService ผ่าน interface
3. ต้องไม่เปลี่ยน API contract ที่ handler เรียกใช้
4. เขียนเป็น step-by-step plan ก่อน ยังไม่ต้อง implement
```

Claude Code ร่างแผน 5 ขั้นตอน พร้อม interface ที่ต้องสร้าง และลำดับการแยกไฟล์

### ทำ refactoring ทีละ step

👤 คุณแบ็ค ทำตามแผน:

```
Step 1: สร้าง PaymentService interface และ implementation
ย้าย method ที่เกี่ยวกับ payment จาก order_service.go ไปยัง payment_service.go
อย่าลืมเขียน test สำหรับ PaymentService ด้วย
```

```
Step 2: สร้าง NotificationService interface และ implementation
ย้าย method ที่เกี่ยวกับ notification ไปยัง notification_service.go
```

```
Step 3: แก้ไข OrderService ให้ inject PaymentService และ NotificationService
แทนที่จะเรียก method ภายในตัวเอง ให้เรียกผ่าน interface แทน
รัน test ทั้งหมดให้ผ่าน
```

หลัง refactor เสร็จ:

| ไฟล์ | บรรทัด | Responsibility |
|------|--------|----------------|
| `order_service.go` | ~300 | Order lifecycle management |
| `payment_service.go` | ~250 | Payment processing & refund |
| `notification_service.go` | ~200 | Email/SMS notifications |

✅ **Checkpoint:** Test ทั้งหมดผ่าน, API contract ไม่เปลี่ยน, แต่โค้ดอ่านง่ายขึ้นมาก

📌 ดูหลักการ refactoring เพิ่มเติมที่ [บทที่ 6](../06_architecture_design.md) หัวข้อ Refactoring

💡 **Tip:** Refactoring ที่ดีคือ "เปลี่ยนโครงสร้างโดยไม่เปลี่ยนพฤติกรรม" — ถ้า test ทั้งหมดผ่านโดยไม่ต้องแก้ test แปลว่าคุณ refactor ได้ถูกต้อง

---

## Scenario 4: Onboarding — สมาชิกใหม่เข้าทีม

### สมาชิกใหม่

👤 คุณนิว เป็น Backend Developer ที่เพิ่งเข้าร่วมทีม ShopFast มีประสบการณ์ Go แต่ไม่รู้จักโปรเจกต์นี้เลย

### ใช้ Claude Code เรียนรู้ codebase

คุณนิวเปิด Claude Code แล้วเริ่มถามคำถาม:

```
อธิบายสถาปัตยกรรมของโปรเจกต์นี้
มีกี่ layer, แต่ละ layer ทำหน้าที่อะไร
ใช้ framework และ library อะไรบ้าง
```

Claude Code อ่าน codebase แล้วอธิบายว่าโปรเจกต์ใช้ Clean Architecture 3 layers (Handler → Service → Repository), ใช้ Gin framework, GORM สำหรับ database, และ Redis สำหรับ caching

```
flow ของ order creation เป็นยังไง
ตั้งแต่ request เข้ามาจนถึง response กลับไป
อยากเข้าใจว่าผ่าน file ไหนบ้าง
```

Claude Code trace ตั้งแต่ `POST /api/orders` → `order_handler.go` → `order_service.go` → `order_repository.go` → database พร้อมอธิบายว่าแต่ละ step ทำอะไร

```
อธิบาย business rules ของระบบส่วนลดและ coupon
ดูจาก code แล้วสรุปให้หน่อยว่ามี rule อะไรบ้าง
```

### ตั้งค่า onboarding checklist

👤 คุณเอส เพิ่ม onboarding section ใน CLAUDE.md:

```
เพิ่ม section "Onboarding Checklist" ใน CLAUDE.md สำหรับ developer ใหม่
รวมถึง:
- วิธี setup โปรเจกต์ (docker compose up)
- คำสั่ง test ที่ใช้บ่อย
- architecture overview สั้นๆ
- conventions ที่ต้องรู้ (naming, error handling, logging)
- ไฟล์สำคัญที่ควรอ่านก่อน
```

Claude Code เพิ่ม section ที่ช่วยให้คนใหม่เริ่มต้นได้เร็ว:

```markdown
## Onboarding Checklist

### Setup
1. Clone repo และรัน `docker compose up -d`
2. รัน `make migrate` สร้าง database schema
3. รัน `make seed` ใส่ข้อมูลตัวอย่าง
4. รัน `make test` ตรวจสอบว่าทุกอย่างทำงาน

### ไฟล์สำคัญที่ควรอ่านก่อน
- `CLAUDE.md` — conventions และ project rules
- `internal/service/order_service.go` — core business logic
- `internal/handler/` — API endpoints ทั้งหมด
- `docs/api.yaml` — API specification
```

### task แรกของคุณนิว

คุณเอสมอบหมาย task เล็ก ๆ: เพิ่ม endpoint `GET /api/orders/:id/status` สำหรับเช็คสถานะออเดอร์

```
ดูตัวอย่าง endpoint ที่มีอยู่ใน order_handler.go
แล้วช่วยเขียน endpoint GET /api/orders/:id/status
ที่ return เฉพาะ status ของ order
ทำตาม pattern เดียวกับ endpoint อื่น ๆ
```

Claude Code ช่วยให้คุณนิวเขียนโค้ดที่ตรงตาม convention ของโปรเจกต์ตั้งแต่ครั้งแรก — ไม่ต้องรอถูก review แล้วแก้

✅ **Checkpoint:** คุณนิว setup โปรเจกต์สำเร็จ เข้าใจ architecture และส่ง PR แรกได้ภายในวันแรก

📌 ดูแนวทาง onboarding เพิ่มเติมที่ [บทที่ 8](../08_roles_responsibilities.md) หัวข้อ Onboarding

💡 **Tip:** Claude Code เป็นเหมือน "เพื่อนร่วมทีมที่อ่านโค้ดทุกบรรทัดแล้ว" — ใช้มันเป็น onboarding buddy ที่ตอบคำถามได้ทุกเรื่องเกี่ยวกับ codebase

---

## Scenario 5: Performance Issue — API ช้าเมื่อ product มีเยอะ

### ปัญหา

หลังเปิดใช้งานไปสักพัก product ในระบบเพิ่มขึ้นเรื่อย ๆ จนมีกว่า 50,000 รายการ `GET /api/products` ที่เคยตอบภายใน 200ms ตอนนี้ใช้เวลากว่า 3 วินาที

### วิเคราะห์ปัญหา

👤 คุณแบ็ค ใช้ Claude Code วิเคราะห์:

```
API GET /api/products ช้ามาก ใช้เวลา 3 วินาที
เมื่อมี product 50,000 รายการ
ช่วยวิเคราะห์ code ตั้งแต่ handler ถึง repository
หาจุดที่น่าจะเป็น bottleneck
```

Claude Code ตรวจสอบ code flow แล้วพบปัญหาหลัก 3 จุด:

**ปัญหาที่ 1: N+1 Query**

```go
// ❌ N+1 Query — ดึง product แล้ว loop ดึง category ทีละตัว
products, _ := repo.FindAll()
for _, p := range products {
    p.Category, _ = categoryRepo.FindByID(p.CategoryID)
}
```

**ปัญหาที่ 2: ไม่มี pagination**

```go
// ❌ ดึงทั้ง 50,000 records ทุกครั้ง
func (r *ProductRepository) FindAll() ([]Product, error) {
    var products []Product
    r.db.Find(&products)
    return products, nil
}
```

**ปัญหาที่ 3: ไม่มี database index ที่เหมาะสม**

### แก้ไขทีละจุด

```
แก้ N+1 query ใน product repository
ใช้ GORM Preload ดึง category มาพร้อมกับ product ในครั้งเดียว
```

```go
// ✅ Preload — 1 query สำหรับ products, 1 query สำหรับ categories
func (r *ProductRepository) FindAll(opts ...QueryOption) ([]Product, error) {
    var products []Product
    query := r.db.Preload("Category")
    // apply options (pagination, filters)
    for _, opt := range opts {
        query = opt(query)
    }
    query.Find(&products)
    return products, nil
}
```

```
เพิ่ม pagination สำหรับ product list API
รับ query parameter page และ limit
default limit = 20, max limit = 100
```

```
เพิ่ม database index สำหรับ field ที่ใช้ค้นหาบ่อย:
- products.category_id (foreign key)
- products.name (search)
- products.created_at (sort)
เขียนเป็น migration
```

### เพิ่ม Redis cache

```
เพิ่ม Redis cache สำหรับ product list
- cache key pattern: products:page:{page}:limit:{limit}
- TTL 5 นาที
- invalidate cache เมื่อ product มีการเปลี่ยนแปลง
ใช้ pattern เดียวกับที่ใช้ใน project อยู่แล้ว
```

### ผลลัพธ์ before/after

| Metric | Before | After |
|--------|--------|-------|
| Response time (p50) | 3,200ms | 45ms |
| Response time (p99) | 5,800ms | 120ms |
| Database queries per request | 50,001 | 2 |
| Records returned | 50,000 | 20 (paginated) |
| Cache hit rate | — | ~85% |

✅ **Checkpoint:** API ตอบสนองเร็วขึ้น 70 เท่า ลูกค้าไม่ต้องรอหน้าสินค้าโหลดอีกต่อไป

⚠️ **Warning:** อย่าเพิ่ม cache ก่อนแก้ root cause — cache ไม่ใช่ยาวิเศษ ถ้า query ยังเป็น N+1 อยู่ cache miss ครั้งเดียวก็ช้าเท่าเดิม

---

## เคล็ดลับ Maintenance ด้วย Claude Code

### 1. ใช้ Claude Code เป็น first responder สำหรับ bug

เมื่อได้รับ bug report ให้ copy รายละเอียดทั้งหมดแปะให้ Claude Code วิเคราะห์ — มันจะช่วย narrow down จุดที่น่าจะมีปัญหาได้เร็วกว่าการ grep ด้วยมือ

### 2. เขียน test ก่อน fix เสมอ

```
เขียน test ที่ reproduce bug นี้ก่อน:
[วาง bug description]
test ต้อง fail กับ code ปัจจุบัน
```

### 3. ใช้ Claude Code วางแผน refactoring

อย่า refactor แบบ "คิดไปทำไป" — ให้ Claude Code วิเคราะห์ dependencies ก่อน แล้ววางแผนเป็น step-by-step

### 4. Onboarding ด้วย CLAUDE.md

ทุกครั้งที่มีคนใหม่เข้าทีม ให้ update CLAUDE.md ด้วย — ถ้าคนใหม่ถามคำถามที่ CLAUDE.md ไม่มี นั่นคือสิ่งที่ควรเพิ่ม

### 5. Profile ก่อน optimize

```
วิเคราะห์ performance ของ [ชื่อ API]
หา bottleneck จาก code ก่อน
อย่าเพิ่งเสนอวิธีแก้จนกว่าจะระบุ root cause ได้
```

💡 **Tip สุดท้าย:** Maintenance phase ไม่มีวันจบ — แต่ถ้าทีมมี process ที่ดี (DEVLOG, test, CLAUDE.md) และใช้เครื่องมือช่วยอย่าง Claude Code แต่ละปัญหาจะแก้ได้เร็วขึ้นเรื่อย ๆ เพราะความรู้ถูกสะสมไว้ใน codebase ไม่ใช่แค่ในหัวคน

---

← [Phase 5: Integration & CI/CD](05_phase5_integration_cicd.md) | [Prompt Catalog →](07_prompt_catalog.md)
