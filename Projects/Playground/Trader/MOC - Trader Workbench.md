---
title: MOC - Trader Workbench
tags:
  - trader
  - moc
  - playground
  - trading
  - finance
type: moc
status: active
created: 2026-04-19
updated: 2026-04-19
links:
  - "[[Projects/Playground/Trader/Trader - Overview]]"
  - "[[Projects/Playground/Trader/Trader - Architecture]]"
  - "[[Projects/Playground/Trader/Trader - Workflow and State]]"
  - "[[Projects/Playground/Trader/Trader - Legacy Context]]"
  - "[[Areas/Finance - Overview]]"
---

# MOC - Trader Workbench

> จุดเข้า note cluster ของโปรเจกต์ Trader: autonomous paper-trading bot บน Claude Code + Alpaca

## Start Here
- [[Projects/Playground/Trader/Trader - Overview|Trader - Overview]]
- [[Projects/Playground/Trader/Trader - Architecture|Trader - Architecture]]
- [[Projects/Playground/Trader/Trader - Workflow and State|Trader - Workflow and State]]
- [[Projects/Playground/Trader/Trader - Legacy Context|Trader - Legacy Context]]

## Verified Current State (2026-04-19)
- repo ยังอยู่ระยะ scaffold/bootstrap มากกว่าระยะ production
- git history ที่เห็นตอนนี้มี 2 commits: `Initial commit` และ `bootstrap trading bot scaffold per Nate Herk guide`
- `memory/TRADE-LOG.md` ยังเป็น baseline day 0 ไม่มี position
- `memory/RESEARCH-LOG.md` กับ `memory/WEEKLY-REVIEW.md` ยังเป็น template รอรันจริง
- runtime model ปัจจุบันคือ prompt + shell wrapper + markdown memory ไม่ใช่ Python trading engine

## Working Boundary
- Active runtime: `CLAUDE.md`, `routines/`, `scripts/`, `memory/`, `env.template`
- Local operator helpers: `.claude/commands/`, `.claude/settings.local.json`
- Reference only: `Trading.pdf`
- Archive only: `legacy_data/`

## Cluster Map
- [[Projects/Playground/Trader/Trader - Overview|Project overview + readiness verdict]]
- [[Projects/Playground/Trader/Trader - Architecture|Execution model, components, local vs cloud]]
- [[Projects/Playground/Trader/Trader - Workflow and State|Routine matrix, memory ownership, trade lifecycle]]
- [[Projects/Playground/Trader/Trader - Legacy Context|Old stock-analysis archive and source lineage]]

## Recent Trader Notes
```dataview
TABLE file.link AS Note, type AS Type, updated AS Updated
FROM "Projects/Playground/Trader"
WHERE file.name != "MOC - Trader Workbench"
SORT updated DESC
```
