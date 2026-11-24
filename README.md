# NOVA cloud POC

NOVA is an Augmented Reality web app for children aged 9-12 with needle fear, used on a tablet by a JGZ nurse during vaccination. The AR session runs entirely on the tablet. Camera and microphone data never leave the device.

This repository holds the cloud backend and delivery for the proof of concept: a write-only serverless API on Azure (EU), a queue, a database, and an offline-first PWA outbox. The AR page itself is out of scope.

Research question: how can NOVA be built as a scalable and secure cloud solution that all hospitals and Youth Healthcare organisations in the Netherlands can use?

## Rules

- No camera, microphone or media data is sent to the API. There is no endpoint that reads records and none that accepts media.
- No record may be lost or duplicated. The tablet keeps a record until it gets a 202.
- The data has no direct identifiers (we do not call it anonymous).
- Everything is reproducible from code: infrastructure, pipeline and scripts.

## Structure

| Folder               | Purpose                                                                  |
| -------------------- | ------------------------------------------------------------------------ |
| `app/`               | PWA: outbox module, service worker, manifest (Vite)                      |
| `api/`               | Azure Functions (Ingest, Processor, enrol, retention) and `openapi.yaml` |
| `infra/`             | Bicep templates; `infra/sql/` holds numbered SQL migrations              |
| `scripts/`           | onboard-tenant, revoke-device, replay-poison, teardown                   |
| `tests/k6/`          | Load tests (ramp, reconnect burst)                                       |
| `tests/e2e/`         | Playwright tests (offline, update, outbox flush)                         |
| `docs/adr/`          | One architecture decision per file (copy `template.md`)                  |
| `docs/architecture/` | C4 model (Structurizr) and diagram changelog                             |
| `.github/workflows/` | CI and deploy pipelines                                                  |

## Getting started

Requires Node 22 or newer.

```bash
npm install
npm run lint:api   # lint the OpenAPI contract
npm run mock       # mock API on http://localhost:4010
```

Try the mock:

```bash
curl -i -X POST http://localhost:4010/api/v1/records \
  -H 'Authorization: Bearer test' \
  -H 'Content-Type: application/json' \
  -d '{"event_id":"3f2b8c1e-5d4a-4b7e-9a10-1c2d3e4f5a6b","age_bracket":"9-12","mode":"blow","completed":true,"duration_s":170,"app_version":"0.1.0","session_hour":"2026-11-24T13:00:00Z"}'
```

Ask the mock for an error response with the `Prefer` header, for example `-H 'Prefer: code=400'` or `-H 'Prefer: code=401'`.

## Working agreement

- Contract first: change `api/openapi.yaml` before changing code.
- No merge to `main` without one review.
- Every decision that changes the architecture gets an ADR in `docs/adr/`.

## Decisions

See `docs/adr/`. Start with `0000-region.md`: it must be filled in after the day-1 region check.
