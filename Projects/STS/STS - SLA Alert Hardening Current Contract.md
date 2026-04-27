---
title: STS - SLA Alert Hardening Current Contract
tags:
  - sts
  - sla
  - alert
  - contract
type: contract
status: active
created: 2026-04-24
updated: 2026-04-24
links:
  - "[[Projects/STS/MOC - STS Workbench]]"
  - "[[Projects/STS/STS - SLA Alert Noisy Alert Fix 2026-04-23]]"
  - "[[Projects/STS/STS - SLA Alert Implementation]]"
  - "[[Projects/STS/STS - Alert Routing Current State]]"
---

# STS - SLA Alert Hardening Current Contract

## What

Current contract ของ SLA alert หลัง hardening คือ worker สแกนงานที่เข้า threshold แล้วส่ง alert แบบจำกัดรอบและกันซ้ำระดับ job + label

## Why It Matters

ถ้า AI ตัวถัดไปจำ contract ผิด จะสรุป flood เป็น duplicate storm ง่ายเกินไป ทั้งที่บางเคสเป็น backlog burst จากงานจริงจำนวนมาก

## Current Rule / Contract

- `SLA-WARN`: แจ้งเมื่อ SLA เข้า warning threshold
- `SLA-OVER`: แจ้งเมื่อ SLA เกิน 100%
- key กันซ้ำควร stable เช่น `CM:123:SLA-WARN` หรือ `PM:456:SLA-OVER`
- 1 job ควรแจ้งสูงสุด 2 event หลักตลอดอายุงาน: WARN 1 + OVER 1
- worker ต้องมี age filter เพื่อไม่สแกน PM เก่าไม่จำกัด
- `MaxAlertsPerScan` คุมจำนวนงานต่อรอบ ไม่ใช่จำนวนผู้รับหลัง fan-out
- config ฝั่ง runtime ต้องเปิดใช้จริง ไม่งั้น logic ใน code ไม่ได้แปลว่า production ทำงานแล้ว

## Evidence

- [[Projects/STS/STS - SLA Alert Noisy Alert Fix 2026-04-23]]
- [[Journal/Daily/2026-04-23]]
- code path ที่ควร reopen: `STS-NOC/Workers/SLAMonitorWorker.cs`

## Common Failure

- dedup ที่อิง `Action` text ตรงๆ พังง่าย เพราะข้อความเปลี่ยนตามเวลาที่เหลือ/เกิน
- dedup ที่เช็คแค่ `Unread` ทำให้ user อ่านแล้ว event เดิมยิงใหม่ได้
- เห็น alert เยอะแล้วสรุปว่า duplicate ทันที ทั้งที่อาจเป็น burst จากหลาย job จริง
- ลืมแยก event count ออกจาก recipient fan-out count

## Next Time

ถ้ากลับมา debug SLA alert ให้เริ่มจากแยก 3 คำถาม:

1. เป็น duplicate ของ job เดิมหรือเป็น burst จากหลาย job
2. worker/config ใน server เปิด logic ชุดเดียวกับ local หรือไม่
3. DB downstream fan-out ทำให้จำนวน row โตจากจำนวนผู้รับหรือไม่
