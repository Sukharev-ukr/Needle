# ADR-0002: Token format and lost-tablet revocation

- Status: accepted
- Date: 2026-11-24
- Deciders: team

## Context

The API is write-only and the client ID is read from the token. Tokens are per client today, so one lost tablet cannot be revoked without revoking all tablets of that client.

## Options

1. One token per client, revoke by secret rotation.
2. One token per client plus a deny list.
3. One token per device, issued from a one-time enrolment code, plus a deny list.

## Decision

- Format: JWT HS256 signed with `jose`, with a `kid` header so keys can rotate. Claims: `tid` (client), `did` (device, from iteration 3), `iat`.
- Iteration 1: option 1, to unblock the pipeline.
- Iteration 3: option 3. The onboarding script creates a one-time enrolment code (short lifetime, limited uses) shown as a QR code. The tablet calls `POST /api/v1/enrol` and stores the returned token in IndexedDB.
- Revocation: add the `did` to a deny list (one JSON blob or a Table Storage table). Ingest caches it in memory and refreshes every 60 s.
- The device ID never goes into a session record.

## Consequences

Option 1 revokes all tablets of a client at once. Option 3 needs one more endpoint and store, and a stolen enrolment code works until it expires. The device ID identifies an organisation's device, not a child; show this to the DPO.

## Evidence

Revoke test: a revoked device gets 401 within 60 s plus the cache refresh; other tablets keep getting 202. Key rotation drill: new `kid`, both keys valid during an overlap window, old key removed.
