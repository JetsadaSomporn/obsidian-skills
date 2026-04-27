---
title: AI Agents - My Arsenal
tags:
  - moc
  - ai
  - tools
  - agents
  - workflow
created: 2026-04-12
updated: 2026-04-12
---

# AI Agents — My Arsenal

> รวม AI ทุกตัวที่ใช้งานอยู่ — วิธีเรียก, จุดแข็ง, เมื่อไหร่ควรใช้ตัวไหน

---

## Agents

| เรียกด้วย | Note | Plan | จุดเด่น |
|-----------|------|------|---------|
| `Claude` | [[AI - Claude]] | Pro Plan | Code + Context ยาว + Memory |
| `Gemini` | [[AI - Gemini]] | Pro Plan | Multimodal + Google Workspace |
| `codex` | [[AI - Codex]] | Plus Plan | Code generation + OpenAI ecosystem |
| `Copilot` | [[AI - Copilot]] | Education Plan | IDE inline + PR review |
| `qwen` | [[AI - Qwen]] | Free Plan | ภาษาไทย + Chinese corpus |
| `cs` | [[AI - Costrict]] | Free / Open Source | Enterprise AI Agent + Code Review + TDD |
| `Kilo` | [[AI - Kilo]] | — | Agentic coding + model-agnostic |

---

## เลือกตัวไหนดี? — Decision Tree

```
งานหลักคืออะไร?
│
├── เขียน/แก้โค้ดใน project ใหญ่ → Claude (memory + context ยาว)
│
├── Autocomplete ขณะพิมพ์ → Copilot (inline, real-time)
│
├── อ่านรูป / PDF / mockup → Gemini (multimodal แรง)
│
├── Data analysis / Python run → codex (Code Interpreter)
│
├── อธิบายภาษาไทย / ฟรี → qwen
│
├── Code review ลึก + TDD + enterprise / ฟรี → cs (Costrict)
│
├── Agentic coding + เลือก model เอง → Kilo
│
└── ไม่รู้ว่าใช้อะไร → เริ่มที่ Claude ก่อน
```

---

## Tips การใช้ร่วมกัน

- · ใช้ **[[AI - Copilot|Copilot]]** เป็น layer แรก (inline) + **[[AI - Claude|Claude]]** เป็น layer ลึก (reasoning)
- · ใช้ **[[AI - Gemini|Gemini]]** อ่าน spec/PDF แล้วส่ง summary มาให้ **[[AI - Claude|Claude]]** implement
- · ใช้ **[[AI - Qwen|qwen]]** สำหรับ draft ภาษาไทย แล้ว polish ด้วย **[[AI - Claude|Claude]]**
- · ใช้ **[[AI - Costrict|cs]]** review PR ก่อน merge — simulate architect + security + test specialist พร้อมกัน
- · ใช้ **[[AI - Costrict|cs]]** เมื่อโค้ดออกนอก org ไม่ได้ — private deployment ข้อมูลอยู่ใน infra เรา
- · ใช้ **[[AI - Kilo|Kilo]]** เมื่ออยากลอง local model โดยไม่เปลี่ยน workflow ใน VS Code
- · อย่าใช้หลายตัวพร้อมกันสำหรับ task เดียว — เลือกตัวที่ดีที่สุดแล้วไปเลย
