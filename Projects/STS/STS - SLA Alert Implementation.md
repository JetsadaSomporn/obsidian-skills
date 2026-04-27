---
tags: [STS, alert, SLA, implementation, snackbar, local-test, verified]
created: 2026-04-12
updated: 2026-04-17
---

# STS — SLA Alert Implementation

Related:
- [[Projects/STS/STS - SLA Alert UAT Checklist|STS - SLA Alert UAT Checklist]]
- [[Projects/STS/STS - SLA Alert Bug Log|STS - SLA Alert Bug Log]]
- [[Projects/STS/STS - SLA Alert Troubleshooting Playbook|STS - SLA Alert Troubleshooting Playbook]]
- [[Projects/STS/Alert Architecture Map|Alert Architecture Map]]

## Purpose
บันทึก implementation จริงของ `SLA Alert` ที่ทำใน STS เพื่อให้รอบถัดไปกลับมาอ่านแล้วเข้าใจ flow, จุดเสี่ยง, และไม่ชน bug เดิมซ้ำ

## Scope
งานนี้ครอบคลุมเฉพาะ
- `SLA-WARN` และ `SLA-OVER`
- `JOB bell` ใน `sts-portal`
- `Snackbar` มุมขวาบนพร้อมเสียง
- local test flow สำหรับ `Teleport1`

ไม่รวม
- mobile push
- Line integration
- legacy email cleanup ทั้งระบบ

## Requirement summary
โจทย์ที่ตกลงกันแล้ว
- ระบบมี `2 bell`
  - `JOB`
  - `PBM`
- `SLA-WARN` และ `SLA-OVER` ต้อง
  - อยู่ใน `JOB bell`
  - ไม่อยู่ใน `PBM bell`
  - เด้งเป็น `Snackbar` มุมขวาบน
  - มีเสียงเตือน
  - มีปุ่ม `เปิดใบงาน`
  - มีปุ่ม `x` ปิดราย toast
- `JOB bell` เก็บ alert ตามจริงได้
- `Snackbar` ต้องไม่ spam งานเดิมซ้ำ

## Final behavior
พฤติกรรมสุดท้ายที่ต้องการและผ่าน local แล้ว

### Bell behavior
- `JOB bell` แสดง
  - portal job alerts
  - SLA alerts
  - WebBoard alerts
- `PBM bell` แสดงเฉพาะ `PBM`

### Snackbar behavior
- ตำแหน่ง `top-right`
- มีเสียงเตือน
- มีปุ่ม `เปิดใบงาน`
- มีปุ่ม `x`
- คลิกพื้นที่อื่นในหน้าแล้ว dismiss ทั้งก้อนได้

### Dedupe rules
ใช้กับ `Snackbar` เท่านั้น
- `1 toast ต่อ jobId ต่อ status`
- `SLA-WARN` ของงานเดิมไม่เด้งซ้ำ
- `SLA-OVER` ของงานเดิมไม่เด้งซ้ำ
- ถ้างานเดิมเปลี่ยนจาก `WARN -> OVER` ให้เด้งได้อีก 1 ครั้ง
- unread เก่าที่มีอยู่ก่อน refresh ต้องไม่เด้งย้อนหลัง

## Current architecture

### 1. Backend producer
ตัว producer อยู่ที่ [SLAMonitorWorker.cs](/Users/jetsadasomporn/Vscode/STS/STS-NOC/Workers/SLAMonitorWorker.cs:16)

หน้าที่
- scan ทุก `5 นาที`
- คำนวณ `CM` กับ `PM` แยกกัน
- ถ้าเข้าเงื่อนไขใกล้เกินหรือเกิน SLA ให้ `CALL spAlertInsert`
- เขียน action เป็น
  - `SLA-WARN|JobNo|ServiceNo|Minutes`
  - `SLA-OVER|JobNo|ServiceNo|Minutes`

### 2. CM vs PM calculation paths
CM ใช้ [fCalcNetSLA.sql](/Users/jetsadasomporn/Vscode/STS/Proposition/fCalcNetSLA.sql:4)
- ใช้ `Job` + `JobActivity`
- รองรับ `SLAType` `B / C / P`
- คิด business hours, OT, hold, NOC hold, TP hold
- worker ใช้ `SlaPercent >= 80` หรือ `IsOverSla = TRUE`

