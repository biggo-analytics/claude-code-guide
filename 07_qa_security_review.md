# การตรวจสอบคุณภาพ ความปลอดภัย และการรีวิว

## QA, Security, and Review

> **สารบัญ**
>
> - [7.1 Code Review ด้วย Claude Code](#71-code-review-ด้วย-claude-code)
> - [7.2 Testing ด้วย Claude Code](#72-testing-ด้วย-claude-code)
> - [7.3 Permission System (ระบบสิทธิ์)](#73-permission-system-ระบบสิทธิ์)
> - [7.4 MCP Server Security](#74-mcp-server-security)
> - [7.5 Sandbox Mode](#75-sandbox-mode)
> - [7.6 Security Best Practices](#76-security-best-practices)
> - [7.7 Performance Review](#77-performance-review)
> - [7.8 Review Checklist Templates](#78-review-checklist-templates)

---

## 7.1 Code Review ด้วย Claude Code

การรีวิวโค้ด (Code Review) เป็นขั้นตอนสำคัญในกระบวนการพัฒนาซอฟต์แวร์
Claude Code สามารถช่วยรีวิวโค้ดได้ทั้งแบบ Manual และ Automated ผ่าน CI/CD pipeline

### Manual Review (Plan Mode)

เข้าสู่ Plan Mode ด้วย `Shift+Tab` แล้วพิมพ์คำสั่งรีวิว:

```
เข้า Plan Mode (Shift+Tab) แล้ว:
"Review code ใน internal/handler/order.go:
1. Code quality และ Go best practices
2. Error handling ครบถ้วนหรือไม่
3. Input validation
4. Performance concerns
5. Security vulnerabilities"
```

Claude จะวิเคราะห์โค้ดและแสดงผลเป็นรายงานครอบคลุมทุกหัวข้อที่ระบุ
Plan Mode เหมาะสำหรับการรีวิวที่ต้องการความละเอียด เพราะ Claude จะวางแผนก่อนตอบ
ไม่ทำการแก้ไขโค้ดจนกว่าจะได้รับอนุญาต

**ตัวอย่าง prompt สำหรับรีวิวเฉพาะด้าน:**

```
# รีวิว Error Handling
"ตรวจสอบ error handling ใน internal/handler/ ทุกไฟล์:
- มี error ที่ถูก swallow (กลืน) โดยไม่ log หรือไม่
- return error message มี sensitive information หรือไม่
- error wrapping ใช้ fmt.Errorf กับ %w ถูกต้องหรือไม่"
```

```
# รีวิว Concurrency
"ตรวจสอบ concurrency patterns ใน internal/service/:
- มี race condition หรือไม่
- ใช้ mutex ถูกต้องหรือไม่
- context cancellation ถูก handle หรือไม่"
```

### Automated Review (GitHub Actions)

ตั้งค่า GitHub Actions (CI/CD) เพื่อให้ Claude รีวิว Pull Request (PR) อัตโนมัติ:

```yaml
name: Claude PR Review
on:
  pull_request:
    types: [opened, synchronize]
jobs:
  review:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: anthropics/claude-code-action@v1
        with:
          model: sonnet
          prompt: |
            Review this PR:
            1. Code quality
            2. Security issues
            3. Performance
            4. Test coverage
            Provide specific, actionable feedback.
          anthropic_api_key: ${{ secrets.ANTHROPIC_API_KEY }}
```

> **NOTE:** เก็บ `ANTHROPIC_API_KEY` ไว้ใน GitHub Secrets เท่านั้น
> ห้ามใส่ API key ลงใน workflow file โดยตรง

### Review Workflow

กระบวนการรีวิวแบ่งเป็น 3 ขั้นตอน:

1. **Pre-review** -- Claude วิเคราะห์ diff (ส่วนต่าง) ของ PR
   - อ่านไฟล์ที่เปลี่ยนแปลงทั้งหมด
   - เข้าใจบริบท (context) ของการเปลี่ยนแปลง
   - ระบุไฟล์ที่เกี่ยวข้องแต่ไม่ได้แก้ไข

2. **Review checklist** -- สร้างรายการตรวจสอบอัตโนมัติ
   - ตรวจสอบตาม coding standards ของโปรเจกต์
   - ตรวจหา common pitfalls ตาม tech stack
   - ตรวจสอบ test coverage สำหรับโค้ดใหม่

3. **Post-review** -- Claude สรุปผลการรีวิว
   - จัดลำดับความสำคัญของ findings
   - แนะนำการแก้ไขที่เจาะจง พร้อมตัวอย่างโค้ด
   - แยกระหว่าง blocker (ต้องแก้) กับ suggestion (ควรแก้)

---

## 7.2 Testing ด้วย Claude Code

Claude Code สามารถช่วยเขียน test ได้หลายระดับ ตั้งแต่ unit test จนถึง integration test
รวมถึงวิเคราะห์ code coverage (ความครอบคลุมของ test)

### Unit Test Generation

```
สร้าง unit tests สำหรับ internal/service/order.go:
1. ใช้ testify suite
2. Mock repository ด้วย mockery
3. ทดสอบ happy path และ error cases
4. Table-driven tests
5. Edge cases: nil input, empty list, max values
```

### ตัวอย่าง Go Test Code

ไฟล์ `internal/service/order_test.go`:

```go
package service_test

import (
    "context"
    "errors"
    "testing"

    "github.com/stretchr/testify/assert"
    "github.com/stretchr/testify/mock"
    "github.com/stretchr/testify/suite"

    "myapp/internal/model"
    "myapp/internal/service"
    mockRepo "myapp/internal/repository/mocks"
)

// OrderServiceTestSuite รวม test cases ทั้งหมดของ OrderService
type OrderServiceTestSuite struct {
    suite.Suite
    mockOrderRepo *mockRepo.MockOrderRepository
    service       service.OrderService
}

func (s *OrderServiceTestSuite) SetupTest() {
    s.mockOrderRepo = mockRepo.NewMockOrderRepository(s.T())
    s.service = service.NewOrderService(s.mockOrderRepo)
}

func TestOrderServiceSuite(t *testing.T) {
    suite.Run(t, new(OrderServiceTestSuite))
}

// --- Table-driven tests สำหรับ CreateOrder ---

func (s *OrderServiceTestSuite) TestCreateOrder() {
    tests := []struct {
        name        string
        input       *model.CreateOrderRequest
        mockSetup   func()
        expected    *model.Order
        expectedErr error
    }{
        {
            name: "สร้าง order สำเร็จ (happy path)",
            input: &model.CreateOrderRequest{
                CustomerID: 1,
                Items: []model.OrderItem{
                    {ProductID: 100, Quantity: 2, Price: 50.00},
                },
            },
            mockSetup: func() {
                s.mockOrderRepo.EXPECT().
                    Create(mock.Anything, mock.AnythingOfType("*model.Order")).
                    Return(nil)
            },
            expected: &model.Order{
                CustomerID: 1,
                TotalAmount: 100.00,
                Status:      "pending",
            },
            expectedErr: nil,
        },
        {
            name:  "input เป็น nil",
            input: nil,
            mockSetup: func() {
                // ไม่เรียก repository เพราะ validate ไม่ผ่าน
            },
            expected:    nil,
            expectedErr: service.ErrInvalidInput,
        },
        {
            name: "items ว่าง (empty list)",
            input: &model.CreateOrderRequest{
                CustomerID: 1,
                Items:      []model.OrderItem{},
            },
            mockSetup:   func() {},
            expected:    nil,
            expectedErr: service.ErrEmptyOrderItems,
        },
        {
            name: "repository error",
            input: &model.CreateOrderRequest{
                CustomerID: 1,
                Items: []model.OrderItem{
                    {ProductID: 100, Quantity: 1, Price: 25.00},
                },
            },
            mockSetup: func() {
                s.mockOrderRepo.EXPECT().
                    Create(mock.Anything, mock.AnythingOfType("*model.Order")).
                    Return(errors.New("database connection lost"))
            },
            expected:    nil,
            expectedErr: service.ErrCreateOrderFailed,
        },
    }

    for _, tt := range tests {
        s.Run(tt.name, func() {
            tt.mockSetup()

            result, err := s.service.CreateOrder(context.Background(), tt.input)

            if tt.expectedErr != nil {
                assert.ErrorIs(s.T(), err, tt.expectedErr)
                assert.Nil(s.T(), result)
            } else {
                assert.NoError(s.T(), err)
                assert.Equal(s.T(), tt.expected.CustomerID, result.CustomerID)
                assert.Equal(s.T(), tt.expected.TotalAmount, result.TotalAmount)
                assert.Equal(s.T(), tt.expected.Status, result.Status)
            }
        })
    }
}
```

> **TIP:** ใช้ table-driven tests เสมอเมื่อทดสอบฟังก์ชันเดียวกันหลาย scenario
> จะทำให้เพิ่ม test case ใหม่ได้ง่าย ไม่ต้องเขียนโค้ดซ้ำ

### Integration Test

```
สร้าง integration test สำหรับ POST /api/v1/orders:
1. Setup test database (testcontainers)
2. Seed test data
3. Send HTTP request
4. Verify response
5. Verify database state
6. Cleanup
```

ตัวอย่าง Integration Test:

```go
package integration_test

import (
    "bytes"
    "context"
    "encoding/json"
    "net/http"
    "net/http/httptest"
    "testing"

    "github.com/stretchr/testify/assert"
    "github.com/stretchr/testify/require"
    "github.com/testcontainers/testcontainers-go"
    "github.com/testcontainers/testcontainers-go/modules/postgres"
)

func TestCreateOrderIntegration(t *testing.T) {
    if testing.Short() {
        t.Skip("skipping integration test in short mode")
    }

    ctx := context.Background()

    // 1. Setup test database
    pgContainer, err := postgres.Run(ctx, "postgres:16-alpine",
        postgres.WithDatabase("testdb"),
        postgres.WithUsername("test"),
        postgres.WithPassword("test"),
    )
    require.NoError(t, err)
    defer pgContainer.Terminate(ctx)

    // 2. Seed test data
    db := setupDB(t, pgContainer)
    seedTestData(t, db)

    // 3. Setup server
    router := setupRouter(db)
    server := httptest.NewServer(router)
    defer server.Close()

    // 4. Send HTTP request
    body := map[string]interface{}{
        "customer_id": 1,
        "items": []map[string]interface{}{
            {"product_id": 100, "quantity": 2, "price": 50.00},
        },
    }
    jsonBody, _ := json.Marshal(body)

    resp, err := http.Post(
        server.URL+"/api/v1/orders",
        "application/json",
        bytes.NewBuffer(jsonBody),
    )
    require.NoError(t, err)
    defer resp.Body.Close()

    // 5. Verify response
    assert.Equal(t, http.StatusCreated, resp.StatusCode)

    var result map[string]interface{}
    json.NewDecoder(resp.Body).Decode(&result)
    assert.Equal(t, float64(100), result["total_amount"])

    // 6. Verify database state
    var count int64
    db.Model(&model.Order{}).Count(&count)
    assert.Equal(t, int64(1), count)
}
```

### Coverage Analysis

ใช้ Claude วิเคราะห์ code coverage อย่างละเอียด:

```bash
# ให้ Claude วิเคราะห์ coverage
claude -p "รัน go test -coverprofile=coverage.out ./... แล้ววิเคราะห์ coverage:
1. Package ไหนต่ำกว่า 80%
2. Function สำคัญที่ยังไม่มี test
3. แนะนำ test cases ที่ควรเพิ่ม"
```

```bash
# ดู coverage แยกตาม package
go test -coverprofile=coverage.out ./...
go tool cover -func=coverage.out
```

```bash
# สร้าง HTML coverage report
go tool cover -html=coverage.out -o coverage.html
```

> **WARNING:** อย่ามุ่งเน้น coverage 100% จนเกินไป
> ให้โฟกัสที่ business logic สำคัญ เช่น service layer และ validation
> มากกว่า boilerplate code เช่น getter/setter หรือ DTO mapping

---

## 7.3 Permission System (ระบบสิทธิ์)

ระบบสิทธิ์ (Permission System) ของ Claude Code ช่วยควบคุมว่า Claude สามารถใช้เครื่องมือ (tool)
ใดได้บ้าง เพื่อความปลอดภัยในการทำงาน

### 5 โหมดการอนุญาต

| โหมด | คำอธิบาย | เหมาะกับ |
|------|----------|---------|
| **Default** | ถามทุกครั้งก่อนใช้ tool | ผู้ใช้ใหม่, งานที่ต้องระวัง |
| **Approve Once** | อนุมัติครั้งเดียวสำหรับ tool นั้น | tool ที่ใช้ไม่บ่อย |
| **Approve for Session** | อนุมัติตลอด session ปัจจุบัน | tool ที่ใช้บ่อยใน session นั้น |
| **Allow All** | อนุมัติ tool ทั้งหมดอัตโนมัติ | งานที่เชื่อถือได้, experienced user |
| **Deny** | ปฏิเสธ tool นั้นเสมอ | tool ที่ไม่ต้องการให้ใช้ |

> **NOTE:** โหมด Allow All ควรใช้เฉพาะเมื่อคุณมั่นใจในคำสั่งที่จะให้ Claude ทำ
> สำหรับงาน production หรืองานที่เกี่ยวกับข้อมูลสำคัญ แนะนำให้ใช้ Default

### settings.json Security

กำหนดสิทธิ์ผ่านไฟล์ `settings.json`:

```json
{
  "permissions": {
    "allow": [
      "Read",
      "Glob",
      "Grep",
      "Edit(**/*.go)",
      "Edit(**/*.ts)",
      "Bash(go test*)",
      "Bash(go build*)",
      "Bash(npm test*)",
      "Bash(npm run*)"
    ],
    "deny": [
      "Bash(rm -rf*)",
      "Bash(git push --force*)",
      "Bash(git reset --hard*)",
      "Bash(curl*)",
      "Bash(wget*)",
      "Write(.env*)",
      "Write(**/credentials*)",
      "Edit(.env*)"
    ]
  }
}
```

**คำอธิบาย Allow Patterns:**

| Pattern | คำอธิบาย |
|---------|----------|
| `Read` | อ่านไฟล์ได้ทั้งหมด -- จำเป็นสำหรับการทำความเข้าใจโค้ด |
| `Glob` | ค้นหาไฟล์ตามชื่อ -- ช่วยนำทางในโปรเจกต์ |
| `Grep` | ค้นหาเนื้อหาในไฟล์ -- จำเป็นสำหรับการวิเคราะห์โค้ด |
| `Edit(**/*.go)` | แก้ไขไฟล์ Go ได้ทั้งหมด -- สิทธิ์หลักในการทำงาน |
| `Edit(**/*.ts)` | แก้ไขไฟล์ TypeScript ได้ทั้งหมด |
| `Bash(go test*)` | รัน Go test ได้ -- ปลอดภัย ไม่เปลี่ยนแปลงระบบ |
| `Bash(go build*)` | build โปรเจกต์ได้ -- ตรวจสอบว่าโค้ด compile ผ่าน |
| `Bash(npm test*)` | รัน JavaScript/TypeScript test ได้ |
| `Bash(npm run*)` | รัน npm scripts ได้ -- ใช้สำหรับ lint, build, etc. |

**คำอธิบาย Deny Patterns:**

| Pattern | เหตุผลที่ปฏิเสธ |
|---------|----------------|
| `Bash(rm -rf*)` | ป้องกันการลบไฟล์โดยไม่ตั้งใจ อาจทำลายข้อมูลสำคัญ |
| `Bash(git push --force*)` | ป้องกันการเขียนทับ commit history ของทีม |
| `Bash(git reset --hard*)` | ป้องกันการสูญเสียการเปลี่ยนแปลงที่ยังไม่ได้ commit |
| `Bash(curl*)` | ป้องกันการดาวน์โหลดหรือส่งข้อมูลไปยัง URL ภายนอก |
| `Bash(wget*)` | เหตุผลเดียวกับ curl |
| `Write(.env*)` | ป้องกันการเขียนทับไฟล์ environment ที่มี secrets |
| `Write(**/credentials*)` | ป้องกันการเข้าถึงไฟล์ credentials |
| `Edit(.env*)` | ป้องกันการแก้ไขไฟล์ environment โดยไม่ตั้งใจ |

### Permission Levels (ลำดับชั้นของสิทธิ์)

Claude Code ใช้ระบบสิทธิ์แบบลำดับชั้น (hierarchical) 3 ระดับ:

1. **Project level:** `.claude/settings.json`
   - ใช้ร่วมกับทีม (shared) -- commit ลง git
   - กำหนดมาตรฐานสิทธิ์ของโปรเจกต์

2. **Local level:** `.claude/settings.local.json`
   - ส่วนตัว (personal) -- อยู่ใน `.gitignore`
   - ปรับแต่งตามความต้องการส่วนบุคคล

3. **User level:** `~/.claude/settings.json`
   - ระดับ global -- ใช้กับทุกโปรเจกต์
   - กำหนดนโยบายพื้นฐานของผู้ใช้

**กฎการรวมสิทธิ์ (Merge Behavior):**

- สิทธิ์จากทุกระดับจะถูกรวมกัน
- **deny ชนะเสมอ** -- ถ้ามี deny ในระดับใดก็ตาม สิทธิ์นั้นจะถูกปฏิเสธ
- allow จะมีผลเฉพาะเมื่อไม่มี deny ที่ขัดกัน

```
ตัวอย่าง:
  Project allow: Edit(**/*.go)
  User deny:     Edit(**/migrations/*.go)

  ผลลัพธ์: แก้ไขไฟล์ .go ได้ทั้งหมด ยกเว้นใน migrations/
```

> **TIP:** ตั้ง deny rules ที่สำคัญที่สุดไว้ที่ User level
> เพื่อให้มีผลกับทุกโปรเจกต์ เช่น deny `Bash(rm -rf /*)` ในระดับ global

---

## 7.4 MCP Server Security

MCP servers (Model Context Protocol) ช่วยเชื่อมต่อ Claude Code กับบริการภายนอก เช่น
database, API, หรือ third-party tools แต่ต้องระวังเรื่องความปลอดภัยเป็นพิเศษ
เพราะ MCP servers มีสิทธิ์เข้าถึง external resources โดยตรง

### แนวทางความปลอดภัยสำหรับ MCP

1. **ใช้เฉพาะ MCP servers ที่เชื่อถือได้**
   - ตรวจสอบ source code ของ MCP server ก่อนใช้งาน
   - ใช้ MCP servers จาก official sources หรือที่ทีมพัฒนาเอง
   - หลีกเลี่ยง MCP servers จากแหล่งที่ไม่รู้จัก

2. **จำกัด permissions ของ MCP server**
   - กำหนดสิทธิ์ให้น้อยที่สุดเท่าที่จำเป็น (Principle of Least Privilege)
   - ใช้ read-only access เมื่อไม่จำเป็นต้องเขียน

3. **ไม่ใส่ credentials ตรงใน `.mcp.json`**

   ไม่ควรทำ:
   ```json
   {
     "mcpServers": {
       "database": {
         "command": "mcp-server-postgres",
         "args": ["postgresql://user:P@ssw0rd@localhost:5432/mydb"]
       }
     }
   }
   ```

   ควรทำ -- ใช้ environment variables:
   ```json
   {
     "mcpServers": {
       "database": {
         "command": "mcp-server-postgres",
         "args": ["$DATABASE_URL"],
         "env": {
           "DATABASE_URL": "${DATABASE_URL}"
         }
       }
     }
   }
   ```

4. **ใส่ `.mcp.json` ใน `.gitignore` ถ้ามี secrets**
   - หากไฟล์ `.mcp.json` มีข้อมูลเฉพาะสภาพแวดล้อม (environment-specific) ควร gitignore
   - สร้าง `.mcp.json.example` เป็น template สำหรับทีม

> **WARNING:** MCP server ที่ถูกโจมตีสามารถอ่าน/เขียนข้อมูลในระบบของคุณได้
> ตรวจสอบ MCP server ทุกตัวที่ใช้งาน และอัปเดตเป็นเวอร์ชันล่าสุดเสมอ

---

## 7.5 Sandbox Mode

Sandbox Mode เป็นโหมดที่จำกัดสิทธิ์การเข้าถึงระบบไฟล์ (filesystem)
ช่วยให้ทดลองคำสั่งต่าง ๆ ได้อย่างปลอดภัย

### การเปิดใช้งาน

```bash
# เปิด Sandbox Mode
/sandbox
```

### สิ่งที่จำกัดใน Sandbox Mode

- จำกัด filesystem access -- เข้าถึงได้เฉพาะ directory ที่กำหนด
- ไม่สามารถเข้าถึง network ภายนอก
- ไม่สามารถรัน commands ที่เปลี่ยนแปลงระบบ

### เหมาะสำหรับ

- **ทดลอง commands ที่ไม่แน่ใจ** -- ลองรันคำสั่งใหม่โดยไม่กลัวทำลายข้อมูล
- **ให้ Claude ลอง approach ใหม่** -- ทดลองวิธีแก้ปัญหาหลายแบบก่อนเลือก
- **Training/learning purposes** -- เรียนรู้ Claude Code อย่างปลอดภัย
- **ทดสอบ scripts ที่ไม่ไว้ใจ** -- รันโค้ดจากภายนอกในพื้นที่ปลอดภัย

---

## 7.6 Security Best Practices

แนวปฏิบัติด้านความปลอดภัยที่ควรยึดถือเมื่อทำงานกับ Claude Code

### ไม่วาง Secrets ในโค้ด

Secrets (ข้อมูลลับ) เช่น API keys, database passwords, tokens ต้องไม่อยู่ในโค้ด:

- **ใช้ environment variables:**
  ```go
  // ไม่ควรทำ
  dbPassword := "SuperSecret123!"

  // ควรทำ
  dbPassword := os.Getenv("DB_PASSWORD")
  ```

- **ใช้ `.env` files (gitignored):**
  ```env
  DB_PASSWORD=SuperSecret123!
  API_KEY=sk-xxxxxxxxxxxx
  JWT_SECRET=my-jwt-secret
  ```

- **ใช้ secret managers สำหรับ production:**
  - AWS Secrets Manager
  - HashiCorp Vault
  - Google Cloud Secret Manager

### ระวัง .env files

- ไม่ให้ Claude อ่าน `.env` ที่มี production secrets
- ตั้ง deny pattern ใน settings.json:

  ```json
  {
    "permissions": {
      "deny": [
        "Write(.env*)",
        "Edit(.env*)"
      ]
    }
  }
  ```

- สร้าง `.env.example` สำหรับทีม (ไม่มีค่าจริง):
  ```env
  DB_PASSWORD=
  API_KEY=
  JWT_SECRET=
  ```

### .gitignore

ตรวจสอบว่าไฟล์ต่อไปนี้อยู่ใน `.gitignore` เสมอ:

```gitignore
# Environment files
.env
.env.local
.env.production
.env.*.local

# Claude Code local settings
.claude/settings.local.json

# MCP configuration (ถ้ามี secrets)
.mcp.json

# IDE and OS files
.idea/
.vscode/
.DS_Store

# Credentials
**/credentials*
**/serviceAccountKey*
*.pem
*.key
```

### OWASP Top 10 Awareness

Claude Code สามารถช่วยตรวจจับช่องโหว่ตาม OWASP Top 10 ได้:

**1. SQL Injection:**
- ใช้ parameterized queries เสมอ
- GORM ทำให้อัตโนมัติเมื่อใช้ methods มาตรฐาน

```go
// เสี่ยง -- SQL Injection
db.Raw("SELECT * FROM users WHERE name = '" + name + "'").Scan(&users)

// ปลอดภัย -- Parameterized query
db.Where("name = ?", name).Find(&users)
```

**2. XSS (Cross-Site Scripting):**
- sanitize output ก่อนแสดงผล
- ใช้ template engine ที่ escape HTML อัตโนมัติ

**3. CSRF (Cross-Site Request Forgery):**
- ใช้ CSRF tokens สำหรับ form submissions
- ตรวจสอบ `Origin` / `Referer` headers

**4. Authentication ที่ไม่ปลอดภัย:**
- ใช้ secure password hashing (bcrypt):
  ```go
  hashedPassword, err := bcrypt.GenerateFromPassword(
      []byte(password), bcrypt.DefaultCost,
  )
  ```

**5. Authorization ที่ไม่เพียงพอ:**
- ใช้ RBAC (Role-Based Access Control) middleware
- ตรวจสอบสิทธิ์ทุก endpoint

**6. Claude Code จะเตือนอัตโนมัติ:**
- เมื่อเห็น hardcoded credentials ในโค้ด
- เมื่อเห็น raw SQL ที่ไม่ใช้ parameterized queries
- เมื่อเห็น insecure configurations

```
# ให้ Claude ตรวจสอบ OWASP vulnerabilities
"ตรวจสอบ security vulnerabilities ตาม OWASP Top 10 ในโปรเจกต์นี้:
1. SQL Injection
2. XSS
3. Broken Authentication
4. Sensitive Data Exposure
5. Security Misconfiguration"
```

---

## 7.7 Performance Review

Claude Code ช่วยวิเคราะห์ปัญหาด้าน performance ที่พบบ่อยในแอปพลิเคชัน

### Query Analysis

```
วิเคราะห์ performance ของ repository/order.go:
1. N+1 query problems
2. Missing indexes
3. Unnecessary data loading
4. Connection pool issues
```

### N+1 Detection

**N+1 Problem คืออะไร?**

N+1 query problem เกิดขึ้นเมื่อโปรแกรมรัน 1 query เพื่อดึงรายการ (list) แล้วรัน
N queries เพิ่มเพื่อดึงข้อมูลที่เกี่ยวข้องสำหรับแต่ละรายการ
ทำให้เกิดจำนวน query มากเกินจำเป็น ส่งผลให้ระบบช้า

**ตัวอย่าง N+1 Problem กับ GORM:**

```go
// N+1 Problem -- 1 query สำหรับ orders + N queries สำหรับ items
var orders []model.Order
db.Find(&orders)  // SELECT * FROM orders (1 query)

for _, order := range orders {
    var items []model.OrderItem
    db.Where("order_id = ?", order.ID).Find(&items)
    // SELECT * FROM order_items WHERE order_id = ? (N queries)
    order.Items = items
}
// รวม: 1 + N queries
```

**แก้ไขด้วย Preload:**

```go
// ใช้ Preload -- 2 queries เท่านั้น ไม่ว่าจะมีกี่ orders
var orders []model.Order
db.Preload("Items").Find(&orders)
// SELECT * FROM orders             (1 query)
// SELECT * FROM order_items WHERE order_id IN (1,2,3,...) (1 query)
// รวม: 2 queries เสมอ
```

**ตัวอย่าง prompt ให้ Claude ตรวจหา N+1:**

```
"ตรวจสอบ N+1 query problems ใน internal/repository/ ทุกไฟล์:
1. หา loop ที่มี database query ข้างใน
2. หา GORM Find/First ที่ไม่มี Preload
3. แนะนำวิธีแก้ไข พร้อมตัวอย่างโค้ด
4. ประเมินผลกระทบต่อ performance"
```

### Performance Optimization Prompts

```
# วิเคราะห์ Memory Usage
"ตรวจสอบ memory usage ใน internal/service/:
1. Large slice allocations ที่ไม่ได้ pre-allocate
2. Goroutine leaks
3. Buffer ที่ไม่ได้ reuse
4. Large struct ที่ pass by value แทน pointer"
```

```
# วิเคราะห์ API Response Time
"วิเคราะห์ endpoint GET /api/v1/orders ว่าช้าตรงไหน:
1. Database queries ที่ไม่จำเป็น
2. Missing caching
3. Serialization overhead
4. Middleware ที่ทำงานหนัก"
```

---

## 7.8 Review Checklist Templates

ใช้ checklist templates เหล่านี้เป็นมาตรฐานในการตรวจสอบ
สามารถให้ Claude ตรวจสอบตาม checklist ได้โดยตรง

### Code Review Checklist

```markdown
## Code Review Checklist
- [ ] Code follows project conventions (naming, structure, formatting)
- [ ] Error handling is complete (no swallowed errors)
- [ ] Input validation is present for all public functions
- [ ] No hardcoded values or secrets
- [ ] Tests are included for new/changed code
- [ ] Documentation updated if public API changed
- [ ] No N+1 queries in database operations
- [ ] Proper logging with appropriate log levels
- [ ] No security vulnerabilities (injection, XSS, etc.)
- [ ] No unnecessary dependencies added
```

### Security Review Checklist

```markdown
## Security Review Checklist
- [ ] No secrets in code (API keys, passwords, tokens)
- [ ] Input sanitization for all user inputs
- [ ] Authentication checks on protected endpoints
- [ ] Authorization checks (role/permission verification)
- [ ] SQL injection prevention (parameterized queries)
- [ ] XSS prevention (output encoding/sanitization)
- [ ] CSRF protection on state-changing endpoints
- [ ] Rate limiting on public-facing endpoints
- [ ] Proper error messages (no stack traces in production)
- [ ] Sensitive data not logged or exposed in responses
- [ ] HTTPS enforced for all external communications
- [ ] Dependencies scanned for known vulnerabilities
```

### PR Review Checklist

```markdown
## PR Review Checklist
- [ ] PR title and description clearly explain the change
- [ ] PR is focused on a single concern (not mixing features)
- [ ] Breaking changes are documented
- [ ] Database migrations are reversible
- [ ] Feature flags used for incomplete features
- [ ] CI/CD pipeline passes (lint, test, build)
- [ ] Code review approved by at least 1 team member
- [ ] No merge conflicts with target branch
- [ ] Related issues/tickets are linked
- [ ] Screenshots included for UI changes (if applicable)
```

### Pre-deployment Checklist

```markdown
## Pre-deployment Checklist
- [ ] All tests pass (unit, integration, e2e)
- [ ] Code coverage meets minimum threshold (>= 80%)
- [ ] No critical/high security vulnerabilities
- [ ] Database migrations tested on staging
- [ ] Environment variables configured in target environment
- [ ] Monitoring and alerting configured
- [ ] Rollback plan documented
- [ ] Load testing completed (if applicable)
- [ ] API documentation updated (Swagger/OpenAPI)
- [ ] Change log updated
- [ ] Team notified of deployment schedule
- [ ] Feature flags configured correctly
```

**ใช้ Checklist กับ Claude Code:**

```
"ตรวจสอบ PR #42 ตาม Code Review Checklist:
- Code follows project conventions
- Error handling is complete
- Input validation is present
- No hardcoded values
- Tests are included
ให้คะแนนแต่ละข้อ (pass/fail) พร้อมเหตุผล"
```

```
"ก่อน deploy version 2.1.0 ตรวจสอบตาม Pre-deployment Checklist:
- ตรวจสอบ test results
- ตรวจสอบ coverage report
- ตรวจสอบ migration files
- ตรวจสอบ environment variables ที่ต้องเพิ่ม"
```

---

## อ้างอิงข้ามบท (Cross-references)

| บท | ชื่อ | ความเกี่ยวข้อง |
|----|------|----------------|
| [01 - แนะนำ Claude Code](./01_intro.md) | บทนำและการติดตั้ง | พื้นฐานก่อนเข้าเรื่อง QA |
| [02 - Agentic Mastery](./02_agentic_mastery.md) | การใช้งาน Agent ขั้นสูง | Plan Mode ที่ใช้ในการรีวิว |
| [03 - Advanced Usage](./03_advanced_usage.md) | การใช้งานขั้นสูง | CLI flags, configuration |
| [04 - Fullstack Collaboration](./04_fullstack_collaboration.md) | การทำงานร่วมกัน | ใช้ร่วมกับ review workflow |
| [05 - Spec-Driven Development](./05_spec_driven_dev.md) | การพัฒนาจาก spec | Spec เป็นพื้นฐานของ test |
| [06 - Workflow Automation](./06_workflow_automation.md) | CI/CD Automation | GitHub Actions สำหรับ review |
| [08 - Troubleshooting](./08_troubleshooting.md) | การแก้ปัญหา | เมื่อ review/test พบ bugs |
| [09 - Appendix](./09_appendix.md) | ภาคผนวก | Prompt templates เพิ่มเติม |
