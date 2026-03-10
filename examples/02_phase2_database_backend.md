# Phase 2: Database & Backend Foundation

> **ระยะเวลา:** สัปดาห์ 2–3 (~2 สัปดาห์)
> **เป้าหมาย:** ออกแบบ database schema, สร้าง GORM models, ตั้งโครงสร้าง Go project, และสร้าง base CRUD สำหรับ resource แรก

---

## ภาพรวม Phase นี้

Phase 2 คือช่วงที่เปลี่ยนจาก "กระดาษ" เป็น "โค้ด" ครั้งแรก — DBA ออกแบบ database schema จาก specs ที่ได้จาก Phase 1 แล้ว Backend Dev เอา models เหล่านั้นมาสร้างเป็น API พร้อม three-layer architecture

**เป้าหมายหลัก:**

1. ER diagram ครบ 8 entities พร้อม relationships
2. GORM models ที่ map กับ database schema
3. Migration strategy ที่ทีมเห็นพ้อง
4. Go project structure ตาม three-layer architecture
5. Product CRUD เป็น reference implementation
6. Input validation + error response format มาตรฐาน
7. Database indexes สำหรับ query ที่ใช้บ่อย

```
สัปดาห์ 2                              สัปดาห์ 3
├──── Day 1-2 ────┼──── Day 3-5 ────┼──── Day 1-3 ────┼──── Day 4-5 ────┤
│  ER Design &    │  Project Setup  │  Base CRUD &    │  Validation,   │
│  GORM Models    │  & Migrations   │  Repository     │  Index, Review │
│  (คุณดีบี)       │  (คุณแบ็ค+ดีบี)  │  (คุณแบ็ค)       │  (ทุกคน)        │
```

---

## ใครทำอะไรใน Phase นี้

| Role | ชื่อ | งานหลัก | วัน |
|------|------|---------|-----|
| 👤 **DBA/DevOps** | คุณดีบี | ER diagram, GORM models, migrations, indexes | Day 1–2, 8–9 |
| 👤 **Backend Dev** | คุณแบ็ค | Scaffold project, base CRUD, validation | Day 3–8 |
| 👤 **SA** | คุณเอส | Review architecture, naming conventions | Day 9–10 |
| 👤 **PM** | คุณแพร | ติดตาม progress, review checkpoint | Day 5, 10 |
| 👤 **BA** | คุณบี | ตรวจ field names ตรงกับ specs | Day 10 |

---

## Step 1: DBA ออกแบบ ER Diagram

**วันที่ 4 ของโปรเจกต์ (สัปดาห์ 2, Day 1):**

👤 **คุณดีบี** เปิด Claude Code พร้อมกับ specs จาก Phase 1 เพื่อออกแบบ database schema:

```
ออกแบบ ER diagram สำหรับระบบ ShopFast E-commerce ที่มี entities:
User, Product, Category, Order, OrderItem, Payment, Address, Review

ข้อกำหนด:
- User มี role enum: customer, admin, super_admin
- Product many-to-many กับ Category
- Order has many OrderItems, แต่ละ OrderItem อ้าง Product
- Payment ผูก Order แบบ one-to-one
- Address ผูก User แบบ one-to-many
- Review ผูก User+Product (unique per user per product)
- ทุกตารางมี created_at, updated_at, deleted_at (soft delete)

แสดงเป็น text-based ER diagram พร้อม field types
```

Claude Code สร้าง ER diagram:

