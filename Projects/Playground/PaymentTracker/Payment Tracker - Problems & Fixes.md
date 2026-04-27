---
title: Payment Tracker - Problems & Fixes
tags:
  - project
  - payment-tracker
  - debugging
  - security
  - vercel
  - supabase
type: troubleshooting
status: resolved
created: 2026-04-26
updated: 2026-04-26
links:
  - "[[Payment Tracker - Overview]]"
  - "[[Payment Tracker - Handoff 2026-04-25]]"
  - "[[Payment Tracker - Security]]"
  - "[[Payment Tracker - Vercel Deploy 404 Resolution]]"
  - "[[Payment Tracker - OCR and NVIDIA DeepSeek Flow]]"
---

# Payment Tracker — ปัญหาที่เจอ & การแก้ไข

> รวมปัญหาทั้งหมดที่เจอระหว่าง build + hardening รอบ 2026-04-25 พร้อม root cause และสิ่งที่แก้ไปแล้ว

---

## สรุปภาพรวม

| # | ปัญหา | สถานะ |
|---|---|---|
| 1 | Security — localStorage + encryption อ่อน | ✅ แก้แล้ว |
| 2 | NVIDIA DeepSeek ไม่รับ image payload | ✅ แก้แล้ว |
| 3 | OCR — ยังไม่มี provider | ✅ ตัดสินใจแล้ว |
| 4 | Supabase migration ยังไม่ apply | ✅ apply แล้ว |
| 5 | Vercel 404 / 401 ทั้งที่ build ผ่าน | ✅ แก้แล้ว |

---

## ปัญหาที่ 1 — Security

> [!danger] Security Risks Found
> พบ risk หลักหลายจุดก่อน hardening รอบนี้

### Root Cause

- Transaction ทั้งก้อนเคย persist ลง `localStorage` โดยตรง
- Encoding ใน localStorage เป็น XOR + base64 ไม่ใช่ encryption จริง
- Sensitive fields (`title`, `bank_name`, `receiver_name`, `reference_no`) เก็บใน DB เป็น plaintext
- `reference_no` ถ้า encrypt ด้วย AES-GCM random IV → ciphertext ไม่ซ้ำ → duplicate check พัง
- Rate limiter ถ้า DB RPC fail มันเคย fallback เป็น in-memory อย่างเงียบ ๆ ใน production
- `.env` local มี `SUPABASE_SERVICE_ROLE_KEY` อยู่

### สิ่งที่แก้

- ลบการ persist transaction ออกจาก `localStorage` ทั้งหมด
- App โหลด transaction ผ่าน `GET /api/transactions` แทน
- App clear cache เก่า `payment-tracker.transactions.v1` ออก
- Encrypt sensitive fields ฝั่ง server ก่อน insert:
  - `title`, `bank_name`, `receiver_name`, `reference_no`
- เพิ่ม `reference_no_hash` ด้วย HMAC แทนการ encrypt สำหรับ duplicate check
- Rate limiter ตอนนี้ **fail closed** ถ้า DB RPC ไม่ available
- Redact service role key ออกจาก local `.env`
- Generate `INTERNAL_ENCRYPTION_SECRET`, `INTERNAL_ENCRYPTION_SALT`, `CRON_SECRET` ใหม่

### Files ที่แก้

- `src/lib/security/encryption.ts`
- `src/lib/security/api.ts`
- `src/app/api/transactions/route.ts`
- `src/components/payment-tracker-app.tsx`
- `.env.example`

---

## ปัญหาที่ 2 — NVIDIA DeepSeek ไม่รับ Image

> [!bug] Model ไม่ใช่ Vision Model
> ส่ง image payload ตรงไปที่ `deepseek-ai/deepseek-v4-flash` แล้วได้ error

### Root Cause

เข้าใจผิดว่า DeepSeek V4 Flash บน NVIDIA API รับรูปได้ แต่จริง ๆ เป็น text-only model:

```text
status: 400
error: deepseek_v4 is not a multimodal model
```

### สิ่งที่แก้

- เปลี่ยน flow จาก `image → DeepSeek` เป็น `image → OCR → text → DeepSeek`
- `/api/slips/process` ตอนนี้รับ `rawText` จาก OCR แทน image payload
- DeepSeek ทำหน้าที่แค่ parse structured JSON จาก OCR text

### Validation

```text
Text call:   status=200  ✅
Image call:  status=400  error: deepseek_v4 is not a multimodal model  ❌ (คาดไว้แล้ว)
```

---

## ปัญหาที่ 3 — ไม่มี OCR Provider

> [!question] ต้องการ OCR ฟรีที่ deploy บน Vercel ได้

### Root Cause

