---
tags: [STS, alert, SLA, design, pending]
created: 2026-04-20
updated: 2026-04-20
status: pending
type: working-note
---

# STS — SLA Alert Once-Per-Job Plan

Related:
- [[Projects/STS/STS - SLA Alert Implementation|STS - SLA Alert Implementation]]
- [[Projects/STS/STS - SLA Alert Bug Log|STS - SLA Alert Bug Log]]
- [[Projects/STS/STS - SLA Alert Troubleshooting Playbook|STS - SLA Alert Troubleshooting Playbook]]

## Goal
ทำให้ `SLA Alert` แจ้งเตือนต่อ `1 งาน` แค่:
- `ใกล้เกิน SLA` (`SLA-WARN`) `1 ครั้ง`
- `เกิน SLA แล้ว` (`SLA-OVER`) `1 ครั้ง`

หลังจากนั้นไม่แจ้งซ้ำอีกสำหรับงานเดิม

นี่คือ goal แบบ `ไม่ overengineer`
- ไม่เพิ่ม table ใหม่
- ไม่เพิ่ม interface/factory
- ไม่ redesign alert architecture
- ใช้ข้อมูลที่มีอยู่แล้วใน `AlertQ.Action` เป็นหลัก

## Current understanding
ตอนนี้ producer หลักอยู่ที่ `STS-NOC/Workers/SLAMonitorWorker.cs`

worker ตัวนี้:
- scan ทุก `5 นาที`
- ยิง `SLA-WARN` เมื่อ `% SLA >= 80`
- ยิง `SLA-OVER` เมื่อเกิน SLA แล้ว
- กันซ้ำแค่ `30 นาที` ด้วยการดู `AlertQ` ย้อนหลัง

จุดสำคัญในโค้ดตอนนี้
- `STS-NOC/Workers/SLAMonitorWorker.cs`
- `STS-NOC/Program.cs`
- `STS-ALERT/database/stored_f/fAlertGetUnreadCount.sql`
- `STS-ALERT/database/stored_f/spAlertUpdateStatus.sql`
- `sts-portal/components/_layout/main/header.tsx`

## Actual current behavior
behavior ปัจจุบันไม่ตรงกับที่ต้องการ

### 1. มันไม่ได้แจ้งแค่ครั้งเดียวต่อ stage
ตอนนี้ worker ใช้ cooldown `30 นาที` ไม่ใช่ once-per-stage

ผลคือ:
- งานเดิมสามารถโดน `SLA-WARN` ซ้ำได้ทุก 30 นาทีถ้ายังอยู่ในช่วง warn
- งานเดิมสามารถโดน `SLA-OVER` ซ้ำได้ทุก 30 นาทีถ้ายัง over อยู่
- มันไม่ใช่ `warn 1 ครั้ง + over 1 ครั้ง`

### 2. มันเช็กซ้ำแบบรวม `SLA%` ไม่แยก `WARN` กับ `OVER`
query กันซ้ำตอนนี้ดูแค่ `Action LIKE 'SLA%'`

ผลคือ:
- ถ้าเพิ่งยิง `SLA-WARN` ไป แล้วอีกไม่กี่นาทีงานกลายเป็น `SLA-OVER`
- worker อาจยังไม่ยิง `SLA-OVER` ทันที เพราะโดน cooldown เดิม block อยู่

สรุปคือ logic ปัจจุบันไม่ใช่ทั้ง `once per stage` และไม่ใช่ `state transition aware`

### 3. ถ้ามีหลาย instance ของ `STS-NOC` อาจยิงซ้ำคูณกัน
`SLAMonitorWorker` ถูก register ตรงๆ ใน `STS-NOC/Program.cs`

ถ้ามีหลาย server หรือหลาย process รันพร้อมกัน:
- แต่ละ instance จะ scan เอง
- แต่ละ instance มีโอกาสยิง alert ซ้ำกัน
- อาการที่เห็นว่า "เตือนสนั่น" หลังเปลี่ยน server มีโอกาสมาจากจุดนี้ด้วย

