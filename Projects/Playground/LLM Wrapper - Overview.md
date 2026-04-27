---
tags: [project, playground, python, aws, bedrock, llm, wrapper]
created: 2026-04-11
updated: 2026-04-11
path: /Users/jetsadasomporn/Vscode/Playground/Wrapper/
---

# LLM Wrapper — AWS Bedrock Data Extraction

> Python wrapper สำหรับดึงข้อมูลจากเอกสารภาษาไทย-English ด้วย LLM บน AWS Bedrock
> Path: `/Users/jetsadasomporn/Vscode/Playground/Wrapper/`

---

## ภาพรวม

Structured data extraction จากเอกสารภาษาไทย/English โดยบังคับให้ LLM ตอบเป็น JSON ตาม Pydantic schema

---

## Tech Stack

```
Python
boto3 + botocore     — AWS Bedrock SDK
pydantic             — Data validation + JSON schema
LiteLLM              — Multi-provider LLM wrapper
requests             — HTTP calls
```

---

## Files หลัก

| File | หน้าที่ |
|------|--------|
| `llm_processor.py` | Core processor — build messages, call Bedrock |
| `LiteLLM.py` | LiteLLM integration |
| `Model_Catalog.py` | รายการ models ที่ใช้ได้ |
| `logging_config.py` | Setup logging |
| `ap_processor.py` | AP (Accounts Payable?) document processor |
| `refresh.py` | Data refresh utility |
| `requirements.txt` | Dependencies |

---

## Architecture

```python
Input (Thai/EN document text)
    ↓
_build_json_only_messages()
    # สร้าง system prompt บังคับ JSON
    # ใส่ Pydantic schema
    # + task instruction
    ↓
AWS Bedrock (via boto3)
    ↓
JSON Output (validated by Pydantic)
```

---

## System Prompt หลักการ

```
กฏบังคับ:
1. Output = JSON เท่านั้น (ห้าม Markdown)
2. ใช้ key ตาม Schema ครบทุกตัว
3. ถ้าไม่มีข้อมูล → null
4. number ต้องเป็น JSON number (ไม่ใช่ string)
```

---

## Pydantic Models ที่มี

- `VendorAddress` — ที่อยู่ vendor (full_address, street, sub_district, district, province)

---

## Test Files

- `test_bedrock.py`
- `test_bedrock_litellm_library.py`
- `test_bedrock_new_function.py`
- `test_bedrock_wrap_bedrock.py`
- `test_node.js`
- `chat-model-tool-calling.ipynb`
- `Wrapper_Optimization.ipynb`

---

เทคนิค prompting → [[Tech/LLM Prompting|LLM Prompting]]

> [!note] Use Case หลัก
> ดึงข้อมูลแบบ structured จากเอกสารราชการ/ธุรกิจภาษาไทย
> เช่น ที่อยู่, ชื่อ vendor, ตัวเลขทางการเงิน
