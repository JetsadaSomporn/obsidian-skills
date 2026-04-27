---
title: PostgreSQL - Function Overload Verification
tags:
  - tech
  - postgresql
  - debugging
  - atomic
type: atomic
status: active
created: 2026-04-24
updated: 2026-04-24
links:
  - "[[Tech/MOC - Tech Overview]]"
  - "[[Tech/PostgreSQL Patterns]]"
  - "[[Projects/STS/STS - SLA Downtime Calculation Contract]]"
---

# PostgreSQL - Function Overload Verification

## What

PostgreSQL function ชื่อเดียวกันมีหลาย signature ได้ การแก้ไฟล์ SQL ที่ชื่อถูกจึงยังอาจแก้ผิด overload

## Why It Matters

ในงาน STS SLA เคยต้องแยกว่า caller ใช้ `fJobSlaDowntimeUpd` overload กี่ parameters ถ้า deploy ผิด signature ระบบจะยังเรียก behavior เก่าอยู่

## Check Query

```sql
SELECT
  p.oid,
  n.nspname AS schema_name,
  p.proname AS function_name,
  pg_get_function_identity_arguments(p.oid) AS args
FROM pg_proc p
JOIN pg_namespace n ON n.oid = p.pronamespace
WHERE p.proname = 'function_name_here'
ORDER BY p.proname, args;
```

ดู definition:

```sql
SELECT pg_get_functiondef(p.oid)
FROM pg_proc p
JOIN pg_namespace n ON n.oid = p.pronamespace
WHERE p.proname = 'function_name_here';
```

## Common Failure

- ใช้ `DROP FUNCTION name` โดยไม่ระบุ argument types
- แก้ 14-param function แต่ application เรียก 16-param function
- เช็คแค่ชื่อไฟล์ใน repo แล้วคิดว่าตรงกับ DB runtime
- ลืมว่า schema search path อาจทำให้เรียกคนละ schema

## Next Time

ถ้าเจอ error `function ... is not unique` หรือ behavior ไม่เปลี่ยนหลัง deploy SQL ให้เริ่มจาก `pg_proc` ก่อน ไม่ต้องเดาจาก filename
