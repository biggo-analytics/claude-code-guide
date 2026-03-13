# บทที่ 10: Troubleshooting & FAQ

> Last updated: 2026-03-11 | Claude Code compatible version: latest

คู่มือแก้ไขปัญหาที่พบบ่อยในการใช้งาน Claude Code ครอบคลุมตั้งแต่การติดตั้ง การใช้งานประจำวัน
ปัญหาคุณภาพโค้ด ปัญหาในทีม พร้อม FAQ และ Decision Trees สำหรับการตัดสินใจ

---

## สารบัญ

- [10.1 ปัญหาการติดตั้งและเริ่มต้น](#101-ปัญหาการติดตั้งและเริ่มต้น)
- [10.2 ปัญหาขณะใช้งาน](#102-ปัญหาขณะใช้งาน)
- [10.3 ปัญหา Code Quality](#103-ปัญหา-code-quality)
- [10.4 ปัญหาทีมและ Workflow](#104-ปัญหาทีมและ-workflow)
- [10.5 FAQ](#105-faq)
- [10.6 Decision Trees](#106-decision-trees)

---

## 10.1 ปัญหาการติดตั้งและเริ่มต้น

### npm install ไม่ได้ / Authentication ค้าง

**อาการ:**
```
npm ERR! code E401
npm ERR! 401 Unauthorized
```
หรือ `claude` ค้างอยู่ที่หน้า authentication ไม่ยอมไปต่อ

**สาเหตุและวิธีแก้:**

| สาเหตุ | วิธีแก้ |
|--------|---------|
| API key หมดอายุหรือไม่ถูกต้อง | ตรวจสอบด้วย `claude config list` แล้ว set ใหม่ด้วย `claude config set apiKey sk-ant-xxx` |
| Network ถูก block โดย proxy/firewall | ตรวจสอบว่า `api.anthropic.com` ไม่ถูก block, ตั้งค่า proxy ด้วย `export HTTPS_PROXY=http://proxy:port` |
| npm registry ไม่ถูกต้อง | รัน `npm config set registry https://registry.npmjs.org/` |
| Node.js version ต่ำเกินไป | ตรวจสอบด้วย `node --version` ต้อง >= 18, แนะนำใช้ `nvm install 18` |

```bash
# วิธีแก้ไขแบบเร็ว
npm cache clean --force
npm install -g @anthropic-ai/claude-code@latest

# ตรวจสอบ API key
claude config list

# ถ้า authentication ค้าง ลอง login ใหม่
claude auth login
```

### WSL/Windows Setup Issues

**อาการ:** Claude Code ทำงานผิดปกติบน Windows หรือ WSL — path ผิด, ตัวอักษรภาษาไทยเพี้ยน, line endings ไม่ตรง

**วิธีแก้:**

```bash
# 1. ตรวจสอบว่า run ใน WSL ไม่ใช่ PowerShell โดยตรง
wsl --version

# 2. แก้ปัญหา line endings (CRLF vs LF)
git config --global core.autocrlf input

# 3. ตั้งค่า encoding สำหรับ terminal
export LANG=en_US.UTF-8
export LC_ALL=en_US.UTF-8

# 4. หลีกเลี่ยง path แบบ Windows — ใช้ /mnt/c/ แทน C:\
cd /mnt/c/Users/username/projects/my-app
```

**เรื่อง path ที่ต้องระวัง:**
- ใช้ `/mnt/c/...` แทน `C:\...` เสมอเมื่ออยู่ใน WSL
- หลีกเลี่ยงชื่อ folder ที่มีช่องว่าง เช่น `My Project` — ใช้ `my-project` แทน
- ตรวจสอบว่า `CLAUDE.md` เป็น LF ไม่ใช่ CRLF ด้วย `file CLAUDE.md`

### Model ไม่ตอบ / Timeout

**อาการ:** Claude ค้าง ไม่ตอบ หรือแสดง timeout error

**สาเหตุและวิธีแก้:**

| สาเหตุ | วิธีตรวจสอบ | วิธีแก้ |
|--------|-------------|---------|
| Rate limit | ดูข้อความ `429 Too Many Requests` | รอ 1-2 นาทีแล้วลองใหม่ |
| API key หมดอายุ | `claude config list` | สร้าง key ใหม่ที่ console.anthropic.com |
| Network ไม่เสถียร | `curl -I https://api.anthropic.com` | ตรวจสอบ internet, VPN, proxy |
| Request ใหญ่เกินไป | context ยาวมากๆ | ใช้ `/compact` หรือเริ่ม session ใหม่ |

### Permission Denied Errors on First Run

**อาการ:**
```
Error: EACCES: permission denied, mkdir '/usr/local/lib/node_modules'
```

**วิธีแก้:**

```bash
# วิธีที่ 1: ใช้ nvm (แนะนำ)
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.0/install.sh | bash
nvm install 18
nvm use 18
npm install -g @anthropic-ai/claude-code

# วิธีที่ 2: แก้ ownership ของ npm global directory
mkdir -p ~/.npm-global
npm config set prefix '~/.npm-global'
echo 'export PATH=~/.npm-global/bin:$PATH' >> ~/.zshrc
source ~/.zshrc

# อย่าใช้ sudo npm install -g — จะทำให้เกิดปัญหา permission ตามมา
```

### Claude Code Version Mismatch

**อาการ:** ฟีเจอร์ใหม่ใช้ไม่ได้ หรือ command ที่เคยใช้ได้กลับ error

```bash
# ตรวจสอบ version ปัจจุบัน
claude --version

# อัพเดทเป็นล่าสุด
claude update

# ถ้า update ไม่ได้ ลอง reinstall
npm uninstall -g @anthropic-ai/claude-code
npm install -g @anthropic-ai/claude-code@latest

# ตรวจสอบว่าทั้งทีมใช้ version เดียวกัน — ระบุใน CLAUDE.md
# ตัวอย่างใน CLAUDE.md:
# > Claude Code version: >= 1.x.x
```

---

## 10.2 ปัญหาขณะใช้งาน

### Claude สร้าง Code ไม่ตรง Convention

**อาการ:** Claude สร้าง code ที่ใช้ naming convention ผิด, โครงสร้างไฟล์ไม่ตรงกับ project, หรือ style ต่างจาก codebase ที่มีอยู่

**วิธีแก้:**

1. **อัพเดท CLAUDE.md ให้ชัดเจน:**

```markdown
# CLAUDE.md — Project Conventions

## Go Backend
- ใช้ snake_case สำหรับชื่อไฟล์: `order_handler.go`
- ใช้ CamelCase สำหรับ exported functions: `CreateOrder()`
- Error wrapping ใช้ fmt.Errorf("operation: %w", err)
- ทุก handler ต้องอยู่ใน internal/handler/
- ทุก service ต้องอยู่ใน internal/service/
- ใช้ repository pattern สำหรับ database access

## React Frontend
- ใช้ PascalCase สำหรับ component: `OrderList.tsx`
- ใช้ camelCase สำหรับ hooks: `useOrderList.ts`
- State management ใช้ Zustand
- Styling ใช้ Tailwind CSS
```

2. **ให้ Claude อ่านไฟล์ตัวอย่างก่อน:**

```
อ่าน internal/handler/product_handler.go แล้วสร้าง order_handler.go
ตาม pattern เดียวกัน — ทั้งโครงสร้าง error handling และ response format
```

### Context Window เต็ม

**อาการ:** Claude เริ่มลืมสิ่งที่คุยไว้ก่อนหน้า หรือแสดง warning เรื่อง context length

**กลยุทธ์ /compact:**

```
# ใช้ /compact เมื่อ:
- Session ยาวแต่ยังทำ task เดิมอยู่
- Claude เริ่มถามซ้ำหรือลืม context
- ค่าใช้จ่าย token เริ่มสูง (ดูด้วย /cost)

# เริ่ม session ใหม่เมื่อ:
- เปลี่ยน task/feature ใหม่ทั้งหมด
- /compact แล้วยังลืม context สำคัญ
- Session ยาวเกิน 30-40 exchanges
```

| สถานการณ์ | ใช้ /compact | เริ่ม session ใหม่ |
|-----------|-------------|-------------------|
| ทำ feature เดิม ยังไม่เสร็จ | ใช่ | ไม่ |
| เปลี่ยน feature ใหม่ | ไม่ | ใช่ |
| Claude ลืม context หลัง /compact | ไม่ | ใช่ |
| Fix bug ที่เกี่ยวกับ task ปัจจุบัน | ใช่ | ไม่ |
| Review code คนละชุดกัน | ไม่ | ใช่ |

### Session ยาวเกินจน Hallucinate

**สัญญาณที่ต้องสังเกต:**
- Claude อ้างถึงไฟล์หรือ function ที่ไม่มีอยู่จริง
- ตอบ "เสร็จแล้ว" แต่ไม่ได้สร้างไฟล์จริง
- สร้าง import ของ package ที่ไม่มีอยู่ใน project
- ใช้ API signature ที่ผิดของ library ที่มีอยู่

**วิธีรับมือ:**

```bash
# 1. ตรวจสอบว่า Claude ทำจริงหรือไม่
"แสดง git diff ให้ดูว่าแก้อะไรไปบ้าง"

# 2. ถ้าเริ่ม hallucinate — /clear หรือ session ใหม่
/clear

# 3. เริ่ม session ใหม่พร้อม context ที่จำเป็น
claude
"อ่าน CLAUDE.md และ internal/handler/order_handler.go แล้วช่วย fix
  bug ที่ order total คำนวณผิดเมื่อมี discount"
```

**เมื่อไรใช้ /clear vs session ใหม่:**

| | /clear | session ใหม่ |
|--|--------|-------------|
| context ถูก reset | ใช่ | ใช่ |
| working directory เดิม | ใช่ | ใช่ (ถ้า cd ไปที่เดิม) |
| session history ถูกเก็บ | ใช่ (อยู่ใน session เดิม) | ไม่ (เป็น session ใหม่) |
| เหมาะกับ | เปลี่ยน task ย่อยใน feature เดียวกัน | เปลี่ยน feature ทั้งหมด |

### Claude แก้ไฟล์ผิด / ลบ Code ที่ไม่ควรลบ

**วิธีกู้คืน:**

```bash
# 1. ดูว่า Claude แก้อะไรไปบ้าง
git diff

# 2. กู้คืนไฟล์เฉพาะที่ผิด
git restore internal/service/payment_service.go

# 3. กู้คืนทุกไฟล์ (ระวัง — จะ discard ทุกอย่าง)
git restore .

# 4. ใช้ /undo ใน Claude Code (undo การแก้ไขล่าสุดของ Claude)
/undo
```

**ป้องกัน:**
- ใช้ Plan Mode (`Shift+Tab`) ให้ Claude อธิบายก่อนว่าจะแก้อะไร
- Commit บ่อยๆ ก่อนให้ Claude ทำงานใหญ่
- ระบุขอบเขตชัดเจนในคำสั่ง: "แก้เฉพาะ function `CalculateTotal` ใน `order_service.go` ห้ามแก้ไฟล์อื่น"

### Permission ถูก Block

**อาการ:** Claude ขอ permission แล้ว ถูก deny หรือ ไม่สามารถรันคำสั่งได้

**วิธีตรวจและแก้:**

```bash
# ตรวจสอบ settings ปัจจุบัน
claude config list

# แก้ไข project-level settings
# ไฟล์: .claude/settings.json
```

```json
{
  "permissions": {
    "allow": [
      "Read",
      "Edit",
      "Bash(go build:*)",
      "Bash(go test:*)",
      "Bash(npm run:*)",
      "Bash(git:*)"
    ],
    "deny": [
      "Bash(rm -rf:*)",
      "Bash(docker:*)",
      "Bash(sudo:*)"
    ]
  }
}
```

```bash
# แก้ไข user-level settings (เฉพาะเครื่องตัวเอง)
# ไฟล์: ~/.claude/settings.local.json
```

**Tip:** ตั้ง allow list ใน `.claude/settings.json` (commit เข้า repo สำหรับทั้งทีม) และใช้ `settings.local.json` สำหรับ override เฉพาะตัว

### Claude ทำงานช้า

**สาเหตุและวิธีแก้:**

| สาเหตุ | วิธีแก้ |
|--------|---------|
| Context ใหญ่เกินไป | ใช้ `/compact` หรือเริ่ม session ใหม่ |
| ใช้ Opus กับ task ง่ายๆ | เปลี่ยนเป็น Sonnet ด้วย `/model sonnet` |
| ให้ Claude อ่านไฟล์เยอะเกินจำเป็น | ระบุไฟล์ที่ต้องการให้ชัดเจน |
| Network ช้า | ตรวจสอบ internet speed, ลองเปลี่ยน DNS |
| Claude คิด loop (ทำซ้ำไม่จบ) | กด `Esc` แล้วให้ทิศทางใหม่ |

```
# ตัวอย่าง prompt ที่ช่วยให้ Claude ทำงานเร็วขึ้น

# แทนที่: "ดู codebase ทั้งหมดแล้วหา bug"
# ใช้: "ดู internal/service/order_service.go function CalculateTotal
#       มี bug ที่ discount คำนวณผิด — หาและแก้ไข"
```

---

## 10.3 ปัญหา Code Quality

### Code ซ้ำกับที่มีอยู่

**อาการ:** Claude สร้าง utility function ใหม่ทั้งที่มีอยู่แล้วใน codebase

**วิธีแก้:**

```
# Prompt pattern: ชี้ให้ Claude ดูก่อน
"ก่อนสร้าง helper function ใหม่ ค้นหาใน pkg/utils/ และ internal/helpers/
ว่ามี function ที่ทำหน้าที่คล้ายกันอยู่แล้วหรือไม่ ถ้ามีให้ reuse"

# หรือ ให้ดูตัวอย่างแล้วทำตาม pattern เดียวกัน
"ดูตัวอย่าง internal/handler/product_handler.go แล้วสร้าง
order_handler.go ตาม pattern เดียวกัน — ใช้ shared middleware
และ response helper ที่มีอยู่แล้ว"
```

**ระบุใน CLAUDE.md:**

```markdown
## Reuse Policy
- ตรวจสอบ pkg/utils/ ก่อนสร้าง utility function ใหม่
- ใช้ shared response format จาก pkg/response/response.go
- ใช้ shared error types จาก pkg/apperror/errors.go
- ห้ามสร้าง helper ซ้ำกับที่มีอยู่แล้ว
```

### Test ไม่ครอบคลุม

**Prompt patterns สำหรับ edge cases:**

```
"เขียน test สำหรับ CreateOrder() ใน order_service_test.go ครอบคลุม:
1. Happy path — สร้าง order สำเร็จ
2. Empty items — ไม่มีสินค้า
3. Negative quantity — จำนวนติดลบ
4. Zero price — ราคาเป็น 0
5. Exceed stock — สั่งเกินจำนวนคงเหลือ
6. Invalid user ID — user ไม่มีอยู่ในระบบ
7. Database error — จำลอง DB connection error
8. Concurrent orders — 2 orders พร้อมกันสินค้าเหลือ 1 ชิ้น"
```

```
"ตรวจสอบ test coverage ของ internal/service/ แล้วเพิ่ม test ให้ครอบคลุม
boundary cases ที่ยังขาด — โดยเฉพาะ nil input, empty string,
max int value, และ context cancellation"
```

```bash
# ให้ Claude รัน coverage แล้วเพิ่ม test ที่ขาด
"รัน go test -coverprofile=coverage.out ./internal/service/...
แล้ววิเคราะห์ว่า line ไหนยังไม่ถูก cover — เขียน test เพิ่ม"
```

### Security Issues ใน Generated Code

**Review checklist ที่ต้องตรวจทุกครั้ง:**

```
"ตรวจสอบ security ของ code ที่เพิ่งสร้าง ตาม checklist นี้:

1. SQL Injection:
   - ใช้ parameterized query ($1, $2) ไม่ใช่ string concatenation
   - ไม่มีการ fmt.Sprintf ใส่ user input เข้า SQL query

2. XSS (Cross-Site Scripting):
   - React component ไม่ใช้ dangerouslySetInnerHTML
   - User input ถูก sanitize ก่อนแสดงผล

3. Auth Bypass:
   - ทุก endpoint มี middleware ตรวจ authentication
   - Authorization check (role/permission) อยู่ใน handler

4. Sensitive Data:
   - ไม่ log password, token, หรือ PII
   - API response ไม่ส่ง field ที่ไม่จำเป็น (เช่น password hash)

5. Input Validation:
   - ทุก request body ถูก validate ก่อน process
   - File upload ตรวจ size limit และ file type"
```

**ตัวอย่าง code ที่มีปัญหาและวิธีแก้:**

```go
// ผิด — SQL Injection
query := fmt.Sprintf("SELECT * FROM orders WHERE user_id = '%s'", userID)

// ถูก — Parameterized Query
query := "SELECT * FROM orders WHERE user_id = $1"
row := db.QueryRow(query, userID)
```

### Code ที่ Generate มา Compile ไม่ผ่าน

**วิธีแก้:** ให้ Claude รัน build เอง

```
"สร้าง internal/handler/payment_handler.go สำหรับ payment API
แล้วรัน go build ./... เพื่อตรวจสอบว่า compile ผ่าน
ถ้าไม่ผ่าน ให้แก้ไขจนกว่าจะ compile สำเร็จ"
```

```
# สำหรับ Frontend
"สร้าง component PaymentForm.tsx แล้วรัน npm run build
เพื่อตรวจสอบว่าไม่มี TypeScript error — ถ้ามีให้แก้ไข"
```

**Tip:** เพิ่มใน CLAUDE.md ให้ Claude รัน build อัตโนมัติหลังสร้าง code:

```markdown
## Build Verification
- หลังสร้างหรือแก้ไขไฟล์ .go ให้รัน `go build ./...` ทุกครั้ง
- หลังสร้างหรือแก้ไขไฟล์ .tsx/.ts ให้รัน `npm run build` ทุกครั้ง
- หลังแก้ test ให้รัน test ที่เกี่ยวข้องเพื่อยืนยันว่าผ่าน
```

### Naming ไม่สม่ำเสมอ

**วิธี enforce ผ่าน CLAUDE.md:**

```markdown
## Naming Conventions

### Go Backend
| ประเภท | Convention | ตัวอย่าง |
|--------|-----------|---------|
| File name | snake_case | `order_handler.go` |
| Package name | lowercase, ไม่มี underscore | `handler`, `service` |
| Exported function | PascalCase | `CreateOrder()` |
| Unexported function | camelCase | `validateInput()` |
| Struct | PascalCase | `OrderRequest` |
| Interface | PascalCase + er suffix | `OrderRepository` |
| Constant | PascalCase หรือ ALL_CAPS | `MaxRetries`, `DEFAULT_TIMEOUT` |
| Table name (DB) | snake_case, พหูพจน์ | `orders`, `order_items` |

### React Frontend
| ประเภท | Convention | ตัวอย่าง |
|--------|-----------|---------|
| Component file | PascalCase | `OrderList.tsx` |
| Hook file | camelCase, prefix use | `useOrderList.ts` |
| Utility file | camelCase | `formatCurrency.ts` |
| Type/Interface | PascalCase, prefix I (optional) | `OrderResponse` |
| CSS class | kebab-case (Tailwind) | `text-primary` |
```

---

## 10.4 ปัญหาทีมและ Workflow

### CLAUDE.md Conflict ระหว่างสมาชิก

**ปัญหา:** หลายคนแก้ CLAUDE.md พร้อมกัน เกิด merge conflict หรือเนื้อหาขัดแย้งกัน

**Governance Model:**

```
CLAUDE.md Ownership:
├── ## Project Overview        → PM/Tech Lead (owner)
├── ## Architecture            → SA (owner)
├── ## Coding Conventions      → Tech Lead + Senior Dev (owner)
├── ## Testing Standards       → QA Lead (owner)
├── ## Deployment              → DevOps (owner)
└── ## Team-specific Rules     → แต่ละทีม (owner)
```

**กระบวนการแก้ไข:**
1. สร้าง PR สำหรับการแก้ CLAUDE.md (ห้ามแก้ตรงใน main)
2. ต้องได้ review จาก section owner ก่อน merge
3. ใช้ comment ใน CLAUDE.md ระบุว่าใครเป็น owner

```markdown
<!-- Owner: SA Team | Last reviewed: 2026-03-01 -->
## Architecture Decisions
...
```

### DEVLOG ไม่ Sync

**ปัญหา:** สมาชิกบางคนลืมอัพเดท DEVLOG ทำให้ข้อมูลไม่ตรงกัน

**Convention + Hook:**

```bash
# .githooks/pre-push — ตรวจสอบว่า DEVLOG ถูกอัพเดท
#!/bin/bash
CHANGED_FILES=$(git diff --name-only HEAD~1)
HAS_CODE_CHANGE=$(echo "$CHANGED_FILES" | grep -E '\.(go|tsx?|sql)$')
HAS_DEVLOG=$(echo "$CHANGED_FILES" | grep 'DEVLOG.md')

if [ -n "$HAS_CODE_CHANGE" ] && [ -z "$HAS_DEVLOG" ]; then
    echo "WARNING: Code changed but DEVLOG.md not updated."
    echo "Please update DEVLOG.md before pushing."
    exit 1
fi
```

**DEVLOG convention:**

```markdown
## 2026-03-11 — [ชื่อ] — feature/payment-gateway
- เพิ่ม payment handler สำหรับ PromptPay
- แก้ bug discount calculation (issue #142)
- Token used: ~$2.50
- Model: sonnet
```

### Cost Overrun

**ปัญหา:** ค่าใช้จ่าย Claude Code สูงเกินงบ

**Budget Alert + Model Selection Strategy:**

| Task | Model แนะนำ | เหตุผล |
|------|-------------|--------|
| แก้ typo, rename variable | Haiku | task ง่าย ราคาถูก |
| สร้าง CRUD handler, เขียน test | Sonnet | งานทั่วไป สมดุลราคา/คุณภาพ |
| ออกแบบ architecture, refactor ใหญ่ | Opus | ต้องการ reasoning ลึก |
| Debug ปัญหาซับซ้อน | Opus | ต้องวิเคราะห์หลาย layer |
| Generate boilerplate | Haiku/Sonnet | ไม่ต้องการ reasoning มาก |

```bash
# ตรวจสอบค่าใช้จ่ายระหว่าง session
/cost

# เปลี่ยน model ระหว่าง session เพื่อประหยัด
/model haiku    # สำหรับ task ง่ายๆ
/model sonnet   # สำหรับงานทั่วไป
/model opus     # สำหรับงานซับซ้อน
```

**Tip:** ใช้ `/cost` เป็นประจำ และตั้งเป้า budget รายวัน/รายสัปดาห์ต่อคน

### คนในทีมใช้ Claude Code ต่างระดับ

**Training Path:**

```
Level 1: Beginner (สัปดาห์ที่ 1)
├── ติดตั้ง + setup CLAUDE.md
├── ใช้คำสั่งพื้นฐาน (claude, /compact, /clear, /cost)
├── สร้าง code ง่ายๆ จาก prompt
└── เข้าใจ permission system

Level 2: Intermediate (สัปดาห์ที่ 2-3)
├── ใช้ Plan Mode (Shift+Tab)
├── Multi-file editing
├── Pipe mode (claude -p)
├── อ่าน spec แล้วสร้าง code
└── เขียน test ด้วย Claude

Level 3: Advanced (สัปดาห์ที่ 4+)
├── Worktree mode (claude -w)
├── Custom slash commands
├── MCP server integration
├── CI/CD integration (GitHub Actions)
└── Cost optimization strategies
```

**แนวทางสำหรับ team lead:**
- จัด pair session ให้คนที่เก่งกว่าสอนคนที่เริ่มต้น
- สร้าง prompt templates ไว้ใน `examples/07_prompt_catalog.md` ให้ทีมใช้
- Review DEVLOG เพื่อดูว่าใครใช้ Claude Code อย่างไร

### PR ที่ Claude Generate ถูก Reject

**สาเหตุที่พบบ่อย:**

| เหตุผล reject | วิธีป้องกัน |
|---------------|------------|
| ไม่ตรง spec | ให้ Claude อ่าน spec ก่อนเริ่มทำ |
| ไม่มี test | ระบุใน prompt "พร้อม unit test" |
| Naming ผิด convention | อัพเดท CLAUDE.md ให้ชัดเจน |
| ไม่มี error handling | ใช้ prompt "handle ทุก error case" |
| Performance ไม่ดี | ให้ Claude review performance ก่อน submit |

**Review Feedback Loop:**

```
# หลัง PR ถูก reject ให้ feed review comment กลับเข้า Claude
"PR #142 ถูก reject ด้วยเหตุผลนี้:
- ไม่มี pagination สำหรับ list endpoint
- ไม่ validate max page size
- ขาด index บน orders.created_at

ช่วยแก้ไขตาม feedback แล้วอัพเดท code"
```

---

## 10.5 FAQ

### Setup

**Q: Claude Code ต้องใช้ internet ตลอดเวลาไหม?**
A: ใช่ Claude Code ต้องเชื่อมต่อ internet เพื่อส่ง request ไปยัง Anthropic API ไม่สามารถใช้งาน offline ได้ แต่ตัว code ที่ generate มาจะอยู่ใน local เครื่องคุณ

**Q: ใช้ Claude Code กับ private repo ได้ไหม?**
A: ได้ Claude Code ทำงานใน local terminal ของคุณ อ่านไฟล์จาก filesystem โดยตรง ไม่ต้อง clone ผ่าน Anthropic — ดังนั้น private repo ใช้ได้ปกติ เพียงแค่ `cd` เข้าไปใน repo แล้วรัน `claude`

**Q: ต้องมี API key คนละตัวไหม ในทีม?**
A: แนะนำให้แต่ละคนมี API key ของตัวเอง เพื่อติดตาม usage และค่าใช้จ่ายแยกรายคน ถ้าใช้ key เดียวกันจะไม่สามารถแยก cost ได้ และอาจชน rate limit พร้อมกัน สำหรับองค์กร สามารถใช้ Anthropic Admin Console จัดการ key ของทั้งทีมได้

**Q: Claude Code รองรับ Windows native ไหม?**
A: Claude Code ทำงานบน macOS, Linux, และ WSL (Windows Subsystem for Linux) ไม่แนะนำให้ใช้บน PowerShell หรือ CMD โดยตรง — ใช้ WSL แทนจะเสถียรกว่ามาก

### Usage

**Q: ทำยังไงให้ Claude จำ context ข้ามวัน?**
A: ใช้ `/resume` เพื่อกลับเข้า session เก่า หรือเขียนสรุปสิ่งที่ทำไว้ใน DEVLOG.md แล้วให้ Claude อ่านตอนเริ่ม session ใหม่ ถ้าใช้ `claude -c` จะต่อ session ล่าสุดได้เลย

**Q: Claude Code กับ Cursor/Copilot ต่างกันยังไง?**
A: Cursor เป็น AI-native IDE เน้น inline editing, Copilot เน้น autocomplete ใน IDE ส่วน Claude Code เป็น CLI agent ที่สามารถอ่าน/แก้/สร้างไฟล์, รันคำสั่ง, ข้ามโปรเจกต์ได้ ทั้งสามใช้ร่วมกันได้ — ใช้ Copilot สำหรับ autocomplete และ Claude Code สำหรับ task ที่ใหญ่กว่า

**Q: Claude Code อ่าน .env ได้ไหม? ปลอดภัยไหม?**
A: Claude Code สามารถอ่านไฟล์ .env ได้ถ้ามี permission แต่จะไม่ส่ง content ของ .env ไปที่อื่นนอกจาก Anthropic API เพื่อ process ถ้ากังวลเรื่องความปลอดภัย ให้ตั้ง deny pattern ใน settings.json: `"deny": ["Read(.env)"]` หรือเพิ่มใน CLAUDE.md ว่า "ห้ามอ่านไฟล์ .env"

**Q: ใช้ Claude Code review code ของคนอื่นได้ไหม?**
A: ได้ และเป็น use case ที่ดีมาก ใช้ Plan Mode (`Shift+Tab`) แล้ว prompt: "Review code changes ใน PR นี้ ดู git diff main...feature-branch" Claude จะวิเคราะห์และให้ feedback โดยไม่แก้ไข code

**Q: /compact กับ /clear ต่างกันยังไง?**
A: `/compact` บีบอัด context เก่าให้เล็กลง ยังคงจำสรุปสิ่งที่คุยไว้ได้ ส่วน `/clear` ลบ context ทั้งหมด เริ่มใหม่จากศูนย์ ใช้ `/compact` เมื่อยังทำงานเดิม ใช้ `/clear` เมื่อเปลี่ยนงานใหม่

**Q: จะรู้ได้ยังไงว่า Claude กำลัง hallucinate?**
A: ตรวจสอบด้วยการให้ Claude รัน build/test ทุกครั้ง ถ้า Claude อ้างถึงไฟล์ที่ไม่มี สร้าง import ที่ผิด หรือใช้ API ที่ไม่ตรงกับ docs — นั่นคือสัญญาณ hallucination ให้ `git diff` ตรวจสอบและ /clear เริ่มใหม่

### Team

**Q: CLAUDE.md ควรมีกี่ไฟล์ในโปรเจกต์?**
A: มี CLAUDE.md หลัก 1 ไฟล์ที่ root แต่สามารถมี CLAUDE.md ย่อยใน subdirectory ได้ เช่น `frontend/CLAUDE.md` สำหรับ rules เฉพาะ frontend Claude จะอ่านทุก CLAUDE.md ที่อยู่ใน path ที่ทำงาน

**Q: ถ้า Claude generate code ผิด ใครรับผิดชอบ?**
A: Developer ที่ใช้ Claude Code รับผิดชอบ Claude เป็นเครื่องมือ ไม่ใช่ developer — ต้อง review code ที่ Claude สร้างเสมอก่อน commit เปรียบเสมือนคุณใช้ Stack Overflow ได้ แต่ต้อง review ก่อน copy-paste

**Q: ควรใส่ CLAUDE.md เข้า git ไหม?**
A: ใช่ CLAUDE.md ควร commit เข้า repo เพื่อให้ทุกคนในทีมใช้ rules เดียวกัน ยกเว้น settings ส่วนตัวที่ควรอยู่ใน `~/.claude/settings.local.json` ไม่ต้อง commit

**Q: ทำยังไงให้ทีมใช้ Claude Code เป็นมาตรฐานเดียวกัน?**
A: 1) สร้าง CLAUDE.md ที่ครบถ้วน 2) ตั้ง `.claude/settings.json` สำหรับ permission ระดับ project 3) สร้าง prompt templates 4) จัด training ตาม level 5) review DEVLOG เป็นประจำ

### Cost

**Q: Claude Code คิดค่าใช้จ่ายยังไง?**
A: คิดตาม token ที่ใช้ (input + output) คูณด้วยราคาของ model ที่เลือก Opus แพงสุด, Sonnet กลาง, Haiku ถูกสุด ใช้ `/cost` เพื่อดูค่าใช้จ่ายระหว่าง session

**Q: วิธีประหยัดค่าใช้จ่ายที่ได้ผลจริง?**
A: 1) ใช้ model ให้เหมาะกับ task (Haiku สำหรับงานง่าย) 2) `/compact` เมื่อ context ยาว 3) ระบุ context ให้แคบ (ชี้ไฟล์เฉพาะ ไม่ใช่ "ดูทั้ง codebase") 4) ใช้ pipe mode สำหรับ task ที่ชัดเจน 5) เริ่ม session ใหม่เมื่อเปลี่ยนงาน

