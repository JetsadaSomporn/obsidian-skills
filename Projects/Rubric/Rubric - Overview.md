---
tags: [project, rubric, essay, evaluation, python]
created: 2026-04-11
updated: 2026-04-11
path: /Users/jetsadasomporn/Vscode/Rubric/
---

# Rubric — Essay Evaluation System

> ระบบประเมิน essay ด้วย rubric
> Path: `/Users/jetsadasomporn/Vscode/Rubric/`

---

## Files

| File | หน้าที่ |
|------|--------|
| `Rubric.json` | เกณฑ์การให้คะแนน (rubric definitions) |
| `essay.json` | ข้อมูล essay ที่ต้องประเมิน |
| `transfer.py` | แปลง JSON array → JSON Lines (JSONL) |

---

## transfer.py — Utility

แปลงไฟล์ `essay.json` (JSON array) เป็น `essay.jsonl` (JSON Lines format)

```python
# การทำงาน:
# 1. อ่าน essay.json (JSON array)
# 2. แปลงเป็น JSONL (1 record ต่อบรรทัด)
# 3. เขียนออกเป็น essay.jsonl
```

> JSONL ใช้ง่ายกว่าสำหรับ batch processing

---

> [!note] สถานะ
> Experimental — ยังอยู่ในขั้นทดลอง
