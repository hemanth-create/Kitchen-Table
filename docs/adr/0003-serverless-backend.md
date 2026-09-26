# ADR-0003: Serverless backend (API Gateway + Lambda + Hono)

- **Status:** Accepted
- **Date:** 2026-09-26

## Context
Traffic is tiny (a family) and bursty. Always-on containers + a database would cost
~$50+/month idle and need patching.

## Decision
- **API Gateway HTTP API** + **Lambda** for REST, using **Hono** as the router.
- **API Gateway WebSocket API** + **Lambda** for realtime delivery.
- **SQS** for background work (AI jobs, notifications), each with a dead-letter queue.

## Consequences
- Near-zero idle cost; scales automatically.
- Cold starts add latency on the first request after idle (acceptable; mitigated by small bundles).
- WebSocket fan-out is done by Lambda via the API Gateway management API — fine at family
  scale, would need rethinking at large scale.
