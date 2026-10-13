# Shelf-Life Monitor (ชุดแยก / standalone)

เว็บ Shelf-Life Monitor ชุดแยกอิสระ — มีที่เก็บข้อมูลของตัวเอง ไม่เชื่อมกับแอปหรือฐานข้อมูลอื่นเลย

ประกอบด้วยแท็บ: ภาพรวมวันนี้ · Monitor item · Item NBS · OOS+ · %Shrink · Negative Stock · High Stock to DMS · สินค้า & กฎ · Buyer / Commercial · แปลง Julian · นำเข้า/ส่งออก

## ตอนนี้ทำงานแบบไหน

ค่าเริ่มต้นคือ **เก็บข้อมูลในเบราว์เซอร์ของแต่ละเครื่องเท่านั้น** (ยังไม่แชร์ระหว่างเครื่อง) ชื่อที่เก็บในเบราว์เซอร์ใช้คำนำหน้า `slmStandalone.` / `slmx-` จึงไม่ชนกับเว็บอื่นในบัญชี GitHub เดียวกัน

## เปิดใช้ฐานข้อมูลทีมของตัวเอง (Firebase)

1. เข้า <https://console.firebase.google.com> → **Add project** (ตั้งชื่อเอง เช่น `shelf-life-team`) — ไม่ต้องเปิด Analytics ก็ได้
2. เมนู **Build → Firestore Database → Create database** (เลือก Production mode และ region ใกล้เคียง เช่น `asia-southeast1`)
3. แท็บ **Rules** → วางเนื้อหาจากไฟล์ [`firestore.rules`](firestore.rules) → **Publish**
4. **Project settings (ฟันเฟือง) → General → Your apps → Web (`</>`)** → ตั้งชื่อแอป → จะได้ `firebaseConfig = { apiKey: ..., authDomain: ..., projectId: ..., storageBucket: ..., messagingSenderId: ..., appId: ... }`
5. เปิด `index.html` หา `const FIREBASE_CONFIG=null;` (ใกล้บรรทัด 1584) แล้วแทนที่ `null` ด้วยค่า config ที่ได้ เช่น

   ```js
   const FIREBASE_CONFIG={apiKey:"...",authDomain:"....firebaseapp.com",projectId:"...",storageBucket:"...",messagingSenderId:"...",appId:"..."};
   ```
6. commit + push — หน้าเว็บจะขึ้นป้าย “ข้อมูลทีม · Firebase” และทุกคนที่เปิดลิงก์จะเห็นข้อมูลชุดเดียวกัน

> **ความปลอดภัย:** กฎใน `firestore.rules` เปิดให้ทุกคนที่มีลิงก์อ่าน/เขียนได้ (เหมือนเว็บทีมทั่วไป ไม่มีล็อกอิน) — เหมาะกับลิงก์ที่แชร์เฉพาะในทีม ถ้าต้องการจำกัดสิทธิ์ ให้เพิ่ม Firebase Authentication แล้วแก้กฎ

ข้อมูลที่เก็บใน Firestore (คอลเลกชัน `shelfLife`): `shelfLife_products`, `shelfLife_contacts`, `shelfLife_batches`, `shelfLife_nbs_0…7`, `shelfLife_oos_0…3`, `shelfLife_imgs_0…7`, `shelfLife_reports` (+ ซับคอลเลกชัน `chunks` สำหรับรายงาน %Shrink / Negative Stock / High Stock)

## ไฟล์ข้อมูลที่อยู่ข้างหน้าเว็บ (snapshot — ไม่อัปเดตเอง)

| ไฟล์ | ใช้ทำอะไร |
|---|---|
| `ItemLocation_23_Aug_2026.xlsx` | ตำแหน่งสินค้า (ค้นสินค้า / เพิ่ม Item NBS) — ชื่อไฟล์ตั้งไว้ใน `loadMakroData()` (`const file=...`) |
| `item_images.json`, `item_images_extra.json` | รูปสินค้าตามเลข Item (extra = ดึงจาก makro.pro) |
| `item_barcodes.json`, `alt_barcodes.json` | Barcode ของสินค้า |
| `item_class.json` | Dept / Art Grp / Sub / Buyer UID ของสินค้า → ใช้หา Buyer จาก Buyer / Commercial ในหน้า Item NBS |
| `employee_only_items.json` | สินค้าที่ยังไม่มีตำแหน่ง (ชื่อ + barcode) |
| `product-img/`, `*.png` | รูปประกอบ / ไอคอน |

อยากอัปเดตข้อมูล: แทนที่ไฟล์เหล่านี้ด้วยชุดใหม่ (ชื่อเดิม) แล้ว push — ถ้าเปลี่ยนชื่อไฟล์ `ItemLocation_*.xlsx` ให้แก้ชื่อใน `index.html` ด้วย

## ทดสอบในเครื่อง

เปิด `index.html` ตรงๆ จากเครื่อง (file://) ได้ แต่จะไม่โหลดรูป/ข้อมูลสินค้า ควรเปิดผ่านเซิร์ฟเวอร์ เช่น `npx serve` หรือเปิดผ่าน GitHub Pages
