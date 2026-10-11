# Chem RecHUB — ระบบพิมพ์สติกเกอร์ติดกระสอบ

เว็บแอปหน้าเดียว (`index.html`) ที่อ่านไฟล์ Excel ในเบราว์เซอร์แล้วพิมพ์สติกเกอร์ — **ข้อมูลไม่ถูกส่งไปที่ใดเลย** ประมวลผลในเครื่องผู้ใช้ทั้งหมด

โปรเจกต์นี้ให้ PM2 เป็นผู้ดูแลโปรเซสเสิร์ฟหน้าเว็บ (`server.js`) — เซิร์ฟเวอร์ไม่มี dependency จึง **ไม่ต้องรัน `npm install` บนเครื่อง production**

| รายการ | ค่า |
|---|---|
| ชื่อโปรเซส PM2 | `chem-rechub` (cluster 2 ตัว) |
| พอร์ตเริ่มต้น | `4101` (แก้ที่ `ecosystem.config.js`) |
| ตรวจสุขภาพ | `GET /health` |
| Node.js | 18 ขึ้นไป |

## โครงสร้าง

```
index.html              หน้าเว็บทั้งหมด
server.js               เซิร์ฟเวอร์ไฟล์สถิต (เสิร์ฟเฉพาะ index.html, /vendor, /health)
ecosystem.config.js     ตั้งค่า PM2
vendor/                 ไลบรารีที่ฝังไว้ (xlsx, ไอคอน, ฟอนต์ Prompt, Tailwind CSS ที่คอมไพล์แล้ว) — ใช้งานได้โดยไม่ต้องต่ออินเทอร์เน็ต
tests/server.test.js    ทดสอบเซิร์ฟเวอร์ (npm test)
styles/, tailwind.config.js   ใช้ตอน build CSS เท่านั้น
logs/                   log ของ PM2 (ไม่ขึ้น git)
```

## ติดตั้งครั้งแรกบนเครื่อง production (Windows)

```bat
git clone https://github.com/TUAPUAN-CODE/Chem_RecHUB.git
cd Chem_RecHUB
npm test
pm2 start ecosystem.config.js
pm2 save
```

1. `npm test` ตรวจว่าเซิร์ฟเวอร์และไฟล์ vendor ครบ (ไม่ต้องติดตั้งอะไรก่อน)
2. ตรวจว่าพอร์ต `4101` ว่าง: `netstat -ano | findstr :4101` (ถ้าไม่ว่างให้เปลี่ยน `PORT` ใน `ecosystem.config.js`)
3. เปิดไฟร์วอลล์ขาเข้าพอร์ตนี้เฉพาะเครือข่ายภายใน
4. เปิด `http://<ที่อยู่เครื่อง>:4101/` และ `http://<ที่อยู่เครื่อง>:4101/health`

ให้ PM2 เริ่มเองหลังเปิดเครื่อง: ตั้งตามที่ใช้กับโปรเจกต์อื่นบนเครื่อง (เช่น Task Scheduler ที่สั่ง `pm2 resurrect`) แล้ว `pm2 save` ทุกครั้งที่เพิ่ม/ลบโปรเซส

## อัปเดตโค้ด

```bat
cd Chem_RecHUB
git pull
npm test
pm2 reload chem-rechub
```

`reload` สลับทีละตัวจากทั้งหมด 2 ตัว (cluster) และรอให้ตัวใหม่เปิดพอร์ตสำเร็จก่อน (`wait_ready`) ผู้ใช้จึงไม่เจอช่วงที่เปิดไม่ได้ — **ห้ามใช้ `pm2 restart all`** เพราะจะกระทบโปรเซสของโปรเจกต์อื่นบนเครื่อง

### ถอยกลับ

```bat
git log --oneline -5
git checkout <commit ก่อนหน้า> -- .
pm2 reload chem-rechub
```

## คำสั่งที่ใช้บ่อย

| ต้องการ | คำสั่ง |
|---|---|
| ดูสถานะ | `pm2 list` / `pm2 describe chem-rechub` |
| ดู log สด | `pm2 logs chem-rechub` |
| ดู log ย้อนหลัง | ไฟล์ `logs/out.log`, `logs/error.log` |
| ดูว่าใครเปิดหน้านี้ | ตั้ง `ACCESS_LOG: '1'` ใน `ecosystem.config.js` แล้ว `pm2 reload chem-rechub --update-env` |
| หยุดชั่วคราว | `pm2 stop chem-rechub` |

ตั้งตัวหมุน log ครั้งเดียวต่อเครื่อง (ใช้ร่วมกับทุกโปรเซส): `pm2 install pm2-logrotate`

## แก้ไขหน้าเว็บ

- แก้ `index.html` ได้ตรงๆ แล้ว `git pull` + `pm2 reload` ที่เครื่อง production (เซิร์ฟเวอร์ตรวจเวลาแก้ไฟล์ทุกคำขอ)
- ถ้าเพิ่ม/เปลี่ยน class ของ Tailwind ต้อง build CSS ใหม่ **บนเครื่องพัฒนา** (ไม่ทำบน production):

```bat
npm install
npm run build:css
npm test
git add vendor/tailwind.css
```

- ถ้าจะอัปเกรดไลบรารีใน `vendor/` ให้ทดสอบกับไฟล์ Excel จริงก่อน แล้วอัปเดตรายการใน `LICENSES-vendor.txt`

## ความปลอดภัย

- ไม่มีระบบ login — ใครเข้าถึงที่อยู่ได้ก็ใช้ได้ ควรเปิดเฉพาะเครือข่ายภายในโรงงาน
- เซิร์ฟเวอร์ตั้ง Content-Security-Policy ให้โหลดได้เฉพาะจากตัวเอง (ไม่เรียกอินเทอร์เน็ต), ห้ามฝังในเว็บอื่น, ปฏิเสธ method ที่ไม่ใช่ GET/HEAD
- เสิร์ฟเฉพาะ `index.html`, `/vendor/*`, `/health` — `server.js`, `.git`, `logs` เปิดจากเว็บไม่ได้ (มี test ตรวจ)
- ห้ามใส่รหัสผ่าน/โทเคน/ข้อมูลลูกค้าลงในรีโปนี้

## แก้ปัญหาเบื้องต้น

| อาการ | สาเหตุที่พบบ่อย | วิธีแก้ |
|---|---|---|
| `pm2 list` เห็น `errored` | พอร์ตชน | ดู `logs/error.log` ถ้าเห็น `port_in_use` ให้เปลี่ยน `PORT` |
| เปิดหน้าไม่ขึ้นจากเครื่องอื่น | ไฟร์วอลล์ | เปิดพอร์ตขาเข้า `4101` |
| ไอคอน/ตัวอักษรเพี้ยน | แคชเบราว์เซอร์เก่า | กด Ctrl+F5 |
| พิมพ์สติกเกอร์ไม่ออก | เบราว์เซอร์บล็อก pop-up | อนุญาต pop-up ของที่อยู่เว็บนี้ |
