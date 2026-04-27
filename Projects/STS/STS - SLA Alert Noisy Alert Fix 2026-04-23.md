---
tags: [STS, alert, SLA, bug-fix, noisy-alert, dedup]
created: 2026-04-23
updated: 2026-04-23
---

# STS — SLA Alert Noisy Alert Fix (2026-04-23)

Related:
- [[Projects/STS/STS - SLA Alert Implementation|STS - SLA Alert Implementation]]
- [[Projects/STS/STS - SLA Alert Bug Log|STS - SLA Alert Bug Log]]
- [[Tech/CSharp .NET API Patterns|C# .NET API Patterns]]

---

## สิ่งที่ผิด (Root Cause)

SLA Alert แจ้งเตือนแทบทุกเสี้ยววินาที จนกลายเป็น **Noisy Alert** — ฐานข้อมูล AlertQ/AlertTo/AlertLogs โตผิดปกติ

### อาการที่พบ

- 1 job ต่อ 4 ชม. → สร้าง ~48 alert rows (ทุก 5 นาที × 48 รอบ)
- 20 jobs → ~6,720+ rows ใน 4 ชม.
- User อ่านแจ้งเตือนแล้ว → แจ้งใหม่อีก
- Action string เปลี่ยนทุกรอบ (`SLA-WARN|J001|S001|30` → `SLA-WARN|J001|S001|25`) → dedup จับไม่ได้

### Root Cause 4 จุด

| # | จุดที่ผิด | ผลกระทบ |
|---|---|---|
| 1 | **Merge conflict 4 จุด** ใน `SLAMonitorWorker.cs` ยังไม่ resolve | code ไม่ compile ได้, feature ที่ควรทำงานไม่ทำงาน |
| 2 | **AlertRepository dedup ใช้ exact Action match** (`q."Action" = @Action`) แต่ Action เปลี่ยนทุกรอบ (minutes เปลี่ยน) | dedup ไม่ทำงาน → ยิงซ้ำทุกรอบ |
| 3 | **AlertRepository dedup เช็คเฉพาะ `StatusTo = 'Unread'`** | user อ่านแล้ว → dedup หาไม่เจอ → ยิงใหม่ |
| 4 | **PM query ไม่มี age filter** | สแกน PM เก่าๆ ทุกรอบ → noise เพิ่ม |

### Flow ข้อมูลที่ทำให้เละ

```
SLAMonitorWorker (ทุก 5 นาที)
  └── CALL spAlertInsert
        ├── INSERT AlertTemp_QBuffer (buffer)
        ├── CALL fAlertQInsertBatch()
        │     ├─ pg_sleep(3)
        │     └─ INSERT AlertQ  ← ทุกรอบ = row ใหม่เสมอ
        └── วนลูปทุก recipient
              ├── INSERT AlertTemp_ToBuffer (buffer)
              └── CALL fAlertToInsertBatch()
                    ├─ pg_sleep(3)
                    ├─ INSERT AlertTo  ← ทุกรอบ = row ใหม่เสมอ
                    └─ INSERT AlertLogs
```

`spAlertInsert` **ไม่มี dedup ภายใน** → ทุกครั้งที่ถูกเรียน = INSERT ใหม่เสมอ

---

## วิธีการแก้ (What We Did)

### แก้ที่ `SLAMonitorWorker.cs` อย่างเดียว — ไม่ยุ่งกับ DB/SP/AlertRepository

### Task 1: Resolve Merge Conflict 4 จุด

เลือกเวอร์ชัน **stashed** (มี IOptionsMonitor + configurable options) แทน upstream (มี hardcoded values)

| จุด | สิ่งที่เลือก |
|---|---|
| จุดที่ 1 | `SYSTEM_USER_ID` constant |
| จุดที่ 2 | Constructor ที่รับ `IOptionsMonitor<SlaAlertMonitorOptions>` |
| จุดที่ 3 | Query parameters ครบ (WarnPct, MaxAlertsPerScan, CmAlertMaxAgeHours, PmAlertMaxAgeHours) |
| จุดที่ 4 | Log ครบ (Count, MaxAlertsPerScan, WarnPct) |

### Task 2: เพิ่ม In-Memory Dedup (ConcurrentDictionary)

```csharp
// เพิ่ม field ระดับ class
private static readonly ConcurrentDictionary<string, byte> _sentAlerts = new();

// เพิ่ม logic ก่อนยิง alert (ใน foreach loop)
var dedupKey = $"{job.JobType}:{job.JobId}:{labelCheck}";
// เช่น "CM:123:SLA-WARN" หรือ "PM:456:SLA-OVER"

if (!_sentAlerts.TryAdd(dedupKey, 0))
{
    // TryAdd fail = key มีอยู่แล้ว = ส่งไปแล้ว → skip
    continue;
}
// TryAdd สำเร็จ = ยังไม่เคยส่ง → ยิง alert
```

**ทำไมใช้ ConcurrentDictionary:**
- `TryAdd` เป็น atomic operation → thread-safe โดยไม่ต้องใช้ lock
- O(1) lookup → เร็วมาก
- Key ใช้ fixed string (`JobType:JobId:Label`) → ไม่เปลี่ยนทุกรอบเหมือน Action string

### Task 3: เพิ่ม PM Age Filter

```sql
-- เพิ่มใน PM WHERE clause
AND pm."CreateDate" >= NOW() - (@PmAlertMaxAgeHours * INTERVAL '1 hour')
```

CM มีอยู่แล้ว แต่ PM ไม่มี → เพิ่มให้เหมือนกัน (default 24 ชม.)

---

## ผลลัพธ์หลังแก้ (Result)

### Dedup 3 ชั้นที่ทำงานอยู่

```
Scan ทุก 5 นาที
  │
  ├─ ชั้น 1: SQL NOT EXISTS (DB level)
  │   → เช็ค AlertQ ว่ามี SLA-WARN% / SLA-OVER% สำหรับ JobId นั้นหรือยัง
  │   → ไม่สน StatusTo (อ่านแล้วก็ไม่ยิงซ้ำ)
  │   → Safety net: กัน worker restart แล้ว memory หาย
  │
  ├─ ชั้น 2: ConcurrentDictionary (Memory level)
  │   → Key: "CM:123:SLA-WARN" / "PM:456:SLA-OVER"
  │   → O(1) lookup, thread-safe
  │   → เร็วสุด: ไม่ต้อง query DB
  │
  └─ ชั้น 3: Advisory Lock (Instance level)
      → pg_try_advisory_lock → ป้องกันหลาย instance ยิงพร้อมกัน
```

### Before vs After (1 job, 3 recipients, 4 ชม.)

| | ก่อนแก้ | หลังแก้ |
|---|---|---|
| **AlertQ** | ~48 rows | **2 rows** (WARN 1 + OVER 1) |
| **AlertTo** | ~144 rows | **6 rows** (2 × 3 recipients) |
| **AlertLogs** | ~144 rows | **6 rows** |
| **รวม** | **~336 rows** | **14 rows** |

**ลด ~96%**

### Behavior หลังแก้ (Timeline ของ 1 Job)

```
เวลา 00:00  เปิดงาน → ไม่มีอะไรเกิดขึ้น
เวลา 03:15  SLA = 80% → ยิง SLA-WARN 1 ครั้ง ✓
เวลา 03:20  SLA = 83% → สแกนเจอแต่ memory บอกส่งไปแล้ว → ข้าม
เวลา 03:25  SLA = 87% → สแกนเจอแต่ memory บอกส่งไปแล้ว → ข้าม
เวลา 04:05  SLA = 102% → IsOverSla=true → ยิง SLA-OVER 1 ครั้ง ✓
เวลา 04:10  SLA = 110% → สแกนเจอแต่ memory บอกส่ง OVER ไปแล้ว → ข้าม
เวลา 04:15  User อ่านแจ้งเตือน → สแกน → ข้าม (ไม่สน StatusTo)
เวลา 06:00  ปิดงาน → ไม่เข้าเงื่อนไข → ไม่สแกน

รวม: Job นี้แจ้งเตือนทั้งหมด 2 ครั้ง ตลอดอายุงาน
```

### สิ่งที่ไม่เปลี่ยน

- ยังสแกนทุก **5 นาที** (ตาม config)
- ยังแจ้งเตือนที่ **80% (WARN)** และ **>100% (OVER)**
- ยังใช้ `spAlertInsert` เดิม
- ไม่แตะ DB schema / stored procedure / AlertRepository / Frontend

---

## สิ่งที่ได้เรียนรู้ (Lessons Learned)

### 1. Merge conflict ที่ค้างอยู่ = feature พัง
- code ไม่ compile → feature ที่ควรทำงานอยู่ไม่ทำงาน
- ต้อง resolve ทันที อย่าปล่อยค้าง

### 2. Dedup key ต้องเป็น fixed value ไม่ใช่ dynamic string
- ❌ ใช้ Action string: `SLA-WARN|J001|S001|30` → เปลี่ยนทุกรอบ (minutes เปลี่ยน)
- ✅ ใช้ fixed key: `CM:123:SLA-WARN` → ไม่เปลี่ยน

### 3. Dedup ต้องไม่เช็ค StatusTo
- ถ้าเช็คแค่ `StatusTo = 'Unread'` → user อ่านแล้ว → dedup หาไม่เจอ → ยิงใหม่
- Dedup ต้องตอบคำถามว่า "เคยส่งหรือยัง?" ไม่ใช่ "ยังไม่อ่านอยู่หรือไม่?"

### 4. หลายชั้น dedup ดีกว่าชั้นเดียว
- **ชั้น 1: Memory** (fast, O(1)) → ConcurrentDictionary
- **ชั้น 2: DB** (persistent, กัน restart) → SQL NOT EXISTS
- **ชั้น 3: Lock** (concurrent, กัน multi-instance) → advisory lock
- แต่ละชั้นเป็น safety net ของอีกชั้น

### 5. spAlertInsert ใช้ buffer + pg_sleep(3) x2 = 6+ วินาที
- race condition ได้ถ้ามี caller หลายตัวยิงพร้อมกัน
- ต้องมี dedup ที่ caller ก่อนเรียก spAlertInsert

### 6. Worker Enabled = false ต้องเพิ่ม appsettings.json
- default `Enabled = false` + ไม่มี config section → worker ไม่ทำงาน
- ต้องเพิ่ม `"SlaAlertMonitorOptions": { "Enabled": true, ... }` ใน appsettings.json ก่อน deploy

### 7. SLAMonitorWorker เป็นแหล่งเดียวที่ยิง SLA alert
- ตรวจทั้ง codebase แล้ว → ไม่มี code อื่นยิง `SLA-WARN` / `SLA-OVER`
- ดังนั้นแก้ที่จุดเดียว = แก้จบ

---

## Files Changed

| ไฟล์ | สิ่งที่แก้ |
|---|---|
| `STS-NOC/Workers/SLAMonitorWorker.cs` | resolve merge conflict + ConcurrentDictionary dedup + PM age filter |

### Config Model (ไม่ได้แก้, อ้างอิง)
- `STS-NOC/Models/SlaAlertMonitorOptions.cs` — Enabled, IntervalMinutes, WarnPercent, etc.
- `STS-NOC/Program.cs:83` — Configure SlaAlertMonitorOptions
- `STS-NOC/Program.cs:86` — AddHostedService SLAMonitorWorker

### Proposal & Task (สร้างใหม่)
- `.cospec/plan/changes/fix-sla-alert-dedup/proposal.md`
- `.cospec/plan/changes/fix-sla-alert-dedup/task.md`

---

## Next Steps

- [ ] เพิ่ม `"SlaAlertMonitorOptions": { "Enabled": true }` ใน `STS-NOC/appsettings.json`
- [ ] Deploy และ monitor log ว่า alert ไม่เกิน 2 ครั้งต่อ job
- [ ] ถ้าอนาคตมี code อื่นยิง SLA alert ผ่าน `AlertRepository` API → ต้องแก้ dedup จาก exact match เป็น `LIKE 'SLA-WARN|%'`

---

## ความเสี่ยงที่เหลือ

| ความเสี่ยง | ระดับ | อธิบาย |
|---|---|---|
| Worker restart → memory dedup หาย | 🟢 ต่ำ | SQL NOT EXISTS เป็น safety net ชั้น 2 |
| AlertQ ถูก truncate | 🟢 ต่ำ | Memory dedup ยังทำงานอยู่ |
| ConcurrentDictionary โตเรื่อยๆ | 🟢 ต่ำ | Key เล็กมาก (`"CM:123:SLA-WARN"`) |
| AlertRepository exact match | 🟡 กลาง | ถ้าอนาคตมี code ใหม่ยิงผ่าน API → dedup จะไม่ทำงาน |
