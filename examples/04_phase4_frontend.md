# Phase 4: Frontend — Admin Dashboard

> **สัปดาห์ 4–5** — คุณฟร้อนท์สร้าง React admin dashboard โดยอ้างอิง API จาก backend ที่คุณแบ็คสร้างไว้ใน Phase 2–3
> คุณคิวทดสอบ UI flows และรายงาน bugs

---

## ภาพรวม Phase นี้

Phase นี้เป็นจุดที่ frontend เริ่มเป็นรูปเป็นร่าง โดยใช้ **cross-project references** ของ Claude Code เพื่ออ่าน Go DTOs จาก backend แล้วสร้าง TypeScript types ที่ตรงกัน จากนั้นสร้าง API client, React Query hooks และหน้า admin ทั้งหมด

```
shopfast-admin/src/
├── types/          ← สร้างจาก Go DTOs
├── lib/api.ts      ← Axios instance + interceptors
├── hooks/          ← React Query hooks
├── components/     ← Shared UI components
├── pages/
│   ├── products/   ← Product CRUD pages
│   └── orders/     ← Order management pages
├── layouts/        ← Sidebar, Header, Breadcrumbs
└── routes.tsx      ← React Router config
```

---

## ใครทำอะไรใน Phase นี้

| ลำดับ | ใครทำ | ทำอะไร | Claude Code ช่วยอย่างไร |
|-------|-------|--------|------------------------|
| 4.1 | 👤 คุณฟร้อนท์ | Scaffold โปรเจกต์ | สร้างโครงสร้าง Vite + React + TS + Tailwind |
| 4.2 | 👤 คุณฟร้อนท์ | สร้าง TS types จาก Go | Cross-project: อ่าน Go DTOs → สร้าง TS interfaces |
| 4.3 | 👤 คุณฟร้อนท์ | สร้าง API client + hooks | Axios instance, React Query hooks |
| 4.4 | 👤 คุณฟร้อนท์ | หน้า Product management | List, form, detail pages |
| 4.5 | 👤 คุณฟร้อนท์ | หน้า Order management | Order list, detail, status timeline |
| 4.6 | 👤 คุณฟร้อนท์ | Layout และ Navigation | Sidebar, header, breadcrumbs, responsive |
| 4.7 | 👤 คุณคิว | ทดสอบ Frontend | UI flows, accessibility, cross-browser |
| 4.8 | 👤 คุณฟร้อนท์ | แก้ UI bugs | Responsive issues, form validation edge cases |

---

## 4.1 Scaffold โปรเจกต์

👤 **คุณฟร้อนท์** เริ่มต้นด้วยการสร้างโครงสร้างโปรเจกต์ frontend ทั้งหมดในคำสั่งเดียว

```
สร้างโปรเจกต์ React admin dashboard ด้วย Vite + TypeScript 5 + Tailwind CSS v3:
1. โครงสร้างโฟลเดอร์: src/types/, src/lib/, src/hooks/, src/components/, src/pages/, src/layouts/
2. Vite config พร้อม path aliases (@/ → src/)
3. Tailwind พร้อม custom theme (brand colors, spacing)
4. React Router v6, React Query (TanStack Query v5) พร้อม providers
5. สร้าง .env.example กับ VITE_API_BASE_URL
```

Claude Code สร้างไฟล์ config ทั้งหมดให้ — `tsconfig.json` (strict mode), `vite.config.ts` (path aliases + API proxy), `tailwind.config.ts` (custom theme)

```typescript
// vite.config.ts (ย่อ)
export default defineConfig({
  plugins: [react()],
  resolve: { alias: { "@": path.resolve(__dirname, "./src") } },
  server: { proxy: { "/api": "http://localhost:3000" } },
});
```

💡 **เคล็ดลับ:** ระบุ proxy setting ตั้งแต่ scaffold เพื่อให้ dev server forward API requests ไปยัง backend ได้เลย ไม่ต้องแก้ CORS ระหว่าง develop

---

## 4.2 สร้าง TypeScript Types จาก Go DTOs

👤 **คุณฟร้อนท์** ใช้ cross-project references เพื่ออ่าน Go structs จาก backend repo แล้วสร้าง TypeScript interfaces ที่ตรงกันทุก field

