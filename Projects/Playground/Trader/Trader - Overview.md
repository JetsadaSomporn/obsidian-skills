---
title: Trader - Overview
tags:
  - project
  - playground
  - trader
  - alpaca
  - claude-code
  - finance
type: project-overview
status: active
created: 2026-04-19
updated: 2026-04-19
path: /Users/jetsadasomporn/Vscode/Playground/Trader/
links:
  - "[[Projects/MOC - Projects Overview]]"
  - "[[Projects/Playground/Trader/MOC - Trader Workbench]]"
  - "[[Projects/Playground/Trader/Trader - Architecture]]"
  - "[[Projects/Playground/Trader/Trader - Workflow and State]]"
  - "[[Projects/Playground/Trader/Trader - Legacy Context]]"
  - "[[Areas/Finance - Overview]]"
---

# Trader - Overview

> โปรเจกต์นี้คือ orchestration repo สำหรับ autonomous trading bot ที่ให้ Claude Code เป็นตัวคิดและลงมือ, ใช้ Alpaca paper account สำหรับ order execution, และใช้ Git markdown files เป็น memory กลาง

## One-Line Read
- เป้าหมาย: ชนะ S&P 500 ใน challenge window
- บัญชี: Alpaca PAPER ประมาณ $10,000
- instrument: หุ้นเท่านั้น ไม่มี options
- สไตล์: aggressive but disciplined swing trading
- persistence model: ทุก state สำคัญถูกเก็บใน `memory/*.md` แล้ว commit กลับ repo

## Current State (Verified 2026-04-19)
- สถานะตอนนี้ยังเป็น scaffold พร้อมใช้งาน มากกว่าระบบที่มี trading history จริง
- `memory/TRADE-LOG.md` ยังมีแค่ baseline day 0: พอร์ต $10,000 เงินสด 100% ไม่มี position
- `memory/RESEARCH-LOG.md` ยังเป็น template format ไม่มี pre-market entry จริง
- `memory/WEEKLY-REVIEW.md` ยังเป็น template ไม่มีผลลัพธ์สะสม
- ความเป็น autonomous จริงจะเกิดก็ต่อเมื่อมีการตั้ง Claude cloud routines + env vars นอก repo แล้วรันจริงตาม schedule

## Repo Pieces ที่มีผลจริง

| Path | บทบาท | ใช้เมื่อไร |
|------|--------|-----------|
| `CLAUDE.md` | rulebook หลักของ agent | ทุก session |
| `memory/` | state ที่ bot อ่าน/เขียน | ทุก workflow |
| `scripts/alpaca.sh` | wrapper สำหรับ Alpaca account / orders / quotes | ตอนดึง state และยิงคำสั่งเทรด |
| `scripts/clickup.sh` | wrapper สำหรับแจ้งเตือน ClickUp chat | ตอนส่งสรุปหรือ alert |
| `routines/*.md` | prompt สำหรับ cloud-scheduled runs | production path |
| `.claude/commands/*.md` | prompt mirror สำหรับ local/manual run | local testing / operator-driven |
| `env.template` | ตัวอย่างตัวแปรแวดล้อม | local setup |
| `Trading.pdf` | guide ต้นทางจาก Nate Herk | ใช้อ่านที่มาและ intent |
| `legacy_data/` | stock-analysis archive จากยุคก่อน | reference only |

## ระบบนี้เป็นอะไร และไม่เป็นอะไร

### เป็น
- prompt-driven trading system
- shell-wrapper based broker/chat integration
- git-backed memory model
- schedule-first workflow ที่แยก pre-market / open / midday / close / weekly ชัดเจน

### ไม่เป็น
- ไม่ใช่ backtester
- ไม่ใช่ Python trading engine ที่มี strategy class หรือ broker abstraction
- ไม่ใช่ quantitative model repo ที่มี factor pipeline
- ไม่ใช่ service/app ที่มี database หรือ API server ของตัวเอง
- ไม่ใช่ options bot

## Core Constraints ที่ห้ามหลุด
- paper trading เป็น default และห้าม flip เป็น live เอง
- max 5-6 positions
- max 20% ต่อ position
- max 3 new trades ต่อสัปดาห์
- 75-85% capital deployed
- ทุก position ต้องมี stop จริง
- cut losers ที่ -7%
- ถ้า thesis ไม่อยู่หรือ sector พัง ต้องออก

## Readiness Verdict
- good enough สำหรับเป็น operating scaffold และ handoff ให้ AI ตัวอื่นมาทำงานต่อ
- ยังไม่พอจะเรียกว่า proven trading system เพราะยังไม่เห็นร่องรอย research log, trade log, หรือ weekly review ที่เกิดจากรันจริง
- ถ้า AI ตัวต่อไปต้องลงมือ ให้เริ่มจากอ่าน `CLAUDE.md` + `memory/*` แล้วเช็ค env / account state ก่อนทุกครั้ง

## Verified From
- `/Users/jetsadasomporn/Vscode/Playground/Trader/README.md`
- `/Users/jetsadasomporn/Vscode/Playground/Trader/CLAUDE.md`
- `/Users/jetsadasomporn/Vscode/Playground/Trader/memory/TRADING-STRATEGY.md`
- `/Users/jetsadasomporn/Vscode/Playground/Trader/memory/TRADE-LOG.md`
- `/Users/jetsadasomporn/Vscode/Playground/Trader/memory/RESEARCH-LOG.md`
- `/Users/jetsadasomporn/Vscode/Playground/Trader/memory/WEEKLY-REVIEW.md`
- `/Users/jetsadasomporn/Vscode/Playground/Trader/.git` history as observed on 2026-04-19
