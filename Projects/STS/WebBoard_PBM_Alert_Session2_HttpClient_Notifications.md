# Web Board & PBM Alert — Session 2: HttpClient Lifecycle + Notification Architecture

## Metadata
- Project: STS
- Scope: STS-NOC backend, sts-portal frontend, PBM + WebBoard alerts
- Date: 2026-04-16
- Repos:
  - `STS-NOC` — WebBoardRepository.cs, PBMRepository.cs
  - `sts-portal` — header.tsx, WebBoardPage.tsx, WebBoardDetailPage.tsx
- เชื่อมต่อกับ: [[WebBoard_Alert_and_UX_Debugging_Playbook]]

---

# 1. โจทย์รอบนี้

## 1.1 อาการที่ user รายงาน
- **Alert ส่งแล้ว แต่ bell ไม่ขึ้นใน portal** — ระบบดูเหมือนทำงาน แต่ไม่มีเสียง/badge
- **PBM แจ้งเตือน "ก่อนมอบหมายงาน"** — ดูเหมือนระบบรู้อนาคต
- **PBM แจ้งเตือนซ้ำหลายครั้งสำหรับงานเดียวกัน** — บางงานเห็น 3-4 notifications
- **WebBoard มี 13 ปัญหา UX** ที่ค้างอยู่

## 1.2 สิ่งที่ทำให้โจทย์นี้ยาก
- อาการ "ส่งแล้วแต่ไม่ขึ้น" เป็น class ของ bug ที่มีหลาย root cause
- `Task.Run` fire-and-forget + DI lifecycle เป็น gotcha ที่คนที่ไม่เคยเจอจะ debug วนนาน
- session mapping (UserId ≠ PersonId) ซ้อนทับอยู่กับ bug HttpClient
- PBM notification flow กระจายอยู่หลายจุด ต้องอ่านทุก method ถึงเข้าใจ

---

# 2. Bug หลัก: `Cannot access a disposed object` — HttpClient ใน Task.Run

## 2.1 อาการในระบบ
```
[WebBoard] Sending alert to PersonId=1
SendWebBoardAlert exception: Cannot access a disposed object.
Object name: 'System.Net.Http.HttpClient'.
```

CreateComment ตอบ 200 ปกติ แต่ alert ไม่ถึง STS-ALERT เลย

## 2.2 Root Cause

### ที่มาของปัญหา
```csharp
// WebBoardRepository constructor (เดิม)
private readonly HttpClient _httpClient;

public WebBoardRepository(HttpClient httpClient, ...) {
    _httpClient = httpClient;  // ← ปัญหาอยู่ที่นี่
}

public async Task CreateComment(...) {
    // ... logic ...
    Task.Run(async () => {
        await SendWebBoardAlert(personId, postId);  // ← ถูกเรียกหลัง scope ปิด
    });
}

private async Task SendWebBoardAlert(...) {
    await _httpClient.PostAsync(...);  // ← BOOM: HttpClient ถูก dispose ไปแล้ว
}
```

### ทำไมถึงพัง
```
Request arrives → DI creates scope → HttpClient injected into Repository
→ CreateComment called → Task.Run fires (async, no await)
→ HTTP Request scope ENDS → DI disposes HttpClient
→ Task.Run continues later → tries to use disposed HttpClient → EXCEPTION
```

**กุญแจสำคัญ:** DI-injected `HttpClient` มีอายุแค่ request scope  
`Task.Run` ที่ fire-and-forget ไม่ผูกอายุกับ request scope  
ดังนั้นมันยังรันอยู่หลังจาก HttpClient ถูก dispose ไปแล้ว

## 2.3 การแก้ไข

```csharp
// เปลี่ยนจาก HttpClient เป็น IHttpClientFactory
private readonly IHttpClientFactory _httpClientFactory;

public WebBoardRepository(IHttpClientFactory httpClientFactory, ...) {
    _httpClientFactory = httpClientFactory;  // ✅ factory ไม่ถูก dispose
}

private async Task SendWebBoardAlert(...) {
    using var client = _httpClientFactory.CreateClient();  // ✅ สร้าง client ใหม่ in-scope
    using var req = new HttpRequestMessage(HttpMethod.Post, $"{_alertBaseUrl}/api/Alert/insert-alertQ");
    // ... serialize body ...
    var response = await client.SendAsync(req);
}
```

