---
tags: [STS, webboard, requirement, feature]
created: 2026-04-11
updated: 2026-04-11
status: in-progress
---

# STS — Web Board Requirement

> อ้างอิง: `/Users/jetsadasomporn/Vscode/STS/Web_Board_requirement.md`
> ที่มา: รายงานการประชุม 20 มีนาคม 2569, กระดานถามตอบ.md

---

## วัตถุประสงค์

- พื้นที่ให้ **Teleport ตั้งโพสต์คำถาม** เมื่อพบปัญหา
- สมาชิกที่เกี่ยวข้องรับ Alert และเข้ามาตอบ
- ทำงานร่วมกับระบบ Alert อัตโนมัติ

---

## ผู้ใช้งาน

| กลุ่ม | รับ Alert? | หมายเหตุ |
|-------|-----------|---------|
| NOC | ✅ | |
| Teleport | ✅ | ผู้ตั้งโพสต์หลัก |
| PM | ✅ | |
| Teleport Outsource | ❌ | ยกเว้น |

---

## Alert Requirements

| เหตุการณ์ | ผู้รับ Alert |
|----------|------------|
| มีโพสต์ใหม่ | NOC, Teleport, PM (ยกเว้น Outsource) |
| มีผู้ตอบโพสต์ | เจ้าของโพสต์ |
| กระทู้มีการ update | เจ้าของโพสต์ |

> [!important] ระบบ Alert ของ Web Board ต้องแยกจากระบบแจ้งเตือนเดิม

---

## Features ที่ระบุแล้ว ✅

- ตั้งโพสต์คำถาม
- แนบ **รูปภาพ** (ยืนยันชัด)
- แนบ **คลิปสั้น** (จากภาพอ้างอิง)
- **Like** / **Unlike**
- **Comment**
- ❌ ไม่ต้องมี **Share**

---

## UI/UX

- เรียบง่าย อ้างอิงรูปแบบ **Pantip**
- **WebBoard** เป็นเมนูแยกจาก Knowledge Base (KB)
- Layout แบบ card/feed เรียงต่อกัน

---

## สิ่งที่ยังไม่ระบุ (ห้ามสมมติ)

- [ ] สิทธิ์ในการสร้าง/แก้ไข/ลบโพสต์ต่อกลุ่ม
- [ ] การ reply comment
- [ ] Hashtag, search, filter
- [ ] ขนาด/จำนวน file แนบ
- [ ] รูปแบบ notification (bell, email, LINE?)
- [ ] Moderation system

---

> [!note] ที่มา
> มติที่ประชุม: "บริษัทรับทราบ" (ระเบียบวาระที่ 6 — 20 มี.ค. 2569)
