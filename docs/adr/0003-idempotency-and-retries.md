# ADR-0003: Idempotency and retry strategy

- Status: accepted
- Date: 2026-11-24
- Deciders: team

## Context

No record may be lost or duplicated, also when the tablet is offline, the network drops mid-request, or the database is paused (Azure SQL serverless takes about a minute to resume).

## Options

1. At-most-once delivery (risk: lost records).
2. At-least-once delivery with an idempotent insert.
3. Exactly-once infrastructure (not available on Storage Queue).

## Decision

Option 2.

- Client: `event_id = crypto.randomUUID()` is created when the record is written to the outbox, not when it is sent. The record is deleted only after a 202. Backoff 2 s, 4 s, 8 s, up to 5 min, with jitter. Flush on app start, on the `online` event, and after each session.
- Ingest: returns 202 only after `await queueClient.sendMessage(...)` succeeded (Storage SDK, not an output binding), so a failed enqueue becomes a 5xx.
- Database: `UNIQUE (tenant_id, event_id)`. SQL error 2627 or 2601 means duplicate and counts as success.
- Queue: `maxDequeueCount` 5 and a visibility timeout of 60 s, longer than the SQL resume time. After 5 failures the message goes to `records-poison` and an alert fires when it has more than 0 messages. A script (`scripts/replay-poison`) moves messages back after the fix.

## Consequences

Duplicates can be delivered but never stored twice. A paused database delays records instead of losing them.

## Evidence

Chaos tests: cut the network in the middle of a POST; send the same record twice; pause the database and send 100 records. Expected: 100 rows, 0 duplicates, 0 messages in the poison queue.
