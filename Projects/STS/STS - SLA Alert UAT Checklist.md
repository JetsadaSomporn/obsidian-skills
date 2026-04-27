---
tags: [STS, alert, SLA, UAT, checklist, snackbar]
created: 2026-04-17
updated: 2026-04-17
---

# STS — SLA Alert UAT Checklist

> ใช้โน้ตนี้สำหรับทดสอบงาน `SLA alert` ฝั่ง `sts-portal` แบบเร็ว  
> ไม่ลงรายละเอียด implementation

---

## Scope

ทดสอบเฉพาะ:

- `JOB bell`
- `SLA snackbar`
- เสียงเตือน
- routing จาก alert ไปหน้าใบงาน
- พฤติกรรม dedupe ของ snackbar

ไม่รวม:

- mobile push
- Line
- email legacy cleanup

---

## Environment

ต้องเปิด service local ดังนี้

### 1. STS-ALERT

```bash
cd /Users/jetsadasomporn/Vscode/STS/STS-ALERT
dotnet run
```

ต้องฟังที่:

```text
http://localhost:5028
```

### 2. sts-portal

```bash
cd /Users/jetsadasomporn/Vscode/STS/sts-portal
npm run uat
```

เปิดที่:

```text
http://localhost:3000/app
```

### 3. STS-NOC

```bash
cd /Users/jetsadasomporn/Vscode/STS/STS-NOC
dotnet run
```

---

## Test user

ใช้ account นี้:

- username: `Teleport1`
- `userId = 937`
- `PersonId = 941`

---

## Current expected behavior

### Bell

มี `2 bell`

- `JOB`
- `PBM`

`SLA-WARN` และ `SLA-OVER` ต้องอยู่ใน:

- `JOB bell`

และต้องไม่ไปอยู่ใน:

- `PBM bell`

### Snackbar

`SLA-WARN` และ `SLA-OVER` ต้อง:

- เด้งที่มุมขวาบน
- มีเสียง
- มีปุ่ม `เปิดใบงาน`
- มีปุ่ม `x`

### Dedupe

กติกาของ `Snackbar`:

- `1 toast ต่อ jobId ต่อ status`
- `SLA-WARN` ของงานเดิมไม่เด้งซ้ำ
- `SLA-OVER` ของงานเดิมไม่เด้งซ้ำ
- ถ้า `WARN -> OVER` ของงานเดิม เด้งได้อีก 1 ครั้ง

กติกาของ `JOB bell`:

- เก็บ record ตามจริงได้

---

## Quick sanity check

1. login เป็น `Teleport1`
2. เปิดหน้า `http://localhost:3000/app/pm-service/monitor`
3. คลิกหน้า 1 ครั้งเพื่อปลดล็อกเสียง
4. ให้ระบบ insert SLA alert ใหม่

Expected:

- snackbar เด้ง
- มีเสียง
- `JOB bell` count เพิ่ม

---

## Test cases

### Case 1 — alert ใหม่

Expected:

- `Snackbar` เด้ง
- `JOB bell` มีรายการใหม่

### Case 2 — refresh

เงื่อนไข:

- มี unread เดิมค้างอยู่

Expected:

- unread ยังอยู่ใน `JOB bell`
- snackbar เก่าไม่เด้งซ้ำหลัง refresh

### Case 3 — same job same status

เงื่อนไข:

- insert `SLA-WARN` ของ job เดิมซ้ำ

Expected:

- `JOB bell` อาจมี record เพิ่ม
- `Snackbar` ไม่เด้งซ้ำ

### Case 4 — same job WARN -> OVER

เงื่อนไข:

- insert `SLA-WARN`
- จากนั้น insert `SLA-OVER` ของงานเดียวกัน

Expected:

- `Snackbar` เด้งอีกครั้งตอน `OVER`

### Case 5 — click-away dismiss

เงื่อนไข:

- ทำให้มี snackbar หลายตัวพร้อมกัน

Expected:

- เมื่อคลิกพื้นที่อื่นในหน้า snackbar หายทั้งก้อน

### Case 6 — route from alert

Expected:

- PM alert กดแล้วไป `/pm-service/job-detail/{JobPMId}`
- CM/NOC alert กดแล้วไป `/job-monitor/view/{JobId}`

---

## Manual insert examples

### PM

```sql
CALL "SCS0112"."spAlertInsert"(
  397,
  'High',
  109,
  '["941"]'::json,
  937,
  'SLA-OVER|PMNO260400065|PMOR260400065|109'
);
```

### PM warn

```sql
CALL "SCS0112"."spAlertInsert"(
  398,
  'Medium',
  96,
  '["941"]'::json,
  937,
  'SLA-WARN|PMNO260400066|PMOR260400066|96'
);
```

---

## Pass criteria

ถือว่าผ่านเมื่อครบทุกข้อ:

- alert เข้า `JOB bell`
- snackbar เด้งเฉพาะลูกใหม่
- refresh แล้วไม่เด้ง unread เก่าซ้ำ
- same job same status ไม่ spam snackbar
- `WARN -> OVER` เด้งได้
- dismiss snackbar ได้
- route จาก alert ถูกหน้า

---

## Related notes

- [[Projects/STS/STS - SLA Alert Implementation|STS - SLA Alert Implementation]]
- [[Projects/STS/Alert Architecture Map|Alert Architecture Map]]
