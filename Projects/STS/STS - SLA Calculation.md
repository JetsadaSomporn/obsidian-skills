---
tags: [STS, SLA, database, postgresql, formula, verified]
created: 2026-04-11
updated: 2026-04-11
status: verified
---

# STS — SLA Calculation (Verified 15/15)

> สูตรคำนวณ SLA ที่ verify แล้ว 15/15 records
> อ้างอิง: `/Users/jetsadasomporn/Vscode/STS/PM_SLA_ReverseEngineer.md`

---

## Tables ที่เกี่ยวข้อง

| Table | Schema | บทบาท |
|-------|--------|-------|
| **JobPM** | SCS0112 | หัวงาน PM — เก็บ SLA, StartDate, FixDate |
| **JobPMActivity** | SCS0112 | Activity log — เก็บ ActivityTime (H:M:S) |
| **FunctionalLocation** | SCS0112 | ไซต์งาน — มี SLA default (ชม.) |
| **SLAConfig** | SCS0112 | Config SLA (Type B/C/P) — สำหรับ NOC CM |
| **SLAConfigDetail** | SCS0112 | รายละเอียด: เวลาทำการ, ชม. Normal/OT |
| **JobStatus** | SCS0112 | Status 1-13 |

---

## Columns สำคัญ — JobPM

| Column | Type | หมายเหตุ |
|--------|------|---------|
| JobPMNo | varchar(50) | เลขที่ใบงาน เช่น PMNO260200001 |
| JobStartDate | timestamp | วันเริ่มงาน |
| JobFixDate | timestamp | วันกำหนดแล้วเสร็จ |
| JobCloseDate | timestamp NULL | วันปิดงานจริง |
| **SLA** | integer | **ชั่วโมง = Round(FixDate - StartDate)** |
| JobPriority | integer | 1=Low, 2=Medium, 3=High |
| JobStatusId | integer | 6=OutGoing, 5=Closed, 7=ClosedComplete |

---

## สูตร SLA (Verified ✅)

### PostgreSQL
```sql
ROUND(EXTRACT(EPOCH FROM (JobFixDate - JobStartDate)) / 3600.0)::INTEGER
```

### C# (.NET)
```csharp
(int)Math.Round((jobFixDate - jobStartDate).TotalHours)
```

### JavaScript (Frontend)
```javascript
Math.round((jobFix.getTime() - jobStart.getTime()) / (1000 * 60 * 60))
```

---

## ActivityTime Format

- Column: `JobPMActivity.ActivityTime`
- Format: `"H:MM:SS"` — exact diff (ไม่ round ต่างจาก SLA)
- เก็บเป็น varchar

---

## HoldType Values

| Value | ความหมาย |
|-------|---------|
| 0 | ไม่ hold |
| 4 | Hold by SCS |
| 10 | Hold by Customer |
| 11 | Hold by Accident |

---

## JobStatus Values สำคัญ

| ID | ความหมาย |
|----|---------|
| 5 | Closed |
| 6 | OutGoing (กำลังดำเนินการ) |
| 7 | ClosedComplete |

---

> [!warning] ยังไม่ implement
> - `RealDowntimeInt` — ชม.จริงตอนปิดงาน
> - `TPDowntimeInt` — Third Party Downtime
> - `SLAPercent` — เปอร์เซ็นต์ SLA

---

## Stored Procedures

- `spPMGetJobList` — ดึงรายการสำหรับแสดงหน้าจอ (มี pagination)
- `spJobPMExportSearch` — Export Excel (❌ ยังไม่มี SP ใน DB — [[STS - Pending Items|ดู Pending]])
