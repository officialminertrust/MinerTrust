# MinerTrust TH — เว็บไซต์ ASIC Miner

เว็บไซต์เต็มรูปแบบสำหรับร้านขายเครื่องขุด ASIC ในไทย โทนสีฟ้าหรู ๆ เรียบ ๆ พร้อม mock backend ใช้งานได้ทันที

## ✨ ฟีเจอร์ครบ

- **10 หน้าหลัก** — Home, Login, Sign Up, Products, Product Detail, Cart, Checkout, Order Success, Blog, Blog Post, Contact, Dashboard
- **🆕 Admin Dashboard (หลังบ้าน)** — แก้ราคา / สถานะสต็อก / ดูยอดขาย กำไร และคำสั่งซื้อทั้งหมด โดยไม่ต้องแก้โค้ด (ดู `ADMIN.md`)
- **Mock Backend** ด้วย localStorage — ใช้งานได้ทันที (signup, login, cart, orders ทำงานจริง)
- **Live Crypto Prices** จาก Binance API — อัปเดตทุก 15 วินาที
- **ROI Calculator** เลือกรุ่นแบบคลิก พร้อมแสดงราคาเหรียญสด
- **Responsive** ทำงานบน desktop, tablet, mobile
- **โลโก้ช้าง** ทั้ง navigation และ favicon
- **อิงตามมาตรฐาน UX** ครบทั้ง 10 หน้าตาม infographic ที่ให้ไว้

## 🚀 วิธีใช้งาน

1. แตก zip ออกมา
2. เปิด `index.html` ในเบราว์เซอร์ — ทำงานได้ทันที ไม่ต้อง install อะไร
3. ลองสมัครสมาชิก → login → ใส่สินค้าลงตะกร้า → checkout → ดู dashboard

## 📁 โครงสร้างไฟล์

```
mt/
├── index.html              ← หน้าแรก (Homepage)
├── BACKEND_INTEGRATION.md  ← วิธีเชื่อม backend จริง
├── README.md
├── assets/
│   ├── css/
│   │   ├── main.css        ← styles หลัก (navigation, hero, ฯลฯ)
│   │   └── pages.css       ← styles เสริม (auth, cart, dashboard, ฯลฯ)
│   ├── js/
│   │   ├── data.js         ← ข้อมูล ASIC Miners (27 รุ่น)
│   │   ├── blog-data.js    ← ข้อมูลบทความ
│   │   ├── api.js          ← Mock API (← จุดเดียวที่ต้องแก้เมื่อเชื่อม backend จริง)
│   │   ├── common.js       ← nav, footer, toast, cart badge
│   │   └── home.js         ← JS เฉพาะหน้าแรก
│   └── img/
│       └── logo.png        ← โลโก้ช้าง
└── pages/
    ├── login.html
    ├── signup.html
    ├── product.html        ← ?id=m_01
    ├── cart.html
    ├── checkout.html
    ├── order-success.html  ← ?id=ORD-2026-0001
    ├── blog.html
    ├── blog-post.html      ← ?slug=...
    ├── contact.html
    └── dashboard.html
```

## 🎨 ระบบสี (60-30-10)

| % | สี | ใช้กับ |
|---|---|---|
| 60% | `#F4F7FB` ฟ้าขาวอ่อน | พื้นหลังหลัก, การ์ดสีขาว |
| 30% | `#EAF1F9` ฟ้าอ่อน + `#0F1B2D` กรมท่า | พื้นรอง, navigation, CTA section |
| 10% | `#2563B8` น้ำเงิน royal + `#6FA8DC` ฟ้าอ่อน | accent, ราคา, ปุ่ม, ไอคอน |

## 🛠 เทคโนโลยี

- **Vanilla HTML/CSS/JS** — ไม่มี build step ไม่มี dependencies
- **Fonts:** Cormorant Garamond (headline serif) + Sarabun (body ภาษาไทย)
- **Icons:** Tabler Icons (webfont CDN)
- **Live data:** Binance public API (BTC, LTC, ETC, ZEC, XMR + อื่น ๆ)

## 🔌 เชื่อม Backend จริง

อ่าน `BACKEND_INTEGRATION.md` — มีรายละเอียดครบ:
- ตารางสรุปทุก endpoint
- SQL schema พร้อมใช้
- ตัวอย่าง code เปลี่ยน mock → fetch
- คำแนะนำ Supabase/Firebase/Node.js
- Checklist สำหรับ production

## 🧪 Demo Promo Codes (Cart)

ลองใช้ในหน้า cart:
- `WELCOME10` — ลด 10%
- `MINER500` — ลด ฿500

## 📝 หมายเหตุ

นี่เป็น demo / template ก่อน production ต้อง:
- ใส่ payment gateway จริง (Omise / 2C2P / Stripe)
- เปลี่ยน mock auth เป็น JWT + bcrypt
- เพิ่ม HTTPS + CORS + rate limiting
- ใช้ database จริง

ดู checklist ใน `BACKEND_INTEGRATION.md`
