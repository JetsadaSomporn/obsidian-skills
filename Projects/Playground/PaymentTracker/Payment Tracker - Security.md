---
title: Payment Tracker - Security
tags:
  - security
  - csp
  - supabase
  - rls
type: note
status: active
created: 2026-04-25
updated: 2026-04-25
links:
  - "[[Payment Tracker - Overview]]"
---

# Security

> สถานะหลัง hardening วันที่ 2026-04-25: ใช้ได้สำหรับ MVP / deploy ทดลองจริง แต่ยังไม่ใช่ financial-grade production

## Layer ที่ทำแล้ว

### 1. CSP Nonce-based (proxy.ts)
```ts
const nonce = Buffer.from(crypto.randomUUID()).toString("base64");
// script-src 'nonce-{value}' 'strict-dynamic'
// ทำใน proxy.ts ไม่ใช่ next.config.ts เพราะต้องการ per-request value
```
- `'strict-dynamic'` ทำให้ scripts ที่ load โดย trusted script ผ่านได้โดยไม่ต้อง whitelist
- `'unsafe-inline'` ถูกลบออก — CSP มีความหมายจริง

### 2. SRI (Subresource Integrity)
```ts
// next.config.ts
experimental: { sri: { algorithm: "sha256" } }
```

### 3. Auth — requireAuthenticatedUser
```ts
// ทุก API route เรียก requireAuthenticatedUser(request) ก่อนเสมอ
// อ่าน Bearer token → supabase.auth.getUser(token)
// ถ้าไม่มี token หรือ invalid → 401 ทันที
```

### 4. Row Level Security (Supabase)
- ทุก 8 ตาราง: `profiles`, `categories`, `slips`, `transactions`, `daily_summaries`, `budgets`, `merchant_rules`, `ai_logs`
- ทุก policy ใช้ `auth.uid() = user_id`

### 5. Magic Bytes Validation (upload)
```ts
// ตรวจ byte จริงก่อนส่งไป AI — ป้องกัน file type spoofing
// JPEG: FF D8 FF
// PNG:  89 50 4E 47 0D 0A 1A 0A
// WebP: 52 49 46 46 ... 57 45 42 50
```

### 6. readJsonBody — วัด byte จริง
```ts
// ไม่ใช้ Content-Length header (bypass ได้)
// อ่าน request.text() → TextEncoder().encode(raw).byteLength
```

### 7. Timing-safe CRON_SECRET
```ts
// ใช้ XOR loop แทน === เพื่อป้องกัน timing attack
let mismatch = lengthsMatch ? 0 : 1;
for (let i = 0; i < expected.byteLength; i++) {
  mismatch |= expected[i]! ^ paddedProvided[i]!;
}
```

### 8. Zod Validation ทุก endpoint
- `/api/transactions` — `transactionSchema`
- `/api/summaries/daily` — `summaryPayloadSchema` (array max 500 items)
- `/api/slips/process` — metadata + magic bytes

### 8.1 CSRF + Same-Origin
- POST endpoints ใช้ `requireSameOrigin`
- POST endpoints ใช้ double-submit CSRF cookie/header
- Browser client ส่ง `x-csrf-token` จาก cookie พร้อม Bearer token

### 9. Security Headers (next.config.ts)
```
Referrer-Policy: strict-origin-when-cross-origin
X-Content-Type-Options: nosniff
X-Frame-Options: DENY
Permissions-Policy: camera=(), microphone=(), geolocation=(), ...
HSTS: max-age=63072000 (production only)
Cache-Control: no-store (api routes)
```

### 10. Google Fonts via next/font
- Self-hosted ที่ build time — ไม่มี external font CDN
- ทำให้ `font-src 'self'` ใน CSP ทำงานได้จริง

### 11. DB-backed rate limiting
- `check_rate_limit` RPC ใน Supabase
- Production fail closed ถ้า DB-backed rate limiter ใช้ไม่ได้
- Local/dev ยัง fallback เป็น memory bucket ได้เพื่อ dev สะดวก

### 12. Field encryption
Sensitive transaction fields ถูก encrypt ก่อนลง DB:
- `title`
- `bank_name`
- `receiver_name`
- `reference_no`

ใช้ AES-256-GCM ใน `src/lib/security/encryption.ts`

Duplicate reference ใช้ `reference_no_hash` แบบ HMAC แยก เพราะ AES-GCM ใช้ random IV ทำให้ ciphertext ของค่าเดิมไม่ซ้ำกัน

### 13. Browser local storage disabled for financial data
รอบก่อนมี `localStorage` + XOR/base64 ซึ่งไม่ถือว่าเป็น encryption

รอบแก้:
- ย้าย transaction persistence ไป Supabase API
- ลบ browser cache เก่า `payment-tracker.transactions.v1` ตอนเปิด app
- Settings แสดง `Browser cache: Disabled for financial data`

### 14. Secret handling
- `.env` ถูก ignore
- `SUPABASE_SERVICE_ROLE_KEY` ไม่จำเป็นกับ runtime app
- tracked files ถูกเช็คแล้วไม่เจอ service role / NVIDIA key / internal secret จริง

## สิ่งที่ยังขาด / ยอมรับ limitation

| รายการ | สถานะ | เหตุผล |
|---|---|---|
| PostCSS advisory ผ่าน Next dependency chain | ยอมรับชั่วคราว | `npm audit fix --force` จะ downgrade/break; รอ Next patch |
| OCR accuracy | ต้อง verify เพิ่ม | Tesseract ฟรีและอาจอ่านภาษาไทยเพี้ยน |
| Vercel Deployment Protection/domain settings | ต้องดูใน Vercel UI | build ผ่านแต่ access/domain อาจติด setting |
| Audit log | ยังไม่มี | `ai_logs` table มีใน schema แต่ยังไม่ได้ wire |
