# ADR-0010: Games first — family game night, not another chat app

- **Status:** Accepted (changes the product focus of the original plan)
- **Date:** 2026-09-26

## Context
The first plan was a family chat app with an AI assistant. But families already have chat
apps (WhatsApp etc.) — another one gives them no reason to switch. What's missing is a place
to *have fun together*. Separately, the founder wants to learn game development, and is also
learning TypeScript, AWS and React Native at the same time — too much to take on at once.

## Decision
- Kitchen Table is **family game night, a few minutes a day**: small async games, an AI host
  ("Chef"), and chat only as *table talk* around the games.
- Design principles: 1–5 minutes, async/turn-based, about each other, cooperative first.
- Build order is chosen so **each step teaches one new thing and ends with something playable**:
  1. Daily Word Puzzle — no AWS
  2. Families + leaderboard — first AWS
  3. Daily Question + Chef — Bedrock, schedules
  4. Arcade mini-game — Phaser
  5. Trivia, Who Knows Mom Best, table talk, scam checker
- **EventBridge Scheduler** moves into v1 (daily drops, weekly trivia and recap).
- Live real-time multiplayer is a non-goal for v1.

## Consequences
- Something fun reaches the family in weeks, not months — early feedback.
- The architecture (ADR-0001 to 0009) still holds; only priorities and a few data entities change.
- The scam checker and general AI chat drop from headline features to utilities (step 5).
- Retention now depends on game quality and a daily habit, which we measure with streaks.
