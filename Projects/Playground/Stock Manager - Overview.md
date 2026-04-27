---
tags: [project, playground, stock, python, AI, analysis]
created: 2026-04-11
updated: 2026-04-11
path: /Users/jetsadasomporn/Vscode/Playground/Stock_Manager/
---

# Stock Manager — AI Stock Analysis Tool

> เครื่องมือวิเคราะห์หุ้นแบบ Fundamental Analysis ด้วย AI
> Path: `/Users/jetsadasomporn/Vscode/Playground/Stock_Manager/`

---

## ภาพรวม

Python script ที่ใช้ AI วิเคราะห์หุ้นแบบ Full Fundamental Analysis พร้อม prompt ที่ออกแบบมาให้ครอบคลุม 10 phases

---

## Files

| File | หน้าที่ |
|------|--------|
| `analyze_stocks.py` | เวอร์ชัน 1 (ต้นแบบ) |
| `analyze_stocks_v4.py` – `v10.py` | พัฒนาต่อเนื่อง |
| `analyze_stocks4.2.py` | branch version 4.2 |
| `GEMINI.md` | Prompt template สำหรับ AI |
| `Instructure.md` | คำแนะนำการใช้งาน |
| `*.txt` (daily logs) | บันทึกผลการวิเคราะห์รายวัน (มี.ค.–เม.ย. 2026) |

---

## Prompt Framework (GEMINI.md)

วิเคราะห์หุ้น **[TICKER]** แบบ 10 Phases:

| Phase | หัวข้อ |
|-------|-------|
| 0 | **Snapshot** — บริษัทคืออะไร Market Cap, ราคา, Earnings date |
| 1 | **Business Model** — Revenue breakdown, B2C/B2B/B2G |
| 2 | **Financial Statements** — Revenue, GP, Net Income, FCF (YoY 3 ปี) |
| 3 | **Profitability & Efficiency** — GM, EBITDA, ROIC, ROE, ROA |
| 4 | **Competitive Moat** — Brand, Network effect, Cost advantage |
| 5 | **Growth** — Revenue growth, expansion plan |
| 6 | **Risk** — Competition, regulation, macro risks |
| 7 | **Valuation** — P/E, P/S, DCF, EV/EBITDA |
| 8 | **Insider & Institutional** — Ownership, insider trades |
| 9 | **Verdict** — Buy/Hold/Avoid + confidence level |

---

## หลักการ

- ตอบเป็นภาษาไทย
- ใช้ตัวเลขจริงเสมอ ไม่มั่ว
- ถ้าไม่มีข้อมูล = บอกตรงๆ
- ROIC > WACC = สร้างมูลค่าจริง
- Net Debt/EBITDA > 3x = หนี้เยอะ

---

เทคนิค prompting → [[Tech/LLM Prompting|LLM Prompting]]

> [!tip] ใช้ร่วมกับ skill
> ใช้ `/stock_analysis` skill ใน Claude Code สำหรับวิเคราะห์หุ้น
