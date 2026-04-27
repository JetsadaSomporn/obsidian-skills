---
tags: [STS, architecture, backend, frontend, database]
created: 2026-04-11
updated: 2026-04-11
---

# STS — Architecture

---

## System Overview

```
┌─────────────────────────────────────┐
│         sts-portal (Next.js 15)     │
│  TypeScript · Tailwind · Radix UI   │
└──────────────┬──────────────────────┘
               │ HTTP (JWT Bearer)
       ┌───────┴────────────────────┐
       │                            │
  ┌────▼──────┐   ┌──────────────┐  ┌──────────────┐
  │ STS-ADM   │   │  STS-NOC     │  │  PMService   │
  │ Auth/User │   │  Jobs/Email  │  │  PM/SLA      │
  │ .NET 8    │   │  WebBoard    │  │  .NET 8      │
  └────┬──────┘   └──────┬───────┘  └──────┬───────┘
       │                 │                 │
  ┌────▼─────────────────▼─────────────────▼───────┐
  │              PostgreSQL (SCS0112)               │
  └─────────────────────────────────────────────────┘
                          │
                   ┌──────▼──────┐
                   │  STS-ALERT  │
                   │  Alert API  │
                   │  .NET 8     │
                   └─────────────┘
```

---

## STS-NOC Controllers

| Controller | หน้าที่ |
|-----------|--------|
| `EmailController` | ส่ง email, log customer feedback |
| `JobController` | จัดการ NOC jobs |
| `PBMController` | PBM (Problem Management) |
| `PMServiceController` | Preventive Maintenance |
| `WebBoardController` | Web Board (Q&A) |

---

## STS-ADM

- ASP.NET Core 8 + **JWT Bearer Auth**
- **EF Core Identity** (User management)
- **Swagger/OpenAPI** (dev docs)
- Upload file support

---

## sts-portal (Frontend)

```json
Dependencies หลัก:
- Next.js 15
- TypeScript
- Tailwind CSS
- Radix UI (components)
- amCharts 5 (charts)
- react-hook-form + @hookform/resolvers
- zustand (state management — stores/)
- next-auth (providers/)
```

### Environments

```bash
npm run local      # .env.loc
npm run uat        # .env.uat
npm run build-dev  # .env.dev
npm run build-uat  # .env.uat
```

---

## SCS-TELEPORT (Legacy)

- **ASP.NET WebForms** (ASPX)
- ระบบเก่าสำหรับ Teleport job monitoring
- Files: `frmJobPM.aspx`, `frmJobMonitor.aspx` ฯลฯ

---

## Database

```
Engine:   PostgreSQL
Schema:   SCS0112
DB name:  sts
Host:     127.0.0.1:5432 (local)
```

### Tables สำคัญ

| Table | หน้าที่ |
|-------|--------|
| `JobPM` | Preventive Maintenance jobs |
| `JobPMActivity` | PM Activity logs |
| `FunctionalLocation` | ไซต์งาน + SLA config |
| `SLAConfig` / `SLAConfigDetail` | SLA configuration |
| `JobStatus` | สถานะงาน (1-13) |
| `LKVDef` / `LKVValue` | Lookup values (Priority ฯลฯ) |
