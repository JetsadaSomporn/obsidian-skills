---
title: Payment Tracker - Database Schema
tags:
  - database
  - supabase
  - postgresql
  - rls
type: note
status: active
created: 2026-04-25
updated: 2026-04-25
links:
  - "[[Payment Tracker - Overview]]"
---

# Database Schema

> Supabase (PostgreSQL) — ทุกตารางเปิด RLS และผูกกับ `auth.uid()`

## Tables (8 ตาราง)

### `profiles`
| Column | Type | Note |
|---|---|---|
| id | uuid | FK → `auth.users.id` |
| email | text | |
| display_name | text | |
| created_at | timestamptz | |

- Auto-created เมื่อ user sign up ผ่าน trigger `handle_new_user`
- Function ใช้ `security definer` + `set search_path = public`

### `categories`
| Column | Type | Note |
|---|---|---|
| id | uuid | PK |
| user_id | uuid | FK → profiles |
| name | text | |
| color | text | hex |
| icon | text | |

### `slips`
| Column | Type | Note |
|---|---|---|
| id | uuid | PK |
| user_id | uuid | |
| storage_path | text | Supabase Storage path |
| extracted_json | jsonb | raw AI output |
| confidence | numeric | 0–1 |
| status | text | pending/confirmed/rejected |
| created_at | timestamptz | |

### `transactions`
| Column | Type | Note |
|---|---|---|
| id | uuid | PK |
| user_id | uuid | |
| type | text | income/expense/transfer |
| amount | numeric | |
| fee | numeric | |
| currency | text | THB |
| title | text | |
| ai_category | text | |
| bank_name | text | nullable |
| receiver_name | text | nullable |
| reference_no | text | nullable, encrypted |
| reference_no_hash | text | nullable, HMAC hash for duplicate check |
| transaction_date | date | |
| transaction_time | time | nullable |
| source | text | slip/manual |
| status | text | confirmed |
| created_at | timestamptz | |

### `daily_summaries`
| Column | Type | Note |
|---|---|---|
| id | uuid | PK |
| user_id | uuid | |
| summary_date | date | |
| total_expense | numeric | |
| total_income | numeric | |
| net_amount | numeric | |
| transaction_count | int | |
| top_category | text | |
| ai_insight | text | |
| created_at | timestamptz | |

### `budgets`
| Column | Type | Note |
|---|---|---|
| id | uuid | PK |
| user_id | uuid | |
| category_name | text | |
| monthly_limit | numeric | |
| period_start | date | |

### `merchant_rules`
| Column | Type | Note |
|---|---|---|
| id | uuid | PK |
| user_id | uuid | |
| pattern | text | regex |
| category_name | text | |
| auto_apply | boolean | |

### `ai_logs`
| Column | Type | Note |
|---|---|---|
| id | uuid | PK |
| user_id | uuid | |
| provider | text | deepseek/nvidia |
| model | text | |
| input_tokens | int | |
| output_tokens | int | |
| latency_ms | int | |
| created_at | timestamptz | |

## Sensitive Field Policy

หลังรอบ hardening 2026-04-25:

| Column | Storage |
|---|---|
| `title` | AES-256-GCM encrypted text |
| `bank_name` | AES-256-GCM encrypted text |
| `receiver_name` | AES-256-GCM encrypted text |
| `reference_no` | AES-256-GCM encrypted text |
| `reference_no_hash` | HMAC-SHA256 of normalized reference number |

Reason:
- AES-GCM ใช้ random IV ทำให้ encrypted output ของค่าเดิมไม่เท่ากัน
- unique duplicate detection จึงต้องใช้ hash แยก

## RLS Policies (pattern เดียวกันทุกตาราง)

```sql
-- SELECT
CREATE POLICY "user_select" ON transactions
  FOR SELECT USING (auth.uid() = user_id);

-- INSERT
CREATE POLICY "user_insert" ON transactions
  FOR INSERT WITH CHECK (auth.uid() = user_id);

-- UPDATE / DELETE (similar)
```

## Storage Bucket
- Bucket: `slips` (private)
- Policy ล่าสุดใช้ hashed folder path:

```sql
md5(auth.uid()::text) = (storage.foldername(name))[1]
```

Note: application route ตอนนี้ยังไม่ได้ upload slip image เข้า Storage จริง เป็น pending work

## Migration
```
supabase/migrations/001_initial_schema.sql
supabase/migrations/002_storage_rls_hash.sql
supabase/migrations/003_rate_limits.sql
supabase/migrations/004_transaction_reference_hash.sql
```
