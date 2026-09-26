# ADR-0005: Cognito passwordless auth

- **Status:** Accepted
- **Date:** 2026-09-26

## Context
Forgotten passwords are the #1 reason parents get stuck. We want AWS-native auth.

## Decision
- **Amazon Cognito** user pool with passwordless sign-in: **email one-time code**, and
  **passkeys** for returning devices.
- API Gateway validates Cognito JWTs; WebSocket `$connect` validates the token too.
- SMS codes are **not** enabled in v1 (per-message cost, extra setup).

## Consequences
- No password resets to support.
- Parents need access to their email on first sign-in; passkeys remove that afterwards.
- SMS can be added later if email proves hard for a family member.
