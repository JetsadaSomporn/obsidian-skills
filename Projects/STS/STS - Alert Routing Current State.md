---
tags: [STS, alert, PBM, WebBoard, current-state]
created: 2026-04-12
updated: 2026-04-12
---

# STS — Alert Routing Current State

> สถานะปัจจุบันของการกระจายแจ้งเตือนในระบบ STS
> โน้ตนี้แยกจากงาน [[STS - SLA Alert Implementation]] เพื่อไม่ให้ปนกับ Req#7 / Req#30

---

## สรุปสั้น

- **Problem Management (PBM)** → เข้า **PBM Bell**
- **Web Board** → เข้า **JOB Bell**

---

## Problem Management (PBM)

ปัจจุบันฝั่ง backend ส่ง alert โดยใช้ `Action` format แบบนี้:

```text
PBM|{jobNo}|{jobId}
```

เมื่อ frontend โหลดรายการ alert ใน header จะใช้ prefix `PBM|` เพื่อแยก alert ไปกองที่กระดิ่ง **PBM** โดยเฉพาะ

ผลลัพธ์:
- เปิดงาน PBM → ขึ้น PBM Bell
- Assign / Update Status / Discovered / Approve → ขึ้น PBM Bell
- กดรายการจาก PBM Bell → ไปหน้า `problem-management/job-detail/{jobId}` ได้

สรุป: **PBM alert ทำงานใน PBM Bell แล้ว**

---

## Web Board

ปัจจุบัน Web Board มีการยิง alert แล้วจาก backend เช่น:

```text
WEBBOARD|POST|{postId}
WEBBOARD|COMMENT|{postId}
```

แต่ frontend ฝั่ง header แยกไป PBM Bell เฉพาะ alert ที่ขึ้นต้นด้วย `PBM|` เท่านั้น
ดังนั้น alert ของ Web Board จะไม่เข้า PBM Bell และจะถูกรวมอยู่ฝั่ง **JOB Bell**

ผลลัพธ์:
- สร้างโพสต์ใหม่ → ขึ้น JOB Bell
- มีคนคอมเมนต์ / แก้ไขคอมเมนต์ → ขึ้น JOB Bell

สรุป: **Web Board alert ทำงานแล้ว แต่ตอนนี้อยู่ใน JOB Bell**

---

## ข้อจำกัดปัจจุบัน

### PBM
- flow หลักใช้งานได้แล้วใน PBM Bell
- มี route ตอนกด notification แล้ว

### Web Board
- alert ขึ้นใน JOB Bell ได้
- แต่ยังไม่มีการแยก bell เฉพาะของ Web Board
- และควรเพิ่ม deep link ตอนกด notification เพื่อเปิดหน้า post/comment ตรง ๆ

---

## แนวทางใช้งานตอนนี้

ถ้าเอาแบบ pragmatic และเน้นให้ใช้งานได้ก่อน:

- ให้ **PBM อยู่ PBM Bell ต่อไป**
- ให้ **Web Board อยู่ JOB Bell ไปก่อน**
- ถ้าจะปรับต่อภายหลัง ค่อยเพิ่ม routing ตอนกด Web Board notification ให้เปิดหน้า Web Board โดยตรง

---

## หมายเหตุ

โน้ตนี้ตั้งใจแยกจาก:
- [[STS - SLA Alert Implementation]]
- งาน Req#7 / Req#30

เพื่อให้ใช้เป็น reference สำหรับ **สถานะปัจจุบันของ alert routing** โดยไม่ปนกับงาน SLA alert