### 4. unread/UI path ยังทำให้ noise ดูหนักขึ้น
ถึงต้นเหตุจริงอยู่ที่ worker แต่ฝั่ง unread/UI ก็ทำให้ความรู้สึกว่ามัน spam หนักกว่าเดิม

เพราะตอนนี้:
- unread query คืนทุก unread row ตรงๆ
- job alert ใน header ยังไม่ dedupe ตอนแสดงผล
- mark read ต่อ `JobId` ยังหยิบแค่แถวเดียวในบาง flow

อันนี้ไม่ใช่ต้นเหตุของการ insert ซ้ำ แต่ทำให้ UX แย่ลง

## Problem list

### P1. SLA alert rule ไม่ตรง requirement
requirement ที่ต้องการ:
- ต่อ 1 งาน แจ้ง `WARN` 1 ครั้ง
- ต่อ 1 งาน แจ้ง `OVER` 1 ครั้ง

แต่ implementation ตอนนี้คือ:
- ต่อ 1 งาน แจ้งได้เรื่อยๆ ทุก 30 นาที

### P2. `WARN` ไป block `OVER` ชั่วคราว
เพราะ dedupe ใช้ `LIKE 'SLA%'` รวมกัน

ผลเสีย:
- งานอาจเข้า state `OVER` แล้ว แต่ยังไม่แจ้ง `OVER` ทันที
- semantics ของ alert เพี้ยน

### P3. มีความเสี่ยงยิงซ้ำข้าม instance
ถ้ามีหลาย instance ของ `STS-NOC` รัน worker พร้อมกัน

ผลเสีย:
- insert ซ้ำใน `AlertQ`
- user เห็นกระดิ่งและ toast หลายลูก

### P4. UI ยังขยาย noise ของข้อมูลซ้ำ
ไม่ใช่ root cause แต่เป็น multiplier

ผลเสีย:
- unread เยอะเกินจริงในมุม user
- อ่านแล้วไม่รู้สึกว่าหายหมด
- debug ยาก เพราะแยกยากว่า spam จาก producer หรือจาก display path

## Minimal fix recommendation
ถ้าจะทำแบบไม่ overengineer ให้แก้ logic ที่ producer ก่อนเป็นหลัก

### Decision
ใช้ `AlertQ.Action` ที่มีอยู่แล้วเป็น source of truth

กติกาใหม่:
- ถ้างานนี้ยังไม่เคยมี `SLA-WARN` -> ยิง `SLA-WARN` ได้ 1 ครั้ง
- ถ้างานนี้ยังไม่เคยมี `SLA-OVER` -> ยิง `SLA-OVER` ได้ 1 ครั้ง
- ถ้าเคยมีแล้ว stage นั้น ห้ามยิงอีก

แปลว่าต้องเปลี่ยนจาก:
- `cooldown 30 นาที`

ไปเป็น:
- `existence check by exact stage`

### Why this is the minimum viable fix
ข้อดี:
- แก้ตรง requirement ที่ user ต้องการพอดี
- ไม่ต้องเพิ่ม schema ใหม่
- ไม่ต้องเปลี่ยน SP หลัก
- ไม่ต้องแตะ backend/frontend หลายจุดพร้อมกัน
- อ่านและ maintain ง่าย

ข้อเสีย:
- ถ้ามีหลาย instance รันพร้อมกัน ยังมี race condition ได้อยู่
- ถ้าต้องการ reset state เมื่อมีการ reopen/reset SLA ในอนาคต จะต้องออกแบบเพิ่มอีกรอบ

แต่สำหรับโจทย์ปัจจุบัน นี่คือ fix ที่เล็กสุดและตรงที่สุด

## Files to change

### 1. Must change
#### `STS-NOC/Workers/SLAMonitorWorker.cs`
ไฟล์นี้ต้องแก้แน่นอน