### ทำไม IHttpClientFactory ถึงแก้ได้
- `IHttpClientFactory` เป็น singleton — ไม่ถูก dispose ตาม request scope
- `.CreateClient()` สร้าง `HttpClient` ใหม่ที่ managed lifecycle แยก
- `using` บน client/request จัดการ cleanup ใน Task.Run เอง
- ไม่ต้องพึ่ง DI lifetime ของ outer scope

## 2.4 Files ที่แก้
- `STS-NOC/Repositories/WebBoardRepository.cs`
- `STS-NOC/Repositories/PBMRepository.cs`
  - ทั้งสองเปลี่ยน pattern เดียวกัน

## 2.5 บทเรียนสำคัญ

> **Rule: ถ้าโค้ดที่อยู่ใน `Task.Run` หรือ fire-and-forget จะใช้ object จาก DI — ต้องตรวจ lifetime ก่อนเสมอ**

### Checklist เวลาเจอ `Cannot access a disposed object`
- [ ] Object นั้น inject มาจาก DI ไหม
- [ ] มันถูกใช้ข้ามขอบ request scope ไหม (`Task.Run`, callbacks, async void)
- [ ] ถ้าใช่ → เปลี่ยนเป็น factory pattern แทน direct inject
- [ ] `IHttpClientFactory` สำหรับ HttpClient
- [ ] `IServiceScopeFactory` สำหรับ DbContext และ service อื่น

---

# 3. PBM Notification: InsertJob Broadcast ผิดพลาด

## 3.1 อาการ
- สร้างงานใหม่ → ทุกคนในกลุ่ม PBM (GroupId=30) ได้รับ notification
- ทั้งที่งานยังไม่มีผู้รับผิดชอบ
- user รู้สึกว่าระบบ "รู้อนาคต" ว่าจะได้งานก่อนที่ manager จะมอบหมาย

## 3.2 Root Cause

```csharp
// InsertJob (เดิม) — โค้ดที่ถูกลบออก
var groupMembers = notifConn.Query<int>(
    "SELECT PersonId FROM ... WHERE GroupId = 30"
);
// ส่ง alert ให้ทุกคนในกลุ่ม PBM ทันทีที่สร้างงาน
foreach (var personId in groupMembers) {
    InsertAlertQ(personId, jobId, "PBM|...");
}
```

### ปัญหา
- Broadcast to everyone ≠ Notify relevant person
- งานยังไม่มีผู้รับ → ใครควรได้รับ alert กันแน่?
- ทำให้ notification flood ก่อน assignment

## 3.3 การแก้ไข

**ลบ InsertJob broadcast ออกทั้งหมด**

แจ้งเตือนเฉพาะในจุดที่ "ชัดเจนว่าใครต้องรู้":

| Action | แจ้งใคร |
|---|---|
| ClassifyAndAssign | ผู้รับงาน (assignee) |
| UpdateStatus | คู่กรณีที่เกี่ยวข้อง |
| Discovered | ผู้จัดการที่รับผิดชอบ |
| Approve | assignee |
| ~~InsertJob~~ | ~~ทุกคนในกลุ่ม~~ ← **ลบออก** |

## 3.4 บทเรียน

> **Rule: Notification ควรยิงเมื่อ "คนที่ถูก notify มีเหตุผลชัดเจนที่ต้องรู้" ไม่ใช่ broadcast ก่อนแล้วค่อยคิดทีหลัง**

- Premature notification ทำให้ user ไม่เชื่อถือระบบ ("ทำไมแจ้งเรื่องนี้มาด้วย?")
- ถ้าไม่แน่ใจว่าต้องแจ้งใคร → ไม่แจ้งดีกว่าแจ้งเกิน

---

# 4. PBM Notification: Dedup ด้วย jobId