📌 **Cross-ref:** [บทที่ 4](../04_fullstack_collaboration.md) — หัวข้อ 4.2.1 Cross-Project References

```bash
# เปิด Claude Code พร้อมอ้างอิง backend project
claude --add-dir /path/to/shopfast-api
```

```
อ่าน Go DTOs ทั้งหมดใน /path/to/shopfast-api/internal/dto/ แล้วสร้าง TypeScript interfaces:
1. อ่านทุก struct ที่มี json tags
2. แปลง Go types → TS types (uint→number, time.Time→string, *string→string|null)
3. สร้างแยกไฟล์ตาม domain: types/product.ts, types/order.ts, types/user.ts, types/common.ts
4. สร้าง types สำหรับทั้ง Request และ Response
5. Export type สำหรับ pagination: PaginatedResponse<T>
```

Claude Code อ่าน Go structs ข้ามโปรเจกต์แล้วสร้าง TypeScript types ที่ตรงกัน:

```typescript
// src/types/product.ts (ย่อ)
export interface Product {
  id: number;
  name: string;
  description: string;
  price: number;
  stock: number;
  category_id: number;
  image_url: string | null;
  is_active: boolean;
  created_at: string;
  updated_at: string;
}

export interface CreateProductRequest {
  name: string;
  description: string;
  price: number;
  stock: number;
  category_id: number;
  image_url?: string;
}

// src/types/common.ts
export interface PaginatedResponse<T> {
  data: T[];
  meta: { total: number; page: number; per_page: number; total_pages: number };
}
```

⚠️ **ข้อควรระวัง:** Go ใช้ `omitempty` กับ json tags — field ที่มี `omitempty` ใน response ควรเป็น optional (`?`) ใน TypeScript เพราะอาจไม่ส่งกลับเมื่อเป็น zero value

💡 **เคล็ดลับ:** ทุกครั้งที่ backend เปลี่ยน DTO ให้ใช้ prompt นี้:

```
เปรียบเทียบ Go struct ใน /path/to/shopfast-api/internal/dto/ กับ TypeScript interface ใน src/types/
หา fields ที่ไม่ตรงกัน แล้วอัพเดท TypeScript ให้ตรงกับ Go
```

---

## 4.3 สร้าง API Client และ React Query Hooks

👤 **คุณฟร้อนท์** สร้าง Axios instance พร้อม interceptors แล้วสร้าง React Query hooks ที่ใช้ API client นี้

📌 **Cross-ref:** [บทที่ 4](../04_fullstack_collaboration.md) — หัวข้อ 4.2.3 React Component Patterns (API Integration ด้วย React Query)

```
สร้าง API client ด้วย Axios ใน src/lib/api.ts:
1. Axios instance กับ baseURL จาก env
2. Request interceptor: แนบ JWT token จาก localStorage
3. Response interceptor: handle 401 → redirect to login, handle 500 → show toast
4. Type-safe error handling ด้วย ApiError type
```

Claude Code สร้าง Axios instance พร้อม request interceptor (แนบ JWT) และ response interceptor (401 → redirect to login) จากนั้นสร้าง React Query hooks แยกตาม domain:

```
สร้าง React Query hooks สำหรับ Product API ใน src/hooks/useProducts.ts:
1. useProducts(params) — GET /v1/products พร้อม search, filter, pagination
2. useProduct(id) — GET /v1/products/:id
3. useCreateProduct() — POST mutation + invalidate query
4. useUpdateProduct() — PUT mutation + invalidate query
5. useDeleteProduct() — DELETE mutation + invalidate query
ใช้ types จาก src/types/product.ts และ API client จาก src/lib/api.ts
```

```typescript
// src/hooks/useProducts.ts (ย่อ — แสดง pattern)
export function useProducts(filters: ProductFilters = {}) {
  return useQuery({
    queryKey: ["products", filters],
    queryFn: () => api.get<PaginatedResponse<Product>>("/v1/products", { params: filters })
      .then((res) => res.data),
  });
}

export function useCreateProduct() {
  const qc = useQueryClient();
  return useMutation({
    mutationFn: (data: CreateProductRequest) =>
      api.post<Product>("/v1/products", data).then((res) => res.data),
    onSuccess: () => qc.invalidateQueries({ queryKey: ["products"] }),
  });
}
```

