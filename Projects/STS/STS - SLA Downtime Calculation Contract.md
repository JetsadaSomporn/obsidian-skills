---
title: STS - SLA Downtime Calculation Contract
tags:
  - sts
  - sla
  - postgresql
  - contract
type: contract
status: active
created: 2026-04-24
updated: 2026-04-24
links:
  - "[[Projects/STS/MOC - STS Workbench]]"
  - "[[Projects/STS/STS - SLA Calculation]]"
  - "[[Tech/PostgreSQL Patterns]]"
  - "[[Tech/Atomic/PostgreSQL - Function Overload Verification]]"
---

# STS - SLA Downtime Calculation Contract

## What

Contract สำหรับ SLA downtime ฝั่ง STS: คำนวณเวลาเป็น seconds, format เป็น `hh:mm:ss`, และแยก calculator ออกจาก write-back path

## Why It Matters

SLA เพี้ยนได้จาก 2 จุดใหญ่: ปัดเวลาผิดระดับนาที และหัก hold ผิดชนิดจน SLA ดูดีปลอม

## Current Rule / Contract

- downtime internal value ควรเก็บเป็น seconds
- display/string field ใช้ `hh:mm:ss`
- `...Int` field เก็บ seconds
- `IsOverSLA` และ `SLAPercent` ต้อง derive จากผลคำนวณ ไม่ใช่เดาจาก `StatusReason`
- hold ที่ deduct ได้คือ `HoldType IS NULL OR HoldType = 0`
- `Hold by customer` ห้ามทำให้ SLA ดูดีปลอม
- `fCalcNetSLA` ควรเป็น calculator
- `fJobSlaDowntimeUpd` คือ write-back path ที่ต้องเช็ค signature ให้ตรงกับ caller

## Evidence

- [[Projects/STS/STS - SLA Calculation]]
- [[Journal/Daily/2026-04-23]]
- code path ที่ควร reopen: `STS-NOC/Repositories/JobRepository.cs`
- DB check ที่ควรใช้: `pg_proc`, `pg_get_functiondef`

## Common Failure

- แก้ function ชื่อถูกแต่ overload ผิด
- เชื่อไฟล์ SQL ใน repo โดยไม่เช็ค deployed function
- ปัด duration เป็นนาทีแล้วทำให้ SLA threshold เพี้ยน
- เอา hold ทุกประเภทมาหักรวมกัน

## Next Time

ก่อน deploy SQL รอบใหม่ ให้ยืนยัน 3 อย่าง:

1. NOC caller ยังเรียก `fJobSlaDowntimeUpd` signature เดิมหรือไม่
2. deployed PostgreSQL function definition ตรงกับไฟล์ที่จะ deploy หรือไม่
3. sample job ได้ downtime seconds และ `hh:mm:ss` ตรงกันหรือไม่
