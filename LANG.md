# ระบบสลับภาษา ไทย / English (i18n)

เว็บรองรับ 2 ภาษา: **ไทย (ค่าเริ่มต้น)** และ **English**
ผู้ใช้กดปุ่ม **TH/EN** บน nav bar เพื่อสลับ ระบบจะจำภาษาที่เลือกไว้

## ไฟล์ที่เกี่ยวข้อง
- `assets/js/i18n-dict.js` — คลังคำแปล (ไทย + อังกฤษ)
- `assets/js/i18n.js` — เอนจินสลับภาษา + ปุ่ม toggle
- ทุกหน้าโหลด 2 ไฟล์นี้ก่อน `data.js`

## วิธีเพิ่ม/แก้คำแปล
แก้ใน `i18n-dict.js` ทั้งสองภาษาให้ key ตรงกัน เช่น:
```js
th: { home: { hero: { ctaAll: 'ดูสินค้าทั้งหมด' } } }
en: { home: { hero: { ctaAll: 'View All Products' } } }
```

## วิธีทำให้ข้อความใหม่แปลได้
ใน HTML ใส่ attribute `data-i18n`:
```html
<h2 data-i18n="home.products.title">เครื่องขุด พร้อมส่ง</h2>
<input data-i18n="home.models.search" data-i18n-attr="placeholder" ...>
<h1 data-i18n="home.hero.title" data-i18n-html="true">...<em>...</em></h1>
```
- `data-i18n-attr="placeholder"` = แปล attribute แทน textContent
- `data-i18n-html="true"` = อนุญาต tag เช่น `<br>`, `<em>`

ใน JavaScript เรียก `window.t('key')` เพื่อดึงคำแปลภาษาปัจจุบัน
และฟัง event `mt-lang-changed` เพื่อ render ใหม่เมื่อสลับภาษา

## สถานะการแปล
- ✅ Nav + Footer (ทุกหน้า)
- ✅ หน้าแรก (index.html) — ครบทุก section
- ⏳ หน้าอื่น (cart, checkout, blog, admin ฯลฯ): nav/footer แปลแล้ว
  ส่วนเนื้อหาในหน้าเพิ่ม `data-i18n` ตามรูปแบบเดียวกันได้เลย
