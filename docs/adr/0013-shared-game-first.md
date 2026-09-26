# ADR-0013: Family & friends, adults, shared game first

- **Status:** Accepted
- **Date:** 2026-09-26
- **Supersedes:** the build order in ADR-0010 (the games-first direction itself stands).

## Context
The games-first plan (ADR-0010) started with a solo Daily Word Puzzle. A review
(`.chat/Threads/2026-09-26-family-games-review.md`) pointed out that a solo game cannot prove the
core promise — that people come back to play *with each other*. The audience also widened:
friend groups, not only families. No children will play.

## Decision
- **Audience:** families **and** friend groups, adults only. A group is called a **Table**;
  a user can sit at several Tables.
- **First game:** *Who Knows Us Best?* — a shared, async game with written rules
  (`docs/games/who-knows-us-best.md`).
- **Validate before building:** Step 0 is a no-code WhatsApp playtest; Step 2 is a deployed
  playtest with real groups before accounts, AI or push are built.
- **Every game gets a rules page** in `docs/games/` before any code.
- The **Daily Word Puzzle and Phaser arcade** become an **optional learning track**, not the
  product's first step.

## Consequences
- The riskiest question (is it fun together?) is answered first and cheaply.
- Game-development learning still happens: Step 1 is game state, rotation, reveal and scoring
  in TypeScript; engine work is available on the optional track.
- Friend groups mean Tables must stay strictly private from each other, and tone must suit
  groups that aren't family — reflected in the question bank and Chef's persona.
- Adults only removes age-appropriateness rules, but questions stay light and non-sensitive.
