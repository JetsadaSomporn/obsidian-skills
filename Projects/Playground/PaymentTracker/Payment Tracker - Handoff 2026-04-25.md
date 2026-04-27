---
title: Payment Tracker - Handoff 2026-04-25
tags:
  - project
  - handoff
  - payment-tracker
  - vercel
  - security
  - ocr
type: handoff
status: active
created: 2026-04-25
updated: 2026-04-25
links:
  - "[[Payment Tracker - Overview]]"
  - "[[Payment Tracker - Security]]"
  - "[[Payment Tracker - OCR and NVIDIA DeepSeek Flow]]"
  - "[[Payment Tracker - Vercel Deploy 404 Resolution]]"
---

# Payment Tracker - Handoff 2026-04-25

> รอบนี้คือรอบ hardening + AI provider pivot + Vercel deploy unblock ของโปรเจค Payment Tracker

## Repo / Deploy

- Local repo: `/Users/jetsadasomporn/Vscode/Payment_tracking`
- GitHub repo: `JetsadaSomporn/Payment-Tracking`
- Main branch: `main`
- Vercel production project: user ลบ project เดิมแล้วสร้างใหม่ จึงใช้งานได้แล้ว
- Important commits:
  - `e45b11b docs: expand project readme`
  - `65a3204 chore: remove broken submodule link`

## Original Goal

ทำเว็บบันทึกรายรับรายจ่ายจากสลิปธนาคารไทย:

```text
อัปโหลดสลิป
-> AI/OCR อ่านข้อมูล
-> user ตรวจและยืนยัน
-> บันทึก transaction ลง Supabase
-> ดู dashboard / ledger / insight
```

แนวคิดหลักคือ `confirm-first` ไม่ใช่ auto-save จาก AI

## Key Architecture After This Round

```text
Slip image
-> tesseract.js OCR in browser
-> raw OCR text
-> /api/slips/process
-> NVIDIA DeepSeek V4 Flash parses text into JSON
-> user reviews form
-> /api/transactions
-> encrypt sensitive fields
-> Supabase Postgres with RLS
```

DeepSeek ไม่เห็นรูปโดยตรงแล้ว เพราะ model ที่ใช้เป็น text model

## Problems Found

### 1. Security readiness ไม่พอ

พบ risk หลัก:
- transaction เคย persist ใน `localStorage`
- localStorage encoding เป็น XOR/base64 ไม่ใช่ encryption
- sensitive fields ใน DB ยังเป็น plaintext
- duplicate check จะพังถ้า encrypt `reference_no` ด้วย AES-GCM เพราะ random IV ทำ ciphertext ไม่ซ้ำ
- rate limiter ถ้า DB RPC fail เคย fallback เป็น memory เงียบๆ ใน production
- `.env` local เคยมี service role key อยู่

### 2. DeepSeek provider misunderstanding

User ต้องการเลิกใช้ DeepSeek API ตรง แล้วใช้ NVIDIA API model:

```text
deepseek-ai/deepseek-v4-flash
```

ทดสอบจริง:
- text call ผ่าน `200`
- image payload fail `400`
- error สำคัญ: `deepseek_v4 is not a multimodal model`

Conclusion:
- DeepSeek V4 Flash ใช้ parse text ได้
- มันไม่ใช่ OCR/vision model
- ต้องมี OCR step ก่อน

### 3. OCR provider decision

ถามว่ามี OCR ฟรีไหม

Decision:
- ใช้ `tesseract.js` ฝั่ง browser ก่อน
- เหตุผล: deploy Vercel ง่ายกว่า native Tesseract binary, ฟรี, ไม่ต้องส่งรูปไป OCR provider
- Trade-off: ช้าบนเครื่อง user และภาษาไทยอาจเพี้ยน

### 4. Supabase migration blocker

หลังเพิ่ม encrypted `reference_no` ต้องเพิ่ม:

```sql
reference_no_hash text
```

และ unique index:

```sql
create unique index if not exists transactions_user_reference_hash_unique
on transactions (user_id, reference_no_hash)
where reference_no_hash is not null;
```

ตอนแรกเช็คแล้ว production DB ยังไม่มี column นี้:

```text
column transactions.reference_no_hash does not exist
```

ต่อมาผู้ใช้ apply migration แล้ว schema check ได้:

```text
reference_no_hash_column=true
```

### 5. Vercel 404 / deployment confusion

Build log ผ่านและมี route:

```text
/
/app
/upload
/transactions
/insights
/settings
```

แต่เข้าเว็บไม่ได้:
- deployment URLs ตอบ `401 Authentication Required`
- production alias `payment-tracking-tawny.vercel.app` ตอบ Vercel edge `404 NOT_FOUND`