```
┌──────────────────┐     ┌──────────────────┐     ┌──────────────────┐
│     users         │     │    addresses      │     │     orders        │
├──────────────────┤     ├──────────────────┤     ├──────────────────┤
│ id        BIGINT PK│──┐ │ id      BIGINT PK│     │ id        BIGINT PK│
│ email  VARCHAR(255)│  ├→│ user_id BIGINT FK│     │ order_number V(20)│
│ password   V(255)  │  │ │ label    V(50)   │     │ user_id  BIGINT FK│←┐
│ name     V(100)    │  │ │ street   V(255)  │     │ address_id BIGINT │ │
│ role      V(20)    │  │ │ province V(100)  │     │ status     V(20)  │ │
│ is_active    BOOL  │  │ │ postal_code V(10)│     │ total_amount DEC  │ │
│ timestamps...      │  │ │ timestamps...    │     │ timestamps...     │ │
└──┬────────────────┘  │ └──────────────────┘     └──┬───────────────┘ │
   │                    │                              │                  │
   │  ┌────────────────┘                              ▼                  │
   │  │  ┌──────────────────┐     ┌──────────────────┐                  │
   │  │  │    reviews        │     │   order_items     │                  │
   │  │  ├──────────────────┤     ├──────────────────┤                  │
   │  │  │ user_id  BIGINT FK│     │ order_id BIGINT FK│                  │
   │  │  │ product_id  " FK │     │ product_id  " FK │                  │
   │  │  │ rating  SMALLINT │     │ quantity  INTEGER │                  │
   │  │  │ comment     TEXT │     │ unit_price DECIMAL│                  │
   │  │  │ UNIQUE(user,prod)│     │ subtotal  DECIMAL │                  │
   │  │  └──────────────────┘     └──────────────────┘                  │
   │  │                                                                  │
   ▼  ▼                                                                  │
┌──────────────────┐     ┌──────────────────┐     ┌──────────────────┐ │
│    products       │     │product_categories│     │    payments       │ │
├──────────────────┤     ├──────────────────┤     ├──────────────────┤ │
│ id      BIGINT PK│◄───│ product_id  " FK │     │ order_id BIGINT FK│─┘
│ name    V(255)   │     │ category_id " FK │     │ method     V(20) │
│ slug    V(255) UQ│     └──────────────────┘     │ status     V(20) │
│ price    DECIMAL │            │                  │ amount    DECIMAL│
│ stock_qty INTEGER│            ▼                  │ transaction_id   │
│ sku     V(50) UQ │     ┌──────────────────┐     │ timestamps...    │
│ is_active   BOOL │     │   categories      │     └──────────────────┘
│ timestamps...    │     ├──────────────────┤
└──────────────────┘     │ id      BIGINT PK│
                          │ name    V(100)   │
                          │ slug    V(100)   │
                          │ parent_id " FK   │ ──→ (self-ref)
                          │ sort_order  INT  │
                          └──────────────────┘
```

💡 **เคล็ดลับ:** text-based ER diagram แปะใน PR description หรือ Slack ได้เลย — ทีมทั้งหมด review ได้โดยไม่ต้องเปิดเครื่องมือพิเศษ

⚠️ **ข้อควรระวัง:** แยก `id` (internal PK) กับ `order_number` (human-readable) — ไม่ควรใช้ auto-increment id เป็นเลข order เพราะเปิดเผยจำนวน order ได้

📌 **Cross-ref:** ดูหลักการออกแบบ database schema ใน [บทที่ 6: Architecture Design](../06_architecture_design.md)

---

## Step 2: DBA สร้าง GORM Models

👤 **คุณดีบี** แปลง ER diagram เป็น Go structs:

```
จาก ER diagram สร้าง GORM models ทั้ง 8 ตัว:
- ใช้ gorm.Model สำหรับ timestamps
- GORM tags: column, type, not null, unique, index
- Relationships: HasMany, BelongsTo, Many2Many ตาม ER
- json tags สำหรับ API response
- แต่ละ model อยู่คนละไฟล์ใน internal/models/
```

ตัวอย่าง Product model ที่ Claude Code สร้าง:

```go
// internal/models/product.go (snippet)
type Product struct {
    ID            uint            `gorm:"primaryKey" json:"id"`
    Name          string          `gorm:"type:varchar(255);not null" json:"name"`
    Slug          string          `gorm:"type:varchar(255);uniqueIndex;not null" json:"slug"`
    Description   string          `gorm:"type:text" json:"description"`
    Price         decimal.Decimal `gorm:"type:decimal(10,2);not null" json:"price"`
    StockQuantity int             `gorm:"not null;default:0" json:"stock_quantity"`
    SKU           string          `gorm:"type:varchar(50);uniqueIndex" json:"sku"`
    IsActive      bool            `gorm:"default:true" json:"is_active"`
    CreatedAt     time.Time       `json:"created_at"`
    UpdatedAt     time.Time       `json:"updated_at"`
    DeletedAt     gorm.DeletedAt  `gorm:"index" json:"deleted_at,omitempty"`

    Categories []Category  `gorm:"many2many:product_categories" json:"categories,omitempty"`
    Reviews    []Review    `gorm:"foreignKey:ProductID" json:"reviews,omitempty"`
    OrderItems []OrderItem `gorm:"foreignKey:ProductID" json:"-"`
}
```

คุณดีบีสังเกตว่า `Price` ควรใช้ `shopspring/decimal` แทน `float64`:

