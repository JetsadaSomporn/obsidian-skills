---
title: STS - Alert Flood Diagnosis Playbook
tags:
  - sts
  - alert
  - debugging
  - playbook
type: playbook
status: active
created: 2026-04-24
updated: 2026-04-24
links:
  - "[[Projects/STS/MOC - STS Workbench]]"
  - "[[Projects/STS/STS - SLA Alert Hardening Current Contract]]"
  - "[[Projects/STS/STS - Alert Routing Current State]]"
  - "[[Projects/STS/Alert Architecture Map]]"
---

# STS - Alert Flood Diagnosis Playbook

## What

Playbook สำหรับแยก alert flood ว่าเป็น duplicate bug, backlog burst, หรือ fan-out ปกติจากจำนวนผู้รับ

## Why It Matters

ถ้าวินิจฉัยผิด จะไปแก้ throughput/config ทั้งที่ root cause อาจเป็น dedup key หรือกลับกัน อาจไปทำ dedup เพิ่มทั้งที่ระบบแค่มีงานเข้าเงื่อนไขเยอะจริง

## Diagnosis Order

1. ดูก่อนว่า alert มาจาก job เดิมซ้ำหรือหลาย job
2. แยก event table ออกจาก recipient/log table
3. เช็ค `Action` หรือ label ว่าเปลี่ยนทุกรอบหรือ stable
4. เช็ค worker interval, startup delay, enabled config, และ cap ต่อ scan
5. เช็ค age filter ว่าตัด backlog เก่าออกจริงไหม
6. เช็คว่ามี code path อื่นยิง alert type เดียวกันหรือไม่

## Current STS Clues

- `AlertQ` คือ event
- `AlertTo` และ `AlertLogs` โตได้มากกว่า เพราะ fan-out ตามจำนวนผู้รับ
- `spAlertInsert` มี buffer/downstream flow จึงมีโอกาส race ถ้าไม่มี dedup ที่ดี
- `MaxAlertsPerScan` ไม่ได้จำกัดจำนวน row หลัง fan-out

## Common Failure

- Query นับ row จาก `AlertTo` แล้วคิดว่า event ถูกยิงเท่านั้นครั้ง
- เห็น volume สูงแล้วข้ามการแยก duplicate vs burst
- เช็ค local code แล้วไม่เช็ค server config/runtime
- ไม่ดู live DB ก่อนสรุป root cause

## Next Time

ถ้ามี incident แบบ "alert กำลังพัง server" ให้ทำ conservative throttle ก่อน แล้วค่อยแยก duplicate, burst, fan-out ด้วย query จาก live DB
