---
tags: [tech, LLM, AI, prompting, python, bedrock, structured-output]
created: 2026-04-11
updated: 2026-04-11
used-in:
  - [[Playground/Stock Manager - Overview]]
  - [[Playground/LLM Wrapper - Overview]]
---

# LLM Prompting

> เทคนิค prompting จากสองโปรเจคที่ใช้จริง — คนละสไตล์ คนละ use case

---

## แนวคิดหลัก 2 สไตล์

| สไตล์ | ใช้เมื่อ | Project ที่ใช้ |
|-------|---------|--------------|
| **Phase-based** | ถามเชิงวิเคราะห์ ต้องการ reasoning ลึก | Stock Manager |
| **Schema-forced JSON** | ดึงข้อมูลแบบ structured ต้องแม่น 100% | LLM Wrapper |

---

## สไตล์ที่ 1 — Phase-based Prompt (Stock Manager)

แบ่งคำถามเป็น phases ให้ AI ตอบทีละ phase ไม่กระโดด

```
PHASE 0 — SNAPSHOT    (context ก่อน: บริษัทคืออะไร)
PHASE 1 — ธุรกิจ      (revenue model)
PHASE 2 — งบการเงิน   (ตัวเลขจริง YoY 3 ปี)
PHASE 3 — Efficiency  (margin, ROIC, ROE)
PHASE 4 — Moat        (ประเมินทีละ dimension)
PHASE 5 — Valuation   (P/E, EV/EBITDA, DCF)
PHASE 6 — Growth      (TAM, driver)
PHASE 7 — ความเสี่ยง  (bullet จริง ไม่พูดกว้างๆ)
PHASE 8 — ผู้บริหาร   (track record)
PHASE 9 — Peer        (เทียบคู่แข่ง)
PHASE 10 — Verdict    (Buy/Hold/Avoid + คะแนน)
```

**หลักการที่ทำให้ผลดี:**
- บอกตรงๆ ว่า "ใช้ตัวเลขจริง อย่ามั่ว ถ้าไม่มีให้บอก"
- กำหนด output format เป็น table ให้ชัดก่อน
- ระบุ rule ทางการเงินที่ต้องยึด เช่น `ROIC > WACC = สร้างมูลค่า`

---

## สไตล์ที่ 2 — Schema-forced JSON (LLM Wrapper)

บังคับ AI ให้ตอบแค่ JSON ตาม Pydantic schema — ไม่มีข้อความอื่น

### System Prompt Template

```python
system_msg = f"""
คุณเป็น AI ผู้เชี่ยวชาญด้าน Data Extraction จากเอกสารภาษาไทย-English
หน้าที่ของคุณคือวิเคราะห์ข้อความที่ได้รับ และตอบกลับเป็น JSON ที่ถูกต้องตาม RFC 8259 ตาม Schema เท่านั้น

กฏข้อบังคับ (สำคัญมาก):
1. Output ต้องเป็น JSON เท่านั้น (ห้ามมี Markdown เช่น ```json)
2. ต้องใช้ key ตาม Schema นี้อย่างเคร่งครัด ครบทุกตัว
3. หากข้อมูลส่วนใดไม่มี/ไม่แน่ใจ ให้ใส่ค่า null
4. ค่า number ต้องเป็น JSON number (ห้ามใส่เป็น string)

JSON Schema:
{schema_json}
"""
```

### วิธีดึง Schema จาก Pydantic

```python
from pydantic import BaseModel, Field
from typing import Optional
import json

class VendorData(BaseModel):
    tax_id: Optional[str] = Field(None, description="เลขประจำตัวผู้เสียภาษี 13 หลัก")
    vendor_name: Optional[str] = Field(None, description="ชื่อบริษัท/ผู้ขาย")

# ได้ JSON Schema อัตโนมัติ
schema_json = json.dumps(VendorData.model_json_schema(), ensure_ascii=False, indent=2)
```

### Clean JSON จาก LLM (รับมือ Markdown ที่หลุดออกมา)

```python
def clean_json_text(text: str) -> str:
    text = text.strip()
    # ลบ ```json ... ```
    pattern = r"^```(?:json)?\s*(.*?)\s*```$"
    match = re.search(pattern, text, re.DOTALL)
    if match:
        return match.group(1).strip()
    
    # ตัดเฉพาะช่วง { } หรือ [ ]
    first = text.find("{")
    last = text.rfind("}")
    if first != -1 and last > first:
        return text[first:last+1]
    return text
```

---

## API Endpoints ที่ใช้

### AWS Bedrock (Claude)

```python
import boto3
from botocore.config import Config

config = Config(region_name="ap-southeast-1",
                retries={"max_attempts": 3, "mode": "adaptive"})
bedrock = boto3.client("bedrock-runtime", config=config)

body = {
    "anthropic_version": "bedrock-2023-05-31",
    "messages": conversation_messages,
    "system": system_prompt,
    "temperature": 0.0,      # สำคัญ: 0.0 สำหรับ extraction
    "max_tokens": 1000,
}

response = bedrock.invoke_model(
    modelId="anthropic.claude-3-5-sonnet-20240620-v1:0",
    body=json.dumps(body),
    contentType="application/json",
    accept="application/json",
)
```

### ModelHarbor (Custom endpoint)

```python
url = "https://samartinfonet.chat.modelharbor.com/api/chat/completions"
# Models ที่ใช้ได้:
# "qwen/qwen3-235b-a22b-instruct-2507"
# "google/gemini-2.5-pro"
# "openai/gpt-4.1"
# "deepseek-ocr"
```

---

## บทเรียนจาก iteration (v1 → v10)

- `temperature=0.0` เสมอสำหรับ data extraction — ลด hallucination
- ถ้า AI ยัง hallucinate → เพิ่ม "ถ้าไม่มีข้อมูลให้บอกตรงๆ" ใน prompt
- Schema-forced ดีกว่า free-form เมื่อต้องการ field ที่แน่นอน
- Phase-based ดีกว่าถามรวดเดียวเมื่อต้องการ reasoning ลึก

---

## Used In

- [[Playground/Stock Manager - Overview]] — Phase-based
- [[Playground/LLM Wrapper - Overview]] — Schema-forced JSON
