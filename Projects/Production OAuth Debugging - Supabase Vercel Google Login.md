# Production OAuth Debugging Postmortem: Supabase + Vercel + Google Login

## บทสรุป (Summary)
การแก้ไขปัญหา Google OAuth ลอ็กอินบน Production (Vercel) ไม่สำเร็จ ทั้งที่ใช้งานบน Localhost ได้ปกติ โดยพบว่าปัญหาไม่ได้เกิดจากหน้าต่างลอ็กอินหรือ Provider โดยตรง แต่เกิดจาก 2 ประเด็นหลัก คือ **React Hydration ล้มเหลวเนื่องจาก SRI** และ **Session หลุดเมื่อ Refresh หน้าเว็บเนื่องจากการจัดการ Cookie ใน Middleware ผิดพลาด**

## ผลกระทบ (Impact)
1. **ลอ็กอินไม่ได้เลย (ตอนแรก):** ผู้ใช้งานไม่สามารถกดปุ่ม "Sign in with Google" ได้เลย (คลิกแล้วนิ่ง) เนื่องจาก Script ของหน้าเว็บถูกบล็อก
2. **Session หลุด (หลังแก้บั๊กแรก):** เมื่อผู้ใช้ล็อกอินสำเร็จและเข้าไปในแอปได้แล้ว แต่พอ Refresh หน้าเว็บ (หรือเปิดแท็บใหม่) ระบบจะลืมสถานะทันที และกลับไปแสดงปุ่ม "Sign in" เหมือนเดิม ทำให้แอปพลิเคชันไม่สามารถใช้งานจริงได้

## สมมติฐานแรกเริ่ม (Initial Assumption)
ในตอนแรกทีมงานสันนิษฐานว่าปัญหาอาจเกิดจากการตั้งค่า **Redirect URL** ใน Supabase หรือ Google ไม่ถูกต้อง หรืออาจโดนบล็อกโดเมนคำว่า `vercel.app` (Phishing Protection) นอกจากนี้ยังสงสัยว่าอาจเกิดจากการตั้งค่า `SameSite=Strict` ใน Cookie

## สาเหตุที่แท้จริง (Actual Root Cause)
ปัญหาที่ทำให้โปรเจกต์ใช้งานจริงไม่ได้แบ่งเป็น 2 ช่วง:
1. **SRI Mismatch (ช่วงแรก):** ในไฟล์ `next.config.ts` มีการเปิด `experimental.sri` ไว้ ทำให้ Browser ตรวจสอบ Hash ของไฟล์ JS แต่ Vercel ส่งไฟล์ที่มี Hash ไม่ตรงมาให้ Browser จึงบล็อก Script ไม่ให้ทำงาน React จึงไม่รัน (Hydration Failed) ทำให้ปุ่มทำงานไม่ได้
2. **Cookie Dropped / Middleware Overwrite (ช่วงหลัง):** หลังจากแก้เรื่อง SRI แล้ว ลอ็กอินได้ แต่พอรีเฟรชแล้วหลุด เกิดจาก:
   - ใน Next.js App Router (โดยเฉพาะบน Vercel Edge) การสั่ง `NextResponse.redirect()` อาจ**ไม่ส่งผ่าน (Drop) Cookie** ที่เพิ่งถูกเซ็ตโดย Supabase กลับไปยัง Browser ในจังหวะที่รับ Callback
   - ไฟล์ Middleware (`src/proxy.ts`) มีการเรียกใช้คำสั่ง `NextResponse.next({ request: { headers } })` ซึ่งเป็นการ**เขียนทับ (Overwrite) Request เดิม** ทำให้ Cookie ที่ถูกสั่งให้อัปเดตโดยฟังก์ชัน `setAll()` ของ Supabase ถูกทำลายทิ้งไปก่อนที่จะส่งถึง Client

