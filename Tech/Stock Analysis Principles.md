---
title: Stock Analysis Principles
tags:
  - tech
  - finance
  - stock-analysis
  - framework
created: 2026-04-12
updated: 2026-04-24
source: /Users/jetsadasomporn/Vscode/Playground/Stock_Manager/Instructure.md
used-in:
  - [[Projects/Playground/Stock Manager - Overview|Stock Manager]]
---

# Stock Analysis Principles

> Playbook สำหรับให้ AI วิเคราะห์หุ้นในสไตล์ที่ผมต้องการ

## Goal

วิเคราะห์หุ้นแบบ Full Fundamental เพื่อช่วยตัดสินใจลงทุน ไม่ใช่เพื่อเล่า story ให้ดูฉลาด

## Non-Negotiables

- ใช้ตัวเลขจริงเท่านั้น
- ถ้าไม่มีข้อมูล ให้พูดว่า `ไม่พบข้อมูล`
- ห้ามชมลอย ๆ
- ทุก claim สำคัญต้องมีตัวเลขรองรับ
- ต้องพูด valuation ตรง ๆ
- ถ้า peer ดีกว่าชัด ต้องบอก
- ถ้าหุ้นแพง ต้องพูดว่าแพง
- ถ้ายังเป็น story มากกว่าตัวเลข ต้องพูดตรง ๆ

## Preferred Analysis Structure

### Phase 0 — Snapshot

- บริษัททำอะไรในประโยคเดียว
- Market Cap, Sector, Industry
- ราคาปัจจุบัน vs 52w High/Low
- Earnings ล่าสุดและถัดไป

### Phase 1 — Business Model

- รายได้มาจากอะไร
- ลูกค้าเป็น `B2B` / `B2C` / `B2G` / `Mixed`
- recurring vs one-time revenue
- customer concentration
- switching cost

### Phase 2 — Financials

- Revenue
- Gross Profit / Gross Margin
- Operating Income / Operating Margin
- Net Income / Net Margin
- EPS
- Operating Cash Flow
- Free Cash Flow
- Cash
- Debt
- Net Debt / Net Cash

### Phase 3 — Profitability & Efficiency

- Gross Margin
- EBITDA Margin
- ROIC
- ROE
- ROA
- ตอบให้ได้ว่าบริษัทนี้ใช้เงินเก่งหรือไม่

### Phase 4 — Moat

- Brand Power
- Network Effect
- Switching Cost
- Cost Advantage
- Proprietary Technology / IP / Data
- Regulatory Moat

### Phase 5 — Valuation

- P/E
- Forward P/E
- EV/EBITDA
- P/S
- P/FCF
- PEG
- เทียบ peer เสมอ

### Phase 6 — Growth Optionality

- TAM
- growth drivers
- อะไรที่ตลาดยังอาจ price ไม่เต็ม
- อะไรยังเป็นแค่ story

### Phase 7 — Risks

- Competition
- Customer concentration
- Regulation
- Macro sensitivity
- Margin compression
- Valuation reset risk
- dilution
- debt maturity
- key man risk

### Phase 8 — Management

- CEO / CFO
- insider ownership
- capital allocation
- earnings call credibility

### Phase 9 — Peer Comparison

- Revenue Growth
- Gross Margin
- Net Margin
- P/E
- EV/EBITDA
- ROIC

### Phase 10 — Verdict

- 3 จุดเด่น
- 3 จุดเสี่ยง
- เหมาะกับนักลงทุนสายไหน
- final verdict: `น่าสนใจ` / `ดูต่อ` / `หลีกเลี่ยง`

## Key Checks

- รายได้โตจริงไหม
- margin ดีขึ้นไหม
- FCF เป็นบวกไหม
- หนี้อันตรายไหม
- ถ้า `Net Debt/EBITDA > 3x` ให้ระวัง
- ถ้า `ROIC > WACC` ถือว่าสร้างมูลค่าได้จริง

## Interpretation Bias

- ชอบหุ้นที่มีรายได้จริงมากกว่า narrative ล้วน
- ชอบ hidden gem ที่เริ่มมีตัวเลขรองรับ
- hard tech น่าสนใจได้ แต่ต้องมี survivability
- quality ของรายได้และ balance sheet สำคัญพอ ๆ กับ growth
