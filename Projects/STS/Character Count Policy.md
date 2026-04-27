---
title: Character Count Policy
tags:
  - sts
  - policy
  - text
  - thai-language
  - ux
type: policy
status: draft
created: 2026-04-16
updated: 2026-04-16
links:
  - "[[Projects/STS/WebBoard_Alert_and_UX_Debugging_Playbook|WebBoard Alert and UX Debugging Playbook]]"
  - "[[Web Board Feature Map]]"
---

# Character Count Policy

## Purpose
กำหนด policy กลางว่าระบบควรนับ “ข้อความ” อย่างไร เพื่อให้
- user เข้าใจง่าย
- frontend/backend ตรงกัน
- field แต่ละแบบมีข้อจำกัดที่ชัด

Related: [[Projects/STS/WebBoard_Alert_and_UX_Debugging_Playbook|WebBoard Alert and UX Debugging Playbook]]

---

# 1. Problem Statement
ภาษาไทยมีสระ วรรณยุกต์ และอักขระประกอบ ทำให้คำว่า “1 ตัวอักษร” ไม่ชัดถ้าไม่กำหนด metric

ปัญหาที่เจอในงานจริง
- `.length` ทำให้ user รู้สึกว่าถูกหัก quota เร็วเกินไป
- grapheme cluster ถูกต้องทาง Unicode มากขึ้น แต่ใน editor นี้กลับเข้มเกินจน user รู้สึกนับไม่ตรง
- ถ้า frontend กับ backend ใช้ metric คนละแบบ จะเกิดอาการผ่านหน้าจอแต่โดน reject ตอน submit

---

# 2. Metrics Considered

## 2.1 Code Unit / `.length`
### ข้อดี
- ง่าย
- ใช้งานตรงกับ `maxLength`

### ข้อเสีย
- ภาษาไทยเพี้ยนจากสิ่งที่ user รู้สึก
- วรรณยุกต์/เครื่องหมายประกอบกิน quota เต็ม

## 2.2 Grapheme Cluster
### ข้อดี
- ใกล้ “สิ่งที่ตาเห็น” ตาม Unicode มากขึ้น

### ข้อเสีย
- บาง pattern ภาษาไทยถูกจับรวมเป็นกลุ่มแบบที่ user ไม่คาด
- ตัวเลข counter อาจลดฮวบเกินไปจน user งง

## 2.3 Thai-friendly Count
### แนวคิด
- นับตัวฐาน
- ไม่นับ combining marks เป็น quota หลัก

### ข้อดี
- ใกล้ความรู้สึกคนไทยมากกว่า
- practical กว่าสำหรับ editor งานนี้

### ข้อเสีย
- ไม่ใช่ Unicode-pure metric
- ต้องประกาศ policy ให้ชัด

---

# 3. Recommended Policy for STS Web Board

## 3.1 Policy Choice
สำหรับ Web Board ให้ใช้ metric ที่ predictable และ user เข้าใจง่ายก่อน

## 3.2 Field Limits
- Title: `150`
- Post content: `1500`
- Comment: `500`

## 3.3 Enforcement Rule
- frontend และ backend ต้องใช้ policy เดียวกัน
- counter ที่แสดงบน UI ต้องใช้ metric เดียวกับ validation
- preview/truncation rule ต้องระบุแยกจาก count policy

---

# 4. Preview Policy
count policy และ preview policy เป็นคนละเรื่อง

## 4.1 Current Web Board Preview Rule
- post detail แสดง preview ก่อนที่ `400`
- ถ้าเกินให้ขึ้น `อ่านเพิ่มเติม`

## 4.2 Why Separate It
- count policy = สิทธิ์ที่ user พิมพ์ได้
- preview policy = วิธีแสดงผลให้หน้าอ่านง่าย

---

# 5. Validation Design Principles
- ห้ามให้ frontend และ backend คนละ metric
- ห้ามให้ counter กับ submit validation คนละ metric
- ห้ามเปลี่ยน policy แบบเงียบๆ โดยไม่มี note/reference

---

# 6. Reuse Questions
ก่อนกำหนด count policy ให้ field ใหม่ ถามก่อนเสมอ
- field นี้เป็น free text หรือ structured text
- user คาดหวังอะไรจากคำว่า “1 ตัวอักษร”
- field นี้เน้นภาษาไทยมากแค่ไหน
- มี preview/truncation ไหม
- ต้อง sync policy ข้ามกี่ repo

---

# 7. Future Nodes
- [[Projects/STS/WebBoard_Alert_and_UX_Debugging_Playbook|WebBoard Alert and UX Debugging Playbook]]
- [[Web Board Feature Map]]
