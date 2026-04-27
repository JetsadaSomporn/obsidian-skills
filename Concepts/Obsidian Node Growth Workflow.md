---
title: Obsidian Node Growth Workflow
tags:
  - concept
  - obsidian
  - second-brain
  - workflow
type: workflow
status: active
created: 2026-04-24
updated: 2026-04-24
links:
  - "[[Home]]"
  - "[[Journal/Journal - Overview]]"
  - "[[Projects/STS/MOC - STS Workbench]]"
  - "[[Tech/MOC - Tech Overview]]"
---

# Obsidian Node Growth Workflow

> เป้าหมายคือเพิ่ม node ที่ทำให้คิดเร็วขึ้น ไม่ใช่เพิ่มไฟล์ให้ graph ดูแน่นเฉยๆ

## Rule

- Daily note ใช้ capture ว่าวันนี้เกิดอะไรและ link ไป node ที่เกี่ยวข้อง
- Project note ใช้เก็บ context ที่ผูกกับงานเดียว เช่น STS, Trader, Stock Manager
- Tech note ใช้เก็บ pattern ที่ใช้ซ้ำได้หลายงาน
- Area note ใช้เก็บเรื่องที่ต้องดูแลต่อเนื่อง ไม่มี finish line ชัด
- Identity note ใช้เก็บตัวตน, เป้าหมาย, operating principles

## When To Split A Node

แตก node ใหม่เมื่อเจออย่างน้อย 1 ข้อ:

- note เริ่มยาวเกิน 80-120 บรรทัดและมีมากกว่า 1 เรื่อง
- insight นี้จะใช้ซ้ำเกิน 2 ครั้ง
- เป็น root cause, playbook, checklist, business rule, หรือ contract
- AI ตัวถัดไปอ่าน node นี้แล้วเริ่มงานต่อได้เร็วขึ้น
- daily note เริ่มกลายเป็น transcript/debug dump

## Node Template

```md
# Node Name

## What
เรื่องนี้คืออะไร

## Why It Matters
ทำไมต้องจำ

## Current Rule / Contract
กฎล่าสุดคืออะไร

## Evidence
อ้างอิงจากไฟล์, query, note, หรือ session ไหน

## Common Failure
พังหรือเข้าใจผิดยังไงบ่อย

## Next Time
ครั้งหน้าถ้ากลับมาเรื่องนี้ ให้เริ่มจากตรงไหน
```

## Placement Rule

| ถ้าเป็น | วางที่ |
|---|---|
| งาน/bug/requirement ของระบบเดียว | `Projects/<project>/` |
| pattern ใช้ซ้ำข้าม project | `Tech/` หรือ `Tech/Atomic/` |
| การเงิน/สุขภาพ/career ต่อเนื่อง | `Areas/` |
| ตัวตน/เป้าหมาย/วิธีคิด | `Identity/` |
| ยังไม่รู้จะเอาไปไหน | `Inbox/Inbox` |

## Maintenance

- MOC เป็นสารบัญ ไม่ใช่ที่เก็บเนื้อหาหนัก
- ใช้ vault-path wikilink เช่น `Projects/STS/<note name>` ไม่ใช้ `../`
- หลังแตก node แล้ว daily note ควรเหลือ summary + link ไม่ต้อง copy เนื้อหาซ้ำ
- ถ้า node ใช้ซ้ำข้าม project แล้ว ค่อย promote หรือ link จาก `Tech`
