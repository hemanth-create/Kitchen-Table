# ADR-0011: Pure-TS game logic package + Phaser for arcade games

- **Status:** Accepted
- **Date:** 2026-09-26

## Context
Game rules need to run in the app (to play) and on the server (to verify results for
leaderboards). Arcade games also need a real 2D engine, and the founder wants to learn
game development in TypeScript.

## Decision
- **`packages/games`** holds all game rules as **pure, deterministic TypeScript**:
  no React, no Phaser, no AWS; randomness comes from an injected seed.
  The app uses it to play; the `api` Lambda uses it to replay and verify results.
- Puzzle-style games (word puzzle, trivia) are rendered with **React Native** components.
- Arcade games use **Phaser 3** in `apps/arcade`, rendered directly on web and inside a
  **WebView** on iOS/Android. Scores are **plausibility-checked** server-side (not fully verified).

## Consequences
- Game rules are trivially unit-testable and shared, so no rule drift between client and server.
- Phaser is well documented with a large community — a good way to learn game loops,
  sprites and collisions.
- WebView adds a bridge between Phaser and the app (score/events via `postMessage`); performance
  is fine for simple 2D games. If it isn't, React Native Skia is the fallback.
- Arcade scores can be faked by a determined user; acceptable for a family app.