## 4.1 อาการ
- user เห็น notification ซ้ำ 3-4 รายการสำหรับงานเดียวกัน
- เกิดจาก backend แจ้งเตือนหลาย event (assign, update, discover) สำหรับงานเดียว
- ทุก event มีสิทธิ์สร้าง AlertQ แยกกัน

## 4.2 การแก้ไข (Frontend)

```typescript
// header.tsx — pbmAlertsDisplay
const pbmAlertsDisplay = useMemo(() => {
  const map = new Map<number, AlertUnreadItem>();
  
  for (const item of pbmAlerts) {
    const existing = map.get(item.jobId);
    // เก็บเฉพาะ alert ล่าสุดของแต่ละ jobId
    if (!existing || new Date(item.createdAt) > new Date(existing.createdAt)) {
      map.set(item.jobId, item);
    }
  }
  
  return Array.from(map.values()).sort(
    (a, b) => new Date(b.createdAt).getTime() - new Date(a.createdAt).getTime()
  );
}, [pbmAlerts]);
```

**หลักการ:** แยก "data store" กับ "display" ออกจากกัน
- `pbmAlerts` = ข้อมูลดิบทั้งหมด (ใช้สำหรับ mark-read ทั้งกลุ่ม)
- `pbmAlertsDisplay` = deduplicated สำหรับแสดงผล (1 job = 1 notification)

## 4.3 บทเรียน

> **Rule: เมื่อ notification system สร้าง multiple events สำหรับ entity เดียว ให้ dedup ที่ display layer โดย group ด้วย entity ID**

- เก็บ raw data ไว้ครบ (เผื่อต้องการ mark-read ทั้งหมด)
- dedup เฉพาะตอนแสดงผล
- แสดงรายการล่าสุดเสมอ ไม่ใช่รายการแรก

---

# 5. JWT UserId ≠ PersonId — Alert หาเจ้าของไม่เจอ

## 5.1 ปัญหา

```
STS-ADM ออก JWT token ที่มี userId ใน claim
แต่ AlertQ บันทึกด้วย PersonId (คนละ ID)
→ getUnreadByPersonId(personId) หา record ไม่เจอ เพราะ personId ผิด
```

## 5.2 สาเหตุ
- `userId` คือ login credential ID
- `personId` คือ HR/profile ID
- ทั้งสองอาจเป็นตัวเลขต่างกัน ถ้าระบบมี migration ประวัติ

## 5.3 การแก้
- เพิ่ม endpoint ใหม่ `getUnreadPBMWebBoardByUserId` ที่รับ `userId`
- backend resolve userId → personId เองก่อน query
- frontend ใช้ `userId` จาก session ที่มีอยู่แน่นอน

```typescript
// header.tsx
const userId = Number((session?.userData as any)?.userId ?? 0);

// polling ด้วย userId (ไม่ต้องพึ่ง personId ที่อาจหาไม่เจอ)
const res = await alertService.getUnreadPBMWebBoardByUserId(userId, session);
```

## 5.4 บทเรียน

> **Rule: เวลาออกแบบ notification ข้าม system ต้องระบุให้ชัดว่าใช้ ID ชุดไหน และใครรับผิดชอบ resolve**

- ถ้า consumer รู้แค่ `userId` → API ต้อง accept userId
- อย่าบังคับ consumer หา `personId` เองถ้า session ไม่มี
- ดีกว่าให้ backend resolve เพราะอยู่ใกล้ DB

---

# 6. WebBoard 13 Issues — สิ่งที่แก้จริง

## 6.1 Content/UX Issues

