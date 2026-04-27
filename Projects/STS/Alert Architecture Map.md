---
title: Alert Architecture Map
tags:
  - sts
  - alert
  - architecture
  - pbm
  - web-board
  - sla
type: architecture
status: active
created: 2026-04-16
updated: 2026-04-17
links:
  - "[[Projects/STS/WebBoard_Alert_and_UX_Debugging_Playbook|WebBoard Alert and UX Debugging Playbook]]"
  - "[[Projects/STS/STS - SLA Alert Implementation|STS - SLA Alert Implementation]]"
---

# Alert Architecture Map

Related:
- [[Projects/STS/STS - SLA Alert Implementation|STS - SLA Alert Implementation]]
- [[Projects/STS/STS - SLA Alert Troubleshooting Playbook|STS - SLA Alert Troubleshooting Playbook]]
- [[Projects/STS/WebBoard_Alert_and_UX_Debugging_Playbook|WebBoard Alert and UX Debugging Playbook]]

## Purpose
อธิบายเส้นทางของ alert ใน STS แบบสั้นแต่ใช้ debug ต่อได้จริง

---

# 1. Repos Involved
- `sts-portal`
- `STS-NOC`
- `STS-ALERT`
- `STS-ADM` สำหรับ config / email relation

---

# 2. High-Level Flow

## 2.1 WebBoard Comment Alert Flow
1. user ส่ง comment ใน Web Board
2. `STS-NOC` สร้าง comment
3. `STS-NOC` หา owner/recipient ที่เกี่ยวข้อง
4. `STS-NOC` ส่ง alert ไป `STS-ALERT`
5. `STS-ALERT` สร้าง row ใน queue/target tables
6. `sts-portal` poll unread alerts
7. user เห็น bell และ click เข้า post detail

## 2.2 SLA Alert Flow
1. `SLAMonitorWorker` scan CM + PM jobs
2. `CM` ใช้ `fCalcNetSLA`
3. `PM` ใช้ `fCalsNetSLAPM`
4. worker ประเมิน `WARN / OVER`
5. worker เรียก `spAlertInsert`
6. `STS-ALERT` เก็บ unread targets
7. `sts-portal` poll `JOB` unread by `UserId`
8. header แสดง `JOB bell` + `Snackbar`

---

# 3. Important Concepts

## 3.1 `UserId` vs `PersonId`
โจทย์นี้เจอชัดว่า 2 ค่าไม่ควรถูก assume ว่าเท่ากัน

### บทเรียน
- ถ้า unread/read flow ผูกกับ person-level data
- แต่ session หรือ frontend context ถือ user-level identity
- ต้องมี bridge ที่ชัดเจน

## 3.2 Action Format
ตัวอย่าง action
- `PBM|...`
- `WEBBOARD|COMMENT|{postId}`
- `WEBBOARD|POST|{postId}`
- `SLA-WARN|{JobNo}|{ServiceNo}|{Minutes}`
- `SLA-OVER|{JobNo}|{ServiceNo}|{Minutes}`

### บทเรียน
ถ้า format ใหม่ถูกเพิ่ม แต่ UI parse ไม่รู้จัก
- count อาจมี
- row อาจมี
- แต่ user จะยังมองว่า “ระบบไม่ทำงาน”

## 3.3 Bell versus Snackbar
- bell = backlog / persistent record
- snackbar = realtime awareness
- dedupe ควรมีเฉพาะ snackbar

---

# 4. Failure Points

## 4.1 Data Exists But UI Silent
เช็ก
- unread API คืนข้อมูลไหม
- UI แยก count/list state ไหม
- action parse ถูกไหม
- route ถูกไหม
- initial hydration กลืน new arrival ไหม

## 4.2 Wrong Service Target
เช็ก
- frontend env
- backend env
- local vs UAT
- DB ที่ดูตรงกับ DB ที่ service ใช้จริงไหม

## 4.3 Identity Mapping Bug
เช็ก
- request ใช้ `UserId` หรือ `PersonId`
- service ปลายทาง expect อะไร
- session มี field อะไรจริง

## 4.4 CM / PM Semantic Mismatch
เช็ก
- debug อยู่ฝั่ง `CM` หรือ `PM`
- over-SLA ใช้ field ไหน
- warning ใช้ field ไหน

---

# 5. Debug Sequence
- เช็ก `AlertQ`
- เช็ก `AlertTo`
- เช็ก unread function
- เช็ก unread endpoint
- เช็ก frontend polling
- เช็ก parse/route
- เช็ก runtime process ล่าสุด
- เช็ก env ว่าหน้าเว็บกับ service คุย DB เดียวกันหรือไม่

---

# 6. Notes for Future Refactor
- ควรมี alert contract note กลาง
- ควรมี mapping policy ระหว่าง `UserId` และ `PersonId`
- ควรมี action schema registry ถ้า alert types จะเพิ่มอีก
- ถ้าจะปิด mail SLA ให้จริง ต้องมี note ต่อเรื่อง `EmailTemplateR*` และ `isSendByEmail`
