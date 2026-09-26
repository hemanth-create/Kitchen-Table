# ADR-0008: No end-to-end encryption in v1

- **Status:** Accepted
- **Date:** 2026-09-26

## Context
End-to-end encryption means the server cannot read messages — which also means the AI
cannot read the chat to answer `@ai` or check for scams. It also adds major complexity
(key management, multi-device, recovery).

## Decision
No E2E encryption in v1. Instead: TLS in transit, AWS-managed encryption at rest,
strict per-family authorization, no message content in logs, private media via signed URLs.

## Consequences
- AI features work across the whole chat.
- The operator (us) could technically read data — acceptable for a family app we run ourselves;
  must be stated clearly in a privacy note before any public launch.
- Revisit if the app goes public: e.g. E2E for DMs, with AI only in opted-in chats.