| # | ปัญหา | แก้อะไร |
|---|---|---|
| 0 | เนื้อหายาวเกินใน detail | preview 400 chars + ปุ่ม "อ่านเพิ่มเติม" / "ย่อ" |
| 3 | ไม่มี character limit/counter | post 1500, comment 500, title 150 + live counter |
| 4 | ไม่มี file size validation | reject > 1MB ทันที + clear input |
| 5 | ปิด composer โดยไม่ warn | `isComposerDirty` check + Swal confirm |
| 6 | Like reload ทั้งหน้า | optimistic state update เฉพาะ post |
| 7 | Spacebar ยิง search | guard `if (!nextKeyword) return` |
| 8 | กลับจาก detail เสีย search | keyword ใน URL `?q=`, `backHref` carry ค่าไป |
| 9 | Badge "เปิดคำถาม" แสดงตลอด | แสดงเฉพาะ status === 'resolved' |
| 10 | Icon ตา → หัวใจ | เปลี่ยน Eye → Heart ให้ตรง semantic |
| 11 | ไม่มีรูป profile | Avatar component + AvatarFallback initials |
| 12 | Mobile layout แตก | overflow-y-auto modal, full-width buttons mobile |
| 13 | ข้อมูลไม่ update realtime | 15s polling + focus event trigger |

## 6.2 Edit Post with Attachments (Issue #1 — ซับซ้อนที่สุด)

**3 states ที่ต้องแยกกัน:**
```typescript
const [editFiles, setEditFiles] = useState<File[]>([]);           // ไฟล์ใหม่ที่จะอัปโหลด
const [removedAttachmentIds, setRemovedAttachmentIds] = useState<number[]>([]); // ไฟล์เดิมที่จะลบ
// existing attachments = data?.attachments (จาก backend)
```

**Save flow ที่ถูกต้อง:**
```
1. UpdatePost (text changes)
2. Delete removed attachments (Promise.all)
3. Upload new files (Promise.allSettled)
```

สำคัญ: ทำ step ตามลำดับนี้เสมอ — ถ้า parallel อาจเกิด race condition

## 6.3 Modal Overflow Fix (Issue #2)

```jsx
// ผิด — content ดัน button ออกนอก viewport
<div className="fixed inset-0 flex items-center justify-center">
  <div className="max-w-[640px] bg-white">

// ถูก — scroll ใน modal ได้
<div className="fixed inset-0 z-[80] flex items-start justify-center overflow-y-auto p-3 sm:p-6">
  <div className="relative z-[1] my-auto flex max-h-[calc(100vh-2rem)] flex-col overflow-hidden">
```

**Key CSS:**
- outer: `overflow-y-auto` + `items-start`
- inner: `max-h-[calc(100vh-2rem)]` + `overflow-hidden` + `flex-col`

---

# 7. Code Review Findings