PM ใช้ [fCalsNetSLAPM.sql](/Users/jetsadasomporn/Vscode/STS/Proposition/fCalsNetSLAPM.sql:3)
- ใช้ `JobPM` + `JobPMActivity`
- ใช้ wall-clock downtime เป็นหลัก
- hold ใช้ `LAG(ActivityStatusId)` แล้วมอง `4,10,11` เป็น hold
- worker ใช้ `SLAPercent >= 80` หรือ `IsDelay = TRUE`

สรุปสำคัญ
- `CM` กับ `PM` แยกคำนวณคนละฟังก์ชันจริง
- semantics ของ `CM` และ `PM` ไม่เหมือนกัน
- `PM` มีความเสี่ยงที่ `% SLA` กับ `IsDelay` จะไม่สัมพันธ์กัน 100% ถ้า `JobFixDate` กับ `SLA ชั่วโมง` ไม่ represent deadline เดียวกัน

### 3. Alert storage / unread API
ฝั่ง unread อยู่ที่ [AlertRepository.cs](/Users/jetsadasomporn/Vscode/STS/STS-ALERT/Repositories/AlertRepository.cs:74)

paths ที่ใช้จริง
- generic person-based
  - `GetUnread(PersonId)`
- additive user-based
  - `GetUnreadPBMWebBoardByUserId(UserId)` ที่ [AlertRepository.cs](/Users/jetsadasomporn/Vscode/STS/STS-ALERT/Repositories/AlertRepository.cs:96)
  - `GetUnreadJobByUserId(UserId)` ที่ [AlertRepository.cs](/Users/jetsadasomporn/Vscode/STS/STS-ALERT/Repositories/AlertRepository.cs:126)

แนวคิดสำคัญ
- resolve `UserId -> PersonId` ใน backend
- ไม่พึ่ง `personId` จาก session ฝั่งหน้าโดยตรงสำหรับ `JOB/SLA`

### 4. Frontend consumer
จุดหลักอยู่ที่ [header.tsx](/Users/jetsadasomporn/Vscode/STS/sts-portal/components/_layout/main/header.tsx:494)

หน้าที่หลัก
- parse action เป็น `PBM`, `WEBBOARD`, `SLA-WARN`, `SLA-OVER`
- route ตามประเภท alert
- poll `JOB` ผ่าน `getUnreadJobByUserId`
- poll `PBM/WebBoard` ผ่าน `getUnreadPBMWebBoardByUserId`
- แสดง `Snackbar`
- ดูแล dedupe และ dismiss behavior

## Files changed

### Frontend
- [header.tsx](/Users/jetsadasomporn/Vscode/STS/sts-portal/components/_layout/main/header.tsx:494)
- [CommonUIProvider.tsx](/Users/jetsadasomporn/Vscode/STS/sts-portal/providers/base/CommonUIProvider.tsx:76)
- [sonner.tsx](/Users/jetsadasomporn/Vscode/STS/sts-portal/components/npxshadcn/sonner.tsx:12)
- [alert.service.ts](/Users/jetsadasomporn/Vscode/STS/sts-portal/services/alert/alert.service.ts:95)
- [alert.api.ts](/Users/jetsadasomporn/Vscode/STS/sts-portal/lib/api/alert/alert.api.ts:67)

### Backend / Alert API
- [SLAMonitorWorker.cs](/Users/jetsadasomporn/Vscode/STS/STS-NOC/Workers/SLAMonitorWorker.cs:16)
- [AlertRepository.cs](/Users/jetsadasomporn/Vscode/STS/STS-ALERT/Repositories/AlertRepository.cs:96)
- [AlertController.cs](/Users/jetsadasomporn/Vscode/STS/STS-ALERT/Controllers/AlertController.cs:179)

## Important design decisions

### 1. SLA belongs to JOB bell only
เหตุผล
- SLA เป็น operational alert ระดับ job
- `PBM bell` ควรสงวนไว้สำหรับ `PBM` โดยตรง

### 2. Keep both Snackbar and JOB bell
เหตุผล
- `Snackbar` = realtime awareness
- `JOB bell` = backlog / persistent record

