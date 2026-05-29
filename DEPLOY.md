# 🚀 วิธี Deploy เว็บ MinerTrust ขึ้นอินเทอร์เน็ตจริง

คู่มือนี้สอนวิธีเอาเว็บที่ทำเสร็จแล้ว ขึ้นออนไลน์ให้ใครก็เข้าได้ผ่าน URL — **ใช้เวลา 2 นาที ฟรี ไม่ต้องบัตรเครดิต**

---

## 🥇 วิธีที่ 1: Netlify Drop (ง่ายสุด แนะนำ) — 2 นาที

**สิ่งที่จะได้:** URL เช่น `https://amazing-miner-abc123.netlify.app`

### ขั้นตอน

1. **แตก zip ออก** — แตก `minertrust-th.zip` จะได้โฟลเดอร์ `mt/`

2. **เปิดเว็บไซต์**: https://app.netlify.com/drop

3. **ลากโฟลเดอร์ `mt/` ทั้งโฟลเดอร์** ไปวางในกรอบที่เขียนว่า "Drag and drop your site folder here"

   > ⚠️ **สำคัญ:** ลาก**โฟลเดอร์** ไม่ใช่ไฟล์ zip ไม่ใช่ index.html เดี่ยว ๆ

4. **รอ ~30 วินาที** — Netlify จะ upload และ deploy ให้อัตโนมัติ

5. **ได้ URL** เช่น `https://amazing-miner-abc123.netlify.app` — copy ไปแชร์ให้ใครก็ได้

### ถ้าอยากตั้งชื่อ URL เอง (เช่น `minertrust-th.netlify.app`)

หลัง deploy แล้ว:
1. คลิก "Sign up" (ฟรี ใช้ Google login ได้)
2. ไปที่ Site settings → Change site name → ใส่ชื่อที่อยากได้
3. URL จะเปลี่ยนเป็น `https://YOURNAME.netlify.app` ทันที

### ถ้าอยากใช้โดเมนของตัวเอง (เช่น `minertrust.co.th`)

1. ซื้อโดเมนจาก GoDaddy / Namecheap / RuayHost
2. ใน Netlify → Domains → Add custom domain
3. ทำตามคำแนะนำตั้งค่า DNS (Netlify จะบอกค่าที่ต้อง copy ไปใส่)
4. รอ DNS propagate ~10 นาที ก็ใช้โดเมนตัวเองได้

---

## 🥈 วิธีที่ 2: Vercel — 3 นาที

**สิ่งที่จะได้:** URL เช่น `https://minertrust-th.vercel.app`

1. ไปที่ https://vercel.com/new
2. Sign up ด้วย GitHub / GitLab / Email (ฟรี)
3. คลิก "Browse" → เลือกโฟลเดอร์ `mt/`
4. คลิก Deploy
5. รอ ~30 วินาที ได้ URL

---

## 🥉 วิธีที่ 3: Cloudflare Pages — 3 นาที

**ดีกว่าตรงไหน:** เร็วที่สุดในเอเชีย (มี edge server ในไทย-สิงคโปร์)

1. ไปที่ https://pages.cloudflare.com
2. Sign up ด้วย email (ฟรี)
3. Create a project → Direct Upload
4. ลากโฟลเดอร์ `mt/` ไปวาง
5. ตั้งชื่อ project → Deploy
6. ได้ URL เช่น `https://minertrust-th.pages.dev`

---

## 🆓 วิธีที่ 4: GitHub Pages — 5 นาที (ถ้ามี GitHub account)

**ดีกว่าตรงไหน:** ถ้าคุณจะแก้ไขเว็บบ่อย ๆ จะ deploy ใหม่ทุกครั้งที่ commit อัตโนมัติ

1. สร้าง repository ใหม่บน GitHub ชื่อ `minertrust-th`
2. Upload ไฟล์ทั้งหมดในโฟลเดอร์ `mt/` ไปที่ root ของ repository
3. ไปที่ Settings → Pages → Source: Deploy from a branch → main → / (root) → Save
4. รอ ~1 นาที ได้ URL เช่น `https://yourusername.github.io/minertrust-th/`

---

## ⚠️ ข้อควรรู้

### Deploy บน static hosting (ทั้ง 4 วิธีด้านบน) = **เว็บใช้งานได้** แต่
- ข้อมูล user (signup, cart, orders) อยู่ใน **localStorage ของแต่ละเครื่องที่เปิดเว็บ**
- คนละคนเปิดเว็บ → เห็นข้อมูลคนละชุด (เพราะยังไม่มี database จริงที่ share ข้อมูล)
- คนเดิมเปิดเครื่องอื่น → ก็เห็นข้อมูลคนละชุด

### ถ้าอยากให้ข้อมูลของ user อยู่บน cloud (เห็นเหมือนกันทุกที่)
ต้องเชื่อม backend จริง ดูใน `BACKEND_INTEGRATION.md`

แนะนำ **Supabase** (ฟรี, ไม่ต้องเขียน server, มี database + auth พร้อมใช้)

---

## 📋 Checklist ก่อน Deploy

- [ ] แตก zip แล้ว ได้โฟลเดอร์ `mt/`
- [ ] เปิด `mt/index.html` ในเบราว์เซอร์ทดสอบก่อน — ดูว่าโหลดได้, รูปขึ้น, ลิงก์ไปหน้าอื่นได้
- [ ] เลือก hosting (Netlify ง่ายสุด)
- [ ] Deploy
- [ ] ทดสอบ URL ที่ได้ — ลองเข้าจากมือถือดูด้วย

---

## 🐛 Troubleshooting

### "รูปไม่ขึ้น / ลิงก์ไปหน้าอื่นไม่ได้"
→ คุณ deploy แค่ `index.html` ไฟล์เดียว ต้อง deploy ทั้งโฟลเดอร์ `mt/` (รวม `assets/` และ `pages/`)

### "เปิด URL แล้วเป็นหน้า 404"
→ ตรวจว่าใน root ของ deploy มีไฟล์ชื่อ `index.html` หรือไม่ (ไม่ใช่ `mt/index.html`)
→ ถ้ามีโฟลเดอร์ `mt/` ครอบอยู่อีกชั้น ให้ลากเฉพาะข้างใน `mt/` ออกมา หรือเลือกโฟลเดอร์ `mt/` ตรง ๆ

### "ตอน upload Netlify บอก files too large"
→ ลบ `BACKEND_INTEGRATION.md`, `README.md`, `DEPLOY.md` ออกก่อน (ไม่ใช่ไฟล์เว็บ) ขนาดจะเล็กลง

### "ทำไม URL ดูแปลก ๆ เปลี่ยนได้ไหม"
→ Sign up บน Netlify (ฟรี) แล้วเปลี่ยนชื่อในส่วน Site Settings (ทำได้)

---

## 🎯 สรุปสำหรับคุณ

**แนะนำ:** ใช้วิธีที่ 1 (Netlify Drop) — ลากโฟลเดอร์ `mt/` ไปวางที่ https://app.netlify.com/drop ได้ URL ทันที 2 นาที จบ

ถ้าติดตรงไหน ส่งข้อความหาผม ผมช่วยแก้ได้