💡 **เคล็ดลับ:** ให้ Claude Code สร้าง hooks ทุก domain ในคำสั่งเดียว — "สร้าง hooks เหมือนกับ useProducts แต่สำหรับ Order API และ User API ตาม types ที่มี" — Claude จะอ่าน pattern แล้วสร้างให้เหมือนกัน

---

## 4.4 สร้างหน้า Product Management

👤 **คุณฟร้อนท์** ใช้ **component prompt pattern** — ระบุ hook ที่ใช้, UI elements, states, interactions

### 4.4.1 Product List Page

```
สร้าง ProductListPage ใน src/pages/products/ProductListPage.tsx:
- ใช้ useProducts hook
- ตาราง: ชื่อ, ราคา, stock, category, สถานะ, วันที่สร้าง
- Search bar (debounced 300ms), filter dropdown (category, active/inactive)
- Pagination controls, ปุ่ม "เพิ่มสินค้า"
- แต่ละแถว: ปุ่ม edit และ delete (confirm dialog)
- Loading skeleton, empty state
- Responsive: table → card บน mobile
```

Claude Code สร้าง page ที่ใช้ `useProducts` hook พร้อม debounced search, filter state, และ conditional rendering — `ProductTableSkeleton` ขณะโหลด, `EmptyState` เมื่อไม่มีข้อมูล, และ `ProductTable` + `Pagination` เมื่อมีข้อมูล

### 4.4.2 Product Form (Create/Edit)

```
สร้าง ProductForm ใน src/pages/products/ProductForm.tsx:
- ใช้ได้ทั้ง create (ไม่มี initialData) และ edit (มี initialData)
- React Hook Form + Zod validation
- Fields: name (required, min 2), description (required), price (>0),
  stock (>=0), category_id (select), image_url (optional), is_active (toggle)
- Inline validation errors เป็นภาษาไทย
- Submit → useCreateProduct หรือ useUpdateProduct ตาม mode
- Toast notification สำเร็จ/ล้มเหลว, ปุ่มยกเลิก → navigate back
```

### 4.4.3 Product Detail Page

```
สร้าง ProductDetailPage ใน src/pages/products/ProductDetailPage.tsx:
- useProduct(id) — id จาก URL params
- แสดงรายละเอียดทั้งหมด: รูป, ชื่อ, ราคา, stock, category, สถานะ, timestamps
- ปุ่ม "แก้ไข" → /products/:id/edit, ปุ่ม "ลบ" พร้อม confirm
- Loading skeleton, error state (404 → not found)
```

---

## 4.5 สร้างหน้า Order Management

👤 **คุณฟร้อนท์** สร้างหน้าจัดการคำสั่งซื้อ ซับซ้อนกว่า product เพราะมี status timeline

### 4.5.1 Order List Page

```
สร้าง OrderListPage ใน src/pages/orders/OrderListPage.tsx:
- ตาราง: order number, ลูกค้า, จำนวนสินค้า, ยอดรวม, สถานะ (badge สี), วันที่
- Filter ตาม status: pending, confirmed, processing, shipped, delivered, cancelled
- Search ตาม order number หรือชื่อลูกค้า
- Status badge สีต่างกัน: pending=yellow, confirmed=blue, processing=indigo,
  shipped=purple, delivered=green, cancelled=red
```

### 4.5.2 Order Detail พร้อม Status Timeline

```
สร้าง OrderDetailPage ใน src/pages/orders/OrderDetailPage.tsx:
- ส่วนบน: order info (number, customer, date, total)
- ส่วนกลาง: Status Timeline แนวตั้ง — icon + ชื่อ status + timestamp + ใครเป็นคน update
  → status ปัจจุบัน highlight, ยังไม่ถึง greyed out
- ส่วนล่าง: ตาราง order items (สินค้า, จำนวน, ราคา, subtotal)
- ปุ่ม "อัพเดทสถานะ" → dropdown เลือก status ถัดไป พร้อม confirm
```

Claude Code สร้าง `StatusTimeline` component ที่ reusable — แต่ละ step แสดง icon (check สำหรับ past, number สำหรับ future), label, timestamp และ status ปัจจุบัน highlight ด้วยสีน้ำเงิน

### 4.5.3 Order Status Update