**Q: มีวิธี set budget limit ไหม?**
A: สามารถตั้ง budget ใน Anthropic Console ได้ระดับ organization/team นอกจากนี้ใช้ `/cost` ตรวจสอบเป็นประจำ และตั้ง convention ในทีมว่าถ้า session เกิน $X ให้หยุดและประเมินใหม่

### Security

**Q: ข้อมูลที่ส่งให้ Claude ปลอดภัยไหม?**
A: Code ที่ส่งไป process ผ่าน Anthropic API มี encryption in transit (TLS) Anthropic ไม่ใช้ข้อมูลจาก API เพื่อ train model (ตาม commercial terms) แต่ควรหลีกเลี่ยงส่ง secrets, passwords, หรือ production credentials ไปใน prompt โดยตรง

**Q: Claude Code สามารถรัน command อันตรายได้ไหม?**
A: ได้ ถ้ามี permission Claude Code มีระบบ permission ที่ถามก่อนรัน command ที่มีความเสี่ยง ตั้ง deny list ใน settings.json เพื่อ block คำสั่งอันตราย เช่น `rm -rf`, `sudo`, `docker` ตามที่ต้องการ

**Q: ควร gitignore อะไรบ้างที่เกี่ยวกับ Claude Code?**
A: ไม่ต้อง gitignore CLAUDE.md และ `.claude/settings.json` (ควร commit เข้า repo) แต่ควร gitignore `settings.local.json` (personal settings) ส่วนไฟล์ `.env`, credentials, และ API keys ต้อง gitignore เหมือนปกติ

