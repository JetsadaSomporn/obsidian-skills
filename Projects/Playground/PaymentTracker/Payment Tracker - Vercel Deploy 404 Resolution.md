---
title: Payment Tracker - Vercel Deploy 404 Resolution
tags:
  - payment-tracker
  - vercel
  - deploy
  - incident
type: incident
status: resolved
created: 2026-04-25
updated: 2026-04-25
links:
  - "[[Payment Tracker - Overview]]"
  - "[[Payment Tracker - Handoff 2026-04-25]]"
---

# Vercel Deploy 404 Resolution

> Build ผ่าน แต่เข้า URL ไม่ได้ เพราะเป็น Vercel access/domain/deployment issue ไม่ใช่ Next route หาย

## Symptom

Vercel build log:

```text
Status: Ready
Environment: Production
Route (app)
┌ ○ /
├ ƒ /api/slips/process
├ ƒ /api/transactions
├ ƒ /app
├ ○ /upload
└ ○ /transactions
Build Completed
Deployment completed
```

แต่เปิด URL แล้วเจอ:

```text
404: NOT_FOUND
Code: NOT_FOUND
```

และมี warning:

```text
Warning: Failed to fetch one or more git submodules
```

## What Was Verified

Curl production alias:

```text
https://payment-tracking-tawny.vercel.app
-> 404
-> x-vercel-error: NOT_FOUND
```

Curl deployment/branch URL:

```text
payment-tracking-git-main-...
payment-tracking-erbjq4cai-...
-> 401 Authentication Required
```

Interpretation:
- `401` means deployment exists but Vercel Deployment Protection is active
- `404 x-vercel-error: NOT_FOUND` from production alias means Vercel edge/domain mapping issue, not Next route issue
- Next app had route `/` in build output

## Root Causes

### 1. Deployment Protection

Vercel project had:

```text
Deployment Protection: Standard Protection
```

This blocks public access to deployment URLs unless authenticated through Vercel.

### 2. Domain alias / project mapping confusion

Production alias responded with Vercel edge 404 even though deployment was Ready.

Likely domain alias not mapped cleanly to the current production deployment.

### 3. Broken git submodule link

Repo root had `Payment Tracking` tracked as gitlink/submodule:

```text
160000 commit ... Payment Tracking
```

But no `.gitmodules` mapping existed:

```text
fatal: no submodule mapping found in .gitmodules for path 'Payment Tracking'
```

This caused Vercel warning:

```text
Failed to fetch one or more git submodules
```

## Fixes Applied

### Git repo fix

Ran:

```bash
git rm --cached -f 'Payment Tracking'
```

Added to `.gitignore`:

```gitignore
# local nested repo, not part of the Vercel app
/Payment Tracking/
```

Committed and pushed:

```text
65a3204 chore: remove broken submodule link
```

Validation:

```bash
npm run lint
npm run build
```

Both passed.

### Vercel project fix

User ultimately deleted the problematic Vercel project and created/imported it again.

Result:

```text
ตอนนี้ทำงานได้ละ
```

## Debug Decision Tree For Next Time

### Build fails

Check:
- build logs
- env vars
- dependency install
- TypeScript/lint

### Build passes but deployment URL is 401

Check:
- Vercel Deployment Protection
- project/team access
- share URL / bypass token

### Build passes but production alias is 404

Check:
- Domains tab
- alias assignment
- project import mismatch
- latest production deployment
- project was renamed/deleted/recreated

### Vercel warns about submodules

Check:

```bash
git ls-tree HEAD
find . -name .gitmodules -print
git submodule status
```

If a path is mode `160000` but `.gitmodules` has no mapping, remove gitlink or correctly configure submodule.

## Final Lesson

Do not treat Vercel `404: NOT_FOUND` as app route failure immediately.

Separate:

```text
Next build routes
Vercel deployment protection
Vercel domain alias
Vercel project import/submodule state
```
