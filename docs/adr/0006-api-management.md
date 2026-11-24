# ADR-0006: Azure API Management

- Status: accepted (not in the POC)
- Date: 2026-11-24
- Deciders: team

## Context

API Management was dropped in design V2. We have one write-only endpoint, our own token check in Ingest, and no second API consumer yet.

## Options

1. No API Management in the POC.
2. API Management (Consumption tier) in front of Ingest.

## Decision

Option 1. Add API Management when one of these happens: a second API consumer exists (the Phase 2 dashboard), per-tenant rate limits or quotas are needed, or a contract requires a central gateway or WAF.

## Consequences

Rate limiting per tenant is not available in the POC. Abuse from one tenant is limited only by Function scaling settings.

## Evidence

The busy-client and reconnect-burst load tests show whether one tenant can hurt the others. If it can, that is the trigger to add it.
