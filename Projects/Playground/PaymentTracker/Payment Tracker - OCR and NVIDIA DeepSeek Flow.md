---
title: Payment Tracker - OCR and NVIDIA DeepSeek Flow
tags:
  - payment-tracker
  - ocr
  - nvidia
  - deepseek
type: note
status: active
created: 2026-04-25
updated: 2026-04-25
links:
  - "[[Payment Tracker - Overview]]"
  - "[[Payment Tracker - Handoff 2026-04-25]]"
---

# OCR and NVIDIA DeepSeek Flow

> DeepSeek ใน NVIDIA endpoint นี้เป็น text model จึงต้องให้ OCR อ่านรูปก่อน

## Correct Flow

```text
slip image
-> browser OCR with tesseract.js
-> raw slip text
-> NVIDIA DeepSeek V4 Flash
-> structured JSON
-> user confirm
```

## Why This Exists

User เปลี่ยน requirement:

```text
ไม่ใช้ DeepSeek API ตรง
ใช้ NVIDIA API model deepseek-ai/deepseek-v4-flash แทน
```

ทดสอบจริงแล้ว:

```text
Text request: 200 OK
Image request: 400 BadRequest
Error: deepseek_v4 is not a multimodal model
```

Conclusion:
- DeepSeek ใช้ parse ข้อความ OCR ได้
- DeepSeek ไม่เห็นจำนวนเงินจากรูปเอง
- OCR เป็น step ที่ทำให้จำนวนเงินกลายเป็น text

## Implementation

Client:
- `src/components/payment-tracker-app.tsx`
- function `extractTextFromSlipImage(file)`
- uses `tesseract.js`
- languages: `tha+eng`

Server:
- `src/app/api/slips/process/route.ts`
- expects `file` and `rawText`
- validates file metadata and magic bytes
- validates OCR text exists and size <= 64 KB
- calls `extractSlipTextWithNvidiaDeepSeek(rawText)`

AI parser:
- `src/lib/ai/slip-extraction.ts`
- calls NVIDIA OpenAI-compatible endpoint:

```text
https://integrate.api.nvidia.com/v1/chat/completions
```

Default model:

```text
deepseek-ai/deepseek-v4-flash
```

## Required Env

```env
NVIDIA_API_KEY=
NVIDIA_BASE_URL=https://integrate.api.nvidia.com/v1
NVIDIA_SLIP_MODEL=deepseek-ai/deepseek-v4-flash
```

## Tesseract Runtime Sources

CSP must allow:

```text
https://cdn.jsdelivr.net
https://tessdata.projectnaptha.com
```

Because browser downloads:
- worker script
- wasm core
- Thai traineddata
- English traineddata

## Known Trade-offs

| Option | Pros | Cons |
|---|---|---|
| Browser `tesseract.js` | free, Vercel friendly, image stays client-side | slower, Thai OCR may be imperfect |
| Google Vision / Azure OCR | better OCR | paid API + sends slip to provider |
| NVIDIA vision model | one ecosystem | must choose real multimodal model |
| DeepSeek V4 Flash only | simple text parser | cannot read image directly |

## Test Evidence

OCR-like text sample:

```text
ธนาคารกสิกรไทย
โอนเงินสำเร็จ
จำนวนเงิน 1,250.00 บาท
วันที่ 25/04/2026 เวลา 14:32
ผู้รับ นายสมชาย ใจดี
เลขที่รายการ 202604251432123456
```

NVIDIA DeepSeek parsed:

```json
{
  "document_type": "thai_bank_slip",
  "bank_name": "กสิกรไทย",
  "status": "success",
  "amount": 1250,
  "currency": "THB",
  "transaction_date_iso": "2026-04-25",
  "transaction_time": "14:32",
  "receiver_name": "นายสมชาย ใจดี",
  "reference_no": "202604251432123456",
  "confidence": 1
}
```

## Debug Rule

If extraction looks wrong:

1. Inspect OCR raw text first
2. If OCR text is wrong, improve image/preprocess/OCR provider
3. If OCR text is right but JSON is wrong, adjust DeepSeek prompt/schema
4. Do not blame Supabase or save route first
