# ADR-0007: Schema and data retention

- Status: accepted; schema goes to the DPO for review before iteration 3
- Date: 2026-11-24
- Deciders: team

## Context

Session records have no direct identifiers (we do not call the data anonymous). Session records are kept 24 months. Comments, if added, are kept 12 months.

## Options

1. Delete with a scheduled job in the application.
2. A database feature for automatic expiry. Azure SQL has no row-level time-to-live.

## Decision

Option 1: a daily timer Function deletes in batches of 5,000 rows where `session_hour` is older than 24 months, and logs only the number of deleted rows. Comments (Phase 2): a fixed list of choices only, 12 months. Other retention: Application Insights 30 to 90 days, queue message TTL 7 days, poison queue cleanup after 14 days.

Schema (model A shown; model C has no `tenant_id`):

| Column        | Type             | Note                                                  |
| ------------- | ---------------- | ----------------------------------------------------- |
| tenant_id     | uniqueidentifier | Part of the unique key                                |
| event_id      | uniqueidentifier | Created on the tablet                                 |
| age_bracket   | tinyint code     | 5-8, 9-12, 13-18 (the old 12-18 overlapped at 12)     |
| mode          | char(1)          | b (blow) or t (tap)                                   |
| completed     | bit              |                                                       |
| duration_s    | smallint         | CHECK 0 to 1800                                       |
| rating        | tinyint, null    | CHECK 1 to 5                                          |
| app_version   | varchar(16)      |                                                       |
| session_hour  | datetime2(0)     | Cut to the hour on the tablet at session end          |
| received_hour | datetime2(0)     | Cut to the hour on the server; used to monitor delays |

## Consequences

Reports hide groups below the minimum group size. Open point for the DPO: tenant, hour and age bracket in a small clinic could single out a child together with an appointment calendar; consider date plus day-part instead of the hour.

## Evidence

Privacy check: field-by-field table (what it is, why we need it, direct identifier yes/no) with the DPO's verdict. Retention test: seed rows with dates in the past, run the job, count rows before and after. Five report queries with minimum group size, run on 1M seeded rows.
