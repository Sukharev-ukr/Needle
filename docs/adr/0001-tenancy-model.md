# ADR-0001: Tenancy model

- Status: accepted (default); confirmed or overturned by the tenancy tests
- Date: 2026-11-24
- Deciders: team

## Context

Many hospitals and youth healthcare organisations (clients) will use NOVA. Client data must be separated. Sub-question 4 of the research asks which model is best.

## Options

1. A: shared database, `tenant_id` column, row-level security (RLS).
2. B: tables per client. Dropped: no advantage over A or C and the most migration work.
3. C: database per client.
4. Hybrid: A by default, C for a client that asks for physical separation.

## Decision

A is the default. The code uses a `TenancyStrategy` interface so C can be switched on with the `TENANCY` setting:

```ts
interface TenancyStrategy {
  insertRecord(
    tenantId: string,
    r: SessionRecord,
  ): Promise<"inserted" | "duplicate">;
  provisionTenant(tenantId: string): Promise<void>; // A: insert tenant row; C: create DB + migrate
  deprovisionTenant(tenantId: string): Promise<void>; // Phase 2 (client deletion), interface only
  migrateAll(migration: Migration): Promise<MigrationResult[]>;
}
```

- A: one connection pool; `sp_set_session_context 'tenant_id'` on each connection; a security policy with filter and block predicates on `tenant_id`.
- C: map tenant ID to database name; one pool per tenant, created lazily, idle pools closed.

## Consequences

A has one schema to migrate and the lowest idle cost. C gives physical separation but costs one database to migrate, monitor and pay storage for per client. The SQL free offer (10 free databases per subscription) makes C look free in the POC, so cost is compared at list price.

## Evidence

Same schema, same vCore range, same k6 script, 10 tenants per model:

1. Separation: wrong session context returns 0 rows; insert with another `tenant_id` is blocked (A); connection to another tenant's database is refused (C).
2. Time and manual steps to add a client.
3. Time to add one column to all tenants, with failures and rollback.
4. Idle cost and cost per 1,000 records at list price, for 10 and 100 tenants.
5. Busy client: one tenant at 10x load, p95 insert latency of the others vs baseline.

A wins unless a client contractually needs physical separation.