สิ่งที่ต้องแก้:
- เปลี่ยน query scan ฝั่ง `CM`
- เปลี่ยน query scan ฝั่ง `PM`
- เอา logic `cooldown 30 นาที` ออกจาก decision หลัก
- แยกการเช็กว่าเคยส่ง `SLA-WARN` แล้วหรือยัง
- แยกการเช็กว่าเคยส่ง `SLA-OVER` แล้วหรือยัง
- ให้ stage ตัดสินจาก state ปัจจุบันของงาน ไม่ใช่จาก cooldown history

รูปแบบ logic ที่ควรได้:
- ถ้า state ปัจจุบันเป็น warn และยังไม่มี `SLA-WARN` สำหรับ `JobId` นี้ -> send warn
- ถ้า state ปัจจุบันเป็น over และยังไม่มี `SLA-OVER` สำหรับ `JobId` นี้ -> send over
- ถ้ามีแล้ว -> skip

สิ่งที่ไม่ต้องทำในรอบนี้:
- ไม่ต้องเพิ่ม class ใหม่
- ไม่ต้องเพิ่ม abstraction ใหม่
- ไม่ต้องแตก worker เป็นหลาย service

### 2. Should review, but not required for the minimum fix
#### `STS-NOC/Program.cs`
ไฟล์นี้ไม่จำเป็นสำหรับ logic once-per-stage โดยตรง

แต่น่าดูเพิ่มถ้า production รันหลาย instance เพราะ worker ถูก register ตรงๆ

ถ้าพบว่ามีหลาย instance จริง มีทางเลือกง่ายๆ 2 แบบ:
- ใช้ config ให้รัน worker แค่เครื่องเดียว
- ปิด worker บางเครื่องชั่วคราว

ถ้ายังไม่แก้จุดนี้ ต่อให้ logic ดีขึ้น ก็ยังมีโอกาสซ้ำจากหลาย instance ได้

### 3. Nice to have later, not part of the minimum fix
#### `STS-ALERT/database/stored_f/fAlertGetUnreadCount.sql`
ใช้ถ้าต้องลด noise ฝั่ง unread display เพิ่ม

#### `STS-ALERT/database/stored_f/spAlertUpdateStatus.sql`
ใช้ถ้าต้องให้การ mark read จัดการ row ซ้ำได้ดีขึ้น

#### `sts-portal/components/_layout/main/header.tsx`
ใช้ถ้าต้อง dedupe display ฝั่ง JOB bell/toast ให้เนียนขึ้น

สามไฟล์นี้ไม่ใช่ตัวบังคับสำหรับการทำ `warn 1 ครั้ง + over 1 ครั้ง`
แค่ช่วยเก็บ UX ถ้ามีข้อมูลซ้ำเก่าค้างอยู่แล้ว

## Proposed implementation detail

### Existing alert action format
ตอนนี้ worker สร้าง action แบบนี้
- `SLA-WARN|{JobNo}|{ServiceNo}|{Minutes}`
- `SLA-OVER|{JobNo}|{ServiceNo}|{Minutes}`

ตรงนี้ใช้ต่อได้เลย

### Dedupe rule to use
ให้เช็กจาก `AlertQ` ตามนี้

สำหรับ `WARN`
- `EXISTS AlertQ WHERE JobId = currentJobId AND Action LIKE 'SLA-WARN|%'`

สำหรับ `OVER`
- `EXISTS AlertQ WHERE JobId = currentJobId AND Action LIKE 'SLA-OVER|%'`

### Resulting behavior
- งานหนึ่งจะมี `SLA-WARN` ได้มากสุด 1 record
- งานหนึ่งจะมี `SLA-OVER` ได้มากสุด 1 record
- ถ้างาน warn มาก่อน แล้ววันต่อมา over ค่อยยิง over ได้อีก 1 record
- หลัง over แล้ว จะไม่ยิง over ซ้ำอีก

## Acceptance criteria
ถือว่าเสร็จเมื่อได้ครบทุกข้อ