---

## 10.6 Decision Trees

### Decision Tree 1: "Claude ให้ผลลัพธ์ไม่ดี"

```
Claude ให้ผลลัพธ์ไม่ดี
│
├─ CLAUDE.md มี convention ชัดเจนไหม?
│  ├─ ไม่ → อัพเดท CLAUDE.md ให้ครบ แล้วลองใหม่
│  └─ มี → ไปข้อถัดไป
│
├─ Claude ได้อ่าน context ที่จำเป็นไหม?
│  ├─ ไม่ → ชี้ไฟล์ให้ Claude อ่านก่อน เช่น
│  │        "อ่าน internal/handler/product_handler.go ก่อน"
│  └─ ได้อ่านแล้ว → ไปข้อถัดไป
│
├─ Prompt ละเอียดพอไหม?
│  ├─ ไม่ → เพิ่มรายละเอียด:
│  │        - ระบุ input/output ที่ต้องการ
│  │        - ระบุ constraints
│  │        - ให้ตัวอย่าง
│  └─ ละเอียดแล้ว → ไปข้อถัดไป
│
├─ Model เหมาะกับ task ไหม?
│  ├─ Task ซับซ้อนแต่ใช้ Haiku → เปลี่ยนเป็น Sonnet/Opus
│  └─ Model เหมาะแล้ว → ไปข้อถัดไป
│
├─ Context เต็มหรือ session ยาวเกินไหม?
│  ├─ ใช่ → /compact หรือเริ่ม session ใหม่
│  └─ ไม่ → ไปข้อถัดไป
│
└─ ลองใช้ Plan Mode (Shift+Tab)
   ├─ ให้ Claude วางแผนก่อนทำ
   └─ Review แผนก่อน approve
```

