# ADR-0005: Secrets

- Status: accepted
- Date: 2026-11-24
- Deciders: team

## Context

We want as few secrets as possible, none in the repository, and a rotation path for the token signing key.

## Options

1. Plain app settings.
2. Managed identity where possible, Key Vault for what remains.

## Decision

Option 2. Managed identity for Storage and SQL (Entra authentication, no passwords). The JWT signing key is the one remaining secret, stored in Key Vault and referenced from app settings. GitHub Actions logs in to Azure with OIDC, so no Azure credentials are stored in GitHub.

## Consequences

Slightly more Bicep (role assignments, Key Vault reference). No connection strings with passwords exist anywhere.

## Evidence

Search the repo and the app settings for secrets in CI. Run a rotation drill: new `kid`, both keys valid during an overlap window, old key removed.
