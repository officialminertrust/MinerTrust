# แผงควบคุมหลังบ้าน (Admin Dashboard)

แผงควบคุมสำหรับเจ้าของเว็บ — แก้ราคา อัปเดตสต็อก จัดการคำสั่งซื้อ และดูสถิติยอดขาย โดยไม่ต้องแตะโค้ด

---

## 🔐 บัญชี Admin เริ่มต้น

เมื่อเปิดเว็บครั้งแรก ระบบจะสร้างบัญชี admin ให้อัตโนมัติ:

| Email | Password |
|---|---|
| `admin@minertrust.co.th` | `admin1234` |

> ⚠️ ในระบบ production จริงให้เปลี่ยนรหัสและใช้ bcrypt hash (ดู `BACKEND_INTEGRATION.md`)

## 🚪 วิธีเข้า

1. ไปที่ `pages/login.html`
2. ล็อกอินด้วยบัญชี admin ข้างบน
3. กดเมนู "แผงควบคุมหลังบ้าน" ที่ sidebar (จะเห็นเฉพาะ admin)
   หรือเข้าตรงที่ `pages/admin.html`

ถ้าผู้ใช้ทั่วไปพยายามเข้า `admin.html` จะถูก redirect ออกทันที

---

## 🧭 4 แท็บหลัก

### 1. ภาพรวม (Overview)
- KPI: ยอดขายรวม · กำไรขั้นต้น · จำนวนคำสั่งซื้อ · ลูกค้าในระบบ
- กราฟยอดขาย 30 วันล่าสุด
- โดนัทชาร์ตสถานะคำสั่งซื้อ
- ตารางคำสั่งซื้อล่าสุด 5 รายการ

### 2. เครื่องขุด (Miners)
- ค้นหาด้วยรุ่น/แบรนด์, กรองตามแบรนด์และสถานะสต็อก
- **แก้ไขราคา** — กรอกราคาใหม่ → ฟิลด์จะเปลี่ยนเป็นสีส้ม (dirty) → กดปุ่ม 💾 บันทึก
- **เปลี่ยนสถานะสต็อก** — เลือก `มีของพร้อมส่ง` / `เหลือน้อย` / `พรีออเดอร์` / `สินค้าหมด`
- **แก้กำไรอ้างอิง** ($/วัน — ใช้คำนวณรายได้โดยประมาณในแดชบอร์ดผู้ใช้)
- ปุ่ม "คืนค่าเริ่มต้น" ลบ override ทั้งหมด กลับไปใช้ราคาตั้งต้นใน `data.js`

> 💡 การแก้ไขจะมีผลทันที — ราคาใหม่จะโชว์ที่ index.html, product.html, cart.html, ROI calculator ทุกที่

### 3. คำสั่งซื้อ (Orders)
- ดูคำสั่งซื้อจากทุก user
- ค้นหา (หมายเลข / ชื่อ / อีเมล) + กรองสถานะ
- **เปลี่ยนสถานะ** ผ่าน dropdown (รอชำระ → ชำระแล้ว → จัดส่งแล้ว / ยกเลิก)
- กดปุ่ม 👁 เพื่อเปิด modal ดูรายละเอียดเต็ม (ที่อยู่จัดส่ง, รายการสินค้า)

### 4. สถิติยอดขาย (Stats)
- มูลค่าเฉลี่ยต่อคำสั่งซื้อ (AOV)
- จำนวนเครื่องที่ขายไปแล้ว
- อัตรากำไรขั้นต้น (default ตั้งต้นทุน 78% ของราคาขาย แก้ใน `api.js` ค่า `COGS_RATE`)
- โดนัทชาร์ตยอดขายแยกตามแบรนด์
- Progress bar ยอดขายแยกตามเหรียญ (BTC, DOGE/LTC, Monero ฯลฯ)
- ตารางรุ่นขายดี Top 8

---

## 🔌 API Reference (สำหรับนักพัฒนา)

ทั้งหมดอยู่ที่ `window.MT_API.admin.*` — async, throw error ถ้าไม่ใช่ admin:

```javascript
MT_API.admin.listMiners()                                  // → [Miner[]]
MT_API.admin.updateMiner(id, { price, stock, profitRef })  // → { ok, miner }
MT_API.admin.resetMiners()                                 // ล้าง override ทั้งหมด
MT_API.admin.listOrders()                                  // → [Order[]] ทั้งหมดในระบบ
MT_API.admin.updateOrder(id, { status })                   // → { ok, order }
MT_API.admin.stats()                                       // → KPI + breakdowns
```

ค่า `stock` ที่ valid: `in_stock` | `low_stock` | `preorder` | `out_of_stock`
ค่า `status` ที่ valid: `pending` | `paid` | `shipped` | `cancelled`

---

## 💾 การเก็บข้อมูล

| Storage Key | เนื้อหา |
|---|---|
| `mt_users` | บัญชีผู้ใช้ทั้งหมด (รวม admin) |
| `mt_orders` | คำสั่งซื้อทั้งหมด |
| `mt_miner_overrides` | การแก้ไข price/stock/profitRef ของแต่ละรุ่น |

ตอน `data.js` โหลด จะอ่าน `mt_miner_overrides` แล้ว apply ทับ defaults — เพราะฉะนั้นทุกหน้าจะเห็นค่าที่ admin แก้ทันที

---

## 🔄 เชื่อม Backend จริง

ทุก function ใน `MT_API.admin.*` มี comment endpoint shape ไว้แล้ว เช่น:

```javascript
// PATCH /api/admin/miners/:id   body: { price?, stock?, profitRef? }
async function apiAdminUpdateMiner(id, patch) { ... }
```

เปลี่ยนตัวบอดี้เป็น `fetch()` ตามตัวอย่างใน `BACKEND_INTEGRATION.md` ได้เลย — UI ไม่ต้องแก้