### Decision Tree 2: "ควรใช้ Model ไหน?"

```
ควรใช้ Model ไหน?
│
├─ Task ง่าย / boilerplate / rename / format?
│  └─ ใช้ Haiku ✓
│     ราคาถูก, เร็ว, เพียงพอสำหรับงานง่าย
│
├─ Task ทั่วไป? (CRUD, handler, test, component)
│  └─ ใช้ Sonnet ✓
│     สมดุลราคา/คุณภาพ เหมาะกับงาน dev ส่วนใหญ่
│
├─ Task ซับซ้อน?
│  ├─ ออกแบบ architecture → ใช้ Opus ✓
│  ├─ Refactor ข้ามหลายไฟล์ → ใช้ Opus ✓
│  ├─ Debug ปัญหาที่ซับซ้อน → ใช้ Opus ✓
│  ├─ Performance optimization → ใช้ Opus ✓
│  └─ Security review → ใช้ Opus ✓
│
└─ ไม่แน่ใจ?
   └─ เริ่มด้วย Sonnet
      ถ้าผลลัพธ์ไม่ดีค่อยเปลี่ยนเป็น Opus
      (ใช้ /model เปลี่ยนระหว่าง session ได้)
```

### Decision Tree 3: "Session เก่าหรือเริ่มใหม่?"

