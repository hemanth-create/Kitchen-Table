# ADR-0014: Async-only v1 — no WebSockets, lazy reveals, idempotent Chef

- **Status:** Accepted
- **Date:** 2026-09-26
- **Supersedes:** the WebSocket API in ADR-0003 and streaming in ADR-0006. The rest of both stands.

## Context
The plan excluded live multiplayer but still included a WebSocket API and streamed AI replies.
Every v1 game is turn-based with a deadline, so nothing needs sub-second updates. SQS may also
deliver a job more than once, which the plan didn't handle.

## Decision
- **No WebSockets in v1.** Clients refresh on open/focus and poll lightly (e.g. every 10 s)
  only while a round screen is open. Push notifications (Step 4) bring people back.
- **Lazy reveal:** a round counts as revealed when it is read after everyone has submitted or
  its deadline has passed. No scheduled job is needed to close rounds.
- **All time rules use the Table's timezone**; deadlines are stored as UTC timestamps.
- **Chef is non-streaming and idempotent:** each game event has a stable ID
  (`ROUND#<id>#reveal`); Chef's message and usage record are saved with conditional writes,
  so retries never duplicate them. A timeout or failure shows plain fallback text.

## Consequences
- One fewer API, no connection table, no fan-out code, fewer failure modes.
- Updates can lag by a few seconds — fine for async games.
- If a future game needs live play (e.g. scheduled trivia), that needs a new ADR covering
  presence, timing and reconnection; AppSync Events or an API Gateway WebSocket API are the candidates.
