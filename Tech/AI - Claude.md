---
title: AI - Claude
tags:
  - ai
  - claude
  - anthropic
  - coding-agent
created: 2026-04-12
updated: 2026-04-12
---

# Claude — Claude Code Pro Plan

> เรียกด้วย: **`Claude`**
> Provider: [[Provider - Anthropic|Anthropic]]

---

## ข้อมูลทั่วไป

- · Provider: Anthropic
- · Plan: Pro Plan
- · Model ปัจจุบัน: claude-sonnet-4-6 / claude-opus-4-6
- · Interface หลัก: Claude Code CLI (Terminal), claude.ai/code, VS Code Extension

---

## จุดแข็ง

- · Context window ยาวมาก — อ่านไฟล์ใหญ่ได้ทั้งก้อน
- · Memory system — จำ project context ข้ามบทสนทนาได้
- · Agentic tools — อ่าน/แก้ไฟล์, รัน bash, search codebase ได้เอง
- · Code reasoning แม่นยำ — วิเคราะห์ logic ได้ลึก
- · MCP integration — ต่อ Figma, Supabase, Gmail, Google Calendar ได้
- · Skill system — เรียก /commit, /review-pr, /loop ได้
- · ไม่ hallucinate API ที่ไม่มีอยู่จริง (เมื่อเทียบกับตัวอื่น)

---

## เมื่อไหร่ควรใช้

- · งาน refactor หรือ debug ที่ต้องอ่านหลายไฟล์พร้อมกัน
- · งานที่ต้องการ context จาก memory เดิม
- · งาน architecture หรือ planning ที่ต้องคิดหลายชั้น
- · งานที่ต้องการความแม่นยำสูง เช่น SLA formula, business logic

---

## วิธีเรียกใช้

- · Terminal: `claude` แล้ว type prompt
- · IDE: Claude Code extension ใน VS Code
- · Web: claude.ai/code
- · Slash commands: `/commit`, `/loop`, `/review-pr`

---

## Links

- ← [[AI Agents - My Arsenal]]
