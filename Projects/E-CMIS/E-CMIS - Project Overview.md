---
tags: [project, E-CMIS, proposal, government, microservices]
created: 2026-04-11
updated: 2026-04-11
path: /Users/jetsadasomporn/Vscode/E-CMIS/
status: proposal
---

# E-CMIS — Electronic Case Management Intelligence System

> ระบบจัดการคดีทุจริตสำหรับ **ป.ป.ท. (NACC)**
> Path: `/Users/jetsadasomporn/Vscode/E-CMIS/`
> ผู้เสนอ: **SS Consortium** (Samart Comtech + Smarterware)

---

## วัตถุประสงค์

- Modernize กระบวนการจัดการคดีทุจริตผ่าน Digital Transformation
- ติดตาม สอบสวน และจัดการคดีอย่างมีประสิทธิภาพ
- Analytics & Reporting สำหรับผู้บริหาร
- ความปลอดภัยของข้อมูลในทุกกระบวนการ

---

## Tech Stack

```
Architecture:  Container-based Microservices
Deployment:    Hyperconverged Infrastructure (HCI) + Container Platform
Standard:      ISO/IEC 29110
Security:      OWASP Top 10, VA, Penetration Testing
Environments:  Dev / UAT / Production (แยกกัน)
```

---

## Hardware & Infrastructure

| รายการ | รายละเอียด |
|-------|-----------|
| Server | HPE ProLiant DL385 Gen11 (3 ชุด HCI) |
| Storage | Sangfor HCI Software |
| Container | Sangfor SKE |
| Network | Huawei CloudEngine S6730 |
| Load Balancer | Barracuda ADC 540 |
| UPS | Vertiv Liebert ITA2 10kVA |
| Client | Dell Pro 16 Laptops + Dell Pro 24 Monitors |

---

## 16 Functional Modules

1. Requirement Analysis & Data Mining
2. System Design
3. Hardware/Software Procurement
4. **Complaint Management** — รับเรื่องร้องเรียน
5. **Case Investigation** — ติดตามการสอบสวน
6. **Witness Protection** — ข้อมูลพยาน (secure)
7. **Commission/Committee Management** — การตัดสิน
8. **Disciplinary & Follow-up** — ติดตามหลังการตัดสิน
9. **Warrant & Arrest Management** — เชื่อมกับ enforcement
10. **Legal & Court Process** — ดำเนินคดีในชั้นศาล
11. Data Migration — ย้ายข้อมูลเก่า
12. Analytics & Reporting — Dashboard + KPI
13. Data Integration — เชื่อมระบบภายใน/ภายนอก
14. Central Admin — ตั้งค่าระบบ
15. Testing & Installation — QA + Deploy
16. Training — ถ่ายทอดความรู้

---

## Maintenance & Support

| รายการ | รายละเอียด |
|-------|-----------|
| Warranty | 2 ปี |
| PM | ทุก 3 เดือน (8 ครั้ง) |
| CM SLA | response 1 ชม., on-site 4 ชม. |

---

## Training Plan

| กลุ่ม | จำนวน | Training |
|------|-------|---------|
| Users | 80+ คน | 7 sessions |
| System Admins | 10 คน | 2 วัน |
| Management | 20 คน | 1 วัน |
| IT Support | 5 คน | 3 วัน |

---

> [!note] สถานะ
> อยู่ในขั้น **Proposal** — เอกสารใน `/Vscode/E-CMIS/Proposal/` มีเอกสารสนับสนุนหลายฉบับ (PDF)
