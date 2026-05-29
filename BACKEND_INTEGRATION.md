# Backend Integration Guide — MinerTrust TH

เอกสารนี้สอนวิธีเปลี่ยน **mock backend** (เก็บข้อมูลใน `localStorage`) ให้เป็น **backend จริง** เมื่อคุณพร้อม

## ภาพรวม

ปัจจุบันทุก endpoint รวมอยู่ในไฟล์ `assets/js/api.js` เป็น `window.MT_API` object — ทุกฟังก์ชันเป็น `async` และ return Promise (โครงเดียวกับ `fetch()`) ดังนั้นเวลาเปลี่ยนเป็น API จริง **HTML/UI ทุกหน้าไม่ต้องแก้** — แค่แก้ใน `api.js` เท่านั้น

```
ตอนนี้  : Frontend → api.js (localStorage)
อนาคต  : Frontend → api.js (fetch real API) → Backend → Database
```

---

## ตารางสรุป Endpoints

| Mock function | HTTP Method | Real endpoint (แนะนำ) | Body / Params | Response |
|---|---|---|---|---|
| `MT_API.signup({name,email,password})` | POST | `/api/auth/signup` | `{ name, email, password }` | `{ token, user }` |
| `MT_API.login({email,password})` | POST | `/api/auth/login` | `{ email, password }` | `{ token, user }` |
| `MT_API.logout()` | POST | `/api/auth/logout` | (header: Bearer token) | `{ ok: true }` |
| `MT_API.getSession()` | — | (decode JWT) | — | `{ user } \| null` |
| `MT_API.cart.get()` | GET | `/api/cart` | (auth header) | `{ items, count, subtotal }` |
| `MT_API.cart.add(product, qty)` | POST | `/api/cart/items` | `{ productId, qty }` | `{ items, count, subtotal }` |
| `MT_API.cart.update(id, qty)` | PATCH | `/api/cart/items/:id` | `{ qty }` | `{ items, count, subtotal }` |
| `MT_API.cart.remove(id)` | DELETE | `/api/cart/items/:id` | — | `{ items, count, subtotal }` |
| `MT_API.cart.clear()` | DELETE | `/api/cart` | — | `{ items, count, subtotal }` |
| `MT_API.orders.create(data)` | POST | `/api/orders` | `{ shipping, payment, items, total, ... }` | `{ orderId, order }` |
| `MT_API.orders.list()` | GET | `/api/orders` | (auth header) | `[orders]` |
| `MT_API.contact({...})` | POST | `/api/contact` | `{ name, email, phone, subject, message }` | `{ ok: true }` |
| `MT_API.newsletter(email)` | POST | `/api/newsletter` | `{ email }` | `{ ok: true }` |

---

## ตัวอย่างการเปลี่ยนเป็น fetch จริง

**ก่อน** (mock — ใน `api.js`):
```js
async function apiLogin({ email, password }) {
  await delay();
  const users = storage.get(STORAGE_KEYS.USERS, []);
  const user = users.find(u => u.email === email);
  if (!user) throw new Error('ไม่พบบัญชีนี้');
  // ...
}
```

**หลัง** (เชื่อม backend จริง):
```js
async function apiLogin({ email, password }) {
  const res = await fetch(API_BASE + '/api/auth/login', {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({ email, password })
  });
  if (!res.ok) {
    const err = await res.json();
    throw new Error(err.message || 'Login failed');
  }
  const data = await res.json();
  // เก็บ token ลง localStorage
  localStorage.setItem('mt_token', data.token);
  localStorage.setItem(STORAGE_KEYS.SESSION, JSON.stringify(data.user));
  return data;
}
```

ทำคล้ายกันกับทุกฟังก์ชัน — เพิ่มตัวแปร config ตอนบนไฟล์:
```js
const API_BASE = 'https://api.minertrust.co.th'; // เปลี่ยนตาม backend ของคุณ

function authHeaders() {
  const token = localStorage.getItem('mt_token');
  return token ? { 'Authorization': 'Bearer ' + token } : {};
}
```

แล้วทุก endpoint ที่ต้อง auth (cart, orders) ใส่ `...authHeaders()` ใน headers

---

## Database Schema (แนะนำ)

ถ้าใช้ PostgreSQL/MySQL/SQLite ใช้ schema นี้ได้เลย:

```sql
CREATE TABLE users (
  id VARCHAR(64) PRIMARY KEY,
  name VARCHAR(120) NOT NULL,
  email VARCHAR(180) UNIQUE NOT NULL,
  password_hash VARCHAR(255) NOT NULL,        -- bcrypt
  role VARCHAR(20) DEFAULT 'customer',
  created_at TIMESTAMP DEFAULT NOW()
);

CREATE TABLE products (
  id VARCHAR(64) PRIMARY KEY,                 -- e.g. m_01
  brand VARCHAR(60),
  model VARCHAR(120),
  price INTEGER,                              -- baht
  hash VARCHAR(40),
  hash_val NUMERIC,
  hash_unit VARCHAR(8),
  power INTEGER,                              -- watts
  coin VARCHAR(40),
  algo VARCHAR(40),
  release_date VARCHAR(20),
  profit_ref NUMERIC,                         -- USD/day reference
  stock VARCHAR(20)                           -- in_stock / low_stock / preorder
);

CREATE TABLE cart_items (
  id SERIAL PRIMARY KEY,
  user_id VARCHAR(64) REFERENCES users(id),
  product_id VARCHAR(64) REFERENCES products(id),
  qty INTEGER NOT NULL,
  added_at TIMESTAMP DEFAULT NOW(),
  UNIQUE(user_id, product_id)
);

CREATE TABLE orders (
  id VARCHAR(40) PRIMARY KEY,                 -- e.g. ORD-2026-0001
  user_id VARCHAR(64) REFERENCES users(id),
  status VARCHAR(20) DEFAULT 'pending',       -- pending/paid/shipped/cancelled
  subtotal INTEGER,
  shipping_cost INTEGER,
  total INTEGER,
  shipping_method VARCHAR(20),
  payment_method VARCHAR(20),
  shipping_json JSONB,                        -- address details
  items_json JSONB,                           -- snapshot of items
  created_at TIMESTAMP DEFAULT NOW()
);

CREATE TABLE contacts (
  id SERIAL PRIMARY KEY,
  name VARCHAR(120),
  email VARCHAR(180),
  phone VARCHAR(40),
  subject VARCHAR(120),
  message TEXT,
  created_at TIMESTAMP DEFAULT NOW()
);

CREATE TABLE newsletter (
  email VARCHAR(180) PRIMARY KEY,
  subscribed_at TIMESTAMP DEFAULT NOW()
);
```

