---
tags: [STS, pending, todo, bugs]
created: 2026-04-11
updated: 2026-04-11
---

# STS — Pending / ค้างอยู่

> รายการที่ยังไม่เสร็จ / รอดำเนินการ

---

## ⏸ spJobPMExportSearch — Export PM เป็น Excel

> อ้างอิง: `/Users/jetsadasomporn/Vscode/STS/PENDING_spJobPMExportSearch.md`

### ปัญหา

```
C# PMServiceController  → JobPMExportSearch()  ✅ มีแล้ว
C# PMRepository         → JobPMExportSearch()  ✅ มีแล้ว  
PostgreSQL SP           → spJobPMExportSearch  ❌ ไม่มี!
```

ถ้า user กดปุ่ม Export → **error ทันที**

### Endpoint
`/PMService/JobPMExportSearch`

### Parameters ที่ต้องรับ

```sql
IN  p_startdate  text    -- วันเริ่มต้น กรองวันเปิดงาน
IN  p_enddate    text    -- วันสิ้นสุด กรองวันเปิดงาน
INOUT result     refcursor
```

C# เรียกว่า:
```csharp
CALL "SCS0112"."spJobPMExportSearch"('{p.pstartdate}', '{p.penddate}', 'result');
```

### Columns ที่ต้อง Return (ตาม XLS เก่า)

| # | Column | ที่มา |
|---|--------|-------|
| 0 | สถานะ | `jt."JobStatusName"` |
| 1 | โครงการ | `pj."ProjectName"` |
| 2 | ไซต์งาน | `fc."FunctionalLocationName"` |
| 3 | ประเภทงาน | `lk."LKVValueTh"` |
| 4 | เลขที่ใบงาน | `j."JobPMNo"` |

> [!todo] สร้าง SP `spJobPMExportSearch` ใน PostgreSQL schema SCS0112

---

## ยังไม่ Implement ใน JobPM

- `RealDowntimeInt` — ชม.จริงตอนปิดงาน
- `TPDowntimeInt` — Third Party downtime
- `TPHoldDowntimeInt` — Third Party hold
- `RealDowntime` — format H:M:S ตอนปิด
- `SLAPercent` — เปอร์เซ็นต์ SLA

---

## SLA Alert (งานนิว Req#7 + Req#30)

แผนงานละเอียด → [[STS - SLA Alert Implementation]]

สถานะ (2026-04-12):
- `fCalsNetSLAPM.sql` ยังไม่ได้รันใน DB ❌ → ทำ Step 0 ก่อน
- Worker / Bell SLA ยังไม่มี ❌

เริ่มทำ: 16/4/2026

---

## Web Board

ฟีเจอร์ใหม่ที่ต้องสร้าง — ดู [[STS - Web Board Requirement]]