### Scenario A
งานยังไม่เคยมี alert มาก่อน และเข้า warn
- ต้องยิง `SLA-WARN` 1 ครั้ง
- scan รอบต่อไปต้องไม่ยิง `SLA-WARN` ซ้ำ

### Scenario B
งานเคยมี `SLA-WARN` แล้ว และต่อมาเข้า over
- ต้องยิง `SLA-OVER` 1 ครั้ง
- scan รอบต่อไปต้องไม่ยิง `SLA-OVER` ซ้ำ

### Scenario C
งานเข้า over โดยไม่เคยผ่าน warn หรือ warn หลุดไปแล้ว
- ต้องยิง `SLA-OVER` ได้ 1 ครั้ง

### Scenario D
งานเดิมค้างอยู่ใน state เดิมนานหลายชั่วโมง
- ต้องไม่มี alert ใหม่เพิ่ม

## Risks / things to verify before coding

### R1. `JobId` ของ CM กับ PM อยู่คนละ table แต่ใช้ field เดียวกันใน `AlertQ`
ตอนนี้ worker เอา `Job.JobId` กับ `JobPM.JobPMId` มา map ลง `AlertQ.JobId` field เดียว

ต้องระวัง:
- ถ้า `CM.JobId = 123` และ `PM.JobPMId = 123` เกิดชนกันได้
- ถ้าชนจริง การ dedupe แบบดูแค่ `JobId` อย่างเดียวจะผิด

ทางตัดสินใจที่ต้องเช็กตอนลงมือ:
- ถ้า id space ของสอง table ไม่ชนกันจริง ใช้ `JobId` เดิมได้
- ถ้ามีโอกาสชน ต้องรวม `JobType` เข้าไปใน dedupe condition ด้วย

นี่เป็น risk สำคัญสุดก่อนแก้

### R2. multiple instance race
ถ้ามีหลาย worker รันพร้อมกัน และ query กับ insert ไม่ atomic
- worker A เช็กว่าไม่มี
- worker B เช็กว่าไม่มี
- ทั้งคู่ insert พร้อมกัน

minimum fix รอบนี้อาจยังยอมรับ risk นี้ไว้ก่อน ถ้า goal คือหยุด spam ส่วนใหญ่จาก cooldown logic

แต่ถ้า prod ยังมีหลาย instance active จริง จุดนี้ต้องตามเก็บ

## Recommended sequence when we come back to implement
1. เปิด `SLAMonitorWorker.cs`
2. เช็กก่อนว่า `CM.JobId` กับ `PM.JobPMId` มีโอกาสชนกันไหม
3. แก้ query dedupe เป็น stage-based
4. ตัด cooldown ออกจาก requirement logic
5. test 4 scenarios ข้างบน
6. ถ้ายังมี duplicate เยอะผิดปกติ ให้เช็กจำนวน instance ของ `STS-NOC`

## Short answer for future me / future AI
ถ้าคำถามคือ:
> จะทำให้มันแจ้งเตือน ใกล้เกิน 1 ครั้ง หลังเกิน 1 ครั้ง ต่อ 1 งาน ต้องแก้ไฟล์ไหนบ้าง แบบไม่ overengineer

คำตอบคือ:
- แก้จริงแค่ `STS-NOC/Workers/SLAMonitorWorker.cs`
- review เพิ่ม `STS-NOC/Program.cs` ถ้ามีหลาย instance รัน worker พร้อมกัน
- ไม่ต้องแตะ DB schema, SP, หรือ frontend ถ้าเอาเฉพาะ logic นี้ก่อน

## Notes from the current conversation
- expectation ของ user: `warn 1 ครั้ง + over 1 ครั้ง ต่อ 1 งาน`
- observation จาก production-ish behavior: หลังเปลี่ยน server มีอาการเตือนสนั่น
- probable causes:
  - cooldown-based logic ทำให้ยิงซ้ำทุก 30 นาที
  - อาจมีหลาย instance ของ worker รันพร้อมกัน
- implementation style requested: `KISS`, ไม่ overengineer, เน้นแก้ตรงจุด
