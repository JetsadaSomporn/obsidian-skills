---
title: Projects - Overview
tags:
  - moc
  - projects
  - overview
type: moc
status: active
created: 2026-04-11
updated: 2026-04-19
links:
  - "[[Home]]"
  - "[[Projects/STS/STS - Project Overview]]"
  - "[[Projects/Playground/Trader/MOC - Trader Workbench]]"
---

# Projects - Overview

> แผนที่รวมของ project ทั้งหมดที่กำลังทำ เคยทำ หรือกำลังทดลอง

## Project Index
```dataview
TABLE file.link AS Project, status AS Status, updated AS Updated
FROM "Projects"
WHERE file.name != "MOC - Projects Overview"
SORT updated DESC
```

## Project Families
- [[Projects/STS/STS - Project Overview|STS]]
- [[Projects/E-CMIS/E-CMIS - Project Overview|E-CMIS]]
- [[Projects/MPI/MPI - Overview|MPI]]
- [[Projects/Playground/Stock Manager - Overview|Stock Manager]]
- [[Projects/Playground/Trader/MOC - Trader Workbench|Trader]]
- [[Projects/Playground/LLM Wrapper - Overview|LLM Wrapper]]
- [[Projects/Rubric/Rubric - Overview|Rubric]]
- [[Projects/Playground/PaymentTracker/Payment Tracker - Overview|Payment Tracker]]

## STS Cluster
- [[Projects/STS/STS - Project Overview]]
- [[Projects/STS/STS - Architecture]]
- [[Projects/STS/STS - Web Board Requirement]]
- [[Projects/STS/STS - SLA Alert Implementation]]
- [[Projects/STS/STS - SLA Calculation]]
- [[Projects/STS/STS - Pending Items]]
- [[Projects/STS/MOC - STS Workbench|STS Workbench]]
- [[Projects/STS/WebBoard_Alert_and_UX_Debugging_Playbook|WebBoard Alert Playbook]]
- [[Projects/STS/Alert Architecture Map|Alert Architecture Map]]
- [[Projects/STS/Web Board Feature Map|Web Board Feature Map]]
- [[Projects/STS/Character Count Policy|Character Count Policy]]

## Promotion Rule
- ถ้า note ยังเป็น execution detail ของงานเดียว ให้อยู่ใน project
- ถ้าเริ่มใช้ซ้ำได้หลายงาน ให้ promote ไป [[Tech/MOC - Tech Overview|Tech]]
- ถ้าเป็นเรื่องดูแลต่อเนื่อง ไม่ได้มี finish line ชัด ให้ย้ายแนวคิดไป [[Areas/MOC - Areas Overview|Areas]]
