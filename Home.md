---
title: Home
aliases:
  - Start Here
tags:
  - moc
  - home
  - second-brain
type: dashboard
status: active
created: 2026-04-11
updated: 2026-04-24
links:
  - "[[Inbox/Inbox]]"
  - "[[Journal/Journal - Overview]]"
  - "[[Projects/MOC - Projects Overview]]"
  - "[[Areas/MOC - Areas Overview]]"
  - "[[Tech/MOC - Tech Overview]]"
  - "[[Projects/STS/MOC - STS Workbench]]"
  - "[[Concepts/Obsidian Node Growth Workflow]]"
---

# Home

> หน้าเริ่มต้นของ vault นี้สำหรับงาน, ความคิด, การตัดสินใจ, และความรู้ที่ต้องใช้ซ้ำ

## Start Here
- [[Inbox/Inbox|Inbox]]
- [[Journal/Journal - Overview|Journal]]
- [[Projects/MOC - Projects Overview|Projects]]
- [[Areas/MOC - Areas Overview|Areas]]
- [[Tech/MOC - Tech Overview|Tech Library]]
- [[Identity/About Me|About Me]]
- [[Identity/Goals|Goals]]
- [[Identity/Now Next Later|Now / Next / Later]]
- [[Identity/Operating Principles|Operating Principles]]

## Current Focus
- [[Journal/Daily/2026-04-23]]
- [[Projects/STS/MOC - STS Workbench|STS Workbench]]
- [[Projects/STS/STS - Project Overview|STS]]
- [[Projects/STS/STS - SLA Alert Hardening Current Contract|SLA Alert Current Contract]]
- [[Projects/STS/STS - Alert Flood Diagnosis Playbook|Alert Flood Diagnosis]]
- [[Concepts/Obsidian Node Growth Workflow|Node Growth Workflow]]

## Active Projects
```dataview
TABLE file.link AS Project, status AS Status, updated AS Updated
FROM "Projects"
WHERE file.name != "MOC - Projects Overview"
SORT updated DESC
```

## Recent STS Notes
```dataview
TABLE file.link AS Note, type AS Type, updated AS Updated
FROM "Projects/STS"
SORT updated DESC
LIMIT 15
```

## Tech Library Snapshot
```dataview
TABLE file.link AS Note, type AS Type, updated AS Updated
FROM "Tech"
WHERE file.name != "MOC - Tech Overview"
SORT updated DESC
LIMIT 12
```

## Workflow
1. จับของดิบหรือไอเดียไว้ที่ [[Inbox/Inbox]]
2. ถ้าเป็นงานที่มี outcome ชัด ให้ย้ายไป [[Projects/MOC - Projects Overview|Projects]]
3. ถ้าเป็นเรื่องที่ต้องดูแลต่อเนื่อง ให้ย้ายไป [[Areas/MOC - Areas Overview|Areas]]
4. ถ้าเป็น pattern หรือความรู้ที่ reuse ได้ ให้ย้ายไป [[Tech/MOC - Tech Overview|Tech Library]]
5. ถ้าเป็นสิ่งที่เกิดขึ้นวันนี้ ให้ทิ้งร่องรอยไว้ใน [[Journal/Journal - Overview|Journal]]
6. ถ้า note เริ่มเป็น knowledge ใช้ซ้ำ ให้แตกตาม [[Concepts/Obsidian Node Growth Workflow|Node Growth Workflow]]

> [!tip]
> Vault นี้ไม่ต้องบันทึกทุกอย่าง
> เก็บเฉพาะสิ่งที่ควรจำ, ควรใช้ซ้ำ, หรือมีผลต่อการตัดสินใจในอนาคต

## Vault Hygiene Notes
- ใช้ `Projects/` สำหรับงานที่มี outcome ชัด
- ใช้ `Tech/` สำหรับ pattern ที่ใช้ซ้ำได้
- ย้าย note STS ทั้งหมดไว้ใต้ `Projects/STS/`
- เคลียร์ไฟล์ `Untitled*` ทันทีเมื่อยังไม่มีเนื้อหา
