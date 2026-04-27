---
title: Payment Tracker - Overview
tags:
  - project
  - nextjs
  - supabase
  - finance
  - playground
type: project
status: active
created: 2026-04-25
updated: 2026-04-25
links:
  - "[[MOC - Projects Overview]]"
  - "[[Payment Tracker - Handoff 2026-04-25]]"
---

# Payment Tracker

> แอปติดตามรายจ่ายจากสลิปธนาคารไทย — confirm-first flow ไม่มีการ auto-save จาก OCR

## One-liner
อัปโหลดสลิป → AI อ่านข้อมูล → ผู้ใช้ยืนยัน → บันทึก ledger

## Stack
| Layer | Technology |
|---|---|
| Framework | Next.js 16 App Router |
| Language | TypeScript |
| Styling | Tailwind CSS v4 + CSS Variables |
| Database | Supabase (PostgreSQL + RLS) |
| Auth | Supabase Auth — Google OAuth |
| Storage | Supabase Storage (private bucket) |
| OCR | tesseract.js ฝั่ง browser |
| AI — Slip | NVIDIA API + `deepseek-ai/deepseek-v4-flash` parse OCR text |
| AI — Summary | local summary ตอนนี้; env เตรียม NVIDIA summary model ไว้ |
| Deploy | Vercel |

## Notes Index
- [[Payment Tracker - Architecture]]
- [[Payment Tracker - Security]]
- [[Payment Tracker - UI Design]]
- [[Payment Tracker - API Routes]]
- [[Payment Tracker - Database Schema]]
- [[Payment Tracker - Pending Work]]
- [[Payment Tracker - OCR and NVIDIA DeepSeek Flow]]
- [[Payment Tracker - Vercel Deploy 404 Resolution]]
- [[Payment Tracker - Handoff 2026-04-25]]
- [[Payment Tracker - Problems & Fixes]]

## Current State — 2026-04-25
- Repo: `/Users/jetsadasomporn/Vscode/Payment_tracking`
- GitHub: `JetsadaSomporn/Payment-Tracking`
- Vercel redeploy ใช้งานได้หลังลบ broken git submodule แล้วสร้าง Vercel project ใหม่
- ระบบใช้ Google Login ผ่าน Supabase Auth
- Slip extraction flow ปัจจุบันคือ `image -> browser OCR -> NVIDIA DeepSeek text parse -> user confirm -> save`
- ต้อง apply Supabase migration `004_transaction_reference_hash.sql` ใน project DB ก่อน save transaction ใช้ได้ครบ