⚠️ **ข้อควรระวัง:** **อย่าใช้ `float64` สำหรับเงิน** — ปัญหา floating point (`0.1 + 0.2 != 0.3`) ทำให้ยอดเงินผิดพลาดสะสมได้ ใช้ `decimal` type เสมอ

💡 **เคล็ดลับ:** Order model ใช้ `*Payment` (pointer) เพราะ order อาจยังไม่มี payment — ไม่ใช้ pointer จะได้ zero value struct แทน `nil`

📌 **Cross-ref:** ดู GORM model patterns ใน [บทที่ 4: Full-Stack Collaboration](../04_fullstack_collaboration.md)

---

## Step 3: DBA เขียน Migration Strategy

**วันที่ 5 (สัปดาห์ 2, Day 2):**

👤 **คุณดีบี** ต้องตัดสินใจระหว่าง GORM AutoMigrate กับ manual SQL migrations:

```
เปรียบเทียบ migration strategy สำหรับ Go/GORM:
1. GORM AutoMigrate
2. golang-migrate/migrate (manual SQL)
ในด้าน: dev convenience, production safety, rollback, team conflicts, seed data
แนะนำ strategy ที่เหมาะกับทีม 7 คน + PostgreSQL
```

Claude Code แนะนำ **Hybrid Approach**:
- **Development:** ใช้ AutoMigrate เพื่อความเร็ว
- **Staging/Production:** ใช้ golang-migrate สำหรับ reviewed SQL migrations

คุณดีบีให้ Claude Code สร้าง migration setup:

```
สร้าง migration setup ตาม hybrid approach:
1. internal/database/connection.go — DB connection
2. internal/database/migrate.go — AutoMigrate สำหรับ dev
3. migrations/ — SQL migrations สำหรับ production
4. cmd/migrate/main.go — migration CLI
5. internal/database/seed.go — seed data (admin user, 5 categories, 10 products)
```

ตัวอย่าง seed data ที่ได้ (snippet):

```go
// internal/database/seed.go
func SeedDevelopmentData(db *gorm.DB) error {
    admin := models.User{
        Email: "admin@shopfast.com", Name: "ShopFast Admin",
        Role: "super_admin", IsActive: true,
    }
    db.FirstOrCreate(&admin, models.User{Email: admin.Email})
    // ... categories, products ...
    log.Println("✅ Development seed data created")
    return nil
}
```

💡 **เคล็ดลับ:** ใช้ `FirstOrCreate` ใน seed data แทน `Create` เพื่อให้ run ซ้ำได้ (idempotent)

📌 **Cross-ref:** ดู Database Tasks ใน [บทที่ 4](../04_fullstack_collaboration.md) หัวข้อ Database Tasks

---

## Step 4: Backend Dev ตั้งโครงสร้างโปรเจกต์

**วันที่ 6 (สัปดาห์ 2, Day 3):**

👤 **คุณแบ็ค** รับไม้ต่อ — มี GORM models พร้อมแล้ว ถึงเวลาสร้างโครงสร้างทั้งหมด:

```
สร้าง Go project structure ตาม three-layer architecture:
cmd/api/main.go, cmd/migrate/main.go
internal/handlers/    — HTTP handlers (controller layer)
internal/service/     — business logic layer
internal/repository/  — data access layer (GORM queries)
internal/models/      — GORM models (มีแล้ว)
internal/dto/         — request/response DTOs
internal/middleware/   — auth, logging, error handling
internal/config/      — app configuration
internal/database/    — connection, migrate, seed (มีแล้ว)
pkg/response/         — standard API response format
pkg/validator/        — custom validation helpers
pkg/pagination/       — pagination utilities

สร้าง interface สำหรับ repository/service เพื่อ mock ตอน test
สร้าง standard response: { success, data, message, meta }
สร้าง error response: { success:false, error: { code, message, details } }
```

โครงสร้างที่ได้:

```
shopfast-api/
├── cmd/api/main.go
├── cmd/migrate/main.go
├── internal/
│   ├── config/config.go
│   ├── database/{connection,migrate,seed}.go
│   ├── dto/response.go
│   ├── handlers/health.go
│   ├── middleware/{error_handler,logger,recover}.go
│   ├── models/{user,product,category,order,order_item,payment,address,review}.go
│   ├── repository/   (ว่าง — สร้างใน Step 5)
│   └── service/       (ว่าง — สร้างใน Step 5)
├── pkg/{response,validator,pagination}/
├── migrations/000001_init_schema.{up,down}.sql
├── go.mod, Makefile, .env.example, CLAUDE.md
```