```
Session เก่าหรือเริ่มใหม่?
│
├─ ทำ task/feature เดิมกับ session ก่อนหน้าไหม?
│  ├─ ใช่ → ต่อ session เดิม
│  │        ใช้ claude -c (session ล่าสุด)
│  │        หรือ claude --resume (เลือก session)
│  │
│  │  └─ Context เต็มไหม? (Claude เริ่มลืม/ช้า)
│  │     ├─ ยัง → ทำต่อได้เลย ✓
│  │     └─ เต็มแล้ว → /compact
│  │        └─ หลัง /compact ยังลืมไหม?
│  │           ├─ ไม่ → ทำต่อได้ ✓
│  │           └─ ใช่ → เริ่ม session ใหม่
│  │                    พร้อมสรุป context ที่จำเป็น
│  │
│  └─ ไม่ → ทำ task/feature ใหม่
│     └─ เริ่ม session ใหม่ ✓
│        "อ่าน CLAUDE.md แล้ว [task ใหม่]"
│
└─ ไม่แน่ใจ?
   └─ เริ่ม session ใหม่ (ปลอดภัยกว่า)
      ถ้าต้องการ context เดิม สรุปใส่ DEVLOG
      แล้วให้ Claude อ่าน DEVLOG ตอนเริ่ม session
```

