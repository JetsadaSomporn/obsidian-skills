---
title: Payment Tracker - API Routes
tags:
  - api
  - nextjs
  - backend
type: note
status: active
created: 2026-04-25
updated: 2026-04-25
links:
  - "[[Payment Tracker - Overview]]"
---

# API Routes

> ทุก mutating route ผ่าน pattern หลัก: Origin/CSRF → Rate limit → Auth → Validate → Execute

## Pattern ร่วม

```ts
if (!requireSameOrigin(request)) return jsonError("forbidden origin", 403);
if (!requireCsrfToken(request)) return jsonError("invalid csrf token", 403);
if (!checkRateLimit(request, { ... })) return jsonError("too many requests", 429);
const auth = await requireAuthenticatedUser(request);
if (!auth.ok) return jsonError(auth.error, auth.status);
```

## Routes

### POST `/api/slips/process`
อ่านสลิปจาก OCR text ด้วย NVIDIA DeepSeek

| | |
|---|---|
| Rate limit | 20 req / 60s |
| Input | `FormData` — `file: File`, `rawText: string` |
| Validation | `validateSlipFileMetadata` (type, size) + `validateSlipFileSignature` (magic bytes) |
| Output | `{ ok, userId, provider, slip: SlipExtractionResult }` |
| Provider | `provider: "nvidia"` |
| **สถานะ** | เชื่อม NVIDIA `deepseek-ai/deepseek-v4-flash` แล้ว แต่รับเฉพาะ OCR text ไม่ใช่รูปโดยตรง |

Client flow:
```text
file -> tesseract.js OCR -> rawText -> /api/slips/process
```

### GET `/api/transactions`
โหลด transaction ของ user จาก Supabase

| | |
|---|---|
| Rate limit | 120 req / 60s |
| Auth | Bearer token |
| DB | Select `transactions` ผ่าน Supabase + RLS |
| Output | `{ ok, transactions }` |
| Security | decrypt sensitive fields server-side ก่อนส่งกลับ client |

### POST `/api/transactions`
บันทึก transaction หลัง user confirm

| | |
|---|---|
| Rate limit | 60 req / 60s |
| Input | JSON body (max 32KB) |
| Validation | `transactionSchema` (Zod) |
| DB | Insert → `transactions` table ผ่าน Supabase + RLS |
| Security | encrypt `title`, `bankName`, `receiverName`, `referenceNo` ก่อน insert |
| Duplicate | ใช้ `reference_no_hash` HMAC จับ duplicate |
| Dev mode | ไม่มี mock save แล้ว; ต้องใช้ Supabase persistence |

```ts
// Zod schema
{
  type: "income" | "expense" | "transfer",
  amount: number (positive, max 100M),
  fee: number (0–100K),
  title: string (1–120 chars),
  categoryName: string (1–60 chars),
  transactionDate: "YYYY-MM-DD",
  transactionTime?: "HH:MM" | null,
  bankName?: string | null,
  receiverName?: string | null,
  referenceNo?: string | null,
}
```

### POST `/api/summaries/daily`
คำนวณ daily summary (เรียกจาก frontend)

| | |
|---|---|
| Rate limit | 30 req / 60s |
| Input | `{ transactions: TransactionItem[] }` (max 500 items) |
| Output | AI-generated insight text + totals |
| **สถานะ** | ⚠️ `summarizeToday` ยังเป็น local computation ไม่ได้เรียก NVIDIA จริง |

### GET `/api/cron/daily-summary`
Vercel Cron Job — trigger ทุกเที่ยงคืน

| | |
|---|---|
| Auth | `Authorization: Bearer {CRON_SECRET}` |
| Security | Timing-safe XOR comparison (ป้องกัน timing attack) |
| Output | ตอนนี้เป็น wired placeholder; Supabase-backed generation ยังเป็นงานต่อไป |

## ยังไม่มี (Pending)

| Route | ใช้ทำ |
|---|---|
| `POST /api/slips/create` | บันทึก slip image เข้า Supabase Storage |
| `GET /api/export/csv` | Export transactions เป็น CSV |
| `POST /api/chat/money` | Chat กับ AI เรื่องรายจ่าย |
