# STS - Notification Bug Report (2026-04-21)

## โจทย์
ระบบแจ้งเตือน (Bell) ทำงานได้บน Localhost แต่ไม่ทำงานบน Server และมีขั้นตอนที่หายไป

---

## ปัญหาที่เจอทั้งหมด (3 ข้อ)

---

### 🔴 ปัญหาที่ 1: แจ้งเตือนไม่ทำงานบน Server เลย

**อาการ**: เปิด browser บน server → Bell ไม่ขึ้นเลย

**สาเหตุ**: `.env` / `.env.uat` ใช้ `localhost` ใน `NEXT_PUBLIC_` variable
```
NEXT_PUBLIC_API_ALERT_SERVICE=http://localhost:5028
```

`NEXT_PUBLIC_` ถูก bake เข้า JS bundle ตอน `npm run build`  
→ browser ของ user fetch ไป `localhost` ของตัวเอง ไม่ใช่ server  
→ PC ของ user ไม่มี STS-ALERT → fail

**แก้**:
```
NEXT_PUBLIC_API_ALERT_SERVICE=http://10.0.60.10:5028
```
แล้ว **rebuild Next.js ใหม่** (แค่ restart ไม่พอ เพราะ bake ตอน build)

---

### 🔴 ปัญหาที่ 2: แจ้งเตือนขาดขั้นแรก (ตอนเปิดงาน) บน Server

**อาการ**: 
- Localhost → เปิดงาน ✅ แจ้ง, Assign TP ✅ แจ้ง  
- Server → เปิดงาน ❌ ไม่แจ้ง, Assign TP ✅ แจ้ง

**สาเหตุ**: `SendJobAlertAsync()` query หา recipients จาก Group `'Teleport Head'`  
ด้วย field `PersonId` แต่ version เก่าของ code return `UserId`  
→ Query ปัจจุบันกรองแค่ `WHERE g.GroupName = 'Teleport Head'` ทั้งระบบ  
→ ถ้า User ใน group นั้นไม่มี `PersonId` → return 0 row  
→ throw `InvalidOperationException`  
→ `catch { }` กลืน error เงียบๆ → ไม่มีการแจ้งเตือน

**Root Cause เพิ่มเติม**: Logic เดิมกรองด้วย TeleportId ของงาน แต่ถูก simplify ออกไป ทำให้ query หลุดออกนอก scope

**แก้**: กลับไปใช้ logic เดิม — filter by TeleportId ของงาน
```sql
-- ใหม่ (เหมือนเดิม + return PersonId แทน UserId)
SELECT DISTINCT u."PersonId"::TEXT
FROM "SCS0112"."Job" j
JOIN "SCS0112"."FunctionalLocation" f
  ON j."FunctionalLocationId" = f."FunctionalLocationId"
JOIN "SCS0112"."Person" p
  ON p."TeleportId" = f."TeleportId"
JOIN "SCS0112"."User" u
  ON u."PersonId" = p."PersonId"
JOIN "SCS0112"."Member" m
  ON m."UserId" = u."UserId"
JOIN "SCS0112"."Group" g
  ON g."GroupId" = m."GroupId"
WHERE j."JobId" = @JobId
  AND g."GroupName" = 'Teleport Head'
  AND u."PersonId" IS NOT NULL
  AND u."isEnabled" = true
  AND u."isDeleted" = false;
```

**ไฟล์ที่แก้**: `STS-NOC/Repositories/JobRepository.cs` → `SendJobAlertAsync()`

---

### 🟡 ปัญหาที่ 3: `catch {}` กลืน Error เงียบๆ

**อาการ**: error เกิดขึ้นแต่ไม่มีทางรู้ได้เลย debug ไม่ได้

**สาเหตุ**: `JobController` ใช้ `catch { }` เปล่าๆ

**แก้**: เปลี่ยนเป็น log warning
```csharp
// ก่อน
catch { }

// หลัง
catch (Exception ex)
{
    _logger.LogWarning(ex, "SendJobAlertAsync failed for JobId={JobId}", inserted.jobid);
}
```

