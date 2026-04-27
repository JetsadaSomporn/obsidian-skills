---
title: Payment Tracker - UI Design
tags:
  - ui
  - design
  - css
  - tailwind
type: note
status: active
created: 2026-04-25
updated: 2026-04-25
links:
  - "[[Payment Tracker - Overview]]"
---

# UI Design

> Aesthetic direction: Apple / OpenAI / Google — minimal, confident, product-forward

## Design Tokens (globals.css)

```css
/* Light */
--background: #F5F5F7;   /* Apple off-white */
--foreground: #111111;
--muted:      #6B7280;
--panel:      #FFFFFF;
--chrome:     #F0F1F3;   /* Sidebar background */
--soft:       #F7F7F8;
--line:       #E5E7EB;
--accent:     #2563EB;
--good:       #059669;
--bad:        #DC2626;

/* Dark */
--background: #0E0E0F;
--foreground: #F0EFEB;
--panel:      #181817;
```

## Typography
| Role | Font | CSS Variable |
|---|---|---|
| Hero / Display | Bricolage Grotesque | `--font-display` |
| Body | DM Sans | `--font-sans` |
| Numbers / Money | DM Mono | `--font-mono` |

- ทั้ง 3 ตัวโหลดผ่าน `next/font/google` → self-hosted, CSP-safe
- `.font-figures` — `font-variant-numeric: tabular-nums` สำหรับตัวเลขเงิน

## Component Classes (globals.css)

| Class | ใช้ทำ |
|---|---|
| `.app-canvas` | Wrapper หลักของ app — override CSS tokens |
| `.sidebar-chrome` | Sidebar พื้นหลัง `var(--chrome)` + border |
| `.app-hero-panel` | Hero panel สีดำ `#0A0A0A` ไม่มี gradient |
| `.surface` | Card ทั่วไป — border + panel bg |
| `.hero-amount` | ตัวเลขหลัก `clamp(3rem, 8vw, 6.5rem)` |
| `.metric-tile` | ช่องสถิติ 4 ช่อง |
| `.nav-link` | Sidebar nav item |
| `.field` | Input / Select |
| `.primary-button` | ปุ่มหลัก |
| `.animate-in` | fade-up animation 280ms |
| `.delay-1` ~ `.delay-4` | animation-delay stagger |

## Layout

```
┌─────────────────────────────────────────────┐
│ Sidebar (224px, slide-able)  │  Content area │
│  Brand mark                  │  TopBar       │
│  Nav links                   │               │
│  Today summary (bottom)      │  View         │
└─────────────────────────────────────────────┘
```

- Sidebar toggle ด้วย `PanelLeftClose / PanelLeftOpen` icon
- Transition `width` + `opacity` 200ms — ไม่มี tab bar ด้านบนแล้ว

## Hero Panel (Dashboard)
```
┌──────────────────────────────────────────┐  bg: #0A0A0A
│ Bangkok · วันที่        [Upload] [Ledger] │
│                                          │
│  ฿  0.00                                 │  Bricolage Grotesque
│                                          │
│  ยังไม่มีรายการวันนี้                      │
│                                          │
│  EXPENSE  INCOME  NET                    │  metric-tile
└──────────────────────────────────────────┘
```

- ไม่มี gradient สี ไม่มี box-shadow — Tesla instrument cluster aesthetic
- `border-top: rgba(255,255,255,0.14)` — subtle bezel เพื่อให้ panel ไม่หายเข้า background

## Landing Page

### Sections
1. **Hero** — Dark full-screen, `hero-field-console.jpg` เป็น texture (overlay 80%+), ไม่ใช่ subject
2. **Product screenshot** — Browser chrome mockup กับ `product-dashboard.png` จริง
3. **Upload flow** — Steps + browser chrome mockup กับ `product-upload.png`
4. **Trust** — Dark section, 3 cards (Auth / Storage / Confidence)
5. **AI** — "Assist extraction. Never invent the ledger."

### Hero Copy
```
"Every baht.
Confirmed."
```
— 3 คำ, confident เหมือน Tesla

### Aesthetic ที่ได้
- Pure dark hero (ไม่มี stock photo ฟาร์มเป็น subject แล้ว)
- Product screenshots จริงในทุก section
- Rounded-full CTAs (Google/OpenAI style)
- Stats strip แทน metric box
- ~75-80% ใกล้เคียง Tesla/OpenAI/Google

### สิ่งที่ยังขาดเพื่อขึ้น 90%
- Scroll animation (intersection observer)
- Thai font ใน hero render สวยขึ้น