```
สร้าง OrderStatusUpdate component:
- dropdown แสดงเฉพาะ next valid states
- useUpdateOrderStatus mutation + confirm dialog + toast
- Business rules: pending→confirmed→processing→shipped→delivered
  cancelled ได้จากทุก status ยกเว้น delivered
```

⚠️ **ข้อควรระวัง:** Business rules สำหรับ state transitions ควรอยู่ที่ backend ด้วย (Phase 3) — frontend เป็นแค่ UX convenience ที่ซ่อน invalid options

---

## 4.6 Layout และ Navigation

👤 **คุณฟร้อนท์** สร้าง layout หลักของ admin dashboard

```
สร้าง AdminLayout ใน src/layouts/AdminLayout.tsx:
1. Sidebar: Logo, nav links (Dashboard, สินค้า, คำสั่งซื้อ, ลูกค้า, ตั้งค่า)
   พร้อม icons (Heroicons), active highlight, collapsible บน tablet, drawer บน mobile
2. Header: Breadcrumbs (auto จาก route), search bar, notification bell, user dropdown
3. Main: <Outlet /> ของ React Router
4. Responsive: desktop=expanded sidebar, tablet=icon-only, mobile=hamburger drawer
```

Claude Code สร้าง route config ที่รองรับ breadcrumbs อัตโนมัติ — แต่ละ route กำหนด `handle.breadcrumb` แล้ว Breadcrumbs component ใช้ `useMatches()` สร้าง path เช่น "สินค้า / iPhone 15 / แก้ไข" โดยอัตโนมัติ

---

## 4.7 QA ทดสอบ Frontend

👤 **คุณคิว** เริ่มทดสอบ admin dashboard ใช้ Claude Code ช่วยเขียน test plan และวิเคราะห์ UI

```
วิเคราะห์ UI flows ของ admin dashboard แล้วสร้าง test cases:
1. Product CRUD: list → create → verify → edit → verify → delete → verify
2. Order management: list → detail → update status → verify timeline
3. Navigation: sidebar links, breadcrumbs, back buttons
4. Search/filter: search → verify results → clear → verify reset
สำหรับแต่ละ flow ระบุ steps, expected results, edge cases
```

```
ตรวจสอบ accessibility ของ components ใน src/pages/ และ src/components/:
1. Interactive elements มี aria labels ไหม
2. Keyboard navigation ใช้งานได้ไหม
3. Color contrast ผ่าน WCAG AA ไหม
4. Screen reader: ตาราง, forms, dialogs มี proper roles ไหม
5. Focus management: dialog trap focus, ปิดแล้ว focus กลับที่เดิมไหม
```

คุณคิวรายงานปัญหาที่พบ:

| # | ประเภท | ปัญหา | ความรุนแรง |
|---|--------|-------|-----------|
| 1 | Responsive | ตารางสินค้าล้นจอบน iPhone SE (320px) | High |
| 2 | Validation | กรอกราคา -1 แล้ว submit ได้ ไม่ขึ้น error | High |
| 3 | A11y | Status badges ไม่มี aria-label | Medium |
| 4 | A11y | Delete confirm dialog ไม่ trap focus | Medium |
| 5 | UX | Pagination ไม่ scroll to top เมื่อเปลี่ยนหน้า | Low |
| 6 | Cross-browser | Sidebar transition กระตุกบน Firefox | Low |

---

## 4.8 แก้ไข UI Bugs

👤 **คุณฟร้อนท์** แก้ bugs ที่คุณคิวรายงาน

### แก้ Responsive Table (Bug #1)

```
ตาราง ProductListPage ล้นจอบน 320px:
- แก้ให้บน mobile (< 640px) เปลี่ยนเป็น card layout แทนตาราง
- card แต่ละใบ: ชื่อ, ราคา, stock, status badge, action buttons
- ใช้ Tailwind responsive classes, ดู pattern ใน src/components/
```

### แก้ Validation Edge Case (Bug #2)

```
แก้ Zod schema ใน ProductForm.tsx:
1. price ต้อง > 0, ทศนิยมไม่เกิน 2 ตำแหน่ง
2. stock ต้องเป็น integer >= 0
3. ทดสอบ edge cases: -1, 0, 0.001, 999999999
```