Standard response format (snippet):

```go
// pkg/response/response.go
func OK(c *fiber.Ctx, data interface{}, message string) error {
    return c.JSON(Response{Success: true, Data: data, Message: message})
}
func Error(c *fiber.Ctx, status int, code, message string) error {
    return c.Status(status).JSON(ErrorResponse{
        Success: false, Error: ErrorDetail{Code: code, Message: message},
    })
}
```

💡 **เคล็ดลับ:** สร้าง interfaces ตั้งแต่แรกทำให้เขียน unit test ง่ายมากโดยใช้ mock — ไม่ต้องต่อ database จริงตอน test business logic

---

## Step 5: Backend Dev สร้าง Base CRUD

**วันที่ 7–8 (สัปดาห์ 2, Day 4–5):**

👤 **คุณแบ็ค** สร้าง Product CRUD — เลือกเป็น resource แรกเพราะมี relationships หลายแบบ:

```
สร้าง Product CRUD ครบ 3 layers:
Repository: Create, GetByID, List (pagination+filter+sort), Update, Delete
Service: business logic — validate price>0, generate slug, check SKU unique
Handler: POST/GET/GET:id/PUT/DELETE /api/v1/products
DTO: CreateProductRequest, UpdateProductRequest, ProductResponse

ใช้ dependency injection, pkg/response, pkg/pagination
Include Preload("Categories") เมื่อ load Product
```

ตัวอย่าง repository layer (List method):

```go
// internal/repository/product_repository.go (snippet)
func (r *productRepository) List(params pagination.Params, filters ProductFilters) ([]models.Product, int64, error) {
    var products []models.Product
    var total int64
    query := r.db.Model(&models.Product{})

    if filters.Search != "" {
        query = query.Where("name ILIKE ?", "%"+filters.Search+"%")
    }
    if filters.CategoryID > 0 {
        query = query.Joins("JOIN product_categories pc ON pc.product_id = products.id").
            Where("pc.category_id = ?", filters.CategoryID)
    }
    query.Count(&total)
    err := query.Preload("Categories").Scopes(pagination.Paginate(params)).Find(&products).Error
    return products, total, err
}
```

ตัวอย่าง handler layer:

```go
// internal/handlers/product_handler.go (snippet)
func (h *ProductHandler) Create(c *fiber.Ctx) error {
    var req dto.CreateProductRequest
    if err := c.BodyParser(&req); err != nil {
        return response.Error(c, 400, "INVALID_BODY", "Invalid request body")
    }
    if errs := validator.Validate(req); errs != nil {
        return response.ValidationError(c, errs)
    }
    product, err := h.service.CreateProduct(req)
    if err != nil {
        return response.Error(c, 500, "CREATE_FAILED", err.Error())
    }
    return response.OK(c, dto.ToProductResponse(product), "Product created")
}
```

📌 **Cross-ref:** ดู CRUD patterns ใน [บทที่ 4](../04_fullstack_collaboration.md) หัวข้อ 4.1

💡 **เคล็ดลับ:** สร้าง CRUD แรกให้สมบูรณ์แล้วใช้เป็น "template" — prompt Claude Code ว่า "สร้าง Category CRUD ตาม pattern เดียวกับ Product" ได้เลย

---

## Step 6: Backend Dev เพิ่ม Input Validation

**วันที่ 9 (สัปดาห์ 3, Day 1):**

👤 **คุณแบ็ค** สร้างระบบ validation ด้วย `go-playground/validator`:

```
สร้าง input validation สำหรับ ShopFast API:
1. pkg/validator/ ใช้ go-playground/validator v10
2. แปลง errors เป็น { field, message } format
3. DTOs พร้อม tags: name(required,min=2,max=255), price(required,gt=0),
   stock_quantity(gte=0), sku(required,max=50), category_ids(required,min=1)
4. UpdateProductRequest — ทุก field optional (pointer + omitempty)
```

DTO ที่ได้:

```go
// internal/dto/product_dto.go (snippet)
type CreateProductRequest struct {
    Name          string  `json:"name" validate:"required,min=2,max=255"`
    Price         float64 `json:"price" validate:"required,gt=0"`
    StockQuantity int     `json:"stock_quantity" validate:"gte=0"`
    SKU           string  `json:"sku" validate:"required,max=50"`
    CategoryIDs   []uint  `json:"category_ids" validate:"required,min=1,dive,gt=0"`
}

type UpdateProductRequest struct {
    Name  *string  `json:"name,omitempty" validate:"omitempty,min=2,max=255"`
    Price *float64 `json:"price,omitempty" validate:"omitempty,gt=0"`
    // ... pointer fields สำหรับ partial update ...
}
```

