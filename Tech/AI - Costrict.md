---
title: AI - Costrict
tags:
  - ai
  - costrict
  - enterprise
  - code-review
  - tdd
  - open-source
created: 2026-04-12
updated: 2026-04-12
---

# Costrict — Enterprise AI Coding Agent

> เรียกด้วย: **`cs`**
> Provider: [[Provider - Sangfor|Sangfor]] / zgsm-ai (Open Source)

---

## ข้อมูลทั่วไป

- · Provider: Sangfor / zgsm-ai
- · Plan: Free — open source, private deployment ได้
- · Interface หลัก: VS Code Extension
- · ชื่อเดิม: Cline / RooCode + Code Review AI Coding Agent
- · GitHub: github.com/zgsm-ai/costrict

---

## จุดแข็ง

- · AI Agent — ทำงาน end-to-end จาก requirement ถึง code ได้เอง
- · AI Code Review — review code แบบ multi-expert (architect + security + test specialist)
- · AI Completion — inline autocomplete ตอบสนองเร็วระดับ sub-second
- · Strict Mode Workflow — แตก requirement เป็น module → วิเคราะห์ → ออกแบบ → plan → generate test → เขียนโค้ด ครบ loop
- · Test-Driven Development (TDD) Engine — สร้าง test suite ก่อนเขียนโค้ดจริงเสมอ
- · Multi-Expert Review Model — จำลองมุมมอง 3 ด้านต่อโค้ดทุกบรรทัด: architecture, security, test
- · Repository-wide indexing + RAG — อ่าน codebase ทั้ง repo แล้วตอบได้ในบริบทจริง
- · Enterprise Quality Gates — ต่อ GitLab automate quality check ก่อน merge ได้
- · MCP Service support — ต่อ MCP tools ได้
- · Multiple free models — ใช้ model ฟรีหลายตัวได้ หรือ custom API/model เองได้
- · Private deployment — โค้ดไม่ออกไปข้างนอก เหมาะกับงาน enterprise ที่ sensitive
- · Open source — ดู source, แก้ไข, self-host ได้ทั้งหมด

---

## เมื่อไหร่ควรใช้

- · ต้องการ code review ที่ลึกกว่า Copilot — มอง security + architecture พร้อมกัน
- · งาน enterprise ที่โค้ดออกไปนอก organization ไม่ได้
- · ต้องการ TDD workflow — ให้ AI เขียน test ก่อนแล้วค่อย implement
- · ต้องการ agentic coding ฟรี (ไม่อยากจ่าย Claude/Copilot)
- · ต้องการ review PR / merge request แบบอัตโนมัติใน GitLab pipeline

---

## วิธีเรียกใช้

- · VS Code: ติดตั้ง extension ชื่อ **CoStrict** (zgsm-ai.zgsm) จาก Marketplace
- · เปิด sidebar CoStrict → เลือก model → พิมพ์ requirement
- · Code Review: เปิดไฟล์ → คลิก `cs review` หรือ trigger จาก command palette
- · GitLab integration: ตั้งค่า webhook → CoStrict review MR อัตโนมัติ

---

## Links

- ← [[AI Agents - My Arsenal]]
