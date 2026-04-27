---
tags: [STS, alert, SLA, troubleshooting, playbook]
created: 2026-04-17
updated: 2026-04-17
---

# STS — SLA Alert Troubleshooting Playbook

Related:
- [[Projects/STS/STS - SLA Alert Implementation|STS - SLA Alert Implementation]]
- [[Projects/STS/STS - SLA Alert Bug Log|STS - SLA Alert Bug Log]]
- [[Projects/STS/STS - SLA Alert UAT Checklist|STS - SLA Alert UAT Checklist]]

## Purpose
playbook นี้ใช้สำหรับ debug `SLA Alert` แบบเร็ว โดยเฉพาะเวลามีอาการว่า bell ไม่ขึ้น, snackbar ไม่เด้ง, หรือสงสัยว่า CM/PM คำนวณคนละทาง

## Step 1 — identify the alert type
ตอบก่อนว่า debug เรื่องไหน
- `CM SLA`
- `PM SLA`
- `JOB bell unread`
- `Snackbar`
- `Email relation`

## Step 2 — confirm calculation path
### CM
- function: [fCalcNetSLA.sql](/Users/jetsadasomporn/Vscode/STS/Proposition/fCalcNetSLA.sql:4)
- worker call: [SLAMonitorWorker.cs](/Users/jetsadasomporn/Vscode/STS/STS-NOC/Workers/SLAMonitorWorker.cs:87)
- warn rule: `SlaPercent >= 80`
- over rule: `IsOverSla = TRUE`

### PM
- function: [fCalsNetSLAPM.sql](/Users/jetsadasomporn/Vscode/STS/Proposition/fCalsNetSLAPM.sql:3)
- worker call: [SLAMonitorWorker.cs](/Users/jetsadasomporn/Vscode/STS/STS-NOC/Workers/SLAMonitorWorker.cs:119)
- warn rule: `SLAPercent >= 80`
- over rule: `IsDelay = TRUE`

## Step 3 — confirm producer
ดู worker ที่ [SLAMonitorWorker.cs](/Users/jetsadasomporn/Vscode/STS/STS-NOC/Workers/SLAMonitorWorker.cs:58)
เช็ก
- service รันอยู่ไหม
- scan รอบล่าสุดทำงานไหม
- job เข้าเงื่อนไขหรือไม่
- recipient ว่างหรือเปล่า
- cooldown 30 นาทีชนหรือไม่

## Step 4 — confirm storage
ถ้าใช้ test insert ตรง ให้พิสูจน์
- มี row ใน `AlertQ`
- มี row ใน `AlertTo`
- `fAlertGetUnreadCount(PersonId)` คืนรายการนั้นไหม

## Step 5 — confirm unread API path
### JOB / SLA
- frontend service: [alert.service.ts](/Users/jetsadasomporn/Vscode/STS/sts-portal/services/alert/alert.service.ts:102)
- API path: [alert.api.ts](/Users/jetsadasomporn/Vscode/STS/sts-portal/lib/api/alert/alert.api.ts:67)
- backend repo: [AlertRepository.cs](/Users/jetsadasomporn/Vscode/STS/STS-ALERT/Repositories/AlertRepository.cs:126)

### PBM / WebBoard
- frontend service: [alert.service.ts](/Users/jetsadasomporn/Vscode/STS/sts-portal/services/alert/alert.service.ts:95)
- backend repo: [AlertRepository.cs](/Users/jetsadasomporn/Vscode/STS/STS-ALERT/Repositories/AlertRepository.cs:96)

หลักสำคัญ
- `JOB/SLA` และ `PBM/WebBoard` ตอนนี้ใช้ additive user-based endpoints แยกกัน
- backend เป็นคน map `UserId -> PersonId`

## Step 6 — confirm frontend behavior
ดูที่ [header.tsx](/Users/jetsadasomporn/Vscode/STS/sts-portal/components/_layout/main/header.tsx:653)

เช็ก 4 จุด
1. parse action ถูกหรือไม่
2. route ถูกหรือไม่
3. job alerts ถูก poll เข้ามาหรือไม่
4. snackbar dedupe กลืนลูกใหม่หรือไม่

## Step 7 — confirm environment
ก่อน debug ยาว ให้ตอบ 3 ข้อนี้
- `sts-portal` ชี้ `NEXT_PUBLIC_API_ALERT_SERVICE` ไปที่ไหน
- `STS-ALERT` ที่รันอยู่คือ process ไหน / port ไหน
- DB ที่ insert test alert เข้าไป ตรงกับ DB ที่ service อ่านไหม

## Step 8 — email-specific check
ถ้าคำถามคือ `SLA mail ยังส่งอยู่ไหม`

ดู 3 ชั้น
### A. SMTP sender จริง
- [STS-NOC/Repositories/EmailRepository.cs](/Users/jetsadasomporn/Vscode/STS/STS-NOC/Repositories/EmailRepository.cs:164)
- [STS-ADM/Repositories/EmailRepository.cs](/Users/jetsadasomporn/Vscode/STS/STS-ADM/Repositories/EmailRepository.cs:549)

### B. Alert config
- [SetAlertToFromRepository.cs](/Users/jetsadasomporn/Vscode/STS/STS-ADM/Repositories/SetAlertToFromRepository.cs:97)
- field: `isSendByEmail`

### C. Alert schema/template
- [AlertQ.sql](/Users/jetsadasomporn/Vscode/STS/STS-ALERT/database/table/AlertQ.sql:91)
- fields: `EmailTemplateR1Id..R4Id`

ข้อสรุปปัจจุบัน
- worker SLA ไม่ได้เรียก SMTP sender ตรง
- แต่ระบบยังมี config/schema ฝั่ง email ค้างอยู่
- ถ้าจะปิด mail SLA ให้สะอาด ต้องไล่ต่อที่ DB/SP และ consumer ของ `EmailTemplateR*`

## Known good local test user
- username: `Teleport1`
- `userId = 937`
- `PersonId = 941`

## When to use direct insert
ใช้ `spAlertInsert` ตรงเมื่อ
- ต้องการทดสอบ snackbar / bell / route
- ไม่ต้องการรอ worker scan
- ยังไม่มี job SLA จริงสำหรับ reproduction

## When not to assume success
สิ่งที่ห้ามสรุปเร็วเกินไป
- DB มี alert = UI ต้องเห็นแล้ว
- `% SLA สูง` = over SLA เสมอ
- PM กับ CM คิดเหมือนกัน
- เปิด local UI = ชี้ local alert API แน่นอน