💡 **เคล็ดลับ:** `UpdateProductRequest` ใช้ pointers (`*string`) เพื่อแยก "ส่ง empty string" กับ "ไม่ส่ง field นี้" — สำคัญมากสำหรับ partial update

⚠️ **ข้อควรระวัง:** `category_ids` ต้องใส่ `dive,gt=0` เพื่อ validate แต่ละ element ใน array ด้วย ไม่ใช่แค่ตรวจว่า array ไม่ว่าง

---

## Step 7: SA Reviews Code Structure

**วันที่ 10 (สัปดาห์ 3, Day 2):**

👤 **คุณเอส** (SA) review โค้ดด้วย Claude Code:

```
Review Go project structure ตาม three-layer architecture:
1. Layer separation — handler ไม่ import gorm, repository ไม่ import fiber
2. Dependency direction — handler → service → repository (ไม่ย้อนกลับ)
3. Interface usage — ทุก layer มี interface
4. Naming conventions — consistent ทั้ง project
5. Error handling — errors wrap ด้วย context
6. No circular dependency
```

**ผล review:**

```
✅ ผ่าน: Layer separation, dependency direction, interfaces, naming

⚠️ ข้อเสนอแนะ:
1. repo.List() ยาวเกินไป → แยก filter building เป็น private method
2. Service ไม่ wrap errors → ใช้ fmt.Errorf("product service: create: %w", err)
3. ไม่มี logging ใน service → inject zerolog ผ่าน constructor
4. ไม่มี request ID → เพิ่ม request ID middleware สำหรับ tracing
```

คุณแบ็คแก้ตาม review ทั้ง 4 ข้อ

💡 **เคล็ดลับ:** SA review ตั้งแต่ resource แรกช่วยให้ patterns ถูกต้องตั้งแต่ต้น — ถ้าปล่อยไว้แล้วมี 10+ resources จะแก้ยากมาก

📌 **Cross-ref:** ดู architecture review ใน [บทที่ 6: Architecture Design](../06_architecture_design.md)

---

## Step 8: DBA ตั้ง Database Indexing

**วันที่ 10–11 (สัปดาห์ 3, Day 2–3):**

👤 **คุณดีบี** วิเคราะห์ queries ที่ใช้บ่อยและสร้าง indexes:

```
วิเคราะห์ ShopFast queries แล้วแนะนำ indexes:
1. ค้นหาสินค้าด้วยชื่อ (ILIKE) 2. กรองตาม category 3. กรองตามราคา
4. Orders ของ user 5. Orders ตาม status 6. Login by email
7. Reviews ของ product 8. Order number lookup
แนะนำ index type (B-tree, GIN, composite) พร้อมเหตุผล
```

Claude Code แนะนำ indexes:

```sql
-- migrations/000002_add_indexes.up.sql
CREATE EXTENSION IF NOT EXISTS pg_trgm;

-- Product search: GIN สำหรับ ILIKE '%keyword%' (B-tree ทำไม่ได้)
CREATE INDEX idx_products_name_gin ON products USING gin(name gin_trgm_ops);
CREATE INDEX idx_products_price ON products(price);

-- Junction table
CREATE INDEX idx_product_categories_product ON product_categories(product_id);
CREATE INDEX idx_product_categories_category ON product_categories(category_id);

-- Order queries: composite index สำหรับ user + sort by date
CREATE INDEX idx_orders_user_created ON orders(user_id, created_at DESC);
CREATE INDEX idx_orders_status ON orders(status) WHERE deleted_at IS NULL;

-- Review by product
CREATE INDEX idx_reviews_product ON reviews(product_id, created_at DESC);
```

คุณดีบีถามเรื่อง GIN index — Claude Code อธิบายว่า GIN + pg_trgm แตกข้อความเป็น trigrams เพื่อรองรับ ILIKE ที่มี leading wildcard ซึ่ง B-tree ทำไม่ได้ Trade-off คือ write ช้ากว่า ~2-3x แต่สำหรับ products ที่ไม่ update บ่อยถือว่าคุ้ม

⚠️ **ข้อควรระวัง:** GIN index ต้อง enable `pg_trgm` extension ก่อน — production database อาจไม่ grant `CREATE EXTENSION` ให้ app user ต้องให้ DBA run แยก