**ไฟล์ที่แก้**: `STS-NOC/Controllers/JobController.cs` ทั้ง 2 จุด

---

## 🤦 Bug ที่โง่แต่ง่ายมาก (แก้ไม่กี่วิ)

### Bug A: AlertAPI URL มี path ซ้ำ → 404
**ไฟล์**: `STS-NOC/appsettings.Development.json`
```json
// ผิด ❌ — path ซ้อนกัน
"AlertAPI": "http://localhost:5028/api_alert_service"
// ผล: http://localhost:5028/api_alert_service/api/Alert/insert-alertQ → 404

// ถูก ✅
"AlertAPI": "http://localhost:5028"
// ผล: http://localhost:5028/api/Alert/insert-alertQ → 200
```
**เหตุ**: STS-ALERT ไม่มี `UsePathBase("/api_alert_service")` ใน Program.cs  
→ path `/api_alert_service` ไม่มีอยู่จริงใน STS-ALERT เลย

---

### Bug B: appsettings.json (Production) ใส่ค่า Dev ไว้
**ไฟล์**: `STS-NOC/appsettings.json`
```json
// ผิด ❌ — ใส่ password dev และ localhost ใน production config
"ConnString": "...Password=1309903084779;"
"AlertAPI": "http://localhost:5028/api_alert_service"

// ถูก ✅
"ConnString": "...Password=S@mart@2024;"
"AlertAPI": "http://10.0.60.10:5028"
```
**เหตุ**: มีคนก็อป config จาก Development มาใส่ใน Production โดยไม่เปลี่ยนค่า

---

## สรุป Logic การแจ้งเตือน (ปัจจุบัน)

```
1. เปิดใบงาน (JobInsert)
   → SendJobAlertAsync()
   → หา Teleport Head ของ Teleport เดียวกับงาน (ผ่าน FunctionalLocation)
   → POST → STS-ALERT → spAlertInsert → AlertQ

2. Assign งานให้ TP (JobNOCHDUpdate + pisassigntotp=true)
   → SendAssignedJobAlertAsync()
   → หา recipients จาก fJobNOCHDUpdateSendJobAlert(jobId)
   → POST → STS-ALERT → spAlertInsert → AlertQ

3. SLA Monitor (ทุก 5 นาที อัตโนมัติ)
   → SLAMonitorWorker
   → SLA >= 80% → SLA-WARN | SLA > 100% → SLA-OVER
   → Direct DB call → spAlertInsert

4. Frontend Poll (ทุก 5 วิ)
   → GET /api/Alert/get-unread-job-by-userid
   → เจอใหม่ → Bell + เสียง + Toast (เฉพาะ SLA)
```

---

## ไฟล์ที่แก้ทั้งหมด

| ไฟล์ | สิ่งที่แก้ |
|------|-----------|
| `STS-NOC/Repositories/JobRepository.cs` | recipient query กลับเป็น TeleportId filter |
| `STS-NOC/Controllers/JobController.cs` | `catch {}` → `LogWarning` (2 จุด) |
| `STS-NOC/appsettings.json` | password + AlertAPI → production values |
| `STS-NOC/appsettings.Development.json` | AlertAPI → `http://localhost:5028` |
| `sts-portal/.env` / `.env.uat` | `NEXT_PUBLIC_API_ALERT_SERVICE` → IP จริง *(ยังไม่แก้)* |

---

## สิ่งที่ยังต้องทำ

- [ ] แก้ `sts-portal/.env` → `NEXT_PUBLIC_API_ALERT_SERVICE=http://10.0.60.10:5028`
- [ ] Rebuild + Redeploy Next.js
- [ ] ตรวจสอบ `'Teleport Head'` group บน server DB มี member ที่มี PersonId มั้ย
- [ ] Deploy STS-NOC ขึ้น server

---

*บันทึก: 2026-04-21*
