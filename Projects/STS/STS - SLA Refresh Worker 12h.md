---
tags: [STS, SLA, worker, backend, pending]
created: 2026-04-21
updated: 2026-04-21
status: pending
type: working-note
---

# STS - SLA Refresh Worker 12h

Related:
- [[Projects/STS/STS - SLA Calculation|STS - SLA Calculation]]
- [[Projects/STS/STS - SLA Alert Once-Per-Job Plan|STS - SLA Alert Once-Per-Job Plan]]

## Goal
ให้ระบบ refresh ค่า SLA ของงาน `CM/NOC` แบบ background โดย
- ตัดงานสถานะ `5`, `7`, `13` ออก
- อัปเดตงานที่เหลือทั้งหมด
- รันทุก `12 ชั่วโมง`

โจทย์นี้ตั้งใจทำแบบไม่ overengineer
- ไม่แก้ frontend
- ไม่เพิ่ม dirty flag ใน DB รอบแรก
- ไม่เปลี่ยน flow ตอน user กด save

## Requirement ล่าสุด
logic ที่ตกลงล่าสุดคือ
- `5` = `Job Closed`
- `7` = `Closed Complete`
- `13` = `ยังไม่สามารถปิดงานได้`

สามสถานะนี้ไม่ต้องเอามาคำนวณใน background refresh

## What changed in code

### 1. เพิ่ม worker ใหม่
เพิ่มไฟล์:
- `STS-NOC/Workers/SlaRefreshWorker.cs`

หน้าที่:
- รอ initial delay
- รันทุก `12 ชั่วโมง`
- query หา `JobId` จาก `SCS0112.Job`
- ใช้เงื่อนไข `JobStatusId NOT IN (5, 7, 13)`
- ประมวลผลเป็น batch
- ต่อ 1 งาน เรียก
  - `fCalcNetSLA(jobId)`
  - `fJobSlaDowntimeUpd(...)`

### 2. กันหลาย instance ยิงพร้อมกัน
worker ใช้ PostgreSQL advisory lock

เป้าหมาย:
- ถ้ามี `STS-NOC` หลาย instance
- ให้มีแค่ instance เดียวที่ได้รอบ refresh นั้น

### 3. เพิ่ม options class
เพิ่มไฟล์:
- `STS-NOC/Models/SlaRefreshOptions.cs`

มีค่าหลัก:
- `Enabled`
- `IntervalHours`
- `BatchSize`
- `InitialDelaySeconds`
- `AdvisoryLockKey`

### 4. register worker ใน Program
แก้:
- `STS-NOC/Program.cs`

เพิ่ม:
- `Configure<SlaRefreshOptions>(...)`
- `AddHostedService<SlaRefreshWorker>()`

### 5. เพิ่ม config default ใน appsettings
แก้:
- `STS-NOC/appsettings.json`
- `STS-NOC/appsettings.Development.json`

ค่าที่ใส่ตอนนี้:
- `Enabled = true`
- `IntervalHours = 12`
- `BatchSize = 200`
- `InitialDelaySeconds = 60`
- `AdvisoryLockKey = 2026042101`

หมายเหตุ:
- ค่าเดียวกันนี้มี default อยู่ใน `SlaRefreshOptions.cs`
- ถ้าจะลบ section `SlaRefreshOptions` ออกจาก `appsettings` ก็ยังรันได้จาก code default

## Query / flow ที่ worker ใช้

### คัดงาน
```sql
SELECT "JobId"
FROM "SCS0112"."Job"
WHERE "JobStatusId" NOT IN (5, 7, 13)
```

### refresh ต่อ 1 งาน
```sql
SELECT * FROM "SCS0112"."fCalcNetSLA"(@JobId)
SELECT "SCS0112"."fJobSlaDowntimeUpd"(...)
```

## Touched files
- `STS-NOC/Workers/SlaRefreshWorker.cs`
- `STS-NOC/Models/SlaRefreshOptions.cs`
- `STS-NOC/Program.cs`
- `STS-NOC/appsettings.json`
- `STS-NOC/appsettings.Development.json`

## Verification
ตรวจแล้ว:
- `dotnet build STS-NOC/NOC.csproj` ผ่าน
- `0 Error`

ยังไม่ได้ตรวจ:
- รัน worker กับ DB จริง
- จำนวนงานจริงใน production ว่ารอบละกี่ job
- เวลารันจริงต่อ batch
- ผลกระทบต่อ DB load

## Risks / next checks
- ตอนนี้เป็น `full active-job refresh` ทุก 12 ชั่วโมง ไม่ได้คัดเฉพาะงาน dirty
- ถ้างาน active เยอะมาก อาจต้องลด `BatchSize`
- ยังไม่ได้ confirm ว่าควร include สถานะ `8` หรือไม่ เพราะ requirement ล่าสุดตัดแค่ `5,7,13`
- ถ้าจะเอา config ออกจาก `appsettings` ให้ลบได้ แต่ code ตอนนี้ยังรองรับอยู่ทั้งสองแบบ

## Current verdict
logic ตอนนี้ตรง requirement ล่าสุดของหัวหน้า:
- exclude แค่ `5,7,13`
- งานที่เหลือ refresh หมด
- refresh ทุก `12 ชั่วโมง`

แต่สถานะยังเป็น `pending`
เพราะ compile ผ่านแล้วอย่างเดียว ยังไม่ใช่ `verified on runtime`