💡 **เคล็ดลับ:** เขียน down migration ด้วยเสมอ — ตอน development ที่ต้อง reset database บ่อย ๆ จะช่วยมาก

---

## ✅ Checkpoint — สิ้นสุด Phase 2

| # | เรื่อง | ผู้รับผิดชอบ | สถานะ |
|---|--------|-------------|--------|
| 1 | ER diagram ครบ 8 entities + relationships | คุณดีบี | ☐ |
| 2 | GORM models ครบใน `internal/models/` | คุณดีบี | ☐ |
| 3 | Migration strategy ตกลง (hybrid) | คุณดีบี + คุณเอส | ☐ |
| 4 | Initial migration + seed data ทำงานได้ | คุณดีบี | ☐ |
| 5 | Project structure ตาม three-layer | คุณแบ็ค | ☐ |
| 6 | Product CRUD ครบ (create, read, list, update, delete) | คุณแบ็ค | ☐ |
| 7 | Input validation + error response มาตรฐาน | คุณแบ็ค | ☐ |
| 8 | SA reviewed code structure | คุณเอส | ☐ |
| 9 | Database indexes สร้างเสร็จ | คุณดีบี | ☐ |
| 10 | `go build ./...` + `go test ./...` ผ่าน | ทุกคน | ☐ |

**Smoke test:**

```bash
make run
curl http://localhost:3000/api/v1/health                    # → {"success":true}
curl -X POST http://localhost:3000/api/v1/products \
  -H "Content-Type: application/json" \
  -d '{"name":"iPhone 15","price":32900,"stock_quantity":100,"sku":"IP15-001","category_ids":[1]}'
curl "http://localhost:3000/api/v1/products?page=1&search=iphone"
curl -X POST http://localhost:3000/api/v1/products \
  -d '{"name":"","price":-1}'                               # → validation error
```

---

## เคล็ดลับ & ข้อผิดพลาดที่พบบ่อย

### 💡 เคล็ดลับ

1. **ER diagram ก่อนโค้ดเสมอ** — ให้ทีม review text diagram ใน PR ก่อนจะลด rework มาก
2. **CRUD แรกต้องสมบูรณ์ที่สุด** — มันคือ template ที่ทุก resource จะ copy pattern
3. **ใช้ Claude Code สร้าง CRUD ซ้ำ** — "สร้าง Category CRUD ตาม pattern เดียวกับ Product"
4. **Hybrid migration** — AutoMigrate สำหรับ dev, SQL migrations สำหรับ production (ห้ามใช้ AutoMigrate ใน production เด็ดขาด)
5. **SA review ตั้งแต่ resource แรก** — แก้ pattern ให้ถูกก่อนสร้าง resource ที่เหลือ

### ⚠️ ข้อผิดพลาดที่พบบ่อย

| ปัญหา | ตัวอย่าง |
|--------|---------|
| ใช้ float64 สำหรับเงิน | `Price float64` → ใช้ `decimal.Decimal` แทน |
| ลืม Preload | `db.First(&product, id)` → ใช้ `db.Preload("Categories").First(...)` |
| ไม่มี down migration | ทุก `.up.sql` ต้องมี `.down.sql` คู่กัน |
| Circular dependency | handler import repository (ข้าม layer) → handler ต้อง import แค่ service |
| Hard-code config | `gorm.Open("host=localhost...")` → ใช้ `cfg.DatabaseURL` |
| N+1 query | Loop query ทีละตัว → ใช้ `Preload()` ใน query เดียว |
| ไม่ validate ที่ handler | Validate ที่ handler layer เสมอ ก่อนส่งให้ service |

---

## สรุป Phase 2

| สิ่งที่สร้าง | จำนวน | Claude Code ช่วย |
|-------------|--------|-----------------|
| ER diagram | 1 (8 entities) | ออกแบบ + text diagram |
| GORM models | 8 ไฟล์ | generate จาก ER |
| Migrations | 2 ชุด (init + indexes) | สร้าง SQL + review |
| Project structure | ~20 ไฟล์ | scaffold ทั้งหมด |
| Product CRUD | 4 ไฟล์ (handler/service/repo/dto) | generate 3 layers |
| Validation + response | 3 ไฟล์ | generate tags + helpers |

**Prompt ที่ใช้:** ~15 prompts (ดูทั้งหมดใน [07_prompt_catalog.md](07_prompt_catalog.md))

---

← [Phase 1: Inception](01_phase1_inception.md) | [Phase 3: Core Features →](03_phase3_core_features.md)
