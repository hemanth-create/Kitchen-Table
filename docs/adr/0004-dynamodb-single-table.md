# ADR-0004: DynamoDB single-table design

- **Status:** Accepted
- **Date:** 2026-09-26

## Context
Chat access patterns are few and known up front: list my families, list conversations,
page messages newest-first, find a user's live connections. We want serverless pricing.

## Decision
- One DynamoDB table per stage, generic `PK`/`SK` keys, one GSI, TTL on `expiresAt`.
- Access patterns documented in `ARCHITECTURE.md` §6; every new query must be added there first.
- On-demand capacity.

## Consequences
- Pay-per-request, no idle cost, single-digit-ms reads.
- Ad-hoc queries (analytics, reporting) are hard — export to S3/Athena if ever needed.
- Single-table design has a learning curve; the documented access-pattern table is the guide.
