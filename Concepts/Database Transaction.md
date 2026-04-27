---
title: Database Transaction
tags: [concept, database, reliability]
ai-context: "Fundamental concept of atomicity in database operations."
aliases: [Transaction, ACID]
created: 2026-04-16
---

# Database Transaction

## Definition
กลุ่มของคำสั่ง SQL ที่ทำงานรวมกันเป็นหนึ่งเดียว (All or Nothing) เพื่อรักษาความถูกต้องของข้อมูล (Data Integrity)

## ACID Properties
- **Atomicity:** ทำงานสำเร็จทั้งหมด หรือล้มเหลวทั้งหมด
- **Consistency:** ข้อมูลอยู่ในสถานะที่ถูกต้องเสมอหลังจบ transaction
- **Isolation:** การทำงานขนานกันของหลาย transaction ไม่รบกวนกัน
- **Durability:** เมื่อ commit แล้ว ข้อมูลจะอยู่ถาวรแม้ระบบล่ม

## Practical Use
ในงาน STS-NOC, Transaction ถูกใช้เพื่อรักษาการเปิด `refcursor` ใน PostgreSQL ระหว่างการเรียก SP และการ Fetch ข้อมูล

## Related
- [[PostgreSQL - Dapper SP Refcursor Pattern]]
- [[PostgreSQL Patterns]]