---

## การอ้างอิงข้ามบท

| บท | หัวข้อ | ความเกี่ยวข้อง |
|----|--------|----------------|
| [บทที่ 1](01_intro.md) | บทนำและการติดตั้ง | การติดตั้ง, setup, system requirements |
| [บทที่ 2](02_agentic_mastery.md) | Agentic Mastery | Plan Mode, prompt techniques |
| [บทที่ 3](03_advanced_usage.md) | การใช้งานขั้นสูง | CLI flags, worktree, pipe mode |
| [บทที่ 4](04_fullstack_collaboration.md) | Full-Stack Collaboration | การทำงานร่วม FE/BE |
| [บทที่ 5](05_spec_driven_dev.md) | Spec-Driven Development | การพัฒนาจาก spec |
| [บทที่ 6](06_architecture_design.md) | Architecture & Design | สถาปัตยกรรมและการออกแบบ |
| [บทที่ 7](07_qa_security_review.md) | QA, Security, Review | Security checklist, testing |
| [บทที่ 8](08_roles_responsibilities.md) | Roles & Responsibilities | บทบาทในทีม |
| [บทที่ 9](09_cheat_sheet.md) | Cheat Sheet | คำสั่งอ้างอิงเร็ว |

---

> **สรุป**: บทนี้รวบรวมปัญหาที่พบบ่อยและวิธีแก้ไขทั้งหมดไว้ในที่เดียว
> ใช้ Decision Trees เป็นแนวทางตัดสินใจเมื่อไม่แน่ใจ
> และ FAQ สำหรับคำถามที่ทีมมักจะถาม
> สำหรับรายละเอียดเชิงลึก ให้อ้างอิงจากบทที่เกี่ยวข้องในตารางข้างต้น

---

← [บทที่ 9: สรุปรวมและ Cheat Sheet](09_cheat_sheet.md) | [บทที่ 11: Cost Optimization →](11_cost_optimization.md)
