# บทที่ 4: การทำงานร่วมกันแบบ Full-Stack กับ Claude Code

> **บทนี้ครอบคลุม:** การใช้ Claude Code ในทุกชั้นของการพัฒนา ตั้งแต่ Backend (Go/Fiber), Frontend (React/TypeScript), Database (PostgreSQL/GORM), CI/CD, Docker และ DEVLOG pattern สำหรับการทำงานเป็นทีม

---

## Table of Contents

- [4.1 Backend Development กับ Claude Code](#41-backend-development)
- [4.2 Frontend Development](#42-frontend-development)
- [4.3 Database Tasks](#43-database-tasks)
- [4.4 CI/CD Integration](#44-cicd-integration)
- [4.5 Docker & Deployment](#45-docker--deployment)
- [4.6 DEVLOG Pattern](#46-devlog-pattern)
- [Cross-references](#cross-references)

---

## 4.1 Backend Development

การพัฒนาฝั่ง Backend (แบ็กเอนด์) ด้วย Go เป็นจุดแข็งของ Claude Code เพราะสามารถอ่าน codebase ที่มีอยู่แล้ว เข้าใจ patterns ที่ใช้งาน และสร้างโค้ดใหม่ที่สอดคล้องกับ conventions ของโปรเจกต์ได้อย่างแม่นยำ

### 4.1.1 สร้าง API Endpoints

เมื่อต้องสร้าง endpoint ใหม่ ควรระบุให้ Claude Code ทราบทั้ง framework, ORM และ architecture pattern ที่ใช้

**Prompt ที่แนะนำสำหรับสร้าง REST API ใน Go Fiber:**

```
สร้าง CRUD endpoints สำหรับ User resource ใน Go Fiber:
- POST /api/v1/users — สร้าง user ใหม่
- GET /api/v1/users/:id — ดึง user ตาม ID
- PUT /api/v1/users/:id — อัพเดท user
- DELETE /api/v1/users/:id — ลบ user
ใช้ GORM เป็น ORM, PostgreSQL, และ three-layer architecture (handler → service → repository)
```

**ผลลัพธ์: สถาปัตยกรรมสามชั้น (Three-Layer Architecture)**

Claude Code จะสร้างโค้ดตาม pattern ที่กำหนด โดยแบ่งเป็นชั้นชัดเจน:

**Handler Layer** (`internal/handlers/user_handler.go`):

```go
package handlers

import (
    "strconv"

    "github.com/gofiber/fiber/v2"
    "github.com/your-org/your-app/internal/dto"
    "github.com/your-org/your-app/internal/service"
)

type UserHandler struct {
    userService service.UserServiceInterface
}

func NewUserHandler(userService service.UserServiceInterface) *UserHandler {
    return &UserHandler{userService: userService}
}

// CreateUser สร้างผู้ใช้ใหม่
func (h *UserHandler) CreateUser(c *fiber.Ctx) error {
    var req dto.CreateUserRequest
    if err := c.BodyParser(&req); err != nil {
        return c.Status(fiber.StatusBadRequest).JSON(dto.ErrorResponse{
            Code:    "INVALID_BODY",
            Message: "Invalid request body",
        })
    }

    result, err := h.userService.CreateUser(c.Context(), &req)
    if err != nil {
        return c.Status(fiber.StatusInternalServerError).JSON(dto.ErrorResponse{
            Code:    "INTERNAL_ERROR",
            Message: "Failed to create user",
        })
    }

    return c.Status(fiber.StatusCreated).JSON(result)
}

// GetUser ดึงข้อมูลผู้ใช้ตาม ID
func (h *UserHandler) GetUser(c *fiber.Ctx) error {
    id, err := strconv.ParseUint(c.Params("id"), 10, 64)
    if err != nil {
        return c.Status(fiber.StatusBadRequest).JSON(dto.ErrorResponse{
            Code:    "INVALID_ID",
            Message: "Invalid user ID format",
        })
    }

    result, err := h.userService.GetUserByID(c.Context(), uint(id))
    if err != nil {
        return c.Status(fiber.StatusNotFound).JSON(dto.ErrorResponse{
            Code:    "NOT_FOUND",
            Message: "User not found",
        })
    }

    return c.JSON(result)
}

// UpdateUser อัพเดทข้อมูลผู้ใช้
func (h *UserHandler) UpdateUser(c *fiber.Ctx) error {
    id, err := strconv.ParseUint(c.Params("id"), 10, 64)
    if err != nil {
        return c.Status(fiber.StatusBadRequest).JSON(dto.ErrorResponse{
            Code:    "INVALID_ID",
            Message: "Invalid user ID format",
        })
    }

    var req dto.UpdateUserRequest
    if err := c.BodyParser(&req); err != nil {
        return c.Status(fiber.StatusBadRequest).JSON(dto.ErrorResponse{
            Code:    "INVALID_BODY",
            Message: "Invalid request body",
        })
    }

    result, err := h.userService.UpdateUser(c.Context(), uint(id), &req)
    if err != nil {
        return c.Status(fiber.StatusInternalServerError).JSON(dto.ErrorResponse{
            Code:    "INTERNAL_ERROR",
            Message: "Failed to update user",
        })
    }

    return c.JSON(result)
}

// DeleteUser ลบผู้ใช้ (soft delete)
func (h *UserHandler) DeleteUser(c *fiber.Ctx) error {
    id, err := strconv.ParseUint(c.Params("id"), 10, 64)
    if err != nil {
        return c.Status(fiber.StatusBadRequest).JSON(dto.ErrorResponse{
            Code:    "INVALID_ID",
            Message: "Invalid user ID format",
        })
    }

    if err := h.userService.DeleteUser(c.Context(), uint(id)); err != nil {
        return c.Status(fiber.StatusInternalServerError).JSON(dto.ErrorResponse{
            Code:    "INTERNAL_ERROR",
            Message: "Failed to delete user",
        })
    }

    return c.SendStatus(fiber.StatusNoContent)
}
```

**Service Layer** (`internal/service/user_service.go`):

```go
package service

import (
    "context"
    "fmt"

    "github.com/your-org/your-app/internal/dto"
    "github.com/your-org/your-app/internal/models"
    "github.com/your-org/your-app/internal/repository"
)

type UserServiceInterface interface {
    CreateUser(ctx context.Context, req *dto.CreateUserRequest) (*dto.UserResponse, error)
    GetUserByID(ctx context.Context, id uint) (*dto.UserResponse, error)
    UpdateUser(ctx context.Context, id uint, req *dto.UpdateUserRequest) (*dto.UserResponse, error)
    DeleteUser(ctx context.Context, id uint) error
}

type userService struct {
    userRepo repository.UserRepositoryInterface
}

func NewUserService(userRepo repository.UserRepositoryInterface) UserServiceInterface {
    return &userService{userRepo: userRepo}
}

func (s *userService) CreateUser(ctx context.Context, req *dto.CreateUserRequest) (*dto.UserResponse, error) {
    user := &models.User{
        FullName: req.FullName,
        Email:    req.Email,
    }

    if err := s.userRepo.Create(ctx, user); err != nil {
        return nil, fmt.Errorf("failed to create user: %w", err)
    }

    return dto.ToUserResponse(user), nil
}
```

**Repository Layer** (`internal/repository/user_repository.go`):

```go
package repository

import (
    "context"

    "github.com/your-org/your-app/internal/models"
    "gorm.io/gorm"
)

type UserRepositoryInterface interface {
    Create(ctx context.Context, user *models.User) error
    FindByID(ctx context.Context, id uint) (*models.User, error)
    Update(ctx context.Context, user *models.User) error
    Delete(ctx context.Context, id uint) error
}

type userRepository struct {
    db *gorm.DB
}

func NewUserRepository(db *gorm.DB) UserRepositoryInterface {
    return &userRepository{db: db}
}

func (r *userRepository) Create(ctx context.Context, user *models.User) error {
    return r.db.WithContext(ctx).Create(user).Error
}

func (r *userRepository) FindByID(ctx context.Context, id uint) (*models.User, error) {
    var user models.User
    if err := r.db.WithContext(ctx).First(&user, id).Error; err != nil {
        return nil, err
    }
    return &user, nil
}
```

> **Tip:** ระบุ architecture pattern ใน prompt เสมอ (เช่น "three-layer", "clean architecture") เพื่อให้ Claude Code สร้างโค้ดที่สอดคล้องกับโครงสร้างโปรเจกต์

---

### 4.1.2 Business Logic Patterns

**Service Layer Pattern:**

การเขียน business logic ควรอยู่ใน service layer เท่านั้น ไม่ควรปนอยู่ใน handler หรือ repository

```
"เขียน service method สำหรับ CreateOrder ที่:
1. ตรวจสอบว่า user มีอยู่จริง
2. ตรวจสอบว่า product มี stock เพียงพอ
3. คำนวณราคารวม (รวม discount ถ้ามี)
4. สร้าง order record
5. ลด stock ของ product
ทั้งหมดอยู่ใน database transaction"
```

**Error Handling Patterns ใน Go:**

Claude Code สามารถสร้าง custom error types ที่เหมาะกับโปรเจกต์ได้

```go
// internal/apperror/errors.go
package apperror

import "fmt"

type AppError struct {
    Code    string
    Message string
    Err     error
}

func (e *AppError) Error() string {
    if e.Err != nil {
        return fmt.Sprintf("[%s] %s: %v", e.Code, e.Message, e.Err)
    }
    return fmt.Sprintf("[%s] %s", e.Code, e.Message)
}

func (e *AppError) Unwrap() error {
    return e.Err
}

// สร้าง error constructors สำหรับ common cases
func NotFound(resource string, id interface{}) *AppError {
    return &AppError{
        Code:    "NOT_FOUND",
        Message: fmt.Sprintf("%s with ID %v not found", resource, id),
    }
}

func ValidationFailed(message string) *AppError {
    return &AppError{
        Code:    "VALIDATION_FAILED",
        Message: message,
    }
}

func InternalError(err error) *AppError {
    return &AppError{
        Code:    "INTERNAL_ERROR",
        Message: "An internal error occurred",
        Err:     err,
    }
}
```

**Prompt สำหรับ Error Handling:**

```
"เพิ่ม error handling ใน service/order_service.go ตาม pattern ของ apperror package:
- ใช้ AppError struct
- Wrap error ด้วย fmt.Errorf + %w
- return error ที่ handler สามารถแปลงเป็น HTTP status ได้"
```

**Middleware Patterns:**

```
"สร้าง Fiber middleware สำหรับ:
1. JWT Authentication — ตรวจสอบ token จาก Authorization header
2. Request Logging — log method, path, status, duration
3. Rate Limiting — จำกัด 100 requests/minute ต่อ IP
ตาม pattern middleware ที่มีอยู่ใน internal/middleware/"
```

ตัวอย่าง JWT Middleware ที่ Claude Code สร้างให้:

```go
package middleware

import (
    "strings"

    "github.com/gofiber/fiber/v2"
    "github.com/golang-jwt/jwt/v5"
)

func JWTAuth(secretKey string) fiber.Handler {
    return func(c *fiber.Ctx) error {
        authHeader := c.Get("Authorization")
        if authHeader == "" {
            return c.Status(fiber.StatusUnauthorized).JSON(fiber.Map{
                "code":    "UNAUTHORIZED",
                "message": "Missing authorization header",
            })
        }

        tokenString := strings.TrimPrefix(authHeader, "Bearer ")
        token, err := jwt.Parse(tokenString, func(token *jwt.Token) (interface{}, error) {
            if _, ok := token.Method.(*jwt.SigningMethodRSA); !ok {
                return nil, fmt.Errorf("unexpected signing method: %v", token.Header["alg"])
            }
            return publicKey, nil
        })

        if err != nil || !token.Valid {
            return c.Status(fiber.StatusUnauthorized).JSON(fiber.Map{
                "code":    "INVALID_TOKEN",
                "message": "Invalid or expired token",
            })
        }

        claims := token.Claims.(jwt.MapClaims)
        c.Locals("userID", claims["sub"])
        c.Locals("role", claims["role"])

        return c.Next()
    }
}
```

---

### 4.1.3 Debugging ด้วย Claude Code

การ debug เป็นหนึ่งใน use case ที่ทรงพลังที่สุดของ Claude Code เพราะสามารถวิเคราะห์ error log, อ่านโค้ดที่เกี่ยวข้อง และเสนอวิธีแก้ไขได้ในขั้นตอนเดียว

**Workflow การ Debug:**

```bash
# ส่ง error log ให้ Claude วิเคราะห์
cat error.log | claude -p "วิเคราะห์ error นี้ หาสาเหตุ และเสนอวิธีแก้ไข"

# Debug specific test failure
claude -p "รัน go test ./internal/service/... แล้ววิเคราะห์ test ที่ fail"

# วิเคราะห์ panic stack trace
claude -p "อ่าน panic stack trace นี้ แล้วหาสาเหตุ: $(cat /tmp/panic.log)"

# Debug race condition
claude -p "รัน go test -race ./... แล้ววิเคราะห์ data race ที่พบ"
```

**Prompt สำหรับ Debug ที่ซับซ้อน:**

```
"API endpoint POST /api/v1/orders return 500
1. อ่าน handler, service, repository ที่เกี่ยวข้อง
2. ดู error log ล่าสุด
3. หาสาเหตุ
4. เสนอวิธีแก้ไขพร้อม test case"
```

> **Note:** Claude Code สามารถรัน `go test` โดยตรงและวิเคราะห์ผลลัพธ์ได้ทันที ไม่ต้องคัดลอก output มาวางเอง

---

## 4.2 Frontend Development

การพัฒนาฝั่ง Frontend (ฟรอนต์เอนด์) กับ Claude Code มีจุดเด่นคือสามารถอ้างอิงข้ามโปรเจกต์ (Cross-Project References) ได้ ทำให้ Frontend และ Backend ทำงานสอดคล้องกันเสมอ

### 4.2.1 Cross-Project References

```bash
# เปิด Claude Code พร้อมอ้างอิง backend project
claude --add-dir /path/to/backend-api

# ให้ Claude อ่าน API types แล้วสร้าง TypeScript types
claude -p "อ่าน Go struct ใน models/user.go แล้วสร้าง TypeScript interface ที่ตรงกัน"
```

เมื่อใช้ `--add-dir` Claude Code จะสามารถ:
- อ่านไฟล์จาก backend project ได้โดยตรง
- เปรียบเทียบ Go struct กับ TypeScript interface
- ตรวจจับความไม่สอดคล้องระหว่าง API response กับ frontend type

**ตัวอย่าง: Go Struct to TypeScript Interface**

จาก Go struct:

```go
// models/user.go
type User struct {
    ID        uint           `json:"id" gorm:"primaryKey"`
    FullName  string         `json:"full_name" gorm:"not null"`
    Email     string         `json:"email" gorm:"uniqueIndex;not null"`
    Role      string         `json:"role" gorm:"default:'user'"`
    CreatedAt time.Time      `json:"created_at"`
    UpdatedAt time.Time      `json:"updated_at"`
    DeletedAt gorm.DeletedAt `json:"-" gorm:"index"`
}
```

Claude Code จะสร้าง TypeScript interface:

```typescript
// types/user.ts
export interface User {
  id: number;
  full_name: string;
  email: string;
  role: string;
  created_at: string; // ISO 8601 datetime
  updated_at: string; // ISO 8601 datetime
}

export interface CreateUserRequest {
  full_name: string;
  email: string;
  role?: string;
}

export interface UpdateUserRequest {
  full_name?: string;
  email?: string;
  role?: string;
}
```

---

### 4.2.2 อัพเดท TypeScript Types ตาม API Changes

เมื่อ API เปลี่ยนแปลง ต้องอัพเดท frontend ให้สอดคล้อง workflow มีดังนี้:

**Workflow:** API เปลี่ยน -> Claude อ่าน Go struct -> สร้าง/อัพเดท TS types -> อัพเดท React components -> อัพเดท tests

**Prompt ตัวอย่าง:**

```
API response ของ GET /api/v1/users เปลี่ยน field "name" เป็น "full_name"
1. อัพเดท TypeScript interface ใน types/user.ts
2. อัพเดท React components ที่ใช้ field นี้
3. อัพเดท test files ที่เกี่ยวข้อง
```

Claude Code จะ:
1. ค้นหา TypeScript interface ที่มี field `name`
2. เปลี่ยนเป็น `full_name` ทุกที่
3. ค้นหา React components ที่ใช้ `user.name` แล้วเปลี่ยนเป็น `user.full_name`
4. อัพเดท test files, mock data, และ fixtures ที่เกี่ยวข้อง

**Prompt สำหรับตรวจสอบความสอดคล้อง:**

```
"เปรียบเทียบ Go struct ใน models/ กับ TypeScript interface ใน types/
หา fields ที่ไม่ตรงกัน และแก้ไข TypeScript ให้ตรงกับ Go"
```

> **Warning:** เมื่อใช้ `--add-dir` ให้ระวังเรื่องสิทธิ์การเข้าถึงไฟล์ Claude Code จะถามยืนยันก่อนแก้ไขไฟล์ในโปรเจกต์ที่อ้างอิง

---

### 4.2.3 React Component Patterns

**สร้าง Component จาก Design Spec:**

```
"สร้าง React component UserProfileCard:
- แสดง avatar, full_name, email, role
- ใช้ Tailwind CSS
- มี loading skeleton state
- มี error state
- รองรับ dark mode
ตาม pattern ของ component ใน src/components/"
```

**Form Handling with Validation:**

```
"สร้าง form สำหรับ CreateUser:
- ใช้ React Hook Form + Zod validation
- Fields: full_name (required, min 2), email (required, valid format), role (select)
- แสดง validation errors แบบ inline
- Submit ผ่าน useMutation (React Query)
- แสดง toast notification เมื่อสำเร็จ/ล้มเหลว"
```

**API Integration ด้วย React Query:**

```typescript
// ตัวอย่างที่ Claude Code สร้างให้
import { useQuery, useMutation, useQueryClient } from "@tanstack/react-query";
import { api } from "@/lib/api";
import type { User, CreateUserRequest } from "@/types/user";

// Query hook
export function useUser(id: number) {
  return useQuery({
    queryKey: ["users", id],
    queryFn: () => api.get<User>(`/api/v1/users/${id}`),
    enabled: !!id,
  });
}

// Mutation hook
export function useCreateUser() {
  const queryClient = useQueryClient();

  return useMutation({
    mutationFn: (data: CreateUserRequest) =>
      api.post<User>("/api/v1/users", data),
    onSuccess: () => {
      queryClient.invalidateQueries({ queryKey: ["users"] });
    },
  });
}
```

> **Tip:** เมื่อให้ Claude Code สร้าง React components ควรระบุ state management pattern ที่ใช้ (React Query, Zustand, Redux) เพื่อให้ได้โค้ดที่สอดคล้องกับโปรเจกต์

---

## 4.3 Database Tasks

การจัดการฐานข้อมูล (Database) ด้วย Claude Code ครอบคลุมตั้งแต่การออกแบบ model, เขียน migration, ไปจนถึงการ optimize query

### 4.3.1 GORM Models

**Prompt สำหรับสร้าง Model ที่มี Relations:**

```
"สร้าง GORM model สำหรับ Order ที่มี relation กับ User และ Product:
- Order belongs to User
- Order has many OrderItems
- OrderItem belongs to Product
- ใช้ soft delete
- มี JSON tags ตาม API spec"
```

**ตัวอย่าง GORM Model ที่ Claude Code สร้าง:**

```go
package models

import (
    "time"

    "gorm.io/gorm"
)

type Order struct {
    ID          uint           `json:"id" gorm:"primaryKey"`
    UserID      uint           `json:"user_id" gorm:"not null;index"`
    User        User           `json:"user,omitempty" gorm:"foreignKey:UserID"`
    OrderItems  []OrderItem    `json:"order_items,omitempty" gorm:"foreignKey:OrderID"`
    Status      string         `json:"status" gorm:"type:varchar(20);default:'pending';index"`
    TotalAmount float64        `json:"total_amount" gorm:"type:decimal(10,2);not null"`
    Note        *string        `json:"note,omitempty" gorm:"type:text"`
    OrderedAt   time.Time      `json:"ordered_at" gorm:"not null"`
    CreatedAt   time.Time      `json:"created_at"`
    UpdatedAt   time.Time      `json:"updated_at"`
    DeletedAt   gorm.DeletedAt `json:"-" gorm:"index"`
}

type OrderItem struct {
    ID        uint    `json:"id" gorm:"primaryKey"`
    OrderID   uint    `json:"order_id" gorm:"not null;index"`
    ProductID uint    `json:"product_id" gorm:"not null;index"`
    Product   Product `json:"product,omitempty" gorm:"foreignKey:ProductID"`
    Quantity  int     `json:"quantity" gorm:"not null"`
    UnitPrice float64 `json:"unit_price" gorm:"type:decimal(10,2);not null"`
    Subtotal  float64 `json:"subtotal" gorm:"type:decimal(10,2);not null"`
}
```

**Conventions สำหรับ GORM Models:**

| Convention | ตัวอย่าง | คำอธิบาย |
|-----------|---------|---------|
| Naming | `UserID` ไม่ใช่ `UserId` | ใช้ PascalCase ตาม Go convention |
| JSON tags | `json:"user_id"` | ใช้ snake_case สำหรับ JSON |
| Index | `gorm:"index"` | เพิ่ม index สำหรับ FK และ fields ที่ query บ่อย |
| Soft delete | `gorm.DeletedAt` | ใช้ GORM built-in soft delete |
| Omit in JSON | `json:"-"` | ซ่อน fields เช่น DeletedAt จาก API response |
| Pointer fields | `*string` | ใช้ pointer สำหรับ nullable fields |

---

### 4.3.2 Migration Strategy

**สร้าง Migration:**

```bash
# ให้ Claude สร้าง migration
claude -p "สร้าง SQL migration สำหรับเพิ่ม orders table ที่มี foreign key ไปยัง users"
```

**ตัวอย่าง Migration ที่ Claude Code สร้าง:**

```sql
-- migrations/000003_create_orders_table.up.sql
CREATE TABLE IF NOT EXISTS orders (
    id BIGSERIAL PRIMARY KEY,
    user_id BIGINT NOT NULL REFERENCES users(id) ON DELETE RESTRICT,
    status VARCHAR(20) NOT NULL DEFAULT 'pending',
    total_amount DECIMAL(10, 2) NOT NULL,
    note TEXT,
    ordered_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    updated_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    deleted_at TIMESTAMPTZ
);

CREATE INDEX idx_orders_user_id ON orders(user_id);
CREATE INDEX idx_orders_status ON orders(status);
CREATE INDEX idx_orders_deleted_at ON orders(deleted_at);

-- migrations/000003_create_orders_table.down.sql
DROP TABLE IF EXISTS orders;
```

**Review Migration ก่อน Apply:**

```
"review migration file ล่าสุด ตรวจสอบ:
1. มี rollback (down) ที่ถูกต้อง
2. มี index ที่จำเป็น
3. ไม่มี breaking changes
4. data types เหมาะสมกับ use case
5. foreign key constraints ถูกต้อง"
```

> **Warning:** ควร review migration ทุกครั้งก่อน apply ไปยัง production โดยเฉพาะ migration ที่มี `DROP`, `ALTER`, หรือ `DELETE` ต้องตรวจสอบ rollback plan ให้ละเอียด

---

### 4.3.3 Query Optimization

การ optimize query เป็น use case ที่ Claude Code ช่วยได้มาก เพราะสามารถอ่านโค้ดทั้ง repository layer แล้ววิเคราะห์ปัญหาที่พบบ่อย

**Prompt สำหรับวิเคราะห์ Query:**

```
วิเคราะห์ repository/order.go:
1. หา N+1 query problems
2. เสนอวิธีแก้ด้วย GORM Preload/Joins
3. แนะนำ index ที่ควรเพิ่ม
```

**ตัวอย่าง N+1 Problem ที่ Claude Code ตรวจพบ:**

```go
// ก่อนแก้ไข: N+1 query problem
func (r *orderRepo) ListOrders(ctx context.Context) ([]models.Order, error) {
    var orders []models.Order
    // Query 1: ดึง orders ทั้งหมด
    r.db.WithContext(ctx).Find(&orders)

    for i := range orders {
        // Query N: ดึง user สำหรับแต่ละ order (N+1!)
        r.db.WithContext(ctx).First(&orders[i].User, orders[i].UserID)
        // Query N: ดึง order items สำหรับแต่ละ order (N+1!)
        r.db.WithContext(ctx).Where("order_id = ?", orders[i].ID).Find(&orders[i].OrderItems)
    }

    return orders, nil
}

// หลังแก้ไข: ใช้ Preload
func (r *orderRepo) ListOrders(ctx context.Context) ([]models.Order, error) {
    var orders []models.Order
    err := r.db.WithContext(ctx).
        Preload("User").
        Preload("OrderItems").
        Preload("OrderItems.Product").
        Find(&orders).Error

    return orders, err
}
```

**Prompt สำหรับวิเคราะห์ Slow Query:**

```
"วิเคราะห์ slow query นี้:
SELECT * FROM orders WHERE user_id = ? AND status = ? ORDER BY created_at DESC

1. อธิบายว่าทำไมช้า
2. แนะนำ index ที่ควรเพิ่ม
3. เสนอ query ที่ optimize แล้ว"
```

Claude Code อาจแนะนำ composite index (ดัชนีแบบรวม):

```sql
-- Composite index สำหรับ query pattern ที่ใช้บ่อย
CREATE INDEX idx_orders_user_status_created
ON orders(user_id, status, created_at DESC);
```

> **Tip:** ให้ Claude Code วิเคราะห์ผลลัพธ์ของ `EXPLAIN ANALYZE` เพื่อหาจุดคอขวด (bottleneck) ที่แท้จริง

---

## 4.4 CI/CD Integration

Claude Code สามารถรวมเข้ากับ CI/CD pipeline (ระบบส่งมอบอย่างต่อเนื่อง) ได้ทั้งแบบ GitHub Actions, GitLab CI และ Jenkins โดยทำงานในโหมด non-interactive

### 4.4.1 GitHub Actions with Claude Code

**Automated Code Review:**

```yaml
# .github/workflows/claude-review.yml
name: Claude Code Review
on:
  pull_request:
    types: [opened, synchronize]

jobs:
  review:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0

      - uses: anthropics/claude-code-action@v1
        with:
          model: sonnet
          prompt: |
            Review this PR for:
            1. Code quality and Go best practices
            2. Security vulnerabilities
            3. Performance issues
            4. Test coverage
          anthropic_api_key: ${{ secrets.ANTHROPIC_API_KEY }}
```

**Prompt ใน CI ที่ละเอียดขึ้น:**

```yaml
      - uses: anthropics/claude-code-action@v1
        with:
          model: sonnet
          prompt: |
            Review this PR focusing on:
            1. Go best practices (error handling, naming, package structure)
            2. SQL injection vulnerabilities
            3. N+1 query problems in GORM code
            4. Missing input validation
            5. Proper use of context.Context
            6. Test coverage for new code paths
            Provide specific line-by-line feedback.
          anthropic_api_key: ${{ secrets.ANTHROPIC_API_KEY }}
```

---

### 4.4.2 Non-Interactive Mode สำหรับ CI

ในสภาพแวดล้อม CI/CD ต้องใช้ Claude Code แบบ non-interactive เสมอ:

```bash
# รัน tests และ report ผลลัพธ์
claude -p "run all tests and report failures" --max-turns 5

# สร้าง changelog อัตโนมัติ
claude -p "generate changelog from git log since last release" --output-format json

# แก้ไข lint issues อัตโนมัติ
claude -p "fix all golangci-lint issues in the codebase" --max-turns 10

# ตรวจสอบ breaking changes
claude -p "compare API contracts between main and this branch, report breaking changes"
```

**สำคัญ:** ตั้ง `--max-turns` ให้เหมาะสมเพื่อป้องกัน Claude Code ทำงานนานเกินไปใน CI:

| Task | --max-turns แนะนำ | เหตุผล |
|------|-------------------|--------|
| Code review | 3-5 | อ่านและวิเคราะห์เท่านั้น |
| Fix lint issues | 10-15 | ต้องแก้หลายไฟล์ |
| Generate tests | 10-20 | สร้างและรัน tests |
| Full refactor | 20-30 | แก้ไขหลายไฟล์ ต้อง verify |

---

### 4.4.3 AWS Bedrock / Google Vertex Integration

สำหรับองค์กรที่ต้องการ compliance หรือ data residency สามารถใช้ Claude Code ผ่าน managed service ได้:

**AWS Bedrock:**

```bash
# ตั้งค่า environment variables
export CLAUDE_CODE_USE_BEDROCK=1
export AWS_REGION=us-east-1
export AWS_ACCESS_KEY_ID=your-access-key
export AWS_SECRET_ACCESS_KEY=your-secret-key

# ใช้งานเหมือนปกติ
claude -p "run tests"
```

**Google Vertex AI:**

```bash
# ตั้งค่า environment variables
export CLAUDE_CODE_USE_VERTEX=1
export CLOUD_ML_REGION=us-central1
export ANTHROPIC_VERTEX_PROJECT_ID=my-project

# ใช้งานเหมือนปกติ
claude -p "run tests"
```

**เมื่อไหร่ควรใช้ Bedrock/Vertex:**

| สถานการณ์ | แนะนำ | เหตุผล |
|-----------|--------|--------|
| Data residency requirement | Bedrock/Vertex | ข้อมูลไม่ออกนอก region |
| Enterprise compliance (SOC2, HIPAA) | Bedrock/Vertex | ผ่าน AWS/GCP compliance |
| ใช้ร่วมกับ AWS/GCP billing | Bedrock/Vertex | รวม billing เดียวกัน |
| ต้องการ VPC isolation | Bedrock/Vertex | ใช้ Private Endpoint ได้ |
| ทีมเล็ก, startup | Anthropic API ตรง | ง่าย ตั้งค่าเร็ว |
| ทดลองใช้งาน | Anthropic API ตรง | ไม่ต้อง setup cloud infra |

> **Note:** เมื่อใช้ Bedrock/Vertex ให้ตรวจสอบว่า model version ที่ต้องการใช้ available ใน region นั้นแล้ว ไม่ใช่ทุก region จะมีทุก model

---

## 4.5 Docker & Deployment

Claude Code ช่วยสร้างและจัดการ Docker configuration (การกำหนดค่าคอนเทนเนอร์) ได้อย่างมีประสิทธิภาพ ตั้งแต่ Dockerfile จนถึง Docker Compose สำหรับ development

### 4.5.1 สร้าง Dockerfile

**Prompt สำหรับ Multi-Stage Dockerfile:**

```
"สร้าง multi-stage Dockerfile สำหรับ Go API:
- Build stage: Go 1.22
- Runtime stage: Alpine
- Include health check
- Non-root user
- Optimize สำหรับ Docker layer caching"
```

**ตัวอย่าง Dockerfile ที่ Claude Code สร้าง:**

```dockerfile
# Build stage
FROM golang:1.22-alpine AS builder

RUN apk add --no-cache git ca-certificates

WORKDIR /app

# Copy go.mod/go.sum first for layer caching
COPY go.mod go.sum ./
RUN go mod download

# Copy source code
COPY . .

# Build binary
RUN CGO_ENABLED=0 GOOS=linux GOARCH=amd64 \
    go build -ldflags="-w -s" -o /app/server ./cmd/api

# Runtime stage
FROM alpine:3.19

RUN apk add --no-cache ca-certificates tzdata curl

# Create non-root user
RUN addgroup -S appgroup && adduser -S appuser -G appgroup

WORKDIR /app

# Copy binary from builder
COPY --from=builder /app/server .
COPY --from=builder /app/configs ./configs

# Switch to non-root user
USER appuser

EXPOSE 8080

# Health check
HEALTHCHECK --interval=30s --timeout=5s --start-period=10s --retries=3 \
    CMD curl -f http://localhost:8080/health || exit 1

ENTRYPOINT ["./server"]
```

---

### 4.5.2 Docker Compose สำหรับ Development

**Prompt:**

```
"สร้าง docker-compose.yml สำหรับ development ที่มี:
- Go API service (hot reload ด้วย Air)
- PostgreSQL 16
- Redis
- Adminer สำหรับดู database
- Network isolation"
```

**ตัวอย่าง Docker Compose:**

```yaml
# docker-compose.yml
version: "3.8"

services:
  api:
    build:
      context: .
      dockerfile: Dockerfile.dev
    ports:
      - "8080:8080"
    volumes:
      - .:/app
    environment:
      - DB_HOST=postgres
      - DB_PORT=5432
      - DB_USER=devuser
      - DB_PASSWORD=devpass
      - DB_NAME=myapp_dev
      - REDIS_URL=redis://redis:6379
    depends_on:
      postgres:
        condition: service_healthy
      redis:
        condition: service_started
    networks:
      - app-network

  postgres:
    image: postgres:16-alpine
    ports:
      - "5432:5432"
    environment:
      POSTGRES_USER: devuser
      POSTGRES_PASSWORD: devpass
      POSTGRES_DB: myapp_dev
    volumes:
      - postgres-data:/var/lib/postgresql/data
      - ./scripts/init.sql:/docker-entrypoint-initdb.d/init.sql
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U devuser -d myapp_dev"]
      interval: 5s
      timeout: 5s
      retries: 5
    networks:
      - app-network

  redis:
    image: redis:7-alpine
    ports:
      - "6379:6379"
    networks:
      - app-network

  adminer:
    image: adminer
    ports:
      - "8081:8080"
    networks:
      - app-network

volumes:
  postgres-data:

networks:
  app-network:
    driver: bridge
```

---

### 4.5.3 Deployment Assistance

**Prompt ตัวอย่าง:**

```
"ช่วย deploy Go API ไป AWS ECS:
1. สร้าง ECS task definition
2. สร้าง service configuration
3. ตั้งค่า ALB health check
4. ตั้งค่า environment variables จาก AWS Secrets Manager"
```

```
"ตรวจสอบ Kubernetes deployment.yaml:
1. resource limits เหมาะสมหรือไม่
2. liveness/readiness probe ถูกต้องหรือไม่
3. มี security context หรือไม่
4. แนะนำ best practices ที่ขาดไป"
```

> **Tip:** เมื่อให้ Claude Code ช่วยเรื่อง deployment ควรระบุ cloud provider และ infrastructure pattern (ECS, EKS, Cloud Run, etc.) ที่ใช้อยู่ เพื่อให้ได้คำแนะนำที่ตรงจุด

---

## 4.6 DEVLOG Pattern

DEVLOG (Development Log) เป็น pattern สำหรับบันทึกสิ่งที่ทำในแต่ละ session ช่วยให้ทีมติดตามงานได้ และเป็นประโยชน์เมื่อกลับมาทำงานต่อ

### 4.6.1 โครงสร้าง DEVLOG

สร้างไฟล์ `DEVLOG.md` ที่ root ของโปรเจกต์:

```markdown
# DEVLOG

## 2024-01-15 — Feature: User Authentication
### สิ่งที่ทำ
- สร้าง JWT middleware ใน internal/middleware/jwt.go
- เพิ่ม login/register endpoints
- เขียน unit tests สำหรับ auth service

### Decisions
- ใช้ RS256 แทน HS256 เพราะรองรับ key rotation ได้ง่ายกว่า
- เก็บ refresh token ใน database แทน Redis เพราะต้องการ persistence

### Issues
- CORS issue กับ frontend — แก้โดยเพิ่ม AllowCredentials ใน CORS middleware
- JWT token ไม่หมดอายุใน test — แก้โดยใช้ mock clock

### Next Steps
- เพิ่ม refresh token endpoint
- เพิ่ม logout (revoke token)
- เขียน integration tests

---

## 2024-01-14 — Setup: Project Initialization
### สิ่งที่ทำ
- สร้างโครงสร้างโปรเจกต์ตาม clean architecture
- ตั้งค่า Docker Compose (Go API + PostgreSQL + Redis)
- เพิ่ม golangci-lint configuration

### Decisions
- ใช้ Fiber แทน Gin เพราะ performance ดีกว่าใน benchmark
- ใช้ golang-migrate แทน GORM AutoMigrate เพราะควบคุม migration ได้ดีกว่า

### Issues
- ไม่มี

### Next Steps
- สร้าง User CRUD
- ตั้งค่า CI/CD
```

---

### 4.6.2 ใช้ Claude Code เขียน DEVLOG Entry

หลังจบ task ให้ Claude Code สรุปสิ่งที่ทำเป็น DEVLOG entry:

```bash
# สรุป session เป็น DEVLOG entry
claude -p "สรุปสิ่งที่ทำใน session นี้เป็น DEVLOG entry format:
- สิ่งที่ทำ (รายการ)
- Decisions ที่ตัดสินใจ (พร้อมเหตุผล)
- Issues ที่พบ (และวิธีแก้)
- Next Steps"
```

**Prompt เพิ่มเติมสำหรับ DEVLOG:**

```
"อ่าน git diff ของ commit ล่าสุด 5 commits แล้วสร้าง DEVLOG entry สรุปการเปลี่ยนแปลง"
```

```
"อ่าน DEVLOG.md แล้วสรุป Next Steps ที่ยังไม่ได้ทำเป็น TODO list"
```

---

### 4.6.3 Best Practices สำหรับ DEVLOG

1. **เขียนทุกวัน** - บันทึกทุก session ที่มีการเปลี่ยนแปลงสำคัญ
2. **บันทึก Decision** - เหตุผลที่เลือก approach นี้ มีค่ามากกว่าตัวโค้ด
3. **บันทึก Issues** - ปัญหาและวิธีแก้ช่วยทีมไม่ต้องเจอปัญหาซ้ำ
4. **Next Steps ชัดเจน** - ทำให้กลับมาทำงานต่อได้เร็ว
5. **อ้างอิงไฟล์** - ระบุ path ของไฟล์ที่เปลี่ยนแปลง

**เชื่อมกับ CLAUDE.md:**

เพิ่มคำสั่งใน `CLAUDE.md` (ไฟล์กำหนดพฤติกรรม Claude Code) ให้เตือนเรื่อง DEVLOG:

```markdown
# Project Instructions

## DEVLOG
- หลังจบ task สำคัญ ให้ถามว่าต้องการเขียน DEVLOG entry หรือไม่
- DEVLOG format อยู่ใน DEVLOG.md
- ใส่วันที่และหัวข้อ task ทุกครั้ง
```

> **Tip:** ใช้ DEVLOG ร่วมกับ git commit messages ได้ โดย DEVLOG จะให้รายละเอียดมากกว่า commit message เช่น เหตุผลเบื้องหลังการตัดสินใจ และปัญหาที่พบระหว่างทาง

---

## Cross-references

- [บทที่ 1: บทนำและพื้นฐาน](./01_intro.md)
- [บทที่ 2: Agentic Mastery](./02_agentic_mastery.md)
- [บทที่ 3: การใช้งานขั้นสูง](./03_advanced_usage.md)
- [บทที่ 5: Specification-Driven Development](./05_spec_driven_dev.md)
- [บทที่ 6: Architecture & Design](./06_architecture_design.md)
- [บทที่ 7: QA, Security, and Review](./07_qa_security_review.md)
- [บทที่ 8: บทบาทและความรับผิดชอบ](./08_roles_responsibilities.md)
- [บทที่ 9: Cheat Sheet](./09_cheat_sheet.md)

---

> **สิ้นสุดบทที่ 4** — บทต่อไป: [บทที่ 5: Specification-Driven Development](./05_spec_driven_dev.md)