ตอนเปลี่ยน flow ต้องการ OCR ก่อนส่ง text ให้ DeepSeek แต่ยังไม่มี provider

### ตัวเลือกที่พิจารณา

| Option | ข้อดี | ข้อเสีย |
|---|---|---|
| `tesseract.js` (browser) | ฟรี, ไม่ต้องส่งรูปออก, deploy Vercel ง่าย | ช้าบนเครื่อง user, ภาษาไทยอาจเพี้ยน |
| Native Tesseract binary (server) | แม่นกว่า | ติดตั้งบน Vercel serverless ยาก |
| Third-party OCR API | แม่น | มีค่าใช้จ่าย |

### สิ่งที่ตัดสินใจ

ใช้ **`tesseract.js`** รัน browser-side ก่อน:
- รองรับ `tha+eng` language pack
- ไม่ต้องส่งรูป slip ออกไปนอก browser เลย
- Deploy บน Vercel ได้ทันทีโดยไม่ต้องการ binary

> [!tip] Trade-off ที่ยอมรับ
> ช้าและอาจอ่านไทยเพี้ยนบ้าง แต่ acceptable สำหรับ MVP — DeepSeek จะช่วย normalize text ที่เพี้ยนอีกชั้นนึง

---

## ปัญหาที่ 4 — Supabase Migration ยังไม่ Apply

> [!warning] Save Transaction พังเพราะ Column ไม่มีใน Production DB

### Root Cause

เพิ่ม logic `reference_no_hash` ในโค้ดแล้ว แต่ production Supabase DB ยังไม่มี column:

```text
error: column transactions.reference_no_hash does not exist
```

### Migration ที่ต้อง Apply

```sql
-- supabase/migrations/004_transaction_reference_hash.sql
alter table transactions
  add column if not exists reference_no_hash text;

create unique index if not exists transactions_user_reference_hash_unique
  on transactions (user_id, reference_no_hash)
  where reference_no_hash is not null;
```

### สิ่งที่ทำ

Apply migration ผ่าน Supabase dashboard แล้ว verify:

```text
reference_no_hash_column = true  ✅
```

---

## ปัญหาที่ 5 — Vercel 404 / 401 ทั้งที่ Build ผ่าน

> [!bug] Build Log ผ่านทุก Route แต่เข้าเว็บไม่ได้

### อาการ

- Deployment URL ตอบ `401 Authentication Required`
- Production alias `payment-tracking-tawny.vercel.app` ตอบ Vercel edge `404 NOT_FOUND`
- Build log แสดง routes ครบ (`/`, `/app`, `/upload`, `/transactions`, `/insights`, `/settings`)

### Root Causes (3 ชั้น)

1. **Vercel Deployment Protection** เปิดอยู่ (`Standard Protection`) → deployment URL ทุกอันถูกกั้น 401
2. **Production alias mapping** ไม่ตรงกับ deployment ล่าสุด → Vercel edge 404
3. **Broken git submodule** — repo มี gitlink `Payment Tracking` อยู่ → Vercel warning: `Failed to fetch one or more git submodules`

### สิ่งที่แก้

```bash
# ลบ broken gitlink ออกจาก git index
git rm --cached "Payment Tracking"

# เพิ่มใน .gitignore
echo "/Payment Tracking/" >> .gitignore

git commit -m "chore: remove broken submodule link"
git push
```

สุดท้ายผู้ใช้ **ลบ Vercel project เดิมและสร้างใหม่** → ปัญหา alias mapping หายไปพร้อมกัน

> [!note] Key Lesson
> ถ้า Vercel build ผ่านแต่เข้าไม่ได้ ให้แยกวิเคราะห์ 3 ชั้น:
> 1. **Deployment Protection / 401** — ตรวจ Access settings
> 2. **Domain alias mapping / edge 404** — ตรวจ Domains tab
> 3. **App runtime error / 500** — ตรวจ Function logs

---

## Checklist หลังแก้ทุกปัญหา

- [x] `npm run lint` — passed
- [x] `npm run build` — passed
- [x] NVIDIA text call — `200`
- [x] Supabase `reference_no_hash` column — exists
- [x] Vercel production — accessible
- [ ] ทดสอบสลิปไทยจริงจากหลายธนาคาร
- [ ] Confirm Google OAuth redirect บน Vercel domain จริง
- [ ] Confirm Vercel env vars ครบทุกตัว

---

## See Also

- [[Payment Tracker - Security]]
- [[Payment Tracker - Vercel Deploy 404 Resolution]]
- [[Payment Tracker - OCR and NVIDIA DeepSeek Flow]]
- [[Payment Tracker - Handoff 2026-04-25]]
