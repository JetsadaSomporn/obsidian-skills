---
title: MOC - STS Workbench
tags:
  - sts
  - moc
  - workbench
type: moc
status: active
created: 2026-04-16
updated: 2026-04-24
links:
  - "[[Projects/STS/STS - Project Overview]]"
  - "[[Projects/STS/Alert Architecture Map]]"
  - "[[Projects/STS/Character Count Policy]]"
  - "[[Projects/STS/Web Board Feature Map]]"
  - "[[Projects/STS/WebBoard_Alert_and_UX_Debugging_Playbook]]"
  - "[[Projects/STS/STS - SLA Alert Hardening Current Contract]]"
  - "[[Projects/STS/STS - Alert Flood Diagnosis Playbook]]"
  - "[[Projects/STS/STS - SLA Downtime Calculation Contract]]"
  - "[[Projects/STS/STS - Requirements Review 2026-04-24]]"
---

# MOC - STS Workbench

> หน้าใช้งานหลักของคลัสเตอร์ STS สำหรับ architecture, bug playbooks, และงาน active รอบล่าสุด

## Core Notes
- [[Projects/STS/STS - Project Overview|STS - Project Overview]]
- [[Projects/STS/STS - Architecture|STS - Architecture]]
- [[Projects/STS/STS - Pending Items|STS - Pending Items]]
- [[Projects/STS/STS - Requirements Review 2026-04-24|STS - Requirements Review 2026-04-24]]

## Web Board / Alert Cluster
- [[Projects/STS/WebBoard_Alert_and_UX_Debugging_Playbook|WebBoard Alert and UX Debugging Playbook]]
- [[Projects/STS/WebBoard_PBM_Alert_Session2_HttpClient_Notifications|WebBoard PBM Alert Session2 HttpClient Notifications]]
- [[Projects/STS/Alert Architecture Map|Alert Architecture Map]]
- [[Projects/STS/Character Count Policy|Character Count Policy]]
- [[Projects/STS/Web Board Feature Map|Web Board Feature Map]]
- [[Projects/STS/STS - Web Board Requirement|STS - Web Board Requirement]]

## SLA / PM / Alert Notes
- [[Projects/STS/STS - SLA Calculation|STS - SLA Calculation]]
- [[Projects/STS/STS - SLA Alert Implementation|STS - SLA Alert Implementation]]
- [[Projects/STS/STS - SLA Alert Noisy Alert Fix 2026-04-23|STS - SLA Alert Noisy Alert Fix 2026-04-23]] ← **ใหม่**
- [[Projects/STS/STS - SLA Alert Hardening Current Contract|STS - SLA Alert Hardening Current Contract]]
- [[Projects/STS/STS - Alert Flood Diagnosis Playbook|STS - Alert Flood Diagnosis Playbook]]
- [[Projects/STS/STS - SLA Downtime Calculation Contract|STS - SLA Downtime Calculation Contract]]
- [[Projects/STS/STS - Alert Routing Current State|STS - Alert Routing Current State]]
- [[Projects/STS/STS - SLA Refresh Worker 12h|STS - SLA Refresh Worker 12h]]

## Recent STS Notes
```dataview
TABLE file.link AS Note, type AS Type, updated AS Updated
FROM "Projects/STS"
WHERE file.name != "MOC - STS Workbench"
SORT updated DESC
```

## Next Promotion Candidates
- ถ้า note ไหนเริ่ม reuse นอก STS ให้ promote ไป [[Tech/MOC - Tech Overview|Tech]]
- ถ้า note ไหนเป็น policy กลาง ให้ทำ alias/path ให้ชัดและใช้ลิงก์แบบ path-based
