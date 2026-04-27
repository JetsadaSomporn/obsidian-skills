---
title: Trader - Architecture
tags:
  - trader
  - architecture
  - alpaca
  - claude-code
  - trading
type: architecture
status: active
created: 2026-04-19
updated: 2026-04-19
links:
  - "[[Projects/Playground/Trader/Trader - Overview]]"
  - "[[Projects/Playground/Trader/Trader - Workflow and State]]"
  - "[[Projects/Playground/Trader/Trader - Legacy Context]]"
---

# Trader - Architecture

## Design in One Sentence
Claude Code คือ bot ตัวจริง; repo นี้ให้ prompt scaffold, shell wrappers, และ markdown memory เพื่อให้แต่ละรอบรันตัดสินใจเองแบบ stateless แล้วฝาก state กลับเข้า Git

## Component Map

| Component | Source | หน้าที่ |
|-----------|--------|--------|
| Agent rulebook | `CLAUDE.md` | กำหนดภารกิจ, hard rules, communication style, read-me-first order |
| Persistent state | `memory/*.md` | เก็บ strategy, trade history, research history, project context, weekly review |
| Broker wrapper | `scripts/alpaca.sh` | ยิง Alpaca API ผ่าน subcommands เช่น `account`, `positions`, `orders`, `order`, `close` |
| Notification wrapper | `scripts/clickup.sh` | ส่ง ClickUp chat หรือ fallback ลง `DAILY-SUMMARY.md` ถ้า credential ไม่ครบ |
| Cloud execution prompts | `routines/*.md` | ใช้กับ Claude cloud routines ที่ตั้ง cron ภายนอก repo |
| Local execution prompts | `.claude/commands/*.md` | mirror ของ workflow เดียวกันสำหรับรันใน local Claude Code |
| Env contract | `env.template` + process env | local mode ใช้ `.env`; cloud mode ใช้ exported env vars |
| Reference blueprint | `Trading.pdf` | guide ต้นทางของ Nate Herk |

## Execution Modes

| Mode | ใช้อะไร | Credential source | Persist ยังไง |
|------|---------|------------------|--------------|
| Local | `.claude/commands/*.md` | `.env` ที่ `scripts/*.sh` source ถ้ามี | operator commit เอง |
| Cloud | `routines/*.md` | routine-level env vars | prompt บังคับ commit + push ตอนจบ |

## High-Level Flow
1. routine หรือ slash command เริ่มรันใน environment ใหม่
2. agent อ่าน `memory/TRADING-STRATEGY.md` และ logs ที่เกี่ยวข้องก่อน
3. agent ดึง live state ผ่าน `bash scripts/alpaca.sh ...`
4. ถ้าต้องมี research ให้ใช้ native WebSearch แล้วจด source URL ลง log
5. ถ้ามี trade ให้ run rule gates ก่อนทุก order
6. order execution ทั้งหมดต้องผ่าน `scripts/alpaca.sh` เท่านั้น
7. notification ส่งผ่าน `scripts/clickup.sh`
8. state ใหม่ถูก append กลับ `memory/*.md` แล้ว commit/push ถ้าเป็น cloud flow

## Hard Boundaries ที่ตั้งใจไว้
- ห้าม `curl` Alpaca หรือ ClickUp ตรง ๆ; ต้องผ่าน wrapper script
- ห้ามเทรด options
- ห้ามใช้ live endpoint เอง
- ห้ามเทรดโดยไม่มี thesis/catalyst อยู่ใน research log วันนั้น
- hard rule enforcement อยู่ใน prompt contract ไม่ได้อยู่ใน compiled code layer

## Customizations เทียบกับ Nate Herk Blueprint
- ไม่มี `perplexity.sh`; research ใช้ Claude Code native WebSearch แทน
- timezone หลักเป็น `Asia/Bangkok` ไม่ใช่ US timezone
- paper trading only ถูกย้ำทั้งใน `README.md`, `CLAUDE.md`, `env.template`, และ routine prompts

## Nuance ที่ AI ตัวต่อไปควรรู้
- `scripts/*.sh` รองรับ local `.env` แต่ `routines/*.md` ระบุชัดว่าบน cloud ห้ามสร้างหรือ source `.env`; อันนี้ไม่ขัดกัน มันคือแยก local mode กับ cloud mode
- ตัว broker integration จริงอยู่ใน bash wrapper ไม่ใช่ library; ถ้าจะแก้พฤติกรรม order หรือ auth ต้องดู shell ก่อน
- ระบบนี้ไม่มี layer กลางสำหรับ validation/retries นอกจากที่เขียนไว้ใน prompt; ถ้าจะยกระดับ reliability ต้องตัดสินใจว่าจะเพิ่ม code หรือเพิ่ม prompt contract
- persistence อยู่ที่ markdown + git ดังนั้น format drift ใน `memory/*.md` คือเรื่องใหญ่ เพราะ workflow ถัดไปอ่านต่อจากรูปแบบเดิม

## Things Missing on Purpose
- ไม่มี backtest engine
- ไม่มี unit/integration tests
- ไม่มี portfolio analytics module แยก
- ไม่มี database
- ไม่มี scheduler code ใน repo; schedule จริงอยู่บน Claude cloud routines

## Verified From
- `/Users/jetsadasomporn/Vscode/Playground/Trader/README.md`
- `/Users/jetsadasomporn/Vscode/Playground/Trader/CLAUDE.md`
- `/Users/jetsadasomporn/Vscode/Playground/Trader/routines/README.md`
- `/Users/jetsadasomporn/Vscode/Playground/Trader/routines/*.md`
- `/Users/jetsadasomporn/Vscode/Playground/Trader/.claude/commands/*.md`
- `/Users/jetsadasomporn/Vscode/Playground/Trader/scripts/alpaca.sh`
- `/Users/jetsadasomporn/Vscode/Playground/Trader/scripts/clickup.sh`
- `/Users/jetsadasomporn/Vscode/Playground/Trader/env.template`
- `/Users/jetsadasomporn/Vscode/Playground/Trader/Trading.pdf`