## ลำดับเหตุการณ์การแก้ไขปัญหา (Timeline of Debugging)
1. **ปัญหา Domain:** ถูก Supabase บล็อกโดเมนเนื่องจากมีคำว่า "payment" (Phishing Protection)
2. **เปลี่ยน Domain:** ย้ายไปใช้ `spendly-lovat.vercel.app`
3. **ตรวจสอบ OAuth Config:** ตรวจสอบค่า Client ID/Secret และ Redirect URIs พบว่าถูกต้องทั้งหมด
4. **พบอาการแปลก:** Localhost ลอ็กอินได้ แต่ Production ปุ่มนิ่งสนิท
5. **แก้ Cookie (ครั้งที่ 1):** ปรับ SameSite ของ csrf-token จาก `Strict` เป็น `Lax` (ยังแก้ไม่ได้)
6. **Bypass Middleware:** เพิ่มข้อยกเว้นไม่ให้ Proxy รันในหน้า `/auth/callback` (ยังแก้ไม่ได้)
7. **ตรวจสอบ Network:** ไม่พบ Request วิ่งไปหา Supabase หรือ Google เลย แสดงว่าปุ่มไม่ได้ทำงานจริง
8. **ตรวจสอบ Console:** พบ Error `Failed integrity metadata check` (SRI Block)
9. **แก้ SRI:** ลบ `experimental.sri` ออกจาก `next.config.ts` -> **ลอ็กอินได้แล้ว แต่รีเฟรชแล้วหลุด!**
10. **แก้ Session (รอบสุดท้าย):** 
    - แก้ไขให้ `src/proxy.ts` ส่งต่อ Cookie อย่างถูกต้อง (อัปเดตกระบวนการใน `setAll()`)
    - บังคับยัด Cookie ของ Supabase (ที่ชื่อขึ้นต้นด้วย `sb-`) ใส่ลงใน `NextResponse.redirect()` แบบเจาะจงในไฟล์ `src/app/auth/callback/route.ts` เพื่อการันตีว่า Vercel ไม่ทิ้ง Cookie
    - เพิ่มสถานะ Loading (`isLoadingAuth`) ลงใน State เพื่อแก้ปัญหา UI สั่น (Flicker) ระหว่างรอเช็ค Session

## หลักฐานการตรวจพบ (Debug Evidence)
- **Console Error (ช่วงแรก):** ข้อความชัดเจนว่าไม่สามารถโหลด Script ของ Next.js ได้ (SRI Block)
- **Vercel Logs (ช่วงหลัง):** พบ Log `[auth-callback] exchange result { exchangeSuccess: true }` แต่หลังจาก Redirect เข้าแอปแล้วกลับไม่พบ Session 
- **ปัญหา Localhost VS Production:** Localhost ผ่านได้เพราะ Dev Server ไม่ได้จำลองระบบ SRI และกระบวนการของ Vercel Edge ที่เข้มงวดเรื่อง Header Mutation

## รายละเอียดการแก้ไข (Fix Details)
- **`next.config.ts`:** ลบ `experimental.sri` และนำ `X-Frame-Options: DENY` ออก
- **`src/proxy.ts`:** 
    - อัปเดต `setAll` ให้สร้าง Response ใหม่ผ่าน `NextResponse.next({ request })` เพื่อรักษาข้อมูล Cookie เก่าไว้ แล้วค่อยทยอย copy ตัวใหม่ลงไป แทนที่จะเขียนทับแบบดื้อๆ
    - ใส่คำสั่งให้ข้ามการทำงานในหน้า `/auth/callback`
- **`src/app/auth/callback/route.ts`:** 
    - เขียน Loop กวาดเอา `cookieStore.getAll()` มาเช็คหาตัวที่มีคำว่า `sb-` แล้วบังคับยัดลงใส่ `response.cookies.set(...)` ก่อนที่จะ `return response` เพื่อการันตีว่า Session ไม่หล่นหาย
- **`src/components/payment-tracker-app.tsx`:** 
    - เพิ่มตัวแปร `isLoadingAuth` (เริ่มต้น `true`) ไว้ป้องกันอาการแสดงปุ่ม "Sign in" กระพริบ (Flicker) ก่อนที่ `supabase.auth.getSession()` จะโหลดเสร็จ