## 7.1 สิ่งที่ผ่านแล้ว ✅
- `WebBoardAPI.DeleteAttachment` มีอยู่จริงใน `web-board.api.ts` ✅
- `header.tsx` ไม่มี dead code — `unreadCount/alerts/pollCountOnly` ใช้กับ NOC JOB bell จริง ✅
- Character counters ตรงกันทั้ง frontend (JS) และ backend (C#) ✅
- `running` flag ใน polling ป้องกัน concurrent poll ✅
- File input `event.target.value = ''` หลัง add ป้องกัน duplicate select ✅
- `Promise.allSettled` สำหรับ batch upload — ไม่ fail ทั้งหมดถ้าไฟล์เดียว error ✅
- `isComposerDirty` เช็ก title + content + files ✅

## 7.2 N+1 Pattern ใน hydratePostPreviews

```typescript
// hydratePostPreviews ใน WebBoardPage.tsx
// สำหรับแต่ละ post ที่มี attachments → เรียก GetPostDetail แยก
const detailResults = await Promise.allSettled(
  targets.map((post) => WebBoardAPI.GetPostDetail(...))
);
```

**ประเมิน:** ยอมรับได้ในปัจจุบัน เพราะทำ parallel ด้วย Promise.allSettled  
**ถ้า scale ขึ้น:** ควรขอ backend เพิ่ม preview attachment field ใน GetPostList response

## 7.3 Potential Issue: handleMarkNocAllRead ใช้ markReadByUserId กับ NOC alerts

```typescript
// header.tsx — handleMarkNocAllRead
group.map(a => alertService.markReadByUserId(userId, a.jobId, session))
```

NOC JOB alerts ดั้งเดิมใช้ PersonId-based mark-read  
ถ้า `markReadByUserId` API รองรับเฉพาะ PBM/WebBoard อาจ mark ไม่สำเร็จ  
**ควรตรวจ:** endpoint `mark-read-by-userid` ใน STS-ALERT ว่า filter type ยังไง

---

# 8. Patterns ที่ได้จากงานรอบนี้

## 8.1 Fire-and-Forget + DI Pattern

```csharp
// WRONG: inject object ที่มี DI lifetime แล้วใช้ใน fire-and-forget
Task.Run(async () => {
    await _httpClient.PostAsync(...);  // DISPOSED!
});

// RIGHT: สร้าง object ใหม่ใน scope ของ Task
Task.Run(async () => {
    using var client = _httpClientFactory.CreateClient();
    await client.PostAsync(...);  // ✅ ยังมีชีวิต
});
```

**Pattern นี้ใช้กับ:** DbContext, HttpClient, ทุก scoped DI service

## 8.2 Notification Fan-out Anti-pattern

```
❌ Anti-pattern: Broadcast to all → let receiver filter
   "แจ้งทุกคนก่อน แล้วค่อยกรองเอง"
   
✅ Pattern: Targeted notify → only relevant party
   "ระบุก่อนว่าใครต้องรู้ แล้วค่อยส่ง"
```

## 8.3 Display Dedup vs Data Dedup

```typescript
// Store raw data (สำหรับ operations เช่น mark-read all)
const [pbmAlerts, setPbmAlerts] = useState<AlertUnreadItem[]>([]);

// Compute deduplicated view (สำหรับ render เท่านั้น)
const pbmAlertsDisplay = useMemo(() => dedup(pbmAlerts), [pbmAlerts]);
```

## 8.4 3-State Attachment Edit Pattern

```
existing[]    → มาจาก server, แสดงให้ user เห็น
removed[]     → user mark ให้ลบ, ซ่อนจาก UI แต่ยังไม่ delete จริง
newFiles[]    → user เพิ่มใหม่, ยังไม่อยู่ใน server

save() {
  1. update text
  2. delete removed[] → server
  3. upload newFiles[] → server
}
```

## 8.5 URL-Preserved Search State Pattern

```typescript
// เก็บ search state ไว้ใน URL แทน local state
const syncKeywordToUrl = (keyword: string) => {
  const params = new URLSearchParams(searchParams.toString());
  keyword ? params.set('q', keyword) : params.delete('q');
  router.replace(query ? `/path?${query}` : '/path');
};

// detail page อ่าน keyword จาก URL แล้วสร้าง backHref
const backHref = queryKeyword 
  ? `/path?q=${queryKeyword}` 
  : '/path';
```

---

# 9. Checklist เมื่อเจอ "Alert ส่งแล้วแต่ไม่ขึ้น"

## 9.1 ไล่ layer ตามลำดับ

```
Layer 1: Backend actually ran?
  □ ดู log backend — มี exception ไหม
  □ ถ้ามี "Cannot access disposed object" → IHttpClientFactory fix

Layer 2: AlertQ record created?
  □ Query AlertQ table โดยตรง
  □ ถ้าไม่มี → backend ไม่ได้เรียก insert

Layer 3: Frontend polling the right API?
  □ Network tab — endpoint ถูกไหม
  □ UserId หรือ PersonId?

Layer 4: Frontend parsing action correctly?
  □ console.log action string
  □ format: "PBM|status|jobId" หรือ "WEBBOARD|COMMENT|postId"

Layer 5: UI rendering?
  □ State มีค่าไหม
  □ filter/dedup ตัดออกไหม
```

## 9.2 Checklist Notification Architecture
- [ ] ใช้ ID ชุดเดียวกันทั้ง insert และ query
- [ ] backend มี endpoint ที่รับ userId แล้ว resolve เป็น personId เอง
- [ ] action format มีเอกสาร/constant ที่ทุก layer ใช้ร่วมกัน
- [ ] dedup strategy ชัดเจน (raw vs display)
- [ ] ไม่ broadcast ก่อนมี assignment

---

# 10. สิ่งที่ควรปรับปรุงต่อ

## 10.1 Backend: เพิ่ม preview attachment ใน GetPostList
ตอนนี้ frontend ต้องเรียก GetPostDetail แยกทุก post ที่มีรูป  
ควรให้ GetPostList return `firstAttachment` มาด้วย

## 10.2 Backend: N+1 query ใน PBMRepository
เช็กว่ามีจุดไหนที่ query ใน loop ไหม → อาจต้องเปลี่ยนเป็น JOIN หรือ batch

## 10.3 Frontend: Action format constant
ตอนนี้ `"PBM|"`, `"WEBBOARD|"` เป็น hardcode string  
ควร extract เป็น constant เช่น `AlertActionPrefix.PBM` เพื่อป้องกัน typo

## 10.4 Test cases ภาษาไทย
- ข้อความที่มีวรรณยุกต์ซ้อน
- paste จาก Word/Line ที่มี encoding แปลก
- ยาว 1500 ตัวจริง

---

# 11. สิ่งที่ได้เรียนรู้

## 11.1 DI Lifetime เป็นของที่ต้องเข้าใจลึก
- Scoped service ใน ASP.NET Core ตายพร้อม HTTP request
- fire-and-forget (`Task.Run` โดยไม่ await) ไม่ผูกกับ lifetime ของ request
- ถ้า inject scoped service แล้วใช้ใน background task → disposed exception เสมอ
- Solution: ใช้ factory (IHttpClientFactory, IServiceScopeFactory)

## 11.2 Alert System Design Principles
- **Targeted > Broadcast** — ส่งหาคนที่ควรรู้จริงๆ ไม่ใช่ทุกคน
- **userId vs personId** — ต้องตกลง ID convention ก่อน build
- **Dedup at display layer** — เก็บ raw events ครบ แต่แสดงผลให้สะอาด
- **Action format เป็น contract** — ต้อง parse ได้ทั้ง frontend และ backend

## 11.3 Frontend State Management สำหรับ Edit Media
- แยก state 3 ชั้น: existing / removed / new
- save flow ต้องเรียงลำดับ: update → delete → upload
- ไม่ merge state รวมกัน มิฉะนั้น append vs replace จะพังง่าย

## 11.4 Optimistic UI ลด perceived latency ได้มาก
- Like toggle ไม่ต้อง reload หน้า
- update state เฉพาะ item ที่เปลี่ยน
- ถ้า API fail → rollback state กลับ
- user รู้สึก responsive ทันที

---

# 12. Proposed Next Nodes

## 12.1 Node: ASP.NET Core DI Lifetime Cheatsheet
- Singleton / Scoped / Transient
- เมื่อไหร่ควรใช้แบบไหน
- Anti-patterns ที่เจอบ่อย (inject Scoped into Singleton, fire-and-forget)

## 12.2 Node: Alert Architecture Map (STS)
- flow diagram จาก NOC → ALERT → portal
- action format ทุกชนิด
- UserId vs PersonId mapping table

## 12.3 Node: Optimistic UI Pattern Catalog
- Like toggle
- Mark as read
- Delete (ซ่อนทันที ลบจริงทีหลัง)
- Rollback strategy เมื่อ API fail

## 12.4 Node: Notification Anti-pattern Catalog
- Broadcast before assignment (รอบนี้เจอ)
- Poll too frequently
- No dedup strategy
- Action format mismatch between producer and consumer

## 12.5 Node: 3-State Edit Pattern (Media)
- เขียนเป็น generic component/hook
- รองรับ: ไฟล์, รูป, embed URL
- reuse ใน PBM, WebBoard, Profile

---

# 13. สรุปสั้น

**โจทย์รอบนี้สอน 3 เรื่องหลัก:**

1. **DI Lifetime + fire-and-forget** — HttpClient ที่ inject มาตาย พร้อม request scope; Task.Run ไม่รู้เรื่อง; ใช้ IHttpClientFactory แก้
2. **Notification design** — Broadcast ก่อนมอบหมาย = ระบบรู้อนาคต; ยิง notification เฉพาะเมื่อ "ใครควรรู้" ชัดเจน
3. **Multi-layer bug** — อาการเดียว root cause ซ้อนกันหลายชั้น; ไล่ layer ทีละชั้น อย่าเดา

