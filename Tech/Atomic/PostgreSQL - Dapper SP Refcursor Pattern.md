---
title: "PostgreSQL - Dapper SP Refcursor Pattern"
tags: [tech, postgresql, dapper, csharp, dotnet, pattern]
type: atomic-pattern
ai-context: "How to call PostgreSQL Stored Procedures that return refcursors using Dapper in C#"
usage-pattern: "Requires an explicit transaction (NpgsqlTransaction). Must FETCH from cursor name."
created: 2026-04-16
---

# PostgreSQL - Dapper SP Refcursor Pattern

## Pattern Summary
ใน PostgreSQL เมื่อ SP คืนค่าเป็น `refcursor` เราต้องเรียก 2 ขั้นตอนภายใน Transaction เดียวกัน

## Implementation (C#)
```csharp
using (NpgsqlConnection conn = new NpgsqlConnection(_conn))
{
    conn.Open();
    NpgsqlTransaction tran = conn.BeginTransaction(); // CRITICAL: Must have transaction

    // 1. CALL SP and define cursor name ('resultdata')
    string query1 = string.Format(
        @"CALL ""SCS0112"".""spYourProcedure"" ({0}, '{1}', 'resultdata');",
        p.param1, p.param2
    );

    // 2. FETCH from the defined cursor
    string query2 = @"FETCH ALL FROM resultdata;";

    // Execute CALL
    conn.Query<dynamic>(query1, transaction: tran).Single();

    // Fetch Results
    var data = conn.Query<YourModel>(query2, transaction: tran).ToList();

    tran.Commit();
}
```

> [!danger] Transaction Warning
> If you don't use a transaction, the cursor will be closed immediately after the CALL, and the FETCH will fail.

## บทเรียนจากการใช้จริง (STS-NOC)

- bug ที่เจอครั้งแรก: เรียก FETCH แล้วได้ error "cursor does not exist" — สาเหตุคือลืม BeginTransaction
- PostgreSQL จะปิด cursor ทันทีหลัง CALL จบ ถ้าไม่มี transaction ครอบ — ต่างจาก SQL Server ที่ไม่ต้องทำแบบนี้
- ชื่อ cursor (`resultdata`) ต้องตรงกันทั้งใน CALL และ FETCH — case sensitive

## Related
- [[PostgreSQL Patterns]] (Master)
- [[Concepts/Database Transaction]]
