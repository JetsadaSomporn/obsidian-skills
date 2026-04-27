---
title: Tech - Overview
tags:
  - moc
  - tech
  - resources
type: moc
status: active
created: 2026-04-11
updated: 2026-04-24
links:
  - "[[Home]]"
  - "[[Projects/STS/Character Count Policy]]"
  - "[[Projects/STS/Alert Architecture Map]]"
  - "[[Tech/Atomic/PostgreSQL - Function Overload Verification]]"
---

# Tech - Overview

> ห้องเก็บความรู้ที่ควรใช้ซ้ำได้ ไม่ผูกกับวันใดวันหนึ่ง

## Core Notes
- [[Tech/AI Agents - My Arsenal|AI Agents - My Arsenal]]
- [[Tech/CSharp .NET API Patterns|C# .NET API Patterns]]
- [[Tech/LLM Prompting|LLM Prompting]]
- [[Tech/PostgreSQL Patterns|PostgreSQL Patterns]]

## Atomic Notes
- [[Tech/Atomic/PostgreSQL - Function Overload Verification|PostgreSQL - Function Overload Verification]]

## Recent Reusable Notes
```dataview
TABLE file.link AS Note, type AS Type, updated AS Updated
FROM "Tech" OR ""
WHERE (file.folder = "Tech" OR startswith(file.folder, "Tech/")) AND file.name != "MOC - Tech Overview"
SORT updated DESC
LIMIT 15
```

## STS Knowledge Promoted To Reusable Tech
- [[Projects/STS/Character Count Policy|Character Count Policy]]
- [[Projects/STS/Alert Architecture Map|Alert Architecture Map]]

## What Belongs Here
- Pattern ที่ใช้ซ้ำได้
- Tradeoff ที่คิดจบแล้ว
- สูตร, query, command, หรือ prompt ที่ไม่อยาก reverse engineer ใหม่
- สรุปจาก project ที่ยกระดับเป็น reusable knowledge แล้ว

## Promotion Rule
- ถ้า note อยู่ใน project แล้วใช้ซ้ำเกิน 2 ครั้ง ควรเลื่อนแนวคิดมาที่ `Tech`
- ถ้า note ยังผูกกับ context ของ project มากเกินไป ให้ link จาก Tech มาแทนการย้าย
