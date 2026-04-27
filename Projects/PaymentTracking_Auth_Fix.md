---
title: "Case Study: Fix Google Auth in Next.js 16 with Supabase"
created: 2026-04-27
status: completed
tags: [nextjs, supabase, auth, debugging]
---

# สรุปการแก้ไขปัญหา Google Login - Payment Tracking

## 1. โจทย์ (Objective)
แก้ไขปัญหาระบบลอ็กอินด้วย Google ของโปรเจกต์ Payment Tracking (Next.js 16 + Supabase) ที่ไม่สามารถทำงานได้ ทั้งที่มีการตั้งค่า API Key ครบถ้วนแล้ว

## 2. ปัญหาที่พบ (Root Causes)
จากการตรวจสอบโค้ดอย่างละเอียด พบปัญหา 3 จุดหลัก:
1. **Missing Session Middleware:** ใน Next.js App Router เมื่อใช้ `@supabase/ssr` จำเป็นต้องมี Middleware คอย Refresh Session และ Sync Cookie ระหว่างฝั่ง Client และ Server หากไม่มีจุดนี้ Session จะหายไปทันทีหลังจาก Redirect กลับมาจาก Google
2. **Middleware Conflict (Next.js 16 Custom):** โปรเจกต์นี้ใช้ Next.js เวอร์ชัน 16.2.4 ซึ่งมีการใช้ไฟล์ `proxy.ts` ทำหน้าที่แทน `middleware.ts` มาตรฐาน การสร้างไฟล์ซ้ำซ้อนทำให้ Build พังและ Logic การจัดการ Auth ไม่ทำงาน
3. **Inconsistent Auth Logic:** ตัวช่วยฝั่ง Server (`requireAuthenticatedUser`) ตรวจสอบเฉพาะ Bearer Token แต่ไม่ได้ตรวจสอบ Session จาก Cookies ทำให้การเข้าถึงหน้าเว็บโดยตรง (Direct Navigation) ไม่สามารถระบุตัวตนผู้ใช้ได้

## 3. วิธีการแก้ไข (Solution & Implementation)
ผมได้ดำเนินการแก้ไขดังนี้:
- **Merge Logic เข้า `proxy.ts`:** นำ Logic การ `getUser()` และ `setAll()` ของ Supabase เข้าไปใส่ใน `src/proxy.ts` เพื่อให้ทุก Request มีการตรวจสอบและต่ออายุ Session อัตโนมัติ
- **Cleanup Redundant Files:** ลบไฟล์ `src/middleware.ts` ที่ซ้ำซ้อนออก เพื่อให้ Build ผ่านและใช้ `proxy.ts` เป็น Single Entry Point สำหรับ Middleware Logic
- **Refactor `server.ts`:** ปรับปรุงฟังก์ชัน `requireAuthenticatedUser` ให้เป็นระบบ **Hybrid Auth** (ตรวจสอบ Bearer Token ก่อน ถ้าไม่มีให้ตรวจสอบจาก Session Cookies)
- **Add Debugging System:** เพิ่ม Console Log ใน `src/app/auth/callback/route.ts` เพื่อดักจับข้อมูลจาก Google (Code, Error Message) ทำให้วิเคราะห์ปัญหาในอนาคตได้ง่ายขึ้น

## 4. บั๊กและจุดที่ห้ามพลาด (Lessons Learned & Tips)
- **Cookie Syncing:** สำหรับ `@supabase/ssr` ห้ามลืมเขียน Logic `setAll` ใน Middleware เพราะเป็นจุดที่ใช้เขียน Session กลับลงไปใน Browser Cookies
- **Next.js 16 Proxy Pattern:** ในเวอร์ชันใหม่ๆ หรือโปรเจกต์ที่ตั้งค่าพิเศษ `proxy.ts` อาจถูกใช้แทน `middleware.ts` การมีทั้งสองไฟล์จะทำให้ Next.js สับสนและ Build Error
- **Redirect URLs:** ใน Supabase Dashboard ต้องระบุ `http://localhost:3000/auth/callback` ในรายการ **Redirect URLs** เสมอ ไม่งั้น Google จะไม่ส่ง Auth Code กลับมา
- **Response Handling:** การแก้ไข Response ใน Middleware/Proxy ต้องระวังเรื่องการใช้ `NextResponse.next()` หากมีการแก้ไข Cookie ต้องมั่นใจว่า Response object ที่คืนไปนั้นเป็นตัวที่มีการ Update ข้อมูลแล้ว

---
**Status:** ✅ Fixed & Build Succeeded
