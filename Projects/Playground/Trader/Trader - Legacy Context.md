---
title: Trader - Legacy Context
tags:
  - trader
  - legacy
  - stock-analysis
  - archive
  - playground
type: context
status: active
created: 2026-04-19
updated: 2026-04-19
links:
  - "[[Projects/Playground/Trader/Trader - Overview]]"
  - "[[Projects/Playground/Trader/MOC - Trader Workbench]]"
  - "[[Projects/Playground/Stock Manager - Overview]]"
---

# Trader - Legacy Context

## Why `legacy_data/` Exists
โฟลเดอร์นี้ไม่ใช่แกน runtime ของ trading bot ปัจจุบัน แต่เป็น archive จากงานวิเคราะห์หุ้นยุคก่อนที่ workspace นี้เคยใช้เป็น stock-screen / fundamental-analysis playground

## ของใน `legacy_data/`

| Path group | ความหมาย | สถานะ |
|-----------|----------|-------|
| `analyze_stocks*.py`, `analyze_stock3.py` | Python scripts สำหรับ screen/universe ranking/report generation | archive |
| `Instructure.md`, `GEMINI.md` | prompt contract สำหรับรายงานวิเคราะห์หุ้นภาษาไทย | archive |
| `*.txt` dated reports | output รายวันของ stock analysis หลายเวอร์ชัน | archive |
| `.venv/`, `__pycache__/` | local Python environment และ cache เก่า | noise/archive |

## Boundary ที่ต้องจำ
- trading bot ปัจจุบันไม่ได้ import หรือเรียก Python scripts ใน `legacy_data/`
- current repo flow ใช้ Claude prompts + Alpaca wrapper + memory logs เป็นหลัก
- legacy scripts ยังสะท้อน investing taste ของ operator ได้ เช่นชอบ quality growth, hidden gems, hard tech with real revenue, และไม่ชอบหุ้น pre-revenue fantasy
- ถ้าจะหยิบไอเดียหุ้นจาก archive มาใช้ ต้องแปลงมันเป็น research entry ใหม่ใน `memory/RESEARCH-LOG.md` ก่อน ไม่ใช่ถือว่าเป็น trade-ready thesis ทันที

## ความสัมพันธ์กับโปรเจกต์อื่น
- `[[Projects/Playground/Stock Manager - Overview]]` คือ note ที่อธิบายโลก stock-analysis เดิมในเชิง tool/prompt
- `Trader` คือ evolution ที่เพิ่ม execution layer และ operating discipline เข้ามา
- มองอีกแบบคือ `Stock Manager` = research/generation toolset, `Trader` = execution scaffold + memory discipline

## Source Lineage
- `Trading.pdf` คือ blueprint ต้นทางจาก Nate Herk สำหรับ autonomous trading bot บน Claude Code
- repo นี้ปรับจาก guide นั้น 3 จุดหลัก: ตัด Perplexity wrapper ออก, ใช้ Bangkok timezone, และย้ำ paper trading only
- git history ที่เห็นตอนนี้มีแค่ initial commit และ scaffold commit แปลว่าการ migrate จากของเก่ามาสู่ trading bot ยังใหม่มากในเชิง repo history

## What Still Matters from Legacy
- ภาษาและน้ำเสียงการวิเคราะห์หุ้นแบบ blunt, fact-based
- universe selection logic และ risk taste ของ operator
- prompt contracts ที่บังคับให้บอก `ไม่พบข้อมูล` แทนการมั่ว

## What Should Be Ignored by Default
- daily stock reports เก่า ๆ ถ้าไม่ได้เกี่ยวกับ ticker ที่กำลังจะเทรดตอนนี้
- Python env / cache ใน `legacy_data/.venv` และ `__pycache__`
- assumption ว่า repo นี้ยังขับด้วย Python script; ตอนนี้ไม่ใช่แล้ว

## Verified From
- `/Users/jetsadasomporn/Vscode/Playground/Trader/legacy_data/Instructure.md`
- `/Users/jetsadasomporn/Vscode/Playground/Trader/legacy_data/analyze_stocks_v10.py`
- `/Users/jetsadasomporn/Vscode/Playground/Trader/legacy_data/*.txt`
- `/Users/jetsadasomporn/Vscode/Playground/Trader/Trading.pdf`
- `/Users/jetsadasomporn/Vscode/Playground/Trader/.git` history as observed on 2026-04-19