```typescript
// แก้ไข — refined Zod schema
price: z.number()
  .positive("ราคาต้องมากกว่า 0")
  .multipleOf(0.01, "ราคาต้องมีทศนิยมไม่เกิน 2 ตำแหน่ง")
  .max(99_999_999.99, "ราคาสูงเกินไป"),
stock: z.number()
  .int("จำนวนต้องเป็นจำนวนเต็ม")
  .min(0, "จำนวน stock ต้องไม่ติดลบ"),
```

### แก้ Accessibility Issues (Bug #3, #4)

```
แก้ accessibility issues:
1. เพิ่ม aria-label ให้ status badges — screen reader อ่าน "สถานะ: รอดำเนินการ"
2. แก้ DeleteConfirmDialog ให้ trap focus:
   - focus ปุ่ม "ยกเลิก" เมื่อเปิด, Tab/Shift+Tab วนใน dialog
   - Escape ปิด, focus กลับที่เดิม
   ใช้ <dialog> element หรือ headless UI ที่จัดการ focus อัตโนมัติ
```

---

## ✅ Checkpoint — จบ Phase 4

ก่อนไป Phase 5 ตรวจสอบว่าทุกอย่างเสร็จ:

| # | รายการ | สถานะ |
|---|--------|-------|
| 1 | Vite + React + TS + Tailwind scaffold ทำงานได้ | ☐ |
| 2 | TypeScript types ตรงกับ Go DTOs ทุก field | ☐ |
| 3 | API client มี auth interceptor + error handling | ☐ |
| 4 | React Query hooks ครบทุก API endpoint | ☐ |
| 5 | Product pages: list (search/filter/pagination), form, detail | ☐ |
| 6 | Order pages: list (status filter), detail (timeline), status update | ☐ |
| 7 | Layout: sidebar, header, breadcrumbs, responsive ทุก breakpoint | ☐ |
| 8 | QA bugs ทั้งหมดแก้แล้ว + a11y ผ่าน | ☐ |

```bash
cd shopfast-admin && npm run dev   # รัน dev server
npx tsc --noEmit                   # ตรวจ TypeScript errors
npm run lint                       # ตรวจ lint
# เปิด http://localhost:5173 → ทดสอบ CRUD สินค้า, order status, responsive
```

---

## เคล็ดลับ & ข้อผิดพลาดที่พบบ่อย

### 💡 เคล็ดลับ

1. **Cross-project references ช่วยประหยัดเวลามาก** — ใช้ `claude --add-dir` อ่าน backend code โดยตรง ไม่ต้อง copy-paste types

2. **Component prompt pattern** — ระบุ: (a) hook ที่ใช้, (b) UI elements, (c) states (loading, error, empty), (d) interactions ยิ่งละเอียดยิ่งได้ผลลัพธ์ตรง

3. **สร้าง hooks ก่อน pages** — สร้าง React Query hooks เสร็จก่อนแล้วค่อยสร้าง pages Claude Code จะอ่าน hooks แล้วสร้าง page ที่ใช้งานได้ทันที

4. **Responsive ตั้งแต่แรก** — บอก breakpoints + layout behavior ใน prompt ดีกว่ามาเพิ่มทีหลัง

5. **Validation messages ภาษาไทย** — ระบุใน prompt Claude Code จะสร้าง Zod schema ที่มี custom Thai messages

### ⚠️ ข้อผิดพลาดที่พบบ่อย

1. **Types ไม่ sync กับ backend** — เมื่อ backend เปลี่ยน DTO ต้องรัน cross-project comparison ทุกครั้ง อย่าแก้ types ด้วยมือ

2. **ลืม error/loading states** — ถ้าไม่ระบุใน prompt อาจได้แค่ happy path ต้องระบุ loading skeleton, error boundary, empty state

3. **Query key ไม่ consistent** — ระบุ convention เช่น `["resource", filters]` / `["resource", id]` ตั้งแต่แรก

4. **401 redirect loop** — interceptor redirect ไป `/login` แต่ login เรียก API ที่ return 401 → loop ต้อง exclude login endpoint

5. **Form reset หลัง submit** — ลืมบอก reset form หลัง create หรือ populate data สำหรับ edit ต้องระบุทั้ง create/edit flow

---

← [Phase 3: Core Features](03_phase3_core_features.md) | [Phase 5: Integration & CI/CD →](05_phase5_integration_cicd.md)
