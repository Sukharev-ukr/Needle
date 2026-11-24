# ADR-0008: Tablet calls the Function App directly

- Status: accepted
- Date: 2026-11-24
- Deciders: team

## Context

The first design routed `/api/*` through Azure Static Web Apps. Linking your own Function App to Static Web Apps needs the Standard plan, and Flex Consumption is not on Microsoft's list of supported plans for that link (a Microsoft Q&A thread reports it working and then stopping). We authenticate with our own bearer token and do not use Static Web Apps authentication.

## Options

1. Linked backend through Static Web Apps.
2. The tablet calls the Function App hostname directly, with CORS limited to the Static Web Apps origin.

## Decision

Option 2.

## Consequences

CORS is configured on the Function App. k6 measures Ingest itself and not a proxy. Static Web Apps can stay on the Free plan for tests.

## Evidence

A browser on both tablets can POST to the Function App from the installed PWA (CORS preflight succeeds). k6 results show Ingest latency without proxy overhead.
