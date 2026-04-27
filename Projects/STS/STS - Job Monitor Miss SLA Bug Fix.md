---
tags: [STS, bug-fix, job-monitor, miss-sla, postgresql]
date: 2026-04-21
status: fixed
---

# STS — Job Monitor: Miss SLA Bug Fix

## ปัญหาที่พบ

หน้า `/app/job-monitor` แสดง **Miss SLA = ตัวเลข** บน card แต่เมื่อกด tab Miss SLA **ตารางว่างเปล่า**

---

## Root Cause

### 1. Card กับ Table ดึงข้อมูลคนละแหล่ง

| | แหล่งข้อมูล | เงื่อนไข |
|---|---|---|
| Card (count) | Client-side fallback | `NOW > JobFixDate` |
| Table (rows) | API `getTPSearchList` | `SLAPercent > 100` |

`SLAPercent` ถูก update โดย `fCalcNetSLA` **เฉพาะตอน job เปลี่ยน status** → job ที่ไม่มีใครแตะ → `SLAPercent = NULL` → API คืน 0 jobs

### 2. Auto-refresh ทับข้อมูลทุก 2 วินาที

```
ทุก 2 วินาที:
  fetchJobMonitorData() → handleClickTab(Miss SLA)
  → getTPSearchList(pJobStatusId=100) → SLAPercent filter → 0 rows
  → dispatch([]) → ตารางถูกเคลียร์ซ้ำทุกรอบ
```

### 3. Incoming นับงานที่เกิน SLA รวมอยู่ด้วย

job ที่ Teleport Head ยังไม่ accept จนเกิน SLA → ติดอยู่ใน Incoming แต่ควรย้ายไป Miss SLA

---

## การแก้ไข

### Frontend — `JobMonitorPage.tsx`

เพิ่ม refs เพื่อหลีกเลี่ยง async state lag ใน `handleClickTab`:

```ts
const missSlaFallbackJobsRef = React.useRef<GetTPJobGetSearchListResultData[]>([])
const isMissSlaFallbackLoadedRef = React.useRef(false)
const activeTeleportIdRef = React.useRef('')
const hiddenRoleIdRef = React.useRef(hiddenRoleId)
```

`handleClickTab` — ถ้าเป็น Miss SLA tab + fallback loaded → ใช้ ref data แทน API call:

```ts
if (isMissSlaTab && isMissSlaFallbackLoadedRef.current) {
  const overdueJobs = missSlaFallbackJobsRef.current.filter((job) => {
    const dueDate = getJobDueDate(job)
    return dueDate !== null && dueDate.isBefore(dayjs())
  })
  dispatch(setTPJobTpSearchList(overdueJobs))
  dispatch(setJobMonitorTableIndex(tab))
  return
}
```

> **ทำไมต้องใช้ ref ไม่ใช่ state?**
> `handleClickTab` ถูกเรียกจาก auto-refresh (async) — ตอนนั้น React state อาจยังไม่ update แต่ ref sync ทันที

---

### Database — 2 Stored Procedures

**Logic ใหม่:**

```
Incoming  = JobTPStatusId = 1
            AND (JobFixDate IS NULL OR NOW <= JobFixDate)

Miss SLA  = SLAPercent > 100
            OR (JobFixDate IS NOT NULL AND NOW > JobFixDate)
```

#### `fJobTPSummaryTbl` (Summary counts บน card)

```sql
-- Incoming: ตัดงานเกิน SLA ออก
SELECT 1 AS id, 'Incoming' AS text, COUNT("JobId") AS value
FROM job_data
WHERE "JobTPStatusId" = 1
  AND ("JobFixDate" IS NULL OR NOW() <= "JobFixDate")   -- เพิ่ม

-- Miss SLA: เพิ่ม real-time check
SELECT 100 AS id, 'Miss SLA' AS text, COUNT("JobId") AS value
FROM job_data
WHERE COALESCE("SLAPercent",0) > 100
   OR ("JobFixDate" IS NOT NULL AND NOW() > "JobFixDate")  -- เพิ่ม
```

> ต้องเพิ่ม `j."JobFixDate"` เข้าไปใน CTE `job_data` ด้วย

#### `spJobTPSearchTbl` (ข้อมูลใน table)

```sql
WHEN "pJobStatusId" = 1 AND "pRoleId" = 4 THEN
  ' AND JobTPStatusId = 6 AND (JobFixDate IS NULL OR NOW() <= JobFixDate) '
WHEN "pJobStatusId" = 1 THEN
  ' AND JobTPStatusId = 1 AND (JobFixDate IS NULL OR NOW() <= JobFixDate) '
WHEN "pJobStatusId" = 100 THEN
  ' AND (SLAPercent > 100 OR (JobFixDate IS NOT NULL AND NOW() > JobFixDate)) '
```

---

## ผลลัพธ์หลังแก้

| Tab | ก่อน | หลัง |
|---|---|---|
| All Jobs | 10 | 10 (ไม่เปลี่ยน) |
| Incoming | 10 | 0 (ตัดงานเกิน SLA ออก) |
| Miss SLA | 0 | 10 (ดูดงานเกิน SLA เข้ามา) |

---

## Key Insight

`JobFixDate` = SLA deadline ที่ set ตั้งแต่เปิด job — ใช้ได้ real-time ไม่ต้องรอ `fCalcNetSLA`

**Business rule:** SLA วิ่งตั้งแต่เปิด job ไม่ใช่ตั้งแต่ Teleport Head กด accept → งานที่ไม่ได้ accept จนเกินกำหนดต้องนับเป็น Miss SLA

---

## ไฟล์ที่แก้

| ไฟล์ | สิ่งที่แก้ |
|---|---|
| `sts-portal/.../JobMonitorPage.tsx` | เพิ่ม refs + แก้ `handleClickTab` |
| DB `fJobTPSummaryTbl` | Incoming/Miss SLA filter |
| DB `spJobTPSearchTbl` | Incoming/Miss SLA filter |