Root causes identified:
- Vercel Deployment Protection เปิดอยู่ (`Standard Protection`)
- production alias/domain mapping ดูไม่ตรง deployment
- repo ยังมี broken git submodule/gitlink `Payment Tracking`
- Vercel warning: `Failed to fetch one or more git submodules`

Fix:
- remove broken gitlink from repo index
- add `/Payment Tracking/` to `.gitignore`
- push commit `65a3204`
- user สุดท้ายลบ Vercel project เดิมแล้วสร้างใหม่ จึงใช้งานได้

## Changes Made

### Security

- Removed financial transaction persistence from browser `localStorage`
- App now loads transactions via `GET /api/transactions`
- App clears old `payment-tracker.transactions.v1` local cache
- Sensitive transaction fields encrypted server-side before DB insert:
  - `title`
  - `bank_name`
  - `receiver_name`
  - `reference_no`
- Added `hashField` HMAC for `reference_no_hash`
- Production rate limiter now fails closed if DB RPC unavailable
- Local `.env` service role was redacted
- Generated local `INTERNAL_ENCRYPTION_SECRET`, `INTERNAL_ENCRYPTION_SALT`, `CRON_SECRET`

### AI / OCR

- Removed DeepSeek API direct env from docs/env example
- Added NVIDIA env:
  - `NVIDIA_API_KEY`
  - `NVIDIA_BASE_URL=https://integrate.api.nvidia.com/v1`
  - `NVIDIA_SLIP_MODEL=deepseek-ai/deepseek-v4-flash`
- `/api/slips/process` now expects OCR `rawText`
- Browser uses `tesseract.js` with `tha+eng`
- NVIDIA DeepSeek parses OCR text into JSON

### Docs

- README rewritten with:
  - screenshots
  - setup
  - Google Login
  - Supabase migration
  - NVIDIA DeepSeek
  - OCR
  - Vercel deployment
  - security model
  - testing checklist

### Vercel

- Removed broken submodule link `Payment Tracking`
- Pushed fix to GitHub
- User later recreated Vercel project, which resolved access/deploy confusion

## Files Touched In Repo

Main files:
- `src/components/payment-tracker-app.tsx`
- `src/app/api/slips/process/route.ts`
- `src/app/api/transactions/route.ts`
- `src/lib/ai/slip-extraction.ts`
- `src/lib/security/encryption.ts`
- `src/lib/security/api.ts`
- `src/proxy.ts`
- `supabase/migrations/004_transaction_reference_hash.sql`
- `.env.example`
- `README.md`
- `.gitignore`
- `payment_tracker.md`

## Validations Run

### Local

```bash
npm run lint
npm run build
```

Both passed after the changes.

### NVIDIA

Text call:

```text
status=200
model=deepseek-ai/deepseek-v4-flash
```

Image call:

```text
status=400
deepseek_v4 is not a multimodal model
```

OCR text parsing test:

Input text included:

```text
จำนวนเงิน 1,250.00 บาท
วันที่ 25/04/2026 เวลา 14:32
ผู้รับ นายสมชาย ใจดี
เลขที่รายการ 202604251432123456
```

DeepSeek returned:

```json
{
  "amount": 1250,
  "transaction_date_iso": "2026-04-25",
  "transaction_time": "14:32",
  "receiver_name": "นายสมชาย ใจดี",
  "reference_no": "202604251432123456"
}
```

### Supabase

Checked:

```text
reference_no_hash_column=true
```

### Vercel

Before recreate:
- Build passed
- Deployment URLs protected by 401
- Production alias returned Vercel edge 404

After user recreated Vercel project:
- User reported it works

## Current Verdict

```text
Ready for MVP usage/testing
Not financial-grade production yet
```

Safe enough for:
- personal usage
- MVP demo
- controlled deploy

Still not enough for:
- public financial SaaS
- no-monitor production
- fully automated accounting without user confirmation

## Next Actions

1. Test real Thai slips from multiple banks
2. Confirm OCR quality and confidence handling
3. Confirm Google OAuth redirect on final Vercel domain
4. Confirm Vercel env vars are set in Production
5. Optional: upload original slip image to Supabase Storage and link `slip_id`
6. Optional: wire `ai_logs` for token usage and extraction audit
7. Optional: add export CSV and money chat route

## Notes For Next AI

- Do not try to send slip images directly to `deepseek-ai/deepseek-v4-flash`; it is text-only in the tested NVIDIA endpoint.
- If slip extraction fails, inspect browser OCR output first, not only backend.
- If save transaction fails with `reference_no_hash`, check Supabase migration before changing code.
- If Vercel build passes but URL fails, separate:
  - Deployment Protection / 401
  - domain alias mapping / Vercel edge 404
  - actual app runtime 500
- `Payment Tracking` nested folder is local noise and should stay ignored.
