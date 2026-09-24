# ระบบแจ้งปัญหาคอมพิวเตอร์และ IT — เวอร์ชัน GitHub Pages + Apps Script API

สถาปัตยกรรมใหม่:
- **หน้าเว็บ** (`index.html`, `dashboard.html`) → โฮสต์บน **GitHub Pages** (ฟรี, ลิงก์นิ่ง, ไม่ผ่าน iframe ของ Apps Script)
- **Backend** (`backend/Code.gs`) → ยังเป็น **Google Apps Script + Google Sheets** เหมือนเดิม แต่ทำงานเป็น **JSON API** ให้หน้าเว็บเรียกผ่าน `fetch()`

วิธีนี้เลี่ยงปัญหา iframe ค้าง (`src="/blank"`) ที่เจอในเวอร์ชันก่อนหน้าได้ เพราะไม่ได้ใช้กลไก `HtmlService` ของ Apps Script ในการแสดงหน้าเว็บอีกต่อไป

---

## ส่วนที่ 1: ตั้งค่า Backend (Apps Script)

1. สร้าง Google Sheet ใหม่ → เมนู **ส่วนขยาย (Extensions) → Apps Script**
2. ลบโค้ดเดิมในไฟล์ `Code.gs` ทั้งหมด → วางเนื้อหาจากไฟล์ `backend/Code.gs` ที่ให้มา
3. เปิดไฟล์ `appsscript.json` (กดฟันเฟือง ⚙️ Project Settings → ติ๊ก "Show appsscript.json manifest file" ถ้ายังไม่เห็น) → แทนที่ด้วยเนื้อหาจาก `backend/appsscript.json`
4. บันทึก (Ctrl/Cmd+S) → เลือกฟังก์ชัน `setupSheets` จาก dropdown ด้านบน → กด ▶ Run → Authorize สิทธิ์ตามที่ระบบขอ
5. กลับไปที่ Google Sheet จะเห็นแท็บ **Tickets** และ **Config** ถูกสร้างขึ้นอัตโนมัติ
6. ในแท็บ **Config** กรอกค่า `IT_STAFF_LIST` เป็นรายชื่อทีม IT คั่นด้วยจุลภาค เช่น `สมชาย, สมหญิง, วิชัย` (จะไปแสดงเป็นตัวเลือก "มอบหมายงาน" ในแดชบอร์ด)
7. **Deploy → New deployment** → ⚙️ เลือก **Web app**
   - Execute as: **Me**
   - Who has access: **Anyone**
8. กด **Deploy** → คัดลอก **Web app URL** (ลงท้ายด้วย `/exec`) เก็บไว้ — จะใช้ในขั้นตอนถัดไป

**ทดสอบ API:** เปิด URL ที่คัดลอกมาต่อท้ายด้วย `?action=ping` ในเบราว์เซอร์ เช่น
`https://script.google.com/macros/s/xxxxx/exec?action=ping`
ถ้าเห็นข้อความ `{"success":true,"message":"IT Helpdesk API is running"}` แปลว่า backend พร้อมใช้งาน

---

## ส่วนที่ 2: ตั้งค่า Frontend (GitHub Pages)

1. เปิดไฟล์ `index.html` และ `dashboard.html` ที่ให้มา → หาบรรทัด:
   ```js
   const API_URL = 'https://script.google.com/macros/s/YOUR_DEPLOYMENT_ID/exec';
   ```
   แก้เป็น URL จริงที่คัดลอกมาจากขั้นตอนที่ 8 ด้านบน (ต้องแก้ **ทั้งสองไฟล์**)

2. สร้าง Repository ใหม่บน GitHub:
   - ไปที่ [github.com/new](https://github.com/new)
   - ตั้งชื่อ เช่น `it-helpdesk-system` → Public หรือ Private ก็ได้ (ถ้า Private ต้องมี GitHub Pro ถึงจะเปิด Pages ได้ฟรี ถ้าไม่แน่ใจให้ตั้ง Public)
   - กด **Create repository**

3. อัปโหลดไฟล์:
   - หน้ารีโพที่สร้างใหม่ → กด **uploading an existing file**
   - ลาก `index.html` และ `dashboard.html` (ที่แก้ API_URL แล้ว) ลงไป → กด **Commit changes**

4. เปิดใช้งาน GitHub Pages:
   - ไปที่แท็บ **Settings** ของ repo → เมนูซ้าย **Pages**
   - Source: เลือก **Deploy from a branch**
   - Branch: เลือก **main** และโฟลเดอร์ **/ (root)** → กด **Save**
   - รอสัก 1-2 นาที จะได้ลิงก์ประมาณ `https://<ชื่อบัญชี>.github.io/it-helpdesk-system/`

5. เปิดลิงก์นั้น → ควรเห็นหน้าฟอร์มแจ้งปัญหาทันที และกดลิงก์ "เข้าสู่แดชบอร์ดทีม IT" ด้านล่างเพื่อไปหน้าแดชบอร์ด (หรือเข้าตรงที่ `.../dashboard.html`)

---

## วิธีใช้งานระบบ

**พนักงานทั่วไป:** เข้าลิงก์ GitHub Pages → กรอกชื่อ, แผนก, รายละเอียดปัญหา, ความเร่งด่วน → กดส่ง → ได้เลข Ticket

**ทีม IT:** เข้าลิงก์ `.../dashboard.html` → ดู/กรอง/ค้นหา Ticket → คลิกแถวเพื่อเปลี่ยนสถานะ, มอบหมายงาน, บันทึกหมายเหตุ

ข้อมูลทั้งหมดเก็บในแท็บ **Tickets** ของ Google Sheet — เปิดดู/แก้ไข/Export ได้โดยตรงเช่นกัน (ทีม IT ต้องเปิดแดชบอร์ดหรือเปิด Google Sheet เองเพื่อดูว่ามี Ticket ใหม่เข้ามา เนื่องจากไม่มีการแจ้งเตือนอัตโนมัติแล้ว)

---

## หมายเหตุด้านความปลอดภัย

เว็บแอปนี้เปิดให้ "Anyone" เรียก API ได้ (จำเป็น เพราะพนักงานต้องแจ้งปัญหาได้โดยไม่ต้องล็อกอิน) หน้าแดชบอร์ดก็ไม่มีระบบล็อกอินป้องกัน — ใครมีลิงก์ `dashboard.html` ก็เข้ามาแก้ไขสถานะได้ ถ้าต้องการจำกัดสิทธิ์การเข้าถึงแดชบอร์ดเพิ่มเติม (เช่น ต้องล็อกอินด้วยอีเมลองค์กรก่อน) แจ้งได้ จะช่วยออกแบบเพิ่มให้
