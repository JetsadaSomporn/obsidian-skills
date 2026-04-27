---
title: Web Board Feature Map
tags:
  - sts
  - web-board
  - feature-map
  - ux
  - frontend
  - backend
type: map
status: active
created: 2026-04-16
updated: 2026-04-16
links:
  - "[[Projects/STS/WebBoard_Alert_and_UX_Debugging_Playbook|WebBoard Alert and UX Debugging Playbook]]"
  - "[[Character Count Policy]]"
  - "[[Alert Architecture Map]]"
---

# Web Board Feature Map

Related
- [[Projects/STS/WebBoard_Alert_and_UX_Debugging_Playbook|WebBoard Alert and UX Debugging Playbook]]
- [[Character Count Policy]]
- [[Alert Architecture Map]]

---

# 1. Feature Areas

## 1.1 Post List
Responsibilities
- แสดงรายการกระทู้
- แสดง owner/profile
- แสดง preview ของข้อความ
- แสดง counts เช่น like/comment
- รองรับ search
- รองรับ navigation ไป detail

Key UX rules
- search ต้องคงอยู่เมื่อย้อนกลับจาก detail
- preview ต้องไม่ยาวจนหน้า list บวม
- mobile layout ต้องไม่แตก

## 1.2 Post Detail
Responsibilities
- แสดง title/content เต็ม
- ตัด preview ก่อนเมื่อเนื้อหายาวเกิน threshold
- รองรับ `อ่านเพิ่มเติม`
- แสดงไฟล์แนบ
- รองรับ like/comment/edit/delete

Key UX rules
- detail ต้องอ่านง่าย
- ฟอนต์ไม่ควรใหญ่เกินแบบเอกสาร
- action bar ต้องไม่แตกบนมือถือ

## 1.3 Composer (Create Post)
Responsibilities
- รับ title/content/files
- validate ก่อนส่ง
- รองรับ cancel/discard
- รองรับ confirm ก่อน submit

Key UX rules
- modal ต้อง scroll ได้
- attachment เยอะก็ยังต้องกดโพสต์ได้
- counter ต้องเข้าใจง่าย

## 1.4 Edit Post
Responsibilities
- แก้ title/content
- เพิ่มไฟล์ใหม่
- mark ลบไฟล์เดิม
- save changes

Key UX rules
- ต้องแยก existing/new/removed attachments
- ต้องมี confirm เมื่อ action กระทบข้อมูลสำคัญ

## 1.5 Comment System
Responsibilities
- create comment
- edit comment
- delete comment
- upload comment attachments

Key UX rules
- limit และ counter ต้องชัด
- action สำคัญต้อง confirm ตามความเหมาะสม

## 1.6 Notification Integration
Responsibilities
- comment/post บาง action ต้อง trigger alert
- portal ต้อง parse และ route มาถูก

See: [[Alert Architecture Map]]

---

# 2. Cross-Cutting Concerns

## 2.1 Character Count
See: [[Character Count Policy]]

กระทบพื้นที่ต่อไปนี้
- title
- post content
- comment content
- preview threshold
- backend validation

## 2.2 Attachment Handling
ต้องคิดทุกจุดเสมอ
- append vs replace
- max file size
- invalid file cleanup
- delete old attachments
- mobile preview layout

## 2.3 Search State Persistence
ต้องลง URL เมื่อ user คาดว่าจะย้อนกลับมาเจอ state เดิม

## 2.4 Mobile Readiness
ทุก feature area ต้องถามว่า
- จอแคบแตกไหม
- ปุ่มกดได้จริงไหม
- modal scroll ได้ไหม
- attachment ดัน layout ไหม

---

# 3. Debug Entry Points

## 3.1 ถ้า list แสดงผิด
เช็ก
- API list response
- preview logic
- search query state
- attachment preview hydration

## 3.2 ถ้า detail แสดงผิด
เช็ก
- detail response
- preview threshold logic
- content truncation logic
- attachment rendering

## 3.3 ถ้า edit พัง
เช็ก
- post owner logic
- existing attachment state
- removed attachment state
- new files append logic

## 3.4 ถ้า comment พัง
เช็ก
- create vs edit validation
- attachment upload flow
- alert trigger side effects

---

# 4. Future Expansion Ideas
- moderation node
- tagging/filtering node
- pagination/performance node
- audit trail / activity history node
- websocket/realtime node
