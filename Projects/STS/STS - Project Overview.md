---
title: STS - Project Overview
tags: [project, sts, enterprise, dotnet, nextjs, postgresql]
type: project-overview
status: active
created: 2026-04-11
updated: 2026-04-16
path: /Users/jetsadasomporn/Vscode/STS/
links:
  - "[[Projects/MOC - Projects Overview]]"
  - "[[Projects/STS/MOC - STS Workbench]]"
  - "[[Projects/STS/WebBoard_Alert_and_UX_Debugging_Playbook]]"
  - "[[Projects/STS/Alert Architecture Map]]"
  - "[[Projects/STS/Character Count Policy]]"
  - "[[Projects/STS/Web Board Feature Map]]"
---

# STS — Smart Telecom System

> ระบบจัดการงาน NOC / PM / Teleport สำหรับองค์กร Telecom
> Path: `/Users/jetsadasomporn/Vscode/STS/`

## Sub-systems

| Module | Stack | หน้าที่ |
|--------|-------|--------|
| **STS-NOC** | C# .NET 8 API | จัดการงาน NOC, Email, Job management |
| **STS-ADM** | C# .NET 8 API | Auth (JWT), User management, Admin |
| **PMService** | C# .NET 8 API | Preventive Maintenance jobs |
| **STS-ALERT** | C# .NET 8 API | ระบบแจ้งเตือน Alert |
| **STS-Common** | C# Library | Shared Models ใช้ร่วมกัน |
| **sts-portal** | Next.js 15 | Frontend portal (web app) |
| **SCS-TELEPORT** | ASPX WebForms | Legacy system (Teleport job monitor) |

## Architecture
- [[Projects/STS/STS - Architecture|Architecture detail]]
- [[Projects/STS/Alert Architecture Map|Alert Architecture Map]]

## Current STS Work Cluster
- [[Projects/STS/MOC - STS Workbench|STS Workbench]]
- [[Projects/STS/WebBoard_Alert_and_UX_Debugging_Playbook|WebBoard Alert Playbook]]
- [[Projects/STS/WebBoard_PBM_Alert_Session2_HttpClient_Notifications|PBM Alert Session 2]]
- [[Projects/STS/Character Count Policy|Character Count Policy]]
- [[Projects/STS/Web Board Feature Map|Web Board Feature Map]]

## Features หลัก
- [ ] Job management (NOC jobs)
- [ ] Preventive Maintenance (PM) + SLA tracking
- [ ] Web Board (Q&A board แบบ Pantip)
- [ ] Alert system
- [ ] Email notifications
- [ ] Export รายงาน Excel
- [ ] Customer Feedback

## Supporting Notes
- [[Projects/STS/STS - SLA Calculation|SLA Calculation (Verified)]]
- [[Projects/STS/STS - SLA Alert Implementation|SLA Alert Implementation]]
- [[Projects/STS/STS - Web Board Requirement|Web Board Requirement]]
- [[Projects/STS/STS - Pending Items|Pending Items]]

## Database Notes
> [!info] Database
> - Host: PostgreSQL (127.0.0.1:5432 local / 203.149.8.100 alert API)
> - DB: `sts`
> - Schema: `SCS0112`
> - User: postgres