---

## Tech Stack แนะนำ (ตามงบและความถนัด)

### ตัวเลือก 1: Supabase (ง่ายสุด แนะนำสำหรับเริ่มต้น)
- **ฟรี:** 500MB database + 1GB storage
- **Auth + Database + API พร้อมใช้** ไม่ต้องเขียน backend เอง
- ใช้ `@supabase/supabase-js` แทน fetch — เปลี่ยน `api.js` ใช้ `supabase.from('orders').select()` แทน
- เหมาะ: ไม่อยาก maintain server, traffic ไม่มากเกิน
- เว็บไซต์: https://supabase.com

### ตัวเลือก 2: Node.js + Express + SQLite (ราคาประหยัด)
- **Stack:** Node + Express + Prisma + SQLite (หรือ PostgreSQL)
- **Deploy:** Render (ฟรี / $7 ต่อเดือน), Railway ($5), หรือ VPS
- ต้องเขียน API เองตาม endpoints ในตารางด้านบน
- เหมาะ: ต้องการ control เต็มที่, มีคนเขียน Node เป็น

### ตัวเลือก 3: Firebase (Google)
- คล้าย Supabase แต่ใช้ NoSQL (Firestore)
- Auth built-in รองรับ social login ครบ
- ฟรี tier ใจดี เหมาะกับ traffic ปานกลาง

---

## ขั้นตอนการ Migrate

1. **ตั้ง backend ก่อน** (Supabase / Express / Firebase) ตามทางเลือกที่เลือก
2. **สร้าง schema** ตามตารางด้านบน
3. **Implement endpoints** ทีละตัวตามตาราง
4. **เปลี่ยน `api.js`** ทีละฟังก์ชัน — แนะนำเริ่มจาก `signup/login` ก่อน แล้วค่อยขยับไป `cart` และ `orders`
5. **เพิ่ม CORS** ใน backend ให้ allow domain ของคุณ
6. **ทดสอบ flow ครบทุกหน้า** — signup → login → เลือกสินค้า → cart → checkout → dashboard
7. **เพิ่ม Payment Gateway จริง** (Omise / 2C2P / Stripe สำหรับการชำระเงินจริง)

---

## เรื่องสำคัญที่ต้องทำเพิ่มก่อน Production

- [ ] เปลี่ยน `pseudoHash()` ใน mock เป็น **bcrypt** ที่ backend
- [ ] ใช้ **JWT** หรือ session cookie สำหรับ auth (mock ใช้แค่ flag ใน localStorage)
- [ ] เพิ่ม **HTTPS** (Let's Encrypt ฟรี)
- [ ] เพิ่ม **rate limiting** ที่ backend (ป้องกัน brute force)
- [ ] **Validate input** ทั้ง frontend และ backend
- [ ] เชื่อม **payment gateway จริง** — แนะนำ Omise (รองรับ PromptPay / TrueMoney / บัตรเครดิต ในไทย)
- [ ] **Email service** — SendGrid / Mailgun / Resend สำหรับส่งอีเมลยืนยันคำสั่งซื้อ
- [ ] **Backup database** อัตโนมัติ
- [ ] **Sentry / LogRocket** สำหรับ monitor error

---

## คำถามที่พบบ่อย

**Q: ตอนนี้เปิดไฟล์ใช้ได้เลยไหม?**
A: ได้เลย เปิด `index.html` ในเบราว์เซอร์ ทุกอย่างทำงานปกติ (ลงทะเบียน, login, ใส่ตะกร้า, สั่งซื้อ, ดู dashboard) แค่ข้อมูลอยู่ในเครื่องของผู้ใช้แต่ละคน

**Q: ข้อมูลที่ user สร้างหายไปไหม?**
A: ข้อมูลอยู่ใน `localStorage` ของเบราว์เซอร์นั้น — ถ้าล้าง cache หรือเปิดในเบราว์เซอร์อื่น/เครื่องอื่น จะไม่เห็น

**Q: หน้าเว็บ deploy ได้ไหมแม้ไม่มี backend จริง?**
A: ได้ — upload ทุกไฟล์ไปที่ static hosting เช่น Netlify, Vercel, GitHub Pages, Cloudflare Pages ก็ใช้งานได้ทันที (ยังเป็น demo mode)

**Q: ถ้าจะ migrate ไป backend จริง ต้องเริ่มจากไหน?**
A: แนะนำ Supabase สำหรับมือใหม่ — สมัครฟรี, สร้าง project, copy SQL schema ด้านบนไปรัน, แล้วใส่ API key/URL ใน `api.js` เปลี่ยน fetch ทีละฟังก์ชัน
