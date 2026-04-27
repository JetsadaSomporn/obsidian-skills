---
title: Payment Tracker - Architecture
tags:
  - architecture
  - nextjs
  - supabase
type: note
status: active
created: 2026-04-25
updated: 2026-04-25
links:
  - "[[Payment Tracker - Overview]]"
---

# Architecture

## File Structure

```
src/
├── app/
│   ├── page.tsx                  # Landing page (public)
│   ├── layout.tsx                # Root layout — Google Fonts (next/font)
│   ├── globals.css               # Design system tokens + component classes
│   ├── app/page.tsx              # Actual app entry — /app route
│   ├── upload/page.tsx
│   ├── transactions/page.tsx
│   ├── insights/page.tsx
│   ├── settings/page.tsx
│   └── api/
│       ├── slips/process/route.ts
│       ├── transactions/route.ts
│       ├── summaries/daily/route.ts
│       └── cron/daily-summary/route.ts
├── components/
│   └── payment-tracker-app.tsx   # Main SPA component (all views)
├── lib/
│   ├── ai/slip-extraction.ts     # NVIDIA DeepSeek OCR-text parser
│   ├── money.ts                  # formatTHB, parseAmount, todayBangkokDate
│   ├── summary.ts                # summarizeToday, buildPeriodSummary
│   ├── types.ts                  # Transaction, SlipExtractionResult
│   ├── security/
│   │   ├── api.ts                # rate limit, CSRF, Same-Origin, JSON helpers
│   │   ├── encryption.ts         # AES-256-GCM + HMAC hash helpers
│   │   └── upload.ts             # validateSlipFileMetadata, magic bytes check
│   └── supabase/
│       ├── client.ts             # getBrowserSupabaseClient
│       └── server.ts             # requireAuthenticatedUser, createAuthenticatedSupabaseClient
└── proxy.ts                      # Next.js 16 proxy (แทน middleware.ts ที่ deprecated)
```

## Routing
- `/` → Landing page (marketing)
- `/app` → Dashboard (Today view)
- `/upload` → Upload + confirm slip
- `/transactions` → Ledger
- `/insights` → AI summary
- `/settings` → Config

## Key Design Decisions

### proxy.ts แทน middleware.ts
Next.js 16 deprecated `middleware.ts` — ใช้ `proxy.ts` แทน
สร้าง CSP nonce per-request ที่นี่ แล้วส่งผ่าน `x-nonce` header

### `await connection()` ใน page.tsx
Force dynamic rendering เพื่อให้ nonce จาก proxy ถูก apply ทุก request

### DB-backed rate limiter
ใช้ Supabase RPC `check_rate_limit`

Production ถ้า DB-backed rate limiter ใช้ไม่ได้จะ fail closed ไม่ fallback เป็น memory เงียบๆ

### Single-file SPA component
`payment-tracker-app.tsx` เป็น component เดียวที่รวมทุก view
แต่ละ page (upload, transactions, ฯลฯ) แค่ render `<PaymentTrackerApp initialView="upload" />`

### OCR before DeepSeek
`deepseek-ai/deepseek-v4-flash` บน NVIDIA endpoint ถูกทดสอบแล้วว่าเป็น text model

ดังนั้น flow ที่ถูกต้องคือ:

```text
image -> tesseract.js OCR -> rawText -> NVIDIA DeepSeek parse
```

ดูเพิ่ม: [[Payment Tracker - OCR and NVIDIA DeepSeek Flow]]

### Supabase as persistence source
Transaction ไม่ถูกเก็บถาวรใน browser `localStorage` แล้ว

Client จะ:
- clear old local cache
- fetch transaction จาก `/api/transactions`
- save ผ่าน `/api/transactions`

Server จะ encrypt sensitive fields ก่อนลง DB
