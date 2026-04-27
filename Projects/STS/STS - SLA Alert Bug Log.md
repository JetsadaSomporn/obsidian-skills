---
tags: [STS, alert, SLA, bug-log, troubleshooting]
created: 2026-04-17
updated: 2026-04-17
---

# STS — SLA Alert Bug Log

Related:
- [[Projects/STS/STS - SLA Alert Implementation|STS - SLA Alert Implementation]]
- [[Projects/STS/STS - SLA Alert Troubleshooting Playbook|STS - SLA Alert Troubleshooting Playbook]]

## Purpose
เก็บ bug จริงที่เจอระหว่างทำ `SLA Alert` พร้อม root cause และ fix ที่ใช้ เพื่อกันการวนกลับไปชน bug ซ้ำ

## Bug table

| Symptom | Root cause | Fix used | Prevention next time |
|---|---|---|---|
| DB มี alert แต่หน้าเว็บไม่เห็น | flow JOB เดิมอิง `PersonId` จาก session | เพิ่ม unread path แบบ `UserId -> PersonId` ใน backend | ถ้า alert เป็น person-scoped อย่า assume ว่า session มี personId ถูกเสมอ |
| ยิง test alert แล้วหน้าเงียบ | หน้าเว็บกับ DB/API คนละ environment | ตรวจ `NEXT_PUBLIC_API_ALERT_SERVICE`, process ที่รัน, และ DB target | ก่อน debug UI ให้พิสูจน์ก่อนว่าหน้าเว็บคุยกับ service/DB ชุดเดียวกับที่ insert |
| refresh แล้ว snackbar เด้ง unread เก่าซ้ำ | unread ชุดแรกถูกตีเป็น new arrival | seed unread ชุดแรกเป็น baseline | toast logic ต้องแยก initial hydration ออกจาก live arrival เสมอ |
| JOB bell ขึ้น แต่ snackbar ไม่ขึ้น | diff logic อยู่หลัง render ใน `useEffect` แล้วพลาดจังหวะ | ย้าย diff ไปอยู่ใน poll cycle | ถ้าฟีเจอร์อาศัย "before/after fetch" ให้เทียบใน fetch flow โดยตรง |
| snackbar ปิดได้ไม่หมดเมื่อซ้อนกัน | dismiss ไปโดนแค่ toast แรก | เก็บ active toast ids และ dismiss ทั้งก้อน | ถ้ามี stacked toasts ต้องมี state ของ active ids ไม่พอแค่ toast.dismiss() ตัวเดียว |
| snackbar มีปุ่มปิดซ้ำซ้อน | ใช้ทั้ง text button และ close button ของ toaster | เหลือ `x` อย่างเดียว | อย่ามี close affordance สองชุดใน toast เดียว |
| ปุ่ม x อยู่ผิดตำแหน่ง | sonner default class ไม่ตรง design | override `closeButton` class | ถ้าใช้ shared toaster ให้ fix style ที่ wrapper กลาง ไม่ใช่แค่เฉพาะหน้า |
| same job same status เด้งซ้ำทุกครั้งที่ insert | dedupe อิง `AlertQId` ซึ่งเปลี่ยนทุกครั้ง | เปลี่ยน dedupe key เป็น `status + jobId` | bell กับ snackbar ต้องมี policy คนละชุด |
| แก้ rule จน job ใหม่ไม่เด้งเลย | initial hydration กับ dedupe seed กลืน new arrival ที่เข้าก่อน poll แรกตอบ | ปรับ baseline ให้ทำที่ poll รอบแรกแบบระวังช่วงเวลา | เวลาแก้ dedupe อย่าทำให้ first live arrival กลายเป็น stale |
| click-away dismiss หายแค่ตัวแรก | logic dismiss ไม่เก็บทุก toast id | รวม active ids + history/toasts แล้ว dismiss ทั้งหมด | toast stack ต้องมี bulk-dismiss strategy |
| PM/CM semantics ถูกตีความเหมือนกัน | จริง ๆ ใช้คนละ function และคนละความหมายของ over | แยกเอกสาร calculation path ชัดเจน | ทุกครั้งที่ debug SLA ให้ตอบก่อนว่าเป็น CM หรือ PM |

## Calculation-specific traps

### 1. CM and PM are not the same
- `CM` ใช้ `fCalcNetSLA`
- `PM` ใช้ `fCalsNetSLAPM`
- อย่า debug SLA แบบคิดว่าทั้งสองฝั่งใช้ `% SLA` แบบเดียวกัน

### 2. PM over-SLA is not the same thing as PM percentage
- worker ใช้ `SLAPercent >= 80` สำหรับ warn
- worker ใช้ `IsDelay = TRUE` สำหรับ over
- ถ้า `JobFixDate` กับ `SLA ชั่วโมง` ไม่ align กัน `%` กับ over อาจดูขัดกันได้

### 3. Action payload minutes is not percentage
- action format ตอนนี้ลง `MinutesValue`
- อย่าไป parse ช่องสุดท้ายเป็น `% SLA`

## Debug order that saves time
1. พิสูจน์ก่อนว่า alert เข้า `AlertQ/AlertTo` แล้ว
2. พิสูจน์ว่า unread endpoint คืน alert ลูกนั้นได้
3. พิสูจน์ว่าหน้าเว็บเรียก endpoint ถูก env
4. ค่อยดู parse, route, snackbar
5. ถ้าเป็น PM/CM issue ให้เช็ก function คนละตัว อย่ารวมกัน

## Hard rules for future AI / dev
- อย่าเดา `UserId = PersonId`
- อย่าเดา `warn` และ `over` ของ PM ใช้ metric เดียวกัน
- อย่าเดาว่า unread ที่เห็นใน DB จะขึ้นใน UI ถ้ายังไม่พิสูจน์ env
- อย่าเริ่มจาก frontend ก่อนถ้ายังไม่ยืนยัน DB/API target
- อย่าใช้ `AlertQId` เป็น dedupe key ของ realtime UX ถ้าต้องการกัน spam รายงานเดิม
