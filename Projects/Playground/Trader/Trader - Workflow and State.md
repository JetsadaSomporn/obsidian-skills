---
title: Trader - Workflow and State
tags:
  - trader
  - workflow
  - state
  - alpaca
  - finance
type: workflow
status: active
created: 2026-04-19
updated: 2026-04-19
links:
  - "[[Projects/Playground/Trader/Trader - Overview]]"
  - "[[Projects/Playground/Trader/Trader - Architecture]]"
---

# Trader - Workflow and State

## Workflow Matrix

| Workflow | Trigger | Reads | Live calls | Writes |
|----------|--------|-------|-----------|--------|
| `pre-market` | ก่อน US open | strategy + tail trade log + tail research log | account / positions / orders / WebSearch | `memory/RESEARCH-LOG.md` |
| `market-open` | หลังเปิดตลาดไม่นาน | strategy + today's research + tail trade log | account / positions / quote / order | `memory/TRADE-LOG.md` |
| `midday` | กลาง session | strategy + tail trade log + today's research | positions / orders / optional WebSearch | `memory/TRADE-LOG.md`, บางครั้ง `memory/RESEARCH-LOG.md` |
| `daily-summary` | หลังปิดตลาด | tail trade log | account / positions / orders | `memory/TRADE-LOG.md` |
| `weekly-review` | หลัง close ของ Friday | weekly review + all week trade/research logs + strategy | account / positions / WebSearch | `memory/WEEKLY-REVIEW.md`, บางครั้ง `memory/TRADING-STRATEGY.md` |
| `portfolio` | ad-hoc read only | none beyond live account context | account / positions / orders | ไม่มี |
| `trade` | ad-hoc manual helper | trade log + today's research | account / positions / quote / order | `memory/TRADE-LOG.md` |

## Memory Ownership

| File | Owner workflows | ความหมาย |
|------|----------------|----------|
| `memory/TRADING-STRATEGY.md` | weekly-review (and human edits) | rulebook ถาวรของ bot |
| `memory/TRADE-LOG.md` | market-open, midday, daily-summary, trade | entries/exits/stops/EOD snapshot |
| `memory/RESEARCH-LOG.md` | pre-market, midday addendum | thesis/catalyst/risk context ก่อนเทรด |
| `memory/PROJECT-CONTEXT.md` | human-maintained mostly | mission, platform, repo-level constraints |
| `memory/WEEKLY-REVIEW.md` | weekly-review | scoreboard + lessons + strategy adjustment |

## Trade Lifecycle ที่ repo นี้คาดหวัง
1. pre-market เขียน research entry ของวันนั้นก่อน
2. market-open re-check quote + rules แล้วค่อย buy
3. หลัง fill ต้องตั้ง trailing stop หรือ fixed stop fallback ทันที
4. ทุก trade ถูก append ลง `TRADE-LOG.md`
5. midday scan ตัด loser, tighten winner, ตรวจ thesis break
6. daily-summary จับ EOD snapshot เพื่อให้คำนวณ Day P&L วันถัดไปได้
7. weekly-review สรุป performance และค่อยพิจารณาแก้ strategy

## Operational Gotchas
- Alpaca PDT rule อาจทำให้ stop บางแบบโดน reject ในวันเดียวกับ buy; prompt วาง fallback ladder ไว้แล้ว
- trailing stop ใช้ได้เฉพาะ market hours และโดน overnight gap ได้
- quote ใช้คนละ base URL กับ trading API
- `trail_percent` และ `qty` ถูกส่งเป็น string ใน JSON
- cloud run เป็น fresh clone ถ้าไม่ commit/push state จะหาย
- ClickUp credential ขาดแล้ว bot ไม่ตาย แต่จะ fallback ไปเขียน `DAILY-SUMMARY.md`
- timezone ต้องคิดเป็น Bangkok แต่ Alpaca timestamp เป็น UTC

## Current Snapshot (2026-04-19)
- ยังไม่มี open position ที่บันทึกใน `TRADE-LOG.md`
- ยังไม่มี research note ของวันจริง
- ยังไม่มี weekly scorecard
- แปลว่าความซับซ้อนตอนนี้อยู่ที่ operating contract มากกว่าการวิเคราะห์ performance ย้อนหลัง

## Readiness for AI Handoff
- ถ้าจะทำงานต่อแบบ read-only ให้เริ่มที่ `portfolio` flow
- ถ้าจะรัน strategy จริง ต้องเช็คครบ 4 อย่างก่อน: env vars, paper endpoint, today's research entry, weekly trade count
- ถ้าจะ refactor ต้องระวัง format ของ `memory/*.md` มากกว่า code elegance เพราะ prompt ถัดไปอ่านต่อจากไฟล์พวกนี้

## Verified From
- `/Users/jetsadasomporn/Vscode/Playground/Trader/memory/TRADING-STRATEGY.md`
- `/Users/jetsadasomporn/Vscode/Playground/Trader/memory/TRADE-LOG.md`
- `/Users/jetsadasomporn/Vscode/Playground/Trader/memory/RESEARCH-LOG.md`
- `/Users/jetsadasomporn/Vscode/Playground/Trader/memory/WEEKLY-REVIEW.md`
- `/Users/jetsadasomporn/Vscode/Playground/Trader/routines/pre-market.md`
- `/Users/jetsadasomporn/Vscode/Playground/Trader/routines/market-open.md`
- `/Users/jetsadasomporn/Vscode/Playground/Trader/routines/midday.md`
- `/Users/jetsadasomporn/Vscode/Playground/Trader/routines/daily-summary.md`
- `/Users/jetsadasomporn/Vscode/Playground/Trader/routines/weekly-review.md`
- `/Users/jetsadasomporn/Vscode/Playground/Trader/.claude/commands/*.md`
