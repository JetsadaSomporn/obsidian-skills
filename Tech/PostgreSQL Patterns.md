---
tags: [tech, postgresql, sql, dapper, dotnet, patterns]
created: 2026-04-11
updated: 2026-04-11
used-in: [[STS/STS - Project Overview]]
---

# PostgreSQL Patterns

> Pattern ที่ใช้จริงใน STS — copy ได้เลยไม่ต้อง reverse engineer ซ้ำ

---

## 1. เรียก Stored Procedure แบบ refcursor (Dapper)

Pattern นี้ใช้ทุก SP ใน STS schema `SCS0112`

```csharp
using (NpgsqlConnection conn = new NpgsqlConnection(_conn))
{
    conn.Open();
    NpgsqlTransaction tran = conn.BeginTransaction();

    // Step 1: CALL SP (เปิด cursor ชื่อ 'resultdata')
    string query1 = string.Format(
        @"CALL ""SCS0112"".""spYourProcedure"" ({0}, '{1}', 'resultdata');",
        p.param1, p.param2
    );

    // Step 2: FETCH จาก cursor
    string query2 = @"FETCH ALL FROM resultdata;";

    // Execute CALL
    conn.Query<dynamic>(query1, param: null,
        commandType: CommandType.Text, transaction: tran).Single();

    // Fetch results
    var data = conn.Query<YourModel>(query2, param: null,
        commandType: CommandType.Text, transaction: tran).ToList();

    tran.Commit();
}
```

> [!warning] ต้องมี transaction เสมอ
> refcursor ใน PostgreSQL ต้องอยู่ใน transaction เดียวกัน ถ้าไม่มี tran จะ error

---

## 2. สูตรคำนวณ SLA (Verified ✅ 15/15)

```sql
-- PostgreSQL: คำนวณชั่วโมง SLA (rounded)
ROUND(EXTRACT(EPOCH FROM (JobFixDate - JobStartDate)) / 3600.0)::INTEGER

-- C#
(int)Math.Round((jobFixDate - jobStartDate).TotalHours)

-- JavaScript
Math.round((jobFix.getTime() - jobStart.getTime()) / (1000 * 60 * 60))
```

ดูรายละเอียดเพิ่มเติม → [[STS/STS - SLA Calculation]]

---

## 3. fCalcNetSLA — Function ที่ซับซ้อนที่สุด

Function นี้คำนวณ Net SLA แบบเต็ม (หัก Hold, กรอง Business Hours)

```sql
-- เรียกใช้:
SELECT * FROM "SCS0112"."fCalcNetSLA"(p_job_id := 123);

-- Return columns หลัก:
-- "NetSla"       INTEGER   -- วินาที SLA สุทธิ (หลังหัก Hold + กรอง BH)
-- "SlaStatus"    VARCHAR   -- 'Within SLA' / 'Miss SLA'
-- "SlaPercent"   NUMERIC   -- % ที่ใช้ไป (NULL ถ้าเกิน)
-- "IsOverSla"    BOOLEAN   -- TRUE = Miss SLA
-- "Over"         INTEGER   -- วินาทีที่เกิน (NULL ถ้าไม่เกิน)
```

---

## 4. Connection String Pattern

```json
// appsettings.json
{
  "ConnectionStrings": {
    "ConnString": "Host=127.0.0.1;Port=5432;Database=sts;UserId=postgres;Password=S@mart@2024;"
  }
}
```

```csharp
// ใน Repository constructor
_conn = config.GetConnectionString("ConnString");
```

---

## 5. SP Naming Convention ใน SCS0112

| Prefix | ความหมาย | ตัวอย่าง |
|--------|---------|---------|
| `sp` | Stored Procedure | `spPMFunctionalSearch` |
| `f` | Function (returns table) | `fCalcNetSLA` |
| `spPM` | PM-related SP | `spPMGetJobList`, `spJobPMExportSearch` |

---

## 6. Export Excel Pattern (ClosedXML)

```csharp
using ClosedXML.Excel;
// หรือ
using OfficeOpenXml; // EPPlus

// ใช้ใน PMRepository สำหรับ export
```

---

## Used In

- [[STS/STS - Project Overview]] — ทุก service ใช้ pattern นี้
- [[STS/STS - Pending Items]] — `spJobPMExportSearch` ยังไม่มี SP
