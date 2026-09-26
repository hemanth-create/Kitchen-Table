# ADR-0006: Bedrock AI calls through an SQS worker

- **Status:** Accepted
- **Date:** 2026-09-26

## Context
AI replies take seconds, can be throttled, and should stream. Sending a normal message must
never wait on the AI.

## Decision
- Messages that need AI enqueue a job on **`ai-jobs` (SQS)**.
- **`ai-worker` Lambda** calls the **Bedrock Converse API (`ConverseStream`)** and streams
  chunks to clients over the WebSocket, then saves the final message.
- Model choice: fast/low-cost Claude model for chat; stronger Claude model for scam checks.
  Model IDs live in config, not code.
- **Bedrock Guardrails** on every call. Per-user daily token budget enforced by the worker.

## Consequences
- Automatic retries and a dead-letter queue for failed AI jobs.
- A little extra latency from the queue hop (well under a second).
- The AI layer sits behind a `core` port (`AiClient`), so the provider can change without
  touching business logic.