### 3. Dedupe only on Snackbar
เหตุผล
- bell ใช้เป็นประวัติการแจ้งเตือน
- snackbar ใช้ดึงความสนใจ ไม่ควร spam

### 4. Use UserId-based unread for JOB/SLA
เหตุผล
- ปัญหา `UserId != PersonId` เคยทำให้ DB มี alert แต่หน้าเว็บไม่เห็น
- pattern ที่ใช้ได้จริงแล้วใน `PBM/WebBoard` คือให้ backend เป็นคน map `UserId -> PersonId`

## Bugs found and fixes
สรุปย่อ ดูละเอียดต่อที่ [[Projects/STS/STS - SLA Alert Bug Log|STS - SLA Alert Bug Log]]

1. `UserId != PersonId`
- อาการ: alert มีใน DB แต่หน้าเงียบ
- แก้: เพิ่ม `get-unread-job-by-userid`

2. local target ผิด
- อาการ: ยิงเข้า DB/API คนละชุดกับหน้าเว็บที่เปิด
- แก้: ตรวจ env ทุกครั้ง ไม่ assume ว่า local ชี้ localhost จริง

3. refresh แล้ว snackbar เด้ง unread เก่าซ้ำ
- อาการ: reload หน้าแล้ว toast เก่ากลับมา
- แก้: hydrate unread ชุดแรกเป็น baseline

4. bell ขึ้น แต่ snackbar ไม่ขึ้น
- อาการ: count/list มา แต่ toast เงียบ
- แก้: ย้าย diff logic ไปอยู่ใน poll cycle แทน `useEffect` หลัง render

5. toast ซ้อนกันแล้ว dismiss ไม่หมด
- อาการ: click-away หายแค่ตัวแรก
- แก้: เก็บ active toast ids แล้ว dismiss ทั้งชุด

6. same job same status spam
- อาการ: insert record ใหม่ของงานเดิมแล้ว toast ซ้ำ
- แก้: dedupe key = `status + jobId`

## Email relation
ดูละเอียดต่อที่ [[Projects/STS/STS - SLA Alert Troubleshooting Playbook|STS - SLA Alert Troubleshooting Playbook]]

สรุปสั้น
- worker SLA ปัจจุบัน `ไม่ได้ส่ง email ตรง`
- จุดส่งเมลจริงที่เจออยู่ในระบบคือ
  - [STS-NOC/Repositories/EmailRepository.cs](/Users/jetsadasomporn/Vscode/STS/STS-NOC/Repositories/EmailRepository.cs:164)
  - [STS-ADM/Repositories/EmailRepository.cs](/Users/jetsadasomporn/Vscode/STS/STS-ADM/Repositories/EmailRepository.cs:549)
- จุดที่เกี่ยวกับ alert email ในเชิง config/schema คือ
  - `isSendByEmail` ใน [SetAlertToFromRepository.cs](/Users/jetsadasomporn/Vscode/STS/STS-ADM/Repositories/SetAlertToFromRepository.cs:97)
  - `EmailTemplateR1Id..R4Id` ใน [AlertQ.sql](/Users/jetsadasomporn/Vscode/STS/STS-ALERT/database/table/AlertQ.sql:91)
- ถ้าจะยกเลิก mail SLA ให้สะอาด ต้องไล่ต่อว่า DB/SP หยิบ `EmailTemplateR*` ไปใช้ที่ไหน

## Local test setup
### Services
- `STS-ALERT`
- `STS-NOC`
- `sts-portal`

### Test user
- username: `Teleport1`
- `userId = 937`
- `PersonId = 941`

### Insert method
ใช้ `spAlertInsert` เข้าตรง local DB เพื่อทดสอบ UI ได้ทันที โดยไม่ต้องรอ SLA จริง

## Acceptance criteria
ถือว่าผ่านเมื่อ
- `SLA-WARN` และ `SLA-OVER` เข้า `JOB bell`
- ไม่เข้า `PBM bell`
- มี snackbar มุมขวาบน
- มีเสียง
- same job same status ไม่เด้งซ้ำ
- `WARN -> OVER` เด้งได้
- refresh แล้ว unread เก่าไม่เด้งย้อนหลัง
- route จาก alert ไปหน้า job ถูก

## Out of scope
- push notification มือถือ
- Line
- cleanup legacy email ทั้งระบบ
- redesign worker calculation rules
