# ADR-0015: Room links before accounts

- **Status:** Accepted
- **Date:** 2026-09-26

## Context
The playtest (Step 2) should reach real groups quickly. Building sign-in, invites and account
recovery first would delay learning whether the game is fun.

## Decision
- For Steps 1–2, a Table is joined through a **secret room link** (unguessable ID, ≥128-bit).
- Joining asks only for a display name and issues a random **device token**, stored on the
  device and stored **hashed** on the server. The creator's token can remove players.
- From Step 3, **Cognito passwordless** accounts (ADR-0005) replace this. Playtest players can
  claim their existing seat so history is kept.

## Consequences
- Anyone with the link can join — acceptable for a private playtest; the creator can remove people.
- Clearing browser data loses the seat until accounts exist (the creator can re-invite).
- The permission checks in `core` are written against a "player at a Table" abstraction, so
  switching from device tokens to Cognito users doesn't change game code.
