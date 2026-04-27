---
title: WebBoard Alert and UX Debugging Playbook
tags:
  - sts
  - web-board
  - alert
  - debugging
  - ux
  - playbook
type: playbook
status: active
created: 2026-04-16
updated: 2026-04-16
links:
  - "[[Character Count Policy]]"
  - "[[Alert Architecture Map]]"
  - "[[Web Board Feature Map]]"
---

# WebBoard Alert and UX Debugging Playbook

## Purpose
โน้ตหลักสำหรับสรุปโจทย์ Web Board รอบนี้แบบใช้งานจริง
- เก็บ root cause ที่เจอจริง
- เก็บวิธีแก้ที่ใช้ได้
- เก็บ pattern สำหรับ reuse รอบหน้า
- แตกต่อเป็น node ย่อยได้

## Related Nodes
- [[Character Count Policy]]
- [[Alert Architecture Map]]
- [[Web Board Feature Map]]

---

# 1. Problem Landscape

## 1.1 Initial Problem Cluster
โจทย์นี้ไม่ใช่ bug เดียว แต่เป็นชุดปัญหาหลายชั้น
- alert ไม่ขึ้น
- UX ของ post/comment ยังไม่พร้อม
- character count ภาษาไทยเพี้ยนจาก mental model ของ user
- responsive/mobile ยังแตก
- attachment flow ยังไม่สมบูรณ์
- search state หายเมื่อย้อนกลับจาก detail

## 1.2 Why This Was Hard
- มีหลาย repo เกี่ยวข้องพร้อมกัน
- อาการหน้าเว็บกับข้อมูลใน DB ไม่ตรงกัน
- ภาษาไทยทำให้เรื่อง count ยุ่งกว่าภาษาอังกฤษ
- บางปัญหาแก้ frontend อย่างเดียวไม่จบ ต้อง sync backend ด้วย

---

# 2. Core Outcomes

## 2.1 Alert Outcome
- แยก PBM/WebBoard alert flow ให้ใช้ `UserId`-based endpoints
- ทำให้ WebBoard action parse และ route ได้ถูกต้องใน portal
- ลดการพึ่ง `PersonId` ตรงจาก session แบบเดิม

อ่านเพิ่ม: [[Alert Architecture Map]]

## 2.2 Web Board UX Outcome
- create/edit/comment flow ใช้ได้จริงขึ้น
- attachment flow รองรับ add/remove/append ได้ดีขึ้น
- search state ไม่หายเมื่อย้อนกลับ
- mobile modal และหน้าจอแคบใช้งานได้ดีขึ้น

อ่านเพิ่ม: [[Web Board Feature Map]]

## 2.3 Character Count Outcome
- ทดลองหลาย algorithm แล้วพบว่า “ถูกทางทฤษฎี” ไม่เท่ากับ “ตรงกับที่ user คาดหวัง”
- สุดท้ายต้องแยก policy การนับให้ชัด และ sync frontend/backend

อ่านเพิ่ม: [[Character Count Policy]]

---

# 3. Key Lessons

## 3.1 Integration Bugs Need Layered Debugging
เวลา alert ไม่ขึ้น อย่ากระโดดสรุปทันทีว่าพังที่ frontend หรือ backend
ต้องแยก layer
- DB layer
- API layer
- session/user mapping layer
- UI parsing/rendering layer
- env/runtime layer

## 3.2 Thai Text Needs Policy, Not Guesswork
เรื่องนับตัวอักษรภาษาไทยห้ามใช้ความรู้สึกเดา
ต้องมี policy ว่า
- นับแบบไหน
- user ควรเห็นอะไร
- backend validate แบบไหน
- preview ตัดแบบไหน

## 3.3 UX Must Be Predictable
editor ที่ดีไม่ใช่แค่ไม่ error แต่ต้อง predictable
- พิมพ์แล้ว counter ต้องเข้าใจได้
- กด action สำคัญแล้วต้องมี confirm เมื่อเหมาะสม
- ย้อนกลับมาแล้วต้องเจอ state เดิมที่คาดไว้

---

# 4. Reusable Patterns

## 4.1 Alert Debug Pattern
เริ่มจากข้อมูลจริงก่อน
1. เช็ก `AlertQ`
2. เช็ก `AlertTo`
3. เช็ก unread function
4. เช็ก network request ของ portal
5. เช็ก mapping `UserId/PersonId`
6. เช็ก action parsing และ route
7. เช็ก env/process ที่รันจริง

## 4.2 Editor/Content Pattern
ทุก field ยาวต้องมี 4 อย่างชัด
- limit rule
- counter rule
- backend validation rule
- preview rule

## 4.3 Attachment Pattern
media editing ต้องแยก 3 state
- existing attachments
- removed attachments
- newly added attachments

---

# 5. Things To Improve Later
- ยกระดับ policy การนับตัวอักษรให้เป็น shared rule ทั้งระบบ
- ถ้า list โตขึ้นมาก ควรพิจารณา preview data จาก backend
- ถ้าต้อง realtime จริง ควรวาง push model แทน polling/focus refresh
- ทำ architecture note ต่อจาก alert flow ให้ลึกขึ้นเป็น sequence diagram

---

# 6. Operational Checklist

## 6.1 When Alert Breaks Again
- [ ] frontend ชี้ service ถูกตัว
- [ ] backend ชี้ service ถูกตัว
- [ ] DB ที่กำลังดูคือ DB เดียวกับที่ service ใช้
- [ ] unread function คืนข้อมูลจริง
- [ ] action format parse ได้
- [ ] route ปลายทางถูก
- [ ] session mapping ถูก
- [ ] process ล่าสุดถูก restart แล้ว

## 6.2 When Text Count Feels Wrong
- [ ] frontend counter ใช้อะไรนับ
- [ ] frontend limit ใช้อะไรนับ
- [ ] backend validate ใช้อะไรนับ
- [ ] preview ใช้อะไรตัด
- [ ] test ด้วยข้อความไทยจริง ไม่ใช่แค่ข้อความอังกฤษ

## 6.3 When Mobile UI Breaks
- [ ] modal scroll ได้
- [ ] action footer ยังอยู่ใน viewport
- [ ] attachment preview ไม่ดันปุ่มหาย
- [ ] ปุ่มสำคัญแตะได้บนมือถือ

---

# 7. Next Nodes
- [[Character Count Policy]]
- [[Alert Architecture Map]]
- [[Web Board Feature Map]]
