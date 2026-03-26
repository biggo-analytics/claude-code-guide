# Use Case: แปลง Web URL / Screenshot เป็น Figma Design ด้วย Figma MCP
## Web URL & Screenshot to Figma Design via MCP

> Last updated: 2026-03-27 | Claude Code compatible version: latest
>
> วิธีใช้ Claude + Figma MCP เพื่อแปลงหน้าเว็บหรือ screenshot ของ UI
> ให้กลายเป็น Figma design ที่พร้อมใช้งาน

---

## สารบัญ

- [1. ภาพรวม (Overview)](#1-ภาพรวม-overview)
- [2. สิ่งที่ต้องเตรียม (Prerequisites)](#2-สิ่งที่ต้องเตรียม-prerequisites)
- [3. Figma MCP Tools ที่ใช้](#3-figma-mcp-tools-ที่ใช้)
- [4. Workflow A: Web URL → Figma Design](#4-workflow-a-web-url--figma-design)
- [5. Workflow B: Screenshot → Figma Design](#5-workflow-b-screenshot--figma-design)
- [6. เทคนิคขั้นสูง (Advanced Techniques)](#6-เทคนิคขั้นสูง-advanced-techniques)
- [7. ข้อจำกัดและข้อควรระวัง (Limitations)](#7-ข้อจำกัดและข้อควรระวัง-limitations)
- [Cross-references](#cross-references)

---

## 1. ภาพรวม (Overview)

ในหลายสถานการณ์ทีมต้องการแปลงหน้าเว็บที่มีอยู่แล้วหรือ screenshot ของ UI ให้เป็น
Figma design ที่แก้ไขได้ เช่น:

- **Redesign** — นำเว็บเดิมเข้า Figma เพื่อปรับปรุง UI/UX
- **Competitor analysis** — จับ UI ของคู่แข่งมาวิเคราะห์ใน Figma
- **Legacy migration** — มีแค่ screenshot ของระบบเก่า ต้องการสร้าง design ใหม่
- **Rapid prototyping** — แปลง wireframe/mockup จากรูปภาพเป็น Figma components

### Flow

```
┌──────────────────┐     ┌──────────────────┐     ┌──────────────────┐
│   Input          │     │   Claude          │     │   Output         │
│                  │     │                  │     │                  │
│  - Web URL       │────▶│  1. วิเคราะห์ UI  │────▶│  Figma Design    │
│  - Screenshot    │     │  2. แยก components│     │  - Frames        │
│                  │     │  3. เรียก Figma   │     │  - Auto Layout   │
│                  │     │     MCP tools     │     │  - Colors/Fonts  │
└──────────────────┘     └──────────────────┘     └──────────────────┘
```

### ใช้ได้กับทั้ง 2 แพลตฟอร์ม

| แพลตฟอร์ม | Figma MCP | วิธี Input |
|-----------|-----------|-----------|
| **claude.ai** (Web UI) | Built-in (เปิดใช้ใน Settings → Integrations) | วาง URL ใน chat / ลาก screenshot เข้า chat |
| **Claude Code** (CLI) | ตั้งค่าผ่าน `.mcp.json` | ระบุ URL ใน prompt / ระบุ path ของไฟล์รูป |

---

## 2. สิ่งที่ต้องเตรียม (Prerequisites)

### 2.1 Figma Account

- Figma account ที่มีสิทธิ์สร้างและแก้ไขไฟล์ (Professional หรือ Organization plan)
- Figma Personal Access Token (สำหรับ Claude Code CLI)
  - ไปที่ Figma → Settings → Personal Access Tokens → Generate new token

### 2.2 Claude

**สำหรับ claude.ai (Web UI):**
- Claude Pro หรือ Team subscription
- เปิด Figma integration: Settings → Integrations → Figma → Connect
- ไม่ต้องตั้งค่าเพิ่มเติม — Figma MCP เป็น built-in

**สำหรับ Claude Code (CLI):**
- ตั้งค่า Figma MCP server ในไฟล์ `.mcp.json` ที่ root ของโปรเจกต์:

```json
{
  "mcpServers": {
    "figma": {
      "command": "npx",
      "args": ["-y", "@anthropic/figma-mcp-server"],
      "env": {
        "FIGMA_ACCESS_TOKEN": "${FIGMA_ACCESS_TOKEN}"
      }
    }
  }
}
```

> 💡 เก็บ token ใน `.env` file ที่อยู่ใน `.gitignore` แทนการ hardcode

### 2.3 ตรวจสอบการเชื่อมต่อ

**claude.ai:** พิมพ์ในแชท:
```
ตรวจสอบว่า Figma MCP เชื่อมต่อแล้ว ดูข้อมูล account ของฉัน
```
Claude จะเรียก `whoami` tool เพื่อแสดงข้อมูล account

**Claude Code CLI:**
```
/mcp
```
ดูว่า figma server มีสถานะ `connected` และแสดง tools ที่พร้อมใช้

---

## 3. Figma MCP Tools ที่ใช้

| Tool | หน้าที่ | ใช้ใน Workflow |
|------|---------|---------------|
| `whoami` | ตรวจสอบ account และ planKey | ทั้งคู่ |
| `create_new_file` | สร้างไฟล์ Figma ใหม่ | ทั้งคู่ |
| `use_figma` | รัน JavaScript ผ่าน Figma Plugin API เพื่อสร้าง/แก้ไข design | ทั้งคู่ |
| `upload_asset_from_url` | อัปโหลดรูปภาพจาก URL เข้า Figma | Workflow A |
| `get_screenshot` | ถ่ายภาพ design ที่สร้างเพื่อตรวจสอบ | ทั้งคู่ |
| `get_design_context` | อ่าน design context (code + hints) | ทั้งคู่ |
| `search_design_system` | ค้นหา components จาก design system | ขั้นสูง |
| `start_editing_transaction` | เริ่ม editing session | ขั้นสูง |
| `perform_editing_operations` | ดำเนินการแก้ไข design | ขั้นสูง |
| `commit_editing_transaction` | บันทึกการแก้ไข | ขั้นสูง |

> ⚠️ **สำคัญ:** Figma MCP ไม่มี tool สำหรับ "import" เว็บไซต์โดยตรง
> Claude จะ **วิเคราะห์ UI แล้วสร้าง design ขึ้นใหม่** ผ่าน `use_figma` (Figma Plugin API)
> ผลลัพธ์จึงเป็น vector/frame-based design ไม่ใช่ pixel-perfect clone

---

## 4. Workflow A: Web URL → Figma Design

### Step 4.1: สร้างไฟล์ Figma ใหม่

**Prompt:**
```
สร้างไฟล์ Figma ใหม่ชื่อ "Landing Page Redesign - example.com"
```

Claude จะเรียก `whoami` เพื่อดู planKey แล้วเรียก `create_new_file` เพื่อสร้างไฟล์

**สิ่งที่ได้:** ลิงก์ไปยังไฟล์ Figma ใหม่ที่พร้อมใช้งาน

### Step 4.2: วิเคราะห์หน้าเว็บ

**Prompt:**
```
ดูหน้าเว็บ https://example.com แล้ววิเคราะห์ UI ทั้งหมด:
- Layout structure (header, hero, content sections, footer)
- Color palette (hex codes)
- Typography (font family, sizes, weights)
- Spacing patterns
- Components ที่ใช้ซ้ำ (buttons, cards, navigation)
- รูปภาพและ assets หลัก ๆ
```

Claude จะ fetch หน้าเว็บ วิเคราะห์ HTML/CSS แล้วสรุป UI components ทั้งหมด
ออกมาเป็นรายการที่ชัดเจน

### Step 4.3: สร้าง Design ใน Figma

**Prompt:**
```
จาก UI ที่วิเคราะห์ไว้ สร้าง design ในไฟล์ Figma ที่เพิ่งสร้าง:

1. Desktop frame (1440×900)
2. วาง layout ตาม structure ของเว็บต้นฉบับ
3. ใช้สีและ typography ที่ตรงกับเว็บ
4. สร้าง Auto Layout สำหรับ sections หลัก
5. ใช้ชื่อ layer ที่สื่อความหมาย เช่น "Header", "Hero Section", "CTA Button"
```

Claude จะเรียก `use_figma` พร้อม JavaScript code ที่ใช้ Figma Plugin API
เพื่อสร้าง frames, rectangles, text nodes, auto layouts ตามที่วิเคราะห์ไว้

**ตัวอย่างสิ่งที่ Claude สร้างได้:**
- Frame หลักขนาด 1440×900
- Header bar พร้อม navigation links
- Hero section พร้อม heading, subtext, CTA button
- Content sections พร้อม cards (Auto Layout)
- Footer พร้อมลิงก์

### Step 4.4: อัปโหลดรูปภาพ (ถ้ามี)

**Prompt:**
```
อัปโหลดรูปภาพจากเว็บต้นฉบับเข้า Figma:
- Hero image: https://example.com/images/hero.jpg
- Logo: https://example.com/images/logo.png
แล้ววางในตำแหน่งที่ถูกต้องใน design
```

Claude จะเรียก `upload_asset_from_url` เพื่อนำรูปเข้า Figma
แล้วใช้ `use_figma` เพื่อวางในตำแหน่งที่ถูกต้อง

### Step 4.5: Review และปรับแก้

**Prompt:**
```
ถ่ายภาพ design ที่สร้างให้ดูหน่อย
```

Claude จะเรียก `get_screenshot` แล้วแสดงผลให้ตรวจสอบ

**Prompt ปรับแก้:**
```
ปรับแก้ design:
1. เพิ่ม padding ของ Hero section เป็น 80px บน-ล่าง
2. เปลี่ยนสี CTA button เป็น #FF6B35
3. เพิ่ม drop shadow ให้ cards
4. ปรับ font size ของ heading เป็น 48px
```

วนซ้ำ screenshot → ปรับแก้ จนพอใจ

### Step 4.6: เพิ่ม Responsive Variants (Optional)

**Prompt:**
```
สร้าง responsive variants เพิ่มเติม:
1. Tablet (768×1024) — ปรับ layout เป็น 2 columns
2. Mobile (390×844) — ปรับเป็น single column, hamburger menu
วางทุก variants ในหน้าเดียวกัน เรียงจากซ้ายไปขวา
```

---

## 5. Workflow B: Screenshot → Figma Design

เหมาะสำหรับกรณีที่มีแค่รูปภาพ UI เช่น screenshot จาก mobile app,
wireframe ที่วาดมือ, หรือ UI จากระบบที่ไม่สามารถเข้าถึง URL ได้

### Step 5.1: อัปโหลด Screenshot และวิเคราะห์

**claude.ai:** ลากรูปภาพเข้า chat แล้วพิมพ์:
```
วิเคราะห์ UI จาก screenshot นี้:
- แยก components ทั้งหมดที่เห็น
- ระบุ layout structure
- ประมาณสี (hex), font sizes, spacing
- แยก sections ของหน้าจอ
- ระบุ interactive elements (buttons, inputs, links)
```

**Claude Code CLI:** ระบุ path ของไฟล์:
```
ดูรูป /path/to/screenshot.png แล้ววิเคราะห์ UI components ทั้งหมด
ระบุ layout, สี, typography, spacing โดยประมาณ
```

### Step 5.2: สร้าง Figma Design

**Prompt:**
```
สร้างไฟล์ Figma ใหม่ชื่อ "App Redesign" แล้วสร้าง design ตาม screenshot ที่วิเคราะห์:
- ใช้ frame ขนาด 390×844 (iPhone 14 Pro)
- สร้าง Auto Layout ที่เหมาะสม
- ใช้สีและ spacing ตามที่วิเคราะห์ได้
- ตั้งชื่อ layers ให้สื่อความหมาย
```

### Step 5.3: Iterate และ Polish

**Prompt ตรวจสอบ:**
```
ถ่ายภาพ design ที่สร้าง แล้วเปรียบเทียบกับ screenshot ต้นฉบับ
บอกจุดที่แตกต่างและแก้ไขให้ใกล้เคียงมากขึ้น
```

**Prompt ปรับ:**
```
ปรับ design ให้:
1. Navigation bar ชิดขอบบนสุด
2. เพิ่ม bottom tab bar ตามที่เห็นใน screenshot
3. ปรับ corner radius ของ cards เป็น 12px
4. เพิ่ม status bar area (44px) ด้านบน
```

---

## 6. เทคนิคขั้นสูง (Advanced Techniques)

### 6.1 ใช้ Design System ที่มีอยู่

ถ้าทีมมี Design System ใน Figma อยู่แล้ว สามารถให้ Claude ใช้ components ที่มีแทนการสร้างใหม่:

**Prompt:**
```
ค้นหา design system ของทีม แล้วดูว่ามี components อะไรบ้าง
ใช้ components จาก design system (เช่น Button, Card, Input)
แทนการสร้าง elements ใหม่ตั้งแต่ต้น
```

Claude จะเรียก `search_design_system` เพื่อค้นหา components ที่มีอยู่

### 6.2 Batch หลายหน้า

**Prompt:**
```
แปลง 3 หน้าจากเว็บ example.com เป็น Figma design:
1. Homepage (https://example.com)
2. Product page (https://example.com/products/1)
3. Cart page (https://example.com/cart)

สร้างแต่ละหน้าเป็น page แยกในไฟล์ Figma เดียวกัน
```

### 6.3 นำ Style จาก Figma Design เดิมมาใช้

**Prompt:**
```
ดู design ใน Figma ไฟล์นี้: https://figma.com/design/ABC123/MyDesign
แล้วนำ color palette และ typography ไปใช้กับ design ใหม่ที่จะสร้างจาก screenshot นี้
```

Claude จะเรียก `get_design_context` เพื่ออ่าน style จาก design เดิม
แล้วนำไป apply กับ design ใหม่

### 6.4 สร้าง Component Library

**Prompt:**
```
จาก design ที่สร้างไว้ แยก elements ที่ใช้ซ้ำเป็น Figma components:
- Button (Primary, Secondary, Ghost variants)
- Card (Product card, Info card)
- Input field (Text, Search, Dropdown)
- Navigation bar
จัดวางใน page แยกชื่อ "Components"
```

---

## 7. ข้อจำกัดและข้อควรระวัง (Limitations)

### ข้อจำกัดทางเทคนิค

| ข้อจำกัด | คำอธิบาย |
|----------|----------|
| ไม่ใช่ pixel-perfect | ผลลัพธ์เป็น vector/frame-based design ที่ใกล้เคียง ไม่ใช่สำเนา 1:1 |
| รูปภาพต้องอัปโหลดแยก | ใช้ `upload_asset_from_url` สำหรับรูปแต่ละรูป |
| SVG/Icon ซับซ้อน | Custom SVG paths อาจไม่สามารถสร้างซ้ำได้ทั้งหมด |
| Animation | ไม่สามารถจับ CSS animations หรือ transitions ได้ |
| Dynamic content | Claude เห็นแค่ static snapshot ของหน้าเว็บ ณ เวลาที่ fetch |
| ขนาด code | JavaScript ที่ส่งผ่าน `use_figma` มีขีดจำกัดขนาด หน้าที่ซับซ้อนมากอาจต้องแบ่งทำทีละส่วน |

### ข้อควรระวัง

> ⚠️ **ความเป็นส่วนตัว:** อย่าแปลง UI ที่มีข้อมูลลับ (internal dashboards, admin panels ที่มี sensitive data)
> ผ่าน cloud service โดยไม่ได้รับอนุญาต เพราะข้อมูลจะถูกส่งผ่าน Claude API

> ⚠️ **ลิขสิทธิ์:** การ clone UI ของเว็บอื่นเพื่อใช้ใน production อาจมีปัญหาเรื่องลิขสิทธิ์
> ใช้เพื่อ reference, learning, หรือ internal analysis เท่านั้น

> 💡 **เคล็ดลับ:** สำหรับหน้าเว็บที่ซับซ้อนมาก แนะนำให้แบ่งทำทีละ section
> เช่น สร้าง Header ก่อน → ตรวจสอบ → สร้าง Hero section → ตรวจสอบ → ต่อไปเรื่อย ๆ
> จะได้ผลลัพธ์ที่ละเอียดกว่าการสั่งทำทั้งหน้าในครั้งเดียว

> 💡 **Rate Limits:** หากใช้ Figma MCP tools หลายครั้งติดต่อกันเร็วมาก
> อาจเจอ rate limit ของ Figma API — ให้รอสักครู่แล้วลองใหม่

---

## Cross-references

📌 **การตั้งค่า MCP เบื้องต้น** → [บทที่ 3 หัวข้อ 3.6](../03_advanced_usage.md#36-mcp-model-context-protocol)

📌 **Prompt Engineering** → [บทที่ 2](../02_agentic_mastery.md) — เทคนิคการเขียน prompt ที่ดี

📌 **Frontend workflows** → [examples/04 — Frontend](04_phase4_frontend.md) — การสร้าง React components จาก design

📌 **Roles & Responsibilities** → [บทที่ 8](../08_roles_responsibilities.md) — บทบาทของ Designer/Frontend ในทีม