## บทเรียนในระดับโค้ด (Code-Level Lessons)
- **อย่าเปิด Experimental SRI มั่วซั่ว:** ถ้าโปรเจกต์ไม่ได้บังคับหรือคุณไม่ชัวร์ว่า Vercel Build ทำงานอย่างไร อย่าเปิด `sri` เพราะมันมักจะระเบิดบน Production
- **ระวังการ Overwrite Request ใน Next.js Middleware:** การแก้ไข Request Headers อาจทำให้ Cookie ที่ฝังอยู่ถูกทำลายทิ้ง (Lost in transit)
- **Explicit Cookie Setting:** ใน Next.js (App Router) การเขียน Cookie ในจังหวะที่สั่ง `NextResponse.redirect()` เป็นสิ่งที่พังง่ายที่สุด (Flaky) วิธีชัวร์ที่สุดคือต้อง Explicit หยิบ Cookie มายัดใส่ Response 객체 ก่อน return
- **Auth UI ต้องมี Loading State เสมอ:** การเช็ค Session จาก Browser มีความหน่วง (Async) หากไม่มี Loading State ยูสเซอร์จะเห็น UI กระพริบ

## บั๊กที่ห้ามพลาดซ้ำสอง (Bugs That Should Not Be Missed Again)
- [ ] หากกดปุ่มแล้ว "นิ่ง" ให้เปิด Console ดู Error ก่อนเสมอ (อย่าเพิ่งไปด่า Supabase)
- [ ] การเปลี่ยน Config ด้าน Security/Build ต้องสั่ง **Clear Build Cache** ตอน Deploy เสมอ
- [ ] หากลอ็กอินได้ แต่รีเฟรชแล้วหลุด สาเหตุ 99% มาจากการเขียน Middleware จัดการ Cookie ผิดพลาด
- [ ] หากใช้ `SameSite=Strict` ให้ระวังว่า Auth Flow ข้ามโดเมนจะพัง ให้เปลี่ยนเป็น `Lax`
- [ ] การใช้งาน `@supabase/ssr` ในฝั่ง Server Component (เช่น `route.ts`) มักจะมีปัญหาการเขียน Cookie กลับ หากไม่มีการจัดการ Object Response ให้ดี

## เช็คลิสต์สำหรับการ Deployment ในอนาคต (Future Deployment Checklist)
- [ ] ยืนยัน URL ของโปรเจกต์จริง (Production Domain)
- [ ] ตั้งค่า **Site URL** ใน Supabase Dashboard ให้ตรงกับ Production Domain
- [ ] เพิ่ม **Redirect URLs** ทุกรูปแบบที่จะใช้ (รวมถึง `http://localhost:3000/auth/callback` สำหรับตอน Dev)
- [ ] ตั้งค่า **Authorized redirect URIs** ใน Google Cloud Console (ต้องชี้ไปที่ URL ของโปรเจกต์ Supabase ไม่ใช่ Vercel)
- [ ] ตรวจสอบ Vercel Environment Variables (`NEXT_PUBLIC_SUPABASE_URL`, `NEXT_PUBLIC_SUPABASE_ANON_KEY`)
- [ ] ตรวจสอบว่าโค้ดไม่มีการผูก Domain เก่า หรือ Localhost แบบ Hardcode ทิ้งไว้
- [ ] มั่นใจว่าหน้า `/auth/callback` ถูกข้าม (Bypass) ใน Middleware เพื่อเลี่ยงการขัดกันของ Session
- [ ] มั่นใจว่า Middleware ได้รับการอัปเดตวิธี `setAll()` แบบที่ไม่ทำลาย Cookie 
- [ ] มี `isLoading` state หุ้มปุ่มลอ็กอินเพื่อป้องกัน UI สั่น
- [ ] ลอ็กอินผ่าน Incognito แล้ว **ทดสอบกด Refresh ทันที** ว่าสถานะหลุดหรือไม่

## สถานะการทำงานปัจจุบัน (Final Working State)
แอปพลิเคชันทำงานได้อย่างสมบูรณ์แบบบน Production (Vercel) ผู้ใช้สามารถล็อกอินด้วย Google OAuth ได้ราบรื่นและสถานะการลอ็กอินยังคงอยู่ (Persist) แม้จะทำการ Refresh หรือเปิดแท็บใหม่ นอกจากนี้ UI ยังได้รับการปรับปรุงให้มีความสวยงาม (Premium Minimal) และไม่เกิดอาการสั่นกระพริบระหว่างโหลด 

## Tags
#debugging #postmortem #nextjs #vercel #supabase #oauth #google-login #csp #sri #cookies #middleware #production-bug

---
**Last Updated:** 2026-04-27
**Status:** ✅ Solved & Documented