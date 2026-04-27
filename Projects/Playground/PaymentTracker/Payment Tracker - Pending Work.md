---
title: Payment Tracker - Pending Work
tags:
  - todo
  - pending
type: note
status: active
created: 2026-04-25
updated: 2026-04-25
links:
  - "[[Payment Tracker - Overview]]"
---

# Pending Work

> สถานะหลังรอบ hardening / OCR / Vercel deploy วันที่ 2026-04-25

## Must Verify Before Calling Production

### 1. Real slip OCR accuracy
ระบบทำงานเป็น `tesseract.js -> NVIDIA DeepSeek parse` แล้ว แต่ OCR ฟรีอาจอ่านสลิปไทยเพี้ยน

Acceptance:
- ทดสอบสลิปจริงหลายธนาคาร
- ตรวจว่า amount/date/reference ถูกพอใช้
- หน้า confirm ต้องยังเป็น source of truth ก่อนบันทึก

### 2. Supabase production data migration
ต้อง confirm ว่า production Supabase มี `transactions.reference_no_hash`

Relevant file:
- `supabase/migrations/004_transaction_reference_hash.sql`

Why:
- `reference_no` ถูก encrypt ด้วย AES-GCM มี random IV
- duplicate check จึงต้องใช้ HMAC hash แยก ไม่ใช่ unique index จาก plaintext

### 3. Vercel env parity
Vercel ต้องมี env เหล่านี้:
- `NEXT_PUBLIC_SUPABASE_URL`
- `NEXT_PUBLIC_SUPABASE_ANON_KEY`
- `NVIDIA_API_KEY`
- `NVIDIA_BASE_URL`
- `NVIDIA_SLIP_MODEL`
- `NVIDIA_SUMMARY_MODEL`
- `INTERNAL_ENCRYPTION_SECRET`
- `INTERNAL_ENCRYPTION_SALT`
- `CRON_SECRET`

## Medium

### 4. Supabase Storage — upload slip image
`/api/slips/process` validate รูปแล้ว แต่ยังไม่ได้ upload slip image เข้า private bucket จริง

ถ้าต้องการเก็บหลักฐานสลิป:
```ts
await supabase.storage
  .from("slips")
  .upload(`${hashedUserPath}/${crypto.randomUUID()}`, file);
```

ต้องระวัง storage path ตอนนี้ policy ใช้ `md5(auth.uid())`

### 5. Export CSV — `/api/export/csv`
ไม่มี route นี้เลย ต้อง implement ใหม่ทั้งหมด

### 6. Chat AI — `/api/chat/money`
ไม่มี route นี้เลย

### 7. Slip → Transaction link
ตอนนี้ save transaction แต่ไม่ได้ link `slip_id` เข้า `transactions.slip_id`

## UI

### 8. Scroll animation (Landing page)
ยังไม่มี intersection observer fade-in เมื่อ scroll

### 9. Mobile sidebar
Sidebar ปัจจุบัน `hidden lg:flex` — mobile ไม่มี nav เลย
ต้องทำ drawer หรือ bottom tab bar สำหรับ mobile

## Low Priority

### 10. Budget alerts
`budgets` table มีใน schema แต่ logic ยังไม่มี

### 11. Merchant rules auto-categorize
`merchant_rules` table มีแต่ยังไม่ได้ใช้

### 12. ai_logs
บันทึก token usage ยังไม่ได้ wire
